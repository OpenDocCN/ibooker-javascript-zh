# 你不懂 JS: 异步与性能 第四章: Generator（上）

在第二章中，我们发现了在使用回调表达异步流程控制时的两个关键缺陷：

*   基于回调的异步与我们的大脑规划任务的各个步骤的过程不相符。
*   由于 *控制倒转* 回调是不可靠的，也是不可组合的。

在第三章中，我们详细地讨论了 Promise 如何反转回调的 *控制倒转*，重建了可靠性/可组合性。

现在让我们把注意力集中到用一种顺序的，看起来同步的风格来表达异步流程控制。使这一切成为可能的“魔法”是 ES6 的 **generator**。

## 打破运行至完成

在第一章中，我们讲解了一个 JS 开发者们在他们的代码中几乎永恒依仗的一个认识：一旦函数开始执行，它将运行直至完成，没有其他的代码可以在运行期间干扰它。

这看起来可能很滑稽，ES6 引入了一种新型的函数，它不按照“运行至完成”的行为进行动作。这种新型的函数称为“generator（生成器）”。

为了理解它的含义，然我们看看这个例子：

```js
var x = 1;

function foo() {
    x++;
    bar();                // <-- 这一行会发生什么？
    console.log( "x:", x );
}

function bar() {
    x++;
}

foo();                    // x: 3
```

在这个例子中，我们确信`bar()`会在`x++`和`console.log(x)`之间运行。但如果`bar()`不在这里呢？很明显结果将是`2`而不是`3`。

现在让我们来燃烧你的大脑。要是`bar()`不存在，但以某种方式依然可以在`x++`和`console.log(x)`语句之间运行呢？这可能吗？

在 **抢占式（preemptive）** 多线程语言中，`bar()`去“干扰”并正好在两个语句之间那一时刻运行，实质上时可能的。但 JS 不是抢占式的，也（还）不是多线程的。但是，如果`foo()`本身可以用某种办法在代码的这一部分指示一个“暂停”，那么这种“干扰”（并发）的 **协作** 形式就是可能的。

**注意：** 我使用“协作”这个词，不仅是因为它与经典的并发术语有关联（见第一章），也因为正如你将在下一个代码段中看到的，ES6 在代码中指示暂停点的语法是`yield`——暗示一个让出控制权的礼貌的 *协作*。

这就是实现这种协作并发的 ES6 代码：

```js
var x = 1;

function *foo() {
    x++;
    yield; // 暂停！
    console.log( "x:", x );
}

function bar() {
    x++;
}
```

**注意：** 你将很可能在大多数其他的 JS 文档/代码中看到，一个 generator 的声明被格式化为`function* foo() { .. }`而不是我在这里使用的`function *foo() { .. }`——唯一的区别是摆放`*`位置的风格。这两种形式在功能性/语法上是完全一样的，还有第三种`function*foo() { .. }`（没空格）形式。这两种风格存在争议，但我基本上偏好`function *foo..`，因为当我在写作中用`*foo()`引用一个 generator 时，这种形式可以匹配我写的东西。如果我只说`foo()`，你就不会清楚地知道我是在说一个 generator 还是一个一般的函数。这纯粹是一个风格偏好的问题。

现在，我们该如何运行上面的代码，使`bar()`在`yield`那一点取代`*foo()`的执行？

```js
// 构建一个迭代器`it`来控制 generator
var it = foo();

// 在这里开始`foo()`！
it.next();
x;                        // 2
bar();
x;                        // 3
it.next();                // x: 3
```

好了，这两段代码中有不少新的，可能使人困惑的东西，所以我们得跋涉好一段了。在我们用 ES6 的 generator 来讲解不同的机制/语法之前，让我们过一遍这个行为的流程：

1.  `it = foo()`操作 *不会* 执行`*foo()`generator，它只不过构建了一个用来控制它执行的 *迭代器（iterator）*。我们一会更多地讨论 *迭代器*。
2.  第一个`it.next()`启动了`*foo()`generator，并且运行`*foo()`第一行上的`x++`。
3.  `*foo()`在`yield`语句处暂停，就在这时第一个`it.next()`调用结束。在这个时刻，`*foo()`依然运行而且是活动的，但是处于暂停状态。
4.  我们观察`x`的值，现在它是`2`.
5.  我们调用`bar()`，它再一次用`x++`递增`x`。
6.  我们再一次观察`x`的值，现在它是`3`。
7.  最后的`it.next()`调用使`*foo()`generator 从它暂停的地方继续运行，而后运行使用`x`的当前值`3`的`console.log(..)`语句。

清楚的是，`*foo()`启动了，但 *没有* 运行到底——它停在`yield`。我们稍后继续`*foo()`，让它完成，但这甚至不是必须的。

所以，一个 generator 是一种函数，它可以开始和停止一次或多次，甚至没必要一定要完成。虽然为什么它很强大看起来不那么明显，但正如我们将要在本章剩下的部分将要讲到的，它是我们用于在我们的代码中构建“generator 异步流程控制”模式的基础构建块儿之一。

### 输入和输出

一个 generator 函数是一种带有我们刚才提到的新型处理模型的函数。但它仍然是一个函数，这意味着依旧有一些不变的基本原则——即，它依然接收参数（也就是“输入”），而且它依然返回一个值（也就是“输出”）：

```js
function *foo(x,y) {
    return x * y;
}

var it = foo( 6, 7 );

var res = it.next();

res.value;        // 42
```

我们将`6`和`7`分别作为参数`x`和`y`传递给`*foo(..)`。而`*foo(..)`将值`42`返回给调用端代码。

现在我们可以看到发生器的调用和一般函数的调用的一个不同之处了。`foo(6,7)`显然看起来很熟悉。但微妙的是，`*foo(..)`generator 不会像一个函数那样实际运行起来。

相反，我们只是创建了 *迭代器* 对象，将它赋值给变量`it`，来控制`*foo(..)`generator。当我们调用`it.next()`时，它指示`*foo(..)`generator 从现在的位置向前推进，直到下一个`yield`或者 generator 的最后。

`next(..)`调用的结果是一个带有`value`属性的对象，它持有从`*foo(..)`返回的任何值（如果有的话）。换句话说，`yield`导致在 generator 运行期间，一个值被从中发送出来，有点儿像一个中间的`return`。

但是，为什么我们需要这个完全间接的 *迭代器* 对象来控制 generator 还不清楚。我们回头会讨论它的，我保证。

