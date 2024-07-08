# 第二十一章：使用 Express 构建 Web 应用程序

[Express](https://expressjs.com)是一个轻量级的 Web 框架，在 Node 中长期领先于 Web 应用程序开发。与 Ruby 的 Sinatra 和 Python 的 Flask 类似，Express 框架本身非常简洁，但可以扩展以构建任何类型的 Web 应用程序。Express 也是 Web 应用程序框架中的一部分，例如[Keystone.js](https://keystonejs.com)、[Sails](https://sailsjs.com)和[Vulcan.js](http://vulcanjs.org)。如果你在 Node 中进行 Web 应用程序开发，你很可能会遇到 Express。本章重点介绍了使用 Express 处理基本应用程序的几种方法，可以扩展到各种 Web 应用程序。

# 使用 Express 响应请求

## 问题

您的 Node 应用程序需要响应 HTTP 请求。

## 解决方案

安装 Express 包：

```
$ npm install express
```

要设置 Express，我们需要引入模块，调用模块，并在名为*index.js*的文件中指定一个连接端口：

```
const express = require('express');

const app = express();
const port = process.env.PORT || '3000';

app.listen(port, () => console.log(`Listening on port ${port}`));
```

要响应请求，指定一个路由并使用 Express 的`.get`方法进行响应：

```
const express = require('express');

const app = express();
const port = process.env.PORT || '3000';

app.get('/', (req, res) => res.send('Hello World'));

app.listen(port, () => console.log(`Listening on port ${port}`));
```

要提供静态文件，我们可以使用`express.static`中间件指定一个目录。

```
const express = require('express');

const app = express();
const port = process.env.PORT || '3000';

// middleware for static files
// will serve static files from the 'files' directory
app.use(express.static('files'));

app.listen(port, () => console.log(`Listening on port ${port}`));
```

要使用从模板生成的 HTML 来响应，请先安装模板引擎：

```
$ npm install pug --save
```

接下来，在*index.js*文件中设置`view engine`并指定将以模板内容响应的路由：

```
app.set('view engine', 'pug')

app.get('/template', (req, res) => {
  res.render('template');
});
```

然后在项目的*views*子目录中创建一个新文件，作为模板文件。模板文件名应与`res.render`中指定的名称匹配。在*views/template.pug*中：

```
html
  head
    title="Using Express"
  body
    h1="Hello World"
```

现在，对*http://localhost:3000/template*的请求将返回 HTML 格式的模板内容。

## 讨论

Express 是一个极简但高度可配置的框架，用于响应 HTTP 请求和构建 Web 应用程序。在示例中，我们将端口设置为`process.env.PORT`或端口`3000`。在开发中，我们可以通过环境变量指定一个新的端口：

```
$ PORT=7777 node index.js
```

或者通过使用配对`dotenv`Node 模块的*.env*文件。在部署应用程序时，应用程序托管平台可能需要特定的端口号或允许我们自己配置端口号。

使用 Express 的`get`方法，应用程序接收到特定 URI 的请求，然后做出响应。在我们的例子中，当应用程序接收到根 URI（*/*）的请求时，我们会回复“Hello World”文本：

```
app.get('/', (req, res) => res.send('Hello World'));
```

这些响应也可以是 HTML、渲染为 HTML 的模板、静态文件和格式化数据（如 JSON 或 XML）。

由于其极简的特性，Express 本身包含了极少的功能，但可以通过中间件进行扩展。在 Express 中，中间件函数可以访问`request`和`response`对象。应用级中间件通过`app.use(MIDDLEWARE)`绑定到`app`对象的一个实例上。在这个例子中，我们正在使用内置的静态文件中间件：

```
app.use(express.static('files'));
```

中间件包可以在许多方面扩展 Express 的功能。`helmet` 中间件包可以用于改善 Express 的安全默认设置：

```
const express = require('express');
const helmet = require('helmet');

const app = express();

app.use(helmet());
```

模板引擎简化了编写 HTML 的过程，并允许你将数据传递给页面。

这里我正在从 `userData` 对象中传递数据到 *views/user.pug* 模板，该模板可以在 */user* 路由中访问：

```
// a user object of data to send to the template
const userData = {
  name: 'Adam',
  email: 'adam@jseverywhere.io',
  avatar: 'https://s.gravatar.com/avatar/33aab819d1ffa11fc4b31a4eebaf0c5a?s=80'
};

// render the template with user data
app.get('/user', (req, res) => {
  res.render('user', { userData });
});
```

然后在我们的模板中，我们可以使用数据：

```
html
  head
    title User Page
  body
    h1 #{userData.name} Profile
    ul
      li
        image(src=userData.avatar)
      li #{userData.name}
      li #{userData.email}
```

Pug 模板引擎由 Express 核心团队维护，是 Express 应用程序的流行选择，但其基于空白符的语法不适合每个人。[EJS](https://ejs.co) 是一个更接近 HTML 的语法的优秀选择。以下是使用 EJS 的上述示例的效果。

首先，指定安装 `ejs` 包：

```
$ npm install ejs
```

然后在你的 Express 应用程序中设置 EJS 作为视图引擎：

```
app.set('view engine', 'ejs');
```

而在 *views/user.ejs* 中：

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>User Page</title>
  </head>
  <body>
    <h1><%= userData.name %> Profile</h1>
    <ul>
      <li><img src=<%= userData.avatar %> /></li>
      <li><%= userData.name %></li>
      <li><%= userData.email %></li>
    </ul>
  </body>
</html>
```

# 使用 Express-Generator

## 问题

你有兴趣使用 Express 管理你的服务器端数据应用程序，但不想自己处理所有设置。

## 解决方案

要启动你的 Express 应用程序，请使用 Express-Generator。这是一个命令行工具，用于生成典型 Express 应用程序的基础架构骨架。

首先，创建一个工作目录，在这里工具可以安全地安装一个新的应用程序子目录。接下来，使用 `npx` 运行 `express-generator` 命令：

```
$ npx express-generator --pug --git
```

我在命令中传递了两个选项：`--pug` 将使用 Pug 模板引擎，而 `--git` 将在项目目录中生成默认的 *.gitignore* 文件。要查看所有选项，请使用 `-h` 选项运行生成器：

```
$ npx express-generator -h
```

生成器创建一个新目录，包含多个子目录、一些基本文件和一个包含所有依赖项的 *package.json* 文件。要安装依赖项，请切换到新创建的目录并输入：

```
$ npm install
```

一旦安装了所有依赖项，使用以下命令运行应用程序：

```
$ npm start
```

现在，你可以通过你的 IP 地址或域名和端口 3000 访问生成的 Express 应用程序，这是 Express 的默认端口。

## 讨论

Express 提供了一个基于 Node 的 Web 应用程序框架，并支持多个模板引擎和 CSS 预处理器。在这个解决方案中，我选择的示例应用程序选项是 Pug 作为模板引擎（默认设置）和纯 CSS（无 CSS 预处理器）。虽然从头开始构建应用程序可以选择更多选项，但 Express 仅支持以下模板引擎：

`--ejs`

添加对 EJS 模板引擎的支持

`--pug`

添加对 Pug 模板引擎的支持

`--hbs`

添加对 Handlebar 模板引擎的支持

`--hogan`

添加对 Hogan.js 模板引擎的支持

Express 还支持以下 CSS 预处理器：

`express --css sass`

支持 Sass

`express --css less`

支持 Less

`express --css stylus`

支持 Stylus

`express --css compass`

支持 Compass

如果不指定任何 CSS 预处理器，默认为纯 CSS。

Express 还假定项目目录为空。如果不是，可以通过使用 `-f` 或 `--force` 选项强制 Express 生成内容。

新生成的子目录具有以下结构（忽略 `node_modules`）：

```
app.js
package-lock.json
package.json
/bin
   www
/node_modules
/public
   /images
   /javascripts
   /stylesheets
      style.css
      style.styl
/routes
   index.js
   users.js
/views
   error.pug
   index.pug
   layout.pug
```

*app.js* 文件是 Express 应用程序的核心。它包括对必要库的引用：

```
var createError = require('http-errors');
var express = require('express');
var path = require('path');
var cookieParser = require('cookie-parser');
var logger = require('morgan');

var indexRouter = require('./routes/index');
var usersRouter = require('./routes/users');
```

###### 注意

尽管本书遵循的惯例是在定义变量时使用 `const` 和 `let`，但在编写时，Express 生成器使用 `var`。

还使用以下行创建 Express 应用程序：

```
var app = express():
```

接下来，通过定义 `views` 和 `view engine` 变量，将 Pug 设置为视图引擎：

```
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'pug');
```

接下来通过 `app.use()` 加载 *middleware* 调用。中间件是位于原始请求和路由之间的功能，用于处理特定类型的请求。中间件的规则是如果不给定路径作为第一个参数，则默认路径为 `/`，这意味着中间件函数加载了默认路径。在以下生成的代码中：

```
app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));
```

每个应用程序请求都加载了前几个中间件。其中中间件包括开发日志记录的支持，以及 JSON 和 *urlencoded* 体的解析器。只有当我们到达 `static` 条目时，我们看到分配到特定路径的静态文件请求中间件：当请求发送到 *public* 目录时加载静态文件请求中间件。

接下来处理路由：

```
app.use('/', indexRouter);
app.use('/users', usersRouter);
```

顶级 Web 请求 (*/*) 被定向到 `routes` 模块，而所有用户请求 (*/users*) 被路由到 `users` 模块。

###### 注意

了解更多关于 Express 中的路由的信息，请参阅 “Routing”。

接下来是错误处理。首先处理的是当请求到一个不存在的 Web 资源时的 `404` 错误处理：

```
app.use(function(req, res, next) {
  next(createError(404));
});
```

接下来是服务器错误处理，适用于生产和开发环境：

```
app.use(function(err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  res.render('error');
});
```

生成文件的最后一行是 `module.exports` 的 `app`：

```
module.exports = app;
```

在 *routes* 子目录中，默认路由包含在 *routes/index.js* 文件中：

```
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});

