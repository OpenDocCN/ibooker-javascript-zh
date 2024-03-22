# 第十六章：用 Node 进行服务器端 JavaScript

Node 是 JavaScript 与底层操作系统的绑定，使得编写 JavaScript 程序读写文件、执行子进程和在网络上通信成为可能。这使得 Node 作为以下用途变得有用：

+   现代替代 shell 脚本的方式，不受 bash 和其他 Unix shell 繁琐语法的困扰。

+   用于运行受信任程序的通用编程语言，不受 Web 浏览器对不受信任代码施加的安全约束。

+   编写高效且高度并发的 Web 服务器的流行环境。

Node 的定义特点是其单线程事件驱动并通过默认异步 API 实现的并发性。如果你已经在其他语言中编程过但并没有做过太多 JavaScript 编码，或者如果你是一位经验丰富的客户端 JavaScript 程序员，习惯为 Web 浏览器编写代码，那么使用 Node 将需要一些调整，就像任何新的编程语言或环境一样。本章首先解释了 Node 的编程模型，重点是并发性，Node 用于处理流数据的 API，以及 Node 用于处理二进制数据的缓冲区类型。这些初始部分之后是突出和演示一些最重要的 Node API 的部分，包括用于处理文件、网络、进程和线程的 API。

一章不足以记录所有 Node 的 API，但我希望这一章能够解释足够的基础知识，让你能够在 Node 上提高效率，并确信你可以掌握任何你需要的新 API。

# 16.1 Node 编程基础

我们将从快速了解 Node 程序的结构以及它们如何与操作系统交互开始这一章节。

## 16.1.1 控制台输出

如果你习惯于为 Web 浏览器编程的 JavaScript，那么关于 Node 的一个小惊喜是 `console.log()` 不仅用于调试，而且是 Node 显示消息给用户或者更一般地向 stdout 流发送输出的最简单方式。以下是 Node 中经典的“Hello World”程序：

```js
console.log("Hello World!");
```

有更低级别的方法可以写入 stdout，但没有比简单调用 `console.log()` 更花哨或更正式的方式。

在 Web 浏览器中，`console.log()`、`console.warn()` 和 `console.error()` 通常在开发者控制台中的输出旁边显示小图标，以指示日志消息的种类。Node 不会这样做，但使用 `console.error()` 显示的输出与使用 `console.log()` 显示的输出有所区别，因为 `console.error()` 写入 stderr 流。如果你正在使用 Node 编写一个程序，该程序旨在将 stdout 重定向到文件或管道，你可以使用 `console.error()` 将文本显示到用户将看到的控制台，即使使用 `console.log()` 打印的文本是隐藏的。

## 16.1.2 命令行参数和环境变量

如果你之前编写过设计为从终端或其他命令行界面调用的类 Unix 风格程序，你会知道这些程序通常主要从命令行参数获取输入，其次从环境变量获取输入。

Node 遵循这些 Unix 约定。一个 Node 程序可以从字符串数组 `process.argv` 中读取其命令行参数。这个数组的第一个元素始终是 Node 可执行文件的路径。第二个参数是 Node 正在执行的 JavaScript 代码文件的路径。在这个数组中的任何剩余元素都是你在调用 Node 时通过命令行传递的以空格分隔的参数。

例如，假设你将这个非常简短的 Node 程序保存到名为 *argv.js* 的文件中：

```js
console.log(process.argv);
```

然后你可以执行该程序并看到如下输出：

```js
$ node --trace-uncaught argv.js --arg1 --arg2 filename
[
  '/usr/local/bin/node',
  '/private/tmp/argv.js',
  '--arg1',
  '--arg2',
  'filename'
]
```

这里有几点需要注意：

+   `process.argv` 的第一个和第二个元素将是完全限定的文件系统路径，指向 Node 可执行文件和正在执行的 JavaScript 文件，即使你没有以这种方式输入它们。

+   用于 Node 可执行文件本身的命令行参数由 Node 可执行文件消耗，不会出现在`process.argv`中。（在上面的示例中，`--trace-uncaught`命令行参数实际上并没有做任何有用的事情；它只是用来演示它不会出现在输出中。）任何出现在 JavaScript 文件名之后的参数（如`--arg1`和`filename`）将出现在`process.argv`中。

Node 程序也可以从类 Unix 环境变量中获取输入。Node 通过`process.env`对象使这些变量可用。该对象的属性名称是环境变量名称，属性值（始终为字符串）是这些变量的值。

这是我系统上环境变量的部分列表：

```js
$ node -p -e 'process.env'
{
  SHELL: '/bin/bash',
  USER: 'david',
  PATH: '/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin',
  PWD: '/tmp',
  LANG: 'en_US.UTF-8',
  HOME: '/Users/david',
}
```

你可以使用`node -h`或`node --help`来查找`-p`和`-e`命令行参数的作用。然而，作为提示，注意你可以将上面的行重写为`node --eval 'process.env' --print`。

## 16.1.3 程序生命周期

`node`命令需要一个命令行参数来指定要运行的 JavaScript 代码文件。这个初始文件通常导入其他 JavaScript 代码模块，并可能定义自己的类和函数。然而，从根本上说，Node 会按顺序执行指定文件中的 JavaScript 代码。一些 Node 程序在执行文件中的最后一行代码后完成执行时退出。然而，通常情况下，一个 Node 程序将在执行初始文件后继续运行。正如我们将在接下来的章节中讨论的那样，Node 程序通常是异步的，基于回调和事件处理程序。Node 程序直到运行完初始文件并且所有回调都被调用且没有更多待处理事件时才会退出。一个基于 Node 的服务器程序监听传入的网络连接，理论上会永远运行，因为它总是会等待更多事件。

一个程序可以通过调用`process.exit()`来强制退出。用户通常可以通过在运行程序的终端窗口中键入 Ctrl-C 来终止 Node 程序。程序可以通过注册一个信号处理程序函数`process.on("SIGINT", ()=>{})`来忽略 Ctrl-C。

如果程序中的代码抛出异常而没有`catch`子句捕获它，程序将打印堆栈跟踪并退出。由于 Node 的异步特性，发生在回调或事件处理程序中的异常必须在本地处理或根本不处理，这意味着处理程序中异步部分发生的异常可能是一个困难的问题。如果你不希望这些异常导致程序完全崩溃，注册一个全局处理程序函数将被调用而不是崩溃：

```js
process.setUncaughtExceptionCaptureCallback(e => {
    console.error("Uncaught exception:", e);
});
```

如果你的程序创建的 Promise 被拒绝并且没有`.catch()`调用来处理它，会出现类似的情况。截至 Node 13，这不是导致程序退出的致命错误，但会在控制台打印详细的错误消息。在未来的某个 Node 版本中，未处理的 Promise 拒绝预计将成为致命错误。如果你不希望未处理的拒绝打印错误消息或终止程序，注册一个全局处理程序函数：

```js
process.on("unhandledRejection", (reason, promise) => {
    // reason is whatever value would have been passed to a .catch() function
    // promise is the Promise object that rejected
});
```

## 16.1.4 Node 模块

第十章记录了 JavaScript 模块系统，涵盖了 Node 模块和 ES6 模块。因为 Node 是在 JavaScript 拥有模块系统之前创建的，所以 Node 不得不创建自己的模块系统。Node 的模块系统使用`require()`函数将值导入模块，使用`exports`对象或`module.exports`属性从模块导出值。这些是 Node 编程模型的基本部分，并在§10.2 中详细介绍。

Node 13 添加了对标准 ES6 模块和基于 require 的模块（Node 称之为“CommonJS 模块”）的支持。这两种模块系统并不完全兼容，因此这有点棘手。Node 需要在加载模块之前知道该模块是否将使用`require()`和`module.exports`，还是将使用`import`和`export`。当 Node 将 JavaScript 代码文件加载为 CommonJS 模块时，它会自动定义`require()`函数以及标识符`exports`和`module`，并且不会启用`import`和`export`关键字。另一方面，当 Node 将代码文件加载为 ES6 模块时，它必须启用`import`和`export`声明，并且不能定义额外的标识符如`require`、`module`和`exports`。

告诉 Node 正在加载的模块的类型最简单的方法是将这些信息编码在文件扩展名中。如果您将 JavaScript 代码保存在以*.mjs*结尾的文件中，那么 Node 将始终将其作为 ES6 模块加载，期望它使用`import`和`export`，并且不会提供`require()`函数。如果您将代码保存在以*.cjs*结尾的文件中，那么 Node 将始终将其视为 CommonJS 模块，提供`require()`函数，并且如果您使用`import`或`export`声明，则会抛出 SyntaxError。

对于没有明确*.mjs*或*.cjs*扩展名的文件，Node 会在与文件相同的目录中查找名为*package.json*的文件，然后在每个包含目录中查找。一旦找到最近的*package.json*文件，Node 会检查 JSON 对象中的顶级`type`属性。如果`type`属性的值是“module”，那么 Node 会将文件加载为 ES6 模块。如果该属性的值是“commonjs”，那么 Node 会将文件加载为 CommonJS 模块。请注意，您不需要有*package.json*文件来运行 Node 程序：当找不到这样的文件时（或者找到文件但它没有`type`属性时），Node 会默认使用 CommonJS 模块。只有当您想要在 Node 中使用 ES6 模块而不想使用*.mjs*文件扩展名时，才需要使用这个*package.json*技巧。

由于有大量使用 CommonJS 模块格式编写的现有 Node 代码，Node 允许 ES6 模块使用`import`关键字加载 CommonJS 模块。然而，反之则不成立：CommonJS 模块无法使用`require()`加载 ES6 模块。

## 16.1.5 Node 包管理器

安装 Node 时，通常也会得到一个名为 npm 的程序。这是 Node 包管理器，它帮助您下载和管理程序所依赖的库。npm 会在项目的根目录中的名为*package.json*的文件中跟踪这些依赖项（以及关于您的程序的其他信息）。由 npm 创建的这个*package.json*文件是您想要为项目使用 ES6 模块时会添加`"type":"module"`的地方。

