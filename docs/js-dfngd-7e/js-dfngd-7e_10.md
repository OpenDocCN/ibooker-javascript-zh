# 第十章：模块

模块化编程的目标是允许从不同作者和来源的代码模块组装大型程序，并且所有这些代码在各个模块作者未预料到的代码存在的情况下仍能正确运行。 从实际角度来看，模块化主要是关于封装或隐藏私有实现细节，并保持全局命名空间整洁，以便模块不会意外修改其他模块定义的变量、函数和类。

直到最近，JavaScript 没有内置模块支持，而在大型代码库上工作的程序员尽力利用类、对象和闭包提供的弱模块化。基于闭包的模块化，结合代码捆绑工具的支持，形成了一种基于`require()`函数的实用模块化形式，这被 Node 所采用。 基于`require()`的模块是 Node 编程环境的基本组成部分，但从未被正式纳入 JavaScript 语言的一部分。 相反，ES6 使用`import`和`export`关键字定义模块。 尽管`import`和`export`多年来一直是语言的一部分，但它们最近才被 Web 浏览器和 Node 实现。 作为一个实际问题，JavaScript 模块化仍然依赖于代码捆绑工具。

接下来的章节涵盖：

+   使用类、对象和闭包自行创建模块

+   使用`require()`的 Node 模块

+   使用`export`、`import`和`import()`的 ES6 模块

# 10.1 使用类、对象和闭包的模块

尽管这可能是显而易见的，但值得指出的是，类的一个重要特性是它们作为其方法的模块。 回想一下示例 9-8。 该示例定义了许多不同的类，所有这些类都有一个名为`has()`的方法。 但是，您可以毫无问题地编写一个使用该示例中多个集合类的程序：例如，SingletonSet 的`has()`方法不会覆盖 BitSet 的`has()`方法。

一个类的方法独立于其他不相关类的方法的原因是，每个类的方法被定义为独立原型对象的属性。 类是模块化的原因是对象是模块化的：在 JavaScript 对象中定义属性很像声明变量，但向对象添加属性不会影响程序的全局命名空间，也不会影响其他对象的属性。 JavaScript 定义了相当多的数学函数和常量，但是不是将它们全部定义为全局的，而是将它们作为单个全局 Math 对象的属性分组。 这种技术可以在示例 9-8 中使用。 该示例可以被编写为仅定义一个名为 Sets 的全局对象，其属性引用各种类。 使用此 Sets 库的用户可以使用类似`Sets.Singleton`和`Sets.Bit`的名称引用类。

在 JavaScript 编程中，使用类和对象进行模块化是一种常见且有用的技术，但这还不够。 特别是，它没有提供任何隐藏模块内部实现细节的方法。 再次考虑示例 9-8。 如果我们将该示例编写为一个模块，也许我们希望将各种抽象类保留在模块内部，只将具体子类提供给模块的用户。 同样，在 BitSet 类中，`_valid()`和`_has()`方法是内部实用程序，不应该真正暴露给类的用户。 `BitSet.bits`和`BitSet.masks`是最好隐藏的实现细节。

正如我们在 §8.6 中看到的，函数内声明的局部变量和嵌套函数对该函数是私有的。这意味着我们可以使用立即调用的函数表达式通过将实现细节和实用函数隐藏在封闭函数中，使模块的公共 API 成为函数的返回值来实现一种模块化。对于 BitSet 类，我们可以将模块结构化如下：

```js
const BitSet = (function() { // Set BitSet to the return value of this function
    // Private implementation details here
    function isValid(set, n) { ... }
    function has(set, byte, bit) { ... }
    const BITS = new Uint8Array([1, 2, 4, 8, 16, 32, 64, 128]);
    const MASKS = new Uint8Array([~1, ~2, ~4, ~8, ~16, ~32, ~64, ~128]);

    // The public API of the module is just the BitSet class, which we define
    // and return here. The class can use the private functions and constants
    // defined above, but they will be hidden from users of the class
    return class BitSet extends AbstractWritableSet {
        // ... implementation omitted ...
    };
}());
```

当模块中有多个项时，这种模块化方法变得更加有趣。例如，以下代码定义了一个迷你统计模块，导出 `mean()` 和 `stddev()` 函数，同时隐藏了实现细节：

