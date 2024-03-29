# 第四章 函数

# 函数

熟练运用函数是 JavaScript 程序员的必备技能，因为在 JavaScript 中函数实在是太常用了。它能够完成的任务种类非常之多，而在其他语言中则需要很多特殊的语法支持才能达到这种能力。

在本章将会介绍在 JavaScript 中定义函数的多种方式，包括函数表达式和函数声明、以及局部作用域和变量声明提前的工作原理。然后会介绍一些有用的模式，帮助你设计 API（为你的函数提供更好的接口）、搭建代码架构（使用尽可能少的全局对象）、并优化性能（避免不必要的操作）。

现在让我们来一起揭秘 JavaScript 函数，我们首先从一些背景知识开始说起。

## 背景知识

JavaScript 的函数具有两个主要特性，正是这两个特性让它们与众不同。第一个特性是，函数是一等对象（first-class object），第二个是函数提供作用域支持。

函数是对象，那么：

*   可以在程序执行时动态创建函数
*   可以将函数赋值给变量，可以将函数的引用拷贝至另一个变量，可以扩充函数，除了某些特殊场景外均可被删除。
*   可以将函数作为参数传入另一个函数，也可以被当作返回值返回。
*   函数可以包含自己的属性和方法

对于一个函数 A 来说，首先它是对象，拥有属性和方法，其中某个属性碰巧是另一个函数 B，B 可以接受函数作为参数，假设这个函数参数为 C，当执行 B 的时候，返回另一个函数 D。乍一看这里有一大堆相互关联的函数。当你开始习惯函数的许多用法时，你会惊叹原来函数是如此强大、灵活并富有表现力。通常说来，一说到 JavaScript 的函数，我们首先认为它是对象，它具有一个可以“执行”的特性，也就是说我们可以“调用”这个函数。

我们通过 new Function()构造器来生成一个函数，这时可以明显看出函数是对象：

```js
// antipattern
// for demo purposes only
var add = new Function('a, b', 'return a + b');
add(1, 2); // returns 3 
```

在这段代码中，毫无疑问 add()是一个对象，毕竟它是由构造函数创建的。这里并不推荐使用 Function()构造器创建函数（和 eval()一样糟糕），因为程序逻辑代码是以字符串的形式传入构造器的。这样的代码可读性差，写起来也很费劲，你不得不对逻辑代码中的引号做转义处理，并需要特别关注为了让代码保持一定的可读性而保留的空格和缩进。

函数的第二个重要特性是它能提供作用域支持。在 JavaScript 中没有块级作用域（译注：在 JavaScript1.7 中提供了块级作用域部分特性的支持，可以通过 let 来声明块级作用域内的“局部变量”），也就是说不能通过花括号来创建作用域，JavaScript 中只有函数作用域（译注：这里作者的表述只针对函数而言，此外 JavaScript 还有全局作用域）。在函数内所有通过 var 声明的变量都是局部变量，在函数外部是不可见的。刚才所指花括号无法提供作用域支持的意思是说，如果在 if 条件句内、或在 for 或 while 循环体内用 var 定义了变量，这个变量并不是属于 if 语句或 for（while）循环的局部变量，而是属于它所在的函数。如果不在任何函数内部，它会成为全局变量。在第二章里提到我们要减少对全局命名空间的污染，那么使用函数则是控制变量的作用域的不二之选。

### 术语释义

首先我们先简单讨论下创建函数相关的术语，因为精确无歧义的术语约定和我们所讨论的各种模式一样重要。

看下这个代码片段：

```js
// named function expression
var add = function add(a, b) {
    return a + b;
}; 
```

这段代码描述了一个函数，这种描述称为“带有命名的函数表达式”。

如果函数表达式将名字省略掉（比如下面的示例代码），这时它是“无名字的函数表达式”，通常我们称之为“匿名函数”，比如：

```js
// function expression, a.k.a. anonymous function
var add = function (a, b) {
    return a + b;
}; 
```

因此“函数表达式”是一个更广义的概念，“带有命名的函数表达式”是函数表达式的一种特殊形式，仅仅当需要给函数定义一个可选的名字时使用。

当省略第二个 add，它就成了无名字的函数表达式，这不会对函数定义和调用语法造成任何影响。带名字和不带名字唯一的区别是函数对象的 name 属性是否是一个空字符串。name 属性属于语言的扩展（未在 ECMA 标准中定义），但很多环境都实现了。如果不省略第二个 add，那么属性 add.name 则是"add"，name 属性在用 Firebug 的调试过程中非常有用，还能让函数递归调用自身，其他情况可以省略它。

最后来看一下“函数声明”，函数声明的语法和其他语言中的语法非常类似：

```js
function foo() {
    // function body goes here
} 
```

从语法角度讲，带有命名的函数表达式和函数声明非常像，特别是当不需要将函数表达式赋值给一个变量的时候（在本章后面所讲到的回调模式中有类似的例子）。多数情况下，函数声明和带命名的函数表达式在外观上没有多少不同，只是它们在函数执行时对上下文的影响有所区别，下一小节会讲到。

两种语法的一个区别是末尾的分号。函数声明末尾不需要分号，而函数表达式末尾是需要分号的。推荐你始终不要丢掉函数表达式末尾的分号，即便 JavaScript 可以进行分号补全，也不要冒险这样做。

> 另外我们经常看到“函数直接量”。它用来表示函数表达式或带命名的函数表达式。由于这个术语是有歧义的，所以最好不要用它。

### 声明 vs 表达式：命名与提前

那么，到底应该用哪个呢？函数声明还是函数表达式？在不能使用函数声明语法的场景下，只能使用函数表达式了。下面这个例子中，我们给函数传入了另一个函数对象作为参数，以及给对象定义方法:

```js
// this is a function expression,
// pased as an argument to the function `callMe`
callMe(function () {
    // I am an unnamed function expression
    // also known as an anonymous function
});

// this is a named function expression
callMe(function me() {
    // I am a named function expression
    // and my name is "me"
});

// another function expression
var myobject = {
    say: function () {
        // I am a function expression
    }
}; 
```

函数声明只能出现在“程序代码”中，也就是说在别的函数体内或在全局。这个定义不能赋值给变量或属性，同样不能作为函数调用的参数。下面这个例子是函数声明的合法用法，这里所有的函数 foo()，bar()和 local()都使用函数声明来定义：