#### 迭代通信

generator 除了接收参数和拥有返回值，它们还内建有更强大，更吸引人的输入/输出消息能力，这是通过使用`yield`和`next(..)`实现的。

考虑下面的代码：

```js
function *foo(x) {
    var y = x * (yield);
    return y;
}

var it = foo( 6 );

// 开始`foo(..)`
it.next();

var res = it.next( 7 );

res.value;        // 42
```

首先，我们将`6`作为参数`x`传入。之后我们调用`it.next()`，它启动了`*foo(..)`.

在`*foo(..)`内部，`var y = x ..`语句开始被处理，但它运行到了一个`yield`表达式。就在这时，它暂停了`*foo(..)`（就在赋值语句的中间！），而且请求调用端代码为`yield`表达式提供一个结果值。接下来，我们调用`it.next(7)`，将`7`这个值传回去作为暂停的`yield`表达式的结果。

所以，在这个时候，赋值语句实质上是`var y = 6 * 7`。现在，`return y`将值`42`作为结果返回给`it.next( 7 )`调用。

注意一个非常重要，而且即便是对于老练的 JS 开发者也非常容易犯糊涂的事情：根据你的角度，在`yield`和`next(..)`调用之间存在着错位。一般来说，你所拥有的`next(..)`调用的数量，会比你所拥有的`yield`语句的数量多一个——前面的代码段中有一个`yield`和两个`next(..)`调用。

为什么会有这样的错位？

因为第一个`next(..)`总是启动一个 generator，然后运行至第一个`yield`。但是第二个`next(..)`调用满足了第一个暂停的`yield`表达式，而第三个`next(..)`将满足第二个`yield`，如此反复。

##### 两个疑问的故事

实际上，你主要考虑的是哪部分代码会影响你是否感知到错位。

仅考虑 generator 代码：

```js
var y = x * (yield);
return y;
```

这 **第一个** `yield`基本上是在 *问一个问题*：“我应该在这里插入什么值？”

谁来回答这个问题？好吧，**第一个** `next()`在这个时候已经为了启动 generator 而运行过了，所以很明显 *它* 不能回答这个问题。所以，**第二个** `next(..)`调用必须回答由 **第一个** `yield`提出的问题。

看到错位了吧——第二个对第一个？

但是让我们反转一下我们的角度。让我们不从 generator 的角度看问题，而从迭代器的角度看。

为了恰当地描述这种角度，我们还需要解释一下，消息可以双向发送——`yield ..`作为表达式可以发送消息来应答`next(..)`调用，而`next(..)`可以发送值给暂停的`yield`表达式。考虑一下这段稍稍调整过的代码：

```js
function *foo(x) {
    var y = x * (yield "Hello");    // <-- 让出一个值！
    return y;
}

var it = foo( 6 );

var res = it.next();    // 第一个`next()`，不传递任何东西
res.value;                // "Hello"

res = it.next( 7 );        // 传递`7`给等待中的`yield`
res.value;                // 42
```

`yield ..`和`next(..)`一起成对地 **在 generator 运行期间** 构成了一个双向消息传递系统。

那么，如果只看 *迭代器* 代码：

```js
var res = it.next();    // 第一个`next()`，不传递任何东西
res.value;                // "Hello"

res = it.next( 7 );        // 传递`7`给等待中的`yield`
res.value;                // 42
```

**注意：** 我们没有传递任何值给第一个`next()`调用，而且是故意的。只有一个暂停的`yield`才能接收这样一个被`next(..)`传递的值，但是当我们调用第一个`next()`时，在 generator 的最开始并 **没有任何暂停的`yield`** 可以接收这样的值。语言规范和所有兼容此语言规范的浏览器只会无声地 **丢弃** 任何传入第一个`next()`的东西。传递这样的值是一个坏主意，因为你只不过创建了一些令人困惑的无声“失败”的代码。所以，记得总是用一个无参数的`next()`来启动 generator。

第一个`next()`调用（没有任何参数的）基本上是在 *问一个问题*：“`*foo(..)`generator 将要给我的 *下一个* 值是什么？”，谁来回答这个问题？第一个`yield`表达式。

看到了？这里没有错位。

根据你认为是 *谁* 在问问题，在`yield`和`next(..)`之间的错位既存在又不存在。

但等一下！跟`yield`语句的数量比起来，还有一个额外的`next()`。那么，这个最后的`it.next(7)`调用又一次在询问 generator *下一个* 产生的值是什么。但是没有`yield`语句剩下可以回答了，不是吗？那么谁来回答？

`return`语句回答这个问题！

而且如果在你的 generator 中 **没有`return`**——比起一般的函数，generator 中的`return`当然不再是必须的——总会有一个假定/隐式的`return;`（也就是`return undefined;`），它默认的目的就是回答由最后的`it.next(7)`调用 *提出* 的问题。

这些问题与回答——用`yield`和`next(..)`进行双向消息传递——十分强大，但还是看不出来这些机制与异步流程控制有什么联系。我们正在接近真相！

### 多迭代器

从语法使用上来看，当你用一个 *迭代器* 来控制 generator 时，你正在控制声明的 generator 函数本身。但这里有一个容易忽视的微妙细节：每当你构建一个 *迭代器*，你都隐含地构建了一个将由这个 *迭代器* 控制的 generator 的实例。

你可以让同一个 generator 的多个实例同时运行，它们甚至可以互动：

```js
function *foo() {
    var x = yield 2;
    z++;
    var y = yield (x * z);
    console.log( x, y, z );
}

var z = 1;

var it1 = foo();
var it2 = foo();

var val1 = it1.next().value;            // 2 <-- 让出 2
var val2 = it2.next().value;            // 2 <-- 让出 2

val1 = it1.next( val2 * 10 ).value;        // 40  <-- x:20,  z:2
val2 = it2.next( val1 * 5 ).value;        // 600 <-- x:200, z:3

it1.next( val2 / 2 );                    // y:300
                                        // 20 300 3
it2.next( val1 / 4 );                    // y:10
                                        // 200 10 3
```

**警告：** 同一个 generator 的多个并发运行实例的最常见的用法，不是这样的互动，而是 generator 在没有输入的情况下，从一些连接着的独立资源中产生它自己的值。我们将在下一节中更多地讨论产生值。

让我们简单地走一遍这个处理过程：

