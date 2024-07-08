# 第九章：Cookies 和会话

在这一章中，您将学习如何使用 cookie 和会话，通过记住他们在页面之间以及甚至浏览器会话之间的偏好，为用户提供更好的体验。

HTTP 是一个 *无状态* 协议。这意味着当您在浏览器中加载页面然后导航到同一个网站上的另一个页面时，无论是服务器还是浏览器都没有内在的方式知道它是同一个浏览器访问同一个站点。另一种说法是，网络工作的方式是 *每个 HTTP 请求包含了服务器满足请求所需的所有信息* 。

然而，这是一个问题：如果故事就此结束，我们将永远无法登录任何东西。流媒体无法工作。网站将无法记住您在一个页面上的首选项。因此，需要一种方法在 HTTP 之上构建状态，这就是 cookie 和会话进入画面的地方。

遗憾的是，由于人们对 cookie 所做的恶劣之事，cookie 不幸地名声扫地。这很不幸，因为 cookie 对于“现代网络”的运行非常重要（尽管 HTML5 已经引入了一些新特性，比如本地存储，可以用于相同的目的）。

cookie 的想法很简单：服务器发送一些信息，浏览器在一定的时间内存储它。实际上，特定的信息取决于服务器。通常只是一个标识特定浏览器的唯一 ID 号码，以便保持状态的假象。

关于 cookie，有一些重要的事情需要知道：

用户能够看到 cookie，而不是保密的。

服务器发送给客户端的所有 cookie 都可以供客户端查看。你当然可以发送一些加密内容以保护其内容，但很少有这样的必要（至少如果你没有做一些恶劣的事情的话！）。*签名* cookie（稍后我们会讨论）可以使 cookie 的内容变得模糊，但这绝对不能从密码窥视者的眼里得到有效的加密保护。

用户可以删除或不允许 cookie

用户对 cookie 拥有完全的控制权，浏览器使得批量或单独删除 cookie 成为可能。除非您没安好心，否则用户没有真正的理由这样做，但在测试过程中这是有用的。用户也可以拒绝 cookie，这更为麻烦，因为只有最简单的 web 应用程序可以在没有 cookie 的情况下运行。

普通的 cookie 可以被篡改。

每当浏览器向您的服务器发出具有相关 cookie 的请求，并您盲目信任该 cookie 的内容时，您就会容易遭受攻击。例如，执行包含在 cookie 中的代码是极其愚蠢的。要确保 cookie 不被篡改，可以使用签名 cookie。

cookie 可以用于攻击。

近年来出现了一类名为 *跨站脚本攻击*（XSS）的攻击。XSS 攻击的一种技术涉及恶意 JavaScript 修改 cookies 的内容。这是不信任回到服务器的 cookie 内容的另一个原因。使用签名 cookies 有助于（篡改会在签名 cookie 中显示出来，无论是用户还是恶意 JavaScript 修改了它），还有一个设置指定只有服务器可以修改 cookies。这些 cookies 的用处可能有限，但确实更安全。

用户会注意到如果你滥用 cookies。

如果在用户计算机上设置了大量的 cookies 或存储了大量数据，会让用户感到恼火，这是你应该避免的事情。尽量将你对 cookies 的使用保持在最低限度。

建议优先选择会话而非 cookies。

大部分情况下，你可以使用 *会话（sessions）* 来维护状态，这通常是明智的选择。这样做更容易，你不必担心滥用用户的存储空间，而且更安全。当然，会话依赖于 cookies，但有了会话，Express 将为你做大部分繁重的工作。

###### 注意

Cookies 并非神奇：当服务器希望客户端存储一个 cookie 时，它发送一个名为 `Set-Cookie` 的头部，包含名称/值对；当客户端向服务器发送请求时，如果有 cookie，它会发送多个 `Cookie` 请求头，包含 cookie 的值。

# 外部化凭据

