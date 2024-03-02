# 你不懂 JS: 异步与性能 第四章: Generator（下）

## Generator 委托

在上一节中，我们展示了从 generator 内部调用普通函数，和它如何作为一种有用的技术来将实现细节（比如异步 Promise 流程）抽象出去。但是为这样的任务使用普通函数的缺陷是，它必须按照普通函数的规则行动，也就是说它不能像 generator 那样用`yield`来暂停自己。

在你身上可能发生这样的事情：你可能会试着使用我们的`run(..)`帮助函数，从一个 generator 中调用另个一 generator。比如：

```js
function *foo() {
    var r2 = yield request( "http://some.url.2" );
    var r3 = yield request( "http://some.url.3/?v=" + r2 );

    return r3;
}

function *bar() {
    var r1 = yield request( "http://some.url.1" );

    // 通过`run(..)`“委托”到`*foo()`
    var r3 = yield run( foo );

    console.log( r3 );
}

run( bar );
```

通过再一次使用我们的`run(..)`工具，我们在`*bar()`内部运行`*foo()`。我们利用了这样一个事实：我们早先定义的`run(..)`返回一个 promise，这个 promise 在 generator 运行至完成时才解析（或发生错误），所以如果我们从一个`run(..)`调用中`yield`出一个 promise 给另一个`run(..)`，它就会自动暂停`*bar()`直到`*foo()`完成。

但这里有一个更好的办法将`*foo()`调用整合进`*bar()`，它称为`yield`委托。`yield`委托的特殊语法是：`yield * __`（注意额外的`*`）。让它在我们前面的例子中工作之前，让我们看一个更简单的场景：

```js
function *foo() {
    console.log( "`*foo()` starting" );
    yield 3;
    yield 4;
    console.log( "`*foo()` finished" );
}

function *bar() {
    yield 1;
    yield 2;
    yield *foo();    // `yield`-delegation!
    yield 5;
}

var it = bar();

it.next().value;    // 1
it.next().value;    // 2
it.next().value;    // `*foo()` starting
                    // 3
it.next().value;    // 4
it.next().value;    // `*foo()` finished
                    // 5
```

**注意：** 在本章早前的一个注意点中，我解释了为什么我偏好`function *foo() ..`而不是`function* foo() ..`，相似地，我也偏好——与关于这个话题的其他大多数文档不同——说`yield *foo()`而不是`yield* foo()`。`*`的摆放是纯粹的风格问题，而且要看你的最佳判断。但我发现保持统一风格很吸引人。

`yield *foo()`委托是如何工作的？

首先，正如我们看到过的那样，调用`foo()`创建了一个 *迭代器*。然后，`yield *`将（当前`*bar()`generator 的） *迭代器* 的控制委托/传递给这另一个`*foo()`*迭代器*。

那么，前两个`it.next()`调用控制着`*bar()`，但当我们发起第三个`it.next()`调用时，`*foo()`就启动了，而且这时我们控制的是`*foo()`而非`*bar()`。这就是为什么它称为委托——`*bar()`将它的迭代控制委托给`*foo()`。

只要`it`*迭代器* 的控制耗尽了整个`*foo()`*迭代器*，它就会自动地将控制返回到`*bar()`。

那么现在回到前面的三个顺序 Ajax 请求的例子：

```js
function *foo() {
    var r2 = yield request( "http://some.url.2" );
    var r3 = yield request( "http://some.url.3/?v=" + r2 );

    return r3;
}

function *bar() {
    var r1 = yield request( "http://some.url.1" );

    // 通过`run(..)`“委托”到`*foo()`
    var r3 = yield *foo();

    console.log( r3 );
}

run( bar );
```

这个代码段和前面使用的版本的唯一区别是，使用了`yield *foo()`而不是前面的`yield run(foo)`。

**注意：** `yield *`让出了迭代控制，不是 generator 控制；当你调用`*foo()`generator 时，你就`yield`委托给它的 *迭代器*。但你实际上可以`yield`委托给任何 *迭代器*；`yield *[1,2,3]`将会消费默认的`[1,2,3]`数组值 *迭代器*。

### 为什么委托？

`yield`委托的目的很大程度上是为了代码组织，而且这种方式是与普通函数调用对称的。

想象两个分别提供了`foo()`和`bar()`方法的模块，其中`bar()`调用`foo()`。它们俩分开的原因一般是由于为了程序将它们作为分离的程序来调用而进行的恰当组织。例如，可能会有一些情况`foo()`需要被独立调用，而其他地方`bar()`来调用`foo()`。

由于这些完全相同的原因，将 generator 分开可以增强程序的可读性，可维护性，与可调试性。从这个角度讲，`yield *`是一种快捷的语法，用来在`*bar()`内部手动地迭代`*foo()`的步骤。

如果`*foo()`中的步骤是异步的，这样的手动方式可能会特别复杂，这就是为什么你可能会需要那个`run(..)`工具来做它。正如我们已经展示的，`yield *foo()`消灭了使用`run(..)`工具的子实例（比如`run(foo)`）的需要。

### 委托消息

你可能想知道，这种`yield`委托在除了与 *迭代器* 控制一起工作以外，是如何与双向消息传递一起工作的。仔细查看下面这些通过`yield`委托进进出出的消息流：