1.  两个`*foo()`在同时启动，而且两个`next()`都分别从`yield 2`语句中得到了`2`的`value`。
2.  `val2 * 10`就是`2 * 10`，它被发送到第一个 generator 实例`it1`，所以`x`得到值`20`。`z`将`1`递增至`2`，然后`20 * 2`被`yield`出来，将`val1`设置为`40`。
3.  `val1 * 5`就是`40 * 5`，它被发送到第二个 generator 实例`it2`中，所以`x`得到值`200`。`z`又一次递增，从`2`到`3`，然后`200 * 3`被`yield`出来，将`val2`设置为`600`。
4.  `val2 / 2`就是`600 / 2`，它被发送到第一个 generator 实例`it1`，所以`y`得到值`300`，然后分别为它的`x y z`值打印出`20 300 3`。
5.  `val1 / 4`就是`40 / 4`，它被发送到第一个 generator 实例`it2`，所以`y`得到值`10`，然后分别为它的`x y z`值打印出`200 10 3`。

这是在你脑海中跑过的一个“有趣”的例子。你还能保持清醒？

#### 穿插

回想第一章中“运行至完成”一节的这个场景：

```js
var a = 1;
var b = 2;

function foo() {
    a++;
    b = b * a;
    a = b + 3;
}

function bar() {
    b--;
    a = 8 + b;
    b = a * 2;
}
```

使用普通的 JS 函数，当然要么是`foo()`可以首先运行完成，要么是`bar()`可以首先运行至完成，但是`foo()`不可能与`bar()`穿插它的独立语句。所以，前面这段代码只有两个可能的结果。

然而，使用 generator，明确地穿插（甚至是在语句中间！）是可能的：

```js
var a = 1;
var b = 2;

function *foo() {
    a++;
    yield;
    b = b * a;
    a = (yield b) + 3;
}

function *bar() {
    b--;
    yield;
    a = (yield 8) + b;
    b = a * (yield 2);
}
```

根据 *迭代器* 控制`*foo()`与`*bar()`分别以什么样的顺序被调用，前面这段代码可以产生几种不同的结果。换句话说，通过两个 generator 在同一个共享的变量上穿插，我们实际上可以展示（以一种模拟的方式）在第一章中讨论的，理论上的“线程的竞合状态”环境。

首先，让我们制造一个称为`step(..)`的帮助函数，让它控制 *迭代器*：

```js
function step(gen) {
    var it = gen();
    var last;

    return function() {
        // 不论`yield`出什么，只管在下一次时直接把它塞回去！
        last = it.next( last ).value;
    };
}
```

`step(..)`初始化一个 generator 来创建它的`it` *迭代器*，然后它返回一个函数，每次这个函数被调用时，都将 *迭代器* 向前推一步。另外，前一个被`yield`出来的值将被直接发给下一步。所以，`yield 8`将变成`8`而`yield b`将成为`b`（不管它在`yield`时是什么值）。

现在，为了好玩儿，让我们做一些实验，来看看将这些`*foo()`与`*bar()`的不同块儿穿插时的效果。我们从一个无聊的基本情况开始，保证`*foo()`在`*bar()`之前全部完成（就像我们在第一章中做的那样）：

```js
// 确保重置了`a`和`b`
a = 1;
b = 2;

var s1 = step( foo );
var s2 = step( bar );

// 首先完全运行`*foo()`
s1();
s1();
s1();

// 现在运行`*bar()`
s2();
s2();
s2();
s2();

console.log( a, b );    // 11 22
```

最终结果是`11`和`22`，就像第一章的版本那样。现在让我们把顺序混合穿插，来看看它如何改变`a`与`b`的值。

```js
// 确保重置了`a`和`b`
a = 1;
b = 2;

var s1 = step( foo );
var s2 = step( bar );

s2();        // b--;
s2();        // 让出 8
s1();        // a++;
s2();        // a = 8 + b;
            // 让出 2
s1();        // b = b * a;
            // 让出 b
s1();        // a = b + 3;
s2();        // b = a * 2;
```

在我告诉你结果之前，你能指出在前面的程序运行之后`a`和`b`的值是什么吗？不要作弊！

```js
console.log( a, b );    // 12 18
```

**注意：** 作为留给读者的练习，试试通过重新安排`s1()`和`s2()`调用的顺序，看看你能得到多少种结果组合。别忘了你总是需要三个`s1()`调用和四个`s2()`调用。至于为什么，回想一下刚才关于使用`yield`匹配`next()`的讨论。

当然，你几乎不会想有意制造 *这种* 水平的，令人糊涂的穿插，因为他创建了非常难理解的代码。但是这个练习很有趣，而且对于理解多个 generator 如何并发地运行在相同的共享作用域来说很有教育意义，因为会有一些地方这种能力十分有用。

我们会在本章末尾更详细地讨论 generator 并发。

## 生成值

在前一节中，我们提到了一个 generator 的有趣用法，作为一种生产值的方式。这 **不是** 我们本章主要关注的，但如果我们不在这里讲一下基本我们会想念它的，特别是因为这种用法实质上是它的名称的由来：生成器。

我们将要稍稍深入一下 *迭代器* 的话题，但我们会绕回到它们如何与 generator 关联，并使用 generator 来 *生成* 值。

### 发生器与迭代器

想象你正在生产一系列的值，它们中的每一个都与前一个值有可定义的关系。为此，你将需要一个有状态的发生器来记住上一个给出的值。

你可以用函数闭包（参加本系列的 *作用域与闭包*）来直接地实现这样的东西：

```js
var gimmeSomething = (function(){
    var nextVal;

    return function(){
        if (nextVal === undefined) {
            nextVal = 1;
        }
        else {
            nextVal = (3 * nextVal) + 6;
        }

        return nextVal;
    };
})();

gimmeSomething();        // 1
gimmeSomething();        // 9
gimmeSomething();        // 33
gimmeSomething();        // 105
```

**注意：** 这里`nextVal`的计算逻辑已经被简化了，但从概念上讲，直到 *下一次* `gimmeSomething()`调用发生之前，我们不想计算 *下一个值*（也就是`nextVal`），因为一般对于持久性更强的，或者比简单的`number`更有限的资源的发生器来说，那可能是一种资源泄漏的设计。

生成随意的数字序列不是是一个很真实的例子。但是如果你从一个数据源中生成记录呢？你可以想象很多相同的代码。