module.exports = router;
```

文件中发生的情况是 Express 路由器用于将任何 HTTP GET 请求路由到 `/` 的回调函数中，其中请求响应接收为特定资源页面渲染的视图。这与 *routes/users.js* 文件中发生的情况形成对比，后者响应接收的是文本消息而不是视图：

```
var express = require('express');
var router = express.Router();

/* GET users listing. */
router.get('/', function(req, res, next) {
  res.send('respond with a resource');
});

module.exports = router;
```

在第一个请求中视图渲染会发生什么？在 *views* 子目录中有三个 Pug 文件：一个用于错误处理，一个定义页面布局，还有一个 *index.pug*，用于渲染页面。*index.pug* 文件包含：

```
extends layout

block content
  h1= title
  p Welcome to #{title}
```

它扩展了 *layout.pug* 文件，其中包含：

```
doctype html
html
  head
    title= title
    link(rel='stylesheet', href='/stylesheets/style.css')
  body
    block content
```

*layout.pug* 文件定义了页面的整体结构，包括对自动生成的 CSS 文件的引用。`block content` 设置定义了内容的位置。内容的格式在 *index.js* 中定义，与同名的 `block content` 设置相对应。

###### 注意

Pug 模板引擎（以前称为 Jade）由 Express 推广，并提供了一种极简的模板化方式，使用空白符代替传统的 HTML 样式标签。这种方法可能不适合所有人，而 Pug 的替代方案（Handlebars、Hogan.js 和 EJS）都提供了更接近 HTML 的语法。

这两个 Pug 文件定义了一个基本的网页，其中包含一个分配了标题变量的 `h1` 元素，以及包含欢迎消息的段落。图 21-1 显示了默认页面。

![jsc3 2101](img/jsc3_2101.png)

###### 图 21-1\. Express 生成的网页

图 21-1 显示了页面并不特别吸引人，但它确实展示了这些组件如何协同工作：应用程序路由将请求路由到适当的路由模块，该模块将响应定向到适当的渲染视图，并使用传递给它的数据生成网页。如果您进行以下 Web 请求：

```
http://yourdomain.com:3000/users
```

您将看到纯文本消息，而不是渲染后的视图。

默认情况下，Express 设置为运行在 *开发模式* 下。要将应用程序切换到 *生产模式*，您需要设置一个 *环境变量* `NODE-ENV` 为 “production”。在 Linux 或 Unix 环境中，可以使用以下方式：

```
$ export NODE_ENV=production
```

# 路由

## 问题

您希望根据请求将用户路由到应用程序中的不同资源。

## 解决方案

在 Express 中使用路由，根据请求路径和参数发送特定资源：

```
// respond with different route paths
app.get('/', (req, res) => res.send('Hello World'));
app.get('/users', (req, res) => res.send('Hello users'));

