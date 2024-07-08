# 第三章：使用 Express 节省时间

在第二章中，您学习了如何仅使用 Node 创建一个简单的 Web 服务器。在本章中，我们将使用 Express 重新创建该服务器。这将为本书其余内容提供一个起点，并介绍 Express 的基础知识。

# 脚手架

*脚手架*并非新概念，但许多人（包括我自己）是通过 Ruby 了解到这一概念的。其思想很简单：大多数项目都需要一定量的所谓*样板*代码，谁又愿意在每次开始新项目时重新创建这些代码呢？一个简单的方法是创建一个项目的粗略框架，每次需要新项目时，只需复制这个框架或模板即可。

Ruby on Rails 将这一概念推进了一步，提供了一个可以自动生成脚手架的程序。这种方法的优势在于，它可以生成比仅从模板集合中选择更复杂的框架。

Express 借鉴了 Ruby on Rails 的一些方法，并提供了一个实用程序来生成脚手架，以启动您的 Express 项目。

虽然 Express 脚手架实用程序很有用，但我认为从零开始学习如何设置 Express 也很有价值。除了学到更多知识外，您还可以控制所安装的内容以及项目的结构。此外，Express 脚手架实用程序面向的是服务器端 HTML 生成，对于 API 和单页面应用程序则不太相关。

虽然我们不会使用脚手架实用程序，但我鼓励您在完成本书后查看它：到那时，您将掌握评估它生成的脚手架是否对您有用的一切知识。更多信息，请参阅[`express-generator`文档](http://bit.ly/2CyvvLr)。

# [Meadowlark Travel 网站](https://meadowlark.example.org) 

在本书的整个过程中，我们将使用一个运行示例：Meadowlark Travel 的虚构网站，该公司为访问俄勒冈州的人们提供服务。如果您更感兴趣创建 API，不必担心：Meadowlark Travel 网站将除了提供功能性网站外还提供 API。

# 初始步骤

首先创建一个新目录：这将是您项目的根目录。在本书中，无论我们提到项目目录、应用目录还是项目根目录，我们都指的是这个目录。

###### 提示

您可能希望将 Web 应用文件与通常陪伴项目的所有其他文件分开，例如会议记录、文档等。因此，我建议将项目根目录设置为项目目录的子目录。例如，对于 Meadowlark Travel 网站，我可能会将项目保存在*~/projects/meadowlark*，并将项目根目录设置为*~/projects/meadowlark/site*。

npm 在一个名为*package.json*的文件中管理项目依赖项以及有关项目的元数据。创建此文件的最简单方法是运行`npm init`：它会询问您一系列问题并生成一个*package.json*文件，以便您开始（对于“入口点”问题，请使用*meadowlark.js*作为项目名称）。

###### 小贴士

每次运行 npm 时，可能会收到有关缺少描述或存储库字段的警告。可以安全地忽略这些警告，但如果您想消除它们，可以编辑*package.json*文件并为 npm 正在投诉的字段提供值。有关此文件中字段的更多信息，请参阅[npm *package.json*文档](http://bit.ly/2O8HrbW)。

第一步将是安装 Express。运行以下 npm 命令：

```
npm install express
```

运行`npm install`将安装所命名的包到*node_modules*目录，并更新*package.json*文件。由于*node_modules*目录可以随时用 npm 重新生成，我们不会将其保存在仓库中。为了确保不会意外将其添加到仓库中，我们创建一个名为*.gitignore*的文件：

```
# ignore packages installed by npm
node_modules

# put any other files you don't want to check in here, such as .DS_Store
# (OSX), *.bak, etc.
```

现在创建一个名为*meadowlark.js*的文件。这将是我们项目的入口点。在整本书中，我们将简称此文件为*应用文件*（在伴随的仓库中是*ch03/00-meadowlark.js*）：

```
const express = require('express')

const app = express()

const port = process.env.PORT || 3000

// custom 404 page
app.use((req, res) => {
  res.type('text/plain')
  res.status(404)
  res.send('404 - Not Found')
})

// custom 500 page
app.use((err, req, res, next) => {
  console.error(err.message)
  res.type('text/plain')
  res.status(500)
  res.send('500 - Server Error')
})

app.listen(port, () => console.log(
  `Express started on http://localhost:${port}; ` +
  `press Ctrl-C to terminate.`))