本章不会详细介绍 npm（但请参见§17.4 以获取更多深入信息）。我在这里提到它是因为除非您编写的程序不使用任何外部库，否则您几乎肯定会使用 npm 或类似工具。例如，假设您将要开发一个 Web 服务器，并计划使用 Express 框架（[*https://expressjs.com*](https://expressjs.com)）来简化任务。要开始，您可以为项目创建一个目录，然后在该目录中输入`npm init`。npm 会询问您项目名称、版本号等信息，然后根据您的回答创建一个初始的*package.json*文件。

现在要开始使用 Express，您可以输入`npm install express`。这告诉 npm 下载 Express 库以及其所有依赖项，并将所有包安装在本地*node_modules/*目录中：

```js
$ npm install express
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN my-server@1.0.0 No description
npm WARN my-server@1.0.0 No repository field.

+ express@4.17.1
added 50 packages from 37 contributors and audited 126 packages in 3.058s
found 0 vulnerabilities
```

当你使用 npm 安装一个包时，npm 会记录这种依赖关系——即你的项目依赖于 Express——在 *package.json* 文件中。有了在 *package.json* 中记录的这种依赖关系，你可以将你的代码和 *package.json* 的副本交给另一个程序员，他们只需输入 `npm install` 就可以自动下载和安装程序运行所需的所有库。

# 16.2 节点默认是异步的

JavaScript 是一种通用编程语言，因此完全可以编写乘法大矩阵或执行复杂统计分析等 CPU 密集型程序。但 Node 是为 I/O 密集型的程序（如网络服务器）而设计和优化的。特别是，Node 的设计使得轻松实现高度并发的服务器成为可能，可以同时处理许多请求。

然而，与许多编程语言不同，Node 不使用线程来实现并发。多线程编程通常很难正确完成，也很难调试。此外，线程是一个相对较重的抽象，如果你想编写一个能够处理数百个并发请求的服务器，使用数百个线程可能需要大量的内存。因此，Node 采用了 Web 使用的单线程 JavaScript 编程模型，这实际上是一种巨大的简化，使得创建网络服务器成为一种常规技能而不是一种神秘技能。

Node 通过将其 API 默认设置为异步和非阻塞来保持高并发水平，同时保持单线程编程模型。Node 非常认真地采取了非阻塞的方法，甚至可能会让你感到惊讶。你可能期望从网络读取和写入的函数是异步的，但 Node 更进一步，为从本地文件系统读取和写入文件定义了非阻塞异步函数。这是有道理的，当你考虑到：Node API 是在旋转硬盘仍然是标准的时代设计的，而在进行文件操作之前确实有毫秒级的阻塞“寻道时间”，等待磁盘旋转以开始文件操作。在现代数据中心，所谓的“本地”文件系统实际上可能在网络的某个地方，上面还有网络延迟。但即使异步读取文件对你来说是正常的，Node 仍然更进一步：例如，用于启动网络连接或查找文件修改时间的默认函数也是非阻塞的。

Node 的 API 中有一些同步但非阻塞的函数：它们运行完成并返回而无需阻塞。但大多数有趣的函数执行某种输入或输出，这些是异步函数，因此它们可以避免甚至最微小的阻塞。Node 是在 JavaScript 拥有 Promise 类之前创建的，因此异步 Node API 是基于回调的。（如果你还没有阅读或已经忘记第十三章，现在是回到那一章的好时机。）通常，你传递给异步 Node 函数的最后一个参数是一个回调函数。Node 使用 *错误优先回调*，通常用两个参数调用。错误优先回调的第一个参数通常在没有错误发生的情况下为 `null`，第二个参数是由你调用的原始异步函数产生的数据或响应。将错误参数放在第一位的原因是为了让你无法忽略它，你应该始终检查这个参数中是否有非空值。如果它是一个错误对象，甚至是一个整数错误代码或字符串错误消息，那么出现了问题。在这种情况下，你回调函数的第二个参数可能是 `null`。

以下代码演示了如何使用非阻塞的`readFile()`函数读取配置文件，将其解析为 JSON，然后将解析后的配置对象传递给另一个回调函数：

```js
const fs = require("fs");  // Require the filesystem module

// Read a config file, parse its contents as JSON, and pass the
// resulting value to the callback. If anything goes wrong,
// print an error message to stderr and invoke the callback with null
function readConfigFile(path, callback) {
    fs.readFile(path, "utf8", (err, text) => {
        if (err) {    // Something went wrong reading the file
            console.error(err);
            callback(null);
            return;
        }
        let data = null;
        try {
            data = JSON.parse(text);
        } catch(e) {  // Something went wrong parsing the file contents
            console.error(e);
        }
        callback(data);
    });
}
```

Node 早于标准化的 promises，但由于它在错误优先回调方面相当一致，使用`util.promisify()`包装器可以轻松创建基于 Promise 的变体。这是我们如何重写`readConfigFile()`函数以返回一个 Promise：

```js
const util = require("util");
const fs = require("fs");  // Require the filesystem module
const pfs = {              // Promise-based variants of some fs functions
    readFile: util.promisify(fs.readFile)
};

function readConfigFile(path) {
    return pfs.readFile(path, "utf-8").then(text => {
        return JSON.parse(text);
    });
}
```

我们还可以使用`async`和`await`简化前面基于 Promise 的函数（再次，如果您尚未阅读第十三章，现在是一个好时机）：

```js
async function readConfigFile(path) {
    let text = await pfs.readFile(path, "utf-8");
    return JSON.parse(text);
}
```

`util.promisify()`包装器可以生成许多 Node 函数的基于 Promise 的版本。在 Node 10 及更高版本中，`fs.promises`对象有许多预定义的基于 Promise 的函数，用于处理文件系统。我们将在本章后面讨论它们，但请注意，在前面的代码中，我们可以用`fs.promises.readFile()`替换`pfs.readFile()`。

我们曾经说过，Node 的编程模型是默认异步的。但为了程序员的方便，Node 确实定义了许多阻塞的同步变体函数，特别是在文件系统模块中。这些函数通常以`Sync`结尾的名称清晰标记。

当服务器首次启动并读取其配置文件时，它尚未处理网络请求，实际上几乎不可能发生并发。因此，在这种情况下，没有必要避免阻塞，我们可以安全地使用像`fs.readFileSync()`这样的阻塞函数。我们可以从这段代码中删除`async`和`await`，并编写我们的`readConfigFile()`函数的纯同步版本。这个函数不是调用回调或返回 Promise，而是简单地返回解析后的 JSON 值或抛出异常：

```js
const fs = require("fs");
function readConfigFileSync(path) {
    let text = fs.readFileSync(path, "utf-8");
    return JSON.parse(text);
}
```

除了其错误优先的两参数回调之外，Node 还有许多使用基于事件的异步性的 API，通常用于处理流数据。我们稍后会更详细地介绍 Node 事件。

现在我们已经讨论了 Node 的积极非阻塞 API，让我们回到并发的话题。Node 的内置非阻塞函数使用操作系统版本的回调和事件处理程序。当您调用这些函数之一时，Node 会采取行动启动操作，然后向操作系统注册某种事件处理程序，以便在操作完成时通知它。您传递给 Node 函数的回调被内部存储，以便 Node 在操作系统向其发送适当事件时调用您的回调。

这种并发通常称为基于事件的并发。在其核心，Node 有一个运行“事件循环”的单个线程。当一个 Node 程序启动时，它运行您告诉它运行的任何代码。这段代码可能调用至少一个非阻塞函数，导致注册回调或事件处理程序与操作系统。 (如果没有，那么您编写了一个同步的 Node 程序，当它到达末尾时，Node 简单地退出。) 当 Node 到达程序末尾时，它会阻塞，直到发生事件，此时操作系统再次启动它。Node 将操作系统事件映射到您注册的 JavaScript 回调，然后调用该函数。您的回调函数可能调用更多的非阻塞 Node 函数，导致注册更多的操作系统事件处理程序。一旦您的回调函数运行完毕，Node 再次进入休眠状态，循环重复。

对于 Web 服务器和其他大部分时间都在等待输入和输出的 I/O 密集型应用程序来说，这种基于事件的并发方式是高效且有效的。只要使用非阻塞 API 并且存在一种从网络套接字到 JavaScript 函数的内部映射，Web 服务器就可以同时处理来自 50 个不同客户端的请求，而无需使用 50 个不同的线程。

# 16.3 缓冲区

在 Node 中你经常会使用的一种数据类型是 Buffer 类。一个 Buffer 很像一个字符串，只不过它是一系列字节而不是一系列字符。在核心 JavaScript 支持类型化数组之前（参见 §11.2），也没有 Uint8Array 来表示无符号字节的数组。Node 定义了 Buffer 类来填补这个需求。现在 Uint8Array 是 JavaScript 语言的一部分，Node 的 Buffer 类是 Uint8Array 的子类。

Buffer 与其 Uint8Array 超类的区别在于它设计用于与 JavaScript 字符串互操作：缓冲区中的字节可以从字符字符串初始化或转换为字符字符串。字符编码将某个字符集中的每个字符映射到一个整数。给定一个文本字符串和一个字符编码，我们可以将字符串中的字符 *编码* 为一系列字节。给定一个（正确编码的）字节序列和一个字符编码，我们可以将这些字节 *解码* 为一系列字符。Node 的 Buffer 类有执行编码和解码的方法，你可以通过这些方法来识别，因为它们期望一个 `encoding` 参数来指定要使用的编码。

在 Node 中，编码是以字符串形式指定的。支持的编码有：

`"utf8"`

这是在没有指定编码时的默认编码，也是你最有可能使用的 Unicode 编码。

`"utf16le"`

两字节的 Unicode 字符，采用小端序排序。编码为 `\uffff` 以上的码点会被编码为一对两字节序列。编码 `"ucs2"` 是一个别名。

`"latin1"`

每个字符一个字节的 ISO-8859-1 编码，定义了适用于许多西欧语言的字符集。因为字节和 latin-1 字符之间有一对一的映射，所以这种编码也被称为 `"binary"`。

`"ascii"`

仅包含 7 位英文 ASCII 编码，是 `"utf8"` 编码的严格子集。

`"hex"`

这种编码将每个字节转换为一对 ASCII 十六进制数字。

`"base64"`

这种编码将每个三字节序列转换为四个 ASCII 字符。

这里有一些示例代码，演示如何使用 Buffer 以及如何进行字符串和 Buffer 之间的转换：

```js
let b = Buffer.from([0x41, 0x42, 0x43]);          // <Buffer 41 42 43>
b.toString()                                      // => "ABC"; default "utf8"
b.toString("hex")                                 // => "414243"

let computer = Buffer.from("IBM3111", "ascii");   // Convert string to Buffer
for(let i = 0; i < computer.length; i++) {        // Use Buffer as byte array
    computer[i]--;                                // Buffers are mutable
}
computer.toString("ascii")                        // => "HAL2000"
computer.subarray(0,3).map(x=>x+1).toString()     // => "IBM"

// Create new "empty" buffers with Buffer.alloc()
let zeros = Buffer.alloc(1024);                   // 1024 zeros
let ones = Buffer.alloc(128, 1);                  // 128 ones
let dead = Buffer.alloc(1024, "DEADBEEF", "hex"); // Repeating pattern of bytes

// Buffers have methods for reading and writing multi-byte values
// from and to a buffer at any specified offset.
dead.readUInt32BE(0)       // => 0xDEADBEEF
dead.readUInt32BE(1)       // => 0xADBEEFDE
dead.readBigUInt64BE(6)    // => 0xBEEFDEADBEEFDEADn
dead.readUInt32LE(1020)    // => 0xEFBEADDE
```

如果你编写一个实际操作二进制数据的 Node 程序，你可能会大量使用 Buffer 类。另一方面，如果你只是处理从文件或网络读取或写入的文本，那么你可能只会遇到 Buffer 作为数据的中间表示。许多 Node API 可以将输入或返回输出作为字符串或 Buffer 对象。通常，如果你从这些 API 中传递一个字符串，或者期望返回一个字符串，你需要指定要使用的文本编码的名称。如果你这样做了，那么你可能根本不需要使用 Buffer 对象。

# 16.4 事件和 EventEmitter

正如描述的那样，Node 的所有 API 默认都是异步的。对于其中的许多 API，这种异步性采用的形式是两个参数的错误优先回调，当请求的操作完成时调用。但是一些更复杂的 API 是基于事件的。当 API 设计围绕对象而不是函数时，或者当需要多次调用回调函数时，或者当可能需要多种类型的回调函数时，通常会出现这种情况。例如，考虑 `net.Server` 类：这种类型的对象是一个服务器套接字，用于接受来自客户端的传入连接。当它首次开始监听连接时，会发出“listening”事件，每当客户端连接时会发出“connection”事件，当关闭并不再监听时会发出“close”事件。

在 Node 中，发出事件的对象是 EventEmitter 的实例或 EventEmitter 的子类：

```js
const EventEmitter = require("events"); // Module name does not match class name
const net = require("net");
let server = new net.Server();          // create a Server object
server instanceof EventEmitter          // => true: Servers are EventEmitters
```

EventEmitters 的主要特点是它们允许您使用 `on()` 方法注册事件处理程序函数。EventEmitters 可以发出多种类型的事件，事件类型通过名称标识。要注册事件处理程序，请调用 `on()` 方法，传递事件类型的名称以及当事件发生时应该调用的函数。EventEmitters 可以使用任意数量的参数调用处理程序函数，您需要阅读特定 EventEmitter 的特定类型事件的文档，以了解您应该期望传递的参数：

```js
const net = require("net");
let server = new net.Server();          // create a Server object
server.on("connection", socket => {     // Listen for "connection" events
    // Server "connection" events are passed a socket object
    // for the client that just connected. Here we send some data
    // to the client and disconnect.
    socket.end("Hello World", "utf8");
});
```

如果您更喜欢使用更明确的方法名称来注册事件侦听器，也可以使用 `addListener()`。您可以使用 `off()` 或 `removeListener()` 来删除先前注册的事件侦听器。作为特例，您可以通过调用 `once()` 而不是 `on()` 来注册一个在第一次触发后将自动删除的事件侦听器。

当特定类型的事件发生在特定的 EventEmitter 对象上时，Node 会调用该 EventEmitter 上当前注册的所有处理程序函数来处理该类型的事件。它们按照从第一个注册到最后注册的顺序依次调用。如果有多个处理程序函数，它们将在单个线程上依次调用：请记住，Node 中没有并行处理。重要的是，事件处理函数是同步调用的，而不是异步调用的。这意味着 `emit()` 方法不会将事件处理程序排队以在以后的某个时间调用。`emit()` 会依次调用所有已注册的处理程序，并且在最后一个事件处理程序返回之前不会返回。

实际上，这意味着当内置的 Node API 发出事件时，该 API 基本上会阻塞在您的事件处理程序上。如果编写一个调用像 `fs.readFileSync()` 这样的阻塞函数的事件处理程序，直到同步文件读取完成，将不会发生进一步的事件处理。如果您的程序是一个需要响应的网络服务器之类的程序，那么重要的是保持事件处理程序函数非阻塞和快速。如果需要在事件发生时进行大量计算，通常最好使用处理程序使用 `setTimeout()` 异步调度该计算（参见 §11.10）。Node 还定义了 `setImmediate()`，它会在处理完所有挂起的回调和事件后立即调度一个函数。

EventEmitter 类还定义了一个`emit()`方法，导致注册的事件处理程序函数被调用。如果您正在定义自己的基于事件的 API，这很有用，但在使用现有 API 进行编程时通常不常用。`emit()`必须以事件类型的名称作为第一个参数调用。传递给`emit()`的任何其他参数都成为注册的事件处理程序函数的参数。处理程序函数还使用设置为 EventEmitter 对象本身的`this`值调用，这通常很方便。（请记住，箭头函数总是使用定义它们的上下文的`this`值，并且不能使用任何其他`this`值调用。尽管如此，箭头函数通常是编写事件处理程序的最方便方式。）

事件处理程序函数返回的任何值都会被忽略。但是，如果事件处理程序函数抛出异常，则它会从`emit()`调用中传播出去，并阻止执行任何在抛出异常之后注册的处理程序函数。

请记住，Node 的基于回调的 API 使用错误优先回调，重要的是您始终检查第一个回调参数以查看是否发生错误。对于基于事件的 API，等效的是“error”事件。由于基于事件的 API 通常用于网络和其他形式的流式 I/O，它们容易受到不可预测的异步错误的影响，大多数 EventEmitters 在发生错误时定义了一个“error”事件。每当使用基于事件的 API 时，您应该养成注册“error”事件处理程序的习惯。“error”事件在 EventEmitter 类中得到特殊处理。如果调用`emit()`来发出“error”事件，并且没有为该事件类型注册处理程序，则将抛出异常。由于这是异步发生的，因此您无法在`catch`块中处理异常，因此这种错误通常会导致程序退出。

# 16.5 流

在实现处理数据的算法时，几乎总是最容易将所有数据读入内存，进行处理，然后将数据写出。例如，您可以编写一个 Node 函数来复制文件，就像这样。¹

```js
const fs = require("fs");

// An asynchronous but nonstreaming (and therefore inefficient) function.
function copyFile(sourceFilename, destinationFilename, callback) {
    fs.readFile(sourceFilename, (err, buffer) => {
        if (err) {
            callback(err);
        } else {
            fs.writeFile(destinationFilename, buffer, callback);
        }
    });
}
```

这个`copyFile()`函数使用异步函数和回调函数，因此不会阻塞，并适用于像服务器这样的并发程序。但请注意，它必须分配足够的内存来一次性容纳整个文件的内容。在某些情况下这可能没问题，但如果要复制的文件非常大，或者您的程序高度并发且可能同时复制许多文件时，它就会开始出现问题。这个`copyFile()`实现的另一个缺点是它在完成读取旧文件之前无法开始写入新文件。

解决这些问题的方法是使用流算法，其中数据“流”进入您的程序，被处理，然后流出您的程序。思路是您的算法以小块处理数据，完整数据集不会一次性保存在内存中。当流式解决方案可行时，它们更节省内存，并且也可能更快。Node 的网络 API 是基于流的，Node 的文件系统模块为读取和写入文件定义了流 API，因此您可能会在编写的许多 Node 程序中使用流 API。我们将在“流动模式”中看到`copyFile()`函数的流式版本。

Node 支持四种基本的流类型：

可读

可读流是数据的来源。例如，由`fs.createReadStream()`返回的流是可以读取指定文件内容的流。`process.stdin`是另一个可读流，返回标准输入的数据。

可写

可写流是数据的接收端或目的地。例如，`fs.createWriteStream()` 的返回值是一个可写流：它允许以块的形式向其写入数据，并将所有数据输出到指定的文件。

双工

双工流将可读流和可写流合并为一个对象。例如，`net.connect()` 返回的 Socket 对象和其他 Node 网络 API 返回的对象都是双工流。如果向套接字写入数据，则数据将通过网络发送到套接字连接的计算机。如果从套接字读取数据，则可以访问另一台计算机写入的数据。

转换

转换流也是可读和可写的，但与双工流有一个重要的区别：写入转换流的数据变得可读，通常以某种转换形式从同一流中读取。例如，`zlib.createGzip()` 函数返回一个转换流，用于压缩（使用 *gzip* 算法）写入其中的数据。类似地，`crypto.createCipheriv()` 函数返回一个转换流，用于加密或解密写入其中的数据。

默认情况下，流读取和写入缓冲区。如果调用可读流的 `setEncoding()` 方法，它将返回解码后的字符串，而不是 Buffer 对象。如果向可写缓冲区写入字符串，它将自动使用缓冲区的默认编码或您指定的任何编码进行编码。Node 的流 API 还支持“对象模式”，其中流读取和写入比缓冲区和字符串更复杂的对象。Node 的核心 API 都不使用此对象模式，但您可能会在其他库中遇到它。

可读流必须从某处读取数据，可写流必须将数据写入某处，因此每个流都有两个端点：一个输入和一个输出，或者一个源和一个目的地。基于流的 API 的棘手之处在于流的两端几乎总是以不同的速度流动。也许从流中读取数据的代码想要比实际写入流中的数据更快地读取和处理数据。或者反过来：也许数据被写入流中的速度比从流的另一端读取和提取数据的速度更快。流实现几乎总是包含一个内部缓冲区，用于保存已写入但尚未读取的数据。缓冲有助于确保在请求时有可读取的数据，并且在写入数据时有空间可用于保存数据。但是这两件事情都无法保证，基于流的编程的本质是读取者有时必须等待数据被写入（因为流缓冲区为空），写入者有时必须等待数据被读取（因为流缓冲区已满）。

在使用基于线程的并发性编程环境中，流式 API 通常具有阻塞调用：读取数据的调用在数据到达流之前不会返回，写入数据的调用会阻塞，直到流的内部缓冲区有足够的空间来容纳新数据。然而，在基于事件的并发模型中，阻塞调用是没有意义的，Node 的流式 API 是基于事件和回调的。与其他 Node API 不同，本章后面将描述的方法没有“Sync”版本。

通过事件协调流的可读性（缓冲区不为空）和可写性（缓冲区不满）的需求使得 Node 的流式 API 稍显复杂。这一复杂性加剧了这些 API 多年来的演变和变化：对于可读流，有两种完全不同的 API 可供使用。尽管复杂，但值得理解和掌握 Node 的流式 API，因为它们能够在程序中实现高吞吐量的 I/O。

接下来的小节演示了如何从 Node 的流类中读取和写入。

## 16.5.1 管道

有时，您需要从流中读取数据，然后将相同的数据写入另一个流。例如，想象一下，您正在编写一个简单的 HTTP 服务器，用于提供静态文件目录。在这种情况下，您需要从文件输入流中读取数据，并将其写入网络套接字。但是，您可以简单地将两个套接字连接在一起作为“管道”，让 Node 为您处理复杂性，而不是编写自己的处理读取和写入的代码。只需将可写流传递给可读流的`pipe()`方法：

```js
const fs = require("fs");

function pipeFileToSocket(filename, socket) {
    fs.createReadStream(filename).pipe(socket);
}
```

以下实用函数将一个流导向另一个流，并在完成或发生错误时调用回调函数：

```js
function pipe(readable, writable, callback) {
    // First, set up error handling
    function handleError(err) {
        readable.close();
        writable.close();
        callback(err);
    }

    // Next define the pipe and handle the normal termination case
    readable
        .on("error", handleError)
        .pipe(writable)
        .on("error", handleError)
        .on("finish", callback);
}
```

转换流在管道中特别有用，并创建涉及两个以上流的管道。以下是一个压缩文件的示例函数：

```js
const fs = require("fs");
const zlib = require("zlib");

function gzip(filename, callback) {
    // Create the streams
    let source = fs.createReadStream(filename);
    let destination = fs.createWriteStream(filename + ".gz");
    let gzipper = zlib.createGzip();

    // Set up the pipeline
    source
        .on("error", callback)   // call callback on read error
        .pipe(gzipper)
        .pipe(destination)
        .on("error", callback)   // call callback on write error
        .on("finish", callback); // call callback when writing is complete
}
```

使用`pipe()`方法将数据从一个可读流复制到一个可写流很容易，但在实践中，通常需要以某种方式处理数据，因为它在程序中流动。一种方法是实现自己的转换流来进行处理，这种方法允许您避免手动读取和写入流。例如，下面是一个类似 Unix `grep`实用程序的函数：它从输入流中读取文本行，但只写入与指定正则表达式匹配的行：

```js
const stream = require("stream");

class GrepStream extends stream.Transform {
    constructor(pattern) {
        super({decodeStrings: false});// Don't convert strings back to buffers
        this.pattern = pattern;       // The regular expression we want to match
        this.incompleteLine = "";     // Any remnant of the last chunk of data
    }

    // This method is invoked when there is a string ready to be
    // transformed. It should pass transformed data to the specified
    // callback function. We expect string input so this stream should
    // only be connected to readable streams that have had
    // setEncoding() called on them.
    _transform(chunk, encoding, callback) {
        if (typeof chunk !== "string") {
            callback(new Error("Expected a string but got a buffer"));
            return;
        }
        // Add the chunk to any previously incomplete line and break
        // everything into lines
        let lines = (this.incompleteLine + chunk).split("\n");

        // The last element of the array is the new incomplete line
        this.incompleteLine = lines.pop();

        // Find all matching lines
        let output = lines                     // Start with all complete lines,
            .filter(l => this.pattern.test(l)) // filter them for matches,
            .join("\n");                       // and join them back up.

        // If anything matched, add a final newline
        if (output) {
            output += "\n";
        }

        // Always call the callback even if there is no output
        callback(null, output);
    }

    // This is called right before the stream is closed.
    // It is our chance to write out any last data.
    _flush(callback) {
        // If we still have an incomplete line, and it matches
        // pass it to the callback
        if (this.pattern.test(this.incompleteLine)) {
            callback(null, this.incompleteLine + "\n");
        }
    }
}

// Now we can write a program like 'grep' with this class.
let pattern = new RegExp(process.argv[2]); // Get a RegExp from command line.
process.stdin                              // Start with standard input,
    .setEncoding("utf8")                   // read it as Unicode strings,
    .pipe(new GrepStream(pattern))         // pipe it to our GrepStream,
    .pipe(process.stdout)                  // and pipe that to standard out.
    .on("error", () => process.exit());    // Exit gracefully if stdout closes.
```

## 16.5.2 异步迭代

在 Node 12 及更高版本中，可读流是异步迭代器，这意味着在`async`函数中，您可以使用`for/await`循环从流中读取字符串或缓冲区块，使用的代码结构类似于同步代码。 （有关异步迭代器和`for/await`循环的更多信息，请参见§13.4。）

使用异步迭代器几乎和使用`pipe()`方法一样简单，当您需要以某种方式处理每个读取的块时，可能更容易。以下是我们如何使用`async`函数和`for/await`循环重写前一节中的`grep`程序的方法：

```js
// Read lines of text from the source stream, and write any lines
// that match the specified pattern to the destination stream.
async function grep(source, destination, pattern, encoding="utf8") {
    // Set up the source stream for reading strings, not Buffers
    source.setEncoding(encoding);

    // Set an error handler on the destination stream in case standard
    // output closes unexpectedly (when piping output to `head`, e.g.)
    destination.on("error", err => process.exit());

    // The chunks we read are unlikely to end with a newline, so each will
    // probably have a partial line at the end. Track that here
    let incompleteLine = "";

    // Use a for/await loop to asynchronously read chunks from the input stream
    for await (let chunk of source) {
        // Split the end of the last chunk plus this one into lines
        let lines = (incompleteLine + chunk).split("\n");
        // The last line is incomplete
        incompleteLine = lines.pop();
        // Now loop through the lines and write any matches to the destination
        for(let line of lines) {
            if (pattern.test(line)) {
                destination.write(line + "\n", encoding);
            }
        }
    }
    // Finally, check for a match on any trailing text.
    if (pattern.test(incompleteLine)) {
        destination.write(incompleteLine + "\n", encoding);
    }
}

let pattern = new RegExp(process.argv[2]);   // Get a RegExp from command line.
grep(process.stdin, process.stdout, pattern) // Call the async grep() function.
    .catch(err => {                          // Handle asynchronous exceptions.
        console.error(err);
        process.exit();
    });
```

## 16.5.3 写入流和处理背压

前面代码示例中的异步`grep()`函数演示了如何将可读流用作异步迭代器，但它还演示了您可以通过将其传递给`write()`方法来简单地向可写流写入数据。`write()`方法将缓冲区或字符串作为第一个参数。 （对象流期望其他类型的对象，但超出了本章的范围。）如果传递缓冲区，则将直接写入该缓冲区的字节。如果传递字符串，则在写入之前将其编码为字节的缓冲区。当您将字符串作为`write()`的唯一参数传递时，可写流具有默认编码。默认编码通常为“utf8”，但您可以通过在可写流上调用`setDefaultEncoding()`来显式设置它。或者，当您将字符串作为`write()`的第一个参数传递时，可以将编码名称作为第二个参数传递。

`write()`可选地将回调函数作为其第三个参数。当数据实际写入并不再位于可写流的内部缓冲区中时，将调用此函数。 （如果发生错误，也可能调用此回调，但不能保证。您应在可写流上注册“error”事件处理程序以检测错误。）

`write()`方法具有非常重要的返回值。当您在流上调用`write()`时，它将始终接受并缓冲您传递的数据块。如果内部缓冲区尚未满，则返回`true`。或者，如果缓冲区现在已满或过满，则返回`false`。此返回值是建议性的，您可以忽略它——如果您继续调用`write()`，可写流将根据需要扩大其内部缓冲区。但请记住，首先使用流式 API 的原因是避免一次性在内存中保存大量数据的成本。

从`write()`方法返回`false`的返回值是一种*背压*形式：流向你发送的消息，表示你写入数据的速度比处理速度快。对这种背压的正确响应是停止调用`write()`，直到流发出“drain”事件，表示缓冲区中再次有空间。例如，下面是一个向流写入数据的函数，并在可以继续向流写入更多数据时调用回调函数：

```js
function write(stream, chunk, callback) {
    // Write the specified chunk to the specified stream
    let hasMoreRoom = stream.write(chunk);

    // Check the return value of the write() method:
    if (hasMoreRoom) {                  // If it returned true, then
        setImmediate(callback);         // invoke callback asynchronously.
    } else {                            // If it returned false, then
        stream.once("drain", callback); // invoke callback on drain event.
    }
}
```

有时可以连续调用`write()`多次，有时必须在写入之间等待事件，这导致算法变得笨拙。这就是使用`pipe()`方法如此吸引人的原因之一：当你使用`pipe()`时，Node 会自动为你处理背压。

如果你在程序中使用`await`和`async`，并将可读流视为异步迭代器，那么实现上面的`write()`实用程序的基于 Promise 的版本以正确处理背压是很简单的。在我们刚刚看过的异步`grep()`函数中，我们没有处理背压。下面示例中的异步`copy()`函数演示了如何正确处理背压。请注意，此函数只是将源流中的块复制到目标流中，并调用`copy(source, destination)`就像调用`source.pipe(destination)`一样：

```js
// This function writes the specified chunk to the specified stream and
// returns a Promise that will be fulfilled when it is OK to write again.
// Because it returns a Promise, it can be used with await.
function write(stream, chunk) {
    // Write the specified chunk to the specified stream
    let hasMoreRoom = stream.write(chunk);

    if (hasMoreRoom) {                     // If buffer is not full, return
        return Promise.resolve(null);      // an already resolved Promise object
    } else {
        return new Promise(resolve => {    // Otherwise, return a Promise that
            stream.once("drain", resolve); // resolves on the drain event.
        });
    }
}

// Copy data from the source stream to the destination stream
// respecting backpressure from the destination stream.
// This is much like calling source.pipe(destination).
async function copy(source, destination) {
    // Set an error handler on the destination stream in case standard
    // output closes unexpectedly (when piping output to `head`, e.g.)
    destination.on("error", err => process.exit());

    // Use a for/await loop to asynchronously read chunks from the input stream
    for await (let chunk of source) {
        // Write the chunk and wait until there is more room in the buffer.
        await write(destination, chunk);
    }
}

// Copy standard input to standard output
copy(process.stdin, process.stdout);
```

在我们结束对流写入的讨论之前，再次注意，不响应背压可能导致程序使用的内存超出预期，当可写流的内部缓冲区溢出并不断增大时。如果你正在编写一个网络服务器，这可能是一个远程可利用的安全问题。假设你编写了一个通过网络传输文件的 HTTP 服务器，但你没有使用`pipe()`，也没有花时间处理`write()`方法的背压。攻击者可以编写一个 HTTP 客户端，发起对大文件（如图像）的请求，但实际上从未读取请求的主体。由于客户端没有从网络中读取数据，而服务器也没有响应背压，服务器上的缓冲区将会溢出。如果攻击者有足够的并发连接，这可能会演变成一个拒绝服务攻击，使你的服务器变慢甚至崩溃。

## 16.5.4 使用事件读取流

Node 的可读流有两种模式，每种模式都有自己的读取 API。如果你的程序不能使用管道或异步迭代，你将需要选择这两种基于事件的 API 之一来处理流。重要的是你只使用其中一种 API，不要混合使用这两种 API。

### 流动模式

在*流动模式*中，当可读数据到达时，它会立即以“data”事件的形式发出。要在此模式下从流中读取数据，只需为“data”事件注册事件处理程序，流将在数据块（缓冲区或字符串）可用时将其推送给你。请注意，在流动模式下不需要调用`read()`方法：你只需要处理“data”事件。请注意，新创建的流不会立即处于流动模式。注册“data”事件处理程序会将流切换到流动模式。方便的是，这意味着流在注册第一个“data”事件处理程序之前不会发出“data”事件。

如果你正在使用流模式从可读流中读取数据，处理数据，然后将其写入可写流，那么你可能需要处理可写流的背压。如果`write()`方法返回`false`表示写入缓冲区已满，你可以在可读流上调用`pause()`来暂时停止`data`事件。然后，当你从可写流中收到“drain”事件时，你可以在可读流上调用`resume()`来重新开始`data`事件的流动。

流在流动模式下在达到流的末尾时会发出一个“end”事件。这个事件表示不会再发出更多的“data”事件。并且，像所有流一样，如果发生错误，将会发出一个“error”事件。

在流部分的开头，我们展示了一个非流式的`copyFile()`函数，并承诺会有一个更好的版本。以下代码展示了如何实现一个使用流动模式 API 并处理背压的流式`copyFile()`函数。这本来更容易通过`pipe()`调用来实现，但在这里作为协调从一个流到另一个流的数据流的多个事件处理程序的有用演示。

```js
const fs = require("fs");

// A streaming file copy function, using "flowing mode".
// Copies the contents of the named source file to the named destination file.
// On success, invokes the callback with a null argument. On error,
// invokes the callback with an Error object.
function copyFile(sourceFilename, destinationFilename, callback) {
    let input = fs.createReadStream(sourceFilename);
    let output = fs.createWriteStream(destinationFilename);

    input.on("data", (chunk) => {          // When we get new data,
        let hasRoom = output.write(chunk); // write it to the output stream.
        if (!hasRoom) {                    // If the output stream is full
            input.pause();                 // then pause the input stream.
        }
    });
    input.on("end", () => {                // When we reach the end of input,
        output.end();                      // tell the output stream to end.
    });
    input.on("error", err => {             // If we get an error on the input,
        callback(err);                     // call the callback with the error
        process.exit();                    // and quit.
    });

    output.on("drain", () => {             // When the output is no longer full,
        input.resume();                    // resume data events on the input
    });
    output.on("error", err => {            // If we get an error on the output,
        callback(err);                     // call the callback with the error
        process.exit();                    // and quit.
    });
    output.on("finish", () => {            // When output is fully written
        callback(null);                    // call the callback with no error.
    });
}

// Here's a simple command-line utility to copy files
let from = process.argv[2], to = process.argv[3];
console.log(`Copying file ${from} to ${to}...`);
copyFile(from, to, err => {
    if (err) {
        console.error(err);
    } else {
        console.log("done.");
    }
});
```

### 暂停模式

可读流的另一种模式是“暂停模式”。这是流开始的模式。如果你从未注册“data”事件处理程序，也从未调用`pipe()`方法，那么可读流将保持在暂停模式。在暂停模式下，流不会以“data”事件的形式向你推送数据。相反，你需要通过显式调用其`read()`方法来从流中拉取数据。这不是一个阻塞调用，如果流上没有可读数据，它将返回`null`。由于没有同步 API 来等待数据，暂停模式 API 也是基于事件的。在暂停模式下，当流上有数据可读时，可读流会发出“readable”事件。作为响应，你的代码应该调用`read()`方法来读取数据。你必须在循环中这样做，重复调用`read()`直到它返回`null`。这样完全排空流的缓冲区是必要的，以便在将来触发新的“readable”事件。如果在仍然有可读数据时停止调用`read()`，你将不会收到另一个“readable”事件，你的程序可能会挂起。

暂停模式下的流会像流动模式下的流一样发出“end”和“error”事件。如果你正在编写一个从可读流读取数据并将其写入可写流的程序，那么暂停模式可能不是一个好选择。为了正确处理背压，你只想在输入流可读且输出流没有积压时才读取。在暂停模式下，这意味着读取和写入直到`read()`返回`null`或`write()`返回`false`，然后在`readable`或`drain`事件上重新开始读取或写入。这是不够优雅的，你可能会发现在这种情况下流动模式（或管道）更容易。

以下代码演示了如何计算指定文件内容的 SHA256 哈希。它使用一个处于暂停模式的可读流以块的形式读取文件的内容，然后将每个块传递给计算哈希的对象。（请注意，在 Node 12 及更高版本中，使用`for/await`循环编写此函数会更简单。）

```js
const fs = require("fs");
const crypto = require("crypto");

// Compute a sha256 hash of the contents of the named file and pass the
// hash (as a string) to the specified error-first callback function.
function sha256(filename, callback) {
    let input = fs.createReadStream(filename); // The data stream.
    let hasher = crypto.createHash("sha256");  // For computing the hash.

    input.on("readable", () => {         // When there is data ready to read
        let chunk;
        while(chunk = input.read()) {    // Read a chunk, and if non-null,
            hasher.update(chunk);        // pass it to the hasher,
        }                                // and keep looping until not readable
    });
    input.on("end", () => {              // At the end of the stream,
        let hash = hasher.digest("hex"); // compute the hash,
        callback(null, hash);            // and pass it to the callback.
    });
    input.on("error", callback);         // On error, call callback
}

// Here's a simple command-line utility to compute the hash of a file
sha256(process.argv[2], (err, hash) => { // Pass filename from command line.
    if (err) {                           // If we get an error
        console.error(err.toString());   // print it as an error.
    } else {                             // Otherwise,
        console.log(hash);               // print the hash string.
    }
});
```

# 16.6 进程、CPU 和操作系统详细信息

全局 Process 对象具有许多有用的属性和函数，通常与当前运行的 Node 进程的状态有关。请查阅 Node 文档以获取完整详情，但以下是一些你应该知道的属性和函数：

```js
process.argv            // An array of command-line arguments.
process.arch            // The CPU architecture: "x64", for example.
process.cwd()           // Returns the current working directory.
process.chdir()         // Sets the current working directory.
process.cpuUsage()      // Reports CPU usage.
process.env             // An object of environment variables.
process.execPath        // The absolute filesystem path to the node executable.
process.exit()          // Terminates the program.
process.exitCode        // An integer code to be reported when the program exits.
process.getuid()        // Return the Unix user id of the current user.
process.hrtime.bigint() // Return a "high-resolution" nanosecond timestamp.
process.kill()          // Send a signal to another process.
process.memoryUsage()   // Return an object with memory usage details.
process.nextTick()      // Like setImmediate(), invoke a function soon.
process.pid             // The process id of the current process.
process.ppid            // The parent process id.
process.platform        // The OS: "linux", "darwin", or "win32", for example.
process.resourceUsage() // Return an object with resource usage details.
process.setuid()        // Sets the current user, by id or name.
process.title           // The process name that appears in `ps` listings.
process.umask()         // Set or return the default permissions for new files.
process.uptime()        // Return Node's uptime in seconds.
process.version         // Node's version string.
process.versions        // Version strings for the libraries Node depends on.
```

“os”模块（与`process`不同，需要使用`require()`显式加载）提供了关于 Node 运行的计算机和操作系统的类似低级细节的访问。你可能永远不需要使用这些功能中的任何一个，但值得知道 Node 提供了它们：

```js
const os = require("os");
os.arch()              // Returns CPU architecture. "x64" or "arm", for example.
os.constants           // Useful constants such as os.constants.signals.SIGINT.
os.cpus()              // Data about system CPU cores, including usage times.
os.endianness()        // The CPU's native endianness "BE" or "LE".
os.EOL                 // The OS native line terminator: "\n" or "\r\n".
os.freemem()           // Returns the amount of free RAM in bytes.
os.getPriority()       // Returns the OS scheduling priority of a process.
os.homedir()           // Returns the current user's home directory.
os.hostname()          // Returns the hostname of the computer.
os.loadavg()           // Returns the 1, 5, and 15-minute load averages.
os.networkInterfaces() // Returns details about available network. connections.
os.platform()          // Returns OS: "linux", "darwin", or "win32", for example.
os.release()           // Returns the version number of the OS.
os.setPriority()       // Attempts to set the scheduling priority for a process.
os.tmpdir()            // Returns the default temporary directory.
os.totalmem()          // Returns the total amount of RAM in bytes.
os.type()              // Returns OS: "Linux", "Darwin", or "Windows_NT", e.g.
os.uptime()            // Returns the system uptime in seconds.
os.userInfo()          // Returns uid, username, home, and shell of current user.
```

# 16.7 处理文件

Node 的“fs”模块是一个用于处理文件和目录的全面 API。它由“path”模块补充，后者定义了用于处理文件和目录名称的实用函数。“fs”模块包含一些高级函数，用于轻松读取、写入和复制文件。但是，该模块中的大多数函数都是低级 JavaScript 绑定到 Unix 系统调用（以及它们在 Windows 上的等效物）。如果之前有过低级文件系统调用的经验（在 C 或其他语言中），那么 Node API 对你来说将是熟悉的。如果没有，你可能会发现“fs”API 的某些部分很简洁和不直观。例如，删除文件的函数称为`unlink()`。

“fs”模块定义了一个庞大的 API，主要是因为通常每个基本操作都有多个变体。正如本章开头所讨论的，大多数函数（如`fs.readFile()`）都是非阻塞的、基于回调的和异步的。通常情况下，每个函数都有一个同步阻塞的变体，比如`fs.readFileSync()`。在 Node 10 及更高版本中，许多这些函数还有基于 Promise 的异步变体，比如`fs.promises.readFile()`。大多数“fs”函数的第一个参数是一个字符串，指定要操作的文件的路径（文件名加可选的目录名）。但是其中一些函数也支持一个以整数“文件描述符”作为第一个参数而不是路径的变体。这些变体的名称以字母“f”开头。例如，`fs.truncate()`截断由路径指定的文件，而`fs.ftruncate()`截断由文件描述符指定的文件。还有一个基于 Promise 的`fs.promises.truncate()`，它期望一个路径，还有另一个基于 Promise 的版本，它作为 FileHandle 对象的方法实现。（FileHandle 类相当于 Promise-based API 中的文件描述符。）最后，在“fs”模块中有一些函数的变体的名称以字母“l”开头。这些“l”变体类似于基本函数，但不会遵循文件系统中的符号链接，而是直接操作符号链接本身。

## 16.7.1 路径、文件描述符和 FileHandles

要使用“fs”模块处理文件，首先需要能够命名要处理的文件。文件通常由*路径*指定，这意味着文件本身的名称，以及文件所在的目录层次结构。如果路径是*绝对*的，这意味着指定了一直到文件系统根目录的所有目录。否则，路径是*相对*的，只有与其他路径相关时才有意义，通常是*当前工作目录*。处理路径可能有点棘手，因为不同的操作系统使用不同的字符来分隔目录名称，当连接路径时很容易意外加倍这些分隔符字符，并且`../`父目录路径段需要特殊处理。Node 的“path”模块和其他几个重要的 Node 功能有所帮助：

```js
// Some important paths
process.cwd()      // Absolute path of the current working directory.
__filename         // Absolute path of the file that holds the current code.
__dirname          // Absolute path of the directory that holds __filename.
os.homedir()       // The user's home directory.

const path = require("path");

path.sep                         // Either "/" or "\" depending on your OS

// The path module has simple parsing functions
let p = "src/pkg/test.js";       // An example path
path.basename(p)                 // => "test.js"
path.extname(p)                  // => ".js"
path.dirname(p)                  // => "src/pkg"
path.basename(path.dirname(p))   // => "pkg"
path.dirname(path.dirname(p))    // => "src"

// normalize() cleans up paths:
path.normalize("a/b/c/../d/")    // => "a/b/d/": handles ../ segments
path.normalize("a/./b")          // => "a/b": strips "./" segments
path.normalize("//a//b//")       // => "/a/b/": removes duplicate /

// join() combines path segments, adding separators, then normalizes
path.join("src", "pkg", "t.js")  // => "src/pkg/t.js"

// resolve() takes one or more path segments and returns an absolute
// path. It starts with the last argument and works backward, stopping
// when it has built an absolute path or resolving against process.cwd().
path.resolve()                   // => process.cwd()
path.resolve("t.js")             // => path.join(process.cwd(), "t.js")
path.resolve("/tmp", "t.js")     // => "/tmp/t.js"
path.resolve("/a", "/b", "t.js") // => "/b/t.js"
```

请注意，`path.normalize()`只是一个字符串操作函数，没有访问实际文件系统。`fs.realpath()`和`fs.realpathSync()`函数执行文件系统感知的规范化：它们解析符号链接并解释相对于当前工作目录的相对路径名。

在前面的示例中，我们假设代码在基于 Unix 的操作系统上运行，`path.sep`是“/”。如果想在 Windows 系统上使用 Unix 风格的路径，可以使用`path.posix`而不是`path`。反之，如果想在 Unix 系统上使用 Windows 路径，可以使用`path.win32`。`path.posix`和`path.win32`定义了与`path`本身相同的属性和函数。

我们将在接下来的章节中介绍一些“fs”函数，它们期望一个*文件描述符*而不是文件名。文件描述符是作为操作系统级别引用“打开”文件的整数。通过调用`fs.open()`（或`fs.openSync()`）函数，你可以为给定的名称获取一个描述符。进程一次只能打开有限数量的文件，因此当你使用完文件描述符时，调用`fs.close()`是很重要的。如果你想要使用最底层的`fs.read()`和`fs.write()`函数，允许你在文件中跳转，不同时间读取和写入文件的位，你需要打开文件。在“fs”模块中有其他使用文件描述符的函数，但它们都有基于名称的版本，只有当你打算打开文件进行读取或写入时，才真正有意义使用基于描述符的函数。

最后，在`fs.promises`定义的基于 Promise 的 API 中，`fs.open()`的等价物是`fs.promises.open()`，它返回一个解析为 FileHandle 对象的 Promise。这个 FileHandle 对象用于与文件描述符具有相同的目的。然而，除非你需要使用 FileHandle 的最底层的`read()`和`write()`方法，否则真的没有理由创建一个。如果你确实创建了一个 FileHandle，记得在使用完毕后调用它的`close()`方法。

## 16.7.2 读取文件

Node 允许你一次性读取文件内容，通过流，或使用低级别的 API。

如果你的文件很小，或者内存使用和性能不是最高优先级，那么通常最容易的方法是一次性读取整个文件的内容。你可以同步地、通过回调或 Promise 来做到这一点。默认情况下，你会得到文件的字节作为缓冲区，但如果指定了编码，你将得到一个解码后的字符串。

```js
const fs = require("fs");
let buffer = fs.readFileSync("test.data");      // Synchronous, returns buffer
let text = fs.readFileSync("data.csv", "utf8"); // Synchronous, returns string

// Read the bytes of the file asynchronously
fs.readFile("test.data", (err, buffer) => {
    if (err) {
        // Handle the error here
    } else {
        // The bytes of the file are in buffer
    }
});

// Promise-based asynchronous read
fs.promises
    .readFile("data.csv", "utf8")
    .then(processFileText)
    .catch(handleReadError);

// Or use the Promise API with await inside an async function
async function processText(filename, encoding="utf8") {
    let text = await fs.promises.readFile(filename, encoding);
    // ... process the text here...
}
```

如果你能够按顺序处理文件的内容，并且不需要同时将文件的整个内容保存在内存中，那么通过流来读取文件可能是最有效的方法。我们已经广泛讨论了流：这里是你如何使用流和`pipe()`方法将文件的内容写入标准输出的示例：

```js
function printFile(filename, encoding="utf8") {
    fs.createReadStream(filename, encoding).pipe(process.stdout);
}
```

最后，如果你需要对从文件中读取的字节以及何时读取它们进行低级别的控制，你可以打开一个文件以获取文件描述符，然后使用`fs.read()`、`fs.readSync()`或`fs.promises.read()`从文件的指定源位置读取指定数量的字节到指定的缓冲区的指定目标位置：

```js
const fs = require("fs");

// Reading a specific portion of a data file
fs.open("data", (err, fd) => {
    if (err) {
        // Report error somehow
        return;
    }
    try {
        // Read bytes 20 through 420 into a newly allocated buffer.
        fs.read(fd, Buffer.alloc(400), 0, 400, 20, (err, n, b) => {
            // err is the error, if any.
            // n is the number of bytes actually read
            // b is the buffer that they bytes were read into.
        });
    }
    finally {          // Use a finally clause so we always
        fs.close(fd);  // close the open file descriptor
    }
});
```

如果你需要从文件中读取多个数据块，基于回调的`read()`API 使用起来很麻烦。如果你可以使用同步 API（或基于 Promise 的 API 与`await`），那么从文件中读取多个数据块变得很容易：

```js
const fs = require("fs");

function readData(filename) {
    let fd = fs.openSync(filename);
    try {
        // Read the file header
        let header = Buffer.alloc(12); // A 12 byte buffer
        fs.readSync(fd, header, 0, 12, 0);

        // Verify the file's magic number
        let magic = header.readInt32LE(0);
        if (magic !== 0xDADAFEED) {
            throw new Error("File is of wrong type");
        }

        // Now get the offset and length of the data from the header
        let offset = header.readInt32LE(4);
        let length = header.readInt32LE(8);

        // And read those bytes from the file
        let data = Buffer.alloc(length);
        fs.readSync(fd, data, 0, length, offset);
        return data;
    } finally {
        // Always close the file, even if an exception is thrown above
        fs.closeSync(fd);
    }
}
```

## 16.7.3 写入文件

在 Node 中写入文件与读取文件非常相似，但有一些额外的细节需要了解。其中一个细节是，创建一个新文件的方式就是简单地向一个尚不存在的文件名写入。

与读取类似，Node 中有三种基本的写入文件的方式。如果文件的整个内容是一个字符串或缓冲区，你可以使用`fs.writeFile()`（基于回调）、`fs.writeFileSync()`（同步）或`fs.promises.writeFile()`（基于 Promise）一次性写入整个内容：

```js
fs.writeFileSync(path.resolve(__dirname, "settings.json"),
                 JSON.stringify(settings));
```

如果要写入文件的数据是字符串，并且想要使用除了“utf8”之外的编码，请将编码作为可选的第三个参数传递。

相关的函数`fs.appendFile()`、`fs.appendFileSync()`和`fs.promises.appendFile()`类似，但当指定的文件已经存在时，它们会将数据追加到末尾而不是覆盖现有文件内容。

如果要写入文件的数据不是一个块，或者不是同时在内存中的所有数据，那么使用 Writable 流是一个不错的方法，假设您计划从头到尾写入数据而不跳过文件中的位置：

```js
const fs = require("fs");
let output = fs.createWriteStream("numbers.txt");
for(let i = 0; i < 100; i++) {
    output.write(`${i}\n`);
}
output.end();
```

最后，如果您想要将数据写入文件的多个块，并且希望能够控制写入每个块的确切位置，那么可以使用`fs.open()`、`fs.openSync()`或`fs.promises.open()`打开文件，然后使用结果文件描述符与`fs.write()`或`fs.writeSync()`函数。这些函数有不同形式的字符串和缓冲区。字符串变体接受文件描述符、字符串和要写入该字符串的文件位置（可选的第四个参数为编码）。缓冲区变体接受文件描述符、缓冲区、偏移量和长度，指定缓冲区内的数据块，并指定要写入该块的字节的文件位置。如果您有要写入的 Buffer 对象数组，可以使用单个`fs.writev()`或`fs.writevSync()`。使用`fs.promises.open()`和它生成的 FileHandle 对象写入缓冲区和字符串存在类似的低级函数。

你可以使用`fs.truncate()`、`fs.truncateSync()`或`fs.promises.truncate()`来截断文件的末尾。这些函数以路径作为第一个参数，长度作为第二个参数，并修改文件使其具有指定的长度。如果省略长度，则使用零，并且文件变为空。尽管这些函数的名称是这样的，但它们也可以用于扩展文件：如果指定的长度比当前文件大小长，文件将扩展为零字节到新大小。如果您已经打开要修改的文件，可以使用带有文件描述符或 FileHandle 的`ftruncate()`或`ftruncateSync()`。

这里描述的各种文件写入函数在数据“写入”后返回或调用其回调或解析其 Promise，这意味着 Node 已将数据交给操作系统。但这并不一定意味着数据实际上已经写入到持久存储中：至少您的一些数据可能仍然在操作系统中的某个地方或设备驱动程序中缓冲，等待写入磁盘。如果调用`fs.writeSync()`同步将一些数据写入文件，并且在函数返回后立即发生停电，您可能仍会丢失数据。如果要强制将数据写入磁盘，以确保它已经安全保存，使用`fs.fsync()`或`fs.fsyncSync()`。这些函数仅适用于文件描述符：没有基于路径的版本。

## 16.7.4 文件操作

Node 的流类的前面讨论包括两个`copyFile()`函数的示例。这些不是您实际使用的实用程序，因为“fs”模块定义了自己的`fs.copyFile()`方法（当然还有`fs.copyFileSync()`和`fs.promises.copyFile()`）。

这些函数将原始文件的名称和副本的名称作为它们的前两个参数。这些可以指定为字符串或 URL 或缓冲区对象。可选的第三个参数是一个整数，其位指定控制`copy`操作细节的标志。对于基于回调的`fs.copyFile()`，最后一个参数是在复制完成时不带参数调用的回调函数，或者如果出现错误则带有错误参数调用。以下是一些示例：

```js
// Basic synchronous file copy.
fs.copyFileSync("ch15.txt", "ch15.bak");

// The COPYFILE_EXCL argument copies only if the new file does not already
// exist. It prevents copies from overwriting existing files.
fs.copyFile("ch15.txt", "ch16.txt", fs.constants.COPYFILE_EXCL, err => {
    // This callback will be called when done. On error, err will be non-null.
});

// This code demonstrates the Promise-based version of the copyFile function.
// Two flags are combined with the bitwise OR opeartor |. The flags mean that
// existing files won't be overwritten, and that if the filesystem supports
// it, the copy will be a copy-on-write clone of the original file, meaning
// that no additional storage space will be required until either the original
// or the copy is modified.
fs.promises.copyFile("Important data",
                     `Important data ${new Date().toISOString()}"
 fs.constants.COPYFILE_EXCL | fs.constants.COPYFILE_FICLONE)
 .then(() => {
 console.log("Backup complete");
 });
 .catch(err => {
 console.error("Backup failed", err);
 });
```

`fs.rename()`函数（以及通常的同步和基于 Promise 的变体）移动和/或重命名文件。调用它时，传入当前文件的路径和所需的新文件路径。没有标志参数，但基于回调的版本将回调作为第三个参数：

```js
fs.renameSync("ch15.bak", "backups/ch15.bak");
```

请注意，没有标志可以防止重命名覆盖现有文件。同时请记住，文件只能在文件系统内重命名。

函数`fs.link()`和`fs.symlink()`及其变体具有与`fs.rename()`相同的签名，并且类似于`fs.copyFile()`，只是它们分别创建硬链接和符号链接，而不是创建副本。

最后，`fs.unlink()`、`fs.unlinkSync()`和`fs.promises.unlink()`是 Node 用于删除文件的函数。（这种不直观的命名是从 Unix 继承而来，其中删除文件基本上是创建其硬链接的相反操作。）调用此函数并传递一个回调（如果使用基于回调的版本）来删除要删除的文件的字符串、缓冲区或 URL 路径：

```js
fs.unlinkSync("backups/ch15.bak");
```

## 16.7.5 文件元数据

`fs.stat()`、`fs.statSync()`和`fs.promises.stat()`函数允许您获取指定文件或目录的元数据。例如：

```js
const fs = require("fs");
let stats = fs.statSync("book/ch15.md");
stats.isFile()         // => true: this is an ordinary file
stats.isDirectory()    // => false: it is not a directory
stats.size             // file size in bytes
stats.atime            // access time: Date when it was last read
stats.mtime            // modification time: Date when it was last written
stats.uid              // the user id of the file's owner
stats.gid              // the group id of the file's owner
stats.mode.toString(8) // the file's permissions, as an octal string
```

返回的 Stats 对象包含其他更隐晦的属性和方法，但此代码演示了您最有可能使用的属性。

`fs.lstat()`及其变体的工作方式与`fs.stat()`完全相同，只是如果指定的文件是符号链接，则 Node 将返回链接本身的元数据，而不是跟随链接。

如果您已打开文件以生成文件描述符或 FileHandle 对象，则可以使用`fs.fstat()`或其变体获取已打开文件的元数据信息，而无需再次指定文件名。

除了使用`fs.stat()`及其所有变体查询元数据外，还有用于更改元数据的函数。

`fs.chmod()`、`fs.lchmod()`和`fs.fchmod()`（以及同步和基于 Promise 的版本）设置文件或目录的“模式”或权限。模式值是整数，其中每个位具有特定含义，并且在八进制表示法中最容易理解。例如，要使文件对其所有者只读且对其他人不可访问，请使用`0o400`：

```js
fs.chmodSync("ch15.md", 0o400);  // Don't delete it accidentally!
```

`fs.chown()`、`fs.lchown()`和`fs.fchown()`（以及同步和基于 Promise 的版本）设置文件或目录的所有者和组（作为 ID）。 （这很重要，因为它们与`fs.chmod()`设置的文件权限交互。）

最后，您可以使用`fs.utimes()`和`fs.futimes()`及其变体设置文件或目录的访问时间和修改时间。

## 16.7.6 处理目录

在 Node 中创建新目录，使用`fs.mkdir()`、`fs.mkdirSync()`或`fs.promises.mkdir()`。第一个参数是要创建的目录的路径。可选的第二个参数可以是指定新目录的模式（权限位）的整数。或者您可以传递一个带有可选`mode`和`recursive`属性的对象。如果`recursive`为`true`，则此函数将创建路径中尚不存在的任何目录：

```js
// Ensure that dist/ and dist/lib/ both exist.
fs.mkdirSync("dist/lib", { recursive: true });
```

`fs.mkdtemp()`及其变体接受您提供的路径前缀，将一些随机字符附加到其后（这对安全性很重要），创建一个以该名称命名的目录，并将目录路径返回（或传递给回调）给您。

要删除一个目录，使用`fs.rmdir()`或其变体之一。请注意，在删除之前目录必须为空：

```js
// Create a random temporary directory and get its path, then
// delete it when we are done
let tempDirPath;
try {
    tempDirPath = fs.mkdtempSync(path.join(os.tmpdir(), "d"));
    // Do something with the directory here
} finally {
    // Delete the temporary directory when we're done with it
    fs.rmdirSync(tempDirPath);
}
```

“fs”模块为列出目录内容提供了两种不同的 API。首先，`fs.readdir()`、`fs.readdirSync()`和`fs.promises.readdir()`一次性读取整个目录，并向您提供一个字符串数组或指定每个项目的名称和类型（文件或目录）的 Dirent 对象数组。这些函数返回的文件名只是文件的本地名称，而不是整个路径。以下是示例：

```js
let tempFiles = fs.readdirSync("/tmp");  // returns an array of strings

// Use the Promise-based API to get a Dirent array, and then
// print the paths of subdirectories
fs.promises.readdir("/tmp", {withFileTypes: true})
    .then(entries => {
        entries.filter(entry => entry.isDirectory())
            .map(entry => entry.name)
            .forEach(name => console.log(path.join("/tmp/", name)));
    })
    .catch(console.error);
```

如果你预计需要列出可能有数千条条目的目录，你可能更喜欢 `fs.opendir()` 及其变体的流式处理方法。这些函数返回表示指定目录的 Dir 对象。你可以使用 Dir 对象的 `read()` 或 `readSync()` 方法逐个读取 Dirent。如果向 `read()` 传递一个回调函数，它将调用该回调。如果省略回调参数，它将返回一个 Promise。当没有更多目录条目时，你将得到 `null` 而不是 Dirent 对象。

使用 Dir 对象最简单的方法是作为异步迭代器与 `for/await` 循环一起使用。以下是一个使用流式 API 列出目录条目、对每个条目调用 `stat()` 并打印文件和目录名称及大小的函数示例：

```js
const fs = require("fs");
const path = require("path");

async function listDirectory(dirpath) {
    let dir = await fs.promises.opendir(dirpath);
    for await (let entry of dir) {
        let name = entry.name;
        if (entry.isDirectory()) {
            name += "/";  // Add a trailing slash to subdirectories
        }
        let stats = await fs.promises.stat(path.join(dirpath, name));
        let size = stats.size;
        console.log(String(size).padStart(10), name);
    }
}
```

# 16.8 HTTP 客户端和服务器

Node 的 “http”，“https” 和 “http2” 模块是完整功能但相对低级的 HTTP 协议实现。它们定义了全面的 API 用于实现 HTTP 客户端和服务器。由于这些 API 相对较低级，本章节无法覆盖所有功能。但接下来的示例演示了如何编写基本的客户端和服务器。

发起基本的 HTTP GET 请求的最简单方法是使用 `http.get()` 或 `https.get()`。这些函数的第一个参数是要获取的 URL。（如果是一个 `http://` URL，你必须使用 “http” 模块，如果是一个 `https://` URL，你必须使用 “https” 模块。）第二个参数是一个回调函数，当服务器的响应开始到达时将调用该回调，并传入一个 IncomingMessage 对象。当回调被调用时，HTTP 状态和头部信息是可用的，但正文可能还没有准备好。IncomingMessage 对象是一个可读流，你可以使用本章前面演示的技术从中读取响应正文。

§13.2.6 结尾的 `getJSON()` 函数使用了 `http.get()` 函数作为 `Promise()` 构造函数演示的一部分。现在你已经了解了 Node 流和 Node 编程模型，值得重新访问该示例，看看如何使用 `http.get()`。

`http.get()` 和 `https.get()` 是稍微简化的 `http.request()` 和 `https.request()` 函数的变体。以下的 `postJSON()` 函数演示了如何使用 `https.request()` 发起包含 JSON 请求体的 HTTPS POST 请求。与 第十三章 的 `getJSON()` 函数一样，它期望一个 JSON 响应，并返回一个解析后的该响应的 Promise：

```js
const https = require("https");

/*
 * Convert the body object to a JSON string then HTTPS POST it to the
 * specified API endpoint on the specified host. When the response arrives,
 * parse the response body as JSON and resolve the returned Promise with
 * that parsed value.
 */
function postJSON(host, endpoint, body, port, username, password) {
    // Return a Promise object immediately, then call resolve or reject
    // when the HTTPS request succeeds or fails.
    return new Promise((resolve, reject) => {
        // Convert the body object to a string
        let bodyText = JSON.stringify(body);

        // Configure the HTTPS request
        let requestOptions = {
            method: "POST",       // Or "GET", "PUT", "DELETE", etc.
            host: host,           // The host to connect to
            path: endpoint,       // The URL path
            headers: {            // HTTP headers for the request
                "Content-Type": "application/json",
                "Content-Length": Buffer.byteLength(bodyText)
            }
        };

        if (port) {                      // If a port is specified,
            requestOptions.port = port;  // use it for the request.
        }
        // If credentials are specified, add an Authorization header.
        if (username && password) {
            requestOptions.auth = `${username}:${password}`;
        }

        // Now create the request based on the configuration object
        let request = https.request(requestOptions);

        // Write the body of the POST request and end the request.
        request.write(bodyText);
        request.end();

        // Fail on request errors (such as no network connection)
        request.on("error", e => reject(e));

        // Handle the response when it starts to arrive.
        request.on("response", response => {
            if (response.statusCode !== 200) {
                reject(new Error(`HTTP status ${response.statusCode}`));
                // We don't care about the response body in this case, but
                // we don't want it to stick around in a buffer somewhere, so
                // we put the stream into flowing mode without registering
                // a "data" handler so that the body is discarded.
                response.resume();
                return;
            }

            // We want text, not bytes. We're assuming the text will be
            // JSON-formatted but aren't bothering to check the
            // Content-Type header.
            response.setEncoding("utf8");

            // Node doesn't have a streaming JSON parser, so we read the
            // entire response body into a string.
            let body = "";
            response.on("data", chunk => { body += chunk; });

            // And now handle the response when it is complete.
            response.on("end", () => {          // When the response is done,
                try {                           // try to parse it as JSON
                    resolve(JSON.parse(body));  // and resolve the result.
                } catch(e) {                    // Or, if anything goes wrong,
                    reject(e);                  // reject with the error
                }
            });
        });
    });
}
```

除了发起 HTTP 和 HTTPS 请求， “http” 和 “https” 模块还允许你编写响应这些请求的服务器。基本的方法如下：

+   创建一个新的 Server 对象。

+   调用其 `listen()` 方法开始监听指定端口的请求。

+   为 “request” 事件注册一个事件处理程序，使用该处理程序来读取客户端的请求（特别是 `request.url` 属性），并编写你的响应。

接下来的代码创建了一个简单的 HTTP 服务器，从本地文件系统提供静态文件，并实现了一个调试端点，通过回显客户端的请求来响应。

```js
// This is a simple static HTTP server that serves files from a specified
// directory. It also implements a special /test/mirror endpoint that
// echoes the incoming request, which can be useful when debugging clients.
const http = require("http");   // Use "https" if you have a certificate
const url = require("url");     // For parsing URLs
const path = require("path");   // For manipulating filesystem paths
const fs = require("fs");       // For reading files

// Serve files from the specified root directory via an HTTP server that
// listens on the specified port.
function serve(rootDirectory, port) {
    let server = new http.Server();  // Create a new HTTP server
    server.listen(port);             // Listen on the specified port
    console.log("Listening on port", port);

    // When requests come in, handle them with this function
    server.on("request", (request, response) => {
        // Get the path portion of the request URL, ignoring
        // any query parameters that are appended to it.
        let endpoint = url.parse(request.url).pathname;

        // If the request was for "/test/mirror", send back the request
        // verbatim. Useful when you need to see the request headers and body.
        if (endpoint === "/test/mirror") {
            // Set response header
            response.setHeader("Content-Type", "text/plain; charset=UTF-8");

            // Specify response status code
            response.writeHead(200);  // 200 OK

            // Begin the response body with the request
            response.write(`${request.method} ${request.url} HTTP/${
                               request.httpVersion
                           }\r\n`);

            // Output the request headers
            let headers = request.rawHeaders;
            for(let i = 0; i < headers.length; i += 2) {
                response.write(`${headers[i]}: ${headers[i+1]}\r\n`);
            }

            // End headers with an extra blank line
            response.write("\r\n");

            // Now we need to copy any request body to the response body
            // Since they are both streams, we can use a pipe
            request.pipe(response);
        }
        // Otherwise, serve a file from the local directory.
        else {
            // Map the endpoint to a file in the local filesystem
            let filename = endpoint.substring(1); // strip leading /
            // Don't allow "../" in the path because it would be a security
            // hole to serve anything outside the root directory.
            filename = filename.replace(/\.\.\//g, "");
            // Now convert from relative to absolute filename
            filename = path.resolve(rootDirectory, filename);

            // Now guess the type file's content type based on extension
            let type;
            switch(path.extname(filename))  {
            case ".html":
            case ".htm": type = "text/html"; break;
            case ".js":  type = "text/javascript"; break;
            case ".css": type = "text/css"; break;
            case ".png": type = "image/png"; break;
            case ".txt": type = "text/plain"; break;
            default:     type = "application/octet-stream"; break;
            }

            let stream = fs.createReadStream(filename);
            stream.once("readable", () => {
                // If the stream becomes readable, then set the
                // Content-Type header and a 200 OK status. Then pipe the
                // file reader stream to the response. The pipe will
                // automatically call response.end() when the stream ends.
                response.setHeader("Content-Type", type);
                response.writeHead(200);
                stream.pipe(response);
            });

            stream.on("error", (err) => {
                // Instead, if we get an error trying to open the stream
                // then the file probably does not exist or is not readable.
                // Send a 404 Not Found plain-text response with the
                // error message.
                response.setHeader("Content-Type", "text/plain; charset=UTF-8");
                response.writeHead(404);
                response.end(err.message);
            });
        }
    });
}

// When we're invoked from the command line, call the serve() function
serve(process.argv[2] || "/tmp", parseInt(process.argv[3]) || 8000);
```

Node 的内置模块就足以编写简单的 HTTP 和 HTTPS 服务器。但请注意，生产服务器通常不直接构建在这些模块之上。相反，大多数复杂的服务器是使用外部库实现的——比如 Express 框架——提供了后端 web 开发人员所期望的 “中间件” 和其他更高级的实用工具。

# 16.9 非 HTTP 网络服务器和客户端

Web 服务器和客户端已经变得如此普遍，以至于很容易忘记可以编写不使用 HTTP 的客户端和服务器。 尽管 Node 以编写 Web 服务器的良好环境而闻名，但 Node 还完全支持编写其他类型的网络服务器和客户端。

如果您习惯使用流，那么网络相对简单，因为网络套接字只是一种双工流。 “net”模块定义了 Server 和 Socket 类。 要创建一个服务器，调用`net.createServer()`，然后调用生成的对象的`listen()`方法，告诉服务器在哪个端口上监听连接。 当客户端在该端口上连接时，Server 对象将生成“connection”事件，并传递给事件侦听器的值将是一个 Socket 对象。 Socket 对象是一个双工流，您可以使用它从客户端读取数据并向客户端写入数据。 在 Socket 上调用`end()`以断开连接。

编写客户端甚至更容易：将端口号和主机名传递给`net.createConnection()`以创建一个套接字，用于与在该主机上运行并在该端口上监听的任何服务器通信。 然后使用该套接字从服务器读取和写入数据。

以下代码演示了如何使用“net”模块编写服务器。 当客户端连接时，服务器讲一个 knock-knock 笑话：

```js
// A TCP server that delivers interactive knock-knock jokes on port 6789.
// (Why is six afraid of seven? Because seven ate nine!)
const net = require("net");
const readline = require("readline");

// Create a Server object and start listening for connections
let server = net.createServer();
server.listen(6789, () => console.log("Delivering laughs on port 6789"));

// When a client connects, tell them a knock-knock joke.
server.on("connection", socket => {
    tellJoke(socket)
        .then(() => socket.end())  // When the joke is done, close the socket.
        .catch((err) => {
            console.error(err);    // Log any errors that occur,
            socket.end();          // but still close the socket!
        });
});

// These are all the jokes we know.
const jokes = {
    "Boo": "Don't cry...it's only a joke!",
    "Lettuce": "Let us in! It's freezing out here!",
    "A little old lady": "Wow, I didn't know you could yodel!"
};

// Interactively perform a knock-knock joke over this socket, without blocking.
async function tellJoke(socket) {
    // Pick one of the jokes at random
    let randomElement = a => a[Math.floor(Math.random() * a.length)];
    let who = randomElement(Object.keys(jokes));
    let punchline = jokes[who];

    // Use the readline module to read the user's input one line at a time.
    let lineReader = readline.createInterface({
        input: socket,
        output: socket,
        prompt: ">> "
    });

    // A utility function to output a line of text to the client
    // and then (by default) display a prompt.
    function output(text, prompt=true) {
        socket.write(`${text}\r\n`);
        if (prompt) lineReader.prompt();
    }

    // Knock-knock jokes have a call-and-response structure.
    // We expect different input from the user at different stages and
    // take different action when we get that input at different stages.
    let stage = 0;

    // Start the knock-knock joke off in the traditional way.
    output("Knock knock!");

    // Now read lines asynchronously from the client until the joke is done.
    for await (let inputLine of lineReader) {
        if (stage === 0) {
            if (inputLine.toLowerCase() === "who's there?") {
                // If the user gives the right response at stage 0
                // then tell the first part of the joke and go to stage 1.
                output(who);
                stage = 1;
            } else  {
                // Otherwise teach the user how to do knock-knock jokes.
                output('Please type "Who\'s there?".');
            }
        } else if (stage === 1) {
            if (inputLine.toLowerCase() === `${who.toLowerCase()} who?`) {
                // If the user's response is correct at stage 1, then
                // deliver the punchline and return since the joke is done.
                output(`${punchline}`, false);
                return;
            } else {
                // Make the user play along.
                output(`Please type "${who} who?".`);
            }
        }
    }
}
```

这样的简单基于文本的服务器通常不需要一个定制的客户端。如果您的系统上安装了`nc`（“netcat”）实用程序，您可以使用它来与这个服务器通信，方法如下：

```js
$ nc localhost 6789
Knock knock!
>> Who's there?
A little old lady
>> A little old lady who?
Wow, I didn't know you could yodel!
```

另一方面，在 Node 中编写一个定制的客户端对于笑话服务器来说很容易。 我们只需连接到服务器，然后将服务器的输出导向 stdout，并将 stdin 导向服务器的输入：

```js
// Connect to the joke port (6789) on the server named on the command line
let socket = require("net").createConnection(6789, process.argv[2]);
socket.pipe(process.stdout);              // Pipe data from the socket to stdout
process.stdin.pipe(socket);               // Pipe data from stdin to the socket
socket.on("close", () => process.exit()); // Quit when the socket closes.
```

除了支持基于 TCP 的服务器，Node 的“net”模块还支持通过“Unix 域套接字”进行进程间通信，这些套接字通过文件系统路径而不是端口号进行标识。 我们不打算在本章中涵盖这种类型的套接字，但 Node 文档中有详细信息。 我们在这里没有空间涵盖的其他 Node 功能包括“dgram”模块用于基于 UDP 的客户端和服务器，以及“tls”模块，它类似于“https”对“http”的关系。 `tls.Server`和`tls.TLSSocket`类允许创建使用 SSL 加密连接的 TCP 服务器（如 knock-knock 笑话服务器），就像 HTTPS 服务器一样。

# 16.10 使用子进程进行操作

除了编写高度并发的服务器，Node 还适用于编写执行其他程序的脚本。 在 Node 中，“child_process”模块定义了许多函数，用于作为子进程运行其他程序。 本节演示了其中一些函数，从最简单的开始，逐渐过渡到更复杂的函数。

## 16.10.1 execSync()和 execFileSync()

运行另一个程序的最简单方法是使用`child_process.execSync()`。 此函数将要运行的命令作为其第一个参数。 它创建一个子进程，在该进程中运行一个 shell，并使用 shell 执行您传递的命令。 然后它阻塞，直到命令（和 shell）退出。 如果命令以错误退出，则`execSync()`会抛出异常。 否则，`execSync()`返回命令写入其 stdout 流的任何输出。 默认情况下，此返回值是一个缓冲区，但您可以在可选的第二个参数中指定编码以获得一个字符串。 如果命令将任何输出写入 stderr，则该输出将直接传递到父进程的 stderr 流。

所以，例如，如果您正在编写一个脚本，性能不是一个问题，您可能会使用`child_process.execSync()`来列出一个目录，而不是使用`fs.readdirSync()`函数：

```js
const child_process = require("child_process");
let listing = child_process.execSync("ls -l web/*.html", {encoding: "utf8"});
```

`execSync()` 调用完整的 Unix shell 意味着您传递给它的字符串可以包含多个以分号分隔的命令，并且可以利用 shell 功能，如文件名通配符、管道和输出重定向。这也意味着您必须小心，永远不要将来自用户输入或类似不受信任来源的命令传递给 `execSync()`。shell 命令的复杂语法很容易被利用，以允许攻击者运行任意代码。

如果您不需要 shell 的功能，可以通过使用 `child_process.execFileSync()` 避免启动 shell 的开销。此函数直接执行程序，而不调用 shell。但由于不涉及 shell，它无法解析命令行，您必须将可执行文件作为第一个参数传递，并将命令行参数数组作为第二个参数传递：

```js
let listing = child_process.execFileSync("ls", ["-l", "web/"],
                                         {encoding: "utf8"});
```

## 16.10.2 exec() 和 execFile()

`execSync()` 和 `execFileSync()` 函数是同步的：它们会阻塞并在子进程退出之前不返回。使用这些函数很像在终端窗口中输入 Unix 命令：它们允许您逐个运行一系列命令。但是，如果您正在编写一个需要完成多个任务且这些任务彼此不依赖的程序，那么您可能希望并行运行它们并同时运行多个命令。您可以使用异步函数 `child_process.exec()` 和 `child_process.execFile()` 来实现这一点。

`exec()` 和 `execFile()` 与它们的同步变体类似，只是它们立即返回一个代表正在运行的子进程的 ChildProcess 对象，并且它们将错误优先的回调作为最后一个参数。当子进程退出时，将调用回调，并实际上会使用三个参数调用它。第一个是错误（如果有的话）；如果进程正常终止，则为 `null`。第二个参数是发送到子进程标准输出流的收集输出。第三个参数是发送到子进程标准错误流的任何输出。

`exec()` 和 `execFile()` 返回的 ChildProcess 对象允许您终止子进程，并向其写入数据（然后可以从其标准输入读取）。当我们讨论 `child_process.spawn()` 函数时，我们将更详细地介绍 ChildProcess。

如果您计划同时执行多个子进程，则最简单的方法可能是使用 `exec()` 的“promisified”版本，它返回一个 Promise 对象，如果子进程无错误退出，则解析为具有 `stdout` 和 `stderr` 属性的对象。例如，这是一个接受 shell 命令数组作为输入并返回一个 Promise 的函数，该 Promise 解析为所有这些命令的结果：

```js
const child_process = require("child_process");
const util = require("util");
const execP = util.promisify(child_process.exec);

function parallelExec(commands) {
    // Use the array of commands to create an array of Promises
    let promises = commands.map(command => execP(command, {encoding: "utf8"}));
    // Return a Promise that will fulfill to an array of the fulfillment
    // values of each of the individual promises. (Instead of returning objects
    // with stdout and stderr properties we just return the stdout value.)
    return Promise.all(promises)
        .then(outputs => outputs.map(out => out.stdout));
}

module.exports = parallelExec;
```

## 16.10.3 spawn()

到目前为止描述的各种 `exec` 函数——同步和异步——都设计用于与快速运行且不产生大量输出的子进程一起使用。即使是异步的 `exec()` 和 `execFile()` 也不是流式的：它们在进程退出后才一次性返回进程输出。

`child_process.spawn()` 函数允许您在子进程仍在运行时流式访问子进程的输出。它还允许您向子进程写入数据（子进程将把该数据视为其标准输入流上的输入）：这意味着可以动态与子进程交互，根据其生成的输出发送输入。

`spawn()` 默认不使用 shell，因此您必须像使用 `execFile()` 一样调用它，提供要运行的可执行文件以及一个单独的命令行参数数组传递给它。`spawn()` 返回一个类似于 `execFile()` 的 ChildProcess 对象，但它不接受回调参数。您可以监听 ChildProcess 对象及其流上的事件，而不是使用回调函数。

由`spawn()`返回的 ChildProcess 对象是一个事件发射器。你可以监听“exit”事件以在子进程退出时收到通知。ChildProcess 对象还有三个流属性。`stdout`和`stderr`是可读流：当子进程写入其 stdout 和 stderr 流时，该输出通过 ChildProcess 流变得可读。请注意这里名称的倒置。在子进程中，`stdout`是一个可写输出流，但在父进程中，ChildProcess 对象的`stdout`属性是一个可读输入流。

类似地，ChildProcess 对象的`stdin`属性是一个可写流：你写入到这个流的任何内容都会在子进程的标准输入上可用。

ChildProcess 对象还定义了一个`pid`属性，指定子进程的进程 ID。它还定义了一个`kill()`方法，用于终止子进程。

## 16.10.4 fork()

`child_process.fork()`是一个专门用于在子 Node 进程中运行 JavaScript 代码模块的函数。`fork()`期望与`spawn()`相同的参数，但第一个参数应指定 JavaScript 代码文件的路径，而不是可执行二进制文件。

使用`fork()`创建的子进程可以通过其标准输入和标准输出流与父进程通信，就像在`spawn()`的前一节中描述的那样。但是，`fork()`还为父子进程之间提供了另一个更简单的通信渠道。

当你使用`fork()`创建一个子进程时，你可以使用返回的 ChildProcess 对象的`send()`方法向子进程发送一个对象的副本。你可以监听 ChildProcess 上的“message”事件来接收子进程发送的消息。在子进程中运行的代码可以使用`process.send()`向父进程发送消息，并且可以监听`process`上的“message”事件来接收父进程发送的消息。

这里，例如，是一些使用`fork()`创建子进程的代码，然后向该子进程发送消息并等待响应的代码：

```js
const child_process = require("child_process");

// Start a new node process running the code in child.js in our directory
let child = child_process.fork(`${__dirname}/child.js`);

// Send a message to the child
child.send({x: 4, y: 3});

// Print the child's response when it arrives.
child.on("message", message => {
    console.log(message.hypotenuse); // This should print "5"
    // Since we only send one message we only expect one response.
    // After we receive it we call disconnect() to terminate the connection
    // between parent and child. This allows both processes to exit cleanly.
    child.disconnect();
});
```

这里是在子进程中运行的代码：

```js
// Wait for messages from our parent process
process.on("message", message => {
    // When we receive one, do a calculation and send the result
    // back to the parent.
    process.send({hypotenuse: Math.hypot(message.x, message.y)});
});
```

启动子进程是一个昂贵的操作，子进程必须进行数量级更多的计算才能使用`fork()`和这种方式的进程间通信才有意义。如果你正在编写一个需要对传入事件非常敏感并且还需要执行耗时计算的程序，那么你可能会考虑使用一个单独的子进程来执行计算，以便它们不会阻塞事件循环并降低父进程的响应性。（尽管在这种情况下，线程—参见§16.11—可能比子进程更好的选择。）

`send()`的第一个参数将使用`JSON.stringify()`进行序列化，并在子进程中使用`JSON.parse()`进行反序列化，因此你应该只包含 JSON 格式支持的值。然而，`send()`有一个特殊的第二个参数，允许你传输 Socket 和 Server 对象（来自“net”模块）到子进程。网络服务器往往是 IO 绑定的，而不是计算绑定的，但如果你编写了一个需要进行比单个 CPU 处理更多计算的服务器，并且在拥有多个 CPU 的机器上运行该服务器，那么你可以使用`fork()`创建多个子进程来处理请求。在父进程中，你可能会监听 Server 对象上的“connection”事件，然后从该“connection”事件中获取 Socket 对象，并使用特殊的第二个参数`send()`到一个子进程中处理。（请注意，这是一个不太常见的情况的不太可能的解决方案。与编写分叉子进程的服务器相比，保持服务器单线程并在生产环境中部署多个实例来处理负载可能更简单。）

# 16.11 Worker Threads

正如本章开头所解释的，Node 的并发模型是单线程和基于事件的。但在版本 10 及更高版本中，Node 确实允许真正的多线程编程，其 API 与由 Web 浏览器定义的 Web Workers API（§15.13）非常相似。多线程编程以难度大而著称。这几乎完全是因为需要仔细同步线程对共享内存的访问。但 JavaScript 线程（无论是在 Node 还是浏览器中）默认不共享内存，因此使用线程的危险和困难不适用于 JavaScript 中的这些“工作线程”。

JavaScript 的工作线程通过消息传递进行通信，而不是使用共享内存。主线程可以通过调用表示该线程的 Worker 对象的`postMessage()`方法向工作线程发送消息。工作线程可以通过监听“message”事件来接收来自其父级的消息。工作线程可以通过自己的`postMessage()`方法向主线程发送消息，父级可以通过自己的“message”事件处理程序接收消息。示例代码将清楚地说明这是如何工作的。

有三个原因可能会让你想在 Node 应用程序中使用工作线程：

+   如果您的应用程序实际上需要进行比一个 CPU 核心处理更多的计算，那么线程可以让您在多个核心之间分配工作，这在今天的计算机上已经很普遍。如果您在 Node 中进行科学计算、机器学习或图形处理，那么您可能希望使用线程来为问题提供更多的计算能力。

+   即使您的应用程序没有充分利用一个 CPU 的全部性能，您可能仍然希望使用线程来保持主线程的响应性。考虑一个处理大型但相对不频繁请求的服务器。假设它每秒只收到一个请求，但需要大约半秒钟的（阻塞 CPU 密集型）计算来处理每个请求。平均而言，它将有 50% 的空闲时间。但当两个请求在几毫秒内同时到达时，服务器甚至无法开始响应第二个请求，直到第一个响应的计算完成。相反，如果服务器使用工作线程执行计算，服务器可以立即开始响应两个请求，并为服务器的客户提供更好的体验。假设服务器有多个 CPU 核心，它还可以并行计算两个响应的主体，但即使只有一个核心，使用工作线程仍然可以提高响应性。

+   一般来说，工作线程允许我们将阻塞的同步操作转换为非阻塞的异步操作。如果您正在编写一个依赖不可避免同步的传统代码的程序，您可以使用工作线程来避免在需要调用该传统代码时阻塞。

工作线程并不像子进程那样沉重，但也不轻量级。通常情况下，除非有大量工作要做，否则创建工作线程是没有意义的。一般来说，如果您的程序既不受 CPU 限制，也没有响应问题，那么您可能不需要工作线程。

## 16.11.1 创建工作线程并传递消息

定义工作线程的 Node 模块被称为“worker_threads”。在本节中，我们将使用标识符`threads`来引用它：

```js
const threads = require("worker_threads");
```

该模块定义了一个 Worker 类来表示一个工作线程，您可以使用`threads.Worker()`构造函数创建一个新线程。以下代码演示了如何使用此构造函数创建一个工作线程，并展示了如何从主线程向工作线程传递消息，以及从工作线程向主线程传递消息。它还演示了一个技巧，允许您将主线程代码和工作线程代码放在同一个文件中。²

```js
const threads = require("worker_threads");

// The worker_threads module exports the boolean isMainThread property.
// This property is true when Node is running the main thread and it is
// false when Node is running a worker. We can use this fact to implement
// the main and worker threads in the same file.
if (threads.isMainThread) {
    // If we're running in the main thread, then all we do is export
    // a function. Instead of performing a computationally intensive
    // task on the main thread, this function passes the task to a worker
    // and returns a Promise that will resolve when the worker is done.
    module.exports = function reticulateSplines(splines) {
        return new Promise((resolve,reject) => {
            // Create a worker that loads and runs this same file of code.
            // Note the use of the special __filename variable.
            let reticulator = new threads.Worker(__filename);

            // Pass a copy of the splines array to the worker
            reticulator.postMessage(splines);

            // And then resolve or reject the Promise when we get
            // a message or error from the worker.
            reticulator.on("message", resolve);
            reticulator.on("error", reject);
        });
    };
} else {
    // If we get here, it means we're in the worker, so we register a
    // handler to get messages from the main thread. This worker is designed
    // to only receive a single message, so we register the event handler
    // with once() instead of on(). This allows the worker to exit naturally
    // when its work is complete.
    threads.parentPort.once("message", splines => {
        // When we get the splines from the parent thread, loop
        // through them and reticulate all of them.
        for(let spline of splines) {
            // For the sake of example, assume that spline objects usually
            // have a reticulate() method that does a lot of computation.
            spline.reticulate ? spline.reticulate() : spline.reticulated = true;
        }

        // When all the splines have (finally!) been reticulated
        // pass a copy back to the main thread.
        threads.parentPort.postMessage(splines);
    });
}
```

`Worker()` 构造函数的第一个参数是要在线程中运行的 JavaScript 代码文件的路径。在上面的代码中，我们使用预定义的 `__filename` 标识符创建一个加载和运行与主线程相同文件的工作线程。不过，一般来说，你会传递一个文件路径。请注意，如果指定相对路径，则相对于 `process.cwd()`，而不是相对于当前运行的模块。如果你想要一个相对于当前模块的路径，可以使用类似 `path.resolve(__dirname, 'workers/reticulator.js')` 的方式。

`Worker()` 构造函数还可以接受一个对象作为其第二个参数，该对象的属性为工作线程提供可选配置。我们稍后会介绍其中一些选项，但现在请注意，如果将 `{eval: true}` 作为第二个参数传递，那么 `Worker()` 的第一个参数将被解释为要评估的 JavaScript 代码字符串，而不是文件名：

```js
new threads.Worker(`
 const threads = require("worker_threads");
 threads.parentPort.postMessage(threads.isMainThread);
`, {eval: true}).on("message", console.log);  // This will print "false"
```

Node 在传递给 `postMessage()` 的对象上创建一个副本，而不是直接与工作线程共享。这样可以防止工作线程和主线程共享内存。你可能会期望这种复制是通过 `JSON.stringify()` 和 `JSON.parse()`（§11.6）来完成的。但事实上，Node 借用了一种更强大的技术，即从 Web 浏览器中知名的结构化克隆算法。

结构化克隆算法可以序列化大多数 JavaScript 类型，包括 Map、Set、Date 和 RegExp 对象以及类型化数组，但通常无法复制由 Node 主机环境定义的类型，如套接字和流。然而，需要注意的是，Buffer 对象部分支持：如果你将一个 Buffer 传递给 `postMessage()`，它将被接收为 Uint8Array，并且可以使用 `Buffer.from()` 转换回 Buffer。在 “结构化克隆算法” 中了解更多关于结构化克隆算法的信息。

## 16.11.2 工作线程执行环境

在大多数情况下，Node 中的工作线程中的 JavaScript 代码运行方式与在 Node 的主线程中一样。有一些差异需要注意，其中一些差异涉及到 `Worker()` 构造函数的可选第二个参数的属性：

+   正如我们所见，`threads.isMainThread` 在主线程中为 `true`，但在任何工作线程中始终为 `false`。

+   在工作线程中，你可以使用 `threads.parentPort.postMessage()` 向父线程发送消息，使用 `threads.parentPort.on` 注册来自父线程的消息的事件处理程序。在主线程中，`threads.parentPort` 始终为 `null`。

+   在工作线程中，`threads.workerData` 被设置为 `Worker()` 构造函数的第二个参数的 `workerData` 属性的副本。在主线程中，此属性始终为 `null`。你可以使用这个 `workerData` 属性向工作线程传递一个初始消息，该消息将在工作线程启动后立即可用，这样工作线程就不必等待“message”事件才能开始工作。

+   默认情况下，在工作线程中，`process.env` 是父线程中 `process.env` 的副本。但父线程可以通过设置 `Worker()` 构造函数的第二个参数的 `env` 属性来指定一组自定义的环境变量。作为一个特殊（可能危险）的情况，父线程可以将 `env` 属性设置为 `threads.SHARE_ENV`，这将导致两个线程共享一组环境变量，以便一个线程中的更改在另一个线程中可见。

+   默认情况下，在工作线程中，`process.stdin` 流永远没有可读数据。你可以通过在 `Worker()` 构造函数的第二个参数中传递 `stdin: true` 来更改此默认行为。如果这样做，那么 Worker 对象的 `stdin` 属性将是一个可写流。父进程写入 `worker.stdin` 的任何数据在工作线程中的 `process.stdin` 上变为可读。

+   默认情况下，工作线程中的`process.stdout`和`process.stderr`流会简单地传输到父线程中对应的流。这意味着，例如，`console.log()`和`console.error()`在工作线程中的输出方式与主线程中完全相同。你可以通过在`Worker()`构造函数的第二个参数中传递`stdout:true`或`stderr:true`来覆盖此默认行为。如果这样做，那么工作线程写入这些流的任何输出都可以在父线程的`worker.stdout`和`worker.stderr`流中读取到。（这里存在一个潜在的令人困惑的流方向倒置，我们在本章前面的子进程中也看到了相同的情况：工作线程的输出流是父线程的输入流，工作线程的输入流是父线程的输出流。）

+   如果工作线程调用`process.exit()`，只有该线程退出，整个进程不会退出。

+   工作线程不允许更改它们所属进程的共享状态。当从工作线程调用`process.chdir()`和`process.setuid()`等函数时，会抛出异常。

+   操作系统信号（如`SIGINT`和`SIGTERM`）只会传递给主线程；它们无法在工作线程中接收或处理。

## 16.11.3 通信通道和 MessagePorts

创建新的工作线程时，会同时创建一个通信通道，允许工作线程和父线程之间传递消息。正如我们所见，工作线程使用`threads.parentPort`与父线程发送和接收消息，父线程使用 Worker 对象与工作线程发送和接收消息。

工作线程 API 还允许使用由 Web 浏览器定义并在 §15.13.5 中介绍的 MessageChannel API 创建自定义通信通道。如果你已经阅读了该部分，接下来的内容会让你感到很熟悉。

假设一个工作线程需要处理主线程中两个不同模块发送的两种不同消息。这两个不同模块可以共享默认通道，并使用`worker.postMessage()`发送消息，但如果每个模块都有自己的私有通道向工作线程发送消息会更清晰。或者考虑主线程创建两个独立工作线程的情况。自定义通信通道可以让这两个工作线程直接相互通信，而不必通过父线程发送所有消息。

使用`MessageChannel()`构造函数创建一个新的消息通道。一个 MessageChannel 对象有两个属性，名为`port1`和`port2`。这些属性指向一对 MessagePort 对象。在其中一个端口上调用`postMessage()`将导致另一个端口生成“message”事件，并携带 Message 对象的结构化克隆：

```js
const threads = require("worker_threads");
let channel = new threads.MessageChannel();
channel.port2.on("message", console.log);  // Log any messages we receive
channel.port1.postMessage("hello");        // Will cause "hello" to be printed
```

你也可以在任一端口上调用`close()`来断开两个端口之间的连接，并表示不会再交换更多消息。当任一端口上调用`close()`时，将向两个端口传递“close”事件。

注意，上面的代码示例创建了一对 MessagePort 对象，然后使用这些对象在主线程内传输消息。为了在工作线程中使用自定义通信通道，我们必须将两个端口中的一个从创建它的线程传输到将要使用它的线程。下一节将解释如何做到这一点。

## 16.11.4 传输 MessagePorts 和 Typed Arrays

`postMessage()` 函数使用结构化克隆算法，正如我们所指出的，它不能复制像 SSockets 和 Streams 这样的对象。它可以处理 MessagePort 对象，但只能使用一种特殊技术作为特例。`postMessage()` 方法（Worker 对象的方法，`threads.parentPort` 的方法，或任何 MessagePort 对象的方法）接受一个可选的第二个参数。这个参数（称为 `transferList`）是一个要在线程之间传输而不是复制的对象数组。

MessagePort 对象不能被结构化克隆算法复制，但可以被传输。如果 `postMessage()` 的第一个参数包含了一个或多个 MessagePorts（在 Message 对象中任意深度嵌套），那么这些 MessagePort 对象也必须作为第二个参数传递的数组的成员出现。这样做告诉 Node 不需要复制 MessagePort，并且可以直接将现有对象交给另一个线程。然而，关于在线程之间传输值的关键是，一旦值被传输，它就不能再在调用 `postMessage()` 的线程中使用。

下面是如何创建一个新的 MessageChannel 并将其中一个 MessagePort 传输给工作线程的方法：

```js
// Create a custom communication channel
const threads = require("worker_threads");
let channel = new threads.MessageChannel();

// Use the worker's default channel to transfer one end of the new
// channel to the worker. Assume that when the worker receives this
// message it immediately begins to listen for messages on the new channel.
worker.postMessage({ command: "changeChannel", data: channel.port1 },
                   [ channel.port1 ]);

// Now send a message to the worker using our end of the custom channel
channel.port2.postMessage("Can you hear me now?");

// And listen for responses from the worker as well
channel.port2.on("message", handleMessagesFromWorker);
```

MessagePort 对象并不是唯一可以传输的对象。如果你使用一个类型化数组作为消息调用 `postMessage()`（或者消息中包含一个或多个任意深度嵌套的类型化数组），那么这个类型化数组（或这些类型化数组）将会被结构化克隆算法简单地复制。但是类型化数组可能很大；例如，如果你正在使用一个工作线程对数百万像素进行图像处理。因此，为了效率起见，`postMessage()` 还给了我们传输类型化数组而不是复制它们的选项。（线程默认共享内存。JavaScript 中的工作线程通常避免共享内存，但当我们允许这种受控传输时，可以非常高效地完成。）这种安全性的保证在于，当一个类型化数组被传输到另一个线程时，它在传输它的线程中将变得无法使用。在图像处理场景中，主线程可以将图像的像素传输给工作线程，然后工作线程在完成后可以将处理后的像素传回主线程。内存不需要被复制，但永远不会被两个线程同时访问。

要传输一个类型化数组而不是复制它，将支持数组的 ArrayBuffer 包含在 `postMessage()` 的第二个参数中：

```js
let pixels = new Uint32Array(1024*1024);  // 4 megabytes of memory

// Assume we read some data into this typed array, and then transfer the
// pixels to a worker without copying. Note that we don't put the array
// itself in the transfer list, but the array's Buffer object instead.
worker.postMessage(pixels, [ pixels.buffer ]);
```

与传输的 MessagePort 一样，一旦传输了一个类型化数组，它就变得无法使用。如果尝试使用已经传输的 MessagePort 或类型化数组，不会抛出异常；当与它们交互时，这些对象只是停止执行任何操作。

## 16.11.5 在线程之间共享类型化数组

除了在线程之间传输类型化数组，实际上还可以在线程之间共享类型化数组。只需创建一个所需大小的 SharedArrayBuffer，然后使用该缓冲区创建一个类型化数组。当通过 `postMessage()` 传递由 SharedArrayBuffer 支持的类型化数组时，底层内存将在线程之间共享。在这种情况下，不应该将共享缓冲区包含在 `postMessage()` 的第二个参数中。

然而，你真的不应该这样做，因为 JavaScript 从未考虑过线程安全，并且多线程编程非常难以正确实现。（这也是为什么 SharedArrayBuffer 没有在 §11.2 中涵盖的原因：它是一个难以正确实现的小众功能。）即使简单的 `++` 运算符也不是线程安全的，因为它需要读取一个值，递增它，然后写回。如果两个线程同时递增一个值，它通常只会被递增一次，如下面的代码所示：

```js
const threads = require("worker_threads");

if (threads.isMainThread) {
    // In the main thread, we create a shared typed array with
    // one element. Both threads will be able to read and write
    // sharedArray[0] at the same time.
    let sharedBuffer = new SharedArrayBuffer(4);
    let sharedArray = new Int32Array(sharedBuffer);

    // Now create a worker thread, passing the shared array to it with
    // as its initial workerData value so we don't have to bother with
    // sending and receiving a message
    let worker = new threads.Worker(__filename, { workerData: sharedArray });

    // Wait for the worker to start running and then increment the
    // shared integer 10 million times.
    worker.on("online", () => {
        for(let i = 0; i < 10_000_000; i++) sharedArray[0]++;

        // Once we're done with our increments, we start listening for
        // message events so we know when the worker is done.
        worker.on("message", () => {
            // Although the shared integer has been incremented
            // 20 million times, its value will generally be much less.
            // On my computer the final value is typically under 12 million.
            console.log(sharedArray[0]);
        });
    });
} else {
    // In the worker thread, we get the shared array from workerData
    // and then increment it 10 million times.
    let sharedArray = threads.workerData;
    for(let i = 0; i < 10_000_000; i++) sharedArray[0]++;
    // When we're done incrementing, let the main thread know
    threads.parentPort.postMessage("done");
}
```

有一种情况下可能合理使用 SharedArrayBuffer，即当两个线程在共享内存的完全不同部分上操作时。你可以通过创建两个作为非重叠区域视图的类型化数组来强制执行这一点，然后让你的两个线程使用这两个单独的类型化数组。例如，可以这样执行并行归并排序：一个线程对数组的下半部分进行排序，另一个线程对数组的上半部分进行排序。或者某些类型的图像处理算法也适合这种方法：多个线程在图像的不同区域上工作。

如果你确实需要允许多个线程访问共享数组的同一区域，你可以通过使用 Atomics 对象定义的函数向线程安全迈出一步。当 SharedArrayBuffer 添加到 JavaScript 时，Atomics 也被添加以定义共享数组元素上的原子操作。例如，`Atomics.add()`函数读取共享数组的指定元素，将指定值添加到其中，并将总和写回数组。它以原子方式执行此操作，就好像它是一个单独的操作，并确保在操作进行时没有其他线程可以读取或写入该值。`Atomics.add()`允许我们重新编写我们刚刚查看的并获得正确结果的并行增量代码，即对共享数组元素进行 2000 万次增量：

```js
const threads = require("worker_threads");

if (threads.isMainThread) {
    let sharedBuffer = new SharedArrayBuffer(4);
    let sharedArray = new Int32Array(sharedBuffer);
    let worker = new threads.Worker(__filename, { workerData: sharedArray });

    worker.on("online", () => {
        for(let i = 0; i < 10_000_000; i++) {
            Atomics.add(sharedArray, 0, 1);  // Threadsafe atomic increment
        }

        worker.on("message", (message) => {
            // When both threads are done, use a threadsafe function
            // to read the shared array and confirm that it has the
            // expected value of 20,000,000.
            console.log(Atomics.load(sharedArray, 0));
        });
    });
} else {
    let sharedArray = threads.workerData;
    for(let i = 0; i < 10_000_000; i++) {
        Atomics.add(sharedArray, 0, 1);      // Threadsafe atomic increment
    }
    threads.parentPort.postMessage("done");
}
```

这个新版本的代码正确地打印出数字 20,000,000。但它比它替换的不正确代码慢大约九倍。在一个线程中执行所有 2000 万次增量会更简单、更快。还要注意，原子操作可能能够确保图像处理算法的线程安全，其中每个数组元素都是完全独立于所有其他值的值。但在大多数实际程序中，多个数组元素通常彼此相关，并且需要某种高级别的线程同步。低级别的`Atomics.wait()`和`Atomics.notify()`函数可以帮助解决这个问题，但本书不涉及它们的使用讨论。

# 16.12 总结

尽管 JavaScript 是为在 Web 浏览器中运行而创建的，但 Node 已经将 JavaScript 变成了一种通用编程语言。它特别受欢迎用于实现 Web 服务器，但它与操作系统的深层绑定意味着它也是 shell 脚本的一个很好的替代品。

这一长章节涵盖的最重要主题包括：

+   Node 的默认异步 API 和其单线程、回调和基于事件的并发风格。

+   Node 的基本数据类型、缓冲区和流。

+   Node 的“fs”和“path”模块用于处理文件系统。

+   Node 的“http”和“https”模块用于编写 HTTP 客户端和服务器。

+   Node 的“net”模块用于编写非 HTTP 客户端和服务器。

+   Node 的“child_process”模块用于创建和与子进程通信。

+   Node 的“worker_threads”模块用于使用消息传递而不是共享内存进行真正的多线程编程。

¹ Node 定义了一个`fs.copyFile()`函数，实际上你会在实践中使用它。

² 将工作代码定义在一个单独的文件中通常更清晰、更简单。但当我第一次遇到 Unix 的`fork()`系统调用时，两个线程运行同一文件的不同部分的技巧让我大吃一惊。我认为值得演示这种技术，仅仅因为它的奇怪优雅。
