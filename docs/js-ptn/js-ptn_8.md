# 第八章 DOM 和浏览器模式

# DOM 和浏览器中的模式

在本书的前面几章中，我们主要关注了 JavaScript 核心（ECMAScript），并没有涉及太多关于在浏览器中使用 JavaScript 的内容。在本章，我们将探索一些在浏览器环境中的模式，因为这是最常见的 JavaScript 程序环境。浏览器脚本编程也是大部分不喜欢 JavaScript 的人对这门语言的认知。这当然是可以理解，因为在浏览器中有非常多不一致的宿主对象和 DOM 实现。很明显，任何能够减轻客户端脚本编程的痛楚的最佳初中都是大有益处的。

在本章中，你会看到一些零散的模式，包括 DOM 编程、事件处理、远程脚本、页面脚本的加载策略以及将 JavaScript 部署到生产环境的步骤。

但首先，让我们来简要讨论一下如何做客户端脚本编程。

## 分离

在 web 应用开发中主要关注的有三种东西：

*   内容

    即 HTML 文档

*   表现

    指定文档样式的 CSS

*   行为

    JavaScript，用来处理用户交互和页面的动态变化

尽可能地将这三者分离可以加强应用在各种用户代理（译注：user agent，即为用户读取页面并呈现的软件，一般指浏览器）的可到达性（译注：delivery，指可被用户代理接受并理解的程度），比如图形浏览器、纯文本浏览器、用于残障人士的辅助技术、移动设备等等。分离常常是和渐进增强的思想一起实现的，我们从一个给最简单的用户代理的最基础的体验（纯 HTML）开始，当用户代理的兼容性提升时再添加更多的可以为体验加分的东西。如果浏览器支持 CSS，那么用户会看到文档更好的呈现。如果浏览器支持 JavaScript，那文档会更像一个应用，提供更多的特性来增强用户体验。

在实践中，分离意味者：

*   在关掉 CSS 的情况下测试页面，看页面是否仍然可用，内容是否可以呈现和阅读
*   在关掉 JavaScript 的情况下测试页面，确保页面仍然可以完成它的主要功能，所有的链接都可以正常工作（没有 href="#"的链接），表单仍然可以正常填写和提交
*   不要使用内联的事件处理（如 onclick）或者是内联的 style 属性，因为它们不属于内容层
*   使用语义化的 HTML 元素，比如头部和列表等

JavaScript（行为）层的地位不应该很显赫，也就是说它不应该成为页面正常工作必须的东西，不应该使得用户在使用不支持的浏览器操作时存在障碍。它只应该被用来增强页面。

通常比较优雅的用来处理浏览器差异的方法是特性检测。它的思想是你不应该使用浏览器类型检测来决定代码的逻辑，而是应该检测在当前环境中你需要使用的某个方法或者是属性是否存在。浏览器检测一般认为是一种“反模式”（译注：anitpattern，指不好的模式）。虽然有的情况下不可避免要使用，但它应该是最后考虑的选择，并且应该只在特性检测没有办法给出明确答案（或者造成明显性能问题）的时候使用：

```js
// antipattern
if (navigator.userAgent.indexOf('MSIE') !== −1) {
    document.attachEvent('onclick', console.log);
}

// better
if (document.attachEvent) {
    document.attachEvent('onclick', console.log);
}

// or even more specific
if (typeof document.attachEvent !== "undefined") {
    document.attachEvent('onclick', console.log);
} 
```

分离也有助于开发、维护，减少升级一个现有应用的难度，因为当出现问题的时候，你知道去看哪一块。当出现一个 JavaScript 错误的时候，你不需要去看 HTML 或者是 CSS 就能修复它。

## DOM 编程

操作页面的 DOM 树是在客户端 JavaScript 编程中最普遍的动作。这也是导致开发者头疼的最主要原因（这也导致了 JavaScript 名声不好），因为 DOM 方法在不同的浏览器中实现得有很多差异。这也是为什么使用一个抽象了浏览器差异的 JavaScript 库能显著提高开发速度的原因。

我们来看一些在访问和修改 DOM 树时推荐的模式，主要考虑点是性能方面。

### DOM 访问

DOM 操作性能不好，这是影响 JavaScript 性能的最主要原因。性能不好是因为浏览器的 DOM 实现通常是和 JavaScript 引擎分离的。从浏览器的角度来讲，这样做是很有意义的，因为有可能一个 JavaScript 应用根本不需要 DOM，而除了 JavaScript 之外的其它语言（如 IE 的 VBScript）也可以用来操作页面中的 DOM。

一个原则就是 DOM 访问的次数应该被减少到最低，这意味者：

*   避免在环境中访问 DOM
*   将 DOM 引用赋给本地变量，然后操作本地变量
*   当可能的时候使用 selectors API
*   遍历 HTML collections 时缓存 length（见第二章）

看下面例子中的第二个（better）循环，尽管它看起来更长一些，但却要快上几十上百倍（取决于具体浏览器）：

```js
// antipattern
for (var i = 0; i < 100; i += 1) {
    document.getElementById("result").innerHTML += i + ", ";
}

// better - update a local variable var i, content = "";
for (i = 0; i < 100; i += 1) {
    content += i + ",";
}
document.getElementById("result").innerHTML += content; 
```

在下一个代码片段中，第二个例子（使用了本地变量 style）更好，尽管它需要多写一行代码，还需要多定义一个变量：

```js
// antipattern
var padding = document.getElementById("result").style.padding,
    margin = document.getElementById("result").style.margin;

// better
var style = document.getElementById("result").style,
    padding = style.padding,
    margin = style.margin; 
```

使用 selectors API 是指使用这个方法：

```js
document.querySelector("ul .selected");
document.querySelectorAll("#widget .class"); 
```

这两个方法接受一个 CSS 选择器字符串，返回匹配这个选择器的 DOM 列表（译注：querySelector 只返回第一个匹配的 DOM）。selectors API 在现代浏览器（以及 IE8+）可用，它总是会比你使用其它 DOM 方法来做同样的选择要快。主流的 JavaScript 库的最近版本都已经使用了这个 API，所以你有理由去检查你的项目，确保使用的是最新版本。

