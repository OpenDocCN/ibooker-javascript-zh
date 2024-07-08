# 第十七章：Node 基础

“旧”和“新” JavaScript 之间的分界线是在 Node.js（主要简称为 Node）发布到世界上时发生的。是的，动态修改页面元素的能力是一个重要的里程碑，以及强调为 ECMAScript 的新版本确立前进路径，但真正让我们以全新的方式看待 JavaScript 的是 Node。而我很喜欢这种方式——我是 Node 和服务器端 JavaScript 开发的铁杆粉丝。

在本章中，我们将探讨 Node 的基础知识。至少，您需要已安装 Node，如“安装 npm 包管理器（与 Node.js 一起）”或“使用 Node Version Manager 管理 Node 版本”中所述。

# 使用 Node Version Manager 管理 Node 版本

## 问题

您需要在开发机器上安装和管理多个 Node 版本。

## 解决方案

使用[Node Version Manager (NVM)](https://github.com/nvm-sh/nvm)，它允许您在每个 shell 基础上安装和使用任何分发版本的 Node。NVM 兼容 Linux、macOS 和 Windows Subsystem for Linux。

要安装 NVM，请在系统的终端应用程序中使用`curl`或`wget`运行安装脚本：

```
## using curl:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash

## using wget:
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash
```

###### 注意

如果您在 Windows 上进行开发，我们建议使用[`nvm-windows`](https://github.com/coreybutler/nvm-windows)，它与 NVM 项目无关，但提供了类似的功能来支持 Windows 操作系统。有关如何使用`nvm-windows`的说明，请参阅该项目的文档。

安装完 NVM 后，您需要安装一个 Node 版本。要安装最新版本的 Node，请运行：

```
$ nvm install node
```

您还可以安装特定版本的 Node：

```
# install the latest path release of a major version
$ nvm install 15

# install a specific major/minor/patch version
$ nvm install 15.6.0
```

安装完 Node 后，您需要为新的 shell 会话设置一个默认版本。这可以是已安装的最新版本 Node 或一个特定的版本号：

```
# default new shell sessions to the latest version of node
nvm alias default node
# default new shell sessions to a specific version
nvm alias default 14
```

要在 shell 会话中切换使用的版本，请使用`nvm use`命令，然后跟上一个特定已安装的版本：

```
$ nvm use 15
```

## 讨论

使用 NVM 允许您在操作系统上轻松下载和切换多个 Node 版本。这在处理支持多个版本和遗留代码库的库时非常有用。它还简化了在开发环境中管理 Node。您可以查看每个发布的[发布和支持时间表](https://oreil.ly/9IY83)。

使用 NVM 可以通过`nvm ls`命令列出您机器上安装的所有版本。这将显示所有已安装的版本、新 shell 会话的默认版本以及您未安装的任何 LTS 版本：

```
$ nvm ls
         v8.1.2
        v8.11.3
       v10.13.0
->     v10.23.1
        v12.8.0
       v12.20.0
       v12.20.1
        v13.5.0
       v14.14.0
       v14.15.1
       v14.15.4
        v15.6.0
         system
default -> 14 (-> v14.15.4)
node -> stable (-> v15.6.0) (default)
stable -> 15.6 (-> v15.6.0) (default)
iojs -> N/A (default)
unstable -> N/A (default)
lts/* -> lts/fermium (-> v14.15.4)
lts/argon -> v4.9.1 (-> N/A)
lts/boron -> v6.17.1 (-> N/A)
lts/carbon -> v8.17.0 (-> N/A)
lts/dubnium -> v10.23.1
lts/erbium -> v12.20.1
lts/fermium -> v14.15.4
```

正如您所见，我在我的机器上安装了几个重复的主要版本的冗余补丁版本。要卸载和移除特定版本，您可以使用`nvm uninstall`命令：

```
nvm uninstall 14.14
```

要追踪项目设计用于使用哪个 Node 版本可能是个挑战。为了简化这一点，您可以在项目的根目录下添加一个 `.nvmrc` 文件。文件的内容是项目设计使用的 Node 版本。例如：

```
# default to the latest LTS version
$ lts/*

# to use a specific version
$ 14.15.4
```

要使用项目 `.nvmrc` 文件中指定的版本，请从目录的根目录运行 `nvm use` 命令。

###### 提示

对于大型项目，使用 Docker 等容器技术是确保版本匹配的极其有用的方式，包括部署。 Node 文档中有一篇关于 [将 Node.js Web 应用程序 Docker 化](https://oreil.ly/phXQZ) 的有用指南。

# 响应简单的浏览器请求

## 问题

您想创建一个能够响应非常基本浏览器请求的 Node 应用程序。

## 解决方案

使用内置的 Node HTTP 服务器来响应请求：

```
// load http module
const http = require('http');

// create http server
http
  .createServer((req, res) => {
    // content header
    res.writeHead(200, { 'content-type': 'text/plain' });

    // write message and signal communication is complete
    res.end('Hello, World!');
  })
  .listen(8124);

console.log('Server running on port 8124');
```

## 讨论

Web 服务器响应浏览器请求是 Node 的“Hello World”应用程序。 它不仅演示了 Node 应用程序的功能，还演示了如何使用一种相当传统的通信方法与其通信：请求 Web 资源。

从顶部开始，解决方案的第一行使用 Node 的 `require()` 函数加载 `http` 模块。 这指示 Node 的模块化系统加载用于应用程序的特定库资源。 `http` 模块是默认情况下随 Node 安装的众多模块之一。

接下来，使用 `http.createServer()` 创建一个 HTTP 服务器，传入一个匿名函数，即 `RequestListener`，带有两个参数。 Node 将此函数附加为每个服务器请求的事件处理程序。 这两个参数是 *request* 和 *response*。 请求是 `http.IncomingMessage` 对象的一个实例，响应是 `http.ServerResponse` 对象的一个实例。

`http.ServerResponse` 用于响应 Web 请求。 `http.IncomingMessage` 对象包含有关请求的信息，例如请求 URL。如果您需要从 URL 获取特定信息（例如查询字符串参数），则可以使用 Node 的 `url` 实用程序模块解析字符串。 示例 17-1 演示了如何使用查询字符串来向浏览器返回更自定义的消息。

##### 示例 17-1\. 解析查询字符串数据

```
// load http module
const http = require('http');
const url = require('url');

// create http server
http
  .createServer((req, res) => {
    // get query string and parameters
    const { query } = url.parse(req.url, true);

    // content header
    res.writeHead(200, { 'content-type': 'text/plain' });

    // write message and signal communication is complete
    const name = query.first ? query.first : 'World';

    // write message and signal communication is complete
    res.end(`Hello, ${name}!`);
  })
  .listen(8124);

console.log('Server running on port 8124');
```

如下所示的 URL：

```
http://localhost:8124/?first=Reader
```

导致一个显示“Hello, Reader!”的网页。

在代码中，`url` 模块对象具有一个 `parse()` 方法，该方法解析 URL 并返回其各个组件（`href`、`protocol`、`host` 等）。 如果将 `true` 作为第二个参数传递，字符串还会由另一个模块 `querystring` 解析，后者将查询字符串作为对象返回，每个参数作为对象属性，而不仅仅返回字符串。

在解决方案和 示例 17-1 中，都返回一个文本消息作为页面输出，使用 `http.ServerResponse` 的 `end()` 方法。我也可以使用 `write()` 输出消息，然后调用 `end()`：

```
res.write(`Hello, ${name}!`);
res.end();
```

无论采用哪种方法，重要的是在设置所有标题和响应主体后，*必须*调用响应的 `end()` 方法。

在 `createServer()` 函数调用的末尾链接了另一个函数调用，这次是 `listen()`，传入服务器监听的端口号。这个端口号也是应用程序的一个特别重要的组成部分。

传统上，端口 80 是大多数 Web 服务器的默认端口（不使用 HTTPS 的服务器，默认端口是 443）。通过使用端口 80，在请求服务的 URL 时无需指定端口。但端口 80 也是我们更传统的 Web 服务器 Apache 的默认端口。如果尝试在 Apache 使用的端口上运行 Node 服务，应用程序将失败。Node 应用程序要么必须独立运行在服务器上，要么在不同的端口上运行。

你也可以指定 IP 地址（主机），除了端口。这样做可以确保人们向特定的主机和端口发出请求。如果不提供主机，则应用程序将监听与服务器关联的任何 IP 地址的请求。你也可以指定域名，Node 将解析主机。

还有其他方法演示的参数，以及大量其他方法，但这将让你入门。有关更多信息，请参阅 [Node 文档](http://nodejs.org/api)。

# 使用 REPL 交互式尝试 Node 代码片段

## 问题

你希望轻松运行基于服务器的 Node 代码片段。

## 解决方案

使用 Node 的 REPL（读取-评估-打印-循环），这是 Node 的交互式命令行版本，可以运行任何代码片段。

要使用 REPL，在命令行中键入 `node` 而不指定要运行的应用程序：

```
$ node
```

然后，你可以以简化的 Emacs 风格进行 JavaScript。你可以导入库，创建函数——可以在静态应用程序中做的任何事情。主要区别是每行代码都会立即解释：

```
> const add = (x, y) => { return x + y };
undefined
> add(2, 2);
4
```

完成后，使用 `.exit` 退出程序：

```
> .exit
```

## 讨论

REPL 可以独立启动，或者在另一个应用程序中启动，如果你想设置某些功能。你输入 JavaScript 就像在文本文件中输入脚本一样。主要的行为差异是在每输入一行后可能会看到一个结果，比如运行时 REPL 中显示的 `undefined`。

但你可以导入模块：

```
> const fs = require('fs');
```

你也可以访问全局对象，在我们使用 `require()` 时刚刚这样做。

在键入某些代码后显示的 `undefined` 是前一行代码执行的返回值。设置新变量和创建函数是一些返回 `undefined` 的 JavaScript 操作，这可能会很快让人厌烦。为了消除这种行为以及进行其他一些修改，可以在一个小的 Node 应用程序中使用 `REPL.start()` 函数触发 REPL（但使用您指定的选项）。

可用的选项包括：

`prompt`

更改显示的提示（默认为 *>*）

`input`

更改输入可读流（默认为 `process.stdin`，标准输入）

`output`

更改输出可写流（默认为 `process.stdout`，标准输出）

`terminal`

如果流应该像 TTY 一样对待并写入 ANSI/VT100 转义代码，则设置为 `true`

`eval`

用于替换异步 `eval()` 函数的函数，用于评估 JavaScript

`useColors`

设置为 `true` 为 `writer` 函数设置输出颜色（默认基于终端的默认值）

`useGlobal`

设置为 `true` 可以使用 `global` 对象，而不是在单独的上下文中运行脚本

`ignoreUndefined`

设置为 `true` 以消除 `undefined` 返回值

`writer`

从评估的代码中返回格式化结果以显示（默认为 `util.inspect` 函数）

以下是一个示例应用程序，它使用新的提示启动 REPL，忽略未定义的值，并使用颜色：

```
const repl = require('repl');

const options = {
  prompt: '-> ',
  useColors: true,
  ignoreUndefined: true
};

repl.start(options);
```

我们想要的选项在 `options` 对象中定义，然后作为参数传递给 `repl.start()`。当我们运行应用程序时，REPL 被启动，但我们不再需要处理未定义的值：

```
-> const add = (x, y) => { return x + y };
-> add(2, 2);
4
```

正如您所看到的，这是一个更清晰的输出，没有所有那些混乱的 `undefined` 打印输出。

## 额外信息：等等，什么是全局对象？

你发现了吗？

JavaScript 在 Node 和浏览器中的一个区别是全局作用域。传统上，在浏览器中，当你在函数外部使用 `var` 创建变量时，它属于顶层全局对象，我们称之为 `window`：

```
var test = 'this is a test';
console.log(window.test); // 'this is a test'
```

类似地，在浏览器中使用 `let` 或 `const` 时，变量是全局作用域的，但不附加到 `window` 对象。

在 Node 中，每个模块都在自己的独立上下文中运行，因此模块可以声明相同的变量，如果它们都在同一个应用程序中使用，则不会发生冲突。

然而，有些对象可以从 Node 的 `global` 对象访问。在前面的例子中，我们使用了一些，包括 `console`、Buffer 对象和 `require()`。其他包括一些非常熟悉的老朋友：`setTimeout()`、`clearTimeout()`、`setInterval()` 和 `clearInterval()`。

# 读取和写入文件数据

## 问题

您想要从本地存储的文件中读取或写入。

## 解决方案

Node 的文件系统管理功能作为 Node 核心的一部分包含在 `fs` 模块中：

```
const fs = require('fs');
```

要读取文件的内容，请使用 `readFile()` 函数：

```
const fs = require('fs');

fs.readFile('main.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

要写入文件，请使用`writeFile()`：

```
const fs = require('fs');

const buf = "I'm going to write this text to a file";
fs.writeFile('main2.txt', buf, err => {
  if (err) throw err;
  console.log('wrote text to file');
});
```

`writeFile()`函数会覆盖现有文件。要向文件追加文本，请使用`appendText()`：

```
const fs = require('fs');

const buf = "\nI'm going to add this text to a file";
fs.appendFile('main.txt', buf, err => {
  if (err) throw err;
  console.log('appended text to file');
});
```

## 讨论

Node 的文件系统支持既全面又简单易用。要从文件中读取，请使用`readFile()`函数，支持以下参数：

+   文件名，包括操作系统路径（如果不是应用程序本地文件）

+   一个选项对象，包括`encoding`选项（如解决方案中所示）和`flag`选项，默认设置为`r`（读取）

+   回调函数，带有错误和读取的数据两个参数

在解决方案中，如果在应用程序中未指定编码，Node 将以原始缓冲区形式返回文件内容。由于我指定了编码，文件内容以字符串形式返回。

`writeFile()`和`appendFile()`函数用于分别写入和追加数据，其参数类似于`readFile()`：

+   文件名和路径

+   要写入文件的字符串或缓冲区数据

+   选项对象，包括`encoding`选项（`writeFile()`默认为`w`，`appendFile()`默认为`a`）和`mode`选项，默认为`438`（八进制中的`0666`）

+   回调函数，只有一个参数：错误信息

如果文件是通过写入或追加创建的，则可以使用`mode`的选项值来设置文件的权限。默认情况下，文件由所有者可读写，并由组和全局用户可读。

我提到，要写入的数据可以是缓冲区或字符串。字符串不能处理二进制数据，因此 Node 提供了缓冲区，可以处理字符串或二进制数据。这两者都可以在本节讨论的所有文件系统函数中使用，但如果要同时使用它们，就需要明确地在两种类型之间进行转换。

例如，在使用`writeFile()`时，不提供*utf8*的`encoding`选项，而是将字符串转换为缓冲区，并在这样做时提供所需的编码：

```
const fs = require('fs');

const str = "I'm going to write this text to a file";
const buf = Buffer.from(str, 'utf8');
fs.writeFile('mainbuf.txt', buf, err => {
  if (err) throw err;
  console.log('wrote text to file');
});
```

反之，将缓冲区转换为字符串同样简单：

```
const fs = require('fs');

fs.readFile('main.txt', (err, data) => {
  if (err) throw err;
  const str = data.toString();
  console.log(str);
});
```

缓冲区的`toString()`函数有三个可选参数：编码、开始转换的位置和结束转换的位置。默认情况下，整个缓冲区使用*utf8*编码进行转换。

`readFile()`、`writeFile()`和`appendFile()`函数是*异步*的，这意味着它们在继续执行代码之前不会等待操作完成。在处理诸如文件访问等速度慢的操作时，这是至关重要的。每个函数还有同步版本：`readFileSync()`、`writeFileSync()`和`appendFileSync()`。我强调不应使用这些同步版本。我只是全面起见才提到它们。

## 高级

从文件中读取或写入的另一种方法是结合使用 `open()` 函数和 `read()` 用于读取文件内容，或 `write()` 用于写入文件。这种方法的优点是在过程中有更精细的控制。缺点是与所有函数相关的额外复杂性，包括只能使用缓冲区进行文件的读取和写入。

`open()` 的参数包括：

+   文件名和路径

+   标志

+   可选模式

+   回调函数

所有操作都使用相同的 `open()`，标志控制发生的情况。有很多标志选项，但这时我们最感兴趣的是：

`r`

打开文件以供读取；文件必须存在

`r`+

打开文件以供读取和写入；如果文件不存在则引发异常

`w`

打开文件以供写入，如果文件存在则截断文件，否则创建文件

`wx`

打开文件以供写入，但如果文件 *存在* 则失败

`w`+

打开文件以供读取和写入；如果文件不存在则创建；如果文件存在则截断文件

`wx+`

类似于 `w+`，但如果文件存在则失败

`a`

打开文件以供追加，如果文件不存在则创建文件

`ax`

打开文件以供追加，如果文件存在则失败

`a`+

打开文件以供读取和追加；如果文件不存在则创建文件

`ax+`

类似于 `a+`，但如果文件存在则失败

模式与前述相同，设置文件的 *粘性* 和 *权限* 位，如果创建，则默认为 `0666`。回调函数有两个参数：如果发生错误，则为错误对象，否则为 *文件描述符*，用于后续文件操作。

`read()` 和 `write()` 函数共享相同类型的基本参数：

+   `open()` 方法回调文件描述符

+   用于保存待写入或追加的数据或进行读取的缓冲区

+   输入/输出（I/O）操作开始的偏移量

+   缓冲区长度（由读取操作设置，控制写入操作）

+   操作将进行的文件位置；如果位置是当前位置则为 *null*

这两种方法的回调函数都有三个参数：一个错误、读取（或写入）的字节数和缓冲区。

这是很多参数和选项。展示如何运作的最佳方式是创建一个完整的 Node 应用程序，用于打开一个全新的文件进行写入，写入一些文本，再写入一些文本，然后读取所有文本并打印到 `console` 中。由于 `open()` 是异步的，读取和写入操作必须在回调函数内进行。在 示例 17-2 中准备好，因为你将首次体验到被称为 *回调地狱* 的概念。

##### 示例 17-2\. 展示打开、读取和写入操作

```
const fs = require('fs');

fs.open('newfile.txt', 'a+', (err, fd) => {
  if (err) {
    throw err;
  } else {
    const buf = Buffer.from('The first string\n');
    fs.write(fd, buf, 0, buf.length, 0, (err, written) => {
      if (err) {
        throw err;
      } else {
        const buf2 = Buffer.from('The second string\n');
        fs.write(fd, buf2, 0, buf2.length, buf.length, (err, written2) => {
          if (err) {
            throw err;
          } else {
            const length = written + written2;
            const buf3 = Buffer.alloc(length);
            fs.read(fd, buf3, 0, length, 0, err => {
              if (err) {
                throw err;
              } else {
                console.log(buf3.toString());
              }
            });
          }
        });
      }
    });
  }
});
```

###### 注意

驯服回调函数在 “管理回调地狱” 中有介绍。

要找出缓冲区的长度，我使用了`length`，它返回缓冲区的字节数。这个值不一定与缓冲区中字符串的长度匹配，但在这种用法中起作用。

那么多级缩进可能会让您毛骨悚然，但该示例演示了`open()`、`read()`和`write()`的工作方式。这些函数的组合是在`readFile()`、`writeFile()`和`appendFile()`函数中用于管理文件访问的。高级函数只是简化了最常见的文件操作。

###### 注意

查看“管理回调地狱”以解决所有这些恶心的缩进问题。

# 从终端获取输入

## 问题

您希望通过终端从应用程序用户获取输入。

## 解决方案

使用 Node 的 Readline 模块。

要从标准输入获取数据，请使用以下代码：

```
const readline = require('readline');

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

rl.question(">>What's your name?  ", answer => {
  console.log(`Hello ${answer}`);
  rl.close();
});
```

## 讨论

Readline 模块提供了从可读流获取文本行的能力。您首先通过`createInterface()`创建 Readline 接口的实例，至少传入可读和可写流。您两者都需要，因为您既要写入提示，又要读取文本。在解决方案中，输入流是`process.stdin`，标准输入流，而输出流是`process.stdout`。换句话说，输入和输出来自命令行。

解决方案使用`question()`函数发布问题，并提供回调函数来处理响应。在函数内部，调用了`close()`，它关闭接口，释放输入和输出流的控制权。

您还可以创建一个应用程序，继续监听输入，对传入的数据采取一些操作，直到某些信号结束应用程序。通常，这个信号是一系列信号，表示个人已经完成，比如*exit*这个词。这种类型的应用程序还使用其他 Readline 函数，如`setPrompt()`用于根据每行文本为个人设置提示；`prompt()`准备输入区域，包括更改为`setPrompt()`设置的提示；以及`write()`，用于写出提示。此外，您还需要使用事件处理程序来处理事件，例如`line`，它监听每一行新的文本。

示例 17-3 包含一个完整的 Node 应用程序，该应用程序继续从用户那里处理输入，直到他们输入`exit`。请注意，该应用程序利用了`process.exit()`。这个函数干净地终止了 Node 应用程序。

##### 示例 17-3. 从 stdin 访问数字，直到用户键入*exit*

```
const readline = require('readline');

let sum = 0;

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

console.log("Enter numbers, one to a line. Enter 'exit' to quit.");

rl.setPrompt('>> ');
rl.prompt();

rl.on('line', input => {
  const userInput = input.trim();
  if (userInput === 'exit') {
    rl.close();
    return;
  }
  sum += Number(userInput);
  rl.prompt();
});

// user typed in 'exit'
rl.on('close', () => {
  console.log(`Total is ${sum}`);
  process.exit(0);
});
```

运行应用程序时，使用多个数字会产生以下输出：

```
Enter numbers, one to a line. Enter 'exit' to quit.
>> 55
>> 209
>> 23.44
>> 0
>> 1
>> 6
>> exit
Total is 294.44
```

我使用了`console.log()`而不是 Readline 接口的`write()`来写入提示，后面跟一个新行，以区分输出和输入。

## 参见

第十九章涵盖了在 Node 应用程序中传递和读取命令行参数。

# 获取当前脚本的路径

## 问题

你的应用程序需要读取正在执行的脚本的路径。

## 解决方案

使用`__dirname`或`__filename`变量，它们在执行模块的作用域中：

```
// logs the directory of the currently executed file
// ex: /Users/Adam/Projects/js-cookbook/node
console.log(__dirname);

// logs the directory and filename of the currently executed file
// ex: /Users/Adam/Projects/js-cookbook/node/example.js
console.log(__filename);
```

## 讨论

`__dirname`或`__filename`变量看起来在全局范围内，但它们实际上存在于模块本身的作用域中。假设你有以下目录结构的项目：

```
example-app
|   index.js
├───dir1
|   |   example.js
|   └───dir3
|       |   nested.js
```

如果你在 index.js 文件中读取`__dirname`，它将是项目根目录的路径。然而，如果在 *nested.js* 文件中从脚本中读取`__dirname`，它将读取到 *dir3* 目录的路径。这使得你可以在模块执行时读取模块的路径，而不仅仅是限于父目录本身。

`__dirname`在创建当前目录内的新文件或目录时的一个有用的示例。在下面的示例中，脚本在当前文件的目录中创建一个名为*cache*的新子目录：

```
const fs = require('fs');
const path = require('path');
const newDirectoryPath = path.join(__dirname, '/cache');

fs.mkdirSync(newDirectoryPath);
```

# 使用 Node 定时器和理解 Node 事件循环

## 问题

你需要在 Node 应用程序中使用定时器，但是你不确定应该使用 Node 的哪个定时器，或者它们有多精确。

## 解决方案

如果你的定时器不需要很精确，你可以使用`setTimeout()`来创建单个定时器事件，或者如果你需要一个重复的定时器，可以使用`setInterval()`：

```
setTimeout(() => {}, 3000);

setInterval(() => {}, 3000);
```

这两个定时器函数都可以被取消：

```
const timer1 = setTimeout(() => {}, 3000);
clearTimeout(timer1);

const timer2 = setInterval(() => {}, 3000);
clearInterval(timer2);
```

然而，如果你需要更精细地控制你的定时器，并且需要立即得到结果，你可能想要使用`setImmediate()`。你不需要为它指定延迟，因为你希望回调在所有 I/O 回调处理完毕之后但是在任何`setTimeout()`或`setInterval()`回调之前*立即*被调用：

```
setImmediate(() => {});
```

它也可以通过`clearImmediate()`清除。

## 讨论

Node，作为基于 JavaScript 的运行在单线程上。它是*同步*的。然而，输入/输出（I/O）和其他本机 API 访问是*异步*的或在单独的线程上运行。Node 处理这种时间上的不连续性的方法是*事件循环*。

在你的代码中，当你执行 I/O 操作时，比如向文件写入文本块，你会指定一个回调函数来处理写入后的活动。一旦你这样做了，剩下的应用程序代码就会被处理。它不会等待文件写入完成。当文件写入完成时，会返回一个事件来通知 Node，并被推送到一个队列中等待处理。Node 处理这个事件队列，当它处理到完成文件写入的事件时，它将该事件与回调匹配，并处理该回调。

作为比较，想象一下走进一家快餐店并点午餐。你排队等候下单，并被分配一个订单号。你坐下来看报纸，或者查看 Twitter 账户等待。与此同时，午餐订单进入另一个队列，供快餐店员工处理订单。但并不是每个午餐请求都会按接收顺序完成。有些午餐订单可能需要更长时间。它们可能需要更长时间烘烤或烤制。因此，快餐店员工通过准备您的午餐项目然后将其放入烤箱，并设置一个定时器以便完成后通知您，然后继续其他任务。

当定时器触发时，快餐店员工迅速完成当前任务，并从烤箱中取出您的午餐订单。然后通过呼叫您的订单号来通知您可以取餐。如果同时处理多个耗时的午餐项目，则快餐店员工会按顺序处理每个项目的定时器触发。

所有的 Node 进程都符合快餐店订单队列的模式：先进先出发送到快餐店（线程）工人。但是，某些操作（如 I/O 操作）就像那些需要额外时间在烤箱或烤架中烘烤的午餐订单，但不需要快餐店员工停止任何其他工作等待烘烤和烤制。烤箱或烤架定时器相当于在 Node 事件循环中出现的消息，触发基于请求操作的最终动作。

现在，你拥有了同步和异步进程的工作混合体。但是定时器会发生什么呢？

`setTimeout()` 和 `setInterval()` 都在给定的延迟后触发，但实际上是将消息添加到事件循环中，按顺序处理。因此，如果事件循环特别拥挤，定时器函数的回调会有延迟：

> 需要注意的是，你的回调函数可能不会在准确的（延迟）毫秒内调用。Node.js 不保证回调函数触发的确切时间，也不保证触发顺序。回调函数会尽可能接近指定的时间调用。
> 
> Node 定时器文档

在大多数情况下，发生的延迟超出了我们的感知能力，但可能导致看起来不流畅的动画。它也可能给其他应用程序添加一个奇怪的效果。

在 示例 17-4 中，我创建了一个 SVG 滚动时间轴，通过 WebSockets 将数据提供给客户端。为了模拟真实世界的数据，我使用了一个三秒的定时器，并随机生成一个数作为数据值。在服务器端的代码中，我使用了 `setInterval()`，因为定时器是循环的：

##### 示例 17-4\. 滚动时间轴示例

```
const app = require('http');
const fs = require('fs');
const ws = require('nodejs-websocket');

let server;

// serve static page
const handler = (req, res) => {
  fs.readFile(`${__dirname}/drawline.html`, (err, data) => {
    if (err) {
      res.writeHead(500);
      return res.end('Error loading drawline.html');
    }
    res.writeHead(200);
    res.end(data);
    return data;
  });
};

/// start the webserver
// connections on Port 8124 will be handled by the handler
app.listen(8124);
app.createServer(handler);

// data timer
const startTimer = () => {
  setInterval(() => {
    const newval = Math.floor(Math.random() * 100) + 1;
    if (server.connections.length > 0) {
      console.log(`sending ${newval}`);
      const counter = { counter: newval };
      server.connections.forEach(conn => {
        conn.sendText(JSON.stringify(counter), () => {
          console.log('conn sent');
        });
      });
    }
  }, 3000);
};

// Create a websocket connection handler on a different port
server = ws
  .createServer(conn => {
    console.log('connected');
    conn.on('close', () => {
      console.log('Connection closed');
    });
  })
  .listen(8001, () => {
    startTimer();
  });
```

我在代码中包含了 `console.log()` 调用，这样你就可以看到计时器事件与通信响应的比较。当调用 `setInterval()` 函数时，它被推送到进程中。当处理其回调时，WebSocket 通信也被推送到队列中。

解决方案使用了 `setInterval()`，这是 Node 的三种不同类型计时器之一。`setInterval()` 函数的格式与我们在浏览器中使用的格式相同。你为第一个函数指定一个回调，提供延迟时间（以毫秒为单位），以及任何潜在的参数。计时器将在三秒后触发，但我们已经知道计时器的回调可能不会立即被处理。

与 WebSocket `sendText()` 调用中传递的回调函数一样。这些基于 Node 的 Net（或者如果安全的话，是 TLS）套接字，正如 `socket.write()`（用于 `sendText()` 的内容）文档所述：

> 可选的回调参数将在数据最终写出时执行 — 这可能不会立即发生。
> 
> Node 文档

如果将计时器设置为立即调用（将延迟值设为零），你会看到发送的数据消息与通信发送消息交错（在浏览器客户端由于套接字通信而冻结之前，你不希望在应用程序中再次使用零值）。

然而，所有客户端的时间表保持不变，因为通信是在计时器的回调函数中*同步*发送的，所以所有通信的数据都是相同的 — 只是回调处理似乎是无序的。

我之前提到过使用延迟为零的 `setInterval()`。实际上，它并不完全是零 — Node 遵循浏览器遵循的 HTML5 规范，并将计时器间隔“夹紧”到最小值四毫秒。虽然这看似过小以至于不会引起问题，但对于动画和时间关键的进程而言，时间延迟可能会影响整体的外观和/或功能。

为了绕过这些约束，Node 开发者使用 Node 的 `process.nextTick()`。与 `process.nextTick()` 关联的回调会在下一个事件循环回合中处理，通常在任何 I/O 回调之前（虽然有一些限制，我马上会讲到）。不再有讨厌的四毫秒节流了。但是，如果有大量递归调用 `process.nextTick()`，会发生什么呢？

回到我们的熟食店比喻，在忙碌的午餐时间，工作人员可能被订单压倒，全神贯注于处理新订单，以至于无法及时响应烤箱和烧烤的提示。这时候事物就会烧焦。如果你去过一个运营良好的熟食店，你会注意到接单的服务员在接受订单之前会评估厨房情况，稍微拖延一下，甚至承担一些厨房职责，让顾客在订单队列中稍微等待久一点。

Node 也是如此。如果 `process.nextTick()` 被允许一直处于受宠的地位，I/O 操作将会被饿死。Node 使用另一个值 `process.maxTickDepth`，默认值为 1000，来限制在允许 I/O 回调之前处理的 `process.next()` 回调数量。这就像是熟食店中的服务员。

在 Node 的更新版本中，添加了 `setImmediate()` 函数。此函数试图解决与定时操作相关的所有问题，并创建一个适合大多数人的良好平衡点。调用 `setImmediate()` 时，其回调将在 I/O 回调之后但在 `setTimeout()` 和 `setInterval()` 回调之前添加。我们不需要传统定时器的四毫秒税，但也不需要 `process.nextTick()` 这个顽皮的家伙。

再次回到熟食店的比喻中，`setImmediate()` 就像是订单队列中的一个顾客，看到熟食店工作人员忙于处理烤箱和烧烤的提示，礼貌地表示他们愿意等待以便让出订单。

###### 注意

然而，在滚动时间轴示例中，*千万不要*使用 `setImmediate()`，因为它会比你眨眼的速度更快地让你的浏览器冻结起来。