```js
function *foo() {
    console.log( "inside `*foo()`:", yield "B" );

    console.log( "inside `*foo()`:", yield "C" );

    return "D";
}

function *bar() {
    console.log( "inside `*bar()`:", yield "A" );

    // `yield`-委托！
    console.log( "inside `*bar()`:", yield *foo() );

    console.log( "inside `*bar()`:", yield "E" );

    return "F";
}

var it = bar();

console.log( "outside:", it.next().value );
// outside: A

console.log( "outside:", it.next( 1 ).value );
// inside `*bar()`: 1
// outside: B

console.log( "outside:", it.next( 2 ).value );
// inside `*foo()`: 2
// outside: C

console.log( "outside:", it.next( 3 ).value );
// inside `*foo()`: 3
// inside `*bar()`: D
// outside: E

console.log( "outside:", it.next( 4 ).value );
// inside `*bar()`: 4
// outside: F
```

特别注意一下`it.next(3)`调用之后的处理步骤：

1.  值`3`被传入（通过`*bar`里的`yield`委托）在`*foo()`内部等待中的`yield "C"`表达式。
2.  然后`*foo()`调用`return "D"`，但是这个值不会一路返回到外面的`it.next(3)`调用。
3.  相反地，值`"D"`作为结果被发送到在`*bar()`内部等待中的`yield *foo()`表示式——这个`yield`委托表达式实质上在`*foo()`被耗尽之前一直被暂停着。所以`"D"`被送到`*bar()`内部来让它打印。
4.  `yield "E"`在`*bar()`内部被调用，而且值`"E"`被让出到外部作为`it.next(3)`调用的结果。

从外部 *迭代器*（`it`）的角度来看，在初始的 generator 和被委托的 generator 之间的控制没有任何区别。

事实上，`yield`委托甚至不必指向另一个 generator；它可以仅被指向一个非 generator 的，一般的 *iterable*。比如：

```js
function *bar() {
    console.log( "inside `*bar()`:", yield "A" );

    // `yield`-委托至一个非 generator
    console.log( "inside `*bar()`:", yield *[ "B", "C", "D" ] );

    console.log( "inside `*bar()`:", yield "E" );

    return "F";
}

var it = bar();

console.log( "outside:", it.next().value );
// outside: A

console.log( "outside:", it.next( 1 ).value );
// inside `*bar()`: 1
// outside: B

console.log( "outside:", it.next( 2 ).value );
// outside: C

console.log( "outside:", it.next( 3 ).value );
// outside: D

console.log( "outside:", it.next( 4 ).value );
// inside `*bar()`: undefined
// outside: E

console.log( "outside:", it.next( 5 ).value );
// inside `*bar()`: 5
// outside: F
```

注意这个例子与前一个之间，被接收/报告的消息的不同之处。

最惊人的是，默认的`array`*迭代器* 不关心任何通过`next(..)`调用被发送的消息，所以值`2`，`3`，与`4`实质上被忽略了。另外，因为这个 *迭代器* 没有明确的`return`值（不像前面使用的`*foo()`），所以`yield *`表达式在它完成时得到一个`undefined`。

#### 异常也委托！

与`yield`委托在两个方向上透明地传递消息的方式相同，错误/异常也在双向传递：

```js
function *foo() {
    try {
        yield "B";
    }
    catch (err) {
        console.log( "error caught inside `*foo()`:", err );
    }

    yield "C";

    throw "D";
}

function *bar() {
    yield "A";

    try {
        yield *foo();
    }
    catch (err) {
        console.log( "error caught inside `*bar()`:", err );
    }

    yield "E";

    yield *baz();

    // note: can't get here!
    yield "G";
}

function *baz() {
    throw "F";
}

var it = bar();

console.log( "outside:", it.next().value );
// outside: A

console.log( "outside:", it.next( 1 ).value );
// outside: B

console.log( "outside:", it.throw( 2 ).value );
// error caught inside `*foo()`: 2
// outside: C

console.log( "outside:", it.next( 3 ).value );
// error caught inside `*bar()`: D
// outside: E

try {
    console.log( "outside:", it.next( 4 ).value );
}
catch (err) {
    console.log( "error caught outside:", err );
}
// error caught outside: F
```

在这段代码中有一些事情要注意：

1.  但我们调用`it.throw(2)`时，它发送一个错误消息`2`到`*bar()`，而`*bar()`将它委托至`*foo()`，然后`*foo()`来`catch`它并平静地处理。之后，`yield "C"`把`"C"`作为返回的`value`发送回`it.throw(2)`调用。
2.  接下来值`"D"`被从`*foo()`内部`throw`出来并传播到`*bar()`，`*bar()`会`catch`它并平静地处理。然后`yield "E"`把`"E"`作为返回的`value`发送回`it.next(3)`调用。
3.  接下来，一个异常从`*baz()`中`throw`出来，而没有被`*bar()`捕获——我们没在外面`catch`它——所以`*baz()`和`*bar()`都被设置为完成状态。这段代码结束后，即便有后续的`next(..)`调用，你也不会得到值`"G"`——它们的`value`将返回`undefined`。

### 异步委托

最后让我们回到早先的多个顺序 Ajax 请求的例子，使用`yield`委托：

```js
function *foo() {
    var r2 = yield request( "http://some.url.2" );
    var r3 = yield request( "http://some.url.3/?v=" + r2 );

    return r3;
}

function *bar() {
    var r1 = yield request( "http://some.url.1" );

    var r3 = yield *foo();

    console.log( r3 );
}

run( bar );
```