事实上，这种任务是一种非常常见的设计模式，通常用迭代器解决。一个 *迭代器* 是一个明确定义的接口，用来逐个通过一系列从发生器得到的值。迭代器的 JS 接口，和大多数语言一样，是在你每次想从发生器中得到下一个值时调用的`next()`。

我们可以为我们的数字序列发生器实现标准的 *迭代器*；

```js
var something = (function(){
    var nextVal;

    return {
        // `for..of`循环需要这个
        [Symbol.iterator]: function(){ return this; },

        // 标准的迭代器接口方法
        next: function(){
            if (nextVal === undefined) {
                nextVal = 1;
            }
            else {
                nextVal = (3 * nextVal) + 6;
            }

            return { done:false, value:nextVal };
        }
    };
})();

something.next().value;        // 1
something.next().value;        // 9
something.next().value;        // 33
something.next().value;        // 105
```

**注意：** 我们将在“Iterables”一节中讲解为什么我们在这个代码段中需要`[Symbol.iterator]: ..`这一部分。在语法上讲，两个 ES6 特性在发挥作用。首先，`[ .. ]`语法称为一个 *计算属性名*（参见本系列的 *this 与对象原型*）。它是一种字面对象定义方法，用来指定一个表达式并使用这个表达式的结果作为属性名。另一个，`Symbol.iterator`是 ES6 预定义的特殊`Symbol`值。

`next()`调用返回一个对象，它带有两个属性：`done`是一个`boolean`值表示 *迭代器* 的完成状态；`value`持有迭代的值。

ES6 还增加了`for..of`循环，它意味着一个标准的 *迭代器* 可以使用原生的循环语法来自动地被消费：

```js
for (var v of something) {
    console.log( v );

    // 不要让循环永无休止！
    if (v > 500) {
        break;
    }
}
// 1 9 33 105 321 969
```

**注意：** 因为我们的`something`迭代器总是返回`done:false`，这个`for..of`循环将会永远运行，这就是为什么我们条件性地放进一个`break`。对于迭代器来说永不终结是完全没有问题的，但是也有一些情况 *迭代器* 将运行在有限的值的集合上，而最终返回`done:true`。

`for..of`循环为每一次迭代自动调用`next()`——他不会给`next()`传入任何值——而且他将会在收到一个`done:true`时自动终结。这对于在一个集合的数据中进行循环十分方便。

当然，你可以手动循环一个迭代器，调用`next()`并检查`done:true`条件来知道什么时候停止：

```js
for (
    var ret;
    (ret = something.next()) && !ret.done;
) {
    console.log( ret.value );

    // 不要让循环永无休止！
    if (ret.value > 500) {
        break;
    }
}
// 1 9 33 105 321 969
```

**注意：** 这种手动的`for`方式当然要比 ES6 的`for..of`循环语法难看，但它的好处是它提供给你一个机会，在有必要时传值给`next(..)`调用。

除了制造你自己的 *迭代器* 之外，许多 JS 中（就 ES6 来说）内建的数据结构，比如`array`，也有默认的 *迭代器*：

```js
var a = [1,3,5,7,9];

for (var v of a) {
    console.log( v );
}
// 1 3 5 7 9
```

`for..of`循环向`a`要来它的迭代器，并自动使用它迭代`a`的值。

**注意：** 看起来像是一个 ES6 的奇怪省略，普通的`object`有意地不带有像`array`那样的默认 *迭代器*。原因比我们要在这里讲的深刻得多。如果你想要的只是迭代一个对象的属性（不特别保证顺序），`Object.keys(..)`返回一个`array`，它可以像`for (var k of Object.keys(obj)) { ..`这样使用。像这样用`for..of`循环一个对象上的键，与用`for..in`循环内很相似，除了在`for..in`中会包含`[[Prototype]]`链的属性，而`Object.keys(..)`不会（参见本系列的 *this 与对象原型*）。

### Iterables

在我们运行的例子中的`something`对象被称为一个 *迭代器*，因为它的接口中有`next()`方法。但一个紧密关联的术语是 *iterable*，它指 **包含有** 一个可以迭代它所有值的迭代器的对象。

在 ES6 中，从一个 *iterable* 中取得一个 *迭代器* 的方法是，*iterable* 上必须有一个函数，它的名称是特殊的 ES6 符号值`Symbol.iterator`。当这个函数被调用是，它就会返回一个 *迭代器*。虽然不是必须的，但一般来说每次调用应当返回一个全新的 *迭代器*。

前一个代码段的`a`就是一个 *iterable*。`for..of`循环自动地调用它的`Symbol.iterator`函数来构建一个 *迭代器*。我们当然可以手动地调用这个函数，然后使用它返回的 *iterator*：

```js
var a = [1,3,5,7,9];

var it = a[Symbol.iterator]();

it.next().value;    // 1
it.next().value;    // 3
it.next().value;    // 5
..
```

在前面定义`something`的代码段中，你可能已经注意到了这一行：

```js
[Symbol.iterator]: function(){ return this; }
```

这段有点让人困惑的代码制造了`something`值——`something`*迭代器* 的接口——也是一个 *iterable*；现在它既是一个 *iterable* 也是一个 *迭代器*。然后，我们把`something`传递给`for..of`循环：

```js
for (var v of something) {
    ..
}
```

`for..of`循环期待`something`是一个 *iterable*，所以它会寻找并调用它的`Symbol.iterator`函数。我们将这个函数定义为简单地`return this`，所以它将自己给出，而`for..of`不会知道这些。

### Generator 迭代器

带着 *迭代器* 的背景知识，让我们把注意力移回 generator。一个 generator 可以被看做一个值的发生器，我们通过一个 *迭代器* 接口的`next()`调用每次从中抽取一个值。

所以，一个 generator 本身在技术上讲并不是一个 *iterable*，虽然很相似——当你执行 generator 时，你就得到一个 *迭代器*：

```js
function *foo(){ .. }

var it = foo();
```

我们可以用 generator 实现早前的`something`无限数字序列发生器，就像这样：

```js
function *something() {
    var nextVal;

    while (true) {
        if (nextVal === undefined) {
            nextVal = 1;
        }
        else {
            nextVal = (3 * nextVal) + 6;
        }

        yield nextVal;
    }
}
```