// parameters
app.get('/users/:userId', (req, res) => {
  res.send(`Hello user ${req.params.userId}`);
});
```

## 讨论

在 Express 中，当用户发出 HTTP 请求时，我们可以向其返回响应。在上面的示例中，我使用了 `get` 请求，但 Express 支持多种额外的方法。其中最常见的方法包括：

+   `app.get`: 请求数据

+   `app.post`: 发送数据

+   `app.put`: 发送或更新数据

+   `app.delete`: 删除数据

```
app.post('/new', (req, res) => {
  res.send('POST request to the `new` route');
});
```

通常情况下，我们可能希望对特定路由启用多个 HTTP 方法。我们可以通过将它们链接在一起来实现这一目标：

```
app
  .route('/record')
  .get((req, res) => {
    res.send('Get a record');
  })
  .post((req, res) => {
    res.send('Add a record');
  })
  .put((req, res) => {
    res.send('Update a record');
  });
```

请求通常具有具体值的参数，我们将在应用程序中使用这些值。我们可以在 URL 中使用冒号 (`:`) 指定这些值：

```
app.get('/users/:userId', (req, res) => {
  res.send(`Hello user ${req.params.userId}`);
});
```

在上面的示例中，当用户访问 */users/adam123* 这个 URL 时，浏览器将发送 `Hello user adam123` 的响应。虽然这是一个简单的示例，但我们也可以利用 URL 参数从数据库中检索数据，并将信息传递给模板。

我们还能够为请求参数指定格式。在以下示例中，我使用正则表达式将 `noteId` 参数限制为六位整数：

```
app.get('^/users/:userId/notes/:noteId([0-9]{6})', (req, res) => {
  res.send(`This is note ${req.params.noteId}`);
});
```

我们还可以使用正则表达式来定义整个路由：

```
app.get(/.*day$/, (req, res) => {
  res.send(`Every day feels like ${req.path}`);
});
```

上述示例将路由任何以`day`结尾的请求。例如，在本地开发中，请求*http://localhost:3000/Sunday*将在页面上打印出“Every day feels like Sunday”。

# 使用 OAuth 进行工作

## 问题

你需要在 Node 应用程序中访问第三方 API（如 GitHub、Facebook 或 Twitter），但它需要授权。具体而言，它需要 OAuth 授权。

## 解决方案

需要在你的应用程序中集成一个 OAuth 客户端。你还需要满足资源提供者所要求的 OAuth 需求。

查看详细讨论。

## 讨论

OAuth 是一个授权框架，与大多数流行的社交媒体和云内容应用程序一起使用。如果你曾经访问过一个网站，并被要求授权访问来自第三方服务的数据，比如 GitHub，那么你已经参与了 OAuth 授权*流程*。

OAuth 有两个版本，1.0 和 2.0，它们彼此不兼容。OAuth 1.0 基于 Flickr 和 Google 开发的专有 API，主要集中在网页上，并未很好地跨越 Web、移动和服务应用程序之间的障碍。在想要在手机应用程序中访问资源时，应用程序需要用户在移动浏览器中登录应用程序，然后将访问令牌复制到应用程序中。对 OAuth 1.0 的其他批评是，授权服务器必须与资源服务器相同，这在涉及 Twitter、Facebook 和 Amazon 等服务提供商时无法扩展。

OAuth 2.0 提供了一个更简单的授权流程，并为不同情况提供了不同类型的授权（不同流程）。尽管如此，有人会说，这是以安全为代价的，因为它对加密哈希令牌和请求字符串没有同样的要求。

大多数开发者不必创建 OAuth 2.0 服务器，这超出了本书甚至本节的范围。但是，应用程序通常会集成一个 OAuth 客户端（1.0 或 2.0）来使用某项服务，因此我将介绍不同类型的 OAuth 使用。首先，让我们讨论授权与认证之间的区别。

### 授权并非认证

授权意味着：“我授权此应用程序访问我在你服务器上的资源。”认证是验证你是否确实是拥有此账户并控制这些资源的人。例如，如果我想在报纸的在线站点上发表评论，它可能会要求我通过某些服务登录。如果我选择使用我的 Facebook 账户作为登录方式，新闻网站很可能会要求一些来自 Facebook 的数据。

首先，新闻网站验证我是合法的 Facebook 用户，拥有已建立的 Facebook 账户。换句话说，我不是只是随便进来匿名评论的任何人。其次，新闻网站希望在为评论提供特权的同时从我这里获取一些东西：它将要求获取有关我的数据。也许它会请求允许代表我发布（如果我将我的评论同时发布到 Facebook 和新闻网站）。这既是身份验证又是授权请求。

如果我还没有登录 Facebook，我将不得不登录。Facebook 使用我正确的用户名和密码来验证，确认我拥有所讨论的 Facebook 账户。登录后，Facebook 会询问我是否同意允许新闻网站访问它想要的资源。如果我同意（因为我非常想评论某个特定的故事），Facebook 会授予新闻网站授权，此时从报纸到我的 Facebook 账户之间就建立了持久连接（你可以在你的 Facebook 设置中看到）。我可以发表我的评论，以及在其他故事中发表评论，直到我注销或撤销 Facebook 授权。

当然，这并不意味着 Facebook 或新闻网站实际上在验证我的身份。在这种情况下，身份验证是指确认我是 Facebook 账户的所有者。只有在社交媒体环境（如 Twitter 为名人创建的认证账户）中，才会涉及*真正*的身份验证。

我们的开发任务因为处理授权的软件通常也是验证个人身份的软件而变得更简单，所以我们不需要处理两种不同的 JavaScript 库/模块/系统。在 Node 应用中，有几个优秀的 OAuth（1.0 和 2.0）模块可供使用。其中最流行的之一是[Passport](http://www.passportjs.org)，并且还有为 Passport 系统专门创建的各种授权服务扩展。然而，也有一些非常简单的 OAuth 客户端，专门为各种服务提供基础授权访问，以及一些为单一服务创建的模块。

###### 注：

Passport.js 的内容可在“使用 Passport.js 进行 OAuth 2 用户身份验证”中找到。您也可以在其网站上了解更多关于 Passport 及其支持不同服务器的各种*策略*。

现在，进入技术方面。

### 客户端凭据授权

如今几乎没有网页资源提供可以在没有某种授权凭证的情况下访问的 API。这意味着必须要向最终用户发出一个往返指令——要求他们在应用程序可以访问数据之前授权访问其服务账户。问题在于，有时你只需要简单的只读访问权限，而不需要更新权限，也不需要前端登录界面，并且也不需要特定用户进行授权授予。

OAuth 2.0 考虑了使用*客户端凭据授予*进行特定类型的授权流程。这种简化授权的图示显示在图 21-2 中。

![jsc3 2102](img/jsc3_2102.png)

###### 图 21-2. 客户端凭据授权流程

Twitter 提供了所谓的应用程序授权，基于 OAuth 2.0 的客户端凭据授予。我们可以使用这种授权类型来访问 Twitter 的搜索 API。

在下面的示例中，我使用了 Node 模块`oauth`来实现授权。这是授权模块中最基本的一种，支持 OAuth 1.0 和 OAuth 2.0 的授权流程：

```
const OAuth = require('oauth');
const fetch = require('node-fetch');
const { promisify } = require('util');