在`*bar()`内部，与调用`yield run(foo)`不同的是，我们调用`yield *foo()`就可以了。

在前一个版本的这个例子中，Promise 机制（通过`run(..)`控制的）被用于将值从`*foo()`中的`return r3`传送到`*bar()`内部的本地变量`r3`。现在，这个值通过`yield *`机制直接返回。

除此以外，它们的行为是一样的。

### “递归”委托

当然，`yield`委托可以一直持续委托下去，你想连接多少步骤就连接多少。你甚至可以在具有异步能力的 generator 上“递归”使用`yield`委托——一个`yield`委托至自己的 generator：

```js
function *foo(val) {
    if (val > 1) {
        // 递归委托
        val = yield *foo( val - 1 );
    }

    return yield request( "http://some.url/?v=" + val );
}

function *bar() {
    var r1 = yield *foo( 3 );
    console.log( r1 );
}

run( bar );
```

**注意：** 我们的`run(..)`工具本可以用`run( foo, 3 )`来调用，因为它支持用额外传递的参数来进行 generator 的初始化。然而，为了在这里高调展示`yield *`的灵活性，我们使用了无参数的`*bar()`。

这段代码之后的处理步骤是什么？坚持住，它的细节要描述起来可是十分错综复杂：

1.  `run(bar)`启动了`*bar()`generator。
2.  `foo(3)`为`*foo(..)`创建了 *迭代器* 并传递`3`作为它的`val`参数。
3.  因为`3 > 1`，`foo(2)`创建了另一个 *迭代器* 并传递`2`作为它的`val`参数。
4.  因为`2 > 1`，`foo(1)`又创建了另一个 *迭代器* 并传递`1`作为它的`val`参数。
5.  `1 > 1`是`false`，所以我们接下来用值`1`调用`request(..)`，并得到一个代表第一个 Ajax 调用的 promise。
6.  这个 promise 被`yield`出来，回到`*foo(2)`generator 实例。
7.  `yield *`将这个 promise 传出并回到`*foo(3)`生成 generator。另一个`yield *`把这个 promise 传出到`*bar()`generator 实例。而又有另一个`yield *`把这个 promise 传出到`run(..)`工具，而它将会等待这个 promise（第一个 Ajax 请求）再处理。
8.  当这个 promise 解析时，它的完成消息会被发送以继续`*bar()`，`*bar()`通过`yield *`把消息传递进`*foo(3)`实例，`*foo(3)`实例通过`yield *`把消息传递进`*foo(2)`generator 实例，`*foo(2)`实例通过`yield *`把消息传给那个在`*foo(3)`generator 实例中等待的一般的`yield`。
9.  这第一个 Ajax 调用的应答现在立即从`*foo(3)`generator 实例中被`return`，作为`*foo(2)`实例中`yield *`表达式的结果发送回来，并赋值给本地`val`变量。
10.  `*foo(2)`内部，第二个 Ajax 请求用`request(..)`发起，它的 promise 被`yield`回到`*foo(1)`实例，然后一路`yield *`传播到`run(..)`（回到第 7 步）。当 promise 解析时，第二个 Ajax 应答一路传播回到`*foo(2)`generator 实例，并赋值到他本地的`val`变量。
11.  最终，第三个 Ajax 请求用`request(..)`发起，它的 promise 走出到`run(..)`，然后它的解析值一路返回，最后被`return`到在`*bar()`中等待的`yield *`表达式。

天！许多疯狂的头脑杂技，对吧？你可能想要把它通读几遍，然后抓点儿零食放松一下大脑！

## Generator 并发

正如我们在第一章和本章早先讨论过的，另个同时运行的“进程”可以协作地穿插它们的操作，而且许多时候这可以产生非常强大的异步表达式。

坦白地说，我们前面关于多个 generator 并发穿插的例子，展示了这真的容易让人糊涂。但我们也受到了启发，有些地方这种能力十分有用。

回想我们在第一章中看过的场景，两个不同但同时的 Ajax 应答处理需要互相协调，来确保数据通信不是竟合状态。我们这样把应答分别放在`res`数组的不同位置中：

```js
function response(data) {
    if (data.url == "http://some.url.1") {
        res[0] = data;
    }
    else if (data.url == "http://some.url.2") {
        res[1] = data;
    }
}
```

但是我们如何在这种场景下使用多 generator 呢？

```js
// `request(..)` 是一个基于 Promise 的 Ajax 工具

var res = [];

function *reqData(url) {
    res.push(
        yield request( url )
    );
}
```

**注意：** 我们将在这里使用两个`*reqData(..)`generator 的实例，但是这和分别使用两个不同 generator 的一个实例没有区别；这两种方式在道理上完全一样的。我们过一会儿就会看到两个 generator 的协调操作。

与不得不将`res[0]`和`res[1]`赋值手动排序不同，我们将使用协调过的顺序，让`res.push(..)`以可预见的顺序恰当地将值放在预期的位置。如此被表达的逻辑会让人感觉更干净。

但是我们将如何实际安排这种互动呢？首先，让我们手动实现它：

```js
var it1 = reqData( "http://some.url.1" );
var it2 = reqData( "http://some.url.2" );

var p1 = it1.next().value;
var p2 = it2.next().value;

p1
.then( function(data){
    it1.next( data );
    return p2;
} )
.then( function(data){
    it2.next( data );
} );
```