**注意：** 在一个真实的 JS 程序中含有一个`while..true`循环通常是一件非常不好的事情，至少如果它没有一个`break`或`return`语句，那么它就很可能永远运行，并同步地，阻塞/锁定浏览器 UI。然而，在 generator 中，如果这样的循环含有一个`yield`，那它就是完全没有问题的，因为 generator 将在每次迭代后暂停，`yield`回主程序和/或事件轮询队列。说的明白点儿，“generator 把`while..true`带回到 JS 编程中了！”

这变得相当干净和简单点儿了，对吧？因为 generator 会暂停在每个`yield`，`*something()`函数的状态（作用域）被保持着，这意味着没有必要用闭包的模板代码来跨调用保留变量的状态了。

不仅是更简单的代码——我们不必自己制造 *迭代器* 借口了——它实际上是更合理的代码，因为它更清晰地表达了意图。比如，`while..true`循环告诉我们这个 generator 将要永远运行——只要我们一直向它请求，它就一直 *产生* 值。

现在我们可以在`for..of`循环中使用新得发亮的`*something()`generator 了，而且你会看到它工作起来基本一模一样：

```js
for (var v of something()) {
    console.log( v );

    // 不要让循环永无休止！
    if (v > 500) {
        break;
    }
}
// 1 9 33 105 321 969
```

不要跳过`for (var v of something()) ..`！我们不仅仅像之前的例子那样将`something`作为一个值引用了，而是调用`*something()`generator 来得到它的 *迭代器*，并交给`for..of`使用。

如果你仔细观察，在这个 generator 和循环的互动中，你可能会有两个疑问：

*   为什么我们不能说`for (var v of something) ..`？因为这个`something`是一个 generator，而不是一个 *iterable*。我们不得不调用`something()`来构建一个发生器给`for..of`，以便它可以迭代。
*   `something()`调用创建一个 *迭代器*，但是`for..of`想要一个 *iterable*，对吧？对，generator 的 *迭代器* 上也有一个`Symbol.iterator`函数，这个函数基本上就是`return this`，就像我们刚才定义的`something`*iterable*。换句话说 generator 的 *迭代器* 也是一个 *iterable*！

#### 停止 Generator

在前一个例子中，看起来在循环的`break`别调用后，`*something()`generator 的 *迭代器* 实例基本上被留在了一个永远挂起的状态。

但是这里有一个隐藏的行为为你处理这件事。`for..of`循环的“异常完成”（“提前终结”等等）——一般是由`break`，`return`，或未捕捉的异常导致的——会向 generator 的 *迭代器* 发送一个信号，以使它终结。

**注意：** 技术上讲，`for..of`循环也会在循环正常完成时向 *迭代器* 发送这个信号。对于 generator 来说，这实质上是一个无实际意义的操作，因为 generator 的 *迭代器* 要首先完成，`for..of`循环才能完成。然而，自定义的 *迭代器* 可能会希望从`for..of`循环的消费者那里得到另外的信号。

虽然一个`for..of`循环将会自动发送这种信号，你可能会希望手动发送信号给一个 *迭代器*；你可以通过调用`return(..)`来这么做。

如果你在 generator 内部指定一个`try..finally`从句，它将总是被执行，即便是 generator 从外部被完成。这在你需要进行资源清理时很有用（数据库连接等）：

```js
function *something() {
    try {
        var nextVal;

        while (true) {
            if (nextVal === undefined) {
                nextVal = 1;
            }
            else {
                nextVal = (3 * nextVal) + 6;
            }

            yield nextVal;
        }
    }
    // 清理用的从句
    finally {
        console.log( "cleaning up!" );
    }
}
```

前面那个在`for..of`中带有`break`的例子将会触发`finally`从句。但是你可以用`return(..)`从外部来手动终结 generator 的 *迭代器* 实例：

```js
var it = something();
for (var v of it) {
    console.log( v );

    // 不要让循环永无休止！
    if (v > 500) {
        console.log(
            // 使 generator 得迭代器完成
            it.return( "Hello World" ).value
        );
        // 这里不需要`break`
    }
}
// 1 9 33 105 321 969
// cleaning up!
// Hello World
```

当我们调用`it.return(..)`时，它会立即终结 generator，从而运行`finally`从句。而且，它会将返回的`value`设置为你传入`return(..)`的任何东西，这就是`Hellow World`如何立即返回来的。我们现在也不必再包含一个`break`，因为 generator 的 *迭代器* 会被设置为`done:true`，所以`for..of`循环会在下一次迭代时终结。

generator 的命名大部分源自于这种 *消费生产的值* 的用法。但要重申的是，这只是 generator 的用法之一，而且坦白的说，在这本书的背景下这甚至不是我们主要关注的。

但是现在我们更加全面地了解它们的机制是如何工作的，我们接下来可以将注意力转向 generator 如何实施于异步并发。

## 异步地迭代 Generator

generator 要怎样处理异步编码模式，解决回调和类似的问题？让我们开始回答这个重要的问题。

我们应当重温一下第三章的一个场景。回想一下这个回调方式：

```js
function foo(x,y,cb) {
    ajax(
        "http://some.url.1/?x=" + x + "&y=" + y,
        cb
    );
}

foo( 11, 31, function(err,text) {
    if (err) {
        console.error( err );
    }
    else {
        console.log( text );
    }
} );
```

如果我们想用 generator 表示相同的任务流控制，我们可以：

```js
function foo(x,y) {
    ajax(
        "http://some.url.1/?x=" + x + "&y=" + y,
        function(err,data){
            if (err) {
                // 向`*main()`中扔进一个错误
                it.throw( err );
            }
            else {
                // 使用收到的`data`来继续`*main()`
                it.next( data );
            }
        }
    );
}

function *main() {
    try {
        var text = yield foo( 11, 31 );
        console.log( text );
    }
    catch (err) {
        console.error( err );
    }
}

var it = main();

// 使一切开始运行！
it.next();
```

一眼看上去，这个代码段要比以前的回调代码更长，而且也许看起来更复杂。但不要让这种印象误导你。generator 的代码段实际上要好 **太多** 了！但是这里有很多我们需要讲解的。

首先，让我们看看代码的这一部分，也是最重要的部分：

```js
var text = yield foo( 11, 31 );
console.log( text );
```

花一点时间考虑一下这段代码如何工作。我们调用了一个普通的函数`foo(..)`，而且我们显然可以从 Ajax 调用那里得到`text`，即便它是异步的。

这怎么可能？如果你回忆一下第一章的最开始，我们有一个几乎完全一样的代码：