```js
// global scope
function foo() {}

function local() {
    // local scope
    function bar() {}
    return bar;
} 
```

### 函数的 name 属性

选择函数定义模式的另一个考虑是只读属性 name 的可用性。尽管标准规范中并未规定，但很多运行环境都实现了 name 属性，在函数声明和带有名字的函数表达式中是有 name 的属性定义的。在匿名函数表达式中，则不一定有定义，这个是和实现相关的，在 IE 中是无定义的，在 Firefox 和 Safari 中是有定义的，但是值为空字符串。

```js
function foo() {} // declaration
var bar = function () {}; // expression
var baz = function baz() {}; // named expression

foo.name; // "foo"
bar.name; // ""
baz.name; // "baz" 
```

在 Firebug 或其他工具中调试程序时 name 属性非常有用，它可以用来显示当前正在执行的函数。同样可以通过 name 属性来递归的调用函数自身。如果你对这些场景不感兴趣，那么请尽可能的使用匿名函数表达式，这样会更简单、且冗余代码更少。

和函数声明相比而言，函数表达式的语法更能说明函数是一种对象，而不是某种特别的语言写法。

> 我们可以将一个带名字的函数表达式赋值给变量，变量名和函数名不同，这在技术上是可行的。比如：`var foo = function bar(){};`。然而，这种用法的行为在浏览器中的兼容性不佳（特别是 IE 中），因此并不推荐大家使用这种模式。

### 函数提前

通过前面的讲解，你可能以为函数声明和带名字的函数表达式是完全等价的。事实上不是这样，主要区别在于“声明提前”的行为。

> 术语“提前”并未在 ECMAScript 中定义，但是并没有其他更好的方法来描述这种行为了。

我们知道，不管在函数内何处声明变量，变量都会自动提前至函数体的顶部。对于函数来说亦是如此，因为他们也是一种对象，赋值给了变量。需要注意的是，函数声明定义的函数不仅能让声明提前，还能让定义提前，看一下这段示例代码：

```js
// antipattern
// for illustration only

// global functions
function foo() {
    alert('global foo');
}
function bar() {
    alert('global bar');
}

function hoistMe() {

    console.log(typeof foo); // "function"
    console.log(typeof bar); // "undefined"

    foo(); // "local foo"
    bar(); // TypeError: bar is not a function

    // function declaration:
    // variable 'foo' and its implementation both get hoisted

    function foo() {
        alert('local foo');
    }

    // function expression:
    // only variable 'bar' gets hoisted
    // not the implementation
    var bar = function () {
        alert('local bar');
    };
}
hoistMe(); 
```

在这段代码中，和普通的变量一样，hoistMe()函数中的 foo 和 bar 被“搬运”到了顶部，覆盖了全局的 foo 和 bar。不同之处在于，局部的 foo()定义提前至顶部并能正常工作，尽管定义它的位置并不靠前。bar()的定义并未提前，只是声明提前了。因此当程序执行到 bar()定义的位置之前，它的值都是 undefined，并不是函数（防止当前上下文查找到作用域链上的全局的 bar()，也就“覆盖”了全局的 bar()）。

到目前为止我们介绍了必要的背景知识和函数定义相关的术语，下面开始介绍一些 JavaScript 所提供的函数相关的好的模式，我们从回调模式开始。同样，再次强调 JavaScript 函数的两个特殊特性，掌握这两点至关重要：

*   函数是对象
*   函数提供局部变量作用域

## 回调模式

函数是对象，也就意味着函数可以当作参数传入另外一个函数中。当你给函数 writeCode()传入一个函数参数 introduceBugs()，在某个时刻 writeCode()执行了（或调用了）introduceBugs()。在这种情况下，我们说 introduceBugs()是一个“回调函数”，简称“回调”：

```js
function writeCode(callback) {
    // do something...
    callback();
    // ...
}

function introduceBugs() {
    // ... make bugs
}

writeCode(introduceBugs); 
```

注意 introduceBugs()是如何作为参数传入 writeCode()的，当作参数的函数不带括号。括号的意思是执行函数，而这里我们希望传入一个引用，让 writeCode()在合适的时机执行它（调用它）。

### 一个回调的例子

我们从一个例子开始，首先介绍无回调的情况，然后在作修改。假设你有一个通用的函数，用来完成某种复杂的逻辑并返回一大段数据。假设我们用 findNodes()来命名这个通用函数，这个函数用来对 DOM 树进行遍历，并返回我所感兴趣的页面节点：

```js
var findNodes = function () {
    var i = 100000, // big, heavy loop
        nodes = [], // stores the result
        found; // the next node found
    while (i) {
        i -= 1;
        // complex logic here...
        nodes.push(found);
    }
    return nodes;
}; 
```

保持这个函数的功能的通用性并一贯返回 DOM 节点组成的数组，并不会发生对节点的实际操作，这是一个不错的注意。可以将操作节点的逻辑放入另外一个函数中，比如放入一个 hide()函数中，这个函数用来隐藏页面中的节点元素：

```js
var hide = function (nodes) {
    var i = 0, max = nodes.length;
    for (; i < max; i += 1) {
        nodes[i].style.display = "none";
    }
};

// executing the functions
hide(findNodes()); 
```

这个实现的效率并不高，因为它将 findNodes()所返回的节点数组重新遍历了一遍。最好在 findNodes()中选择元素的时候就直接应用 hide()操作，这样就能避免第二次的遍历，从而提高效率。但如果将 hide()的逻辑写死在 findNodes()的函数体内，findNodes()就变得不再通用了（译注：如果我将 hide()的逻辑替换成其他逻辑怎么办呢？），因为修改逻辑和遍历逻辑耦合在一起了。如果使用回调模式，则可以将隐藏节点的逻辑写入回调函数，将其传入 findNodes()中适时执行：

```js
// refactored findNodes() to accept a callback
var findNodes = function (callback) {
    var i = 100000,
        nodes = [],
        found;

    // check if callback is callable
    if (typeof callback !== "function") {
        callback = false;
    }
    while (i) {
        i -= 1;

        // complex logic here...

        // now callback:
        if (callback) {
            callback(found);
        }

        nodes.push(found);
    }
    return nodes;
}; 
```