`*reqData(..)`的两个实例都开始发起它们的 Ajax 请求，然后用`yield`暂停。之后我们再`p1`解析时继续运行第一个实例，而后来的`p2`的解析将会重启第二个实例。以这种方式，我们使用 Promise 的安排来确保`res[0]`将持有第一个应答，而`res[1]`持有第二个应答。

但坦白地说，这是可怕的手动，而且它没有真正让 generator 组织它们自己，而那才是真正的力量。让我们用不同的方法试一下：

```js
// `request(..)` 是一个基于 Promise 的 Ajax 工具

var res = [];

function *reqData(url) {
    var data = yield request( url );

    // 传递控制权
    yield;

    res.push( data );
}

var it1 = reqData( "http://some.url.1" );
var it2 = reqData( "http://some.url.2" );

var p1 = it1.next().value;
var p2 = it2.next().value;

p1.then( function(data){
    it1.next( data );
} );

p2.then( function(data){
    it2.next( data );
} );

Promise.all( [p1,p2] )
.then( function(){
    it1.next();
    it2.next();
} );
```

好的，这看起来好些了（虽然仍然是手动），因为现在两个`*reqData(..)`的实例真正地并发运行了，而且（至少是在第一部分）是独立的。

在前一个代码段中，第二个实例在第一个实例完全完成之前没有给出它的数据。但是这里，只要它们的应答一返回这两个实例就立即分别收到他们的数据，然后每个实例调用另一个`yield`来传送控制。最后我们在`Promise.all([ .. ])`的处理器中选择用什么样的顺序继续它们。

可能不太明显的是，这种方式因其对称性启发了一种可复用工具的简单形式。让我们想象使用一个称为`runAll(..)`的工具：

```js
// `request(..)` 是一个基于 Promise 的 Ajax 工具

var res = [];

runAll(
    function*(){
        var p1 = request( "http://some.url.1" );

        // 传递控制权
        yield;

        res.push( yield p1 );
    },
    function*(){
        var p2 = request( "http://some.url.2" );

        // 传递控制权
        yield;

        res.push( yield p2 );
    }
);
```

**注意：** 我们没有包含`runAll(..)`的实现代码，不仅因为它长得无法行文，也因为它是一个我们已经在先前的 `run(..)`中实现的逻辑的扩展。所以，作为留给读者的一个很好的补充性练习，请你自己动手改进`run(..)`的代码，来使它像想象中的`runAll(..)`那样工作。另外，我的 *asynquence* 库提供了一个前面提到过的`runner(..)`工具，它内建了这种能力，我们将在本书的附录 A 中讨论它。

这是`runAll(..)`内部的处理将如何操作：

1.  第一个 generator 得到一个代表从`"http://some.url.1"`来的 Ajax 应答，然后将控制权`yield`回到`runAll(..)`工具。
2.  第二个 generator 运行，并对`"http://some.url.2"`做相同的事，将控制权`yield`回到`runAll(..)`工具。
3.  第一个 generator 继续，然后`yield`出他的 promise`p1`。在这种情况下`runAll(..)`工具和我们前面的`run(..)`做同样的事，它等待 promise 解析，然后继续这同一个 generator（没有控制传递！）。当`p1`解析时，`runAll(..)`使用解析值再一次继续第一个 generator，而后`res[0]`得到它的值。在第一个 generator 完成之后，有一个隐式的控制权传递。
4.  第二个 generator 继续，`yield`出它的 promise`p2`，并等待它的解析。一旦`p2`解析，`runAll(..)`使用这个解析值继续第二个 generator，于是`res[1]`被设置。

在这个例子中，我们使用了一个称为`res`的外部变量来保存两个不同的 Ajax 应答的结果——这是我们的并发协调。

但是这样做可能十分有帮助：进一步扩展`runAll(..)`使它为多个 generator 实例提供 *分享的* 内部的变量作用域，比如一个我们将在下面称为`data`的空对象。另外，它可以接收被`yield`的非 Promise 值，并把它们交给下一个 generator。

考虑这段代码：

```js
// `request(..)` 是一个基于 Promise 的 Ajax 工具

runAll(
    function*(data){
        data.res = [];

        // 传递控制权（并传递消息）
        var url1 = yield "http://some.url.2";

        var p1 = request( url1 ); // "http://some.url.1"

        // 传递控制权
        yield;

        data.res.push( yield p1 );
    },
    function*(data){
        // 传递控制权（并传递消息）
        var url2 = yield "http://some.url.1";

        var p2 = request( url2 ); // "http://some.url.2"

        // 传递控制权
        yield;

        data.res.push( yield p2 );
    }
);
```

在这个公式中，两个 generator 不仅协调控制传递，实际上还互相通信：通过`data.res`，和交换`url1`与`url2`的值的`yield`消息。这强大到不可思议！

这样的认识也是一种更为精巧的称为 CSP（Communicating Sequential Processes——通信顺序处理）的异步技术的概念基础，我们将在本书的附录 B 中讨论它。

## Thunks

至此，我们都假定从一个 generator 中`yield`一个 Promise——让这个 Promise 使用像`run(..)`这样的帮助工具来推进 generator——是管理使用 generator 的异步处理的最佳方法。明白地说，它是的。