给你经常访问的元素加上一个 id 属性也是有好处的，因为 document.getElementById(myid)是找到一个 DOM 元素最容易也是最快的方法。

### DOM 操作

除了访问 DOM 元素之外，你可能经常需要改变它们、删除其中的一些或者是添加新的元素。更新 DOM 会导致浏览器重绘（repaint）屏幕，也经常导致重排(reflow)（重新计算元素的位置），这些操作代价是很高的。

再说一次，通用的原则仍然是尽量少地更新 DOM，这意味着我们可以将变化集中到一起，然后在“活动的”（live）文档树之外去执行这些变化。

当你需要添加一棵相对较大的子树的时候，你应该在完成这棵树的构建之后再放到文档树中。为了达到这个目的，你可以使用文档碎片（document fragment）来包含你的节点。

不要这样添加节点：

```js
// antipattern
// appending nodes as they are created

var p, t;

p = document.createElement('p');
t = document.createTextNode('first paragraph');
p.appendChild(t);
document.body.appendChild(p);

p = document.createElement('p');
t = document.createTextNode('second paragraph');
p.appendChild(t);
document.body.appendChild(p); 
```

一个更好的版本是创建一个文档碎片，然后“离线地”（译注：即不在文档树中）更新它，当它准备好之后再将它加入文档树中。当你将文档碎片添加到 DOM 树中时，碎片的内容将会被添加进去，而不是碎片本身。这个特性非常好用。所以当有好几个没有被包裹在同一个父元素的节点时，文档碎片是一个很好的包裹方式。

下面是使用文档碎片的例子：

```js
var p, t, frag;

frag = document.createDocumentFragment();

p = document.createElement('p');
t = document.createTextNode('first paragraph');
p.appendChild(t);
frag.appendChild(p);

p = document.createElement('p');
t = document.createTextNode('second paragraph');
p.appendChild(t);
frag.appendChild(p);

document.body.appendChild(frag); 
```

这个例子和前面例子中每段更新一次相比，文档树只被更新了一下，只导致一次重排/重绘。

当你添加新的节点到文档中时，文档碎片很有用。当你需要更新已有的节点时，你也可以将这些变化集中。你可以将你要修改的子树的父节点克隆一份，然后对克隆的这份做修改，完成之后再去替换原来的元素。

```js
var oldnode = document.getElementById('result'),
    clone = oldnode.cloneNode(true);

// work with the clone...

// when you're done:
oldnode.parentNode.replaceChild(clone, oldnode); 
```

## 事件

在浏览器脚本编程中，另一块充满兼容性问题并且带来很多不愉快的区域就是浏览器事件，比如 click，mouseover 等等。同样的，一个 JavaScript 库可以解决支持 IE（9 以下）和 W3C 标准实现的双倍工作量。

我们来看一下一些主要的点，因为你在做一些简单的页面或者快速开发的时候可能不会使用已有的库，当然，也有可能你正在写你自己的库。

### 事件处理

麻烦是从给元素绑定事件开始的。假设你有一个按钮，点击它的时候增加计数器的值。你可以添加一个内联的 onclick 属性，这在所有的浏览器中都能正常工作，但是会违反分离和渐进增强的思想。所以你应该尽力在 JavaScript 中来做绑定，而不是在标签中。

假设你有下面的标签：

```js
<button id="clickme">Click me: 0</button> 
```

你可以将一个函数赋给节点的 onclick 属性，但你只能这样做一次：

```js
// suboptimal solution
var b = document.getElementById('clickme'),
    count = 0;
b.onclick = function () {
    count += 1;
    b.innerHTML = "Click me: " + count;
}; 
```

如果你希望在按钮点击的时候执行好几个函数，那么在维持松耦合的情况下就不能用这种方法来做绑定。从技术上讲，你可以检测 onclick 是否已经包含一个函数，如果已经包含，就将它加到你自己的函数中，然后替换 onclick 的值为你的新函数。但是一个更干净的解决方案是使用 addEventListener()方法。这个方法在 IE8 及以下版本中不存在，在这些浏览器需要使用 attachEvent()。

当我们回头看条件初始化模式（第四章）时，会发现一个示例实现是一个很好的解决跨浏览器事件监听的套件。现在我们不讨论细节，只看一下如何给我们的按钮绑定事件：

```js
var b = document.getElementById('clickme');
if (document.addEventListener) { // W3C
    b.addEventListener('click', myHandler, false);
} else if (document.attachEvent) { // IE
    b.attachEvent('onclick', myHandler);
} else { // last resort
    b.onclick = myHandler;
} 
```

现在当按钮被点击时，myHandler 会被执行。让我们来让这个函数实现增加按钮文字“Click me: 0”中的数字的功能。为了更有趣一点，我们假设有好几个按钮，一个 myHandler()函数来处理所有的按钮点击。如果我们可以从每次点击的事件对象中获取节点和节点对应的计数器值，那为每个按钮保持一个引用和计数器就显得不高效了。

我们先看一下解决方案，稍后再来做些评论：

```js
function myHandler(e) {

    var src, parts;

    // get event and source element 
    e = e || window.event;
    src = e.target || e.srcElement;

    // actual work: update label
    parts = src.innerHTML.split(": ");
    parts[1] = parseInt(parts[1], 10) + 1;
    src.innerHTML = parts[0] + ": " + parts[1];

    // no bubble
    if (typeof e.stopPropagation === "function") {
    e.stopPropagation();
    }
    if (typeof e.cancelBubble !== "undefined") {
    e.cancelBubble = true;
    }

    // prevent default action
    if (typeof e.preventDefault === "function") {
    e.preventDefault();
    }
    if (typeof e.returnValue !== "undefined") {
    e.returnValue = false;
    }

} 
```