这里的实现比较直接，findNodes()多作了一个额外工作，就是检查回调函数是否存在，如果存在的话就执行它。回调函数是可选的，因此修改后的 findNodes()也是和之前一样使用，是可以兼容旧代码和旧 API 的。

这时 hide()的实现就非常简单了，因为它不用对元素列表做任何遍历了：

```js
// a callback function
var hide = function (node) {
    node.style.display = "none";
};

// find the nodes and hide them as you go
findNodes(hide); 
```

正如代码中所示，回调函数可以是事先定义好的，也可以是一个匿名函数，你也可以将其称作 main 函数，比如这段代码，我们利用同样的通用函数 findNodes()来完成显示元素的操作：

```js
// passing an anonymous callback
findNodes(function (node) {
    node.style.display = "block";
}); 
```

### 回调和作用域

在上一个例子中，执行回调函数的写法是：

```js
callback(parameters); 
```

尽管这种写法可以适用大多数的情况，而且足够简单，但还有一些场景，回调函数不是匿名函数或者全局函数，而是对象的方法。如果回调函数中使用 this 指向它所属的对象，则回调逻辑往往并不像我们希望的那样执行。

假设回调函数是 paint()，它是 myapp 的一个方法：

```js
var myapp = {};
myapp.color = "green";
myapp.paint = function (node) {
    node.style.color = this.color;
}; 
```

函数 findNodes()大致如下：

```js
var findNodes = function (callback) {
    // ...
    if (typeof callback === "function") {
        callback(found);
    }
    // ...
}; 
```

当你调用 findNodes(myapp.paint)，运行结果和我们期望的不一致，因为 this.color 未定义。因为 findNodes()是全局函数，this 指向的是全局对象。如果 findNodes()是 dom 对象的方法（类似 dom.findNodes()），那么回调函数内的 this 则指向 dom，而不是 myapp。

解决办法是，除了传入回调函数，还需将回调函数所属的对象当作参数传进去：

```js
findNodes(myapp.paint, myapp); 
```

同样需要修改 findNodes()的逻辑，增加对传入的对象的绑定：

```js
var findNodes = function (callback, callback_obj) {
    //...
    if (typeof callback === "function") {
        callback.call(callback_obj, found);
    }
    // ...
}; 
```

在后续的章节会对 call()和 apply()有更详细的讲述。

其实还有一种替代写法，就是将函数当作字符串传入 findNodes()，这样就不必再写一次对象了，换句话说：

```js
findNodes(myapp.paint, myapp); 
```

可以写成：

```js
findNodes("paint", myapp); 
```

在 findNodes()中的逻辑则需要修改为：

```js
var findNodes = function (callback, callback_obj) {

    if (typeof callback === "string") {
        callback = callback_obj[callback];
    }

    //...
    if (typeof callback === "function") {
        callback.call(callback_obj, found);
    }
    // ...
}; 
```

### 异步事件监听

JavaScript 中的回调模式已经是我们的家常便饭了，比如，如果你给网页中的元素绑定事件，则需要提供回调函数的引用，以便事件发生时能调用到它。这里有一个简单的例子，我们将 console.log()作为回调函数绑定了 document 的点击事件：

```js
document.addEventListener("click", console.log, false); 
```

客户端浏览器中的大多数编程都是事件驱动的，当网页下载完成，则触发 load 事件，当用户和页面产生交互时也会触发多种事件，比如 click、keypress、mouseover、mousemove 等等。正是由于回调模式的灵活性，JavaScript 天生适于事件驱动编程。回调模式能够让程序“异步”执行，换句话说，就是让程序不按顺序执行。

“不要打电话给我，我会打给你”，这是好莱坞很有名的一句话，很多电影都有这句台词。电影中的主角不可能同时应答很多个电话呼叫。在 JavaScript 的异步事件模型中也是同样的道理。电影中是留下电话号码，JavaScript 中是提供一个回调函数，当时机成熟时就触发回调。有时甚至提供了很多回调，有些回调压根是没用的，但由于这个事件可能永远不会发生，因此这些回调的逻辑也不会执行。比如，假设你从此不再用“鼠标点击”，那么你之前绑定的鼠标点击的回调函数则永远也不会执行。

### 超时

另外一个最常用的回调模式是在调用超时函数时，超时函数是浏览器 window 对象的方法，共有两个：setTimeout()和 setInterval()。这两个方法的参数都是回调函数。

```js
var thePlotThickens = function () {
    console.log('500ms later...');
};
setTimeout(thePlotThickens, 500); 
```

再次需要注意，函数 thePlotThickens 是作为变量传入 setTimeout 的，它不带括号，如果带括号的话则立即执行了，这里只是用到这个函数的引用，以便在 setTimeout 的逻辑中调用到它。也可以传入字符串“thePlotThickens()”，但这是一种反模式，和 eval()一样不推荐使用。

### 库中的回调

回调模式非常简单，但又很强大。可以随手拈来灵活运用，因此这种模式在库的设计中也非常得宠。库的代码要尽可能的保持通用和重用，而回调模式则可帮助库的作者完成这个目标。你不必预料和实现你所想到的所有情形，因为这会让库变的膨胀而臃肿，而且大多数用户并不需要这些多余的特性支持。相反，你将精力放在核心功能的实现上，提供回调的入口作为“钩子”，可以让库的方法变得可扩展、可定制。

## 返回函数

函数是对象，因此当然可以作为返回值。也就是说，函数不一定非要返回一坨数据，函数可以返回另外一个定制好的函数，或者可以根据输入的不同按需创造另外一个函数。

这里有一个简单的例子：一个函数完成了某种功能，可能是一次性初始化，然后都基于这个返回值进行操作，这个返回值恰巧是另一个函数：

```js
var setup = function () {
    alert(1);
    return function () {
        alert(2);
    };
};

// using the setup function
var my = setup(); // alerts 1
my(); // alerts 2 
```

因为 setup()把返回的函数作了包装，它创建了一个闭包，我们可以用这个闭包来存储一些私有数据，这些私有数据可以通过返回的函数进行操作，但在函数外部不能直接读取到这些私有数据。比如这个例子中提供了一个计数器，每次调用这个函数计数器都会加一：

