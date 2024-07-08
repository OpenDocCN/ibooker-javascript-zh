# 第二章：使用 Node 入门

如果你对 Node 没有任何经验，本章适合你。理解 Express 及其用途需要对 Node 有基本的了解。如果你已经有使用 Node 构建 web 应用程序的经验，可以跳过本章。在本章中，我们将使用 Node 构建一个非常简单的 web 服务器；在下一章中，我们将看到如何使用 Express 完成同样的事情。

# 获取 Node

在你的系统上安装 Node 无比简单。Node 团队确保安装过程在所有主要平台上都简单明了。

前往[Node 主页](http://nodejs.org)。点击大绿色按钮，上面有一个版本号，后面跟着“LTS（推荐大多数用户使用）”。LTS 代表*长期支持*，比当前版本更加稳定，包含了更多的功能和性能改进。

对于 Windows 和 macOS，将下载一个安装程序来引导你完成安装过程。对于 Linux 用户，如果你[使用软件包管理器](http://bit.ly/36UYMxI)，可能会更快上手。

###### 注意

如果你是 Linux 用户，并且想使用软件包管理器，请确保按照前述网页上的说明操作。如果你不添加适当的软件包存储库，许多 Linux 发行版会安装一个非常旧的 Node 版本。

你还可以下载[一个独立安装程序](https://nodejs.org/en/download)，如果你需要将 Node 分发给你的组织，这将非常有帮助。

# 使用终端

我是一个无悔的终端（也称为*控制台*或*命令提示符*）的力量和生产力的粉丝。本书中的所有示例都假定你正在使用终端。如果你不熟悉终端，我强烈建议你花一些时间熟悉你选择的终端。本书中的许多实用程序都有对应的图形界面，所以如果你坚决不使用终端，你也有选择，但你将不得不找到适合自己的方法。

如果你使用 macOS 或 Linux，你有多种老牌 shell（终端命令解释器）可供选择。目前最流行的是 bash，不过 zsh 也有它的支持者。我偏向使用 bash 的主要原因（除了长期熟悉）是它的普及性。无论坐在哪台基于 Unix 的计算机前，99% 的情况下，默认 shell 都会是 bash。

如果你是 Windows 用户，情况就不那么美好了。微软从来都不太关心提供愉快的终端体验，所以你需要做更多的工作。Git 贴心地包含了一个“Git bash” shell，提供类 Unix 的终端体验（虽然只有一小部分通常可用的 Unix 命令行工具，但已经足够实用）。虽然 Git bash 提供了一个简化的 bash shell，但它仍然使用内置的 Windows 控制台应用程序，这会让人感到沮丧（即使是简单的功能如调整控制台窗口大小、选择文本、剪切和粘贴也是不直观且笨拙的）。因此，我建议安装更复杂的终端，比如[ConsoleZ](https://github.com/cbucher/console)或[ConEmu](https://conemu.github.io)。对于 Windows 高级用户，尤其是.NET 开发者或者专业的 Windows 系统或网络管理员，还有另一种选择：微软自家的 PowerShell。PowerShell 名副其实：有人用它做出了惊人的事情，一个熟练的 PowerShell 用户可以媲美 Unix 命令行专家。然而，如果你在 macOS/Linux 和 Windows 之间切换，我仍建议坚持使用 Git bash 以保持一致性。

如果你使用的是 Windows 10 或更新版本，现在可以直接在 Windows 上安装 Ubuntu Linux 了！这不是双系统或虚拟化，而是微软开源团队的杰出工作，将 Linux 体验带到了 Windows。你可以通过[Microsoft 应用商店](http://bit.ly/2KcSfEI)安装 Ubuntu 在 Windows 上。

Windows 用户的最后一个选择是虚拟化。在现代计算机的强大架构下，虚拟机（VM）的性能几乎与实际机器无异。我使用 Oracle 免费的[VirtualBox](https://www.virtualbox.org/)效果非常好。

最后，无论你使用什么系统，都有出色的基于云的开发环境，比如[Cloud9](https://aws.amazon.com/cloud9/)（现在是 AWS 的产品）。Cloud9 会快速搭建一个新的 Node 开发环境，让你可以快速开始 Node 的开发。

一旦你选择了一个让你满意的 shell，我建议你花一些时间了解基础知识。互联网上有许多优秀的教程（[Bash 指南](https://guide.bash.academy)是一个很好的起点），通过现在学习一点点，你可以避免以后的很多麻烦。至少，你应该知道如何浏览目录；复制、移动和删除文件；以及如何退出命令行程序（通常是 Ctrl-C）。如果你想成为一个终端忍者，我鼓励你学习如何在文件中搜索文本，搜索文件和目录，将命令链接在一起（传统的“Unix 哲学”），以及重定向输出。

###### 注意

在许多类 Unix 系统上，Ctrl-S 有着特殊的含义：它会“冻结”终端（曾经用于快速暂停输出）。由于这是“保存”的常见快捷键，人们很容易在不经意间按下它，这会导致大多数人陷入困惑的情况（这种情况对我来说比我愿意承认的更频繁发生）。要解除终端的冻结状态，只需按下 Ctrl-Q。因此，如果你曾经对终端突然冻结感到困惑，请尝试按下 Ctrl-Q 看看是否解决了问题。

# 编辑器

对于程序员而言，很少有什么话题像选择编辑器那样引发激烈的辩论，而且理由充分：编辑器是你的主要工具。我选择的编辑器是 vi（或具有 vi 模式的编辑器）。^(1) vi 并不适合所有人（当我告诉同事他们可以像我一样轻松做他们正在做的事情时，他们经常瞪大眼睛），但是找到一个强大的编辑器并学会使用它将显著提高你的生产力，也可以说，增加你的愉悦感。我特别喜欢 vi 的一个原因（虽然并非最重要的原因之一）是，像 bash 一样，它是无处不在的。如果你有 Unix 系统的访问权限，vi 就在那里等着你。大多数流行的编辑器都有“vi 模式”，允许你使用 vi 的键盘命令。一旦你习惯了它，很难想象使用其他任何编辑器。vi 起初是一条艰难的道路，但回报是值得的。

如果像我一样，你认为熟悉任何地方都可以用的编辑器很有价值，你的另一个选择是 Emacs。我和 Emacs 从来没有完全融洽过（通常你要么是 Emacs 用户，要么是 vi 用户），但我绝对尊重 Emacs 提供的强大和灵活性。如果 vi 的模态编辑方法不适合你，我建议你尝试一下 Emacs。

尽管掌握控制台编辑器（如 vi 或 Emacs）可能非常方便，你可能仍然希望使用一个更现代的编辑器。一个流行的选择是 [Visual Studio Code](https://code.visualstudio.com/)（不要与不带“Code”的 Visual Studio 混淆）。我可以全心推荐 Visual Studio Code；它是一个设计精良、快速高效的编辑器，非常适合 Node 和 JavaScript 的开发。另一个流行的选择是 [Atom](https://atom.io)，它在 JavaScript 社区中也很受欢迎。这两款编辑器都可以在 Windows、macOS 和 Linux 上免费使用（两者都有 vi 模式！）。

现在我们有了一个编辑代码的好工具，让我们把注意力转向 npm，它将帮助我们获取其他人编写的包，以便利用庞大而活跃的 JavaScript 社区。

# npm

npm 是 Node 包的普遍包管理器（也是我们将获取并安装 Express 的方式）。在 PHP、GNU、WINE 等的讽刺传统中，*npm* 不是一个首字母缩略词（这也是为什么它没有大写字母）；相反，它是“npm is not an acronym”的递归缩写。

广义上说，包管理器的两个主要责任是安装包和管理依赖关系。npm 是一个快速、能力强大且无痛的包管理器，我认为它在很大程度上促进了 Node 生态系统的快速增长和多样性。

###### 注意

有一个名为 Yarn 的流行竞争包管理器，它使用与 npm 相同的包数据库；我们将在第十六章中使用 Yarn。

当您安装 Node 时，npm 也会随之安装，因此如果您按照前面列出的步骤操作，您已经拥有它了。那么让我们开始工作吧！

您将在 npm 中主要使用的命令（不足为奇地）是`install`。例如，要安装 nodemon（一种流行的实用程序，在您更改源代码时自动重新启动 Node 程序），您需要发出以下命令（在控制台上）：

```
npm install -g nodemon
```

`-g`标志告诉 npm 在系统上全局安装包，这意味着它在全局范围内可用。当我们讨论*package.json*文件时，这种区别会变得更加清晰。目前的经验法则是，JavaScript 实用程序（如 nodemon）通常会全局安装，而专用于您的 Web 应用程序或项目的包则不会。

###### 注意

与像 Python 这样的语言不同，它在从 2.0 到 3.0 进行了重大语言更改，需要一种能够轻松切换不同环境的方式——Node 平台还很新，因此您可能总是应该运行最新版本的 Node。但是，如果您确实需要支持多个 Node 版本，请查看[nvm](https://github.com/creationix/nvm)或[n](https://github.com/tj/n)，这些工具允许您切换环境。您可以通过输入`node --version`来查看计算机上安装的 Node 版本。

# 使用 Node 创建一个简单的 Web 服务器

如果您以前构建过静态 HTML 网站或者有 PHP 或 ASP 背景，您可能已经习惯于 Web 服务器（例如 Apache 或 IIS）提供您的静态文件，以便浏览器可以通过网络查看它们。例如，如果您创建了文件*about.html*并将其放在正确的目录中，然后您可以导航至*http://localhost/about.html*。根据您的 Web 服务器配置，甚至可以省略*.html*，但 URL 和文件名之间的关系是明确的：Web 服务器只需知道文件在计算机上的位置并将其提供给浏览器。

###### 注意

*localhost*，顾名思义，指的是您所在的计算机。这是 IPv4 回环地址 127.0.0.1 或 IPv6 回环地址::1 的常见别名。您经常会看到使用 127.0.0.1，但本书中我将使用*localhost*。如果您正在使用远程计算机（例如使用 SSH），请记住，浏览到*localhost*不会连接到该计算机。

Node 提供了一个与传统 Web 服务器不同的范式：你编写的应用程序*就是*Web 服务器。Node 只是为你构建 Web 服务器提供了框架。

“但我不想写一个 Web 服务器”，你可能会说！这是一种自然的反应：你想写一个应用程序，而不是一个 Web 服务器。然而，Node 使编写这个 Web 服务器的业务变得非常简单（甚至只需几行代码），而你在应用程序中获得的控制力远远超过了这一点。

所以让我们开始吧。你已经安装了 Node，与终端交朋友，现在你已经准备好了。

## Hello World

我一直觉得很不幸的是，经典的入门编程示例是毫无灵感的消息“Hello world”。然而，到了这一点，违背这样一个沉重的传统似乎是不可取的，所以我们将从这里开始，然后转向更有趣的东西。

在你喜爱的编辑器中创建一个名为*helloworld.js*（在伴随存储库中为*ch02/00-helloworld.js*）的文件：

```
const http = require('http')
const port = process.env.PORT || 3000

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' })
  res.end('Hello world!')
})

server.listen(port, () => console.log(`server started on port ${port}; ` +
  'press Ctrl-C to terminate....'))
```

###### 注意

根据你学习 JavaScript 的时间和地点，这个例子中缺少分号可能会让你感到困惑。我曾是一个坚定的分号支持者，随着我进行更多 React 开发，我不情愿地停止了使用它们。过了一段时间，我的眼前的迷雾消散了，我不禁想知道我曾经为什么对分号如此着迷！现在我坚定地支持“不加分号”团队，本书中的示例也将反映这一点。这是个人选择，如果你愿意，可以继续使用分号。

确保你在与*helloworld.js*相同的目录中，并输入`node helloworld.js`。然后打开浏览器，导航到*http://localhost:3000*，*voilà*！你的第一个 Web 服务器就完成了。这个特定的服务器不提供 HTML，而是仅向你的浏览器显示纯文本消息`'Hello world!'`。如果你愿意，你可以尝试发送 HTML：只需将`text/plain`更改为`text/html`，并将`'Hello world!'`更改为包含有效 HTML 的字符串。我没有演示这一点，因为我尽量避免在 JavaScript 中编写 HTML，原因将在第七章中详细讨论。

## 事件驱动编程

Node 的核心理念是*事件驱动编程*。对于你作为程序员来说，这意味着你必须了解可用的事件以及如何对其做出响应。许多人通过实现用户界面来介绍事件驱动编程：用户点击某些东西，你处理*点击事件*。这是一个很好的隐喻，因为人们理解程序员无法控制用户何时或是否会点击某些东西，所以事件驱动编程实际上非常直观。在服务器上做出事件响应的概念跨越可能会更难一些，但原则是相同的。

在前面的代码示例中，事件是隐含的：正在处理的事件是 HTTP 请求。`http.createServer`方法接受一个函数作为参数；每次发出 HTTP 请求时都会调用该函数。我们的简单程序只是将内容类型设置为纯文本，并发送字符串“Hello world!”

一旦您开始以事件驱动的编程方式思考，您就会在各处看到事件。其中一个事件是当用户从应用程序的一个页面或区域导航到另一个页面时。您的应用程序如何响应该导航事件被称为*路由*。

## 路由

路由是指为客户端提供其请求的内容的机制。对于基于 Web 的客户端/服务器应用程序，客户端在 URL 中指定所需的内容；具体来说，路径和查询字符串（URL 的各个部分将在第六章中进行更详细的讨论）。

###### 注意

服务器路由传统上依赖于路径和查询字符串，但还有其他可用的信息：头部、域名、IP 地址等。这使得服务器可以考虑用户的大致物理位置或用户的首选语言等因素。

让我们扩展我们的“Hello world!”示例，做一些更有趣的事情。让我们提供一个非常简单的网站，包括一个主页、一个关于页面和一个未找到页面。目前，我们将继续使用我们之前的示例，只是提供纯文本而不是 HTML（在配套仓库中的*ch02/01-helloworld.js*）：

```
const http = require('http')
const port = process.env.PORT || 3000

const server = http.createServer((req,res) => {
  // normalize url by removing querystring, optional
  // trailing slash, and making it lowercase
  const path = req.url.replace(/\/?(?:\?.*)?$/, '').toLowerCase()
  switch(path) {
    case '':
      res.writeHead(200, { 'Content-Type': 'text/plain' })
      res.end('Homepage')
      break
    case '/about':
      res.writeHead(200, { 'Content-Type': 'text/plain' })
      res.end('About')
      break
    default:
      res.writeHead(404, { 'Content-Type': 'text/plain' })
      res.end('Not Found')
      break
  } })

server.listen(port, () => console.log(`server started on port ${port}; ` +
  'press Ctrl-C to terminate....'))
```

如果您运行这个示例，您会发现现在可以浏览到主页（*http://localhost:3000*）和关于页面（*http://localhost:3000/about*）。任何查询字符串都将被忽略（因此*http://localhost:3000/?foo=bar*将提供主页），任何其他 URL（*http://localhost:3000/foo*）将提供未找到页面。

## 提供静态资源

现在我们已经实现了一些简单的路由功能，让我们来提供一些真实的 HTML 和一个 logo 图像。这些被称为*静态*资源，因为它们通常不会改变（例如，与股票行情相反：每次重新加载页面时，股票价格可能会发生变化）。

###### 提示

使用 Node 提供静态资源适合开发和小型项目，但对于较大的项目，您可能希望使用代理服务器如 NGINX 或 CDN 来提供静态资源。有关更多信息，请参见第十七章。

如果你曾经使用过 Apache 或者 IIS，你可能习惯于只需创建一个 HTML 文件，导航到它，并自动将其传递给浏览器。Node 不是这样工作的：我们需要打开文件、读取文件，然后将其内容发送给浏览器。因此，让我们在项目中创建一个名为 *public* 的目录（为什么不叫 *static*，在下一章将会显而易见）。在该目录中，我们将创建 *home.html*、*about.html*、*404.html*，一个名为 *img* 的子目录，并且一个名为 *img/logo.png* 的图片。我会留给你来完成；如果你在阅读本书，你可能知道如何编写 HTML 文件和查找图片。在你的 HTML 文件中，像这样引用 logo：`<img src="/img/logo.png" alt="logo">`。

现在修改 *helloworld.js*（伴随的代码库中是 *ch02/02-helloworld.js*）：

```
const http = require('http')
const fs = require('fs')
const port = process.env.PORT || 3000

function serveStaticFile(res, path, contentType, responseCode = 200) {
  fs.readFile(__dirname + path, (err, data) => {
    if(err) {
      res.writeHead(500, { 'Content-Type': 'text/plain' })
      return res.end('500 - Internal Error')
    }
    res.writeHead(responseCode, { 'Content-Type': contentType })
    res.end(data)
  })
}

const server = http.createServer((req,res) => {
  // normalize url by removing querystring, optional trailing slash, and
  // making lowercase
  const path = req.url.replace(/\/?(?:\?.*)?$/, '').toLowerCase()
  switch(path) {
    case '':
      serveStaticFile(res, '/public/home.html', 'text/html')
      break
    case '/about':
      serveStaticFile(res, '/public/about.html', 'text/html')
      break
    case '/img/logo.png':
      serveStaticFile(res, '/public/img/logo.png', 'image/png')
      break
    default:
      serveStaticFile(res, '/public/404.html', 'text/html', 404)
      break
  }
})

server.listen(port, () => console.log(`server started on port ${port}; ` +
  'press Ctrl-C to terminate....'))
```

###### 注意

在这个例子中，我们对路由的设定并不是很有创意。如果你访问 *http://localhost:3000/about*，将会服务于 *public/about.html* 文件。你可以将路由更改为任何你想要的内容，并且更改文件为任何你想要的内容。例如，如果你每周有一个不同的关于页面，你可以有 *public/about_mon.html*、*public/about_tue.html* 等文件，并在你的路由中提供逻辑来在用户访问 *http://localhost:3000/about* 时服务于适当的页面。

注意我们创建了一个辅助函数，`serveStaticFile`，这个函数承担了大部分工作。`fs.readFile` 是一个用于读取文件的异步方法。有一个同步版本的函数，`fs.readFileSync`，但越早开始思考异步操作，效果越好。`fs.readFile` 函数使用了一种叫做 *回调* 的模式。你需要提供一个称为 *回调函数* 的函数，当工作完成时，该回调函数就会被调用（可以说是“回调”）。在这个案例中，`fs.readFile` 读取了指定文件的内容，并在文件读取完毕时执行回调函数；如果文件不存在或者在读取文件时存在权限问题，`err` 变量就会被设置，函数返回 HTTP 状态码 500，表示服务器错误。如果文件成功读取，文件会以指定的响应代码和内容类型发送到客户端。响应代码将在 第六章 中详细讨论。

###### Tip

`__dirname` 将会解析为当前执行脚本所在的目录。所以如果你的脚本位于 */home/sites/app.js*，`__dirname` 将会解析为 */home/sites*。尽可能使用这个方便的全局变量是个好主意。如果不这样做，当你从不同的目录运行应用时，可能会导致难以诊断的错误。

# 进入 Express

到目前为止，Node 可能对你来说并不那么令人印象深刻。我们基本上复制了 Apache 或 IIS 为您自动完成的工作，但现在你已经了解了 Node 如何做事情以及你有多少控制权。我们并没有做出什么特别引人注目的事情，但你可以看到我们可以将其作为一个起点，做更复杂的事情。如果我们继续这条路，编写越来越复杂的 Node 应用程序，最终可能会得到类似 Express 的东西……

幸运的是，我们不必从头开始：Express 已经存在，它可以帮助你避免实现许多耗时的基础设施。现在我们已经积累了一些 Node 经验，我们准备好开始学习 Express 了。

^(1) 当今，`vi`基本上等同于`vim`（vi 改进版）。在大多数系统上，`vi`被别名为`vim`，但我通常会输入`vim`来确保我在使用`vim`。