但是我们跳过了一个被轻度广泛使用的模式，为了完整性我们将简单地看一看它。

在一般的计算机科学中，有一种老旧的前 JS 时代的概念，称为“thunk”。我们不在这里赘述它的历史，一个狭隘的表达是，thunk 是一个 JS 函数——没有任何参数——它连接并调用另一个函数。

换句话讲，你用一个函数定义包装函数调用——带着它需要的所有参数——来 *推迟* 这个调用的执行，而这个包装用的函数就是 thunk。当你稍后执行 thunk 时，你最终会调用那个原始的函数。

举个例子：

```js
function foo(x,y) {
    return x + y;
}

function fooThunk() {
    return foo( 3, 4 );
}

// 稍后

console.log( fooThunk() );    // 7
```

所以，一个同步的 thunk 是十分直白的。但是一个异步的 thunk 呢？我们实质上可以扩展这个狭隘的 thunk 定义，让它接收一个回调。

考虑这段代码：

```js
function foo(x,y,cb) {
    setTimeout( function(){
        cb( x + y );
    }, 1000 );
}

function fooThunk(cb) {
    foo( 3, 4, cb );
}

// 稍后

fooThunk( function(sum){
    console.log( sum );        // 7
} );
```

如你所见，`fooThunk(..)`仅需要一个`cb(..)`参数，因为它已经预先制定了值`3`和`4`（分别为`x`和`y`）并准备传递给`foo(..)`。一个 thunk 只是在外面耐心地等待着它开始工作所需的最后一部分信息：回调。

但是你不会想要手动制造 thunk。那么，让我们发明一个工具来为我们进行这种包装。

考虑这段代码：

```js
function thunkify(fn) {
    var args = [].slice.call( arguments, 1 );
    return function(cb) {
        args.push( cb );
        return fn.apply( null, args );
    };
}

var fooThunk = thunkify( foo, 3, 4 );

// 稍后

fooThunk( function(sum) {
    console.log( sum );        // 7
} );
```

**提示：** 这里我们假定原始的（`foo(..)`）函数签名希望它的回调的位置在最后，而其它的参数在这之前。这是一个异步 JS 函数的相当普遍的“标准”。你可以称它为“回调后置风格”。如果因为某些原因你需要处理“回调优先风格”的签名，你只需要制造一个使用`args.unshift(..)`而非`args.push(..)`的工具。

前面的`thunkify(..)`公式接收`foo(..)`函数的引用，和任何它所需的参数，并返回 thunk 本身（`fooThunk(..)`）。然而，这并不是你将在 JS 中发现的 thunk 的典型表达方式。

与`thunkify(..)`制造 thunk 本身相反，典型的——可能有点儿让人困惑的——`thunkify(..)`工具将产生一个制造 thunk 的函数。

额...是的。

考虑这段代码：

```js
function thunkify(fn) {
    return function() {
        var args = [].slice.call( arguments );
        return function(cb) {
            args.push( cb );
            return fn.apply( null, args );
        };
    };
}
```

这里主要的不同之处是有一个额外的`return function() { .. }`。这是它在用法上的不同：

```js
var whatIsThis = thunkify( foo );

var fooThunk = whatIsThis( 3, 4 );

// 稍后

fooThunk( function(sum) {
    console.log( sum );        // 7
} );
```

明显地，这段代码隐含的最大的问题是，`whatIsThis`叫什么合适？它不是 thunk，它是一个从`foo(..)`调用生产 thunk 的东西。它是一种“thunk”的“工厂”。而且看起来没有任何标准的意见来命名这种东西。

所以，我的提议是“thunkory”（"thunk" + "factory"）。于是，`thunkify(..)`制造了一个 thunkory，而一个 thunkory 制造 thunks。这个道理与第三章中我的“promisory”提议是对称的：

```js
var fooThunkory = thunkify( foo );

var fooThunk1 = fooThunkory( 3, 4 );
var fooThunk2 = fooThunkory( 5, 6 );

// 稍后

fooThunk1( function(sum) {
    console.log( sum );        // 7
} );

fooThunk2( function(sum) {
    console.log( sum );        // 11
} );
```

**注意：** 这个例子中的`foo(..)`期望的回调不是“错误优先风格”。当然，“错误优先风格”更常见。如果`foo(..)`有某种合理的错误发生机制，我们可以改变而使它期望并使用一个错误优先的回调。后续的`thunkify(..)`不会关心回调被预想成什么样。用法的唯一区别是`fooThunk1(function(err,sum){..`。

暴露出 thunkory 方法——而不是像早先的`thunkify(..)`那样将中间步骤隐藏起来——可能看起来像是没必要的混乱。但是一般来讲，在你的程序一开始就制造一些 thunkory 来包装既存 API 的方法是十分有用的，然后你就可以在你需要 thunk 的时候传递并调用这些 thunkory。这两个区别开的步骤保证了功能上更干净的分离。

来展示一下的话：

```js
// 更干净：
var fooThunkory = thunkify( foo );

var fooThunk1 = fooThunkory( 3, 4 );
var fooThunk2 = fooThunkory( 5, 6 );

// 而这个不干净：
var fooThunk1 = thunkify( foo, 3, 4 );
var fooThunk2 = thunkify( foo, 5, 6 );
```

不管你是否愿意明确对付 thunkory，thunk（`fooThunk1(..)`和`fooThunk2(..)`）的用法还是一样的。