```js
var setup = function () {
    var count = 0;
    return function () {
        return (count += 1);
    };
};

// usage
var next = setup();
next(); // returns 1
next(); // 2
next(); // 3 
```

## 自定义函数

我们动态定义函数，并将函数赋值给变量。如果将你定义的函数赋值给已经存在的函数变量的话，则新函数会覆盖旧函数。这样做的结果是，旧函数的引用就丢弃掉了，变量中所存储的引用值替换成了新的。这样看起来这个变量指代的函数逻辑就发生了变化，或者说函数进行了“重新定义”或“重写”。说起来有些拗口，实际上并不复杂，来看一个例子：

```js
var scareMe = function () {
    alert("Boo!");
    scareMe = function () {
        alert("Double boo!");
    };
};
// using the self-defining function
scareMe(); // Boo!
scareMe(); // Double boo! 
```

当函数中包含一些初始化操作，并希望这些初始化只执行一次，那么这种模式是非常适合这个场景的。因为能避免的重复执行则尽量避免，函数的一部分可能再也不会执行到。在这个场景中，函数执行一次后就被重写为另外一个函数了。

使用这种模式可以帮助提高应用的执行效率，因为重新定义的函数执行更少的代码。

> 这种模式的另外一个名字是“函数的懒惰定义”，因为直到函数执行一次后才重新定义，可以说它是“某个时间点之后才存在”，简称“懒惰定义”。

这种模式有一种明显的缺陷，就是之前给原函数添加的功能在重定义之后都丢失了。如果将这个函数定义为不同的名字，函数赋值给了很多不同的变量，或作为对象的方法使用，那么新定义的函数有可能不会执行，原始的函数会照旧执行（译注：由于函数的赋值是引用的赋值，函数赋值给多个变量只是将引用赋值给了多个变量，当某一个变量定义了新的函数，也只是变量的引用值发生变化，原函数本身依旧存在，当程序中存在某个变量的引用还是旧函数的话，旧函数还是会依旧执行）。

让我们来看一个例子，scareMe()函数在这里作为一等对象来使用：

1.  给他增加了一个属性
2.  函数对象赋值给一个新变量
3.  函数依旧可以作为方法来调用

看一下这段代码：

```js
// 1\. adding a new property
scareMe.property = "properly";

// 2\. assigning to a different name
var prank = scareMe;

// 3\. using as a method
var spooky = {
    boo: scareMe
};

// calling with a new name
prank(); // "Boo!"
prank(); // "Boo!"
console.log(prank.property); // "properly"

// calling as a method
spooky.boo(); // "Boo!"
spooky.boo(); // "Boo!"
console.log(spooky.boo.property);    // "properly"

// using the self-defined function
scareMe(); // Double boo!
scareMe(); // Double boo!
console.log(scareMe.property); // undefined 
```

从结果来看，当自定义函数被赋值给一个新的变量的时候，这段使用自定义函数的代码的执行结果与我们期望的结果可能并不一样。每当 prank()运行的时候，它都弹出“Boo!”。同时它也重写了 scareMe()函数，但是 prank()自己仍然能够使用之前的定义，包括属性 property。在这个函数被作为 spooky 对象的 boo()方法调用的时候，结果也一样。所有的这些调用，在第一次的时候就已经修改了全局的 scareMe()的指向，所以当它最终被调用的时候，它的函数体已经被修改为弹出“Double boo”。它也就不能获取到新添加的属性“property”。

## 立即执行的函数

立即执行的函数是一种语法模式，它会使函数在定义后立即执行。看这个例子：

```js
(function () {
    alert('watch out!');
}()); 
```

这种模式本质上只是一个在创建后就被执行的函数表达式（具名或者匿名）。“立即执行的函数”这种说法并没有在 ECMAScript 标准中被定义，但它作为一个名词，有助于我们的描述和讨论。

这种模式由以下几个部分组成：

*   使用函数表达式定义一个函数。（不能使用函数声明。）
*   在最后加入一对括号，这会使函数立即被执行。
*   把整个函数包裹到一对括号中（只在没有将函数赋值给变量时需要）。

下面这种语法也很常见（注意右括号的位置），但是 JSLint 倾向于第一种：

```js
(function () {
    alert('watch out!');
})(); 
```

这种模式很有用，它为我们提供一个作用域的沙箱，可以在执行一些初始化代码的时候使用。设想这样的场景：当页面加载的时候，你需要运行一些代码，比如绑定事件、创建对象等等。所有的这些代码都只需要运行一次，所以没有必要创建一个带有名字的函数。但是这些代码需要一些临时变量，而这些变量在初始化完之后又不会再次用到。显然，把这些变量作为全局变量声明是不合适的。正因为如此，我们才需要立即执行的函数。它可以把你所有的代码包裹到一个作用域里面，而不会暴露任何变量到全局作用域中：

```js
(function () {

    var days = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'],
        today = new Date(),
        msg = 'Today is ' + days[today.getDay()] + ', ' + today.getDate();

    alert(msg);

}()); // "Today is Fri, 13" 
```

如果这段代码没有被包裹到立即执行函数中，那么变量 days、today、msg 都会是全局变量，而这些变量仅仅是由因为初始化而遗留下来的垃圾，没有任何用处。

### 立即执行的函数的参数

立即执行的函数也可以接受参数，看这个例子：

```js
// prints:
// I met Joe Black on Fri Aug 13 2010 23:26:59 GMT-0800 (PST)

(function (who, when) {

    console.log("I met " + who + " on " + when);

}("Joe Black", new Date())); 
```

通常的做法，会把全局对象当作一个参数传给立即执行的函数，以保证在函数内部也可以访问到全局对象，而不是使用 window 对象，这样可以使得代码在非浏览器环境中使用时更具可移植性。

值得注意的是，一般情况下尽量不要给立即执行的函数传入太多的参数，否则会有一件麻烦的事情，就是你在阅读代码的时候需要频繁地上下滚动代码。

### 立即执行的函数的返回值

和其它的函数一样，立即执行的函数也可以返回值，并且这些返回值也可以被赋值给变量：

```js
var result = (function () {
    return 2 + 2;
}()); 
```

如果省略括号的话也可以达到同样的目的，因为如果需要将返回值赋给变量，那么第一对括号就不是必需的。省略括号的代码是这样子：

```js
var result = function () {
    return 2 + 2;
}(); 
```