要使 cookies 安全，需要一个 *cookie 密钥*。Cookie 密钥是服务器知道并用于在发送到客户端之前加密安全 cookies 的字符串。这不是必须记住的密码，因此可以只是一个随机字符串。我通常使用 [受 xkcd 启发的随机密码生成器](http://bit.ly/2QcjuDb) 生成 cookie 密钥，或者仅仅使用一个随机数。

外部化第三方凭据，如 cookie 密钥、数据库密码和 API 令牌（Twitter、Facebook 等），是一种常见做法。这不仅简化了维护（通过轻松定位和更新凭据），还允许你将凭据文件从版本控制系统中排除。对于托管在 GitHub 或其他公共源代码控制库上的开源存储库尤为重要。

为此，我们将在一个 JSON 文件中外部化我们的凭据。创建一个名为 *.credentials.development.json* 的文件：

```
{
  "cookieSecret": "...your cookie secret goes here"
}
```

这将是我们开发工作的凭据文件。通过这种方式，你可以为生产、测试或其他环境拥有不同的凭据文件，这将非常方便。

我们将在凭据文件的顶部添加一个抽象层，以便在应用程序扩展时更容易管理我们的依赖关系。我们的版本将非常简单。创建一个名为 *config.js* 的文件：

```
const env = process.env.NODE_ENV || 'development'
const credentials = require(`./.credentials.${env}`)
module.exports = { credentials }
```

现在，为了确保我们不会意外地将凭据添加到我们的代码库中，请将`*.credentials.*`添加到您的`.gitignore`文件中。要将凭据导入到您的应用程序中，您只需要这样做：

```
const { credentials } = require('./config')
```

以后我们将使用同一文件存储其他凭据，但现在我们只需要我们的 cookie 密钥。

###### 注意

如果您按照伴随仓库的方式进行操作，则必须创建自己的凭据文件，因为它未包含在仓库中。

# Express 中的 Cookies

在应用程序中设置和访问 cookie 之前，您需要包含`cookie-parser`中间件。首先，使用`npm install cookie-parser`，然后（在伴随仓库中的*ch09/meadowlark.js*中）：

```
const cookieParser = require('cookie-parser')
app.use(cookieParser(credentials.cookieSecret))
```

完成此操作后，您可以在任何可以访问响应对象的地方设置 cookie 或已签名的 cookie：

```
res.cookie('monster', 'nom nom')
res.cookie('signed_monster', 'nom nom', { signed: true })
```

###### 注意

签名 cookie 优先于未签名 cookie。如果您将签名 cookie 命名为`signed_monster`，则不能同时拥有同名的未签名 cookie（它将返回`undefined`）。

要检索从客户端发送的 cookie 的值（如果有），只需访问请求对象的`cookie`或`signedCookie`属性：

```
const monster = req.cookies.monster
const signedMonster = req.signedCookies.signed_monster
```

###### 注意

您可以为 cookie 名称使用任何字符串。例如，我们可以使用`\'signed monster'`而不是`\'signed_monster'`，但然后我们将不得不使用括号表示法来检索 cookie：`req.signedCookies[\'signed monster']`。因此，建议使用不带特殊字符的 cookie 名称。

要删除 cookie，请使用`req.clearCookie`：

```
res.clearCookie('monster')
```

设置 cookie 时，可以指定以下选项：

`domain`

控制 cookie 关联的域；这允许您将 cookie 分配给特定的子域。请注意，您不能为与服务器运行的不同域设置 cookie；它将简单地无效。

`path`

控制此 cookie 适用的路径。请注意，路径后面隐含有通配符；如果使用路径`/*`（默认），它将适用于站点上的所有页面。如果使用路径`/foo*`，它将适用于路径`/foo*`、`/foo/bar*`等。

`maxAge`

指定客户端在删除 cookie 之前应保留该 cookie 的时间，单位为毫秒。如果您省略此项，则在关闭浏览器时将删除 cookie（您还可以使用`expires`选项指定到期日期，但语法很令人沮丧。建议使用`maxAge`）。

`secure`

指定此 cookie 仅在安全（HTTPS）连接上发送。

`httpOnly`

将此设置为`true`指定该 cookie 仅由服务器修改。也就是说，客户端 JavaScript 无法修改它。这有助于防止 XSS 攻击。

`signed`

将此设置为`true`将签名此 cookie，使其可在`res.signedCookies`中而不是`res.cookies`中使用。已篡改的签名 cookie 将被服务器拒绝，并且 cookie 值将重置为其原始值。

# 检查 Cookie

作为测试的一部分，您可能需要一种方法来检查系统上的 cookie。大多数浏览器都有查看单个 cookie 及其存储值的方法。在 Chrome 中，打开开发者工具，选择“应用程序”选项卡。在左侧的树状菜单中，您将看到“Cookie”。展开它，您将看到列出的当前访问站点。点击它，您将看到与该站点相关的所有 cookie。您还可以右键单击域名以清除所有 cookie，或右键单击单个 cookie 以具体删除它。

# 会话

*会话*实际上只是一种更方便的维护状态的方式。要实现会话，*必须*在客户端存储一些内容；否则，服务器将无法从一个请求识别客户端到下一个请求。通常的做法是使用包含唯一标识符的 cookie。然后服务器使用该标识符检索相应的会话信息。

Cookie 不是实现这一功能的唯一方式：在“cookie 恐慌”高峰期间（即 cookie 滥用时期），许多用户简单地禁用了 cookie，并且开发了其他维护状态的方法，比如将会话信息装饰在 URL 中。这些技术混乱、困难且效率低下，最好留在过去。HTML5 提供了另一种称为*本地存储*的会话选项，如果需要存储大量数据则优于 cookie。有关此选项的更多信息，请参见[MDN 文档中的 `Window.localStorage`](https://mzl.la/2CDrGo4)。

广义上讲，有两种实现会话的方式：在 cookie 中存储所有信息或者仅在 cookie 中存储唯一标识符，其他信息存储在服务器上。前者被称为*基于 cookie 的会话*，仅仅是相对于使用 cookie 更为便利的一种方式。然而，这仍意味着你添加到会话中的所有内容都将存储在客户端的浏览器中，这是我不推荐的一种做法。我只推荐在你知道将只存储少量信息、不介意用户访问这些信息以及信息不会随时间过多增长的情况下使用这种方式。如果你想采用这种方式，请参见[`cookie-session` 中间件](http://bit.ly/2qNv9h6)。

## 记忆存储

如果您更愿意将会话信息存储在服务器上，我建议您这样做，您必须有一个地方来存储它。入门级选项是内存会话。它们易于设置，但有一个巨大的缺点：当您重新启动服务器（在本书的过程中您将经常这样做！），您的会话信息会消失。更糟糕的是，如果您通过多个服务器进行扩展（参见第十二章），不同的服务器可能会每次服务一个请求；会话数据有时会存在，有时会不存在。这显然是无法接受的用户体验。然而，对于我们的开发和测试需求，它是足够的。我们将看到如何在第十三章中永久存储会话信息。

首先，安装`express-session`（`npm install express-session`）；然后，在链接 cookie 解析器之后，链接`express-session`（*ch09/meadowalrk.js*在伴随存储库中）：

```
const expressSession = require('express-session')
// make sure you've linked in cookie middleware before
// session middleware!
app.use(expressSession({
    resave: false,
    saveUninitialized: false,
    secret: credentials.cookieSecret,
}))
```

`express-session` 中间件接受带有以下选项的配置对象：

`resave`

即使请求未被修改，也强制将会话保存回存储中。通常情况下，将其设置为`false`更可取；有关更多信息，请参阅`express-session`文档。

`saveUninitialized`

将其设置为`true`会导致新的（未初始化的）会话保存到存储中，即使它们没有被修改。通常情况下，将其设置为`false`更可取，并且在需要在设置 cookie 之前获取用户许可时是必需的。有关更多信息，请参阅`express-session`文档。

`secret`

用于签名会话 ID cookie 的密钥（或键）。这可以是用于`cookie-parser`的相同密钥。

`key`

将存储唯一会话标识符的 cookie 的名称。默认为`connect.sid`。

`store`

一个会话存储的实例。默认为`MemoryStore`的实例，对于我们当前的目的来说是可以的。我们将看到如何在第十三章中使用数据库存储。

`cookie`

会话 cookie 的 cookie 设置（`path`，`domain`，`secure`等）。常规 cookie 默认值适用。

## 使用会话

设置了会话后，使用它们变得非常简单；只需使用请求对象的`session`变量的属性：

```
req.session.userName = 'Anonymous'
const colorScheme = req.session.colorScheme || 'dark'
```

注意，使用会话时，我们无需使用请求对象来检索值，并使用响应对象来设置值；所有操作都在请求对象上执行。（响应对象没有`session`属性。）要删除会话，您可以使用 JavaScript 的`delete`操作符：

```
req.session.userName = null       // this sets 'userName' to null,
                                  // but doesn't remove it

delete req.session.colorScheme    // this removes 'colorScheme'
```

# 使用会话实现闪存消息

*闪存*消息（不要与 Adobe Flash 混淆）只是一种为用户提供反馈的方式，不会干扰他们的导航。实施闪存消息的最简单方法是使用会话（也可以使用查询字符串，但除了这些会使 URL 更丑陋外，闪存消息还将包含在书签中，这可能不是您想要的）。现在让我们首先设置我们的 HTML。我们将使用 Bootstrap 的警告消息来显示我们的闪存消息，所以确保您已经链接了 Bootstrap（请参阅 Bootstrap 的[“入门”](http://bit.ly/36YxeYf)文档；您可以在主模板中链接 Bootstrap 的 CSS 和 JavaScript 文件——附录中有一个示例）。在您的模板文件中，通常是在站点标题直接下方的显著位置，添加以下内容：

```
{{#if flash}}
  <div class="alert alert-dismissible alert-{{flash.type}}">
    <button type="button" class="close"
      data-dismiss="alert" aria-hidden="true">&times;</button>
    <strong>{{flash.intro}}</strong> {{{flash.message}}}
  </div>
{{/if}}
```

请注意，我们使用三个大括号来表示`flash.message`；这将允许我们在消息中提供一些简单的 HTML（我们可能想要强调单词或包含超链接）。现在让我们添加一些中间件来在会话中添加`flash`对象到上下文中。一旦我们显示了一条闪存消息，我们希望将其从会话中删除，以便在下一个请求中不显示它。我们将创建一些中间件来检查会话，看看是否有闪存消息，如果有，将其传输到`res.locals`对象中，使其在视图中可用。我们将把我们的中间件放在一个名为*lib/middleware/flash.js*的文件中：

```
module.exports = (req, res, next) => {
  // if there's a flash message, transfer
  // it to the context, then clear it
  res.locals.flash = req.session.flash
  delete req.session.flash
  next()
})
```

而在我们的*meadowalrk.js*文件中，在任何视图路由之前，我们将链接到闪存消息中间件：

```
const flashMiddleware = require('./lib/middleware/flash')
app.use(flashMiddleware)
```

现在让我们看看如何实际使用闪存消息。想象一下，我们正在注册用户的通讯，我们希望在他们注册后将他们重定向到通讯存档。这是我们的表单处理程序可能看起来像这样：

```
// slightly modified version of the official W3C HTML5 email regex:
// https://html.spec.whatwg.org/multipage/forms.html#valid-e-mail-address
const VALID_EMAIL_REGEX = new RegExp('^[a-zA-Z0-9.!#$%&\'*+\/=?^_`{|}~-]+@' +
  'a-zA-Z0-9?' +
  '(?:\.a-zA-Z0-9?)+$')

app.post('/newsletter', function(req, res){
    const name = req.body.name || '', email = req.body.email || ''
    // input validation
    if(VALID_EMAIL_REGEX.test(email)) {
      req.session.flash = {
        type: 'danger',
        intro: 'Validation error!',
        message: 'The email address you entered was not valid.',
      }
      return res.redirect(303, '/newsletter')
    }
    // NewsletterSignup is an example of an object you might create; since
    // every implementation will vary, it is up to you to write these
    // project-specific interfaces. This simply shows how a typical
    // Express implementation might look in your project.
    new NewsletterSignup({ name, email }).save((err) => {
        if(err) {
          req.session.flash = {
            type: 'danger',
            intro: 'Database error!',
            message: 'There was a database error; please try again later.',
          }
          return res.redirect(303, '/newsletter/archive')
        }
        req.session.flash = {
          type: 'success',
          intro: 'Thank you!',
          message: 'You have now been signed up for the newsletter.',
        };
        return res.redirect(303, '/newsletter/archive')
    })
})
```

请注意，我们要仔细区分输入验证和数据库错误。请记住，即使我们在前端进行了输入验证（您应该这样做），您也应该在后端执行它，因为恶意用户可以绕过前端验证。

闪存消息是您网站中可以使用的一个很棒的机制，即使在某些领域中其他方法更合适（例如，闪存消息并不总是适合多表单“向导”或购物车结账流程）。在开发过程中，闪存消息也非常好用，因为它们是一种提供反馈的简单方法，即使稍后用不同的技术替换它们。在设置网站时，支持闪存消息是我做的第一件事情之一，本书的其余部分我们将继续使用这种技术。

###### 提示

由于闪存消息是在中间件中从会话传输到`res.locals.flash`，因此您必须执行重定向才能显示闪存消息。如果要在不重定向的情况下显示闪存消息，请设置`res.locals.flash`而不是`req.session.flash`。

###### 注意

本章的示例使用了浏览器表单提交和重定向，因为使用 sessions 来控制 UI 通常不适用于使用 Ajax 进行表单提交的应用程序。在这种情况下，您希望从表单处理程序返回的 JSON 中指示任何错误，并让前端修改 DOM 以动态显示错误消息。这并不意味着 sessions 在前端渲染应用程序中没有用处，但它们很少用于此目的。

# 何时使用 sessions

当您希望保存跨页面适用的用户偏好时，sessions 非常有用。最常见的用途是提供用户认证信息：您登录后，会话就会被创建。之后，每次重新加载页面时，您都不必再次登录。即使没有用户账户，sessions 也可以很有用。网站通常会记住您喜欢的排序方式或日期格式偏好，所有这些都无需您登录。

虽然我鼓励您更喜欢 sessions 而不是 cookies，但理解 cookies 的工作原理很重要（特别是因为它们使 sessions 能够工作）。这将帮助您诊断问题，并了解应用程序的安全性和隐私考虑。

# 结论

理解**cookies**和**sessions**有助于我们更好地理解 Web 应用在底层协议（HTTP）无状态的情况下如何维持状态的错觉。我们学习了处理 cookies 和 sessions 的技术，以控制用户的体验。

在我们逐步进行中编写中间件时，我们一直没有太多解释中间件的内容。在下一章中，我们将深入探讨中间件，并学习有关它的一切！