### s/promise/thunk/

那么所有这些 thunk 的东西与 generator 有什么关系？

一般性地比较一下 thunk 和 promise：它们是不能直接互换的，因为它们在行为上不是等价的。比起单纯的 thunk，Promise 可用性更广泛，而且更可靠。

但从另一种意义上讲，它们都可以被看作是对一个值的请求，这个请求可能被异步地应答。

回忆第三章，我们定义了一个工具来 promise 化一个函数，我们称之为`Promise.wrap(..)`——我们本来也可以叫它`promisify(..)`的！这个 Promise 化包装工具不会生产 Promise；它生产那些继而生产 Promise 的 promisories。这和我们当前讨论的 thunkory 和 thunk 是完全对称的。

为了描绘这种对称性，让我们首先将`foo(..)`的例子改为假定一个“错误优先风格”回调的形式：

```js
function foo(x,y,cb) {
    setTimeout( function(){
        // 假定 `cb(..)` 是“错误优先风格”
        cb( null, x + y );
    }, 1000 );
}
```

现在，我们将比较`thunkify(..)`和`promisify(..)`（也就是第三章的`Promise.wrap(..)`）：

```js
// 对称的：构建问题的回答者
var fooThunkory = thunkify( foo );
var fooPromisory = promisify( foo );

// 对称的：提出问题
var fooThunk = fooThunkory( 3, 4 );
var fooPromise = fooPromisory( 3, 4 );

// 取得 thunk 的回答
fooThunk( function(err,sum){
    if (err) {
        console.error( err );
    }
    else {
        console.log( sum );        // 7
    }
} );

// 取得 promise 的回答
fooPromise
.then(
    function(sum){
        console.log( sum );        // 7
    },
    function(err){
        console.error( err );
    }
);
```

thunkory 和 promisory 实质上都是在问一个问题（一个值），thunk 的`fooThunk`和 promise 的`fooPromise`分别代表这个问题的未来的答案。这样看来，对称性就清楚了。

带着这个视角，我们可以看到为了异步而`yield`Promise 的 generator，也可以为异步而`yield`thunk。我们需要的只是一个更聪明的`run(..)`工具（就像以前一样），它不仅可以寻找并连接一个被`yield`的 Promise，而且可以给一个被`yield`的 thunk 提供回调。

考虑这段代码：

```js
function *foo() {
    var val = yield request( "http://some.url.1" );
    console.log( val );
}

run( foo );
```

在这个例子中，`request(..)`既可以是一个返回一个 promise 的 promisory，也可以是一个返回一个 thunk 的 thunkory。从 generator 的内部代码逻辑的角度看，我们不关心这个实现细节，这就它强大的地方！

所以，`request(..)`可以使以下任何一种形式：

```js
// promisory `request(..)` （见第三章）
var request = Promise.wrap( ajax );

// vs.

// thunkory `request(..)`
var request = thunkify( ajax );
```

最后，作为一个让我们早先的`run(..)`工具支持 thunk 的补丁，我们可能会需要这样的逻辑：

```js
// ..
// 我们收到了一个回调吗？
else if (typeof next.value == "function") {
    return new Promise( function(resolve,reject){
        // 使用一个错误优先回调调用 thunk
        next.value( function(err,msg) {
            if (err) {
                reject( err );
            }
            else {
                resolve( msg );
            }
        } );
    } )
    .then(
        handleNext,
        function handleErr(err) {
            return Promise.resolve(
                it.throw( err )
            )
            .then( handleResult );
        }
    );
}
```

现在，我们 generator 既可以调用 promisory 来`yield`Promise，也可以调用 thunkory 来`yield`thunk，而不论那种情况，`run(..)`都将处理这个值并等待它的完成，以继续 generator。

在对称性上，这两个方式是看起来相同的。然而，我们应当指出这仅仅从 Promise 或 thunk 表示延续 generator 的未来值的角度讲是成立的。

从更高的角度讲，与 Promise 被设计成的那样不同，thunk 没有提供，它们本身也几乎没有任何可靠性和可组合性的保证。在这种特定的 generator 异步模式下使用一个 thunk 作为 Promise 的替代品是可以工作的，但与 Promise 提供的所有好处相比，这应当被看做是一种次理想的方法。

如果你有选择，那就偏向`yield pr`而非`yield th`。但是使`run(..)`工具可以处理两种类型的值本身没有什么问题。

**注意：** 在我们将要在附录 A 中讨论的，我的 *asynquence* 库中的`runner(..)`工具，可以处理`yield`的 Promise，thunk 和 *asynquence* 序列。

## 前 ES6 时代的 Generator

我希望你已经被说服了，generator 是一个异步编程工具箱里的非常重要的增强工具。但它是 ES6 中的新语法，这意味着你不能像填补 Promise（它只是新的 API）那样填补 generator。那么如果我们不能奢望忽略前 ES6 时代的浏览器，我们该如何将 generator 带到浏览器中呢？

对所有 ES6 中的新语法的扩展，有一些工具——称呼他们最常见的名词是转译器（transpilers），也就是转换编译器（trans-compilers）——它们会拿起你的 ES6 语法，并转换为前 ES6 时代的等价代码（但是明显地变难看了！）。所以，generator 可以被转译为具有相同行为但可以在 ES5 或以下版本进行工作的代码。