这种写法更简洁，但是同时也容易造成误解。如果有人在阅读代码的时候忽略了最后的一对括号，那么他会以为 result 指向了一个函数。而事实上 result 是指向这个函数运行后的返回值，在这个例子中是 4。

还有一种写法也可以得到同样的结果：

```js
var result = (function () {
    return 2 + 2;
})(); 
```

前面的例子中，立即执行的函数返回的是一个基本类型的数值。但事实上，除了基本类型以外，一个立即执行的函数可以返回任意类型的值，甚至返回一个函数都可以。你可以利用立即执行的函数的作用域来存储一些私有的数据，这些数据只能在返回的内层函数中被访问。

在下面的例子中，立即执行的函数的返回值是一个函数，这个函数会简单地返回 res 的值，并且它被赋给了变量 getResult。而 res 是一个预先计算好的变量，它被存储在立即执行函数的闭包中：

```js
var getResult = (function () {
    var res = 2 + 2;
    return function () {
        return res;
    };
}()); 
```

在定义一个对象的属性的时候也可以使用立即执行的函数。设想一下这样的场景：你需要定义一个对象的属性，这个属性在对象的生命周期中都不会改变，但是在定义之前，你需要做一点额外的工作来得到正确的值。这种情况下你就可以使用立即执行的函数来包裹那些额外的工作，然后将它的返回值作为对象属性的值。下面是一个例子：

```js
var o = {
    message: (function () {
        var who = "me",
            what = "call";
        return what + " " + who;
    }()),
    getMsg: function () {
        return this.message;
    }
};

// usage
o.getMsg(); // "call me"
o.message; // "call me" 
```

在这个例子中，o.message 是一个字符串，而不是一个函数，但是它需要一个函数在脚本载入后来得到这个属性值。

### 好处和用法

立即执行的函数应用很广泛。它可以帮助我们做一些不想留下全局变量的工作。所有定义的变量都只是立即执行的函数的本地变量，你完全不用担心临时变量会污染全局对象。

> 立即执行的函数还有一些名字，比如“自调用函数”或者“自执行函数”，因为这些函数会在被定义后立即执行自己。

这种模式也经常被用到书签代码中，因为书签代码会在任何一个页面运行，所以需要非常苛刻地保持全局命名空间干净。

这种模式也可以让你包裹一些独立的特性到一个封闭的模块中。设想你的页面是静态的，在没有 JavaScript 的时候工作正常，然后，本着渐进增强的精神，你给页面加入了一点增加代码。这时候，你就可以把你的代码（也可以叫“模块”或者“特性”）放到一个立即执行的函数中并且保证页面在有没有它的时候都可以正常工作。然后你就可以加入更多的增强特性，或者对它们进行移除、进行独立测试或者允许用户禁用等等。

你可以使用下面的模板定义一段函数代码，我们叫它 module1：

```js
// module1 defined in module1.js
(function () {

    // all the module 1 code ...

}()); 
```

套用这个模板，你就可以编写其它的模块。然后在发布到线上的时候，你就可以决定在这个时间节点上哪些特性是可以使用的，然后使用发布脚本将它们打包上线。

## 立即初始化的对象

还有另外一种可以避免污染全局作用域的方法，和前面描述的立即执行的函数相似，叫做“立即初始化的对象”模式。这种模式使用一个带有 init()方法的对象来实现，这个方法在对象被创建后立即执行。初始化的工作由 init()函数来完成。

下面是一个立即初始化的对象模式的例子：

```js
({
    // here you can define setting values
    // a.k.a. configuration constants
    maxwidth: 600,
    maxheight: 400,

    // you can also define utility methods
    gimmeMax: function () {
        return this.maxwidth + "x" + this.maxheight;
    },

    // initialize
    init: function () {
        console.log(this.gimmeMax());
        // more init tasks...
    }
}).init(); 
```

在语法上，当你使用这种模式的时候就像在使用对象字面量创建一个普通对象一样。除此之外，还需要将对象字面量用括号括起来，这样能让 JavaScript 引擎知道这是一个对象字面量，而不是一个代码块（if 或者 for 循环之类）。在括号后面，紧接着就执行了 init()方法。

你也可以将对象字面量和 init()调用一起写到括号里面。简单地说，下面两种语法都是有效的：

```js
({...}).init();
({...}.init()); 
```

这种模式的好处和自动执行的函数模式是一样的：在做一些一次性的初始化工作的时候保护全局作用域不被污染。从语法上看，这种模式似乎比只包含一段代码在一个匿名函数中要复杂一些，但是如果你的初始化工作比较复杂（这种情况很常见），它会给整个初始化工作一个比较清晰的结构。比如，一些私有的辅助性函数可以被很轻易地看出来，因为它们是这个临时对象的属性，但是如果是在立即执行的函数模式中，它们很可能只是一些散落的函数。

这种模式的一个弊端是，JavaScript 压缩工具可能不能像压缩一段包裹在函数中的代码一样有效地压缩这种模式的代码。这些私有的属性和方法不被会重命名为一些更短的名字，因为从压缩工具的角度来看，保证压缩的可靠性更重要。在写作本书的时候，Google 出品的 Closure Compiler 的“advanced”模式是唯一会重命名立即初始化的对象的属性的压缩工具。一个压缩后的样例是这样：

```js
({d:600,c:400,a:function(){return this.d+"x"+this.c},b:function(){console.log(this.a())}}).b(); 
```

> 这种模式主要用于一些一次性的工作，并且在 init()方法执行完后就无法再次访问到这个对象。如果希望在这些工作完成后保持对对象的引用，只需要简单地在 init()的末尾加上 return this;即可。

## 条件初始化

条件初始化（也叫条件加载）是一种优化模式。当你知道某种条件在整个程序生命周期中都不会变化的时候，那么对这个条件的探测只做一次就很有意义。浏览器探测（或者特征检测）是一个典型的例子。

举例说明，当你探测到 XMLHttpRequest 被作为一个本地对象支持时，就知道浏览器不会在程序执行过程中改变这一情况，也不会出现突然需要去处理 ActiveX 对象的情况。当环境不发生变化的时候，你的代码就没有必要在需要在每次 XHR 对象时探测一遍（并且得到同样的结果）。