```js
// This is how we could define a stats module
const stats = (function() {
    // Utility functions private to the module
    const sum = (x, y) => x + y;
    const square = x => x * x;

    // A public function that will be exported
    function mean(data) {
        return data.reduce(sum)/data.length;
    }

    // A public function that we will export
    function stddev(data) {
        let m = mean(data);
        return Math.sqrt(
            data.map(x => x - m).map(square).reduce(sum)/(data.length-1)
        );
    }

    // We export the public function as properties of an object
    return { mean, stddev };
}());

// And here is how we might use the module
stats.mean([1, 3, 5, 7, 9])   // => 5
stats.stddev([1, 3, 5, 7, 9]) // => Math.sqrt(10)
```

## 10.1.1 自动化基于闭包的模块化

请注意，将 JavaScript 代码文件转换为这种模块的过程是一个相当机械化的过程，只需在文件开头和结尾插入一些文本即可。所需的只是一些约定，用于指示哪些值要导出，哪些不要导出。

想象一个工具，它接受一组文件，将每个文件的内容包装在立即调用的函数表达式中，跟踪每个函数的返回值，并将所有内容连接成一个大文件。结果可能看起来像这样：

```js
const modules = {};
function require(moduleName) { return modules[moduleName]; }

modules["sets.js"] = (function() {
    const exports = {};

    // The contents of the sets.js file go here:
    exports.BitSet = class BitSet { ... };

    return exports;
}());

modules["stats.js"] = (function() {
    const exports = {};

    // The contents of the stats.js file go here:
    const sum = (x, y) => x + y;
    const square = x = > x * x;
    exports.mean = function(data) { ... };
    exports.stddev = function(data) { ... };

    return exports;
}());
```

将模块捆绑成一个单一文件，就像前面示例中所示的那样，你可以想象编写以下代码来利用这些模块：

```js
// Get references to the modules (or the module content) that we need
const stats = require("stats.js");
const BitSet = require("sets.js").BitSet;

// Now write code using those modules
let s = new BitSet(100);
s.insert(10);
s.insert(20);
s.insert(30);
let average = stats.mean([...s]); // average is 20
```

这段代码是对代码捆绑工具（如 webpack 和 Parcel）在 web 浏览器中的工作原理的粗略草图，也是对类似于 Node 程序中使用的 `require()` 函数的简单介绍。

# 10.2 Node 模块

在 Node 编程中，将程序分割为尽可能多的文件是很正常的。这些 JavaScript 代码文件都假定存在于一个快速的文件系统上。与 web 浏览器不同，后者必须通过相对较慢的网络连接读取 JavaScript 文件，将 Node 程序捆绑成一个单一的 JavaScript 文件既没有必要也没有好处。

在 Node 中，每个文件都是具有私有命名空间的独立模块。在一个文件中定义的常量、变量、函数和类对该文件是私有的，除非文件导出它们。一个模块导出的值只有在另一个模块明确导入它们时才能看到。

Node 模块通过 `require()` 函数导入其他模块，并通过设置 Exports 对象的属性或完全替换 `module.exports` 对象来导出它们的公共 API。

## 10.2.1 Node 导出

Node 定义了一个全局的 `exports` 对象，它总是被定义。如果你正在编写一个导出多个值的 Node 模块，你可以简单地将它们分配给这个对象的属性：

```js
const sum = (x, y) => x + y;
const square = x => x * x;

exports.mean = data => data.reduce(sum)/data.length;
exports.stddev = function(d) {
    let m = exports.mean(d);
    return Math.sqrt(d.map(x => x - m).map(square).reduce(sum)/(d.length-1));
};
```

然而，通常情况下，你可能只想定义一个仅导出单个函数或类的模块，而不是一个充满函数或类的对象。为此，你只需将要导出的单个值分配给 `module.exports`：

```js
module.exports = class BitSet extends AbstractWritableSet {
    // implementation omitted
};
```

`module.exports` 的默认值是 `exports` 所指向的相同对象。在之前的 stats 模块中，我们可以将 mean 函数分配给 `module.exports.mean` 而不是 `exports.mean`。像 stats 模块这样的模块的另一种方法是在模块末尾导出一个单一对象，而不是在导出函数时逐个导出：

```js
// Define all the functions, public and private
const sum = (x, y) => x + y;
const square = x => x * x;
const mean = data => data.reduce(sum)/data.length;
const stddev = d => {
    let m = mean(d);
    return Math.sqrt(d.map(x => x - m).map(square).reduce(sum)/(d.length-1));
};

// Now export only the public ones
module.exports = { mean, stddev };
```

## 10.2.2 Node 导入