但是怎么做到的？`yield`的“魔法”听起来不像是那么容易转译的。在我们早先的基于闭包的 *迭代器* 例子中，实际上提示了一种解决方法。

### 手动变形

在我们讨论转译器之前，让我们延伸一下，在 generator 的情况下如何手动转译。这不仅是一个学院派的练习，因为这样做实际上可以帮助我们进一步理解它们如何工作。

考虑这段代码：

```js
// `request(..)` 是一个支持 Promise 的 Ajax 工具

function *foo(url) {
    try {
        console.log( "requesting:", url );
        var val = yield request( url );
        console.log( val );
    }
    catch (err) {
        console.log( "Oops:", err );
        return false;
    }
}

var it = foo( "http://some.url.1" );
```

第一个要注意的事情是，我们仍然需要一个可以被调用的普通的`foo()`函数，而且它仍然需要返回一个 *迭代器*。那么让我们来画出非 generator 的变形草图：

```js
function foo(url) {

    // ..

    // 制造并返回 iterator
    return {
        next: function(v) {
            // ..
        },
        throw: function(e) {
            // ..
        }
    };
}

var it = foo( "http://some.url.1" );
```

下一个需要注意的地方是，generator 通过挂起它的作用域/状态来施展它的“魔法”，但我们可以用函数闭包来模拟。为了理解如何写出这样的代码，我们将先用状态值注释 generator 不同的部分：

```js
// `request(..)` 是一个支持 Promise 的 Ajax 工具

function *foo(url) {
    // 状态 *1*

    try {
        console.log( "requesting:", url );
        var TMP1 = request( url );

        // 状态 *2*
        var val = yield TMP1;
        console.log( val );
    }
    catch (err) {
        // 状态 *3*
        console.log( "Oops:", err );
        return false;
    }
}
```

**注意：** 为了更准去地讲解，我们使用`TMP1`变量将`val = yield request..`语句分割为两部分。`request(..)`发生在状态`*1*`，而将完成值赋给`val`发生在状态`*2*`。在我们将代码转换为非 generator 的等价物后，我们就可以摆脱中间的`TMP1`。

换句话所，`*1*`是初始状态，`*2*`是`request(..)`成功的状态，`*3*`是`request(..)`失败的状态。你可能会想象额外的`yield`步骤将如何编码为额外的状态。

回到我们被转译的 generator，让我们在这个闭包中定义一个变量`state`，用它来追踪状态：

```js
function foo(url) {
    // 管理 generator 状态
    var state;

    // ..
}
```

现在，让我们在闭包内部定义一个称为`process(..)`的内部函数，它用`switch`语句来处理各种状态。

```js
// `request(..)` 是一个支持 Promise 的 Ajax 工具

function foo(url) {
    // 管理 generator 状态
    var state;

    // generator-范围的变量声明
    var val;

    function process(v) {
        switch (state) {
            case 1:
                console.log( "requesting:", url );
                return request( url );
            case 2:
                val = v;
                console.log( val );
                return;
            case 3:
                var err = v;
                console.log( "Oops:", err );
                return false;
        }
    }

    // ..
}
```

在我们的 generator 中每种状态都在`switch`语句中有它自己的`case`。每当我们需要处理一个新状态时，`process(..)`就会被调用。我们一会就回来讨论它如何工作。

对任何 generator 范围的变量声明（`val`），我们将它们移动到`process(..)`外面的`var`声明中，这样它们就可以在`process(..)`的多次调用中存活下来。但是“块儿作用域”的`err`变量仅在`*3*`状态下需要，所以我们将它留在原处。

在状态`*1*`，与`yield request(..)`相反，我们`return request(..)`。在终结状态`*2*`，没有明确的`return`，所以我们仅仅`return;`也就是`return undefined`。在终结状态`*3*`，有一个`return false`，我们保留它。

现在我们需要定义 *迭代器* 函数的代码，以便人们恰当地调用`process(..)`：

```js
function foo(url) {
    // 管理 generator 状态
    var state;

    // generator-范围的变量声明
    var val;

    function process(v) {
        switch (state) {
            case 1:
                console.log( "requesting:", url );
                return request( url );
            case 2:
                val = v;
                console.log( val );
                return;
            case 3:
                var err = v;
                console.log( "Oops:", err );
                return false;
        }
    }

    // 制造并返回 iterator
    return {
        next: function(v) {
            // 初始状态
            if (!state) {
                state = 1;
                return {
                    done: false,
                    value: process()
                };
            }
            // 成功地让出继续值
            else if (state == 1) {
                state = 2;
                return {
                    done: true,
                    value: process( v )
                };
            }
            // generator 已经完成了
            else {
                return {
                    done: true,
                    value: undefined
                };
            }
        },
        "throw": function(e) {
            // 在状态 *1* 中，有唯一明确的错误处理
            if (state == 1) {
                state = 3;
                return {
                    done: true,
                    value: process( e )
                };
            }
            // 否则，是一个不会被处理的错误，所以我们仅仅把它扔回去
            else {
                throw e;
            }
        }
    };
}
```

这段代码如何工作？