另外一些可以从条件初始化中获益的场景是获得一个 DOM 元素的 computed styles 或者是绑定事件处理函数。大部分程序员在他们的客户端编程生涯中都编写过事件绑定和取消绑定相关的组件，像下面的例子：

```js
// BEFORE
var utils = {
    addListener: function (el, type, fn) {
        if (typeof window.addEventListener === 'function') {
            el.addEventListener(type, fn, false);
        } else if (typeof document.attachEvent === 'function') { // IE
            el.attachEvent('on' + type, fn);
        } else { // older browsers
            el['on' + type] = fn;
        }
    },
    removeListener: function (el, type, fn) {
        // pretty much the same...
    }
}; 
```

这段代码的问题就是效率不高。每当你执行 utils.addListener()或者 utils.removeListener()时，同样的检查都会被重复执行。

如果使用条件初始化，那么浏览器探测的工作只需要在初始化代码的时候执行一次。在初始化的时候，代码探测一次环境，然后重新定义这个函数在剩下来的程序生命周期中应该怎样工作。下面是一个例子，看看如何达到这个目的：

```js
// AFTER

// the interface
var utils = {
    addListener: null,
    removeListener: null
};

// the implementation
if (typeof window.addEventListener === 'function') {
    utils.addListener = function (el, type, fn) {
        el.addEventListener(type, fn, false);
    };
    utils.removeListener = function (el, type, fn) {
        el.removeEventListener(type, fn, false);
    };
} else if (typeof document.attachEvent === 'function') { // IE
    utils.addListener = function (el, type, fn) {
        el.attachEvent('on' + type, fn);
    };
    utils.removeListener = function (el, type, fn) {
        el.detachEvent('on' + type, fn);
    };
} else { // older browsers
    utils.addListener = function (el, type, fn) {
        el['on' + type] = fn;
    };
    utils.removeListener = function (el, type, fn) {
        el['on' + type] = null;
    };
} 
```

说到这里，要特别提醒一下关于浏览器探测的事情。当你使用这个模式的时候，不要对浏览器特性过度假设。举个例子，如果你探测到浏览器不支持 window.addEventListener，不要假设这个浏览器是 IE，也不要认为它不支持原生的 XMLHttpRequest，虽然这个结论在整个浏览器历史上的某个点是正确的。当然，也有一些情况是可以放心地做一些特性假设的，比如.addEventListener 和.removeEventListerner，但是通常来讲，浏览器的特性在发生变化时都是独立的。最好的策略就是分别探测每个特性，然后使用条件初始化，使这种探测只做一次。

## 函数属性——Memoization 模式

函数也是对象，所以它们可以有属性。事实上，函数也确实本来就有一些属性。比如，对一个函数来说，不管是用什么语法创建的，它会自动拥有一个 length 属性来标识这个函数期待接受的参数个数：

```js
function func(a, b, c) {}
console.log(func.length); // 3 
```

任何时候都可以给函数添加自定义属性。添加自定义属性的一个有用场景是缓存函数的执行结果（返回值），这样下次同样的函数被调用的时候就不需要再做一次那些可能很复杂的计算。缓存一个函数的运行结果也就是为大家所熟知的 Memoization。

在下面的例子中，myFunc 函数创建了一个 cache 属性，可以通过 myFunc.cache 访问到。这个 cache 属性是一个对象（hash 表），传给函数的参数会作为对象的 key，函数执行结果会作为对象的值。函数的执行结果可以是任何的复杂数据结构：

```js
var myFunc = function (param) {
    if (!myFunc.cache[param]) {
        var result = {};
        // ... expensive operation ...
        myFunc.cache[param] = result;
    }
    return myFunc.cache[param];
};

// cache storage
myFunc.cache = {}; 
```

上面的代码假设函数只接受一个参数 param，并且这个参数是基本类型（比如字符串）。如果你有更多更复杂的参数，则通常需要对它们进行序列化。比如，你需要将 arguments 对象序列化为 JSON 字符串，然后使用 JSON 字符串作为 cache 对象的 key：

```js
var myFunc = function () {

    var cachekey = JSON.stringify(Array.prototype.slice.call(arguments)),
        result;

    if (!myFunc.cache[cachekey]) {
        result = {};
        // ... expensive operation ...
        myFunc.cache[cachekey] = result;
    }
    return myFunc.cache[cachekey];
};

// cache storage
myFunc.cache = {}; 
```

需要注意的是，在序列化的过程中，对象的“标识”将会丢失。如果你有两个不同的对象，却碰巧有相同的属性，那么他们会共享同样的缓存内容。

前面代码中的函数名还可以使用 arguments.callee 来替代，这样就不用将函数名硬编码。不过尽管现阶段这个办法可行，但是仍然需要注意，arguments.callee 在 ECMAScript 5 的严格模式中是不被允许的：

```js
var myFunc = function (param) {

    var f = arguments.callee,
        result;

    if (!f.cache[param]) {
        result = {};
        // ... expensive operation ...
        f.cache[param] = result;
    }
    return f.cache[param];
};

// cache storage
myFunc.cache = {}; 
```

## 配置对象

配置对象模式是一种提供更简洁的 API 的方法，尤其是当你正在写一个即将被其它程序调用的类库之类的代码的时候。

软件在开发和维护过程中需要不断改变是一个不争的事实。这样的事情总是以一些有限的需求开始，但是随着开发的进行，越来越多的功能会不断被加进来。

设想一下你正在写一个名为 addPerson()的函数，它接受一个姓和一个名，然后在列表中加入一个人：

```js
function addPerson(first, last) {...} 
```

然后你意识到，生日也必须要存储，此外，性别和地址也作为可选项存储。所以你修改了函数，添加了一些新的参数（还得非常小心地将可选参数放到最后）：

```js
function addPerson(first, last, dob, gender, address) {...} 
```

这个时候，函数已经显得有点长了。然后，你又被告知需要添加一个用户名，并且不是可选的。现在这个函数的调用者需要将所有的可选参数传进来，并且得非常小心地保证不弄混参数的顺序：

```js
addPerson("Bruce", "Wayne", new Date(), null, null, "batman"); 
```

传一大串的参数真的很不方便。一个更好的办法就是将它们替换成一个参数，并且把这个参数弄成对象；我们叫它 conf，是“configuration”（配置）的缩写：

