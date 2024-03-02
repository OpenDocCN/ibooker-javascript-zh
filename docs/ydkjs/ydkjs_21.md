# 你不懂 JS: 异步与性能 第三章: Promise（下）

## 错误处理

我们已经看过几个例子，Promise 拒绝——既可以通过有意调用`reject(..)`，也可以通过意外的 JS 异常——是如何在异步编程中允许清晰的错误处理的。让我们兜个圈子回去，将我们一带而过的一些细节弄清楚。

对大多数开发者来说，最自然的错误处理形式是同步的`try..catch`结构。不幸的是，它仅能用于同步状态，所以在异步代码模式中它帮不上什么忙：

```js
function foo() {
    setTimeout( function(){
        baz.bar();
    }, 100 );
}

try {
    foo();
    // 稍后会从`baz.bar()`抛出全局错误
}
catch (err) {
    // 永远不会到这里
}
```

能有`try..catch`当然很好，但除非有某些附加的环境支持，它无法与异步操作一起工作。我们将会在第四章中讨论 generator 时回到这个话题。

在回调中，对于错误处理的模式已经有了一些新兴的模式，最有名的就是“错误优先回调”风格：

```js
function foo(cb) {
    setTimeout( function(){
        try {
            var x = baz.bar();
            cb( null, x ); // 成功！
        }
        catch (err) {
            cb( err );
        }
    }, 100 );
}

foo( function(err,val){
    if (err) {
        console.error( err ); // 倒霉 :(
    }
    else {
        console.log( val );
    }
} );
```

**注意：** 这里的`try..catch`仅在`baz.bar()`调用立即地，同步地成功或失败时才能工作。如果`baz.bar()`本身是一个异步完成的函数，它内部的任何异步错误都不能被捕获。

我们传递给`foo(..)`的回调期望通过预留的`err`参数收到一个表示错误的信号。如果存在，就假定出错。如果不存在，就假定成功。

这类错误处理在技术上是 *异步兼容的*，但它根本组织的不好。用无处不在的`if`语句检查将多层错误优先回调编织在一起，将不可避免地将你置于回调地狱的危险之中（见第二章）。

那么我们回到 Promise 的错误处理，使用传递给`then(..)`的拒绝处理器。Promise 不使用流行的“错误优先回调”设计风格，反而使用“分割回调”的风格；一个回调给完成，一个回调给拒绝：

```js
var p = Promise.reject( "Oops" );

p.then(
    function fulfilled(){
        // 永远不会到这里
    },
    function rejected(err){
        console.log( err ); // "Oops"
    }
);
```

虽然这种模式表面上看起来十分有道理，但是 Promise 错误处理的微妙之处经常使它有点儿相当难以全面把握。

考虑下面的代码：

```js
var p = Promise.resolve( 42 );

p.then(
    function fulfilled(msg){
        // 数字没有字符串方法,
        // 所以这里抛出一个错误
        console.log( msg.toLowerCase() );
    },
    function rejected(err){
        // 永远不会到这里
    }
);
```

如果`msg.toLowerCase()`合法地抛出一个错误（它会的！），为什么我们的错误处理器没有得到通知？正如我们早先解释的，这是因为 *这个* 错误处理器是为`p`promise 准备的，也就是已经被值`42`完成的那个 promise。`p`promise 是不可变的，所以唯一可以得到错误通知的 promise 是由`p.then(..)`返回的那个，而在这里我们没有捕获它。

这应当解释了：为什么 Promise 的错误处理是易错的。错误太容易被吞掉了，而这很少是你有意这么做的。

**警告：** 如果你以一种不合法的方式使用 Promise API，而且有错误阻止正常的 Promise 构建，其结果将是一个立即被抛出的异常，**而不是一个拒绝 Promise**。这是一些导致 Promise 构建失败的错误用法：`new Promise(null)`，`Promise.all()`，`Promise.race(42)`等等。如果你没有足够合法地使用 Promise API 来首先实际构建一个 Promise，你就不能得到一个拒绝 Promise！

### 绝望的深渊