1.  第一个对 *迭代器* 的`next()`调用将把 gtenerator 从未初始化的状态移动到状态`1`，然后调用`process()`来处理这个状态。`request(..)`的返回值是一个代表 Ajax 应答的 promise，它作为`value`属性从`next()`调用被返回。
2.  如果 Ajax 请求成功，第二个`next(..)`调用应当送进 Ajax 的应答值，它将我们的状态移动到`2`。`process(..)`再次被调用（这次它被传入 Ajax 应答的值），而从`next(..)`返回的`value`属性将是`undefined`。
3.  然而，如果 Ajax 请求失败，应当用错误调用`throw(..)`，它将状态从`1`移动到`3`（而不是`2`）。`process(..)`再一次被调用，这词被传入了错误的值。这个`case`返回`false`，所以`false`作为`throw(..)`调用返回的`value`属性。

从外面看——也就是仅仅与 *迭代器* 互动——这个普通的`foo(..)`函数与`*foo(..)`generator 的工作方式是一样的。所以我们有效地将 ES6 generator“转译”为前 ES6 可兼容的！

然后我们就可以手动初始化我们的 generator 并控制它的迭代器——调用`var it = foo("..")`和`it.next(..)`等等——或更好地，我们可以将它传递给我们先前定义的`run(..)`工具，比如`run(foo,"..")`。

### 自动转译

前面的练习——手动编写从 ES6 generator 到前 ES6 的等价物的变形过程——教会了我们 generator 在概念上是如何工作的。但是这种变形真的是错综复杂，而且不能很好地移植到我们代码中的其他 generator 上。手动做这些工作是不切实际的，而且将会把 generator 的好处完全抵消掉。

但走运的是，已经存在几种工具可以自动地将 ES6 generator 转换为我们在前一节延伸出的东西。它们不仅帮我们做力气活儿，还可以处理几种我们敷衍而过的情况。

一个这样的工具是 regenerator（[`facebook.github.io/regenerator/），由 Facebook 的聪明伙计们开发的。`](https://facebook.github.io/regenerator/%EF%BC%89%EF%BC%8C%E7%94%B1Facebook%E7%9A%84%E8%81%AA%E6%98%8E%E4%BC%99%E8%AE%A1%E4%BB%AC%E5%BC%80%E5%8F%91%E7%9A%84%E3%80%82)

如果我们用 regenerator 来转译我们前面的 generator，这就是产生的代码（在编写本文时）：

```js
// `request(..)` 是一个支持 Promise 的 Ajax 工具

var foo = regeneratorRuntime.mark(function foo(url) {
    var val;

    return regeneratorRuntime.wrap(function foo$(context$1$0) {
        while (1) switch (context$1$0.prev = context$1$0.next) {
        case 0:
            context$1$0.prev = 0;
            console.log( "requesting:", url );
            context$1$0.next = 4;
            return request( url );
        case 4:
            val = context$1$0.sent;
            console.log( val );
            context$1$0.next = 12;
            break;
        case 8:
            context$1$0.prev = 8;
            context$1$0.t0 = context$1$0.catch(0);
            console.log("Oops:", context$1$0.t0);
            return context$1$0.abrupt("return", false);
        case 12:
        case "end":
            return context$1$0.stop();
        }
    }, foo, this, [[0, 8]]);
});
```

这和我们的手动推导有明显的相似性，比如`switch`/`case`语句，而且我们甚至可以看到，`val`被拉到了闭包外面，正如我们做的那样。

当然，一个代价是这个 generator 的转译需要一个帮助工具库`regeneratorRuntime`，它持有全部管理一个普通 generator/*迭代器* 所需的可复用逻辑。它的许多模板代码看起来和我们的版本不同，但即便如此，概念还是可以看到的，比如使用`context$1$0.next = 4`追踪 generator 的下一个状态。

主要的结论是，generator 不仅限于 ES6+的环境中才有用。一旦你理解了它的概念，你可以在你的所有代码中利用他们，并使用工具将代码变形为旧环境兼容的。

这比使用`Promise`API 的填补来实现前 ES6 的 Promise 要做更多的工作，但是努力完全是值得的，因为对于以一种可推理的，合理的，看似同步的顺序风格来表达异步流程控制来说，generator 实在是好太多了。

一旦你适应了 generator，你将永远不会回到面条般的回调地狱了！

## 复习

generator 是一种 ES6 的新函数类型，它不像普通函数那样运行至完成。相反，generator 可以暂停在一种中间完成状态（完整地保留它的状态），而且它可以从暂停的地方重新开始。

这种暂停/继续的互换是一种协作而非抢占，这意味着 generator 拥有的唯一能力是使用`yield`关键字暂停它自己，而且控制这个 generator 的 *迭代器* 拥有的唯一能力是继续这个 generator（通过`next(..)`）。

`yield`/`next(..)`的对偶不仅是一种控制机制，它实际上是一种双向消息传递机制。一个`yield ..`表达式实质上为了等待一个值而暂停，而下一个`next(..)`调用将把值（或隐含的`undefined`）传递回这个暂停的`yield`表达式。

与异步流程控制关联的 generator 的主要好处是，在一个 generator 内部的代码以一种自然的同步/顺序风格表达一个任务的各个步骤的序列。这其中的技巧是我们实质上将潜在的异步处理隐藏在`yield`关键字的后面——将异步处理移动到控制 generator 的 *迭代器* 代码中。

换句话说，generator 为异步代码保留了顺序的，同步的，阻塞的代码模式，这允许我们的大脑更自然地推理代码，解决了基于回调的异步产生的两个关键问题中的一个。