```js
var data = ajax( "..url 1.." );
console.log( data );
```

但是这段代码不好用！你能发现不同吗？它就是在 generator 中使用的`yield`。

这就是魔法发生的地方！是它允许我们拥有一个看起来是阻塞的，同步的，但实际上不会阻塞整个程序的代码；它仅仅暂停/阻塞在 generator 本身的代码。

在`yield foo(11,31)`中，首先`foo(11,31)`调用被发起，它什么也不返回（也就是`undefined`），所以我们发起了数据请求，然后我们实际上做的是`yield undefined`。这没问题，因为这段代码现在没有依赖`yield`的值来做任何有趣的事。我们在本章稍后再重新讨论这个问题。

在这里，我们没有将`yield`作为消息传递的工具，只是作为进行暂停/阻塞的流程控制的工具。实际上，它会传递消息，但是只是单向的，在 generator 被继续运行之后。

那么，generator 暂停在了`yield`，它实质上再问一个问题，“我该将什么值返回并赋给变量`text`？”谁来回答这个问题？

看一下`foo(..)`。如果 Ajax 请求成功，我们调用：

```js
it.next( data );
```

这将使 generator 使用应答数据继续运行，这意味着我们暂停的`yield`表达式直接收到这个值，然后因为它重新开始以运行 generator 代码，所以这个值被赋给本地变量`text`。

很酷吧？

退一步考虑一下它的意义。我们在 generator 内部的代码看起来完全是同步的（除了`yield`关键字本身），但隐藏在幕后的是，在`foo(..)`内部，操作可以完全是异步的。

**这很伟大！** 这几乎完美地解决了我们前面遇到的问题：回调不能像我们的大脑可以关联的那样，以一种顺序，同步的风格表达异步处理。

实质上，我们将异步处理作为实现细节抽象出去，以至于我们可以同步地/顺序地推理我们的流程控制：“发起 Ajax 请求，然后在它完成之后打印应答。” 当然，我们仅仅在这个流程控制中表达了两个步骤，但同样的能力可以无边界地延伸，让我们需要表达多少步骤，就表达多少。

**提示：** 这是一个如此重要的认识，为了充分理解，现在回过头去再把最后三段读一遍！

### 同步错误处理

但是前面的 generator 代码会 *让* 出更多的好处给我们。让我们把注意力移到 generator 内部的`try..catch`上：

```js
try {
    var text = yield foo( 11, 31 );
    console.log( text );
}
catch (err) {
    console.error( err );
}
```

这是怎么工作的？`foo(..)`调用是异步完成的，`try..catch`不是无法捕捉异步错误吗？就像我们在第三章中看到的？

我们已经看到了`yield`如何让赋值语句暂停，来等待`foo(..)`去完成，以至于完成的响应可以被赋予`text`。牛 X 的是，`yield`暂停 *还* 允许 generator 来`catch`一个错误。我们在前面的例子，我们用这一部分代码将这个错误抛出到 generator 中：

```js
if (err) {
    // 向`*main()`中扔进一个错误
    it.throw( err );
}
```

generator 的`yield`暂停特性不仅意味着我们可以从异步的函数调用那里得到看起来同步的`return`值，还意味着我们可以同步地捕获这些异步函数调用的错误！

那么我们看到了，我们可以将错误 *抛入* generator，但是将错误 *抛出* 一个 generator 呢？和你期望的一样：

```js
function *main() {
    var x = yield "Hello World";

    yield x.toLowerCase();    // 引发一个异常！
}

var it = main();

it.next().value;            // Hello World

try {
    it.next( 42 );
}
catch (err) {
    console.error( err );    // TypeError
}
```

当然，我们本可以用`throw ..`手动地抛出一个错误，而不是制造一个异常。

我们甚至可以`catch`我们`throw(..)`进 generator 的同一个错误，实质上给了 generator 一个机会来处理它，但如果 generator 没处理，那么 *迭代器* 代码必须处理它：

```js
function *main() {
    var x = yield "Hello World";

    // 永远不会跑到这里
    console.log( x );
}

var it = main();

it.next();

try {
    // `*main()`会处理这个错误吗？我们走着瞧！
    it.throw( "Oops" );
}
catch (err) {
    // 不，它没处理！
    console.error( err );            // Oops
}
```

使用异步代码的，看似同步的错误处理（通过`try..catch`）在可读性和可推理性上大获全胜。

## Generators + Promises

在我们前面的讨论中，我们展示了 generator 如何可以异步地迭代，这是一个用顺序的可推理性来取代混乱如面条的回调的一个巨大进步。但我们丢掉了两个非常重要的东西：Promise 的可靠性和可组合性（见第三章）！

别担心——我们会把它们拿回来。在 ES6 的世界中最棒的就是将 generator（看似同步的异步代码）与 Promise（可靠性和可组合性）组合起来。

但怎么做呢？

回想一下第三章中我们基于 Promise 的方式运行 Ajax 的例子：

```js
function foo(x,y) {
    return request(
        "http://some.url.1/?x=" + x + "&y=" + y
    );
}

foo( 11, 31 )
.then(
    function(text){
        console.log( text );
    },
    function(err){
        console.error( err );
    }
);
```

在我们早先的运行 Ajax 的例子的 generator 代码中，`foo(..)`什么也不返回（`undefined`），而且我们的 *迭代器* 控制代码也不关心`yield`的值。

但这里的 Promise 相关的`foo(..)`在发起 Ajax 调用后返回一个 promise。这暗示着我们可以用`foo(..)`构建一个 promise，然后从 generator 中`yield`出来，而后 *迭代器* 控制代码将可以收到这个 promise。

那么 *迭代器* 应当对 promise 做什么？

它应当监听 promise 的解析（完成或拒绝），然后要么使用完成消息继续运行 generator，要么使用拒绝理由向 generator 抛出错误。

让我重复一遍，因为它如此重要。发挥 Promise 和 generator 的最大功效的自然方法是 **`yield`一个 Promise**，并将这个 Promise 连接到 generator 的 *迭代器* 的控制端。

让我们试一下！首先，我们将 Promise 相关的`foo(..)`与 generator`*main()`放在一起：

```js
function foo(x,y) {
    return request(
        "http://some.url.1/?x=" + x + "&y=" + y
    );
}

function *main() {
    try {
        var text = yield foo( 11, 31 );
        console.log( text );
    }
    catch (err) {
        console.error( err );
    }
}
```

