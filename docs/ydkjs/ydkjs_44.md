# 你不懂 JS：ES6 与未来 第三章：组织（上）

编写 JS 代码是一回事儿，而合理地组织它是另一回事儿。利用常见的组织和重用模式在很大程度上改善了你代码的可读性和可理解性。记住：代码在与其他开发者交流上起的作用，与在给计算机喂指令上起的作用同样重要。

ES6 拥有几种重要的特性可以显著改善这些模式，包括：迭代器，generator，模块，和类。

## 迭代器

*迭代器（iterator）* 是一种结构化的模式，用于从一个信息源中以一次一个的方式抽取信息。这种模式在程序设计中存在很久了。而且不可否认的是，不知从什么时候起 JS 开发者们就已经特别地设计并实现了迭代器，所以它根本不是什么新的话题。

ES6 所做的是，为迭代器引入了一个隐含的标准化接口。许多在 JavaScript 中内建的数据结构现在都会暴露一个实现了这个标准的迭代器。而且你也可以构建自己的遵循同样标准的迭代器，来使互用性最大化。

迭代器是一种消费数据的方法，它是组织有顺序的，相继的，基于抽取的。

举个例子，你可能实现一个工具，它在每次被请求时产生一个新的唯一的标识符。或者你可能循环一个固定的列表以轮流的方式产生一系列无限多的值。或者你可以在一个数据库查询的结果上添加一个迭代器来一次抽取一行结果。

虽然在 JS 中它们不经常以这样的方式被使用，但是迭代器还可以认为是每次控制行为中的一个步骤。这会在考虑 generator 时得到相当清楚的展示（参见本章稍后的“Generator”），虽然你当然可以不使用 generator 而做同样的事。

### 接口

在本书写作的时候，ES6 的 25.1.1.2 部分 ([`people.mozilla.org/~jorendorff/es6-draft.html#sec-iterator-interface`](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-iterator-interface)) 详述了`Iterator`接口，它有如下的要求：

```js
Iterator [必须]
    next() {method}: 取得下一个 IteratorResult
```

有两个可选成员，有些迭代器用它们进行了扩展：

```js
Iterator [可选]
    return() {method}: 停止迭代并返回 IteratorResult
    throw() {method}: 通知错误并返回 IteratorResult
```

接口`IteratorResult`被规定为：

```js
IteratorResult
    value {property}: 当前的迭代值或最终的返回值
        （如果它的值为`undefined`，是可选的）
    done {property}: 布尔值，指示完成的状态
```

**注意：** 我称这些接口是隐含的，不是因为它们没有在语言规范中被明确地被说出来 —— 它们被说出来了！—— 而是因为它们没有作为可以直接访问的对象暴露给代码。在 ES6 中，JavaScript 不支持任何“接口”的概念，所以在你自己的代码中遵循它们纯粹是惯例上的。但是，不论 JS 在何处需要一个迭代器 —— 例如在一个`for..of`循环中 —— 你提供的东西必须遵循这些接口，否则代码就会失败。

还有一个`Iterable`接口，它描述了一定能够产生迭代器的对象：

```js
Iterable
    @@iterator() {method}: 产生一个迭代器
```

如果你回忆一下第二章的“内建 Symbol”，`@@iterator`是一种特殊的内建 symbol，表示可以为对象产生迭代器的方法。

#### IteratorResult

`IteratorResult`接口规定从任何迭代器操作的返回值都是这样形式的对象：

```js
{ value: .. , done: true / false }
```

内建迭代器将总是返回这种形式的值，当然，更多的属性也允许出现在这个返回值中，如果有必要的话。

例如，一个自定义的迭代器可能会在结果对象中加入额外的元数据（比如，数据是从哪里来的，取得它花了多久，缓存过期的时间长度，下次请求的恰当频率，等等）。

**注意：** 从技术上讲，在值为`undefined`的情况下，`value`是可选的，它将会被认为是不存在或者是没有被设置。因为不管它是表示的就是这个值还是完全不存在，访问`res.value`都将会产生`undefined`，所以这个属性的存在/不存在更大程度上是一个实现或者优化（或两者）的细节，而非一个功能上的问题。

### `next()`迭代

让我们来看一个数组，它是一个可迭代对象，可以生成一个迭代器来消费它的值：

```js
var arr = [1,2,3];

var it = arr[Symbol.iterator]();

it.next();        // { value: 1, done: false }
it.next();        // { value: 2, done: false }
it.next();        // { value: 3, done: false }

it.next();        // { value: undefined, done: true }
```

每一次定位在`Symbol.iterator`上的方法在值`arr`上被调用时，它都将生成一个全新的迭代器。大多数的数据结构都会这么做，包括所有内建在 JS 中的数据结构。

然而，像事件队列这样的结构也许只能生成一个单独的迭代器（单例模式）。或者某种结构可能在同一时间内只允许存在一个唯一的迭代器，要求当前的迭代器必须完成，才能创建一个新的。

前一个代码段中的`it`迭代器不会再你得到值`3`时报告`done: true`。你必须再次调用`next()`，实质上越过数组末尾的值，才能得到完成信号`done: true`。在这一节稍后会清楚地讲解这种设计方式的原因，但是它通常被认为是一种最佳实践。

基本类型的字符串值也默认地是可迭代对象：

```js
var greeting = "hello world";

var it = greeting[Symbol.iterator]();

it.next();        // { value: "h", done: false }
it.next();        // { value: "e", done: false }
..
```