一个 Node 模块通过调用 `require()` 函数来导入另一个模块。这个函数的参数是要导入的模块的名称，返回值是该模块导出的任何值（通常是一个函数、类或对象）。

如果你想导入 Node 内置的系统模块或通过包管理器在系统上安装的模块，那么你只需使用模块的未限定名称，不需要任何将其转换为文件系统路径的“/”字符：

```js
// These modules are built in to Node
const fs = require("fs");           // The built-in filesystem module
const http = require("http");       // The built-in HTTP module

// The Express HTTP server framework is a third-party module.
// It is not part of Node but has been installed locally
const express = require("express");
```

当您想要导入自己代码的模块时，模块名称应该是包含该代码的文件的路径，相对于当前模块文件。使用以*/*字符开头的绝对路径是合法的，但通常，当导入属于您自己程序的模块时，模块名称将以*./ *或有时是*../ *开头，以指示它们相对于当前目录或父目录。例如：

```js
const stats = require('./stats.js');
const BitSet = require('./utils/bitset.js');
```

（您也可以省略导入文件的*.js*后缀，Node 仍然可以找到这些文件，但通常会看到这些文件扩展名明确包含在内。）

当一个模块只导出一个函数或类时，您只需导入它。当一个模块导出一个具有多个属性的对象时，您可以选择：您可以导入整个对象，或者只导入您打算使用的对象的特定属性（使用解构赋值）。比较这两种方法：

```js
// Import the entire stats object, with all of its functions
const stats = require('./stats.js');

// We've got more functions than we need, but they're neatly
// organized into a convenient "stats" namespace.
let average = stats.mean(data);

// Alternatively, we can use idiomatic destructuring assignment to import
// exactly the functions we want directly into the local namespace:
const { stddev } = require('./stats.js');

// This is nice and succinct, though we lose a bit of context
// without the 'stats' prefix as a namspace for the stddev() function.
let sd = stddev(data);
```

## 10.2.3 Web 上的 Node 风格模块

具有 Exports 对象和`require()`函数的模块内置于 Node 中。但是，如果您愿意使用像 webpack 这样的捆绑工具处理您的代码，那么也可以将这种模块样式用于旨在在 Web 浏览器中运行的代码。直到最近，这是一种非常常见的做法，您可能会看到许多仍在这样做的基于 Web 的代码。

现在 JavaScript 有了自己的标准模块语法，然而，使用捆绑工具的开发人员更有可能使用带有`import`和`export`语句的官方 JavaScript 模块。

# 10.3 ES6 中的模块

ES6 为 JavaScript 添加了`import`和`export`关键字，最终将真正的模块化支持作为核心语言特性。ES6 的模块化在概念上与 Node 的模块化相同：每个文件都是自己的模块，文件中定义的常量、变量、函数和类除非明确导出，否则都是私有于该模块。从一个模块导出的值可以在明确导入它们的模块中使用。ES6 模块在导出和导入的语法以及在 Web 浏览器中定义模块的方式上与 Node 模块不同。接下来的部分将详细解释这些内容。

首先，请注意，ES6 模块在某些重要方面也与常规 JavaScript“脚本”不同。最明显的区别是模块化本身：在常规脚本中，变量、函数和类的顶级声明进入由所有脚本共享的单个全局上下文中。使用模块后，每个文件都有自己的私有上下文，并且可以使用`import`和`export`语句，这毕竟是整个重点。但模块和脚本之间还有其他区别。ES6 模块中的代码（就像 ES6 `class`定义内的代码一样）自动处于严格模式（参见§5.6.3）。这意味着，当您开始使用 ES6 模块时，您将永远不必再编写`"use strict"`。这意味着模块中的代码不能使用`with`语句或`arguments`对象或未声明的变量。ES6 模块甚至比严格模式稍微严格：在严格模式中，作为函数调用的函数中，`this`是`undefined`。在模块中，即使在顶级代码中，`this`也是`undefined`。（相比之下，Web 浏览器和 Node 中的脚本将`this`设置为全局对象。）

# Web 上和 Node 中的 ES6 模块

多年来，借助像 webpack 这样的代码捆绑工具，ES6 模块已经在 Web 上得到了应用，这些工具将独立的 JavaScript 代码模块组合成大型、非模块化的捆绑包，适合包含在网页中。然而，在撰写本文时，除了 Internet Explorer 之外，所有 Web 浏览器终于原生支持 ES6 模块。在原生支持时，ES6 模块通过特殊的`<script type="module">`标签添加到 HTML 页面中，本章后面将对此进行描述。

与此同时，作为 JavaScript 模块化的先驱，Node 发现自己处于一个尴尬的位置，必须支持两种不完全兼容的模块系统。Node 13 支持 ES6 模块，但目前，绝大多数 Node 程序仍然使用 Node 模块。

## 10.3.1 ES6 导出

要从 ES6 模块中导出常量、变量、函数或类，只需在声明之前添加关键字`export`：

```js
export const PI = Math.PI;

export function degreesToRadians(d) { return d * PI / 180; }

export class Circle {
    constructor(r) { this.r = r; }
    area() { return PI * this.r * this.r; }
}
```

作为在模块中散布`export`关键字的替代方案，你可以像通常一样定义常量、变量、函数和类，不写任何`export`语句，然后（通常在模块的末尾）写一个单独的`export`语句，声明在一个地方精确地导出了什么。因此，与在前面的代码中写三个单独的导出相反，我们可以在末尾写一行等效的代码：

```js
export { Circle, degreesToRadians, PI };
```

这个语法看起来像是`export`关键字后跟一个对象字面量（使用简写表示法）。但在这种情况下，花括号实际上并没有定义一个对象字面量。这种导出语法只需要在花括号内写一个逗号分隔的标识符列表。

编写只导出一个值（通常是函数或类）的模块很常见，在这种情况下，我们通常使用`export default`而不是`export`：

```js
export default class BitSet {
    // implementation omitted
}
```

默认导出比非默认导出稍微容易导入，因此当只有一个导出值时，使用`export default`会使使用你导出值的模块更容易。

使用`export`进行常规导出只能用于具有名称的声明。使用`export default`进行默认导出可以导出任何表达式，包括匿名函数表达式和匿名类表达式。这意味着如果你使用`export default`，你可以导出对象字面量。因此，与`export`语法不同，如果你在`export default`后看到花括号，那么实际上导出的是一个对象字面量。

模块既有一组常规导出又有默认导出是合法的，但有些不太常见。如果一个模块有默认导出，它只能有一个。

最后，请注意`export`关键字只能出现在你的 JavaScript 代码的顶层。你不能从类、函数、循环或条件语句中导出值。（这是 ES6 模块系统的一个重要特性，它实现了静态分析：模块的导出在每次运行时都是相同的，并且可以在模块实际运行之前确定导出的符号。）

## 10.3.2 ES6 导入

你可以使用`import`关键字导入其他模块导出的值。最简单的导入形式用于定义默认导出的模块：

```js
import BitSet from './bitset.js';
```

这是`import`关键字，后面跟着一个标识符，然后是`from`关键字，后面是一个字符串字面量，命名了我们要导入默认导出的模块。指定模块的默认导出值成为当前模块中指定标识符的值。

导入值分配给的标识符是一个常量，就好像它已经用`const`关键字声明过一样。与导出一样，导入只能出现在模块的顶层，不允许在类、函数、循环或条件语句中。根据普遍惯例，模块所需的导入应放在模块的开头。然而有趣的是，这并非必须：像函数声明一样，导入被“提升”到顶部，所有导入的值在模块的任何代码运行时都可用。

导入值的模块被指定为一个常量字符串文字，用单引号或双引号括起来。（您不能使用值为字符串的变量或其他表达式，也不能在反引号内使用字符串，因为模板文字可以插入变量并且不总是具有常量值。）在 Web 浏览器中，此字符串被解释为相对于执行导入操作的模块的位置的 URL。（在 Node 中，或者使用捆绑工具时，该字符串被解释为相对于当前模块的文件名，但在实践中这几乎没有区别。）*模块规范符*字符串必须是以“/”开头的绝对路径，或以“./”或“../”开头的相对路径，或具有协议和主机名的完整 URL。ES6 规范不允许未经限定的模块规范符字符串，如“util.js”，因为不清楚这是否意味着要命名与当前目录中的模块或某种安装在某个特殊位置的系统模块。 （这个对“裸模块规范符”的限制不被像 webpack 这样的代码捆绑工具所遵守，它可以很容易地配置为在您指定的库目录中找到裸模块。）语言的未来版本可能允许“裸模块规范符”，但目前不允许。如果要从与当前目录相同的目录导入模块，只需在模块名称前加上“./”，并从“./util.js”而不是“util.js”导入。

到目前为止，我们只考虑了从使用`export default`的模块导入单个值的情况。要从导出多个值的模块导入值，我们使用稍微不同的语法：

```js
import { mean, stddev } from "./stats.js";
```

请记住，默认导出在定义它们的模块中不需要名称。相反，当我们导入这些值时，我们提供一个本地名称。但是，模块的非默认导出在导出模块中有名称，当我们导入这些值时，我们通过这些名称引用它们。导出模块可以导出任意数量的命名值。引用该模块的`import`语句可以通过在花括号内列出它们的名称来导入这些值的任意子集。花括号使这种`import`语句看起来有点像解构赋值，实际上，解构赋值是这种导入样式在做的事情的一个很好的类比。花括号内的标识符都被提升到导入模块的顶部，并且行为像常量。

样式指南有时建议您明确导入模块将使用的每个符号。但是，当从定义许多导出的模块导入时，您可以轻松地使用像这样的`import`语句导入所有内容：

```js
import * as stats from "./stats.js";
```

像这样的`import`语句会创建一个对象，并将其赋值给名为`stats`的常量。被导入模块的每个非默认导出都成为这个`stats`对象的属性。非默认导出始终有名称，并且这些名称在对象内部用作属性名称。这些属性实际上是常量：它们不能被覆盖或删除。在前面示例中显示的通配符导入中，导入模块将通过`stats`对象使用导入的`mean()`和`stddev()`函数，调用它们为`stats.mean()`和`stats.stddev()`。

模块通常定义一个默认导出或多个命名导出。一个模块同时使用`export`和`export default`是合法的，但有点不常见。但是当一个模块这样做时，您可以使用像这样的`import`语句同时导入默认值和命名值：

```js
import Histogram, { mean, stddev } from "./histogram-stats.js";
```

到目前为止，我们已经看到了如何从具有默认导出的模块和具有非默认或命名导出的模块导入。但是还有一种`import`语句的形式，用于没有任何导出的模块。要将没有任何导出的模块包含到您的程序中，只需使用`import`关键字与模块规范符：

```js
import "./analytics.js";
```

这样的模块在第一次导入时运行。（随后的导入不会执行任何操作。）一个仅定义函数的模块只有在导出其中至少一个函数时才有用。但是，如果一个模块运行一些代码，那么即使没有符号，导入它也是有用的。一个用于 Web 应用程序的分析模块可能会运行代码来注册各种事件处理程序，然后在适当的时候使用这些事件处理程序将遥测数据发送回服务器。该模块是自包含的，不需要导出任何内容，但我们仍然需要`import`它，以便它实际上作为我们程序的一部分运行。

请注意，即使有导出的模块，您也可以使用这种导入空内容的`import`语法。如果一个模块定义了独立于其导出值的有用行为，并且您的程序不需要任何这些导出值，您仍然可以导入该模块。只是为了那个默认行为。

## 10.3.3 导入和重命名导出

如果两个模块使用相同名称导出两个不同的值，并且您想要导入这两个值，那么在导入时您将需要重命名其中一个或两个值。同样，如果您想要导入一个名称已在您的模块中使用的值，那么您将需要重命名导入的值。您可以使用带有命名导入的`as`关键字来重命名它们：

```js
import { render as renderImage } from "./imageutils.js";
import { render as renderUI } from "./ui.js";
```

这些行将两个函数导入当前模块。这两个函数在定义它们的模块中都被命名为`render()`，但在导入时使用更具描述性和消除歧义的名称`renderImage()`和`renderUI()`。

请记住，默认导出没有名称。导入模块在导入默认导出时总是选择名称。因此，在这种情况下不需要特殊的重命名语法。

话虽如此，但在导入时重命名的可能性提供了另一种从定义默认导出和命名导出的模块中导入的方式。回想一下前一节中的“./histogram-stats.js”模块。以下是导入该模块的默认导出和命名导出的另一种方式：

```js
import { default as Histogram, mean, stddev } from "./histogram-stats.js";
```

在这种情况下，JavaScript 关键字`default`充当占位符，并允许我们指示我们要导入并为模块的默认导出提供名称。

也可以在导出时重命名值，但只能在使用花括号变体的`export`语句时。通常不需要这样做，但如果您在模块内选择了简短、简洁的名称，您可能更喜欢使用更具描述性的名称导出值，这样就不太可能与其他模块发生冲突。与导入一样，您可以使用`as`关键字来执行此操作：

```js
export {
    layout as calculateLayout,
    render as renderLayout
};
```

请记住，尽管花括号看起来有点像对象文字，但它们并不是，`export`关键字在`as`之前期望一个标识符，而不是一个表达式。这意味着不幸的是，您不能像这样使用导出重命名：

```js
export { Math.sin as sin, Math.cos as cos }; // SyntaxError
```

## 10.3.4 重新导出

在本章中，我们讨论了一个假设的“./stats.js”模块，该模块导出`mean()`和`stddev()`函数。如果我们正在编写这样一个模块，并且我们认为该模块的许多用户只想要其中一个函数，那么我们可能希望在“./stats/mean.js”模块中定义`mean()`，在“./stats/stddev.js”中定义`stddev()`。这样，程序只需要导入它们需要的函数，而不会因导入不需要的代码而臃肿。

即使我们将这些统计函数定义在单独的模块中，我们可能仍然希望有很多程序需要这两个函数，并且希望有一个方便的“./stats.js”模块，可以在一行中导入这两个函数。

鉴于现在实现在单独的文件中，定义这个“./stat.js”模块很简单：

```js
import { mean } from "./stats/mean.js";
import { stddev } from "./stats/stddev.js";
export { mean, stdev };
```

ES6 模块预期这种用法并为其提供了特殊的语法。不再简单地导入一个符号再导出它，你可以将导入和导出步骤合并为一个单独的“重新导出”语句，使用`export`关键字和`from`关键字：

```js
export { mean } from "./stats/mean.js";
export { stddev } from "./stats/stddev.js";
```

请注意，此代码中实际上没有使用`mean`和`stddev`这两个名称。如果我们不选择性地重新导出并且只想从另一个模块导出所有命名值，我们可以使用通配符：

```js
export * from "./stats/mean.js";
export * from "./stats/stddev.js";
```

重新导出语法允许使用`as`进行重命名，就像常规的`import`和`export`语句一样。假设我们想要重新导出`mean()`函数，但同时为该函数定义`average()`作为另一个名称。我们可以这样做：

```js
export { mean, mean as average } from "./stats/mean.js";
export { stddev } from "./stats/stddev.js";
```

该示例中的所有重新导出都假定“./stats/mean.js”和“./stats/stddev.js”模块使用`export`而不是`export default`导出它们的函数。实际上，由于这些是只有一个导出的模块，定义为`export default`是有意义的。如果我们这样做了，那么重新导出语法会稍微复杂一些，因为它需要为未命名的默认导出定义一个名称。我们可以这样做：

```js
export { default as mean } from "./stats/mean.js";
export { default as stddev } from "./stats/stddev.js";
```

如果你想要将另一个模块的命名符号重新导出为你的模块的默认导出，你可以进行`import`，然后进行`export default`，或者你可以将这两个语句结合起来，像这样：

```js
// Import the mean() function from ./stats.js and make it the
// default export of this module
export { mean as default } from "./stats.js"
```

最后，要将另一个模块的默认导出重新导出为你的模块的默认导出（尽管不清楚为什么要这样做，因为用户可以直接导入另一个模块），你可以这样写：

```js
// The average.js module simply re-exports the stats/mean.js default export
export { default } from "./stats/mean.js"
```

## 10.3.5 Web 上的 JavaScript 模块

前面的章节以一种相对抽象的方式描述了 ES6 模块及其`import`和`export`声明。在本节和下一节中，我们将讨论它们在 Web 浏览器中的实际工作方式，如果你还不是一名经验丰富的 Web 开发人员，你可能会发现在阅读第十五章之后更容易理解本章的其余内容。

截至 2020 年初，使用 ES6 模块的生产代码仍然通常与类似 webpack 的工具捆绑在一起。这样做存在一些权衡之处，¹但总体上，代码捆绑往往能提供更好的性能。随着网络速度的增长和浏览器厂商继续优化他们的 ES6 模块实现，这种情况可能会发生变化。

尽管捆绑工具在生产中仍然可取，但在开发中不再需要，因为所有当前的浏览器都提供了对 JavaScript 模块的原生支持。请记住，模块默认使用严格模式，`this`不指向全局对象，并且顶级声明默认情况下不会在全局范围内共享。由于模块必须以与传统非模块代码不同的方式执行，它们的引入需要对 HTML 和 JavaScript 进行更改。如果你想在 Web 浏览器中原生使用`import`指令，你必须通过使用`<script type="module">`标签告诉 Web 浏览器你的代码是一个模块。

ES6 模块的一个很好的特性是每个模块都有一个静态的导入集合。因此，给定一个起始模块，Web 浏览器可以加载所有导入的模块，然后加载第一批模块导入的所有模块，依此类推，直到完整的程序被加载。我们已经看到`import`语句中的模块指示符可以被视为相对 URL。`<script type="module">`标签标记了模块化程序的起点。然而，它导入的模块都不应该在`<script>`标签中，而是按需作为常规 JavaScript 文件加载，并像常规 ES6 模块一样以严格模式执行。使用`<script type="module">`标签来定义模块化 JavaScript 程序的主入口点可以像这样简单：

```js
<script type="module">import "./main.js";</script>
```

内联`<script type="module">`标签中的代码是 ES6 模块，因此可以使用`export`语句。然而，这样做没有任何意义，因为 HTML `<script>`标签语法没有提供任何定义内联模块名称的方式，因此，即使这样的模块导出一个值，也没有办法让另一个模块导入它。

带有`type="module"`属性的脚本会像带有`defer`属性的脚本一样加载和执行。代码加载会在 HTML 解析器遇到`<script>`标签时开始（在模块的情况下，这个代码加载步骤可能是一个递归过程，加载多个 JavaScript 文件）。但是代码执行直到 HTML 解析完成才开始。一旦 HTML 解析完成，脚本（模块和非模块）将按照它们在 HTML 文档中出现的顺序执行。

您可以使用`async`属性修改模块的执行时间，这对于模块和常规脚本的工作方式是相同的。`async`模块将在代码加载后立即执行，即使 HTML 解析尚未完成，即使这会改变脚本的相对顺序。

支持`<script type="module">`的 Web 浏览器也必须支持`<script nomodule>`。了解模块的浏览器会忽略带有`nomodule`属性的任何脚本并且不会执行它。不支持模块的浏览器将不识别`nomodule`属性，因此它们会忽略它并运行脚本。这为处理浏览器兼容性问题提供了一个强大的技术。支持 ES6 模块的浏览器还支持其他现代 JavaScript 特性，如类、箭头函数和`for/of`循环。如果您编写现代 JavaScript 并使用`<script type="module">`加载它，您知道它只会被支持的浏览器加载。作为 IE11 的备用方案（在 2020 年，实际上是唯一一个不支持 ES6 的浏览器），您可以使用类似 Babel 和 webpack 的工具将您的代码转换为非模块化的 ES5 代码，然后通过`<script nomodule>`加载这些效率较低的转换代码。

常规脚本和模块脚本之间的另一个重要区别与跨域加载有关。常规的`<script>`标签将从互联网上的任何服务器加载 JavaScript 代码文件，互联网的广告、分析和跟踪代码基础设施依赖于这一事实。但是`<script type="module">`提供了一种加强这一点的机会，模块只能从包含 HTML 文档的同一源加载，或者在适当的 CORS 标头放置以安全地允许跨源加载时才能加载。这种新的安全限制的一个不幸副作用是，它使得使用`file:` URL 在开发模式下测试 ES6 模块变得困难。使用 ES6 模块时，您可能需要设置一个静态 Web 服务器进行测试。

一些程序员喜欢使用文件扩展名`.mjs`来区分他们的模块化 JavaScript 文件和传统`.js`扩展名的常规非模块化 JavaScript 文件。对于 Web 浏览器和`<script>`标签来说，文件扩展名实际上是无关紧要的。（但 MIME 类型是相关的，因此如果您使用`.mjs`文件，您可能需要配置您的 Web 服务器以相同的 MIME 类型提供它们，如`.js`文件。）Node 对 ES6 的支持确实使用文件扩展名作为提示来区分它加载的每个文件使用的模块系统。因此，如果您编写 ES6 模块并希望它们能够在 Node 中使用，采用`.mjs`命名约定可能会有所帮助。

## 10.3.6 使用 import()进行动态导入

我们已经看到 ES6 的 `import` 和 `export` 指令是完全静态的，并且使 JavaScript 解释器和其他 JavaScript 工具能够在加载模块时通过简单的文本分析确定模块之间的关系，而无需实际执行模块中的任何代码。使用静态导入的模块，你可以确保导入到模块中的值在你的模块中的任何代码开始运行之前就已经准备好供使用。

在 Web 上，代码必须通过网络传输，而不是从文件系统中读取。一旦传输，该代码通常在相对较慢的移动设备上执行。这不是静态模块导入（需要在任何代码运行之前加载整个程序）有很多意义的环境。

Web 应用程序通常只加载足够的代码来渲染用户看到的第一页。然后，一旦用户有一些初步内容可以交互，它们就可以开始加载通常需要更多的代码来完成网页应用程序的其余部分。Web 浏览器通过使用 DOM API 将新的 `<script>` 标签注入到当前 HTML 文档中，使动态加载代码变得容易，而 Web 应用程序多年来一直在这样做。

尽管动态加载已经很久了，但它并不是语言本身的一部分。这在 ES2020 中发生了变化（截至 2020 年初，支持 ES6 模块的所有浏览器都支持动态导入）。你将一个模块规范传递给 `import()`，它将返回一个代表加载和运行指定模块的异步过程的 Promise 对象。当动态导入完成时，Promise 将“完成”（请参阅 第十三章 了解有关异步编程和 Promise 的完整细节），并产生一个对象，就像你使用静态导入语句的 `import * as` 形式一样。

因此，我们可以像这样静态导入“./stats.js”模块：

```js
import * as stats from "./stats.js";
```

我们可以像这样动态导入并使用它：

```js
import("./stats.js").then(stats => {
    let average = stats.mean(data);
})
```

或者，在一个 `async` 函数中（再次，你可能需要在理解这段代码之前阅读 第十三章），我们可以用 `await` 简化代码：

```js
async analyzeData(data) {
    let stats = await import("./stats.js");
    return {
        average: stats.mean(data),
        stddev: stats.stddev(data)
    };
}
```

`import()` 的参数应该是一个模块规范，就像你会在静态 `import` 指令中使用的那样。但是使用 `import()`，你不受限于使用常量字符串文字：任何表达式只要以正确形式评估为字符串即可。

动态 `import()` 看起来像一个函数调用，但实际上不是。相反，`import()` 是一个操作符，括号是操作符语法的必需部分。这种不寻常的语法之所以存在是因为 `import()` 需要能够将模块规范解析为相对于当前运行模块的 URL，这需要一些实现魔法，这是不合法的放在 JavaScript 函数中的。在实践中，函数与操作符的区别很少有影响，但如果尝试编写像 `console.log(import);` 或 `let require = import;` 这样的代码，你会注意到这一点。

最后，请注意动态 `import()` 不仅适用于 Web 浏览器。代码打包工具如 webpack 也可以很好地利用它。使用代码捆绑器的最简单方法是告诉它程序的主入口点，让它找到所有静态 `import` 指令并将所有内容组装成一个大文件。然而，通过策略性地使用动态 `import()` 调用，你可以将这个单一的庞大捆绑拆分成一组可以按需加载的较小捆绑。

## 10.3.7 import.meta.url

ES6 模块系统的最后一个特性需要讨论。在 ES6 模块中（但不在常规的 `<script>` 或使用 `require()` 加载的 Node 模块中），特殊语法 `import.meta` 指的是一个包含有关当前执行模块的元数据的对象。该对象的 `url` 属性是加载模块的 URL。（在 Node 中，这将是一个 `file://` URL。）

`import.meta.url` 的主要用例是能够引用存储在与模块相同目录中（或相对于模块）的图像、数据文件或其他资源。`URL()` 构造函数使得相对 URL 相对于绝对 URL（如 `import.meta.url`）容易解析。例如，假设你编写了一个模块，其中包含需要本地化的字符串，并且本地化文件存储在与模块本身相同目录中的 `l10n/` 目录中。你的模块可以使用类似这样的函数创建的 URL 加载其字符串：

```js
function localStringsURL(locale) {
    return new URL(`l10n/${locale}.json`, import.meta.url);
}
```

# 10.4 总结

模块化的目标是允许程序员隐藏其代码的实现细节，以便来自各种来源的代码块可以组装成大型程序，而不必担心一个代码块会覆盖另一个的函数或变量。本章已经解释了三种不同的 JavaScript 模块系统：

+   在 JavaScript 的早期，模块化只能通过巧妙地使用立即调用的函数表达式来实现。

+   Node 在 JavaScript 语言之上添加了自己的模块系统。Node 模块通过 `require()` 导入，并通过设置 Exports 对象的属性或设置 `module.exports` 属性来定义它们的导出。

+   在 ES6 中，JavaScript 终于拥有了自己的模块系统，使用 `import` 和 `export` 关键字，而 ES2020 正在添加对使用 `import()` 进行动态导入的支持。

¹ 例如：经常进行增量更新并且用户频繁返回访问的 Web 应用程序可能会发现，使用小模块而不是大捆绑包可以更好地利用用户浏览器缓存，从而导致更好的平均加载时间。