在这个重构中最强大的启示是，`*main()`内部的代码 **更本就没变！** 在 generator 内部，无论什么样的值被`yield`出去都是一个不可见的实现细节，所以我们甚至不会察觉它发生了，也不用担心它。

那么我们现在如何运行`*main()`？我们还有一些管道的实现工作要做，接收并连接`yield`的 promise，使它能够根据解析来继续运行 generator。我们从手动这么做开始：

```js
var it = main();

var p = it.next().value;

// 等待`p` promise 解析
p.then(
    function(text){
        it.next( text );
    },
    function(err){
        it.throw( err );
    }
);
```

其实，根本不费事，对吧？

这段代码应当看起来与我们早前做的很相似：手动地连接被错误优先的回调控制的 generator。与`if (err) { it.throw..`不同的是，promise 已经为我们分割为完成（成功）与拒绝（失败），否则 *迭代器* 控制是完全相同的。

现在，我们已经掩盖了一些重要的细节。

最重要的是，我们利用了这样一个事实：我们知道`*main()`里面只有一个 Promise 相关的步骤。如果我们想要能用 Promise 驱动一个 generator 而不管它有多少步骤呢？我们当然不想为每一个 generator 手动编写一个不同的 Promise 链！要是有这样一种方法该多好：可以重复（也就是“循环”）迭代的控制，而且每次一有 Promise 出来，就在继续之前等待它的解析。

另外，如果 generator 在`it.next()`调用期间抛出一个错误怎么办？我们是该退出，还是应该`catch`它并把它送回去？相似地，要是我们`it.throw(..)`一个 Promise 拒绝给 generator，但是没有被处理，又直接回来了呢？

### 带有 Promise 的 Generator 运行器

你在这条路上探索得越远，你就越能感到，“哇，要是有一些工具能帮我做这些就好了。”而且你绝对是对的。这是一种如此重要的模式，而且你不想把它弄错（或者因为一遍又一遍地重复它而把自己累死），所以你最好的选择是把赌注压在一个工具上，而它以我们将要描述的方式使用这种特定设计的工具来 *运行* `yield`Promise 的 generator。

有几种 Promise 抽象库提供了这样的工具，包括我的 *asynquence* 库和它的`runner(..)`，我们将在本书的在附录 A 中讨论它。

但看在学习和讲解的份儿上，让我们定义我们自己的名为`run(..)`的独立工具：

```js
// 感谢 Benjamin Gruenbaum (@benjamingr 在 GitHub)在此做出的巨大改进！
function run(gen) {
    var args = [].slice.call( arguments, 1), it;

    // 在当前的上下文环境中初始化 generator
    it = gen.apply( this, args );

    // 为 generator 的完成返回一个 promise
    return Promise.resolve()
        .then( function handleNext(value){
            // 运行至下一个让出的值
            var next = it.next( value );

            return (function handleResult(next){
                // generator 已经完成运行了？
                if (next.done) {
                    return next.value;
                }
                // 否则继续执行
                else {
                    return Promise.resolve( next.value )
                        .then(
                            // 在成功的情况下继续异步循环，将解析的值送回 generator
                            handleNext,

                            // 如果`value`是一个拒绝的 promise，就将错误传播回 generator 自己的错误处理 g
                            function handleErr(err) {
                                return Promise.resolve(
                                    it.throw( err )
                                )
                                .then( handleResult );
                            }
                        );
                }
            })(next);
        } );
}
```

如你所见，它可能比你想要自己编写的东西复杂得多，特别是你将不会想为每个你使用的 generator 重复这段代码。所以，一个帮助工具/库绝对是可行的。虽然，我鼓励你花几分钟时间研究一下这点代码，以便对如何管理 generator+Promise 交涉得到更好的感觉。

你如何在我们 *正在讨论* 的 Ajax 例子中将`run(..)`和`*main()`一起使用呢？

```js
function *main() {
    // ..
}

run( main );
```

就是这样！按照我们连接`run(..)`的方式，它将自动地，异步地推进你传入的 generator，直到完成。

**注意：** 我们定义的`run(..)`返回一个 promise，它被连接成一旦 generator 完成就立即解析，或者收到一个未捕获的异常，而 generator 没有处理它。我们没有在这里展示这种能力，但我们会在本章稍后回到这个话题。

#### ES7: `async` 和 `await`？

前面的模式——generator 让出一个 Promise，然后这个 Promise 控制 generator 的 *迭代器* 向前推进至它完成——是一个如此强大和有用的方法，如果我们能不通过乱七八糟的帮助工具库（也就是`run(..)`）来使用它就更好了。

在这方面可能有一些好消息。在写作这本书的时候，后 ES6，ES7 化的时间表上已经出现了草案，对这个问题提供早期但强大的附加语法支持。显然，现在还太早而不能保证其细节，但是有相当大的可能性它将蜕变为类似于下面的东西：

```js
function foo(x,y) {
    return request(
        "http://some.url.1/?x=" + x + "&y=" + y
    );
}

async function main() {
    try {
        var text = await foo( 11, 31 );
        console.log( text );
    }
    catch (err) {
        console.error( err );
    }
}

main();
```

如你所见，这里没有`run(..)`调用（意味着不需要工具库！）来驱动和调用`main()`——它仅仅像一个普通函数那样被调用。另外，`main()`不再作为一个 generator 函数声明；它是一种新型的函数：`async function`。而最后，与`yield`一个 Promise 相反，我们`await`它解析。

如果你`await`一个 Promise，`async function`会自动地知道做什么——它会暂停这个函数（就像使用 generator 那样）直到 Promise 解析。我们没有在这个代码段中展示，但是调用一个像`main()`这样的异步函数将自动地返回一个 promise，它会在函数完全完成时被解析。

**提示：** `async` / `await`的语法应该对拥有 C#经验的读者看起来非常熟悉，因为它们基本上是一样的。

这个草案实质上是为我们已经衍生出的模式进行代码化的支持，成为一种语法机制：用看似同步的流程控制代码与 Promise 组合。将两个世界的最好部分组合，来有效解决我们用回调遇到的几乎所有主要问题。

这样的 ES7 化草案已经存在，并且有了早期的支持和热忱的拥护。这一事实为这种异步模式在未来的重要性上信心满满地投了有力的一票。

### Generator 中的 Promise 并发