```js
addPerson(conf); 
```

然后这个函数的使用者就可以这样：

```js
var conf = {
    username: "batman",
    first: "Bruce",
    last: "Wayne"
};
addPerson(conf); 
```

配置对象模式的好处是：

*   不需要记住参数的顺序
*   可以很安全地跳过可选参数
*   拥有更好的可读性和可维护性
*   更容易添加和移除参数

配置对象模式的坏处是：

*   需要记住参数的名字
*   参数名字不能被压缩

举些实例，这个模式对创建 DOM 元素的函数或者是给元素设定 CSS 样式的函数会非常实用，因为元素和 CSS 样式可能会有很多但是大部分可选的属性。

## 柯里化 （Curry）

在本章剩下的部分，我们将讨论一下关于柯里化和部分应用的话题。但是在我们开始这个话题之前，先看一下到底什么是函数应用。

### 函数应用

在一些纯粹的函数式编程语言中，对函数的描述不是被调用（called 或者 invoked），而是被应用（applied）。在 JavaScript 中也有同样的东西——我们可以使用 Function.prototype.apply()来应用一个函数，因为在 JavaScript 中，函数实际上是对象，并且他们拥有方法。

下面是一个函数应用的例子：

```js
// define a function
var sayHi = function (who) {
    return "Hello" + (who ? ", " + who : "") + "!";
};

// invoke a function
sayHi(); // "Hello"
sayHi('world'); // "Hello, world!"

// apply a function
sayHi.apply(null, ["hello"]); // "Hello, hello!" 
```

从上面的例子中可以看出来，调用一个函数和应用一个函数有相同的结果。apply()接受两个参数：第一个是在函数内部绑定到 this 上的对象，第二个是一个参数数组，参数数组会在函数内部变成一个类似数组的 arguments 对象。如果第一个参数为 null，那么 this 将指向全局对象，这正是当你调用一个函数（且这个函数不是某个对象的方法）时发生的事情。

当一个函数是一个对象的方法时，我们不再像前面的例子一样传入 null。（译注：主要是为了保证方法中的 this 绑定到一个有效的对象而不是全局对象。）在下面的例子中，对象被作为第一个参数传给 apply()：

```js
var alien = {
    sayHi: function (who) {
        return "Hello" + (who ? ", " + who : "") + "!";
    }
};

alien.sayHi('world'); // "Hello, world!"
sayHi.apply(alien, ["humans"]); // "Hello, humans!" 
```

在这个例子中，sayHi()中的 this 指向 alien。而在上一个例子中，this 是指向的全局对象。（译注：这个例子的代码有误，最后一行的 sayHi 并不能访问到 alien 的 sayHi 方法，需要使用 alien.sayHi.apply(alien, ["humans"])才可正确运行。另外，在 sayHi 中也没有出现 this。）

正如上面两个例子所展现出来的一样，我们将所谓的函数调用当作函数应用的一种语法糖并没有什么太大的问题。

需要注意的是，除了 apply()之外，Function.prototype 对象还有一个 call()方法，但是它仍然只是 apply()的一种语法糖。（译注：这两个方法的区别在于，apply()只接受两个参数，第二个参数为需要传给函数的参数数组，而 call()则接受任意多个参数，从第二个开始将参数依次传给函数。）不过有种情况下使用这个语法糖会更好：当你的函数只接受一个参数的时候，你可以省去为唯一的一个元素创建数组的工作：

```js
// the second is more efficient, saves an array
sayHi.apply(alien, ["humans"]); // "Hello, humans!"
sayHi.call(alien, "humans"); // "Hello, humans!" 
```

### 部分应用

现在我们知道了，调用一个函数实际上就是给它应用一堆参数，那是否能够只传一部分参数而不传全部呢？这实际上跟我们手工处理数学函数非常类似。

假设已经有了一个 add()函数，它的工作是把 x 和 y 两个数加到一起。下面的代码片段展示了当 x 为 5、y 为 4 时的计算步骤：

```js
// for illustration purposes
// not valid JavaScript

// we have this function
function add(x, y) {
    return x + y;
}

// and we know the arguments
add(5, 4);

// step 1 -- substitute one argument
function add(5, y) {
    return 5 + y;
}

// step 2 -- substitute the other argument
function add(5, 4) {
    return 5 + 4;
} 
```

在这个代码片段中，step 1 和 step 2 并不是有效的 JavaScript 代码，但是它展示了我们手工计算的过程。首先获得第一个参数的值，然后将未知的 x 和已知的值 5 替换到函数中。然后重复这个过程，直到替换掉所有的参数。

step 1 是一个所谓的部分应用的例子：我们只应用了第一个参数。当你执行一个部分应用的时候并不能获得结果（或者是解决方案），取而代之的是另一个函数。

下面的代码片段展示了一个虚拟的 partialApply()方法的用法：

```js
var add = function (x, y) {
    return x + y;
};

// full application
add.apply(null, [5, 4]); // 9

// partial application
var newadd = add.partialApply(null, [5]);
// applying an argument to the new function
newadd.apply(null, [4]); // 9 
```

正如你所看到的一样，部分应用给了我们另一个函数，这个函数可以在稍后调用的时候接受其它的参数。这实际上跟 add(5)(4)是等价的，因为 add(5)返回了一个函数，这个函数可以使用(4)来调用。我们又一次看到，熟悉的 add(5, 4)也差不多是 add(5)(4)的一种语法糖。

现在，让我们回到地球：并不存在这样的一个 partialApply()函数，并且函数的默认表现也不会像上面的例子中那样。但是你完全可以自己去写，因为 JavaScript 的动态特性完全可以做到这样。

让函数理解并且处理部分应用的过程，叫柯里化（Currying）。

### 柯里化（Currying）

柯里化和辛辣的印度菜可没什么关系；它来自数学家 Haskell Curry。（Haskell 编程语言也是因他而得名。）柯里化是一个变换函数的过程。柯里化的另外一个名字也叫 schönfinkelisation，来自另一位数学家——Moses Schönfinkelisation——这种变换的最初发明者。

所以我们怎样对一个函数进行柯里化呢？其它的函数式编程语言也许已经原生提供了支持并且所有的函数已经默认柯里化了。在 JavaScript 中我们可以修改一下 add()函数使它柯里化，然后支持部分应用。