几年前 Jeff Atwood 曾经写到：编程语言总是默认地以这样的方式建立，开发者们会掉入“绝望的深渊”（[`blog.codinghorror.com/falling-into-the-pit-of-success/`](http://blog.codinghorror.com/falling-into-the-pit-of-success/) ）——在这里意外会被惩罚——而你不得不更努力地使它正确。他恳求我们相反地创建“成功的深渊”，就是你会默认地掉入期望的（成功的）行为，而如此你不得不更努力地去失败。

毫无疑问，Promise 的错误处理是一种“绝望的深渊”的设计。默认情况下，它假定你想让所有的错误都被 Promise 的状态吞掉，而且如果你忘记监听这个状态，错误就会默默地凋零/死去——通常是绝望的。

为了回避把一个被遗忘/抛弃的 Promise 的错误无声地丢失，一些开发者宣称 Promise 链的“最佳实践”是，总是将你的链条以`catch(..)`终结，就像这样：

```js
var p = Promise.resolve( 42 );

p.then(
    function fulfilled(msg){
        // 数字没有字符串方法,
        // 所以这里抛出一个错误
        console.log( msg.toLowerCase() );
    }
)
.catch( handleErrors );
```

因为我们没有给`then(..)`传递拒绝处理器，默认的处理器会顶替上来，它仅仅简单地将错误传播到链条的下一个 promise 中。如此，在`p`中发生的错误，与在`p`之后的解析中（比如`msg.toLowerCase()`）发生的错误都将会过滤到最后的`handleErrors(..)`中。

问题解决了，对吧？没那么容易！

要是`handleErrors(..)`本身也有错误呢？谁来捕获它？这里还有一个没人注意的 promise：`catch(..)`返回的 promise，我们没有对它进行捕获，也没注册拒绝处理器。

你不能仅仅将另一个`catch(..)`贴在链条末尾，因为它也可能失败。Promise 链的最后一步，无论它是什么，总有可能，即便这种可能性逐渐减少，悬挂着一个困在未被监听的 Promise 中的，未被捕获的错误。

听起来像一个不可解的迷吧？

### 处理未被捕获的错误

这不是一个很容易就能完全解决的问题。但是有些接近于解决的方法，或者说 *更好的方法*。

一些 Promise 库有一些附加的方法，可以注册某些类似于“全局的未处理拒绝”的处理器，全局上不会抛出错误，而是调用它。但是他们识别一个错误是“未被捕获的错误”的方案是，使用一个任意长的计时器，比如说 3 秒，从拒绝的那一刻开始计时。如果一个 Promise 被拒绝但没有错误处理在计时器被触发前注册，那么它就假定你不会注册监听器了，所以它是“未被捕获的”。

实践中，这个方法在许多库中工作的很好，因为大多数用法不会在 Promise 拒绝和监听这个拒绝之间有很明显的延迟。但是这个模式有点儿麻烦，因为 3 秒实在太随意了（即便它是实证过的），还因为确实有些情况你想让一个 Promise 在一段不确定的时间内持有它的拒绝状态，而且你不希望你的“未捕获错误”处理器因为这些具有正面含义的不成立（还没处理的“未捕获错误”）而被调用。

另一种常见的建议是，Promise 应当增加一个`done(..)`方法，它实质上标志着 Promise 链的“终结”。`done(..)`不会创建并返回一个 Promise，所以传递给`done(..)`的回调很明显地不会链接上一个不存在的 Promise 链，并向它报告问题。

那么接下来会发什么？正如你通常在未处理错误状态下希望的那样，在`done(..)`的拒绝处理器内部的任何异常都作为全局的未捕获错误抛出（基本上扔到开发者控制台）：

```js
var p = Promise.resolve( 42 );

p.then(
    function fulfilled(msg){
        // 数字没有字符串方法,
        // 所以这里抛出一个错误
        console.log( msg.toLowerCase() );
    }
)
.done( null, handleErrors );

// 如果`handleErrors(..)`自身发生异常，它会在这里被抛出到全局
```

这听起来要比永不终结的链条或随意的超时要吸引人。但最大的问题是，它不是 ES6 标准，所以不管听起来多么好，它成为一个可靠而普遍的解决方案还有很长的距离。

那我们就卡在这里了？不完全是。

浏览器有一个我们的代码没有的能力：它们可以追踪并确定一个对象什么时候被废弃并可以作为垃圾回收。所以，浏览器可以追踪 Promise 对象，当它们被当做垃圾回收时，如果在它们内部存在一个拒绝状态，浏览器就可以确信这是一个合法的“未捕获错误”，它可以信心十足地知道应当在开发者控制台上报告这一情况。

**注意：** 在写作本书的时候，Chrome 和 Firefox 都早已试图实现这种“未捕获拒绝”的能力，虽然至多也就是支持的不完整。

然而，如果一个 Promise 不被垃圾回收——通过许多不同的代码模式，这极其容易不经意地发生——浏览器的垃圾回收检测不会帮你知道或诊断你有一个拒绝的 Promise 静静地躺在附近。

还有其他选项吗？有。

### 成功的深渊

以下讲的仅仅是理论上，Promise *可能* 在某一天变成什么样的行为。我相信那会比我们现在拥有的优越许多。而且我想这种改变可能会发生在后 ES6 时代，因为我不认为它会破坏 Web 的兼容性。另外，如果你小心行事，它是可以被填补（polyfilled）/预填补（prollyfilled）的。让我们来看一下：

*   Promise 可以默认为是报告(向开发者控制台)一切拒绝的，就在下一个 Job 或事件轮询 tick，如果就在这时 Promise 上没有注册任何错误处理器。
*   如果你希望拒绝的 Promise 在被监听前，将其拒绝状态保持一段不确定的时间。你可以调用`defer()`，它会压制这个 Promise 自动报告错误。

如果一个 Promise 被拒绝，默认地它会吵吵闹闹地向开发者控制台报告这个情况（而不是默认不出声）。你既可以选择隐式地处理这个报告（通过在拒绝之前注册错误处理器），也可以选择明确地处理这个报告（使用`defer()`）。无论哪种情况，*你* 都控制着这种具有正面意义的不成立。

考虑下面的代码：

```js
var p = Promise.reject( "Oops" ).defer();

// `foo(..)`返回 Promise
foo( 42 )
.then(
    function fulfilled(){
        return p;
    },
    function rejected(err){
        // 处理`foo(..)`的错误
    }
);
...
```

我们创建了`p`，我们知道我们会为了使用/监听它的拒绝而等待一会儿，所以我们调用`defer()`——如此就不会有全局的报告。`defer()`单纯地返回同一个 promise，为了链接的目的。

从`foo(..)`返回的 promise *当即* 就添附了一个错误处理器，所以这隐含地跳出了默认行为，而且不会有全局的关于错误的报告。

但是从`then(..)`调用返回的 promise 没有`defer()`或添附错误处理器，所以如果它被拒绝（从它内部的任意一个解析处理器中），那么它就会向开发者控制台报告一个未捕获错误。

**这种设计称为成功的深渊**。默认情况下，所有的错误不是被处理就是被报告——这几乎是所有开发者在几乎所有情况下所期望的。你要么不得不注册一个监听器，要么不得不有意什么都不做，并指示你要将错误处理推迟到 *稍后*；你仅为这种特定情况选择承担额外的责任。

这种方式唯一真正的危险是，你`defer()`了一个 Promise 但是实际上没有监听/处理它的拒绝。

但你不得不有意地调用`defer()`来选择进入绝望深渊——默认是成功深渊——所以对于从你自己的错误中拯救你这件事来说，我们能做的不多。

我觉得对于 Promise 的错误处理还有希望（在后 ES6 时代）。我希望上层人物将会重新思考这种情况并考虑选用这种方式。同时，你可以自己实现这种方式（给读者们的挑战练习！），或使用一个 *聪明* 的 Promise 库来为你这么做。

**注意：** 这种错误处理/报告的确切的模型已经在我的 *asynquence* Promise 抽象库中实现，我们会在本书的附录 A 中讨论它。

## Promise 模式

我们已经隐含地看到了使用 Promise 链的顺序模式（这个-然后-这个-然后-那个的流程控制），但是我们还可以在 Promise 的基础上抽象出许多其他种类的异步模式。这些模式用于简化异步流程控制的的表达——它可以使我们的代码更易于推理并且更易于维护——即便是我们程序中最复杂的部分。

有两个这样的模式被直接编码在 ES6 原生的`Promise`实现中，所以我们免费的得到了它们，来作为我们其他模式的构建块儿。

### Promise.all([ .. ])

在一个异步序列（Promise 链）中，在任何给定的时刻都只有一个异步任务在被协调——第 2 步严格地接着第 1 步，而第 3 步严格地接着第 2 步。但要是并发（也叫“并行地”）地去做两个或以上的步骤呢？

用经典的编程术语，一个“门（gate）”是一种等待两个或更多并行/并发任务都执行完再继续的机制。它们完成的顺序无关紧要，只是它们不得不都完成才能让门打开，继而让流程控制通过。

在 Promise API 中，我们称这种模式为`all([ .. ])`。

比方说你想同时发起两个 Ajax 请求，在发起第三个 Ajax 请求发起之前，等待它们都完成，而不管它们的顺序。考虑这段代码：

```js
// `request(..)`是一个兼容 Promise 的 Ajax 工具
// 就像我们在本章早前定义的

var p1 = request( "http://some.url.1/" );
var p2 = request( "http://some.url.2/" );

Promise.all( [p1,p2] )
.then( function(msgs){
    // `p1`和`p2`都已完成，这里将它们的消息传入
    return request(
        "http://some.url.3/?v=" + msgs.join(",")
    );
} )
.then( function(msg){
    console.log( msg );
} );
```

`Promise.all([ .. ])`期待一个单独的参数，一个`array`，一般由 Promise 的实例组成。从`Promise.all([ .. ])`返回的 promise 将会收到完成的消息（在这段代码中是`msgs`），它是一个由所有被传入的 promise 的完成消息按照被传入的顺序构成的`array`（与完成的顺序无关）。

**注意：** 技术上讲，被传入`Promise.all([ .. ])`的`array`的值可以包括 Promise，thenable，甚至是立即值。这个列表中的每一个值都实质上通过`Promise.resolve(..)`来确保它是一个可以被等待的纯粹的 Promise，所以一个立即值将被范化为这个值的一个 Promise。如果这个`array`是空的，主 Promise 将会立即完成。

从`Promise.resolve(..)`返回的主 Promise 将会在所有组成它的 promise 完成之后才会被完成。如果其中任意一个 promise 被拒绝，`Promise.all([ .. ])`的主 Promise 将立即被拒绝，并放弃所有其他 promise 的结果。

要记得总是给每个 promise 添加拒绝/错误处理器，即使和特别是那个从`Promise.all([ .. ])`返回的 promise。

### Promise.race([ .. ])

虽然`Promise.all([ .. ])`并发地协调多个 Promise 并假定它们都需要被完成，但是有时候你只想应答“冲过终点的第一个 Promise”，而让其他的 Promise 被丢弃。

这种模式经典地被称为“闩”，但在 Promise 中它被称为一个“竞合（race）”。

**警告：** 虽然“只有第一个冲过终点的算赢”是一个非常合适被比喻，但不幸的是“竞合（race）”是一个被占用的词，因为“竞合状态（race conditions）”通常被认为是程序中的 Bug（见第一章）。不要把`Promise.race([ .. ])`与“竞合状态（race conditions）”搞混了。

“竞合状态（race conditions）”也期待一个单独的`array`参数，含有一个或多个 Promise，thenable，或立即值。与立即值进行竞合并没有多大实际意义，因为很明显列表中的第一个会胜出——就像赛跑时有一个选手在终点线上起跑！

和`Promise.all([ .. ])`相似，`Promise.race([ .. ])`将会在任意一个 Promise 解析为完成时完成，而且它会在任意一个 Promise 解析为拒绝时拒绝。

**注意：** 一个“竞合（race）”需要至少一个“选手”，所以如果你传入一个空的`array`，`race([..])`的主 Promise 将不会立即解析，反而是永远不会被解析。这是砸自己的脚！ES6 应当将它规范为要么完成，要么拒绝，或者要么抛出某种同步错误。不幸的是，因为在 ES6 的`Promise`之前的 Promise 库的优先权高，他们不得不把这个坑留在这儿，所以要小心绝不要传入一个空`array`。

让我们重温刚才的并发 Ajax 的例子，但是在`p1`和`p2`竞合的环境下：

```js
// `request(..)`是一个兼容 Promise 的 Ajax 工具
// 就像我们在本章早前定义的

var p1 = request( "http://some.url.1/" );
var p2 = request( "http://some.url.2/" );

Promise.race( [p1,p2] )
.then( function(msg){
    // `p1`或`p2`会赢得竞合
    return request(
        "http://some.url.3/?v=" + msg
    );
} )
.then( function(msg){
    console.log( msg );
} );
```

因为只有一个 Promise 会胜出，所以完成的值是一个单独的消息，而不是一个像`Promise.all([ .. ])`中那样的`array`。

#### 超时竞合

我们早先看过这个例子，描述`Promise.race([ .. ])`如何能够用于表达“promise 超时”模式：

```js
// `foo()`是一个兼容 Promise

// `timeoutPromise(..)`在早前定义过，
// 返回一个在指定延迟之后会被拒绝的 Promise

// 为`foo()`设置一个超时
Promise.race( [
    foo(),                    // 尝试`foo()`
    timeoutPromise( 3000 )    // 给它 3 秒钟
] )
.then(
    function(){
        // `foo(..)`及时地完成了！
    },
    function(err){
        // `foo()`要么是被拒绝了，要么就是没有及时完成
        // 可以考察`err`来知道是哪一个原因
    }
);
```

这种超时模式在绝大多数情况下工作的很好。但这里有一些微妙的细节要考虑，而且坦率的说它们对于`Promise.race([ .. ])`和`Promise.all([ .. ])`都同样需要考虑。

#### "Finally"

要问的关键问题是，“那些被丢弃/忽略的 promise 发生了什么？”我们不是从性能的角度在问这个问题——它们通常最终会变成垃圾回收的合法对象——而是从行为的角度（副作用等等）。Promise 不能被取消——而且不应当被取消，因为那会摧毁本章稍后的“Promise 不可取消”一节中要讨论的外部不可变性——所以它们只能被无声地忽略。

但如果前面例子中的`foo()`占用了某些资源，但超时首先触发而且导致这个 promise 被忽略了呢？这种模式中存在某种东西可以在超时后主动释放被占用的资源，或者取消任何它可能带来的副作用吗？要是你想做的全部只是记录下`foo()`超时的事实呢？

一些开发者提议，Promise 需要一个`finally(..)`回调注册机制，它总是在 Promise 解析时被调用，而且允许你制定任何可能的清理操作。在当前的语言规范中它还不存在，但它可能会在 ES7+中加入。我们不得不边走边看了。

它看起来可能是这样：

```js
var p = Promise.resolve( 42 );

p.then( something )
.finally( cleanup )
.then( another )
.finally( cleanup );
```

**注意：** 在各种 Promise 库中，`finally(..)`依然会创建并返回一个新的 Promise（为了使链条延续下去）。如果`cleanup(..)`函数返回一个 Promise，它将会链入链条，这意味着你可能还有我们刚才讨论的未处理拒绝的问题。

同时，我们可以制造一个静态的帮助工具来让我们观察（但不干涉）Promise 的解析：

```js
// 填补的安全检查
if (!Promise.observe) {
    Promise.observe = function(pr,cb) {
        // 从侧面观察`pr`的解析
        pr.then(
            function fulfilled(msg){
                // 异步安排回调（作为 Job）
                Promise.resolve( msg ).then( cb );
            },
            function rejected(err){
                // 异步安排回调（作为 Job）
                Promise.resolve( err ).then( cb );
            }
        );

        // 返回原本的 promise
        return pr;
    };
}
```

这是我们在前面的超时例子中如何使用它：

```js
Promise.race( [
    Promise.observe(
        foo(),                    // 尝试`foo()`
        function cleanup(msg){
            // 在`foo()`之后进行清理，即便它没有及时完成
        }
    ),
    timeoutPromise( 3000 )    // 给它 3 秒钟
] )
```

这个`Promise.observe(..)`帮助工具只是描述你如何在不干扰 Promise 的情况下观测它的完成。其他的 Promise 库有他们自己的解决方案。不论你怎么做，你都将很可能有个地方想用来确认你的 Promise 没有意外地被无声地忽略掉。

### Variations on all([ .. ]) and race([ .. ])

原生的 ES6Promise 带有内建的`Promise.all([ .. ])`和`Promise.race([ .. ])`，这里还有几个关于这些语义的其他常用的变种模式：

*   `none([ .. ])`很像`all([ .. ])`，但是完成和拒绝被转置了。所有的 Promise 都需要被拒绝——拒绝变成了完成值，反之亦然。
*   `any([ .. ])`很像`all([ .. ])`，但它忽略任何拒绝，所以只有一个需要完成即可，而不是它们所有的。
*   `first([ .. ])`像是一个带有`any([ .. ])`的竞合，它忽略任何拒绝，而且一旦有一个 Promise 完成时，它就立即完成。
*   `last([ .. ])`很像`first([ .. ])`，但是只有最后一个完成胜出。

某些 Promise 抽象工具库提供这些方法，但你也可以用 Promise 机制的`race([ .. ])`和`all([ .. ])`，自己定义他们。

比如，这是我们如何定义`first([..])`:

```js
// 填补的安全检查
if (!Promise.first) {
    Promise.first = function(prs) {
        return new Promise( function(resolve,reject){
            // 迭代所有的 promise
            prs.forEach( function(pr){
                // 泛化它的值
                Promise.resolve( pr )
                // 无论哪一个首先成功完成，都由它来解析主 promise
                .then( resolve );
            } );
        } );
    };
}
```

**注意：** 这个`first(..)`的实现不会在它所有的 promise 都被拒绝时拒绝；它会简单地挂起，很像`Promise.race([])`。如果需要，你可以添加一些附加逻辑来追踪每个 promise 的拒绝，而且如果所有的都被拒绝，就在主 promise 上调用`reject()`。我们将此作为练习留给读者。

### 并发迭代

有时候你想迭代一个 Promise 的列表，并对它们所有都实施一些任务，就像你可以对同步的`array`做的那样（比如，`forEach(..)`，`map(..)`，`some(..)`，和`every(..)`）。如果对每个 Promise 实施的操作根本上是同步的，它们工作的很好，正如我们在前面的代码段中用过的`forEach(..)`。

但如果任务在根本上是异步的，或者可以/应当并发地实施，你可以使用许多库提供的异步版本的这些工具方法。

比如，让我们考虑一个异步的`map(..)`工具，它接收一个`array`值（可以是 Promise 或任何东西），外加一个对数组中每一个值实施的函数（任务）。`map(..)`本身返回一个 promise，它的完成值是一个持有每个任务的异步完成值的`array`（以与映射（mapping）相同的顺序）：

```js
if (!Promise.map) {
    Promise.map = function(vals,cb) {
        // 一个等待所有被映射的 promise 的新 promise
        return Promise.all(
            // 注意：普通的数组`map(..)`，
            // 将值的数组变为 promise 的数组
            vals.map( function(val){
                // 将`val`替换为一个在`val`
                // 异步映射完成后才解析的新 promise
                return new Promise( function(resolve){
                    cb( val, resolve );
                } );
            } )
        );
    };
}
```

**注意：** 在这种`map(..)`的实现中，你无法表示异步拒绝，但如果一个在映射的回调内部发生一个同步的异常/错误，那么`Promise.map(..)`返回的主 Promise 就会拒绝。

让我们描绘一下对一组 Promise（不是简单的值）使用`map(..)`：

```js
var p1 = Promise.resolve( 21 );
var p2 = Promise.resolve( 42 );
var p3 = Promise.reject( "Oops" );

// 将列表中的值翻倍，即便它们在 Promise 中
Promise.map( [p1,p2,p3], function(pr,done){
    // 确保列表中每一个值都是 Promise
    Promise.resolve( pr )
    .then(
        // 将值作为`v`抽取出来
        function(v){
            // 将完成的`v`映射到新的值
            done( v * 2 );
        },
        // 或者，映射到 promise 的拒绝消息上
        done
    );
} )
.then( function(vals){
    console.log( vals );    // [42,84,"Oops"]
} );
```

## Promise API 概览

让我们复习一下我们已经在本章中零散地展开的 ES6`Promise`API。

**注意：** 下面的 API 尽在 ES6 中是原生的，但也存在一些语言规范兼容的填补（不光是扩展 Promise 库），它们定义了`Promise`和与之相关的所有行为，所以即使是在前 ES6 时代的浏览器中你也以使用原生的 Promise。这类填补的其中之一是“Native Promise Only”（[`github.com/getify/native-promise-only），我写的！`](http://github.com/getify/native-promise-only%EF%BC%89%EF%BC%8C%E6%88%91%E5%86%99%E7%9A%84%EF%BC%81)

### new Promise(..)构造器

*揭示构造器（revealing constructor）* `Promise(..)`必须与`new`一起使用，而且必须提供一个被同步/立即调用的回调函数。这个函数被传入两个回调函数，它们作为 promise 的解析能力。我们通常将它们标识为`resolve(..)`和`reject(..)`：

```js
var p = new Promise( function(resolve,reject){
    // `resolve(..)`给解析/完成的 promise
    // `reject(..)`给拒绝的 promise
} );
```

`reject(..)`简单地拒绝 promise，但是`resolve(..)`既可以完成 promise，也可以拒绝 promise，这要看它被传入什么值。如果`resolve(..)`被传入一个立即的，非 Promise，非 thenable 的值，那么这个 promise 将用这个值完成。

但如果`resolve(..)`被传入一个 Promise 或者 thenable 的值，那么这个值将被递归地展开，而且无论它最终解析结果/状态是什么，都将被 promise 采用。

### Promise.resolve(..) 和 Promise.reject(..)

一个用于创建已被拒绝的 Promise 的简便方法是`Promise.reject(..)`，所以这两个 promise 是等价的：

```js
var p1 = new Promise( function(resolve,reject){
    reject( "Oops" );
} );

var p2 = Promise.reject( "Oops" );
```

与`Promise.reject(..)`相似，`Promise.resolve(..)`通常用来创建一个已完成的 Promise。然而，`Promise.resolve(..)`还会展开 thenale 值（就像我们已经几次讨论过的）。在这种情况下，返回的 Promise 将会采用你传入的 thenable 的解析，它既可能是完成，也可能是拒绝：

```js
var fulfilledTh = {
    then: function(cb) { cb( 42 ); }
};
var rejectedTh = {
    then: function(cb,errCb) {
        errCb( "Oops" );
    }
};

var p1 = Promise.resolve( fulfilledTh );
var p2 = Promise.resolve( rejectedTh );

// `p1`将是一个完成的 promise
// `p2`将是一个拒绝的 promise
```

而且要记住，如果你传入一个纯粹的 Promise，`Promise.resolve(..)`不会做任何事情；它仅仅会直接返回这个值。所以在你不知道其本性的值上调用`Promise.resolve(..)`不会有额外的开销，如果它偶然已经是一个纯粹的 Promise。

### then(..) 和 catch(..)

每个 Promise 实例（**不是** `Promise` API 名称空间）都有`then(..)`和`catch(..)`方法，它们允许你为 Promise 注册成功或拒绝处理器。一旦 Promise 被解析，它们中的一个就会被调用，但不是都会被调用，而且它们总是会被异步地调用（参见第一章的“Jobs”）。

`then(..)`接收两个参数，第一个用于完成回调，第二个用户拒绝回调。如果它们其中之一被省略，或者被传入一个非函数的值，那么一个默认的回调就会分别顶替上来。默认的完成回调简单地将值向下传递，而默认的拒绝回调简单地重新抛出（传播）收到的拒绝理由。

`catch(..)`仅仅接收一个拒绝回调作为参数，而且会自动的顶替一个默认的成功回调，就像我们讨论过的。换句话说，它等价于`then(null,..)`：

```js
p.then( fulfilled );

p.then( fulfilled, rejected );

p.catch( rejected ); // 或者`p.then( null, rejected )`
```

`then(..)`和`catch(..)`也会创建并返回一个新的 promise，它可以用来表达 Promise 链式流程控制。如果完成或拒绝回调有异常被抛出，这个返回的 promise 就会被拒绝。如果这两个回调之一返回一个立即，非 Promise，非 thenable 值，那么这个值就会作为被返回的 promise 的完成。如果完成处理器指定地返回一个 promise 或 thenable 值这个值就会被展开而且变成被返回的 promise 的解析。

### Promise.all([ .. ]) 和 Promise.race([ .. ])

在 ES6 的`Promise`API 的静态帮助方法`Promise.all([ .. ])`和`Promise.race([ .. ])`都创建一个 Promise 作为它们的返回值。这个 promise 的解析完全由你传入的 promise 数组控制。

对于`Promise.all([ .. ])`，为了被返回的 promise 完成，所有你传入的 promise 都必须完成。如果其中任意一个被拒绝，返回的主 promise 也会立即被拒绝（丢弃其他所有 promise 的结果）。至于完成状态，你会收到一个含有所有被传入的 promise 的完成值的`array`。至于拒绝状态，你仅会收到第一个 promise 拒绝的理由值。这种模式通常称为“门”：在门打开前所有人都必须到达。

对于`Promise.race([ .. ])`，只有第一个解析（成功或拒绝）的 promise 会“胜出”，而且不论解析的结果是什么，都会成为被返回的 promise 的解析结果。这种模式通常成为“闩”：第一个打开门闩的人才能进来。考虑这段代码：

```js
var p1 = Promise.resolve( 42 );
var p2 = Promise.resolve( "Hello World" );
var p3 = Promise.reject( "Oops" );

Promise.race( [p1,p2,p3] )
.then( function(msg){
    console.log( msg );        // 42
} );

Promise.all( [p1,p2,p3] )
.catch( function(err){
    console.error( err );    // "Oops"
} );

Promise.all( [p1,p2] )
.then( function(msgs){
    console.log( msgs );    // [42,"Hello World"]
} );
```

**警告：** 要小心！如果一个空的`array`被传入`Promise.all([ .. ])`，它会立即完成，但`Promise.race([ .. ])`却会永远挂起，永远不会解析。

ES6 的`Promise`API 十分简单和直接。对服务于大多数基本的异步情况来说它足够好了，而且当你要把你的代码从回调地狱变为某些更好的东西时，它是一个开始的好地方。

但是依然还有许多应用程序所要求的精巧的异步处理，由于 Promise 本身所受的限制而不能解决。在下一节中，我们将深入这些限制，来看看 Promise 库的优点。

## Promise 限制

本节中我们将要讨论的许多细节已经在这一章中被提及了，但我们将明确地复习这些限制。

### 顺序的错误处理

我们在本章前面的部分详细讲解了 Promise 风格的错误处理。Promise 的设计方式——特别是他们如何链接——所产生的限制，创建了一个非常容易掉进去的陷阱，Promise 链中的错误会被意外地无声地忽略掉。

但关于 Promise 的错误还有一些其他事情要考虑。因为 Promise 链只不过是组成它的 Promise 连在一起，没有一个实体可以用来将整个链条表达为一个单独的 *东西*，这意味着没有外部的方法能够监听可能发生的任何错误。

如果你构建一个不包含错误处理器的 Promise 链，这个链条的任意位置发生的任何错误都将沿着链条向下无限传播，直到被监听为止（通过在某一步上注册拒绝处理器）。所以，在这种特定情况下，拥有链条的最后一个 promise 的引用就够了（下面代码段中的`p`），因为你可以在这里注册拒绝处理器，而且它会被所有传播的错误通知：

```js
// `foo(..)`, `STEP2(..)` 和 `STEP3(..)`
// 都是 promise 兼容的工具

var p = foo( 42 )
.then( STEP2 )
.then( STEP3 );
```

虽然这看起来有点儿小糊涂，但是这里的`p`没有指向链条中的第一个 promise（`foo(42)`调用中来的那一个），而是指向了最后一个 promise，来自于`then(STEP3)`调用的那一个。

另外，这个 promise 链条上看不到一个步骤做了自己的错误处理。这意味着你可以在`p`上注册一个拒绝处理器，如果在链条的任意位置发生了错误，它就会被通知。

```js
p.catch( handleErrors );
```

但如果这个链条中的某一步事实上做了自己的错误处理（也许是隐藏/抽象出去了，所以你看不到），那么你的`handleErrors(..)`就不会被通知。这可能是你想要的——它毕竟是一个“被处理过的拒绝”——但它也可能 *不* 是你想要的。完全缺乏被通知的能力（被“已处理过的”拒绝错误通知）是一个在某些用法中约束功能的一种限制。

它基本上和`try..catch`中存在的限制是相同的，它可以捕获一个异常并简单地吞掉。所以这不是一个 **Promise 特有** 的问题，但它确实是一个我们希望绕过的限制。

不幸的是，许多时候 Promise 链序列的中间步骤不会被留下引用，所以没有这些引用，你就不能添加错误处理器来可靠地监听错误。

### 单独的值

根据定义，Promise 只能有一个单独的完成值或一个单独的拒绝理由。在简单的例子中，这没什么大不了的，但在更精巧的场景下，你可能发现这个限制。

典型的建议是构建一个包装值（比如`object`或`array`）来包含这些多个消息。这个方法好用，但是在你的 Promise 链的每一步上把消息包装再拆开显得十分尴尬和烦人。

#### 分割值

有时你可以将这种情况当做一个信号，表示你可以/应当将问题拆分为两个或更多的 Promise。

想象你有一个工具`foo(..)`，它异步地产生两个值（`x`和`y`）：

```js
function getY(x) {
    return new Promise( function(resolve,reject){
        setTimeout( function(){
            resolve( (3 * x) - 1 );
        }, 100 );
    } );
}

function foo(bar,baz) {
    var x = bar * baz;

    return getY( x )
    .then( function(y){
        // 将两个值包装近一个容器
        return [x,y];
    } );
}

foo( 10, 20 )
.then( function(msgs){
    var x = msgs[0];
    var y = msgs[1];

    console.log( x, y );    // 200 599
} );
```

首先，让我们重新安排一下`foo(..)`返回的东西，以便于我们不必再将`x`和`y`包装进一个单独的`array`值中来传送给一个 Promise。相反，我们将每一个值包装进它自己的 promise：

```js
function foo(bar,baz) {
    var x = bar * baz;

    // 将两个 promise 返回
    return [
        Promise.resolve( x ),
        getY( x )
    ];
}

Promise.all(
    foo( 10, 20 )
)
.then( function(msgs){
    var x = msgs[0];
    var y = msgs[1];

    console.log( x, y );
} );
```

一个 promise 的`array`真的要比传递给一个单独的 Promise 的值的`array`要好吗？语法上，它没有太多改进。

但是这种方式更加接近于 Promise 的设计原理。现在它更易于在未来将`x`与`y`的计算分开，重构进两个分离的函数中。它更清晰，也允许调用端代码更灵活地安排这两个 promise——这里使用了`Promise.all([ .. ])`，但它当然不是唯一的选择——而不是将这样的细节在`foo(..)`内部进行抽象。

#### 展开/散开参数

`var x = ..`和`var y = ..`的赋值依然是一个尴尬的负担。我们可以在一个帮助工具中利用一些函数式技巧（向 Reginald Braithwaite 致敬，在推特上 @raganwald ）：

```js
function spread(fn) {
    return Function.apply.bind( fn, null );
}

Promise.all(
    foo( 10, 20 )
)
.then(
    spread( function(x,y){
        console.log( x, y );    // 200 599
    } )
)
```

看起来好些了！当然，你可以内联这个函数式魔法来避免额外的帮助函数：

```js
Promise.all(
    foo( 10, 20 )
)
.then( Function.apply.bind(
    function(x,y){
        console.log( x, y );    // 200 599
    },
    null
) );
```

这个技巧可能很整洁，但是 ES6 给了我们一个更好的答案：解构（destructuring）。数组的解构赋值形式看起来像这样：

```js
Promise.all(
    foo( 10, 20 )
)
.then( function(msgs){
    var [x,y] = msgs;

    console.log( x, y );    // 200 599
} );
```

最棒的是，ES6 提供了数组参数解构形式：

```js
Promise.all(
    foo( 10, 20 )
)
.then( function([x,y]){
    console.log( x, y );    // 200 599
} );
```

我们现在已经接受了“每个 Promise 一个值”的准则，继续让我们把模板代码最小化！

**注意：** 更多关于 ES6 解构形式的信息，参阅本系列的 *ES6 与未来*。

### 单次解析

Promise 的一个最固有的行为之一就是，一个 Promise 只能被解析一次（成功或拒绝）。对于多数异步用例来说，你仅仅取用这个值一次，所以这工作的很好。

但也有许多异步情况适用于一个不同的模型——更类似于事件和/或数据流。表面上看不清 Promise 能对这种用例适应的多好，如果能的话。没有基于 Promise 的重大抽象过程，它们完全缺乏对多个值解析的处理。

想象这样一个场景，你可能想要为响应一个刺激（比如事件）触发一系列异步处理步骤，而这实际上将会发生多次，比如按钮点击。

这可能不会像你想的那样工作：

```js
// `click(..)` 绑定了一个 DOM 元素的 `"click"` 事件
// `request(..)` 是先前定义的支持 Promise 的 Ajax

var p = new Promise( function(resolve,reject){
    click( "#mybtn", resolve );
} );

p.then( function(evt){
    var btnID = evt.currentTarget.id;
    return request( "http://some.url.1/?id=" + btnID );
} )
.then( function(text){
    console.log( text );
} );
```

这里的行为仅能在你的应用程序只让按钮被点击一次的情况下工作。如果按钮被点击第二次，promise`p`已经被解析了，所以第二个`resolve(..)`将被忽略。

相反的，你可能需要将模式反过来，在每次事件触发时创建一个全新的 Promise 链：

```js
click( "#mybtn", function(evt){
    var btnID = evt.currentTarget.id;

    request( "http://some.url.1/?id=" + btnID )
    .then( function(text){
        console.log( text );
    } );
} );
```

这种方式会 *好用*，为每个按钮上的`"click"`事件发起一个全新的 Promise 序列。

但是除了在事件处理器内部定义一整套 Promise 链看起来很丑以外，这样的设计在某种意义上违背了关注/能力分离原则（SoC）。你可能非常想在一个你的代码不同的地方定义事件处理器：你定义对事件的 *响应*（Promise 链）的地方。如果没有帮助机制，在这种模式下这么做很尴尬。

**注意：** 这种限制的另一种表述方法是，如果我们能够构建某种能在它上面进行 Promise 链监听的“可监听对象（observable）”就好了。有一些库已经建立这些抽象（比如 RxJS——[`rxjs.codeplex.com/），但是这种抽象看起来是如此的重，以至于你甚至再也看不到 Promise 的性质。这样的重抽象带来一个重要的问题：这些机制是否像 Promise 本身被设计的一样`](http://rxjs.codeplex.com/%EF%BC%89%EF%BC%8C%E4%BD%86%E6%98%AF%E8%BF%99%E7%A7%8D%E6%8A%BD%E8%B1%A1%E7%9C%8B%E8%B5%B7%E6%9D%A5%E6%98%AF%E5%A6%82%E6%AD%A4%E7%9A%84%E9%87%8D%EF%BC%8C%E4%BB%A5%E8%87%B3%E4%BA%8E%E4%BD%A0%E7%94%9A%E8%87%B3%E5%86%8D%E4%B9%9F%E7%9C%8B%E4%B8%8D%E5%88%B0Promise%E7%9A%84%E6%80%A7%E8%B4%A8%E3%80%82%E8%BF%99%E6%A0%B7%E7%9A%84%E9%87%8D%E6%8A%BD%E8%B1%A1%E5%B8%A6%E6%9D%A5%E4%B8%80%E4%B8%AA%E9%87%8D%E8%A6%81%E7%9A%84%E9%97%AE%E9%A2%98%EF%BC%9A%E8%BF%99%E4%BA%9B%E6%9C%BA%E5%88%B6%E6%98%AF%E5%90%A6%E5%83%8FPromise%E6%9C%AC%E8%BA%AB%E8%A2%AB%E8%AE%BE%E8%AE%A1%E7%9A%84%E4%B8%80%E6%A0%B7) *可靠*。我们将会在附录 B 中重新讨论“观察者（Observable）”模式。

### 惰性

对于在你的代码中使用 Promise 而言一个实在的壁垒是，现存的所有代码都没有支持 Promise。如果你有许多基于回调的代码，让代码保持相同的风格容易多了。

“一段基于动作（用回调）的代码将仍然基于动作（用回调），除非一个更聪明，具有 Promise 意识的开发者对它采取行动。”

Promise 提供了一种不同的模式规范，如此，代码的表达方式可能会变得有一点儿不同，某些情况下，则根不同。你不得不有意这么做，因为 Promise 不仅只是把那些为你服务至今的老式编码方法自然地抖落掉。

考虑一个像这样的基于回调的场景：

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

将这个基于回调的代码转换为支持 Promise 的代码的第一步该怎么做，是立即明确的吗？这要看你的经验。你练习的越多，它就感觉越自然。但当然，Promise 没有明确告知到底怎么做——没有一个放之四海而皆准的答案——所以这要靠你的责任心。

就像我们以前讲过的，我们绝对需要一种支持 Promise 的 Ajax 工具来取代基于回调的工具，我们可以称它为`request(..)`。你可以制造自己的，正如我们已经做过的。但是不得不为每个基于回调的工具手动定义 Promise 相关的包装器的负担，使得你根本就不太可能选择将代码重构为 Promise 相关的。

Promise 没有为这种限制提供直接的答案。但是大多数 Promise 库确实提供了帮助函数。想象一个这样的帮助函数：

```js
// 填补的安全检查
if (!Promise.wrap) {
    Promise.wrap = function(fn) {
        return function() {
            var args = [].slice.call( arguments );

            return new Promise( function(resolve,reject){
                fn.apply(
                    null,
                    args.concat( function(err,v){
                        if (err) {
                            reject( err );
                        }
                        else {
                            resolve( v );
                        }
                    } )
                );
            } );
        };
    };
}
```

好吧，这可不是一个微不足道的工具。然而，虽然他可能看起来有点儿令人生畏，但也没有你想的那么糟。它接收一个函数，这个函数期望一个错误优先风格的回调作为第一个参数，然后返回一个可以自动创建 Promise 并返回的新函数，然后为你替换掉回调，与 Promise 的完成/拒绝连接在一起。

与其浪费太多时间谈论这个`Promise.wrap(..)`帮助函数 *如何* 工作，还不如让我们来看看如何使用它：

```js
var request = Promise.wrap( ajax );

request( "http://some.url.1/" )
.then( .. )
..
```

哇哦，真简单！

`Promise.wrap(..)` **不会** 生产 Promise。它生产一个将会生产 Promise 的函数。某种意义上，一个 Promise 生产函数可以被看做一个“Promise 工厂”。我提议将这样的东西命名为“promisory”（"Promise" + "factory"）。

这种将期望回调的函数包装为一个 Promise 相关的函数的行为，有时被称为“提升（lifting）”或“promise 化（promisifying）”。但是除了“提升过的函数”以外，看起来没有一个标准的名词来称呼这个结果函数，所以我更喜欢“promisory”，因为我认为他更具描述性。

**注意：** Promisory 不是一个瞎编的词。它是一个真实存在的词汇，而且它的定义是含有或载有一个 promise。这正是这些函数所做的，所以这个术语匹配得简直完美！

那么，`Promise.wrap(ajax)`生产了一个我们称为`request(..)`的`ajax(..)`promisory，而这个 promisory 为 Ajax 应答生产 Promise。

如果所有的函数已经都是 promisory，我们就不需要自己制造它们，所以额外的步骤就有点儿多余。但是至少包装模式是（通常都是）可重复的，所以我们可以把它放进`Promise.wrap(..)`帮助函数中来支援我们的 promise 编码。

那么回到刚才的例子，我们需要为`ajax(..)`和`foo(..)`都做一个 promisory。

```js
// 为`ajax(..)`制造一个 promisory
var request = Promise.wrap( ajax );

// 重构`foo(..)`，但是为了代码其他部分
// 的兼容性暂且保持它对外是基于回调的
// ——仅在内部使用`request(..)`'的 promise
function foo(x,y,cb) {
    request(
        "http://some.url.1/?x=" + x + "&y=" + y
    )
    .then(
        function fulfilled(text){
            cb( null, text );
        },
        cb
    );
}

// 现在，为了这段代码本来的目的，为`foo(..)`制造一个 promisory
var betterFoo = Promise.wrap( foo );

// 并使用这个 promisory
betterFoo( 11, 31 )
.then(
    function fulfilled(text){
        console.log( text );
    },
    function rejected(err){
        console.error( err );
    }
);
```

当然，虽然我们将`foo(..)`重构为使用我们的新`request(..)`promisory，我们可以将`foo(..)`本身制成 promisory，而不是保留基于会掉的实现并需要制造和使用后续的`betterFoo(..)`promisory。这个决定只是要看`foo(..)`是否需要保持基于回调的形式以便于代码的其他部分兼容。

考虑这段代码：

```js
// 现在，`foo(..)`也是一个 promisory
// 因为它委托到`request(..)` promisory
function foo(x,y) {
    return request(
        "http://some.url.1/?x=" + x + "&y=" + y
    );
}

foo( 11, 31 )
.then( .. )
..
```

虽然 ES6 的 Promise 没有为这样的 promisory 包装提供原生的帮助函数，但是大多数库提供它们，或者你可以制造自己的。不管哪种方法，这种 Promise 特定的限制是可以不费太多劲儿就可以解决的（当然是和回调地狱的痛苦相比！）。

### Promise 不可撤销

一旦你创建了一个 Promise 并给它注册了一个完成和/或拒绝处理器，就没有什么你可以从外部做的事情能停止这个进程，即使是某些其他的事情使这个任务变得毫无意义。

**注意：** 许多 Promise 抽象库都提供取消 Promise 的功能，但这是一个非常坏的主意！许多开发者都希望 Promise 被原生地设计为具有外部取消能力，但问题是这将允许 Promise 的一个消费者/监听器影响某些其他消费者监听同一个 Promise 的能力。这违反了未来值得可靠性原则（外部不可变），另外就是嵌入了“远距离行为（action at a distance）”的反模式（[`en.wikipedia.org/wiki/Action_at_a_distance_%28computer_programming%29）。不管它看起来多么有用，它实际上会直接将你引回与回调地狱相同的噩梦。`](http://en.wikipedia.org/wiki/Action_at_a_distance_(computer_programming)%EF%BC%89%E3%80%82%E4%B8%8D%E7%AE%A1%E5%AE%83%E7%9C%8B%E8%B5%B7%E6%9D%A5%E5%A4%9A%E4%B9%88%E6%9C%89%E7%94%A8%EF%BC%8C%E5%AE%83%E5%AE%9E%E9%99%85%E4%B8%8A%E4%BC%9A%E7%9B%B4%E6%8E%A5%E5%B0%86%E4%BD%A0%E5%BC%95%E5%9B%9E%E4%B8%8E%E5%9B%9E%E8%B0%83%E5%9C%B0%E7%8B%B1%E7%9B%B8%E5%90%8C%E7%9A%84%E5%99%A9%E6%A2%A6%E3%80%82)

考虑我们早先的 Promise 超时场景：

```js
var p = foo( 42 );

Promise.race( [
    p,
    timeoutPromise( 3000 )
] )
.then(
    doSomething,
    handleError
);

p.then( function(){
    // 即使是在超时的情况下也会发生 :(
} );
```

“超时”对于 promise`p`来说是外部的，所以`p`本身继续运行，这可能不是我们想要的。

一个选项是侵入性地定义你的解析回调：

```js
var OK = true;

var p = foo( 42 );

Promise.race( [
    p,
    timeoutPromise( 3000 )
    .catch( function(err){
        OK = false;
        throw err;
    } )
] )
.then(
    doSomething,
    handleError
);

p.then( function(){
    if (OK) {
        // only happens if no timeout! :)
    }
} );
```

这很丑。这可以工作，但是远不理想。一般来说，你应当避免这样的场景。

但是如果你不能，这种解决方案的丑陋应当是一个线索，说明 *取消* 是一种属于在 Promise 之上的更高层抽象的功能。我推荐你找一个 Promise 抽象库来辅助你，而不是自己使用黑科技。

**注意：** 我的 *asynquence* Promise 抽象库提供了这样的抽象，还为序列提供了一个`abort()`能力，这一切将在附录 A 中讨论。

一个单独的 Promise 不是真正的流程控制机制（至少没有多大实际意义），而流程控制机制正是 *取消* 要表达的；这就是为什么 Promise 取消显得尴尬。

相比之下，一个链条的 Promise 集合在一起——我称之为“序列”—— *是* 一个流程控制的表达，如此在这一层面的抽象上它就适于定义取消。

没有一个单独的 Promise 应该是可以取消的，但是一个 *序列* 可以取消是有道理的，因为你不会将一个序列作为一个不可变值传来传去，就像 Promise 那样。

### Promise 性能

这种限制既简单又复杂。

比较一下在基于回调的异步任务链和 Promise 链上有多少东西在动，很明显 Promise 有多得多的事情发生，这意味着它们自然地会更慢一点点。回想一下 Promise 提供的保证信任的简单列表，将它和你为了达到相同保护效果而在回调上面添加的特殊代码比较一下。

更多工作要做，更多的安全要保护，意味着 Promise 与赤裸裸的，不可靠的回调相比 *确实* 更慢。这些都很明显，可能很容易萦绕在你脑海中。

但是慢多少？好吧……这实际上是一个难到不可思议的问题，无法绝对，全面地回答。

坦白地说，这是一个比较苹果和橘子的问题，所以可能是问错了。你实际上应当比较的是，带有所有手动保护层的经过特殊处理的回调系统，是否比一个 Promise 实现要快。

如果说 Promise 有一种合理的性能限制，那就是它并不将可靠性保护的选项罗列出来让你选择——你总是一下得到全部。

如果我们承认 Promise 一般来说要比它的非 Promise，不可靠的回调等价物 *慢一点儿*——假定在有些地方你觉得你可以自己调整可靠性的缺失——难道这意味着 Promise 应当被全面地避免，就好像你的整个应用程序仅仅由一些可能的“必须绝对最快”的代码驱动着？

合理性检查：如果你的代码有那么合理，那么 **对于这样的任务，JavaScript 是正确的选择吗？** 为了运行应用程序 JavaScript 可以被优化得十分高效（参见第五章和第六章）。但是在 Promise 提供的所有好处的光辉之下，过于沉迷它微小的性能权衡，*真的* 合适吗？

另一个微妙的问题是 Promise 使 *所有事情* 都成为异步的，这意味着有些应当立即完成的（同步的）步骤也要推迟到下一个 Job 步骤中（参见第一章）。也就是说一个 Promise 任务序列要比使用回调连接的相同序列要完成的稍微慢一些是可能的。

当然，这里的问题是：这些关于性能的微小零头的潜在疏忽，和我们在本章通篇阐述的 Promise 带来的益处相比，*还值得考虑吗？*

我的观点是，在几乎所有你可能认为 Promise 的性能慢到了需要被考虑的情况下，完全回避 Promise 并将它的可靠性和组合性优化掉，实际上一种反模式。

相反地，你应当默认地在代码中广泛使用它们，然后再记录并分析你的应用程序的热（关键）路径。Promise *真的* 是瓶颈？还是它们只是理论上慢了下来？只有在那 *之后*，拿着实际合法的基准分析观测数据（参见第六章），再将 Promise 从这些关键区域中重构移除才称得上是合理与谨慎。

Promise 是有一点儿慢，但作为交换你得到了很多内建的可靠性，无 Zalgo 的可预测性，与组合性。也许真正的限制不是它们的性能，而是你对它们的益处缺乏认识？

## 复习

Promise 很牛。用它们。它们解决了肆虐在回调代码中的 *控制倒转* 问题。

它们没有摆脱回调，而是重新定向了这些回调的组织安排方式，是它成为一种坐落于我们和其他工具之间的可靠的中间机制。

Promise 链还开始以顺序的风格定义了一种更好的（当然，还不完美）表达异步流程的方式，它帮我们的大脑更好的规划和维护异步 JS 代码。我们会在下一章中看到一个更好的解决 *这个* 问题的方法！