// read Twitter keys from a .env file
require('dotenv').config();

// Twitter's search API endpoint and the query we'll be searching
const endpointUrl = 'https://api.twitter.com/2/tweets/search/recent';
const query = 'javascript';

async function getTweets() {
  // consumer key and secret passed in from environment variables
  const oauth2 = new OAuth.OAuth2(
    process.env.TWITTER_CONSUMER_KEY,
    process.env.TWITTER_CONSUMER_SECRET,
    'https://api.twitter.com/',
    null,
    'oauth2/token',
    null
  );

  // retrieve the credentials from Twitter
  const getOAuthAccessToken = promisify(
    oauth2.getOAuthAccessToken.bind(oauth2)
  );
  const token = await getOAuthAccessToken('', {
    grant_type: 'client_credentials'
  });

  // make the request for data with the retrieved token
  const res = await fetch(`${endpointUrl}?query=${query}`, {
    headers: {
      authorization: `Bearer ${token}`
    }
  });

  const json = await res.json();
  return json;
}

(async () => {
  try {
    // Make request
    const response = await getTweets();
    console.log(response);
  } catch (e) {
    console.log(e);
    process.exit(-1);
  }
  process.exit();
})();
```

要使用 Twitter 授权 API，客户端应用程序必须在 Twitter 注册其应用程序。Twitter 提供了*消费者密钥*和*消费者密钥*。

使用`oauth`模块，创建了一个新的 OAuth2 对象，传入：

+   消费者密钥

+   消费者密钥

+   API 基本 URI（API URI 减去查询字符串）

+   空值表示 OAuth 使用默认的*/oauth/authorize*

+   访问令牌路径

+   空，因为我们不使用任何自定义标头

`oauth`模块使用这些数据创建一个 POST 请求到 Twitter，同时传递消费者密钥和密钥，还为请求提供了*范围*。Twitter 的文档提供了一个获取访问令牌的示例 POST 请求（为了可读性插入了换行符）：

```
POST /oauth2/token HTTP/1.1
Host: api.twitter.com
User-Agent: My Twitter App v1.0.23
Authorization: Basic eHZ6MWV2RlM0d0VFUFRHRUZQSEJvZzpMOHFxOVBaeVJn
                NmllS0dFS2hab2xHQzB2SldMdzhpRUo4OERSZHlPZw==
                Content-Type: application/x-www-form-urlencoded;charset=UTF-8
Content-Length: 29
Accept-Encoding: gzip

grant_type=client_credentials
```

响应包括访问令牌（再次为了可读性插入换行符）：

```
HTTP/1.1 200 OK
Status: 200 OK
Content-Type: application/json; charset=utf-8
...
Content-Encoding: gzip
Content-Length: 140