**注意：** 从技术上讲，这个基本类型值本身不是可迭代对象，但多亏了“封箱”，`"hello world"`被强制转换为它的`String`对象包装形式，*它* 才是一个可迭代对象。更多信息参见本系列的 *类型与文法*。

ES6 还包括几种新的数据结构，称为集合（参见第五章）。这些集合不仅本身就是可迭代对象，而且它们还提供 API 方法来生成一个迭代器，例如：

```js
var m = new Map();
m.set( "foo", 42 );
m.set( { cool: true }, "hello world" );

var it1 = m[Symbol.iterator]();
var it2 = m.entries();

it1.next();        // { value: [ "foo", 42 ], done: false }
it2.next();        // { value: [ "foo", 42 ], done: false }
..
```

一个迭代器的`next(..)`方法能够可选地接受一个或多个参数。大多数内建的迭代器不会实施这种能力，虽然一个 generator 的迭代器绝对会这么做（参见本章稍后的“Generator”）。

根据一般的惯例，包括所有的内建迭代器，在一个已经被耗尽的迭代器上调用`next(..)`不是一个错误，而是简单地持续返回结果`{ value: undefined, done: true }`。

### 可选的`return(..)`和`throw(..)`

在迭代器接口上的可选方法 —— `return(..)`和`throw(..)` —— 在大多数内建的迭代器上都没有被实现。但是，它们在 generator 的上下文环境中绝对有某些含义，所以更具体的信息可以参看“Generator”。

`return(..)`被定义为向一个迭代器发送一个信号，告知它消费者代码已经完成而且不会再从它那里抽取更多的值。这个信号可以用于通知生产者（应答`next(..)`调用的迭代器）去实施一些可能的清理作业，比如释放/关闭网络，数据库，或者文件引用资源。

如果一个迭代器拥有`return(..)`，而且发生了可以自动被解释为非正常或者提前终止消费迭代器的任何情况，`return(..)`就将会被自动调用。你也可以手动调用`return(..)`。

`return(..)`将会像`next(..)`一样返回一个`IteratorResult`对象。一般来说，你向`return(..)`发送的可选值将会在这个`IteratorResult`中作为`value`发送回来，虽然在一些微妙的情况下这可能不成立。

`throw(..)`被用于向一个迭代器发送一个异常/错误信号，与`return(..)`隐含的完成信号相比，它可能会被迭代器用于不同的目的。它不一定像`return(..)`一样暗示着迭代器的完全停止。

例如，在 generator 迭代器中，`throw(..)`实际上会将一个被抛出的异常注射到 generator 暂停的执行环境中，这个异常可以用`try..catch`捕获。一个未捕获的`throw(..)`异常将会导致 generator 的迭代器异常中止。

**注意：** 根据一般的惯例，在`return(..)`或`throw(..)`被调用之后，一个迭代器就不应该在产生任何结果了。

### 迭代器循环

正如我们在第二章的“`for..of`”一节中讲解的，ES6 的`for..of`循环可以直接消费一个规范的可迭代对象。

如果一个迭代器也是一个可迭代对象，那么它就可以直接与`for..of`循环一起使用。通过给予迭代器一个简单地返回它自身的`Symbol.iterator`方法，你就可以使它成为一个可迭代对象：

```js
var it = {
 // 使迭代器`it`成为一个可迭代对象
 [Symbol.iterator]() { return this; },

 next() { .. },
 ..
};

it[Symbol.iterator]() === it;        // true
```

现在我们就可以用一个`for..of`循环来消费迭代器`it`了：

```js
for (var v of it) {
    console.log( v );
}
```

为了完全理解这样的循环如何工作，回忆下第二章中的`for..of`循环的`for`等价物：

```js
for (var v, res; (res = it.next()) && !res.done; ) {
    v = res.value;
    console.log( v );
}
```

如果你仔细观察，你会发现`it.next()`是在每次迭代之前被调用的，然后`res.done`才被查询。如果`res.done`是`true`，那么这个表达式将会求值为`false`于是这次迭代不会发生。

回忆一下之前我们建议说，迭代器一般不应与最终预期的值一起返回`done: true`。现在你知道为什么了。

如果一个迭代器返回了`{ done: true, value: 42 }`，`for..of`循环将完全扔掉值`42`。因此，假定你的迭代器可能会被`for..of`循环或它的`for`等价物这样的模式消费的话，你可能应当等到你已经返回了所有相关的迭代值之后才返回`done: true`来表示完成。

**警告：** 当然，你可以有意地将你的迭代器设计为将某些相关的`value`与`done: true`同时返回。但除非你将此情况在文档中记录下来，否则不要这么做，因为这样会隐含地强制你的迭代器消费者使用一种，与我们刚才描述的`for..of`或它的手动等价物不同的模式来进行迭代。

### 自定义迭代器

除了标准的内建迭代器，你还可以制造你自己的迭代器！所有使它们可以与 ES6 消费设施（例如，`for..of`循环和`...`操作符）进行互动的代价就是遵循恰当的接口。

让我们试着构建一个迭代器，它能够以斐波那契（Fibonacci）数列的形式产生无限多的数字序列：