```

###### 小贴士

许多教程以及 Express 脚手架生成器都鼓励您将主文件命名为*app.js*（有时是*index.js*或*server.js*）。除非您使用要求主应用程序文件具有特定名称的托管服务或部署系统，否则我认为没有必要这样做，我更喜欢以项目命名主文件。任何曾经看着一堆标签都写着“index.html”的人立即会看出这种做法的智慧。`npm init`将默认为*index.js*；如果您使用不同的名称作为应用程序文件，请确保更新*package.json*中的`main`属性。

现在您拥有了一个最小化的 Express 服务器。您可以启动服务器（`node meadowlark.js`）并导航至*http://localhost:3000*。结果可能会让人失望：您尚未为 Express 提供任何路由，因此它将只是给出一个通用的 404 消息，表明页面不存在。

###### 注意

注意我们如何选择我们希望应用程序运行的端口：`const port = process.env.PORT || 3000`。这允许我们在启动服务器之前通过设置环境变量来覆盖端口。如果在运行此示例时您的应用程序未在端口 3000 上运行，请检查您的`PORT`环境变量是否已设置。

让我们为主页和关于页面添加一些路由。在 404 处理程序之前，我们将添加两个新路由（在伴随的仓库中是*ch03/01-meadowlark.js*）：

```
app.get('/', (req, res) => {
  res.type('text/plain')
  res.send('Meadowlark Travel');
})

app.get('/about', (req, res) => {
  res.type('text/plain')
  res.send('About Meadowlark Travel')
})

// custom 404 page
app.use((req, res) => {
  res.type('text/plain')
  res.status(404)
  res.send('404 - Not Found')
})
```

`app.get` 是我们添加路由的方法。在 Express 文档中，你会看到 `app.METHOD`。这并不意味着真的有一个叫做 `METHOD` 的方法；它只是一个占位符，代表（小写的）HTTP 动词（`get` 和 `post` 是最常见的）。这个方法接受两个参数：路径和一个函数。

*路径* 定义了路由。请注意，`app.METHOD` 为您完成了大部分工作：默认情况下，它不区分大小写或尾随斜杠，并且在匹配时不考虑查询字符串。因此，About 页面的路由将适用于 */about*、*/About*、*/about/*、*/about?foo=bar*、*/about/?foo=bar* 等。

当路由匹配时，您提供的函数将被调用。传递给该函数的参数是请求和响应对象，我们将在第六章更详细地了解它们。现在，我们只是返回带有状态码 200 的纯文本（Express 默认状态码为 200，您不必显式指定它）。

###### 提示

我强烈建议安装一个浏览器插件，它可以显示 HTTP 请求的状态码以及任何重定向，这将有助于您快速发现代码中的重定向问题或不正确的状态码，这些问题通常被忽视。对于 Chrome，Ayima 的 Redirect Path 插件效果非常好。在大多数浏览器中，您可以在开发者工具的网络部分看到状态码。

我们不再使用 Node 的底层方法 `res.end`，而是切换到 Express 提供的 `res.send`。我们还用 `res.set` 和 `res.status` 替换了 Node 的 `res.writeHead`。Express 还为我们提供了一个便捷方法 `res.type`，它设置 `Content-Type` 头部。虽然仍然可以使用 `res.writeHead` 和 `res.end`，但这并不是必需或建议的。

请注意，我们的自定义 404 和 500 页面必须稍有不同处理。我们不再使用 `app.get`，而是使用 `app.use`。`app.use` 是 Express 添加*中间件*的方法。我们将在第十章更深入地讨论中间件，但现在你可以将其视为一个捕捉未被路由匹配的所有请求的处理程序。这带来了一个重要的观点：*在 Express 中，添加路由和中间件的顺序非常重要*。如果我们将 404 处理程序放在路由之上，主页和 About 页面将无法正常工作；相反，这些 URL 将导致 404 错误。目前，我们的路由相当简单，但也支持通配符，这可能会导致顺序问题。例如，如果我们想为 About 添加子页面，如 */about/contact* 和 */about/directions*，以下方法将无法按预期工作：

```
app.get('/about*', (req,res) => {
  // send content....
}) app.get('/about/contact', (req,res) => {
  // send content....
}) app.get('/about/directions', (req,res) => {
  // send content....
})
```

在这个例子中，`/about/contact` 和 `/about/directions` 处理程序永远不会匹配，因为第一个处理程序在其路径中使用了通配符：`/about*`。

Express 可以通过它们的回调函数接受的参数数量来区分 404 和 500 处理程序。错误路由将在第十章和第十二章中深入讨论。

现在你可以重新启动服务器，看看是否有一个正常运行的首页和关于页面。

到目前为止，我们还没有做过任何不用 Express 就能轻松完成的事情，但是 Express 已经为我们提供了一些功能，这些功能并不是立即显而易见的。还记得上一章中我们如何规范化`req.url`以确定所请求的资源吗？我们不得不手动去掉查询字符串和尾随斜杠，并转换为小写。现在 Express 的路由器已经自动处理了这些细节。虽然现在看起来可能不是很大的事情，但这只是 Express 路由器能够实现的功能的冰山一角。

## 视图和布局

如果你熟悉“模型-视图-控制器”范式，那么视图的概念对你来说不会陌生。本质上，*视图*就是向用户传递的内容。对于网站来说，通常意味着 HTML，尽管你也可以传递 PNG、PDF 或任何客户端可以呈现的内容。在我们的目的中，我们将视图视为 HTML。

视图与静态资源（如图像或 CSS 文件）不同之处在于视图不一定是静态的：HTML 可以动态生成，以为每个请求提供定制页面。

Express 支持许多不同的视图引擎，提供不同程度的抽象化。Express 对名为*Pug*的视图引擎有些偏爱（这一点并不奇怪，因为它也是 TJ Holowaychuk 的作品）。Pug 采用的方法很简洁：你写的东西完全不像 HTML，这无疑减少了很多打字（不再有尖括号或闭合标签）。然后 Pug 引擎会将其转换为 HTML。

###### 注意

Pug 最初被称为 Jade，在第 2 版发布时更名是因为商标问题。

Pug 很吸引人，但这种抽象程度是有代价的。如果你是前端开发人员，即使你实际上是在用 Pug 编写视图，你也必须深入理解 HTML，并且要理解得很透彻。我认识的大多数前端开发人员对他们的主要标记语言被抽象化这个想法感到不舒服。因此，我建议使用另一种不那么抽象的模板框架*Handlebars*。

Handlebars（基于流行的语言无关模板语言 Mustache）不会试图为你抽象 HTML：你需要使用特殊的标记编写 HTML，这些标记允许 Handlebars 注入内容。

###### 注意

在这本书最初发布后的几年里，React 风靡全球……这使得 HTML 对于前端开发者来说被抽象化了！透过这个视角看，我预测前端开发者不希望 HTML 被抽象化的说法并没有经受住时间的考验。然而，JSX（大多数 React 开发者使用的 JavaScript 语言扩展）几乎与编写 HTML 相同，所以我并不完全错。

为了提供 Handlebars 支持，我们将使用 Eric Ferraiuolo 的`express-handlebars`包。在项目目录中，执行以下操作：

```
npm install express-handlebars
```

然后在*meadowlark.js*中修改前几行（*ch03/02-meadowlark.js*在配套存储库中）：

```
const express = require('express')
const expressHandlebars = require('express-handlebars')