至此，所有我们展示过的是一种使用 Promise+generator 的单步异步流程。但是现实世界的代码将总是有许多异步步骤。

如果你不小心，generator 看似同步的风格也许会蒙蔽你，使你在如何构造你的异步并发上感到自满，导致性能次优的模式。那么我们想花一点时间来探索一下其他选项。

想象一个场景，你需要从两个不同的数据源取得数据，然后将这些应答组合来发起第三个请求，最后打印出最终的应答。我们在第三章中用 Promise 探索过类似的场景，但这次让我们在 generator 的环境下考虑它。

你的第一直觉可能是像这样的东西：

```js
function *foo() {
    var r1 = yield request( "http://some.url.1" );
    var r2 = yield request( "http://some.url.2" );

    var r3 = yield request(
        "http://some.url.3/?v=" + r1 + "," + r2
    );

    console.log( r3 );
}

// 使用刚才定义的`run(..)`工具
run( foo );
```

这段代码可以工作，但在我们特定的这个场景中，它不是最优的。你能发现为什么吗？

因为`r1`和`r2`请求可以——而且为了性能的原因，*应该*——并发运行，但在这段代码中它们将顺序地运行；直到`"http://some.url.1"`请求完成之前，`"http://some.url.2"`URL 不会被 Ajax 取得。这两个请求是独立的，所以性能更好的方式可能是让它们同时运行。

但是使用 generator 和`yield`，到底应该怎么做？我们知道`yield`在代码中只是一个单独的暂停点，所以你根本不能再同一时刻做两次暂停。

最自然和有效的答案是基于 Promise 的异步流程，特别是因为它们的时间无关的状态管理能力（参见第三章的“未来的值”）。

最简单的方式：

```js
function *foo() {
    // 使两个请求“并行”
    var p1 = request( "http://some.url.1" );
    var p2 = request( "http://some.url.2" );

    // 等待两个 promise 都被解析
    var r1 = yield p1;
    var r2 = yield p2;

    var r3 = yield request(
        "http://some.url.3/?v=" + r1 + "," + r2
    );

    console.log( r3 );
}

// 使用刚才定义的`run(..)`工具
run( foo );
```

为什么这与前一个代码段不同？看看`yield`在哪里和不在哪里。`p1`和`p2`是并发地（也就是“并行”）发起的 Ajax 请求 promise。它们哪一个先完成都不要紧，因为 promise 会一直保持它们的解析状态。

然后我们使用两个连续的`yield`语句等待并从 promise 中取得解析值（分别取到`r1`和`r2`中）。如果`p1`首先解析，`yield p1`会首先继续执行然后等待`yield p2`继续执行。如果`p2`首先解析，它将会耐心地保持解析值知道被请求，但是`yield p1`将会首先停住，直到`p1`解析。

不管是哪一种情况，`p1`和`p2`都将并发地运行，并且在`r3 = yield request..`Ajax 请求发起之前，都必须完成，无论以哪种顺序。

如果这种流程控制处理模型听起来很熟悉，那是因为它基本上和我们在第三章中介绍的，因`Promise.all([ .. ])`工具成为可能的“门”模式是相同的。所以，我们也可以像这样表达这种流程控制：

```js
function *foo() {
    // 使两个请求“并行”并等待两个 promise 都被解析
    var results = yield Promise.all( [
        request( "http://some.url.1" ),
        request( "http://some.url.2" )
    ] );

    var r1 = results[0];
    var r2 = results[1];

    var r3 = yield request(
        "http://some.url.3/?v=" + r1 + "," + r2
    );

    console.log( r3 );
}

// 使用前面定义的`run(..)`工具
run( foo );
```

**注意：** 就像我们在第三章中讨论的，我们甚至可以用 ES6 解构赋值来把`var r1 = .. var r2 = ..`赋值简写为`var [r1,r2] = results`。

换句话说，在 generator+Promise 的方式中，Promise 所有的并发能力都是可用的。所以在任何地方，如果你需要比“这个然后那个”要复杂的顺序异步流程步骤时，Promise 都可能是最佳选择。

#### Promises，隐藏起来

作为代码风格的警告要说一句，要小心你在 **你的 generator 内部** 包含了多少 Promise 逻辑。以我们描述过的方式在异步性上使用 generator 的全部意义，是要创建简单，顺序，看似同步的代码，并尽可能多地将异步性细节隐藏在这些代码之外。

比如，这可能是一种更干净的方式：

```js
// 注意：这是一个普通函数，不是 generator
function bar(url1,url2) {
    return Promise.all( [
        request( url1 ),
        request( url2 )
    ] );
}

function *foo() {
    // 将基于 Promise 的并发细节隐藏在`bar(..)`内部
    var results = yield bar(
        "http://some.url.1",
        "http://some.url.2"
    );

    var r1 = results[0];
    var r2 = results[1];

    var r3 = yield request(
        "http://some.url.3/?v=" + r1 + "," + r2
    );

    console.log( r3 );
}

// 使用刚才定义的`run(..)`工具
run( foo );
```

在`*foo()`内部，它更干净更清晰地表达了我们要做的事情：我们要求`bar(..)`给我们一些`results`，而我们将用`yield`等待它的发生。我们不必关心在底层一个`Promise.all([ .. ])`的 Promise 组合将被用来完成任务。

**我们将异步性，特别是 Promise，作为一种实现细节。**

如果你要做一种精巧的序列流控制，那么将你的 Promise 逻辑隐藏在一个仅仅从你的 generator 中调用的函数里特别有用。举个例子：

```js
function bar() {
    Promise.all( [
        baz( .. )
        .then( .. ),
        Promise.race( [ .. ] )
    ] )
    .then( .. )
}
```

有时候这种逻辑是必须的，而如果你直接把它扔在你的 generator 内部，你就违背了大多数你使用 generator 的初衷。我们 *应当* 有意地将这样的细节从 generator 代码中抽象出去，以使它们不会搞乱更高层的任务表达。

在创建功能强与性能好的代码之上，你还应当努力使代码尽可能地容易推理和维护。

**注意：** 对于编程来说，抽象不总是一种健康的东西——许多时候它可能在得到简洁的同时增加复杂性。但是在这种情况下，我相信你的 generator+Promise 异步代码要比其他的选择健康得多。虽然有所有这些建议，你仍然要注意你的特殊情况，并为你和你的团队做出合适的决策。