```js
var Fib = {
    [Symbol.iterator]() {
        var n1 = 1, n2 = 1;

        return {
            // 使迭代器成为一个可迭代对象
            [Symbol.iterator]() { return this; },

            next() {
                var current = n2;
                n2 = n1;
                n1 = n1 + current;
                return { value: current, done: false };
            },

            return(v) {
                console.log(
                    "Fibonacci sequence abandoned."
                );
                return { value: v, done: true };
            }
        };
    }
};

for (var v of Fib) {
    console.log( v );

    if (v > 50) break;
}
// 1 1 2 3 5 8 13 21 34 55
// Fibonacci sequence abandoned.
```

**警告：** 如果我们没有插入`break`条件，这个`for..of`循环将会永远运行下去，这回破坏你的程序，因此可能不是我们想要的！

方法`Fib[Symbol.iterator]()`在被调用时返回带有`next()`和`return(..)`方法的迭代器对象。它的状态通过变量`n1`和`n2`维护在闭包中。

接下来让我们考虑一个迭代器，它被设计为执行一系列（也叫队列）动作，一次一个：

```js
var tasks = {
    [Symbol.iterator]() {
        var steps = this.actions.slice();

        return {
            // 使迭代器成为一个可迭代对象
            [Symbol.iterator]() { return this; },

            next(...args) {
                if (steps.length > 0) {
                    let res = steps.shift()( ...args );
                    return { value: res, done: false };
                }
                else {
                    return { done: true }
                }
            },

            return(v) {
                steps.length = 0;
                return { value: v, done: true };
            }
        };
    },
    actions: []
};
```

在`tasks`上的迭代器步过在数组属性`actions`中找到的函数，并每次执行它们中的一个，并传入你传递给`next(..)`的任何参数值，并在标准的`IteratorResult`对象中向你返回任何它返回的东西。

这是我们如何使用这个`tasks`队列：

```js
tasks.actions.push(
    function step1(x){
        console.log( "step 1:", x );
        return x * 2;
    },
    function step2(x,y){
        console.log( "step 2:", x, y );
        return x + (y * 2);
    },
    function step3(x,y,z){
        console.log( "step 3:", x, y, z );
        return (x * y) + z;
    }
);

var it = tasks[Symbol.iterator]();

it.next( 10 );            // step 1: 10
                        // { value:   20, done: false }

it.next( 20, 50 );        // step 2: 20 50
                        // { value:  120, done: false }

it.next( 20, 50, 120 );    // step 3: 20 50 120
                        // { value: 1120, done: false }

it.next();                // { done: true }
```

这种特别的用法证实了迭代器可以是一种具有组织功能的模式，不仅仅是数据。这也联系着我们在下一节关于 generator 将要看到的东西。

你甚至可以更有创意一些，在一块数据上定义一个表示元操作的迭代器。例如，我们可以为默认从 0 开始递增至（或递减至，对于负数来说）指定数字的一组数字定义一个迭代器。

考虑如下代码：

```js
if (!Number.prototype[Symbol.iterator]) {
    Object.defineProperty(
        Number.prototype,
        Symbol.iterator,
        {
            writable: true,
            configurable: true,
            enumerable: false,
            value: function iterator(){
                var i, inc, done = false, top = +this;

                // 正向迭代还是负向迭代？
                inc = 1 * (top < 0 ? -1 : 1);

                return {
                    // 使迭代器本身成为一个可迭代对象！
                    [Symbol.iterator](){ return this; },

                    next() {
                        if (!done) {
                            // 最初的迭代总是 0
                            if (i == null) {
                                i = 0;
                            }
                            // 正向迭代
                            else if (top >= 0) {
                                i = Math.min(top,i + inc);
                            }
                            // 负向迭代
                            else {
                                i = Math.max(top,i + inc);
                            }

                            // 这次迭代之后就完了？
                            if (i == top) done = true;

                            return { value: i, done: false };
                        }
                        else {
                            return { done: true };
                        }
                    }
                };
            }
        }
    );
}
```

现在，这种创意给了我们什么技巧？

```js
for (var i of 3) {
    console.log( i );
}
// 0 1 2 3

[...-3];                // [0,-1,-2,-3]
```

这是一些有趣的技巧，虽然其实际用途有些值得商榷。但是再一次，有人可能想知道为什么 ES6 没有提供如此微小但讨喜的特性呢？

如果我连这样的提醒都没给过你，那就是我的疏忽：像我在前面的代码段中做的那样扩展原生原型，是一件你需要小心并了解潜在的危害后才应该做的事情。

在这样的情况下，你与其他代码或者未来的 JS 特性发生冲突的可能性非常低。但是要小心微小的可能性。并在文档中为后人详细记录下你在做什么。