来看一个例子：

```js
// a curried add()
// accepts partial list of arguments
function add(x, y) {
    var oldx = x, oldy = y;
    if (typeof oldy === "undefined") { // partial
        return function (newy) {
            return oldx + newy;
        };
    }
    // full application
    return x + y;
}

// test
typeof add(5); // "function"
add(3)(4); // 7

// create and store a new function
var add2000 = add(2000);
add2000(10); // 2010 
```

在这段代码中，第一次调用 add()时，在返回的内层函数那里创建了一个闭包。这个闭包将原来的 x 和 y 的值存储到了 oldx 和 oldy 中。当内层函数执行的时候，oldx 会被使用。如果没有部分应用，即 x 和 y 都传了值，那么这个函数会简单地将他们相加。这个 add()函数的实现跟实际情况比起来有些冗余，仅仅是为了更好地说明问题。下面的代码片段中展示了一个更简洁的版本，没有 oldx 和 oldy，因为原始的 x 已经被存储到了闭包中，此外我们复用了 y 作为本地变量，而不用像之前那样新定义一个变量 newy：

```js
// a curried add
// accepts partial list of arguments
function add(x, y) {
    if (typeof y === "undefined") { // partial
        return function (y) {
            return x + y;
        };
    }
    // full application
    return x + y;
} 
```

在这些例子中，add()函数自己处理了部分应用。有没有可能用一种更为通用的方式来做同样的事情呢？换句话说，我们能不能对任意一个函数进行处理，得到一个新函数，使它可以处理部分参数？下面的代码片段展示了一个通用函数的例子，我们叫它 schonfinkelize()，正是用来做这个的。我们使用 schonfinkelize()这个名字，一部分原因是它比较难发音，另一部分原因是它听起来比较像动词（使用“curry”则不是那么明确），而我们刚好需要一个动词来表明这是一个函数转换的过程。

这是一个通用的柯里化函数：

```js
function schonfinkelize(fn) {
    var slice = Array.prototype.slice,
    stored_args = slice.call(arguments, 1);
    return function () {
        var new_args = slice.call(arguments),
        args = stored_args.concat(new_args);
        return fn.apply(null, args);
    };
} 
```

这个 schonfinkelize 可能显得比较复杂了，只是因为在 JavaScript 中 arguments 不是一个真的数组。从 Array.prototype 中借用 slice()方法帮助我们将 arguments 转换成数组，以便能更好地对它进行操作。当 schonfinkelize()第一次被调用的时候，它使用 slice 变量存储了对 slice()方法的引用，同时也存储了调用时的除去第一个之外的参数（stored_args），因为第一个参数是要被柯里化的函数。schonfinkelize()返回了一个函数。当这个返回的函数被调用的时候，它可以（通过闭包）访问到已经存储的参数 stored_args 和 slice。新的函数只需要合并老的部分应用的参数（stored_args）和新的参数（new_args），然后将它们应用到原来的函数 fn（也可以在闭包中访问到）即可。

现在有了通用的柯里化函数，就可以做一些测试了：

```js
// a normal function
function add(x, y) {
    return x + y;
}

// curry a function to get a new function
var newadd = schonfinkelize(add, 5);
newadd(4); // 9

// another option -- call the new function directly
schonfinkelize(add, 6)(7); // 13 
```

用来做函数转换的 schonfinkelize()并不局限于单个参数或者单步的柯里化。这里有些更多用法的例子：

```js
// a normal function
function add(a, b, c, d, e) {
    return a + b + c + d + e;
}

// works with any number of arguments
schonfinkelize(add, 1, 2, 3)(5, 5); // 16

// two-step currying
var addOne = schonfinkelize(add, 1);
addOne(10, 10, 10, 10); // 41
var addSix = schonfinkelize(addOne, 2, 3);
addSix(5, 5); // 16 
```

### 什么时候使用柯里化

当你发现自己在调用同样的函数并且传入的参数大部分都相同的时候，就是考虑柯里化的理想场景了。你可以通过传入一部分的参数动态地创建一个新的函数。这个新函数会存储那些重复的参数（所以你不需要再每次都传入），然后再在调用原始函数的时候将整个参数列表补全，正如原始函数期待的那样。

## 小结

在 JavaScript 中，开发者对函数的理解和运用的要求是比较苛刻的。在本章中，主要讨论了有关函数的一些背景知识和术语。介绍了 JavaScript 函数中两个重要的特性，也就是：

1.  函数是一等对象，他们可以被作为值传递，也可以拥有属性和方法。
2.  函数拥有本地作用域，而大括号不产生块级作用域。另外需要注意的是，变量的声明会被提前到本地作用域顶部。

创建一个函数的语法有：

1.  带有名字的函数表达式
2.  函数表达式（和上一种一样，但是没有名字），也就是为大家熟知的“匿名函数”
3.  函数声明，与其它语言的函数语法相似

在介绍完背景和函数的语法后，介绍了一些有用的模式，按分类列出：

1.  API 模式，它们帮助我们为函数给出更干净的接口，包括：

    *   回调模式

        ```js
        传入一个函数作为参数 
        ```

    *   配置对象

        ```js
         帮助保持函数的参数数量可控 
        ```

    *   返回函数

        ```js
         函数的返回值是另一个函数 
        ```

    *   柯里化

        ```js
         新函数在已有函数的基础上再加上一部分参数构成 
        ```

2.  初始化模式，这些模式帮助我们用一种干净的、结构化的方法来做一些初始化工作（在 web 页面和应用中非常常见），通过一些临时变量来保证不污染全局命名空间。这些模式包括：

    *   立即执行的函数

        ```js
         当它们被定义后立即执行 
        ```

    *   立即初始化的对象

        ```js
         初始化工作被放入一个匿名对象，这个对象提供一个可以立即被执行的方法 
        ```

    *   条件初始化

        ```js
         使分支代码只在初始化的时候执行一次，而不是在整个程序生命周期中反复执行 
        ```

3.  性能模式，这些模式帮助提高代码的执行速度，包括：

    *   Memoization

        ```js
         利用函数的属性，使已经计算过的值不用再次计算 
        ```

    *   自定义函数

        ```js
         重写自身的函数体，使第二次及后续的调用做更少的工作 
        ```