const app = express()

// configure Handlebars view engine
app.engine('handlebars', expressHandlebars({
  defaultLayout: 'main',
}))
app.set('view engine', 'handlebars')
```

这将创建一个视图引擎，并配置 Express 默认使用它。现在创建一个名为*views*的目录，其中有一个名为*layouts*的子目录。如果你是一名经验丰富的 Web 开发者，你可能已经对*layouts*（有时称为*master pages*）的概念感到非常熟悉。当你构建一个网站时，有一部分 HTML 在每个页面上是相同的或非常接近相同的。重复编写所有这些重复的代码不仅变得乏味，而且可能造成维护的噩梦：如果你想在每个页面上做一些改动，你必须改动*所有*的文件。布局解放了你的手，为你的网站上的所有页面提供了一个共同的框架。

所以让我们为我们的网站创建一个模板。创建一个名为*views/layouts/main.handlebars*的文件：

```
<!doctype html>
<html>
  <head>
    <title>Meadowlark Travel</title>
  </head>
  <body>
    {{{body}}}
  </body>
</html>
```

你可能之前没有见过的唯一一件事情是这个：`{{{body}}}`。这个表达式将被每个视图的 HTML 替换。当我们创建 Handlebars 实例时，请注意我们指定了默认布局（`defaultLayout: \'main'`）。这意味着除非你另行指定，否则这将是任何视图使用的布局。

现在让我们为我们的主页创建视图页面，*views/home.handlebars*：

```
<h1>Welcome to Meadowlark Travel</h1>
```

然后是我们的关于页面，*views/about.handlebars*：

```
<h1>About Meadowlark Travel</h1>
```

然后是我们的找不到页面，*views/404.handlebars*：

```
<h1>404 - Not Found</h1>
```

最后我们的服务器错误页面，*views/500.handlebars*：

```
<h1>500 - Server Error</h1>
```

###### 提示

你可能希望你的编辑器将*.handlebars*和*.hbs*（Handlebars 文件的另一种常见扩展名）与 HTML 关联起来，以启用语法高亮和其他编辑器功能。对于 vim，你可以在*~/.vimrc*文件中添加一行`au BufNewFile,BufRead *.handlebars set filetype=html`。对于其他编辑器，请查阅其文档。

现在我们有了一些视图设置，我们必须用这些视图替换旧的路由（*ch03/02-meadowlark.js*在配套存储库中）：

```
app.get('/', (req, res) => res.render('home'))