{"token_type":"bearer","access_token":"AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
%2FAAAAAAAAAAAAAAAAAAAA%3DAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"}
```

任何 API 请求都必须使用访问令牌。没有进一步的授权步骤，因此流程非常简单。此外，由于授权是在应用程序级别进行的，不需要个人的授权，使得对用户的干扰较少。

###### 注意

Twitter 提供了出色的文档。建议阅读[“仅应用程序身份验证概述”](https://oreil.ly/Mikyl)。

### 使用 OAuth 1.0 进行读/写授权

仅应用程序身份验证非常适合访问只读数据，但如果您想要访问用户的特定数据，甚至对其数据进行更改怎么办？那么您将需要完整的 OAuth 授权。在本节中，我们再次使用 Twitter 进行演示，因为它使用 OAuth 1.0 授权。在下一个示例中，我们将介绍 OAuth 2.0。

###### 注意

我称之为 OAuth 1.0，但 Twitter 的服务基于[OAuth Core 1.0 修订版 A](http://oauth.net/core/1.0a)。不过，说 OAuth 1.0 更简单。

OAuth 1.0 需要数字签名。生成此数字签名的步骤在 Twitter 的图 21-3 中有图形表示。

1.  收集 HTTP 方法和基本 URI，去除任何查询字符串。

1.  收集参数，包括消费者密钥、请求数据、随机数、签名方法等等。

1.  创建签名基础字符串，其中包含我们收集的数据，以一种精确的方式形成字符串，并适当编码。

1.  创建签名密钥，这是消费者密钥和 OAuth 令牌密钥的组合，再次以精确的方式组合起来。

1.  将签名基本字符串和签名密钥传递给 HMAC-SHA1 散列算法，返回一个需要进一步编码的二进制字符串。

![jsc3 2103](img/jsc3_2103.png)

###### Figure 21-3\. OAuth 1.0 授权流程

每次请求都必须遵循这个流程。幸运的是，我们有模块和库来完成所有这些令人昏昏欲睡的工作。我不知道你怎么看，但如果我必须这样做，我对将 Twitter 数据和服务整合到我的应用中的兴趣很快就会减退。

我们的朋友 `oauth` 提供了底层的 OAuth 1.0 支持，但这次我们不必直接编写代码。另一个模块 `node-twitter-api` 已经包装了所有 OAuth 的部分。我们只需要创建一个新的 `node-twitter-api` 对象，传入我们的消费者密钥和密钥，以及资源服务所需的回调/重定向 URL，作为授权过程的一部分。处理该 URL 中的 `request` 对象会为我们提供访问令牌和密钥，以便访问 API。每次我们发起请求时，都会传入访问令牌和密钥。

`twitter-node-api` 模块是 REST API 的一个薄包装：为了发起请求，我们从 API 推断功能是什么。如果我们有兴趣发布状态更新，REST API 的端点是：

```
https://api.twitter.com/1.1/statuses/update.json
```

`twitter-node-api` 对象实例函数是 `statuses()`，第一个参数是动词 `update`：

```
 twitter.statuses('update', {
        "status": "Hi from Shelley's Toy Box. (Ignore--developing Node app)"
        }, atoken, atokensec, function(err, data, response) {...});

twitter.statuses(
  'update',
  {
    status: 'Ignore learning OAuth with Node'
  },
  tokenValues.atoken,
  tokenValues.atokensec,
  (err, data) => { ... });
```

回调函数的参数包括任何可能的错误、请求的数据（如果有的话）以及原始响应。

完整的示例显示在 Example 21-1 中。它使用 Express 作为服务器，并为用户提供了一个基本的网页，然后使用另一个模块。

##### 示例 21-1\. 使用 OAuth 1.0 完全授权的 Twitter 应用

```
const express = require('express');
const TwitterAPI = require('node-twitter-api');

require('dotenv').config();

const port = process.env.PORT || '8080';

// keys and callback URL are configured in the Twitter Dev Center
const twitter = new TwitterAPI({
  consumerKey: process.env.TWITTER_CONSUMER_KEY,
  consumerSecret: process.env.TWITTER_CONSUMER_SECRET,
  callback: 'http://127.0.0.1:8080/oauth/callback'
});

// object for storing retrieved token values
const tokenValues = {};

// twitter OAuth API URL
const twitterAPI = 'https://api.twitter.com/oauth/authenticate';

// simple HTML template
const menu =
  '<a href="/post/status/">Say hello</a><br />' +
  '<a href="/get/account/">Account Settings<br />';

// Create a new Express application.
const app = express();

// request Twitter permissions when the / route is visited
app.get('/', (req, res) => {
  twitter.getRequestToken((error, requestToken, requestTokenSecret) => {
    if (error) {
      console.log(`Error getting OAuth request token : ${error}`);
      res.writeHead(200);
      res.end(`Error getting authorization${error}`);
    } else {
      tokenValues.token = requestToken;
      tokenValues.tokensec = requestTokenSecret;
      res.writeHead(302, {
        Location: `${twitterAPI}?oauth_token=${requestToken}`
      });
      res.end();
    }
  });
});

// callback url as specified in the Twitter Developer Center
app.get('/oauth/callback', (req, res) => {
  twitter.getAccessToken(
    tokenValues.token,
    tokenValues.tokensec,
    req.query.oauth_verifier,
    (err, accessToken, accessTokenSecret) => {
      res.writeHead(200);
      if (err) {
        res.end(`problems getting authorization with Twitter${err}`);
      } else {
        tokenValues.atoken = accessToken;
        tokenValues.atokensec = accessTokenSecret;
        res.end(menu);
      }
    }
  );
});

// post a status update from an authenticated and authorized users
app.get('/post/status/', (req, res) => {
  twitter.statuses(
    'update',
    {
      status: 'Ignore teaching OAuth with Node'
    },
    tokenValues.atoken,
    tokenValues.atokensec,
    (err, data) => {
      res.writeHead(200);
      if (err) {
        res.end(`problems posting ${JSON.stringify(err)}`);
      } else {
        res.end(`posting status: ${JSON.stringify(data)}<br />${menu}`);
      }
    }
  );
});

// get account details for an authenticated and authorized user
app.get('/get/account/', (req, res) => {
  twitter.account(
    'settings',
    {},
    tokenValues.atoken,
    tokenValues.atokensec,
    (err, data) => {
      res.writeHead(200);
      if (err) {
        res.end(`problems getting account ${JSON.stringify(err)}`);
      } else {
        res.end(`<p>${JSON.stringify(data)}</p>${menu}`);
      }
    }
  );
});

app.listen(port, () => console.log(`Listening on port ${port}!`));
```

应用中感兴趣的路由包括：

+   `/`: 触发重定向到 Twitter 进行授权的页面

+   `/auth`: 注册在应用程序中的回调或重定向 URL，并在请求中传递

+   `/post/status/`: 向 Twitter 账户发布状态

+   `/get/account/`: 获取个人账户信息

在每种情况下，都使用适当的 `node-twitter-api` 函数：

+   `/`: 使用 `getRequestToken()` 获取请求令牌和请求令牌密钥

+   `/auth/`: 获取 API 访问令牌和令牌密钥，将它们缓存到本地，显示菜单

+   `/post/status/`: `status()` 的第一个参数为 *update*，状态、访问令牌和密钥，以及回调函数

+   `/get/account/`: `account()` 的第一个参数为 *settings*，一个空对象，因为请求不需要数据，还有访问令牌、密钥和回调

弹出的 Twitter 授权页面显示在 Figure 21-4 中，显示本人账户信息的网页显示在 Figure 21-5 中。

###### 注意

虽然它已经不再积极维护，但您可以在其[GitHub 仓库页面](https://github.com/reneraab/node-twitter-api)上阅读有关 `node-twitter-api` 模块的更多信息。其他库更积极地维护并提供相同类型的功能，但我发现 `node-twitter-api` 提供了最简单的功能示例，用于演示目的。

![jsc3 2104](img/jsc3_2104.png)

###### 图 21-4\. 从菜谱应用程序重定向到 Twitter 授权页面

![jsc3 2105](img/jsc3_2105.png)

###### 图 21-5\. 在应用程序中显示 Twitter 用户账户数据

# 使用 Passport.js 进行 OAuth 2 用户认证

## 问题

您希望通过第三方服务在应用程序中对用户进行认证。

## 解决方案

使用 Passport.js 库与所选认证提供程序的适当策略配对。在本例中，我将使用 GitHub 策略，但对于包括 Facebook、Google 和 Twitter 在内的任何 OAuth 2 提供程序，工作流程都将是相同的。

您可以利用 GitHub 策略，首先访问 GitHub 的网站并[注册新的 OAuth 应用程序](https://github.com/settings/applications/new)。一旦注册了应用程序，就可以将 Passport.js OAuth 代码集成到应用程序中。

要开始，请配置 Passport 策略，其中包括 GitHub 提供的客户端 ID 和客户端密钥，以及您指定的回调 URL：

```
const express = require('express');
const passport = require('passport');
const { Strategy } = require('passport-github');

passport.use(
  new Strategy(
    {
      clientID: GITHUB_CLIENT_ID,
      clientSecret: GITHUB_CLIENT_SECRET,
      callbackURL: 'login/github/callback'
    },
    (accessToken, refreshToken, profile, cb) => {
      return cb(null, profile);
    }
  )
);
```

为了在 HTTP 请求之间恢复认证状态，Passport 需要对用户进行序列化和反序列化：

```
passport.serializeUser((user, cb) => {
  cb(null, user);
});

passport.deserializeUser((obj, cb) => {
  cb(null, obj);
});
```

要跨浏览器会话保留用户登录状态，请使用 `express-session` 中间件：

```
app.use(
  require('express-session')({
    secret: SESSION_SECRET,
    resave: true,
    saveUninitialized: true
  })
);

app.use(passport.session());
```

您可以然后使用 `passport.authenticate` 对请求进行认证：

```
app.use(passport.initialize());

app.get('/login/github', passport.authenticate('github'));

app.get(
  '/login/github/callback',
  passport.authenticate('github', { failureRedirect: '/login' }),
  (req, res) => {
    res.redirect('/');
  }
);
```

并引用来自请求的 `user` 对象：

```
app.get('/', (req, res) => {
  res.render('home', { user: req.user });
});
```

## 讨论

OAuth 是用于用户认证的开放标准。它允许我们通过第三方应用程序对用户进行认证。当允许用户轻松创建帐户并登录到您的应用程序时，以及用于从第三方来源认证使用数据时，这将非常有用。

OAuth 请求遵循特定的流程：

1.  您的应用程序向第三方服务发出授权请求。

1.  用户批准该请求。

1.  服务将用户重定向回您的应用程序，并携带授权码。

1.  应用程序使用授权码向第三方服务发出请求。

1.  服务响应访问令牌（及可选的刷新令牌）。

1.  应用程序使用访问令牌向服务发送请求。

1.  服务响应受保护的资源（在我们的案例中是用户账户信息）。

在 Express.js 应用程序中，结合 Passport.js 策略使用 Passport.js 简化了此流程。在本例中，我们将构建一个小型 Express 应用程序，该应用程序通过 GitHub 进行认证，并跨会话保持用户登录状态。

一旦我们已经向服务提供商注册了我们的应用程序，就可以通过安装适当的依赖项开始开发：

```
# install general application dependencies
npm install express pug dotenv
# install passport dependencies
npm install passport passport-github
# install persistent user session dependencies
npm install connect-ensure-login express-session
```

为了存储我们的 OAuth 客户端 ID、客户端密钥和会话密钥值，我们将使用一个 *.env* 文件。或者，你可以使用一个 JavaScript 文件（比如 *config.js* 文件）。重要的是不要将此文件提交到公共源代码控制中，并建议将其添加到你的 *.gitignore* 文件中。在 *.env* 中：

```
GITHUB_CLIENT_ID=<Your client ID>
GITHUB_CLIENT_SECRET=<Your client secret>
SESSION_SECRET=<A session secret - this can be any value you decide>
```

接下来，我们将在 *index.js* 中设置我们的 Express 应用程序与 Passport.js 集成。

```
const express = require('express');
const passport = require('passport');
const { Strategy } = require('passport-github');

require('dotenv').config();

const port = process.env.PORT || '3000';

// Configure the Passport strategy
passport.use(
  new Strategy(
    {
      clientID: process.env.GITHUB_CLIENT_ID,
      clientSecret: process.env.GITHUB_CLIENT_SECRET,
      callbackURL: `http://localhost:${port}/login/github/callback`
    },
    (accessToken, refreshToken, profile, cb) => {
      return cb(null, profile);
    }
  )
);

// Serialize and deserialize the user
passport.serializeUser((user, cb) => {
  cb(null, user);
});

passport.deserializeUser((obj, cb) => {
  cb(null, obj);
});

// create the Express application
const app = express();
app.set('views', `${__dirname}/views`);
app.set('view engine', 'pug');

// use the Express session middleware for preserving user session
app.use(
  require('express-session')({
    secret: process.env.SESSION_SECRET,
    resave: true,
    saveUninitialized: true
  })
);

// Initialize passport and restore the authentication state from the session
app.use(passport.initialize());
app.use(passport.session());

// listen on port 3000 or the PORT set as an environment variable
app.listen(port, () => console.log(`Listening on port ${port}!`));
```

然后，你可以构建你的视图模板，这些模板可以访问用户数据。

在 *views/home.pug* 中：

```
if !user
  p Welcome! Please
    a(href='/login/github') Login with GitHub
else
  h1 Hello #{user.username}!
  p View your
    a(href='/profile') profile
```

在 *views/login.pug* 中：

```
h1 Login
a(href='/login/github') Login with GitHub
```

在 *views/profile.pug* 中：

```
h1 Profile
ul
  li ID: #{user.id}
  li Name: #{user.username}
  if user.emails
    li Email: #{user.emails[0].value}
```

最后，在 *index.js* 文件中设置我们的路由：

```
app.get('/', (req, res) => {
  res.render('home', { user: req.user });
});

app.get('/login', (req, res) => {
  res.render('login');
});

app.get('/login/github', passport.authenticate('github'));

app.get(
  '/login/github/callback',
  passport.authenticate('github', { failureRedirect: '/login' }),
  (req, res) => {
    res.redirect('/');
  }
);

app.get(
  '/profile',
  require('connect-ensure-login').ensureLoggedIn(),
  (req, res) => {
    res.render('profile', { user: req.user });
  }
);
```

这个示例被设计得与 [Express 4.x Facebook 示例](https://github.com/passport/express-4.x-facebook-example) 高度匹配，提供了与 Express 和 Facebook 认证相关的代码详细文档。你可以查看数百种额外的 [Passport.js 策略](http://www.passportjs.org)。

# 提供格式化数据

## 问题

你希望返回格式化的数据（如 XML）给浏览器，而不是提供一个网页或发送纯文本。

## 解决方案

使用 Node 模块来帮助格式化数据。例如，如果你想返回 XML，可以使用一个模块来创建格式化的数据：

```
const builder = require('xmlbuilder');

const xml = builder
  .create('resources')
  .ele('resource')
  .ele('title', 'Ecma-262 Edition 10')
  .up()
  .ele('url', 'https://www.ecma-international.org/ecma-262/10.0/index.html')
  .up()
  .end({ pretty: true });
```

然后创建适当的头部信息，与数据一起返回到浏览器：

```
app.get('/', (req, res) => {
  res.setHeader('Content-Type', 'application/xml');
  res.end(xml.toString(), 'utf8');
});
```

## 讨论

Web 服务器经常提供静态或服务器端生成的资源，但同样频繁的是，返回给浏览器的是格式化的数据，在网页显示之前进行处理。

生成和返回格式化数据有两个关键要素。第一个是利用任何 Node 库来简化数据生成的过程，第二个是确保发送数据的头部信息与数据一致。

在解决方案中，使用了 `xmlbuilder` 模块来辅助我们创建正确的 XML。这不是 Node 默认安装的模块之一，所以我们需要使用 Node 包管理器 npm 来安装它：

```
npm install xmlbuilder
```

然后，创建一个新的 XML 文档，一个根元素，然后每个资源元素，正如解决方案中所示。的确，我们可以自己构建 XML 字符串，但这很麻烦。而且容易出现难以发现的错误。Node 最好的一点之一是有大量的模块可以做我们能想到的几乎任何事情。我们不仅不必亲自编写代码，大多数模块都经过了彻底测试并得到了积极维护。

一旦格式化数据准备好返回，创建对应的头部信息。在解决方案中，因为文档是 XML，所以在将数据返回为字符串之前，设置头部内容类型为 `application/xml`。

# 构建一个 RESTful API

## 问题

你希望使用 Node.js 构建一个 REST API。

## 解决方案

使用 Express 和 `app.get`、`app.post`、`app.put` 和 `app.delete` 方法：

```
const express = require('express');

const app = express();
const port = process.env.PORT || 3000;

app.get('/', (req, res) => {
  return res.send('Received a GET HTTP method');
});
app.post('/', (req, res) => {
  return res.send('Received a POST HTTP method');
});
app.put('/', (req, res) => {
  return res.send('Received a PUT HTTP method');
});
app.delete('/', (req, res) => {
  return res.send('Received a DELETE HTTP method');
});
app.listen(port, () => console.log(`Listening on port ${port}!`));
```

## 讨论

REST 代表“表述性状态转移”，是构建 API 的最常见的架构方法。REST 允许我们使用标准的 HTTP 方法（`GET`、`POST`、`PUT` 和 `DELETE`）通过 HTTP 与远程数据源交互。我们可以利用 Express 路由方法来接受这些请求。

在下面的示例中，我将创建几个作为 API 端点的路由。每个端点将响应 HTTP 请求：

`/todos`

将接受一个 `get` 请求以获取 todo 列表，以及一个 `post` 请求以创建新的 todo。

`/todos/:todoId`

将接受一个 `get` 请求，返回特定的 todo，以及一个 `put` 请求，允许用户更新 todo 的内容或完成状态，以及一个 `delete` 请求，删除特定的 todo。

有了这些定义的路由，我们可以开发一个能够适当响应这些请求的 REST API。

```
const express = require('express');

const port = process.env.PORT || 3000;
const app = express();
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// an array of data
let todos = [
  {
    id: '1',
    text: 'Order pizza',
    completed: true
  },
  {
    id: '2',
    text: 'Pick up pizza',
    completed: false
  }
];

// get the list of todos
app.get('/todos', (req, res) => {
  return res.send({ data: { todos } });
});

// get an individual todo
app.get('/todos/:todoId', (req, res) => {
  const foundTodo = todos.find(todo => todo.id === req.params.todoId);
  return res.send({ data: foundTodo });
});

// create a new todo
app.post('/todos', (req, res) => {
  const todo = {
    id: String(todos.length + 1),
    text: req.body.text,
    completed: false
  };

  todos.push(todo);
  return res.send({ data: todo });
});

// update a todo
app.put('/todos/:todoId', (req, res) => {
  const todoIndex = todos.findIndex(todo => todo.id === req.params.todoId);
  const todo = {
    id: req.params.todoId,
    text: req.body.text || todos[todoIndex].text,
    completed: req.body.completed || todos[todoIndex].completed
  };

  todos[todoIndex] = todo;
  return res.send({ data: todo });
});

// delete a todo
app.delete('/todos/:todoId', (req, res) => {
  const deletedTodo = todos.find(todo => todo.id === req.params.todoId);
  todos = todos.filter(todo => todo.id !== req.params.todoId);
  return res.send({ data: deletedTodo });
});

// listen on port 3000 or the PORT set as an environment variable
app.listen(port, () => console.log(`Listening on port ${port}!`));
```

从终端可以使用 `curl` 测试我们的响应：

```
# get the list of todos
curl http://localhost:3000/todos

# get an individual todo
curl http://localhost:3000/todos/1

# create a new todo
curl -X POST -H "Content-Type:application/json" /
  http://localhost:3000/todos -d '{"text":"Eat pizza"}'

# update a todo
curl -X PUT -H "Content-Type:application/json" /
  http://localhost:3000/todos/2 -d '{"completed": true }

# delete a todo
curl -X DELETE http://localhost:3000/todos/3
```

使用 `curl` 进行手动测试很快会变得乏味。对于 API 开发，您可能还想使用 REST 客户端 UI，例如 [Insomnia](https://insomnia.rest) 或 [Postman](https://postman.com)（参见 Figure 21-6）。

![Insomnia REST 客户端的截图](img/jsc3_2106.png)

###### 图 21-6\. Insomnia REST 客户端中的 GET 请求

在上面的示例中，我正在使用内存数据存储。构建 API 时，您很可能希望连接到数据库。为此，您可以使用诸如 [Sequelize](https://oreil.ly/NuXyR)（用于 SQL 数据库）、[Mongoose](https://oreil.ly/zP8Fr)（用于 MongoDB）或在线数据存储（如 [Firebase](https://oreil.ly/iZSFB)）等库。

# 构建 GraphQL API

## 问题

您想要构建一个 GraphQL API 服务器应用程序或向现有的 Express 应用程序添加 GraphQL 端点。

## 解决方案

使用 Apollo Server 包含 GraphQL 类型定义、GraphQL 解析器和 GraphQL Playground：

```
const express = require('express');
const { ApolloServer, gql } = require('apollo-server-express');

const port = process.env.PORT || 3000;
const app = express();

const typeDefs = gql`
 type Query {
 hello: String
 }
`;

const resolvers = {
  Query: {
    hello: () => 'Hello world!'
  }
};
const server = new ApolloServer({ typeDefs, resolvers });
server.applyMiddleware({ app, path: '/' });
app.listen({ port }, () => console.log(`Listening on port ${port}!`));
```

Apollo Server 提供了对 GraphQL Playground 的访问（参见 Figure 21-7），这使我们可以在开发过程中轻松与 API 进行交互（如果需要，在生产环境中也可以）。

![GraphQL Playground 的截图](img/jsc3_2107.png)

###### 图 21-7\. GraphQL Playground 中的 GraphQL 查询

GraphQL Playground 还根据您提供的类型定义自动生成 API 的文档（参见 Figure 21-8）。

![生成的 GraphQL 文档截图](img/jsc3_2108.png)

###### 图 21-8\. GraphQL Playground 中生成的文档

## 讨论

GraphQL 是用于 API 的开放源码查询语言。它的开发目标是提供数据的单一端点，允许应用程序请求所需的特定数据。[Apollo Server](https://oreil.ly/toPLM) 可以作为独立包使用，也可以集成为流行的 Node.js 服务器应用程序库的中间件，如 Express、Hapi、Fastify 和 Koa。

在 GraphQL 中，类型定义模式是我们数据和交互的书面表示。通过要求模式，GraphQL 强制执行 API 的严格计划。这是因为您的 API 只能返回在模式中定义的数据并执行其中定义的交互。GraphQL 模式的基本组件是对象类型。GraphQL 包含五种内置标量类型：

+   字符串：采用 UTF-8 字符编码的字符串

+   布尔值：一个真或假的值

+   整数：一个 32 位整数

+   浮点数：一个浮点数值

+   ID：一个唯一标识符

一旦模式编写完成，我们就为 API 提供一系列解析器。这些解析器是指定查询中应如何返回数据或在数据变异中进行更改的函数。

在先前的示例中，我们使用了 `apollo-server-express` 包，它应与 `express` 和 `gql` 包一起安装：

```
$ npm install express apollo-server-express gql
```

要创建 CRUD 应用程序，我们可以定义我们的 GraphQL 类型定义和适当的解析器。下面的示例模仿了 “构建 RESTful API” 中找到的示例：

```
const express = require('express');
const { ApolloServer, gql } = require('apollo-server-express');

const port = process.env.PORT || 3000;
const app = express();

// an array of data
let todos = [
  {
    id: '1',
    text: 'Order pizza',
    completed: true
  },
  {
    id: '2',
    text: 'Pick up pizza',
    completed: false
  }
];

// GraphQL Type Definitions
const typeDefs = gql`
 type Query {
 todos: [Todo!]!
 todo(id: ID!): Todo!
 }

 type Mutation {
 newTodo(text: String!): Todo!
 updateTodo(id: ID!, text: String, completed: Boolean): Todo!
 deleteTodo(id: ID!): Todo!
 }

 type Todo {
 id: ID!
 text: String!
 completed: Boolean
 }
`;

// GraphQL Resolvers
const resolvers = {
  Query: {
    todos: () => todos,
    todo: (parent, args) => {
      return todos.find(todo => todo.id === args.id);
    }
  },
  Mutation: {
    newTodo: (parent, args) => {
      const todo = {
        id: String(todos.length + 1),
        text: args.text,
        completed: false
      };

      todos.push(todo);
      return todo;
    },

    updateTodo: (parent, args) => {
      const todoIndex = todos.findIndex(todo => todo.id === args.id);
      const todo = {
        id: args.id,
        text: args.text || todos[todoIndex].text,
        completed: args.completed || todos[todoIndex].completed
      };

      todos[todoIndex] = todo;
      return todo;
    },
    deleteTodo: (parent, args) => {
      const deletedTodo = todos.find(todo => todo.id === args.id);
      todos = todos.filter(todo => todo.id !== args.id);
      return deletedTodo;
    }
  }
};

// Apollo + Express server setup
const server = new ApolloServer({ typeDefs, resolvers });
server.applyMiddleware({ app, path: '/' });
app.listen({ port }, () => console.log(`Listening on port ${port}!`));
```

在上面的示例中，我正在使用内存数据存储。构建 API 时，您很可能希望连接到数据库。为此，您可以使用像 Sequelize（用于 SQL 数据库）、Mongoose（用于 MongoDB）或在线数据存储（如 Firebase）这样的库。

定义的查询直接从 API 返回数据，而变异允许我们对数据执行更改，例如创建新项目、更新项目或删除项目。
