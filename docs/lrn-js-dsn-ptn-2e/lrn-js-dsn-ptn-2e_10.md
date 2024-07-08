# 第十章：模块化 JavaScript 设计模式

在可扩展 JavaScript 的世界中，当我们说一个应用程序是 *模块化* 的时候，通常意味着它由一组高度解耦且独立的功能模块组成。松散耦合通过尽可能去除 *依赖关系*，有助于更轻松地维护应用程序。当有效实施时，很容易看出系统的一部分变更如何影响另一部分。

在前几章中，我们讨论了模块化编程的重要性以及实现现代模块化设计模式的方式。虽然 [ES2015](https://oreil.ly/Pcc5o) 向 JavaScript 引入了原生模块，但在 2015 年之前，编写模块化 JavaScript 仍然是可能的。

在本节中，我们将探讨使用经典 JavaScript（ES5）语法的三种模块化 JavaScript 格式：异步模块定义（AMD）、CommonJS 和通用模块定义（UMD）。要了解更多关于 JavaScript 模块的信息，请参阅 第五章，其中涵盖了 ES2015+ 语法用于模块的导入、导出等。

# 脚本加载器注意事项

谈论 AMD 和 CommonJS 模块时，很难不谈论 [脚本加载器](https://oreil.ly/ssCQT)。脚本加载是一种达到目标的手段。只有使用兼容的脚本加载器才能实现模块化 JavaScript。

有几个优秀的加载器可用于处理 AMD 和 CommonJS 格式的模块加载，但我个人更喜欢 [RequireJS](https://oreil.ly/Ri_9R) 和 [curl.js](https://oreil.ly/s7QRg)。

# AMD

AMD 格式作为定义模块的提案被引入，其中模块和依赖项可以异步加载。AMD 格式的总体目标是为开发人员提供可以使用的模块化 JavaScript 解决方案。它具有几个明显的优点，包括异步加载和高度灵活，这消除了代码与模块标识之间常见的紧密耦合。许多开发人员喜欢使用 AMD，并且可以认为它是通向当时不可用的 JavaScript 模块的可靠过渡阶段。

AMD 最初作为 CommonJS 列表上的一个模块格式草案，但由于无法达成全面共识，该格式的进一步开发移至 [amdjs 组](https://oreil.ly/0-XeU)。

它被包括 Dojo、MooTools 甚至 jQuery 在内的项目所采纳。尽管在野外偶尔可以见到 *CommonJS AMD 格式* 这个术语，但最好将其称为仅仅是 AMD 或异步模块支持，因为并非所有 CommonJS 列表上的参与者都希望追求它。

###### 注意

曾经有一段时间，这个提案被称为“模块传输/C”。然而，由于规范并非面向传输现有的 CommonJS 模块，而是用于定义模块，因此选择 AMD 命名约定更为合理。

## 模块入门

关于 AMD 值得注意的前两个概念是 `define` 方法用于简化模块定义和 `require` 方法用于处理依赖项加载。`define` 用于定义具名或匿名模块，使用以下签名：

```
define(
    module_id /*optional*/,
    [dependencies] /*optional*/,
    definition function {} /*function for instantiating the module or object*/
);
```

如内联注释所示，`module_id` 是一个可选参数，通常仅在使用非 AMD 连接工具时才需要（可能还有一些其他情况下它也很有用）。当省略此参数时，我们将模块称为 *匿名*。

在处理匿名模块时，模块身份的概念是 DRY（不要重复自己），这使得避免文件名和代码重复变得轻而易举。由于代码更具可移植性，可以轻松地将其移动到其他位置（或者文件系统中的其他位置）而无需修改代码本身或更改其模块 ID。将 `module_id` 视为类似于文件夹路径的概念。

###### 注意

开发人员可以在多个环境中使用 AMD 优化器运行相同的代码，该优化器适用于 CommonJS 环境，如 [r.js](https://oreil.ly/48dSL)。

回到 `define` 的签名，`dependencies` 参数表示我们正在定义的模块所需的一组依赖项数组，第三个参数（`definition` `function` 或 `factory function`）是一个执行以实例化我们的模块的函数。一个最简单的模块可以像 示例 10-1 中定义的那样。

##### 示例 10-1\. 理解 AMD：`define()`

```
// A module_id (myModule) is used here for demonstration purposes only
define( "myModule",

    ["foo", "bar"],

    // module definition function
    // dependencies (foo and bar) are mapped to function parameters
    function ( foo, bar ) {
        // return a value that defines the module export
        // (i.e., the functionality we want to expose for consumption)

        // create your module here
        var myModule = {
            doStuff:function () {
                console.log( "Yay! Stuff" );
            }
        };

    return myModule;
});

// An alternative version could be...
define( "myModule",

    ["math", "graph"],

    function ( math, graph ) {

        // Note that this is a slightly different pattern
        // With AMD, it's possible to define modules in a few
        // different ways due to its flexibility with
        // certain aspects of the syntax
        return {
            plot: function( x, y ){
                return graph.drawPie( math.randomGrid( x, y ) );
            }
        };
});
```

另一方面，`require` 通常用于在顶层 JavaScript 文件中加载代码或者在模块内部加载依赖项。其使用示例在 示例 10-2 中。

##### 示例 10-2\. 理解 AMD：`require()`

```
// Consider "foo" and "bar" are two external modules
// In this example, the "exports" from the two modules
// loaded are passed as function arguments to the
// callback (foo and bar) so that they can similarly be accessed

require(["foo", "bar"], function ( foo, bar ) {
        // rest of your code here
        foo.doSomething();
});
```

示例 10-3 展示了动态加载的依赖：

##### 示例 10-3\. 动态加载的依赖

```
define(function ( require ) {
    var isReady = false, foobar;

    // note the inline require within our module definition
    require(["foo", "bar"], function ( foo, bar ) {
        isReady = true;
        foobar = foo() + bar();
    });

    // we can still return a module
    return {
        isReady: isReady,
        foobar: foobar
    };
});
```

示例 10-4 展示了如何定义一个兼容 AMD 的插件。

##### 示例 10-4\. 理解 AMD：插件

```
// With AMD, it's possible to load in assets of almost any kind
// including text-files and HTML. This enables us to have template
// dependencies which can be used to skin components either on
// page-load or dynamically.

define( ["./templates", "text!./template.md","css!./template.css" ],

    function( templates, template ){
        console.log( templates );
        // do something with our templates here
    }

});
```

###### 注意

尽管在前面的示例中包含 `css!` 用于加载层叠样式表（CSS）依赖，但重要的是要记住，这种方法有一些注意事项，例如无法确定 CSS 是否完全加载。根据我们的构建过程的不同方法，这可能导致 CSS 作为优化文件的依赖项被包含在内，因此在这种情况下谨慎使用 CSS 作为加载的依赖项。如果您有兴趣尝试这样做，我们可以探索 [@VIISON’s RequireJS CSS plug-in](https://oreil.ly/PrLim)。

这个例子可以简单地被看作是`requirejs(["app/myModule"], function(){})`，这表明顶级加载器正在被使用。这是如何启动具有不同 AMD 加载器的顶级模块加载的方式。然而，如果`define()`函数作为本地 require 传递，所有`require([])`示例都适用于 curl.js 和 RequireJS（示例 10-5 和 10-6）。

##### 示例 10-5\. 使用 RequireJS 加载 AMD 模块

```
require(["app/myModule"],

    function( myModule ){
        // start the main module which in turn
        // loads other modules
        var module = new myModule();
        module.doStuff();
});
```

##### 示例 10-6\. 使用 curl.js 加载 AMD 模块

```
curl(["app/myModule.js"],

    function( myModule ){
        // start the main module which in turn
        // loads other modules
        var module = new myModule();
        module.doStuff();

});
```

接下来是具有延迟依赖关系的模块代码：

```
<pre xmlns="http://www.w3.org/1999/xhtml" id="I_programlisting11_id234274"
data-type="programlisting" data-code-language="javascript">

// This could be compatible with jQuery's Deferred implementation,
// futures.js (slightly different syntax) or any one of a number
// of other implementations

define(["lib/Deferred"], function( Deferred ){
    var defer = new Deferred();

    require(["lib/templates/?index.html","lib/data/?stats"],
        function( template, data ){
            defer.resolve( { template: template, data:data } );
        }
    );
    return defer.promise();
});

</pre>
```

如前所述，在以往的章节中，设计模式在改善我们处理常见开发问题的解决方案结构化方法上可以发挥极大的效果。[约翰·汉](https://oreil.ly/SrQI5)关于 AMD 模块设计模式的出色演讲，涵盖了单例模式、装饰者模式、中介者模式等等。我强烈推荐查看他的[幻灯片](https://oreil.ly/7koME)。

## 使用 jQuery 的 AMD 模块

jQuery 只有一个文件。然而，考虑到库的插件化特性，我们可以演示如何定义一个使用它的 AMD 模块：

```
// Code in app.js. baseURl set to the lib folder
// containing jquery, jquery.color, and lodash files.
define(["jquery","jquery.color","lodash"], function( $, colorPlugin, _ ){
    // Here we've passed in jQuery, the color plugin, and Lodash
    // None of these will be accessible in the global scope, but we
    // can easily reference them below.

    // Pseudorandomize an array of colors, selecting the first
    // item in the shuffled array
    var shuffleColor = _.first( _.shuffle(["#AAA","#FFF","#111","#F16"]));
    console.log(shuffleColor);

    // Animate the background color of any elements with the class
    // "item" on the page using the shuffled color
    $( ".item" ).animate( {"backgroundColor": shuffleColor } );

    // What we return can be used by other modules
    return function () {};
});
```

但是，这个例子缺少了一些东西，那就是注册概念。

### 将 jQuery 注册为异步兼容模块

jQuery 1.7 中引入的一个关键特性是支持将 jQuery 注册为异步模块。多个兼容的脚本加载器（包括 RequireJS 和 curl）能够使用异步模块格式加载模块，这意味着在使事情运行起来时需要的 hack 更少。

如果开发者希望使用 AMD，并且不希望她的 jQuery 版本泄漏到全局空间，她应该在使用 jQuery 的顶级模块中调用`noConflict`。此外，由于页面上可能存在多个版本的 jQuery，AMD 加载器必须考虑特殊情况，因此 jQuery 只在 AMD 加载器中注册为已识别这些问题的加载器所支持的模块，这些问题由加载器指定的`define.amd.jQuery`表示。RequireJS 和 curl 是两个这样做的加载器。

命名 AMD 为大多数用例提供了一个强大而安全的安全保护：

```
// Account for the existence of more than one global
// instance of jQuery in the document, cater for testing
// .noConflict()

var jQuery = this.jQuery || "jQuery",
$ = this.$ || "$",
originaljQuery = jQuery,
original$ = $;

define(["jquery"] , function ( $ ) {
    $( ".items" ).css( "background","green" );
    return function () {};
});
```

### 为什么 AMD 是编写模块化 JavaScript 的更好选择？

我们现在已经回顾了几个代码示例，展示了 AMD 的功能。它似乎不仅仅是一个典型的模块模式，那么为什么它对于模块化应用开发是更好的选择呢？

+   提供了一个明确的建议，来定义灵活的模块。

+   比目前的全局命名空间和`<script>`标签解决方案干净得多。有一种清晰的方法来声明独立的模块及其可能的依赖关系。

+   模块定义是封装的，帮助我们避免全局命名空间的污染。

+   可以说比某些替代方案（例如，CommonJS，我们很快将会讨论）更有效。它没有跨域、本地或调试问题，并且不依赖于服务器端工具来使用。大多数 AMD 加载器支持在浏览器中加载模块，无需构建过程。

+   提供了一种“传输”方法，可以在单个文件中包含多个模块。其他如 CommonJS 的方法尚未就传输格式达成一致。

+   如果需要的话，可以延迟加载脚本。

###### 注意

大多数提到的观点对于 YUI 的模块加载策略也是有效的。

### 与 AMD 相关的阅读

+   [RequireJS 指南到 AMD](https://oreil.ly/uPEJg)

+   [如何最快地加载 AMD 模块？](https://oreil.ly/Z04H9)

+   [AMD vs. CommonJS，哪个格式更好？](https://oreil.ly/W4Fqi)

+   [未来是模块而不是框架](https://oreil.ly/A9S7c)

+   [AMD 不再是 CommonJS 规范](https://oreil.ly/Tkti9)

+   [关于发明 JavaScript 模块格式和脚本加载器](https://oreil.ly/AB01l)

+   [AMD 邮件列表](https://oreil.ly/jdTYO)

### 支持 AMD 的脚本加载器和框架

浏览器内：

+   [RequireJS](https://oreil.ly/Ri_9R)

+   [curl.js](https://oreil.ly/fi105)

+   [Yabble](https://oreil.ly/oBWDi)

+   [PINF](https://oreil.ly/C28-D)

+   还有更多

服务器端：

+   [RequireJS](https://oreil.ly/Ri_9R)

+   [PINF](https://oreil.ly/TJldu)

## AMD 结论

在几个项目中使用 AMD 后，我得出结论，它符合开发人员从更好的模块格式中可能希望得到的许多需求。它避免了担心全局变量，支持命名模块，不需要服务器转换即可运行，并且在依赖管理方面使用起来非常愉快。

对于使用 Backbone.js、ember.js 或其他结构化框架来保持应用程序组织化的模块化开发，这也是一个很好的补充。

由于 AMD 在 Dojo 和 CommonJS 世界中得到了广泛讨论，我们知道它已经有了时间成熟和演变。我们也知道它已经在实际项目中经过大公司的考验来构建非常规模的应用程序（IBM，BBC iPlayer），所以如果它不起作用，他们很有可能会放弃它，但他们没有。

也就是说，仍然有一些地方可以改进 AMD。使用该格式一段时间的开发人员可能会觉得 AMD 的包装代码是一种烦人的开销。虽然我也有这个担忧，但有一些工具，如[Volo](https://oreil.ly/TLSYv)，帮助解决了这些问题，我认为总体来说，使用 AMD 的利弊远远超过了不利因素。

# CommonJS

CommonJS 模块提案为声明服务器端模块提供了一个简单的 API。与 AMD 不同，它试图涵盖更广泛的问题，如 I/O、文件系统、promises 等。

最初由凯文·丹古尔在 2009 年启动的项目中称为 ServerJS，后来该格式由[CommonJS](https://oreil.ly/EUFt3)，一个志愿工作组正式规范化，旨在设计、原型化和标准化 JavaScript API。他们试图为[模块](https://oreil.ly/v_hsu)和[包](https://oreil.ly/Trgzj)制定标准。

## 入门指南

从结构上看，CommonJS 模块是可重用的 JavaScript 片段，它导出特定对象，供任何依赖代码使用。与 AMD 不同，这些模块通常没有函数包装器（例如，在这里我们不会看到`define`）。

CommonJS 模块包含两个主要部分：一个名为`exports`的自由变量，其中包含模块希望向其他模块提供的对象，以及模块可以使用的`require`函数来导入其他模块的 exports（示例 10-7、10-8 和 10-9）。

##### 示例 10-7\. 理解 CommonJS：`require()` 和 `exports`

```
// package/lib is a dependency we require
var lib = require("package/lib");

// behavior for our module
function foo() {
  lib.log("hello world!");
}

// export (expose) foo to other modules
exports.foo = foo;
```

##### 示例 10-8\. `exports` 的基本使用

```
// Import the module containing the foo function
var exampleModule = require("./example-10-9");

// Consume the 'foo' function from the imported module
exampleModule.foo();
```

在示例 10-8 中，我们首先使用`require()`函数从示例 10-7 导入包含`foo`函数的模块。然后，我们通过从导入的模块调用它来消费`foo`函数，使用`exampleModule.foo()`。

##### 示例 10-9\. 第一个 CommonJS 示例的 AMD 等效

```
// CommonJS module getting started
// AMD-equivalent of CommonJS example
// AMD module format
define(function(require){
var lib = require( "package/lib" );

// some behavior for our module
function foo(){
   lib.log( "hello world!" );
}

// export (expose) foo for other modules
return {
   foobar: foo
};
});
```

这可以通过 AMD 支持的[简化的 CommonJS 包装](https://oreil.ly/IzG9s)功能来完成。

## 使用多个依赖项

*app.js*：

```
var modA = require( "./foo" );
var modB = require( "./bar" );

exports.app = function(){
    console.log( "Im an application!" );
}

exports.foo = function(){
    return modA.helloWorld();
}
```

*bar.js*：

```
exports.name = "bar";
```

*foo.js*：

```
require( "./bar" );
exports.helloWorld = function(){
    return "Hello World!!"
}
```

## Node.js 中的 CommonJS

ES 模块格式已成为封装 JavaScript 代码以便复用的标准格式，但在 Node.js 中默认使用 CommonJS。CommonJS 模块是为[Node.js](https://oreil.ly/4Bh_O)打包 JavaScript 代码的原始方式，尽管从版本 13.2.0 开始，Node.js 稳定支持 ES 模块。

默认情况下，Node.js 将以下内容视为 CommonJS 模块：

+   扩展名为 *.cjs* 的文件

+   当最近的父级*package.json*文件包含值为*commonjs*的顶级字段*type*时，具有扩展名为*.js*的文件

+   当最近的父级*package.json*文件不包含顶级字段*type*时，具有*.js*扩展名的文件

+   具有不是*.mjs*、*.cjs*、*.json*、*.node*或*.js*扩展名的文件

调用`require()`始终使用 CommonJS 模块加载器，而调用`import()`则始终使用 ECMAScript 模块加载器，不考虑在最近的父级*package.json*中配置的 type 值。

许多 Node.js 库和模块使用 CommonJS 编写。为了浏览器支持，所有主流浏览器支持 ES 模块语法，并且你可以在像 React 和 Vue.js 这样的框架中使用 import/export。这些框架使用像 Babel 这样的转译器将 import/export 语法编译成`require()`，而旧版本的 Node.js 本身支持这种方式。如果在 Node 中运行代码，使用 ES6 模块语法编写的库将在底层转译为 CommonJS。

## CommonJS 适合浏览器吗？

有些开发人员认为 CommonJS 更适合服务器端开发，这也是在 ES2015 成为事实标准之前，是否应该使用 AMD 或 CommonJS 存在争议的原因之一。一些反对 CommonJS 的观点是，许多 CommonJS API 涉及服务器导向特性，这些特性在 JavaScript 的浏览器级别可能无法实现，例如*io*、*system*和*js*可以考虑由于其功能的性质而不可实现。

无论如何，了解如何构造 CommonJS 模块非常有用，这样我们就能更好地理解它们在定义可以在各处使用的模块时的适用性。在客户端和服务器都有应用的模块包括验证、转换和模板引擎。一些开发人员在选择使用哪种格式时，选择在可以在服务器端使用模块时使用 CommonJS，而在不是这种情况下则使用 AMD 或 ES2015。

ES2015 和 AMD 模块可以定义更细粒度的东西，如构造函数和函数。CommonJS 模块只能定义对象，如果我们试图从中获取构造函数，则可能会变得繁琐。对于 Node.js 中的新项目，ES2015 模块为服务器端提供了与客户端代码相同的语法，同时也确保了更容易的同构 JavaScript 路径，可以在浏览器或服务器上运行。

尽管这超出了本节的范围，但你可能注意到在讨论 AMD 和 CommonJS 时提到了不同类型的`require`方法。对于类似命名约定的担忧是混淆，社区对于全局`require`函数的优点存在分歧。John Hann 在这里建议，与其称其为`require`，这可能不能实现告知用户全局和内部`require`之间差异的目标，更好地将全局加载器方法命名为其他名称可能更有意义（例如，库的名称）。因此，像 curl.js 这样的加载器使用`curl()`而不是`require`。

## CommonJS 相关阅读

+   [JavaScript 的成长](https://oreil.ly/NeuFT)

+   [关于 CommonJS 的 RequireJS 笔记](https://oreil.ly/Nb-5e)

+   [用 Node.js 和 CommonJS 迈出第一步——创建自定义模块](https://oreil.ly/ZpO5u)

+   [用于浏览器的异步 CommonJS 模块](https://oreil.ly/gJhQA)

+   [CommonJS 邮件列表](https://oreil.ly/rL3C2)

# AMD 和 CommonJS：竞争，但同样有效的标准

AMD 和 CommonJS 都是具有不同终极目标的有效模块格式。

AMD 采用了面向浏览器的开发方法，选择了异步行为和简化的向后兼容性，但它没有任何文件 I/O 的概念。它原生支持在浏览器中运行的对象、函数、构造函数、字符串、JSON 和许多其他类型的模块。它非常灵活。

另一方面，CommonJS 采用了服务器优先的方法，假设同步行为，没有全局*包袱*，并试图满足未来（在服务器上）。我的意思是，因为 CommonJS 支持未包装的模块，它可能感觉更接近 ES2015+ 规范，从而免除了 AMD 强制的 `define()` 包装器。然而，CommonJS 模块仅支持对象作为模块。

## UMD：AMD 和 CommonJS 兼容的插件模块

这些解决方案对希望创建能够在浏览器和服务器端环境中工作的模块的开发人员可能有些欠缺。为了帮助缓解这一问题，James Burke、我和其他几位开发人员创建了[通用模块定义（UMD）](https://oreil.ly/HaHHJ)。

UMD 是一种实验性模块格式，允许定义在客户端和服务器环境中均可运行的模块，并支持写作时的大多数流行脚本加载技术。尽管再次引入（另一种）模块格式的想法可能令人生畏，我们将为了全面性而简要介绍 UMD。

我们开始定义 UMD，看看在 AMD 规范中支持的简化的 CommonJS 包装器。希望以 CommonJS 模块的方式编写模块的开发人员可以使用以下兼容 CommonJS 的格式：

### 基本的 AMD 混合格式

```
define( function ( require, exports, module ){

    var shuffler = require( "lib/shuffle" );

    exports.randomize = function( input ){
        return shuffler.shuffle( input );
    }
});
```

然而，重要的是要注意，如果模块不包含依赖数组并且定义函数包含至少一个参数，那么模块才真正被视为 CommonJS 模块。这在某些设备上（例如 PS3）可能无法正常工作。有关包装器的更多信息，请参阅[RequireJS 文档](https://oreil.ly/7A9k6)。

进一步说，我们希望提供几种不同的模式，这些模式与 AMD 和 CommonJS 兼容，并解决了开发人员在其他环境中希望开发这些模块时遇到的典型兼容性问题。

下面我们可以看到一种变体，允许我们使用 CommonJS、AMD 或浏览器全局来创建模块。

### 使用 CommonJS、AMD 或浏览器全局来创建模块

定义一个名为 `commonJsStrict` 的模块，它依赖于另一个名为 `b` 的模块。文件名暗示了模块的名称，最佳实践是文件名和导出的全局变量具有相同的名称。

如果模块 `b` 在浏览器中也使用相同的样板类型，它将创建一个名为 `.b` 的全局变量供使用。如果我们不希望支持浏览器全局修补程序，我们可以删除 `root` 并将 `this` 作为顶部函数的第一个参数传递：

```
(function ( root, factory ) {
    if ( typeof exports === 'object' ) {
        // CommonJS
        factory( exports, require('b') );
    } else if ( typeof define === 'function' && define.amd ) {
        // AMD. Register as an anonymous module.
        define( ['exports', 'b'], factory);
    } else {
        // Browser globals
        factory( (root.commonJsStrict = {}), root.b );
    }
}(this, function ( exports, b ) {
    //use b in some fashion.

    // attach properties to the exports object to define
    // the exported module properties.
    exports.action = function () {};
}));
```

UMD 仓库包含了各种变体，涵盖了在浏览器中最佳运行的模块、最适合提供导出的模块、在 CommonJS 运行时最佳运行的模块，甚至是最适合定义 jQuery 插件的模块，我们将在下面讨论这些内容。

### 在所有环境中都有效的 jQuery 插件

UMD 提供了两种适用于 jQuery 插件的模式：一种定义适用于 AMD 和浏览器全局的插件，另一种也可以在 CommonJS 环境中使用。jQuery 不太可能在大多数 CommonJS 环境中使用，请记住这一点，除非我们正在处理与之兼容良好的环境。

现在我们将定义一个由核心和该核心的扩展组成的插件。核心插件加载到 `$.core` 命名空间中，可以通过命名空间模式轻松地使用插件扩展（即通过 `script` 标签加载的插件自动填充到 `core` 下的 `plugin` 命名空间中，即 `$.core.plugin.methodName()`）。

这种模式非常好用，因为插件扩展可以访问基础中定义的属性和方法，或者通过一些调整覆盖默认行为，以便进行扩展以实现更多功能。加载器也不需要使这些功能完全可用。

有关正在进行的详细信息，请参阅这些代码示例中的内联注释。

*usage.html:*

```
<script type="text/javascript" src="jquery.min.js"></script>
<script type="text/javascript" src="pluginCore.js"></script>
<script type="text/javascript" src="pluginExtension.js"></script>

<script type="text/javascript">

$(function(){

    // Our plug-in "core" is exposed under a core namespace in
    // this example, which we first cache
    var core = $.core;

    // Then use some of the built-in core functionality to
    // highlight all divs in the page yellow
    core.highlightAll();

    // Access the plug-ins (extensions) loaded into the "plugin"
    // namespace of our core module:

    // Set the first div in the page to have a green background.
    core.plugin.setGreen( "div:first");
    // Here we're making use of the core's "highlight" method
    // under the hood from a plug-in loaded in after it

    // Set the last div to the "errorColor" property defined in
    // our core module/plug-in. If we review the code further down,
    // we can see how easy it is to consume properties and methods
    // between the core and other plug-ins
    core.plugin.setRed("div:last");
});

</script>
```

*pluginCore.js:*

```
// Module/plug-in core
// Note: the wrapper code we see around the module is what enables
// us to support multiple module formats and specifications by
// mapping the arguments defined to what a specific format expects
// to be present. Our actual module functionality is defined lower
// down, where a named module and exports are demonstrated.
//
// Note that dependencies can just as easily be declared if required
// and should work as demonstrated earlier with the AMD module examples.

(function ( name, definition ){
  var theModule = definition(),
      // this is considered "safe":
      hasDefine = typeof define === "function" && define.amd,
      hasExports = typeof module !== "undefined" && module.exports;

  if ( hasDefine ){ // AMD Module
    define(theModule);
  } else if ( hasExports ) { // Node.js Module
    module.exports = theModule;
  } else { // Assign to common namespaces or simply the global object (window)
    ( this.jQuery || this.ender || this.$ || this)[name] = theModule;
  }
})( "core", function () {
    var module = this;
    module.plugins = [];
    module.highlightColor = "yellow";
    module.errorColor = "red";

  // define the core module here and return the public API

  // This is the highlight method used by the core highlightAll()
  // method and all of the plug-ins highlighting elements different
  // colors
  module.highlight = function( el,strColor ){
    if( this.jQuery ){
      jQuery(el).css( "background", strColor );
    }
  }
  return {
      highlightAll:function(){
        module.highlight("div", module.highlightColor);
      }
  };

});
```

*pluginExtension.js:*

```
// Extension to module core

(function ( name, definition ) {
    var theModule = definition(),
        hasDefine = typeof define === "function",
        hasExports = typeof module !== "undefined" && module.exports;

    if ( hasDefine ) { // AMD Module
        define(theModule);
    } else if ( hasExports ) { // Node.js Module
        module.exports = theModule;
    } else {

        // Assign to common namespaces or simply the global object (window)
        // account for flat-file/global module extensions
        var obj = null,
            namespaces,
            scope;

        obj = null;
        namespaces = name.split(".");
        scope = ( this.jQuery || this.ender || this.$ || this );

        for ( var i = 0; i < namespaces.length; i++ ) {
            var packageName = namespaces[i];
            if ( obj && i == namespaces.length - 1 ) {
                obj[packageName] = theModule;
            } else if ( typeof scope[packageName] === "undefined" ) {
                scope[packageName] = {};
            }
            obj = scope[packageName];
        }

    }
})( "core.plugin" , function () {

    // Define our module here and return the public API.
    // This code could be easily adapted with the core to
    // allow for methods that overwrite and extend core functionality
    // in order to expand the highlight method to do more if we wish.
    return {
        setGreen: function ( el ) {
            highlight(el, "green");
        },
        setRed: function ( el ) {
            highlight(el, errorColor);
        }
    };

});
```

UMD 并不旨在取代 AMD 或 CommonJS，而只是为希望在更多环境中使其代码正常工作的开发人员提供一些补充帮助。有关更多信息或对这种实验性格式提出建议，请参阅[此 GitHub 页面](https://oreil.ly/H2pUf)。

### 有关 UMD 和 AMD 的相关阅读

+   [使用 AMD 加载器编写和管理模块化 JavaScript](https://oreil.ly/Zgs_G)

+   [AMD 模块模式：单例](https://oreil.ly/IP22B)

+   [JavaScript 模块和 jQuery 的标准与提案](https://oreil.ly/I-3jy)

# 总结

本节回顾了在 ES2015+ 之前使用不同模块格式编写模块化 JavaScript 的几个选项。

这些格式相对于单独使用模块模式具有几个优点，包括避免管理全局变量的需要，更好地支持静态和动态依赖管理，改进脚本加载器的兼容性，增强服务器上模块的兼容性等。

在结束我们对经典设计和架构模式的讨论之前，我想触及一个领域，即我们可以在下一章关于命名空间模式中应用模式来结构化和组织我们的 JavaScript 代码。