app.get('/about', (req, res) => res.render('about'))

// custom 404 page
app.use((req, res) => {
  res.status(404)
  res.render('404')
})

// custom 500 page
app.use((err, req, res, next) => {
  console.error(err.message)
  res.status(500)
  res.render('500')
})
```

请注意，我们不再需要指定内容类型或状态码：视图引擎将默认返回`text/html`的内容类型和状态码 200。在提供我们自定义 404 页面的通用处理程序以及 500 处理程序中，我们必须显式设置状态码。

如果你启动服务器并查看主页或关于页面，你会发现视图已经渲染完毕。如果你查看源代码，你会看到来自*views/layouts/main.handlebars*的样板 HTML 代码。

尽管每次访问主页时，你都会得到相同的 HTML，但这些路由被视为*动态内容*，因为我们每次调用路由时可以做出不同的决策（这在本书的后面部分会有很多例子）。然而，从来不变的内容，换句话说，静态内容，是常见且重要的，所以我们接下来会考虑静态内容。

## 静态文件和视图

Express 依赖于*中间件*来处理静态文件和视图。中间件是一个将在第十章中更详细介绍的概念。现在，只需知道中间件提供了模块化，使得处理请求更加容易即可。

`static`中间件允许你指定一个或多个目录，其中包含静态资源，这些资源会简单地提供给客户端，不需要任何特殊处理。这是你放置图片、CSS 文件和客户端 JavaScript 文件等内容的地方。

在你的项目目录中，创建一个名为*public*的子目录（我们称之为*public*，因为该目录中的任何内容都会毫不保留地提供给客户端）。然后，在声明任何路由之前，你将添加`static`中间件（*ch03/02-meadowlark.js*在配套的代码库中）：

```
app.use(express.static(__dirname + '/public'))
```

`static`中间件的作用与创建每个要提供的静态文件的路由具有相同的效果，它渲染一个文件并将其返回给客户端。因此，让我们在*public*目录内创建一个*img*子目录，并把我们的*logo.png*文件放在其中。

现在我们可以简单地引用*/img/logo.png*（注意，我们不指定`public`；该目录对客户端是不可见的），而`static`中间件会适当地提供该文件，设置内容类型。现在让我们修改我们的布局，以便我们的 logo 出现在每一页上：

```
<body>
  <header>
    <img src="/img/logo.png" alt="Meadowlark Travel Logo">
  </header>
  {{{body}}}
</body>
```

###### 注意

记住，中间件按顺序处理，通常首先声明或至少非常早地声明的静态中间件将覆盖其他路由。例如，如果你在*public*目录中放置一个*index.html*文件（试试看！），你会发现该文件的内容会被提供，而不是你配置的路由！因此，如果你得到混乱的结果，请检查你的静态文件，并确保没有意外匹配到路由。

## 视图中的动态内容

视图并不仅仅是传递静态 HTML 的复杂方式（尽管它们当然也可以这样做）。视图真正的威力在于它们可以包含动态信息。

假设在关于页面上，我们想要提供一个“虚拟幸运饼干”。在我们的*meadowlark.js*文件中，我们定义了一个幸运饼干的数组：

```
const fortunes = [
  "Conquer your fears or they will conquer you.",
  "Rivers need springs.",
  "Do not fear what you don't know.",
  "You will have a pleasant surprise.",
  "Whenever possible, keep it simple.",
]
```

修改视图（*/views/about.handlebars*）以显示一则幸运饼干：

```
<h1>About Meadowlark Travel</h1>
{{#if fortune}}
  <p>Your fortune for the day:</p>
  <blockquote>{{fortune}}</blockquote>
{{/if}}
```

现在修改路由*/about*，以提供随机的幸运饼干：

```
app.get('/about', (req, res) => {
  const randomFortune = fortunes[Math.floor(Math.random()*fortunes.length)]
  res.render('about', { fortune: randomFortune })
})
```

现在，如果你重新启动服务器并加载 */about* 页面，你会看到一个随机的幸运语，并且每次重新加载页面都会得到一个新的幸运语。模板化非常有用，我们将在第七章中深入讲解。

## 结论

我们用 Express 创建了一个基本的网站。尽管它很简单，但已经包含了我们建立完整功能网站所需的所有基础。在下一章中，我们将开始准备添加更高级功能，严谨地做好每一个细节。