**注意：** 如果你想知道更多细节，我在这篇文章([`blog.getify.com/iterating-es6-numbers/`](http://blog.getify.com/iterating-es6-numbers/)) 中详细论述了这种特别的技术。而且这段评论([`blog.getify.com/iterating-es6-numbers/comment-page-1/#comment-535294)甚至为制造一个字符串字符范围提出了一个相似的技巧。`](http://blog.getify.com/iterating-es6-numbers/comment-page-1/#comment-535294)%E7%94%9A%E8%87%B3%E4%B8%BA%E5%88%B6%E9%80%A0%E4%B8%80%E4%B8%AA%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%AD%97%E7%AC%A6%E8%8C%83%E5%9B%B4%E6%8F%90%E5%87%BA%E4%BA%86%E4%B8%80%E4%B8%AA%E7%9B%B8%E4%BC%BC%E7%9A%84%E6%8A%80%E5%B7%A7%E3%80%82)

### 消费迭代器

我们已经看到了使用`for..of`循环来一个元素一个元素地消费一个迭代器。但是还有一些其他的 ES6 结构可以消费迭代器。

让我们考虑一下附着这个数组上的迭代器（虽然任何我们选择的迭代器都将拥有如下的行为）：

```js
var a = [1,2,3,4,5];
```

扩散操作符`...`将完全耗尽一个迭代器。考虑如下代码：

```js
function foo(x,y,z,w,p) {
    console.log( x + y + z + w + p );
}

foo( ...a );            // 15
```

`...`还可以在一个数组内部扩散一个迭代器：

```js
var b = [ 0, ...a, 6 ];
b;                        // [0,1,2,3,4,5,6]
```

数组解构（参见第二章的“解构”）可以部分地或者完全地（如果与一个`...`剩余/收集操作符一起使用）消费一个迭代器：

```js
var it = a[Symbol.iterator]();

var [x,y] = it;            // 仅从`it`中取前两个元素
var [z, ...w] = it;        // 取第三个，然后一次取得剩下所有的

// `it`被完全耗尽了吗？是的
it.next();                // { value: undefined, done: true }

x;                        // 1
y;                        // 2
z;                        // 3
w;                        // [4,5]
```

## Generator

所有的函数都会运行至完成，对吧？换句话说，一旦一个函数开始运行，在它完成之前没有任何东西能够打断它。

至少对于到目前为止的 JavaScript 的整个历史来说是这样的。在 ES6 中，引入了一个有些异乎寻常的新形式的函数，称为 generator。一个 generator 可以在运行期间暂停它自己，还可以立即或者稍后继续运行。所以显然它没有普通函数那样的运行至完成的保证。

另外，在运行期间的每次暂停/继续轮回都是一个双向消息传递的好机会，generator 可以在这里返回一个值，而使它继续的控制端代码可以发回一个值。

就像前一节中的迭代器一样，有种方式可以考虑 generator 是什么，或者说它对什么最有用。对此没有一个正确的答案，但我们将试着从几个角度考虑。

**注意：** 关于 generator 的更多信息参见本系列的 *异步与性能*，还可以参见本书的第四章。

### 语法

generator 函数使用这种新语法声明：

```js
function *foo() {
    // ..
}
```

`*`的位置在功能上无关紧要。同样的声明还可以写做以下的任意一种：

```js
function *foo() { .. }
function* foo() { .. }
function * foo() { .. }
function*foo() { .. }
..
```

这里 *唯一* 的区别就是风格的偏好。大多数其他的文献似乎喜欢`function* foo(..) { .. }`。我喜欢`function *foo(..) { .. }`，所以这就是我将在本书剩余部分中表示它们的方法。

我这样做的理由实质上纯粹是为了教学。在这本书中，当我引用一个 generator 函数时，我将使用`*foo(..)`，与普通函数的`foo(..)`相对。我发现`*foo(..)`与`function *foo(..) { .. }`中`*`的位置更加吻合。

另外，就像我们在第二章的简约方法中看到的，在对象字面量中有一种简约 generator 形式：

```js
var a = {
    *foo() { .. }
};
```

我要说在简约 generator 中，`*foo() { .. }`要比`* foo() { .. }`更自然。这进一步表明了为何使用`*foo()`匹配一致性。

一致性使理解与学习更轻松。

#### 执行一个 Generator

虽然一个 generator 使用`*`进行声明，但是你依然可以像一个普通函数那样执行它：

```js
foo();
```

你依然可以传给它参数值，就像：

```js
function *foo(x,y) {
    // ..
}

foo( 5, 10 );
```

主要区别在于，执行一个 generator，比如`foo(5,10)`，并不实际运行 generator 中的代码。取而代之的是，它生成一个迭代器来控制 generator 执行它的代码。

我们将在稍后的“迭代器控制”中回到这个话题，但是简要地说：

```js
function *foo() {
    // ..
}

var it = foo();

// 要开始/推进`*foo()`，调用
// `it.next(..)`
```

#### `yield`

Generator 还有一个你可以在它们内部使用的新关键字，用来表示暂停点：`yield`。考虑如下代码：

```js
function *foo() {
    var x = 10;
    var y = 20;

    yield;

    var z = x + y;
}
```

在这个`*foo()`generator 中，前两行的操作将会在开始时运行，然后`yield`将会暂停这个 generator。如果这个 generator 被继续，`*foo()`的最后一行将运行。在一个 generator 中`yield`可以出现任意多次（或者，在技术上讲，根本不出现！）。

你甚至可以在一个循环内部放置`yield`，它可以表示一个重复的暂停点。事实上，一个永不完成的循环就意味着一个永不完成的 generator，这是完全合法的，而且有时候完全是你需要的。

`yield`不只是一个暂停点。它是在暂停 generator 时发送出一个值的表达式。这里是一个位于 generator 中的`while..true`循环，它每次迭代时`yield`出一个新的随机数：

```js
function *foo() {
    while (true) {
        yield Math.random();
    }
}
```

`yield ..`表达式不仅发送一个值 —— 不带值的`yield`与`yield undefined`相同 —— 它还接收（也就是，被替换为）最终的继续值。考虑如下代码：

```js
function *foo() {
    var x = yield 10;
    console.log( x );
}
```

这个 generator 在暂停它自己时将首先`yield`出值`10`。当你继续这个 generator 时 —— 使用我们先前提到的`it.next(..)` —— 无论你使用什么值继续它，这个值都将替换/完成整个表达式`yield 10`，这意味着这个值将被赋值给变量`x`

一个`yield..`表达式可以出现在任意普通表达式可能出现的地方。例如：

```js
function *foo() {
    var arr = [ yield 1, yield 2, yield 3 ];
    console.log( arr, yield 4 );
}
```

这里的`*foo()`有四个`yield ..`表达式。其中每个`yield`都会导致 generator 暂停以等待一个继续值，这个继续值稍后被用于各个表达式环境中。

`yield`在技术上讲不是一个操作符，虽然像`yield 1`这样使用时看起来确实很像。因为`yield`可以像`var x = yield`这样完全通过自己被使用，所以将它认为是一个操作符有时令人困惑。

从技术上讲，`yield ..`与`a = 3`这样的赋值表达式拥有相同的“表达式优先级” —— 概念上和操作符优先级很相似。这意味着`yield ..`基本上可以出现在任何`a = 3`可以合法出现的地方。

让我们展示一下这种对称性：

```js
var a, b;

a = 3;                    // 合法
b = 2 + a = 3;            // 不合法
b = 2 + (a = 3);        // 合法

yield 3;                // 合法
a = 2 + yield 3;        // 不合法
a = 2 + (yield 3);        // 合法
```

**注意：** 如果你好好考虑一下，认为一个`yield ..`表达式与一个赋值表达式的行为相似在概念上有些道理。当一个被暂停的 generator 被继续时，它就以一种与被这个继续值“赋值”区别不大的方式，被这个值完成/替换。

要点：如果你需要`yield ..`出现在`a = 3`这样的赋值本不被允许出现的位置，那么它就需要被包在一个`( )`中。

因为`yield`关键字的优先级很低，几乎任何出现在`yield ..`之后的表达式都会在被`yield`发送之前首先被计算。只有扩散操作符`...`和逗号操作符`,`拥有更低的优先级，这意味着他们会在`yield`已经被求值之后才会被处理。

所以正如带有多个操作符的普通语句一样，存在另一个可能需要`( )`来覆盖（提升）`yield`的低优先级的情况，就像这些表达式之间的区别：

```js
yield 2 + 3;            // 与`yield (2 + 3)`相同

(yield 2) + 3;            // 首先`yield 2`，然后`+ 3`
```

和`=`赋值一样，`yield`也是“右结合性”的，这意味着多个接连出现的`yield`表达式被视为从右到左被`( .. )`分组。所以，`yield yield yield 3`将被视为`yield (yield (yield 3))`。像`((yield) yield) yield 3`这样的“左结合性”解释没有意义。

和其他操作符一样，`yield`与其他操作符或`yield`组合时为了使你的意图没有歧义，使用`( .. )`分组是一个好主意，即使这不是严格要求的。

**注意：** 更多关于操作符优先级和结合性的信息，参见本系列的 *类型与文法*。

#### `yield *`

与`*`使一个`function`声明成为一个`function *`generator 声明的方式一样，一个`*`使`yield`成为一个机制非常不同的`yield *`，称为 *yield 委托*。从文法上讲，`yield *..`的行为与`yield ..`相同，就像在前一节讨论过的那样。

`yield * ..`需要一个可迭代对象；然后它调用这个可迭代对象的迭代器，并将它自己的宿主 generator 的控制权委托给那个迭代器，直到它被耗尽。考虑如下代码：

```js
function *foo() {
    yield *[1,2,3];
}
```

**注意：** 与 generator 声明中`*`的位置（早先讨论过）一样，在`yield *`表达式中的`*`的位置在风格上由你来决定。大多数其他文献偏好`yield* ..`，但是我喜欢`yield *..`，理由和我们已经讨论过的相同。

值`[1,2,3]`产生一个将会步过它的值的迭代器，所以 generator`*foo()`将会在被消费时产生这些值。另一种说明这种行为的方式是，yield 委托到了另一个 generator：

```js
function *foo() {
    yield 1;
    yield 2;
    yield 3;
}

function *bar() {
    yield *foo();
}
```

当`*bar()`调用`*foo()`产生的迭代器通过`yield *`受到委托，意味着无论`*foo()`产生什么值都会被`*bar()`产生。

在`yield ..`中表达式的完成值来自于使用`it.next(..)`继续 generator，而`yield *..`表达式的完成值来自于受到委托的迭代器的返回值（如果有的话）。

内建的迭代器一般没有返回值，正如我们在本章早先的“迭代器循环”一节的末尾讲过的。但是如果你定义你自己的迭代器（或者 generator），你就可以将它设计为`return`一个值，`yield *..`将会捕获它：

```js
function *foo() {
    yield 1;
    yield 2;
    yield 3;
    return 4;
}

function *bar() {
    var x = yield *foo();
    console.log( "x:", x );
}

for (var v of bar()) {
    console.log( v );
}
// 1 2 3
// x: { value: 4, done: true }
```

虽然值`1`，`2`，和`3`从`*foo()`中被`yield`出来，然后从`*bar()`中被`yield`出来，但是从`*foo()`中返回的值`4`是表达式`yield *foo()`的完成值，然后它被赋值给`x`。

因为`yield *`可以调用另一个 generator（通过委托到它的迭代器的方式），它还可以通过调用自己来实施某种 generator 递归：

```js
function *foo(x) {
    if (x < 3) {
        x = yield *foo( x + 1 );
    }
    return x * 2;
}

foo( 1 );
```

取得`foo(1)`的结果并调用迭代器的`next()`来使它运行它的递归步骤，结果将是`24`。第一次`*foo()`运行时`x`拥有值`1`，它是`x < 3`。`x + 1`被递归地传递到`*foo(..)`，所以之后的`x`是`2`。再一次递归调用导致`x`为`3`。

现在，因为`x < 3`失败了，递归停止，而且`return 3 * 2`将`6`给回前一个调用的`yeild *..`表达式，它被赋值给`x`。另一个`return 6 * 2`返回`12`给前一个调用的`x`。最终`12 * 2`，即`24`，从 generator`*foo(..)`运行的完成中被返回。

### 迭代器控制

早先，我们简要地介绍了 generator 是由迭代器控制的概念。现在让我们完整地深入这个话题。

回忆一下前一节的递归`*for(..)`。这是我们如何运行它：

```js
function *foo(x) {
    if (x < 3) {
        x = yield *foo( x + 1 );
    }
    return x * 2;
}

var it = foo( 1 );
it.next();                // { value: 24, done: true }
```

在这种情况下，generator 并没有真正暂停过，因为这里没有`yield ..`表达式。而`yield *`只是通过递归调用保持当前的迭代步骤继续运行下去。所以，仅仅对迭代器的`next()`函数进行一次调用就完全地运行了 generator。

现在让我们考虑一个有多个步骤并且因此有多个产生值的 generator：

```js
function *foo() {
    yield 1;
    yield 2;
    yield 3;
}
```

我们已经知道我们可以是使用一个`for..of`循环来消费一个迭代器，即便它是一个附着在`*foo()`这样的 generator 上：

```js
for (var v of foo()) {
    console.log( v );
}
// 1 2 3
```

**注意：** `for..of`循环需要一个可迭代对象。一个 generator 函数引用（比如`foo`）本身不是一个可迭代对象；你必须使用`foo()`来执行它以得到迭代器（它也是一个可迭代对象，正如我们在本章早先讲解过的）。理论上你可以使用一个实质上仅仅执行`return this()`的`Symbol.iterator`函数来扩展`GeneratorPrototype`（所有 generator 函数的原型）。这将使`foo`引用本身成为一个可迭代对象，也就意味着`for (var v of foo) { .. }`（注意在`foo`上没有`()`）将可以工作。

让我们手动迭代这个 generator：

```js
function *foo() {
    yield 1;
    yield 2;
    yield 3;
}

var it = foo();

it.next();                // { value: 1, done: false }
it.next();                // { value: 2, done: false }
it.next();                // { value: 3, done: false }

it.next();                // { value: undefined, done: true }
```

如果你仔细观察，这里有三个`yield`语句和四个`next()`调用。这可能看起来像是一个奇怪的不匹配。事实上，假定所有的东西都被求值并且 generator 完全运行至完成的话，`next()`调用将总是比`yield`表达式多一个。

但是如果你相反的角度观察（从里向外而不是从外向里），`yield`和`next()`之间的匹配就显得更有道理。

回忆一下，`yield ..`表达式将被你用于继续 generator 的值完成。这意味着你传递给`next(..)`的参数值将完成任何当前暂停中等待完成的`yield ..`表达式。

让我们这样展示一下这种视角：

```js
function *foo() {
    var x = yield 1;
    var y = yield 2;
    var z = yield 3;
    console.log( x, y, z );
}
```

在这个代码段中，每个`yield ..`都送出一个值（`1`，`2`，`3`），但更直接的是，它暂停了 generator 来等待一个值。换句话说，它就像在问这样一个问题，“我应当在这里用什么值？我会在这里等你告诉我。”

现在，这是我们如何控制`*foo()`来启动它：

```js
var it = foo();

it.next();                // { value: 1, done: false }
```

这第一个`next()`调用从 generator 初始的暂停状态启动了它，并运行至第一个`yield`。在你调用第一个`next()`的那一刻，并没有`yield ..`表达式等待完成。如果你给第一个`next()`调用传递一个值，目前它会被扔掉，因为没有`yield`等着接受这样的一个值。

**注意：** 一个“ES6 之后”时间表中的早期提案 *将* 允许你在 generator 内部通过一个分离的元属性（见第七章）来访问一个被传入初始`next(..)`调用的值。

现在，让我们回答那个未解的问题，“我应当给`x`赋什么值？” 我们将通过给 *下一个* `next(..)`调用发送一个值来回答：

```js
it.next( "foo" );        // { value: 2, done: false }
```

现在，`x`将拥有值`"foo"`，但我们也问了一个新的问题，“我应当给`y`赋什么值？”

```js
it.next( "bar" );        // { value: 3, done: false }
```

答案给出了，另一个问题被提出了。最终答案：

```js
it.next( "baz" );        // "foo" "bar" "baz"
                        // { value: undefined, done: true }
```

现在，每一个`yield ..`的“问题”是如何被 *下一个* `next(..)`调用回答的，所以我们观察到的那个“额外的”`next()`调用总是使一切开始的那一个。

让我们把这些步骤放在一起：

```js
var it = foo();

// 启动 generator
it.next();                // { value: 1, done: false }

// 回答第一个问题
it.next( "foo" );        // { value: 2, done: false }

// 回答第二个问题
it.next( "bar" );        // { value: 3, done: false }

// 回答第三个问题
it.next( "baz" );        // "foo" "bar" "baz"
                        // { value: undefined, done: true }
```

在生成器的每次迭代都简单地为消费者生成一个值的情况下，你可认为一个 generator 是一个值的生成器。

但是在更一般的意义上，也许将 generator 认为是一个受控制的，累进的代码执行过程更恰当，与早先“自定义迭代器”一节中的`tasks`队列的例子非常相像。

**注意：** 这种视角正是我们将如何在第四章中重温 generator 的动力。特别是，`next(..)`没有理由一定要在前一个`next(..)`完成之后立即被调用。虽然 generator 的内部执行环境被暂停了，程序的其他部分仍然没有被阻塞，这包括控制 generator 什么时候被继续的异步动作能力。

### 提前完成

正如我们在本章早先讲过的，连接到一个 generator 的迭代器支持可选的`return(..)`和`throw(..)`方法。它们俩都有立即中止一个暂停的的 generator 的效果。

考虑如下代码：

```js
function *foo() {
    yield 1;
    yield 2;
    yield 3;
}

var it = foo();

it.next();                // { value: 1, done: false }

it.return( 42 );        // { value: 42, done: true }

it.next();                // { value: undefined, done: true }
```

`return(x)`有点像强制一个`return x`就在那个时刻被处理，这样你就立即得到这个指定的值。一旦一个 generator 完成，无论是正常地还是像展示的那样提前地，它就不再处理任何代码或返回任何值了。

`return(..)`除了可以手动调用，它还在迭代的最后被任何 ES6 中消费迭代器的结构自动调用，比如`for..of`循环和`...`扩散操作符。

这种能力的目的是，在控制端的代码不再继续迭代 generator 时它可以收到通知，这样它就可能做一些清理工作（释放资源，复位状态，等等）。与普通函数的清理模式完全相同，达成这个目的的主要方法是使用一个`finally`子句：

```js
function *foo() {
    try {
        yield 1;
        yield 2;
        yield 3;
    }
    finally {
        console.log( "cleanup!" );
    }
}

for (var v of foo()) {
    console.log( v );
}
// 1 2 3
// cleanup!

var it = foo();

it.next();                // { value: 1, done: false }
it.return( 42 );        // cleanup!
                        // { value: 42, done: true }
```

**警告：** 不要把`yield`语句放在`finally`子句内部！它是有效和合法的，但这确实是一个可怕的主意。它在某种意义上推迟了`return(..)`调用的完成，因为在`finally`子句中的任何`yield ..`表达式都被遵循来暂停和发送消息；你不会像期望的那样立即得到一个完成的 generator。基本上没有任何好的理由去选择这种疯狂的 *坏的部分*，所以避免这么做！

前一个代码段除了展示`return(..)`如何在中止 generator 的同时触发`finally`子句，它还展示了一个 generator 在每次被调用时都产生一个全新的迭代器。事实上，你可以并发地使用连接到相同 generator 的多个迭代器：

```js
function *foo() {
    yield 1;
    yield 2;
    yield 3;
}

var it1 = foo();
it1.next();                // { value: 1, done: false }
it1.next();                // { value: 2, done: false }

var it2 = foo();
it2.next();                // { value: 1, done: false }

it1.next();                // { value: 3, done: false }

it2.next();                // { value: 2, done: false }
it2.next();                // { value: 3, done: false }

it2.next();                // { value: undefined, done: true }
it1.next();                // { value: undefined, done: true }
```

#### 提前中止

你可以调用`throw(..)`来代替`return(..)`调用。就像`return(x)`实质上在 generator 当前的暂停点上注入了一个`return x`一样，调用`throw(x)`实质上就像在暂停点上注入了一个`throw x`。

除了处理异常的行为（我们在下一节讲解这对`try`子句意味着什么），`throw(..)`产生相同的提前完成 —— 在 generator 当前的暂停点中止它的运行。例如：

```js
function *foo() {
    yield 1;
    yield 2;
    yield 3;
}

var it = foo();

it.next();                // { value: 1, done: false }

try {
    it.throw( "Oops!" );
}
catch (err) {
    console.log( err );    // Exception: Oops!
}

it.next();                // { value: undefined, done: true }
```

因为`throw(..)`基本上注入了一个`throw ..`来替换 generator 的`yield 1`这一行，而且没有东西处理这个异常，它立即传播回外面的调用端代码，调用端代码使用了一个`try..catch`来处理了它。

与`return(..)`不同的是，迭代器的`throw(..)`方法绝不会被自动调用。

当然，虽然没有在前面的代码段中展示，但如果当你调用`throw(..)`时有一个`try..finally`子句等在 generator 内部的话，这个`finally`子句将会在异常被传播回调用端代码之前有机会运行。

### 错误处理

正如我们已经得到的提示，generator 中的错误处理可以使用`try..catch`表达，它在上行和下行两个方向都可以工作。

```js
function *foo() {
    try {
        yield 1;
    }
    catch (err) {
        console.log( err );
    }

    yield 2;

    throw "Hello!";
}

var it = foo();

it.next();                // { value: 1, done: false }

try {
    it.throw( "Hi!" );    // Hi!
                        // { value: 2, done: false }
    it.next();

    console.log( "never gets here" );
}
catch (err) {
    console.log( err );    // Hello!
}
```

错误也可以通过`yield *`委托在两个方向上传播：

```js
function *foo() {
    try {
        yield 1;
    }
    catch (err) {
        console.log( err );
    }

    yield 2;

    throw "foo: e2";
}

function *bar() {
    try {
        yield *foo();

        console.log( "never gets here" );
    }
    catch (err) {
        console.log( err );
    }
}

var it = bar();

try {
    it.next();            // { value: 1, done: false }

    it.throw( "e1" );    // e1
                        // { value: 2, done: false }

    it.next();            // foo: e2
                        // { value: undefined, done: true }
}
catch (err) {
    console.log( "never gets here" );
}

it.next();                // { value: undefined, done: true }
```

当`*foo()`调用`yield 1`时，值`1`原封不动地穿过了`*bar()`，就像我们已经看到过的那样。

但这个代码段最有趣的部分是，当`*foo()`调用`throw "foo: e2"`时，这个错误传播到了`*bar()`并立即被`*bar()`的`try..catch`块儿捕获。错误没有像值`1`那样穿过`*bar()`。

然后`*bar()`的`catch`将`err`普通地输出（`"foo: e2"`）之后`*bar()`就正常结束了，这就是为什么迭代器结果`{ value: undefined, done: true }`从`it.next()`中返回。

如果`*bar()`没有用`try..catch`环绕着`yield *..`表达式，那么错误将理所当然地一直传播出来，而且在它传播的路径上依然会完成（中止）`*bar()`。

### 转译一个 Generator

有可能在 ES6 之前的环境中表达 generator 的能力吗？事实上是可以的，而且有好几种了不起的工具在这么做，包括最著名的 Facebook 的 Regenerator 工具 ([`facebook.github.io/regenerator/)。`](https://facebook.github.io/regenerator/)%E3%80%82)

但为了更好地理解 generator，让我们试着手动转换一下。基本上讲，我们将制造一个简单的基于闭包的状态机。

我们将使原本的 generator 非常简单：

```js
function *foo() {
    var x = yield 42;
    console.log( x );
}
```

开始之前，我们将需要一个我们能够执行的称为`foo()`的函数，它需要返回一个迭代器：

```js
function foo() {
    // ..

    return {
        next: function(v) {
            // ..
        }

        // 我们将省略`return(..)`和`throw(..)`
    };
}
```

现在，我们需要一些内部变量来持续跟踪我们的“generator”的逻辑走到了哪一个步骤。我们称它为`state`。我们将有三种状态：起始状态的`0`，等待完成`yield`表达式的`1`，和 generator 完成的`2`。

每次`next(..)`被调用时，我们需要处理下一个步骤，然后递增`state`。为了方便，我们将每个步骤放在一个`switch`语句的`case`子句中，并且我们将它放在一个`next(..)`可以调用的称为`nextState(..)`的内部函数中。另外，因为`x`是一个横跨整个“generator”作用域的变量，所以它需要存活在`nextState(..)`函数的外部。

这是将它们放在一起（很明显，为了使概念的展示更清晰，它经过了某些简化）：

```js
function foo() {
    function nextState(v) {
        switch (state) {
            case 0:
                state++;

                // `yield`表达式
                return 42;
            case 1:
                state++;

                // `yield`表达式完成了
                x = v;
                console.log( x );

                // 隐含的`return`
                return undefined;

            // 无需处理状态`2`
        }
    }

    var state = 0, x;

    return {
        next: function(v) {
            var ret = nextState( v );

            return { value: ret, done: (state == 2) };
        }

        // 我们将省略`return(..)`和`throw(..)`
    };
}
```

最后，让我们测试一下我们的前 ES6“generator”：

```js
var it = foo();

it.next();                // { value: 42, done: false }

it.next( 10 );            // 10
                        // { value: undefined, done: true }
```

不赖吧？希望这个练习能在你的脑中巩固这个概念：generator 实际上只是状态机逻辑的简单语法。这使它们可以广泛地应用。

### Generator 的使用

我们现在非常深入地理解了 generator 如何工作，那么，它们在什么地方有用？

我们已经看过了两种主要模式：

*   *生产一系列值：* 这种用法可以很简单（例如，随机字符串或者递增的数字），或者它也可以表达更加结构化的数据访问（例如，迭代一个数据库查询结果的所有行）。

    这两种方式中，我们使用迭代器来控制 generator，这样就可以为每次`next(..)`调用执行一些逻辑。在数据解构上的普通迭代器只不过生成值而没有任何控制逻辑。

*   *串行执行的任务队列：* 这种用法经常用来表达一个算法中步骤的流程控制，其中每一步都要求从某些外部数据源取得数据。对每块儿数据的请求可能会立即满足，或者可能会异步延迟地满足。

    从 generator 内部代码的角度来看，在`yield`的地方，同步或异步的细节是完全不透明的。另外，这些细节被有意地抽象出去，如此就不会让这样的实现细节把各个步骤间自然的，顺序的表达搞得模糊不清。抽象还意味着实现可以被替换/重构，而根本不用碰 generator 中的代码。

当根据这些用法观察 generator 时，它们的含义要比仅仅是手动状态机的一种不同或更好的语法多多了。它们是一种用于组织和控制有序地生产与消费数据的强大工具。