一个在线的例子可以在[`jspatterns.com/book/8/click.html`](http://jspatterns.com/book/8/click.html)找到。

在这个事件处理函数中，有四个部分：

*   首先，我们需要访问事件对象，它包含事件的一些信息以及触发这个事件的页面元素。事件对象会被传到事件处理回调函数中，但是使用 onclick 属性时需要使用全局属性 window.event 来获取。
*   第二部分是真正用于更新文字的部分
*   接下来是阻止事件冒泡。在这个例子中它不是必须的，但通常情况下，如果你不阻止的话，事件会一直冒泡到文档根元素甚至 window 对象。同样的，我们也需要用两种方法来阻止冒泡：W3C 标准方式（stopPropagation()）和 IE 的方式（使用 cancelBubble）
*   最后，如果需要的话，阻止默认行为。有一些事件（点击链接、提交表单）有默认的行为，但你可以使用 preventDefault()（IE 是通过设置 returnValue 的值为 false 的方式）来阻止这些默认行为。

如你所见，这里涉及到了很多重复性的工作，所以使用第七章讨论过的外观模式创建自己的事件处理套件是很有意义的。

### 事件委托

事件委托是通过事件冒泡来实现的，它可以减少分散到各个节点上的事件处理函数的数量。如果有 10 个按钮在一个 div 元素中，你可以给 div 绑定一个事件处理函数，而不是给每个按钮都绑定一个。

我们来的睦一个实例，三个按钮放在一个 div 元素中（图 8-1）。你可以在[`jspatterns.com/book/8/click-delegate.html`](http://jspatterns.com/book/8/click-delegate.html)看到这个事件委托的实例。

![图 8-1 事件委托示例：三个在点击时增加计数器值的按钮](img/8-1.jpg)

图 8-1 事件委托示例：三个在点击时增加计数器值的按钮

结构是这样的：

```js
<div id="click-wrap">
    <button>Click me: 0</button>
    <button>Click me too: 0</button>
    <button>Click me three: 0</button>
</div> 
```

你可以给包裹按钮的 div 绑定一个事件处理函数，而不是给每个按钮绑定一个。然后你可以使用和前面的示例中一样的 myHandler()函数，但需要修改一个小地方：你需要将你不感兴趣的点击排除掉。在这个例子中，你只关注按钮上的点击，而在同一个 div 中产生的其它的点击应该被忽略掉。

myHandler()的改变就是检查事件来源的 nodeName 是不是“button”:

```js
// ...
// get event and source element
e = e || window.event;
src = e.target || e.srcElement;

if (src.nodeName.toLowerCase() !== "button") {
    return;
}
// ... 
```

事件委托的坏处是筛选容器中感兴趣的事件使得代码看起来更多了，但好处是性能的提升和更干净的代码，这个好处明显大于坏处，因此这是一种强烈推荐的模式。

主流的 JavaScript 库通过提供方便的 API 的方式使得使用事件委托变得很容易。比如 YUI3 中有 Y.delegate()方法，它允许你指定一个用来匹配包裹容器的 CSS 选择器和一个用于匹配你感兴趣的节点的 CSS 选择器。这很方便，因为如果事件发生在你不关心的元素上时，你的事件处理回调函数不会被调用。在这种情况下，绑定一个事件处理函数很简单：

```js
Y.delegate('click', myHandler, "#click-wrap", "button"); 
```

感谢 YUI 抽象了浏览器的差异，已经处理好了事件的来源，使得回调函数更简单了：

```js
function myHandler(e) {

    var src = e.currentTarget,
        parts;

    parts = src.get('innerHTML').split(": ");
    parts[1] = parseInt(parts[1], 10) + 1;
    src.set('innerHTML', parts[0] + ": " + parts[1]);

    e.halt();
} 
```

你可以在[`jspatterns.com/book/8/click-y-delegate.html`](http://jspatterns.com/book/8/click-y-delegate.html)看到实例。

## 长时间运行的脚本

你可能注意到过，有时候浏览器会提示脚本运行时间过长，询问用户是否要停止执行。这种情况你当然不希望发生在自己的应用中，不管它有多复杂。

同时，如果脚本运行时间太长的话，浏览器的 UI 将变得没有响应，用户不能点击任何东西。这是一种很差的用户体验，应该尽量避免。

在 JavaScript 中没有线程，但你可以在浏览器中使用 setTimeout 来模拟，或者在现代浏览器中使用 web workers。

### setTimeout()

它的思想是将一大堆工作分解成为一小段一小段，然后每隔 1 毫秒运行一段。使用 1 毫秒的延迟会导致整个任务完成得更慢，但是用户界面会保持可响应状态，用户会觉得浏览器没有失控，觉得更舒服。

> 1 毫秒（甚至 0 毫秒）的延迟执行命令在实际运行的时候会延迟更多，这取决于浏览器和操作系统。设定 0 毫秒的延迟并不意味着马上执行，而是指“尽快执行”。比如，在 IE 中，最短的延迟是 15 毫秒。

### Web Workers

现代浏览器为长时间运行的脚本提供了另一种解决方案：web workers。web workers 在浏览器内部提供了后台线程支持，你可以将计算量很大的部分放到一个单独的文件中，比如 my_web_worker.js，然后从主程序（页面）中这样调用它：

```js
var ww = new Worker('my_web_worker.js');
ww.onmessage = function (event) {
    document.body.innerHTML +=
        "<p>message from the background thread: " + event.data + "</p>";
}; 
```

下面展示了一个做 1 亿次简单的数学运算的 web worker：

```js
var end = 1e8, tmp = 1;

postMessage('hello there');

while (end) {
    end -= 1;
    tmp += end;
    if (end === 5e7) { // 5e7 is the half of 1e8
        postMessage('halfway there, `tmp` is now ' + tmp);
    }
}

postMessage('all done'); 
```

web worker 使用 postMessage()来和调用它的程序通讯，调用者通过 onmessage 事件来接受更新。onmessage 事件处理函数接受一个事件对象作为参数，这个对象含有一个由 web worker 传过来 data 属性。类似的，调用者（在这个例子中）也可以使用 ww.postMessage()来给 web worker 传递数据，web worker 可以通过一个 onmessage 事件处理函数来接受这些数据。

上面的例子会在浏览器中打印出：

```js
message from the background thread: hello there
message from the background thread: halfway there, `tmp` is now 3749999975000001 message from the background thread: all done 
```

## 远程脚本编程

现代 web 应用经常会使用远程脚本编程和服务器通讯，而不刷新当前页面。这使得 web 应用更灵活，更像桌面程序。我们来看一下几种用 JavaScript 和服务器通讯的方法。

### XMLHttpRequest

现在，XMLHttpRequest 是一个特别的对象（构造函数），绝大多数浏览器都可以用，它使得我们可以从 JavaScript 来发送 HTTP 请求。发送一个请求有以下三步：

1.  初始化一个 XMLHttpRequest 对象（简称 XHR）
2.  提供一个回调函数，供请求对象状态改变时调用
3.  发送请求

第一步很简单：

```js
var xhr = new XMLHttpRequest(); 
```

但是在 IE7 之前的版本中，XHR 的功能是使用 ActiveX 对象实现的，所以需要做一下兼容处理。

第二步是给 readystatechange 事件提供一个回调函数：

```js
xhr.onreadystatechange = handleResponse; 
```

最后一步是使用 open()和 send()两个方法触发请求。open()方法用于初始化 HTTP 请求的方法（如 GET，POST）和 URL。send()方法用于传递 POST 的数据，如果是 GET 方法，则是一个空字符串。open()方法的最后一个参数用于指定这个请求是不是异步的。异步是指浏览器在等待响应的时候不会阻塞，这明显是更好的用户体验，因此除非必须要同步，否则异步参数应该使用 true：

```js
xhr.open("GET", "page.html", true);
xhr.send(); 
```

下面是一个完整的示例，它获取新页面的内容，然后将当前页面的内容替换掉（可以在<[`jspatterns.com/`](http://jspatterns.com/) book/8/xhr.html>看到示例）：

```js
var i, xhr, activeXids = [
    'MSXML2.XMLHTTP.3.0',
    'MSXML2.XMLHTTP',
    'Microsoft.XMLHTTP'
];

if (typeof XMLHttpRequest === "function") { // native XHR
    xhr = new XMLHttpRequest();
} else { // IE before 7
    for (i = 0; i < activeXids.length; i += 1) {
        try {
            xhr = new ActiveXObject(activeXids[i]);
            break;
        } catch (e) {}
    }
}

xhr.onreadystatechange = function () {
    if (xhr.readyState !== 4) {
        return false;
    }
    if (xhr.status !== 200) {
        alert("Error, status code: " + xhr.status);
        return false;
    }
    document.body.innerHTML += "<pre>" + xhr.responseText + "<\/pre>"; };

xhr.open("GET", "page.html", true);
xhr.send(""); 
```

代码中的一些说明：

*   因为 IE6 及以下版本中，创建 XHR 对象有一点复杂，所以我们通过一个数组列出 ActiveX 的名字，然后遍历这个数组，使用 try-catch 块来尝试创建对象。
*   回调函数会检查 xhr 对象的 readyState 属性。这个属性有 0 到 4 一共 5 个值，4 代表“complete”（完成）。如果状态还没有完成，我们就继续等待下一次 readystatechange 事件。
*   回调函数也会检查 xhr 对象的 status 属性。这个属性和 HTTP 状态码对应，比如 200（OK）或者是 404（Not found）。我们只对状态码 200 感兴趣，而将其它所有的都报为错误（为了简化示例，否则需要检查其它不代表出错的状态码）。
*   上面的代码会在每次创建 XHR 对象时检查一遍支持情况。你可以使用前面提到过的模式（如条件初始化）来重写上面的代码，使得只需要做一次检查。

### JSONP

JSONP（JSON with padding）是另一种发起远程请求的方式。与 XHR 不同，它不受浏览器同源策略的限制，所以考虑到加载第三方站点的安全影响的问题，使用它时应该很谨慎。

一个 XHR 请求的返回可以是任何类型的文档：

*   XML 文档（过去很常用）
*   HTML 片段（很常用）
*   JSON 数据（轻量、方便）
*   简单的文本文件及其它

使用 JSONP 的话，数据经常是被包裹在一个函数中的 JSON，函数名称在请求的时候提供。

JSONP 的请求 URL 通常是像这样：

```js
http://example.org/getdata.php?callback=myHandler 
```

getdata.php 可以是任何类型的页面或者脚本。callback 参数指定用来处理响应的 JavaScript 函数。

这个 URL 会被放到一个动态生成的\<scriptu0005c class="calibre26">元素中，像这样：</scriptu0005c>

```js
var script = document.createElement("script");
script.src = url;
document.body.appendChild(script); 
```

服务器返回一些作为参数传递给回调函数的 JSON 数据。最终的结果实际上是页面中多了一个新的脚本，这个脚本的内容就是一个函数调用，如：

```js
myHandler({"hello": "world"}); 
```

（译注：原文这里说得不是太明白。JSONP 的返回内容如上面的代码片段，它的工作原理是在页面中动态插入一个脚本，这个脚本的内容是函数调用+JSON 数据，其中要调用的函数是在页面中已经定义好的，数据以参数的形式存在。一般情况下数据由服务端动态生成，而函数由页面生成，为了使返回的脚本能调用到正确的函数，在请求的时候一般会带上 callback 参数以便后台动态返回处理函数的名字。）

#### JSONP 示例：井字棋

我们来看一个使用 JSONP 的井字棋游戏示例，玩家就是客户端（浏览器）和服务器。它们两者都会产生 1 到 9 之间的随机数，我们使用 JSONP 去取服务器产生的数字（图 8-2）。

你可以在[`jspatterns.com/book/8/ttt.html`](http://jspatterns.com/book/8/ttt.html)玩这个游戏。

![图 8-2 使用 JSONP 的井字棋游戏](img/8-2.jpg)

图 8-2 使用 JSONP 的井字棋游戏

界面上有两个按钮：一个用于开始新游戏，一个用于取服务器下的棋（客户端下的棋会在一定数量的延时之后自动进行）：

```js
<button id="new">New game</button>
<button id="server">Server play</button> 
```

界面上包含 9 个单元格，每个都有对应的 id，比如：

```js
<td id="cell-1">&nbsp;</td>
<td id="cell-2">&nbsp;</td>
<td id="cell-3">&nbsp;</td>
... 
```

整个游戏是在一个全局对象 ttt 中实现：

```js
var ttt = {
    // cells played so far
    played: [],

    // shorthand
    get: function (id) {
        return document.getElementById(id);
    },

    // handle clicks
    setup: function () {
        this.get('new').onclick = this.newGame;
        this.get('server').onclick = this.remoteRequest;
    },

    // clean the board
    newGame: function () {
        var tds = document.getElementsByTagName("td"),
            max = tds.length,
            i;
        for (i = 0; i < max; i += 1) {
            tds[i].innerHTML = "&nbsp;";
        }
        ttt.played = [];
    },

    // make a request
    remoteRequest: function () {
        var script = document.createElement("script");
        script.src = "server.php?callback=ttt.serverPlay&played=" + ttt.played.join(',');
        document.body.appendChild(script);
    },

    // callback, server's turn to play
    serverPlay: function (data) {
        if (data.error) {
            alert(data.error);
            return;
        }

        data = parseInt(data, 10);
        this.played.push(data);

        this.get('cell-' + data).innerHTML = '<span class="server">X<\/span>';

        setTimeout(function () {
            ttt.clientPlay();
        }, 300); // as if thinking hard
    },

    // client's turn to play
    clientPlay: function () {
        var data = 5;

        if (this.played.length === 9) {
            alert("Game over");
            return;
        }

        // keep coming up with random numbers 1-9
        // until one not taken cell is found
        while (this.get('cell-' + data).innerHTML !== "&nbsp;") {
            data = Math.ceil(Math.random() * 9);
        }
        this.get('cell-' + data).innerHTML = 'O';
        this.played.push(data);
    } 
}; 
```

ttt 对象维护着一个已经填过的单元格的列表 ttt.played，并且将它发送给服务器，这样服务器就可以返回一个没有玩过的数字。如果有错误发生，服务器会像这样响应：

```js
ttt.serverPlay({"error": "Error description here"}); 
```

如你所见，JSONP 中的回调函数必须是公开的并且全局可访问的函数，它并不一定要是全局函数，也可以是一个全局对象的方法。如果没有错误发生，服务器将会返回一个函数调用，像这样：

```js
ttt.serverPlay(3); 
```

这里的 3 是指 3 号单元格是服务器要下棋的位置。在这种情况下，数据非常简单，甚至都不需要使用 JSON 格式，只需要一个简单的值就可以了。

### 框架（frame）和图片信标(image beacon)

另外一种做远程脚本编程的方式是使用框架。你可以使用 JavaScript 来创建框架并改变它的 src URL。新的 URL 可以包含数据和函数调用来更新调用者，也就是框架之外的父页面。

远程脚本编程中最最简单的情况是你只需要传递一点数据给服务器，而并不需要服务器的响应内容。在这种情况下，你可以创建一个新的图片，然后将它的 src 指向服务器的脚本：

```js
new Image().src = "http://example.org/some/page.php"; 
```

这种模式叫作图片信标，当你想发送一些数据给服务器记录时很有用，比如做访问统计。因为信标的响应对你来说完全是没有用的，所以通常的做法（不推荐）是让服务器返回一个 1x1 的 GIF 图片。更好的做法是让服务器返回一个“204 No Content”HTTP 响应。这意味着返回给客户端的响应只有响应头（header）而没有响应体（body）。

## 部署 JavaScript

在生产环境中使用 JavaScript 时，有不少性能方面的考虑。我们来讨论一下最重要的一些。如果需要了解所有的细节，可以参见 O'Reilly 出社的《高性能网站建设指南》和《高性能网站建设进阶指南》。

### 合并脚本

创建高性能网站的第一个原则就是尽量减少外部引用的组件（译注：这里指文件），因为 HTTP 请求的代价是比较大的。具体就 JavaScript 而言，可以通过合并外部脚本来显著提高页面加载速度。

我们假设你的页面正在使用 jQuery 库，这是一个.js 文件。然后你使用了一些 jQuery 插件，这些插件也是单独的文件。这样的话在你还一行代码都没有写的时候就已经有了四五个文件了。把这些文件合并起来是很有意义的，尤其是其中的一些体积很小（2-3kb）时，这种情况下，HTTP 协议中的开销会比下载本身还大。合并脚本的意思就是简单地创建一个新的 js 文件，然后把每个文件的内容粘贴进去。

当然，合并的操作应该放在代码部署到生产环境之前，而不是在开发环境中，因为这会使调试变得困难。

合并脚本的不便之处是：

*   在部署前多了一步操作，但这很容易使用命令行自动化工具来做，比如使用 Linux/Unix 的 cat：

    ```js
     $ cat jquery.js jquery.quickselect.js jquery.limit.js > all.js 
    ```

*   失去一些缓存上的便利——当你对某个文件做了一点小修改之后，会使得整个合并后的代码缓存失效。所以比较好的方法是为大的项目设定一个发布计划，或者是将代码合并为两个文件：一个包含可能会经常变更的代码，另一个包含那些不会轻易变更的“核心”。
*   你需要处理合并后文件的命名或者是版本问题，比如使用一个时间戳 all_20100426.js 或者是使用文件内容的 hash 值。

这就是主要的不便之处，但它带来的好处却是远远大于这些麻烦的。

### 压缩代码

第二章中，我们讨论过代码压缩。部署之前进行代码压缩也是一个很重要的步骤。

从用户的角度来想，完全没有必要下载代码中的注释，因为这些注释根本不影响代码运行。

压缩代码带来的好处多少取决于代码中注释和空白的数量，也取决于你使用的压缩工具。平均来说，压缩可以减少 50%左右的体积。

服务端脚本压缩也是应该要做的事情。配置启用 gzip 压缩是一个一次性的工作，能带来立杆见影的速度提升。即使你正在使用共享的空间，供应商并没有提供那么多服务器配置的空间，大部分的供应商也会允许使用.htaccess 配置文件。所以可以将这些加入到站点根目录的.htaccess 文件中：

```js
AddOutputFilterByType DEFLATE text/html text/css text/plain text/xml application/javascript application/json 
```

平均下来压缩会节省 70%的文件体积。将代码压缩和服务端压缩合计起来，你可以期望你的用户只下载你写出来的未压缩文件体积的 15%。

### 缓存头

与流行的观点相反，文件在浏览器缓存中的时间并没有那么久。你可以尽你自己的努力，通过使用 Expires 头来增加非首次访问时命中缓存的概率：

这也是一个在.htaccess 中做的一次性配置工作：

```js
ExpiresActive On
ExpiresByType application/x-javascript "access plus 10 years" 
```

它的弊端是当你想更改这个文件时，你需要给它重命名，如果你已经处理好了合并的文件命名规则，那你就已经处理好这里的命名问题了。

### 使用 CDN

CDN 是指“文件分发网络”（Content Delivery Network）。这是一项收费（有时候还相当昂贵）的托管服务，它将你的文件分发到世界上各个不同的数据中心，但代码中的 URL 却都是一样的，这样可以使用户更快地访问。

即使你没有 CDN 的预算，你仍然有一些可以免费使用的东西：

*   Google 托管了很多流行的开源库，你可以免费使用，并从它的 CDN 中得到速度提升（译注：鉴于 Google 在国内的尴尬处境，不建议使用）
*   微软托管了 jQuery 和自家的 Ajax 库
*   雅虎在自己的 CDN 上托管了 YUI 库

## 加载策略

怎样在页面上引入脚本，这第一眼看起来是一个简单的问题——使用\<scriptu0005c class="calibre26">元素，然后要么写内联的 JavaScript 代码或者是在 src 属性中指定一个独立的文件：</scriptu0005c>

```js
// option 1
<script>
console.log("hello world"); </script>
// option 2
<script src="external.js"></script> 
```

但是，当你的目标是要构建一个高性能的 web 应用的时候，有些模式和考虑点还是应该知道的。

作为题外话，来看一些比较常见的开发者会用在\<scriptu0005c class="calibre26">元素上的属性：</scriptu0005c>

*   language="JavaScript"

    还有一些不同大小写形式的“JavaScript”，有的时候还会带上一个版本号。language 属性不应该被使用，因为默认的语言就是 JavaScript。版本号也不像想象中工作得那么好，这应该是一个设计上的错误。

*   type="text/javascript"

    这个属性是 HTML4 和 XHTML1 标准所要求的，但它不应该存在，因为浏览器会假设它就是 JavaScript。HTML5 不再要求这个属性。除非是要强制通过难，否则没有任何使用 type 的理由。

*   defer

    （或者是 HTML5 中更好的 async）是一种指定浏览器在下载外部脚本时不阻塞页面其它部分的方法，但还没有被广泛支持。关于阻塞的更多内容会在后面提及。

### \<scriptu0005c class="calibre27">元素的位置</scriptu0005c>

script 元素会阻塞页面的下载。浏览器会同时下载好几个组件（文件），但遇到一个外部脚本的时候，会停止其它的下载，直到脚本文件被下载、解析、执行完毕。这会严重影响页面的加载时间，尤其是当这样的事件在页面加载时发生多次的时候。

为了尽量减小阻塞带来的影响，你可以将 script 元素放到页面的尾部，在\</body>之前，这样就没有可以被脚本阻塞的元素了。此时，页面中的其它组件（文件）已经被下载完毕并呈现给用户了。

最坏的“反模式”是在文档的头部使用独立的文件：

```js
<!doctype html>
<html>
<head>
    <title>My App</title>
    <!-- ANTIPATTERN -->
    <script src="jquery.js"></script>
    <script src="jquery.quickselect.js"></script>
    <script src="jquery.lightbox.js"></script>
    <script src="myapp.js"></script>
</head>
<body>
    ...
</body>
</html> 
```

一个更好的选择是将所有的文件合并起来：

```js
<!doctype html>
<html>
<head>
    <title>My App</title>
    <script src="all_20100426.js"></script>
</head>
<body>
    ...
</body>
</html> 
```

最好的选择是将合并后的脚本放到页面的尾部：

```js
<!doctype html>
<html>
<head>
    <title>My App</title>
</head>
<body>
    ...
    <script src="all_20100426.js"></script>
</body>
</html> 
```

### HTTP 分块

HTTP 协议支持“分块编码”。它允许将页面分成一块一块发送。所以如果你有一个很复杂的页面，你不需要将那些每个站都多多少少会有的（静态）头部信息也等到所有的服务端工作都完成后再开始发送。

一个简单的策略是在组装页面其余部分的时候将页面\ 的内容作为第一块发送。也就是像这样子：

```js
<!doctype html>
<html>
<head>
    <title>My App</title>
</head>
<!-- end of chunk #1 -->
<body>
    ...
    <script src="all_20100426.js"></script> </body>
</html>
<!-- end of chunk #2 --> 
```

这种情况下可以做一个简单的发动，将 JavaScript 移回\<headu0005c class="calibre26">，随着第一块一起发送。</headu0005c>

这样的话可以让服务器在拿到 head 区内容后就开始下载脚本文件，而此时页面的其它部分在服务端还尚未就绪：

```js
<!doctype html>
<html>
<head>
    <title>My App</title>
    <script src="all_20100426.js"></script> </body>
</head>
<!-- end of chunk #1 -->
<body>
    ...
</html>
<!-- end of chunk #2 --> 
```

一个更好的办法是使用第三块内容，让它在页面尾部，只包含脚本。如果有一些每个页面都用到的静态的头部，也可以将这部分随和一块一起发送：

```js
<!doctype html> <html>
<head>
    <title>My App</title> </head>
<body>
    <div id="header">
        <img src="logo.png" />
        ...
    </div>
    <!-- end of chunk #1 -->

    ... The full body of the page ...

    <!-- end of chunk #2 -->
    <script src="all_20100426.js"></script>
</body>
</html>
<!-- end of chunk #3 --> 
```

这种方法很适合使用渐进增强思想的网站（关键业务不依赖 JavaScript）。当 HTML 的第二块发送完毕的时候，浏览器已经有了一个加载、显示完毕并且可用的页面，就像禁用 JavaScript 时的情况。当 JavaScript 随着第三块到达时，它会进一步增强页面，为页面锦上添花。

### 动态\<scriptu0005c class="calibre27">元素实现非阻塞下载</scriptu0005c>

前面已经说到过，JavaScript 会阻塞后面文件的下载，但有一些模式可以防止阻塞：

*   使用 XHR 加载脚本，然后作为一个字符串使用 eval()来执行。这种方法受同源策略的限制，而且引入了 eval()这种“反模式”。
*   使用 defer 和 async 属性，但有浏览器兼容性问题
*   使用动态\<scriptu0005c class="calibre28">元素</scriptu0005c>

最后一种是一个很好并且实际可行的模式。和介绍 JSONP 时所做的一样，创建一个新的 script 元素，设置它的 src 属性，然后将它放到页面上。

这是一个异步加载 JavaScript，不阻塞其它文件下载的示例：

```js
var script = document.createElement("script");
script.src = "all_20100426.js";
document.documentElement.firstChild.appendChild(script); 
```

这种模式的缺点是，在这之后加载的脚本不能依赖这个脚本。因为这个脚本是异步加载的，所以无法保证它什么时候会被加载进来，如果要依赖的话，很可能会访问到（因还未加载完毕导致的）未定义的对象。

如果要解决这个问题，可以让内联的脚本不立即执行，而是作为一个函数放到一个数组中。当依赖的脚本加载完毕后，再执行数组中的所有函数。所以一共有三个步骤。

首先，创建一个数组用来存储所有的内联代码，定义的位置尽量靠前：

```js
var mynamespace = {
    inline_scripts: []
}; 
```

然后你需要将这些单独的内联脚本包裹进一个函数中，然后将每个函数放到 inline_scripts 数组中，也就是这样：

```js
// was:
// <script>console.log("I am inline");</script>

// becomes:
<script>
    mynamespace.inline_scripts.push(function () {
        console.log("I am inline");
    });
</script> 
```

最后一步是使用异步加载的脚本遍历这个数组，然后执行函数：

```js
var i, scripts = mynamespace.inline_scripts, max = scripts.length;
for (i = 0; i < max; max += 1) {
    scripts[i]();
} 
```

#### 插入\<scriptu0005c class="calibre27">元素</scriptu0005c>

通常脚本是插入到文档的中的，但其实你可以插入任何元素中，包括 body（像 JSONP 示例中那样）。

在前面的例子中，我们使用 documentElement 来插到\<headu0005c class="calibre26">中，因为 documentElement 就是\<htmlu0005c class="calibre26">，它的第一个子元素是\<headu0005c class="calibre26">：</headu0005c></htmlu0005c></headu0005c>

```js
document.documentElement.firstChild.appendChild(script); 
```

通常也会这样写：

```js
document.getElementsByTagName("head")[0].appendChild(script); 
```

当你能控制结构的时候，这样做没有问题，但是如果你在写挂件（widget）或者是广告时，你并不知道托管它的是一个什么样的页面。甚至可能页面上连\<headu0005c class="calibre26">和\<bodyu0005c class="calibre26">都没有，尽管 document.body 在绝大多数没有\<bodyu0005c class="calibre26">标签的时候也可以工作：</bodyu0005c></bodyu0005c></headu0005c>

```js
document.body.appendChild(script); 
```

可以肯定页面上一定存在的一个标签是你正在运行的脚本所处的位置——script 标签。（对内联或者外部文件来说）如果没有 script 标签，那么代码就不会运行。可以利用这一事实，在页面的第一个 script 标签上使用 insertBefore()：

```js
var first_script = document.getElementsByTagName('script')[0];
first_script.parentNode.insertBefore(script, first_script); 
```

frist_script 是页面中一定存在的一个 script 标签，script 是你创建的新的 script 元素。

### 延迟加载

所谓的延迟加载是指在页面的 load 事件之后再加载外部文件。通常，将一个大的合并后的文件分成两部分是有好处的：

*   一部分是页面初始化和绑定 UI 元素的事件处理函数必须的
*   第二部分是只在用户交互或者其它条件下才会用到的

目标就是逐步加载页面，让用户尽快可以进行一些操作。剩余的部分可以在用户可以看到页面的时候再在后台加载。

加载第二部分 JavaScript 的方法也是使用动态 script 元素，将它加在 head 或者 body 中：

```js
 .. The full body of the page ...

    <!-- end of chunk #2 -->
    <script src="all_20100426.js"></script>
    <script>
    window.onload = function () {
        var script = document.createElement("script");
        script.src = "all_lazy_20100426.js";
        document.documentElement.firstChild.appendChild(script);
    };
    </script>
</body>
</html>
<!-- end of chunk #3 --> 
```

对很多应用来说，延迟加载的部分大部分情况下会比核心部分要大，因为我们关注的“行为”（比如拖放、XHR、动画）只在用户初始化之后才会发生。

### 按需加载

前面的模式会在页面加载后无条件加载其它的 JavaScript，并假设这些代码很可能会被用到。但我们是否可以做得更好，分部分加载，在真正需要使用的时候才加载那一部分？

假设你页面的侧边栏上有一些 tabs。点击 tab 会发出一个 XHR 请求获取内容，然后更新 tab 的内容，然后有一个更新的动画。如果这是页面上唯一需要 XHR 和动画库的地方，而用户又不点击 tab 的话会怎样？

下面介绍按需加载模式。你可以创建一个 require()函数或者方法，它接受一个需要被加载的脚本文件的文件名，还有一个在脚本被加载完毕后执行的回调函数。

require()函数可以被这样使用：

```js
require("extra.js", function () {
    functionDefinedInExtraJS();
}); 
```

我们来看一下如何实现这样一个函数。加载脚本很简单——你只需要按照动态\<scriptu0005c class="calibre26">元素模式做就可以了。获知脚本已经加载需要一点点技巧，因为浏览器之间有差异：</scriptu0005c>

function require(file, callback) {

```js
var script = document.getElementsByTagName('script')[0], newjs = document.createElement('script');

// IE
newjs.onreadystatechange = function () {
    if (newjs.readyState === 'loaded' || newjs.readyState === 'complete') {
        newjs.onreadystatechange = null;
        callback();
    }
};

// others
newjs.onload = function () {
    callback();
};

newjs.src = file;
script.parentNode.insertBefore(newjs, script); 
```

}

这个实现的几点说明：

*   在 IE 中需要订阅 readystatechange 事件，然后判断状态是否为“loaded”或者“complete”。其它的浏览器会忽略这里。
*   在 Firefox，Safari 和 Opera 中，通过 onload 属性订阅 load 事件。
*   这个方法在 Safari 2 中无效。如果必须要处理这个浏览器，需要设一个定时器，周期性地去检查某个指定的变量（在脚本中定义的）是否有定义。当它变成已定义时，就意味着新的脚本已经被加载并执行。

你可以通过建立一个人为延迟的脚本来测试这个实现（模拟网络延迟），比如 ondemand.js.php，如：

```js
<?php
header('Content-Type: application/javascript');
sleep(1);
?>
function extraFunction(logthis) {
    console.log('loaded and executed');
    console.log(logthis);
} 
```

现在测试 require()函数：

```js
require('ondemand.js.php', function () {
    extraFunction('loaded from the parent page');
    document.body.appendChild(document.createTextNode('done!'));
}); 
```

这段代码会在 console 中打印两条，然后页面中会显示“done!”，你可以在[`jspatterns.com/book/7/ondemand.html`](http://jspatterns.com/book/7/ondemand.html)看到示例。

### 预加载 JavaScript

在延迟加载模式和按需加载模式中，我们加载了当前页面需要用到的脚本。除此之外，我们也可以加载当前页面不需要但可能在接下来的页面中需要的脚本。这样的话，当用户进入第二个页面时，脚本已经被预加载过，整体体验会变得更快。

预加载可以简单地通过动态脚本模式实现。但这也意味着脚本会被解析和执行。解析仅仅会在页面加载时间中增加预加载消耗的时间，但执行却可能导致 JavaScript 错误，因为预加载的脚本会假设自己运行在第二个页面上，比如找一个特写的 DOM 节点就可能出错。

仅加载脚本而不解析和执行是可能的，这也同样适用于 CSS 和图像。

在 IE 中，你可以使用熟悉的图片信标模式来发起请求：

```js
new Image().src = "preloadme.js"; 
```

在其它的浏览器中，你可以使用\<objectu0005c class="calibre26">替代 script 元素，然后将它的 data 属性指向脚本的 URL：</objectu0005c>

```js
var obj = document.createElement('object');
obj.data = "preloadme.js";
document.body.appendChild(obj); 
```

为了阻止 object 可见，你应该设置它的 width 和 height 属性为 0。

你可以创建一个通用的 preload()函数或者方法，使用条件初始化模式（第四章）来处理浏览器差异：

```js
var preload;
if (/*@cc_on!@*/false) { // IE sniffing with conditional comments
    preload = function (file) {
        new Image().src = file;
    };
} else {
    preload = function (file) {
        var obj = document.createElement('object'),
            body = document.body;

        obj.width = 0;
        obj.height = 0;
        obj.data = file;
        body.appendChild(obj);
    };
} 
```

使用这个新函数：

```js
preload('my_web_worker.js'); 
```

这种模式的坏处在于存在用户代理（浏览器）嗅探，但这里无法避免，因为特性检测没有办法告知足够的浏览器行为信息。比如在这个模式中，理论上你可以测试 typeof Image 是否是“function”来代替嗅探。但这种方法其实没有作用，因为所有的浏览器都支持 new Image();只是有一些浏览器会为图片单独做缓存，意味着作为图片缓存下来的组件（文件）在第二个页面中不会被作为脚本取出来，而是会重新下载。

> 浏览器嗅探中使用条件注释很有意思，这明显比在 navigator.userAgent 中找字符串要安全得多，因为用户可以很容易地修改这些字符串。 比如： var isIE = /*@cc_on!@*/false; 会在其它的浏览器中将 isIE 设为 false（因为忽略了注释），但在 IE 中会是 true，因为在条件注释中有取反运算符!。在 IE 中就像是这样： var isIE = !false; // true

预加载模式可以被用于各种组件（文件），而不仅仅是脚本。比如在登录页就很有用。当用户开始输入用户名时，你可以使用打字的时间开始预加载（非敏感的东西），因为用户很可能会到第二个也就是登录后的页面。

## 小结

在前一章中我们讨论了 JavaScript 核心的模式，它们与环境无关，这一章主要关注了只在客户端浏览器环境中应用的模式。

我们看了：

*   分离的思想（HTML：内容，CSS：表现，JavaScript：行为），只用于增强体验的 JavaScript 以及基于特性检测的浏览器探测。（尽管在本章的最后你看到了如何打破这个模式。）
*   DOM 编程——加速 DOM 访问和操作的模式，主要通过将 DOM 操作集中在一起来实现，因为频繁和 DOM 打交道代码是很高的。
*   事件，跨浏览器的事件处理，以及使用事件代码来减少事件处理函数的绑定数量以提高性能。
*   两种处理长时间大计算量脚本的模式——使用 setTimeout()将长时间操作拆分为小块执行和在现代浏览器中使用 web workers。
*   多种用于远程编程，进行服务器和客户端通讯的模式——XHR，JSONP，框架和图片信标。
*   在生产环境中部署 JavaScript 的步骤——将脚本合并为更少的文件，压缩和 gzip（总共节省 85%），可能的话托管到 CDN 并发送 Expires 头来提升缓存效果。
*   基于性能考虑引入页面脚本的模式，包括：放置\<scriptu0005c class="calibre28">元素的位置，同时也可以从 HTTP 分块获益。为了减少页面初始化时加载大的脚本文件引起的初始化工作量，我们讨论了几种不同的模式，比如延迟加载、预加载和按需加载。</scriptu0005c>