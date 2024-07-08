# 第八章：表单处理

收集用户信息的通常方法是使用 HTML *表单*。无论您是让浏览器正常提交表单，使用 Ajax 还是使用复杂的前端控件，其基础机制通常仍然是 HTML 表单。在本章中，我们将讨论处理表单、表单验证和文件上传的不同方法。

# 将客户端数据发送到服务器

总体而言，发送客户端数据到服务器有两种选择：查询字符串和请求体。通常，如果你使用查询字符串，你是在进行`GET`请求；如果使用请求体，则是在进行`POST`请求。（HTTP 协议不会阻止你反过来做，但这没有意义：最好在这里坚持标准做法。）

一种常见的误解是`POST`是安全的，而`GET`不安全：实际上，如果您使用 HTTPS，两者都是安全的，如果不使用 HTTPS，则两者都不安全。如果不使用 HTTPS，入侵者可以轻松查看`POST`请求的正文数据，就像可以查看`GET`请求的查询字符串一样。但是，如果使用`GET`请求，用户将在查询字符串中看到所有输入的内容（包括隐藏字段），这样看起来很乱且不美观。此外，浏览器通常会对查询字符串的长度设置限制（对于请求正文长度没有此限制）。因此，我通常建议在表单提交时使用`POST`。

# HTML 表单

本书侧重于服务器端，但了解构建 HTML 表单的一些基础知识也很重要。这里有一个简单的例子：

```
<form action="/process" method="POST">
    <input type="hidden" name="hush" val="hidden, but not secret!">
    <div>
        <label for="fieldColor">Your favorite color: </label>
        <input type="text" id="fieldColor" name="color">
    </div>
    <div>
        <button type="submit">Submit</button>
    </div>
</form>
```

注意，在`<form>`标签中明确指定了方法为`POST`；如果不这样做，默认为`GET`。`action`属性指定了表单提交时将接收数据的 URL。如果省略此字段，表单将提交到加载表单的同一 URL。我建议您始终提供一个有效的`action`，即使您在使用 Ajax（这是为了防止数据丢失；更多信息请参见第二十二章）。

从服务器的角度来看，在`<input>`字段中，重要的属性是`name`属性：这是服务器识别字段的方式。重要的是要理解，`name`属性与`id`属性是不同的，`id`属性仅用于样式和前端功能（不会传递到服务器）。

注意隐藏字段：这些字段不会在用户的浏览器中显示。但是，不应将其用于秘密或敏感信息；只需查看页面源代码，隐藏字段就会被公开。

HTML 不限制您在同一页上拥有多个表单（这是一些早期服务器框架的不幸限制；ASP，我在看你）。我建议保持您的表单在逻辑上一致；一个表单应包含您希望一次提交的所有字段（可选/空字段是可以的），并且不应包含您不需要的字段。如果在页面上有两个不同的操作，应使用两个不同的表单。例如，一个用于网站搜索，另一个用于订阅电子邮件通讯。也可以使用一个大型表单，并根据用户点击的按钮决定采取什么操作，但这会很麻烦，通常对于残障人士不友好（因为辅助功能浏览器渲染表单的方式）。

在此示例中，当用户提交表单时，将调用 */process* URL，并将字段值传输到请求体中的服务器。

# 编码

当表单提交时（无论是通过浏览器还是通过 Ajax），都必须以某种方式对其进行编码。如果您不明确指定编码方式，默认为 `application/x-www-form-urlencoded`（这只是“URL 编码”的一个较长的媒体类型）。这是一种基本的、易于使用的编码方式，Express 默认支持。

如果需要上传文件，则情况变得更加复杂。没有简单的方法可以使用 URL 编码发送文件，因此必须使用 `multipart/form-data` 编码类型，这种类型不会被 Express 直接处理。

# 表单处理的不同方法

如果您不使用 Ajax，唯一的选项是通过浏览器提交表单，这将重新加载页面。但是，如何重新加载页面由您决定。在处理表单时有两件事需要考虑：处理表单的路径（操作）和发送给浏览器的响应。

如果您的表单使用 `method="POST"`（推荐使用），通常会使用相同的路径来显示表单和处理表单：这些可以区分，因为前者是 `GET` 请求，后者是 `POST` 请求。如果采用这种方法，可以在表单上省略 `action` 属性。

另一种选择是使用单独的路径来处理表单。例如，如果您的联系页面使用路径 */contact*，您可以使用路径 */process-contact* 来处理表单（通过指定 `action="/process-contact"`）。如果使用这种方法，您可以选择通过 `GET` 提交表单（我不推荐这样做；它会不必要地将您的表单字段暴露在 URL 上）。如果您有多个使用相同提交机制的 URL，则可能更喜欢使用单独的端点进行表单提交（例如，您可能在站点的多个页面上都有一个电子邮件注册框）。

无论您用于处理表单的路径是什么，都必须决定发送回浏览器的响应。以下是您的选择：

直接的 HTML 响应

处理表单后，您可以直接向浏览器发送 HTML（例如，一个视图）。这种方法会在用户尝试重新加载页面时产生警告，并且可能会干扰书签和返回按钮，因此不建议使用。

302 重定向

虽然这是一种常见的方法，但这是对 302（已找到）响应代码原始意义的误用。HTTP 1.1 添加了 303（查看其他）响应代码，这是更可取的。除非你有理由针对 1996 年之前制作的浏览器，否则应使用 303。

303 重定向

HTTP 1.1 中添加的 303（查看其他）响应代码是为了解决 302 重定向的误用。HTTP 规范明确指示浏览器在遵循 303 重定向时应使用 `GET` 请求，而不管原始方法如何。这是对表单提交请求做出响应的推荐方法。

由于建议您对表单提交做出 303 重定向响应，下一个问题是：“重定向指向哪里？”这个问题的答案取决于你。以下是最常见的方法：

重定向到专用成功/失败页面

该方法要求您为适当的成功或失败消息指定 URL。例如，如果用户注册了促销邮件，但出现了数据库错误，您可能希望重定向到 */error/database*。如果用户的电子邮件地址无效，您可以重定向到 */error/invalid-email*，如果一切顺利，您可以重定向到 */promo-email/thank-you*。该方法的优势之一是它对分析友好：您 */promo-email/thank-you* 页面的访问次数应该大致对应于注册促销电子邮件的人数。它也很容易实现。然而，它也有一些缺点。这确实意味着您必须为每种可能性分配 URL，这意味着需要设计页面，编写副本，并进行维护。另一个缺点是用户体验可能不够理想：用户喜欢被感谢，但后来他们必须返回到他们之前或想要去的地方。这是我们目前将要使用的方法：我们将在第九章中切换到使用闪存消息（不要与 Adobe Flash 混淆）。

重定向到原始位置并显示闪存消息

对于遍布网站的小型表单（例如电子邮件注册），最好的用户体验是不要中断用户的导航流程。也就是说，提供一种在不离开页面的情况下提交电子邮件地址的方式。当然，其中一种方法是使用 Ajax，但如果你不想使用 Ajax（或者你希望备用机制提供良好的用户体验），你可以重定向回用户最初所在的页面。实现这一点最简单的方法是在表单中使用一个隐藏字段，该字段填充当前 URL。由于你希望能够提供反馈，告知用户提交已收到，你可以使用闪存消息。

重定向到一个新的位置，并显示闪存消息。

大型表单通常有它们自己的页面，一旦提交表单，留在该页面是没有意义的。在这种情况下，你必须智能猜测用户可能希望去哪里，并相应地进行重定向。例如，如果你正在构建一个管理界面，并且有一个表单用于创建新的度假套餐，你可以合理地期望用户提交表单后想要进入显示所有度假套餐的管理页面。然而，你仍然应该使用闪存消息向用户反馈提交结果。

如果你正在使用 Ajax，我建议使用专用的 URL。虽然以一个前缀（例如 */ajax/enter*）开始 Ajax 处理程序是很诱人的，但我不推荐这种方法：它将实现细节附加到 URL 上。另外，正如我们马上会看到的，你的 Ajax 处理程序应该作为常规浏览器提交的容错机制。

# 使用 Express 处理表单

如果你使用`GET`来处理表单，你的字段将会在`req.query`对象中。例如，如果你有一个名为`email`的 HTML 输入字段，其值将作为`req.query.email`传递给处理程序。关于这种方法，其实没有太多需要说的，就是这么简单。

如果你使用`POST`（我推荐的方法），你需要链接中间件来解析 URL 编码的主体。首先，安装`body-parser`中间件（`npm install body-parser`）；然后，在伴随代码库中的 *ch08/meadowlark.js* 中链接它：

```
const bodyParser = require('body-parser')
app.use(bodyParser.urlencoded({ extended: true }))
```

一旦链接了`body-parser`中间件，你会发现`req.body`现在可以用了，这是所有表单字段的存放位置。请注意，`req.body`并不妨碍你使用查询字符串。我们继续在 Meadowlark Travel 中添加一个表单，让用户可以注册邮件列表。为了演示，我们将在 */views/newsletter-signup.handlebars* 中使用查询字符串、一个隐藏字段以及可见字段：

```
<h2>Sign up for our newsletter to receive news and specials!</h2>
<form class="form-horizontal" role="form"
    action="/newsletter-signup/process?form=newsletter" method="POST">
  <input type="hidden" name="_csrf" value="{{csrf}}">
  <div class="form-group">
    <label for="fieldName" class="col-sm-2 control-label">Name</label>
    <div class="col-sm-4">
      <input type="text" class="form-control"
      id="fieldName" name="name">
    </div>
  </div>
  <div class="form-group">
    <label for="fieldEmail" class="col-sm-2 control-label">Email</label>
    <div class="col-sm-4">
      <input type="email" class="form-control" required
          id="fieldEmail" name="email">
    </div>
  </div>
  <div class="form-group">
    <div class="col-sm-offset-2 col-sm-4">
      <button type="submit" class="btn btn-primary">Register</button>
    </div>
  </div>
</form>
```

请注意，我们使用 Bootstrap 样式，这将贯穿整本书。如果你对 Bootstrap 不熟悉，可以参考 [Bootstrap 文档](http://getbootstrap.com)。

我们已经在我们的 body 解析器中添加了链接，现在我们需要为我们的通讯录注册页面、处理函数和感谢页面添加处理程序（*ch08/lib/handlers.js*在伴随代码库中）：

```
exports.newsletterSignup = (req, res) => {
  // we will learn about CSRF later...for now, we just
  // provide a dummy value
  res.render('newsletter-signup', { csrf: 'CSRF token goes here' })
}
exports.newsletterSignupProcess = (req, res) => {
  console.log('Form (from querystring): ' + req.query.form)
  console.log('CSRF token (from hidden form field): ' + req.body._csrf)
  console.log('Name (from visible form field): ' + req.body.name)
  console.log('Email (from visible form field): ' + req.body.email)
  res.redirect(303, '/newsletter-signup/thank-you')
}
exports.newsletterSignupThankYou = (req, res) =>
  res.render('newsletter-signup-thank-you')
```

（如果还没有，请创建一个 *views/newsletter-signup-thank-you.handlebars* 文件。）

最后，我们将我们的处理程序链接到我们的应用程序中（*ch08/meadowlark.js*在伴随代码库中）：

```
app.get('/newsletter-signup', handlers.newsletterSignup)
app.post('/newsletter-signup/process', handlers.newsletterSignupProcess)
app.get('/newsletter-signup/thank-you', handlers.newsletterSignupThankYou)
```

就是这么简单。请注意，在我们的处理程序中，我们正在重定向到“感谢”视图。我们可以在这里渲染一个视图，但如果这样做，访问者浏览器中的 URL 字段将保持 */process*，这可能会令人困惑。发出重定向可以解决这个问题。

###### 注意

在这种情况下，使用 303（或 302）重定向而不是 301 重定向非常重要。301 重定向是“永久”的，这意味着您的浏览器可能会缓存重定向目标。如果您使用 301 重定向并尝试第二次提交表单，您的浏览器可能会完全跳过 `/process` 处理程序，直接转到 */thank-you*，因为它正确地认为重定向是永久的。另一方面，303 重定向告诉您的浏览器：“是的，您的请求有效，您可以在这里找到您的响应”，并且不缓存重定向目标。

大多数前端框架更倾向于使用 `fetch` API 发送 JSON 格式的表单数据，我们接下来将看一下这个。不过，默认情况下了解浏览器如何处理表单提交仍然很重要，因为你仍然会发现以这种方式实现的表单。

现在让我们关注使用 `fetch` 进行表单提交。

# 使用 Fetch 发送表单数据

使用 `fetch` API 发送 JSON 编码的表单数据是一种更现代的方法，可以更好地控制客户端/服务器通信，并减少页面刷新。

因为我们不再向服务器发起往返请求，所以不再需要担心重定向和多个用户 URL（我们仍然会为表单处理本身有一个单独的 URL），因此，我们将整个“通讯录注册体验”统一到一个称为 */newsletter* 的单一 URL 下。

让我们从前端代码开始。HTML 表单本身的内容无需更改（字段和布局都相同），但我们不需要指定 `action` 或 `method`，并且我们将在一个容器 `<div>` 元素中包装我们的表单，这样可以更容易地显示我们的“感谢”消息：

```
<div id="newsletterSignupFormContainer">
  <form class="form-horizontal role="form" id="newsletterSignupForm">
    <!-- the rest of the form contents are the same... -->
  </form>
</div>
```

然后我们将有一个脚本来拦截表单提交事件并取消它（使用 `Event#preventDefault`），这样我们就可以自己处理表单处理（*ch08/views/newsletter.handlebars*在伴随代码库中）：

```
<script>
  document.getElementById('newsletterSignupForm')
    .addEventListener('submit', evt => {
      evt.preventDefault()
      const form = evt.target
      const body = JSON.stringify({
        _csrf: form.elements._csrf.value,
        name: form.elements.name.value,
        email: form.elements.email.value,
      })
      const headers = { 'Content-Type': 'application/json' }
      const container =
        document.getElementById('newsletterSignupFormContainer')
      fetch('/api/newsletter-signup', { method: 'post', body, headers })
        .then(resp => {
          if(resp.status < 200 || resp.status >= 300)
            throw new Error(`Request failed with status ${resp.status}`)
          return resp.json()
        })
        .then(json => {
          container.innerHTML = '<b>Thank you for signing up!</b>'
        })
        .catch(err => {
          container.innerHTML = `<b>We're sorry, we had a problem ` +
            `signing you up. Please <a href="/newsletter">try again</a>`
        })
  })
</script>
```

现在在我们的服务器文件（*meadowlark.js*）中，请确保我们链接了可以解析 JSON 主体的中间件，然后我们指定我们的两个端点：

```
app.use(bodyParser.json())

//...

app.get('/newsletter', handlers.newsletter)
app.post('/api/newsletter-signup', handlers.api.newsletterSignup)
```

请注意，我们将我们的表单处理端点放在以 `api` 开头的 URL 上；这是一种区分用户（浏览器）端点和 API 端点的常用技术，后者旨在使用 `fetch` 访问。

现在我们将这些端点添加到我们的 *lib/handlers.js* 文件中：

```
exports.newsletter = (req, res) => {
  // we will learn about CSRF later...for now, we just
  // provide a dummy value
  res.render('newsletter', { csrf: 'CSRF token goes here' })
}
exports.api = {
  newsletterSignup: (req, res) => {
    console.log('CSRF token (from hidden form field): ' + req.body._csrf)
    console.log('Name (from visible form field): ' + req.body.name)
    console.log('Email (from visible form field): ' + req.body.email)
    res.send({ result: 'success' })
  },
}
```

我们可以在表单处理处理器中进行任何我们需要的处理；通常情况下，我们会将数据保存到数据库中。如果出现问题，我们会返回一个带有 `err` 属性的 JSON 对象（而不是 `result: *success*`）。

###### 提示

在这个例子中，我们假设所有的 Ajax 请求都希望得到 JSON 响应，但并不要求 Ajax 必须使用 JSON 进行通信（事实上，Ajax 曾经是一个首字母缩略词，其中的 “X” 代表 XML）。这种方式非常适合 JavaScript，因为 JavaScript 擅长处理 JSON 数据。如果你的 Ajax 端点是通用的或者你知道你的 Ajax 请求可能使用除 JSON 以外的其他内容，你应该根据 `Accepts` 头部返回合适的响应，你可以通过 `req.accepts` 辅助方法方便地访问它。如果你仅仅基于 `Accepts` 头部来响应请求，你可能还会想要查看 [`res.format`](http://bit.ly/33Syx92)，这是一个方便的方法，可以根据客户端的期望方便地响应适当的内容。如果这样做，你需要确保在使用 JavaScript 发送 Ajax 请求时设置 `dataType` 或 `accepts` 属性。

# 文件上传

我们已经提到文件上传会带来一系列复杂性问题。幸运的是，有一些很棒的项目可以帮助简化文件处理流程。

有四个流行且强大的选项用于处理多部分表单：**busboy**、**multiparty**、**formidable** 和 **multer**。我使用过这四种方式，它们都很不错，但我觉得 **multiparty** 维护得最好，因此我们将在这里使用它。

让我们为 Meadowlark Travel 的度假照片比赛创建一个文件上传表单 (*views/contest/vacation-photo.handlebars*)：

```
<h2>Vacation Photo Contest</h2>

<form class="form-horizontal" role="form"
    enctype="multipart/form-data" method="POST"
    action="/contest/vacation-photo/{{year}}/{{month}}">
  <input type="hidden" name="_csrf" value="{{csrf}}">
  <div class="form-group">
    <label for="fieldName" class="col-sm-2 control-label">Name</label>
    <div class="col-sm-4">
      <input type="text" class="form-control"
      id="fieldName" name="name">
    </div>
  </div>
  <div class="form-group">
    <label for="fieldEmail" class="col-sm-2 control-label">Email</label>
    <div class="col-sm-4">
      <input type="email" class="form-control" required
          id="fieldEmail" name="email">
    </div>
  </div>
  <div class="form-group">
    <label for="fieldPhoto" class="col-sm-2 control-label">Vacation photo</label>
    <div class="col-sm-4">
      <input type="file" class="form-control" required  accept="image/*"
          id="fieldPhoto" name="photo">
    </div>
  </div>
  <div class="form-group">
    <div class="col-sm-offset-2 col-sm-4">
      <button type="submit" class="btn btn-primary">Register</button>
    </div>
  </div>
</form>
```

注意，我们必须指定 `enctype="multipart/form-data"` 以启用文件上传。我们还可以通过使用 `accept` 属性（可选）来限制可以上传的文件类型。

现在我们需要创建路由处理程序，但我们面临一个困境。我们希望能够轻松地测试我们的路由处理程序，但多部分表单处理会增加复杂性（与我们在到达处理程序之前使用中间件处理其他类型的请求主体类似）。由于我们不希望自己测试多部分表单解码过程（我们可以假设 **multiparty** 已经彻底处理了这个过程），我们将通过将已处理的信息传递给处理程序来保持它们的“纯洁性”。由于我们还不知道这个过程的具体情况，我们将从 *meadowlark.js* 中的 Express 框架开始：

```
const multiparty = require('multiparty')

app.post('/contest/vacation-photo/:year/:month', (req, res) => {
  const form = new multiparty.Form()
  form.parse(req, (err, fields, files) => {
    if(err) return res.status(500).send({ error: err.message })
    handlers.vacationPhotoContestProcess(req, res, fields, files)
  })
})
```

我们使用 **multiparty** 的 `parse` 方法将请求数据解析成数据字段和文件。这个方法会将文件存储在服务器上的临时目录中，并将相关信息返回到传递的 `files` 数组中。

现在我们有了额外的信息可以传递给我们（可测试的）路由处理程序：字段（由于我们使用了不同的 body 解析器，所以它们不会像之前的例子一样在`req.body`中）以及收集到的文件的信息。现在我们知道它看起来是什么样子，我们可以编写我们的路由处理程序：

```
exports.vacationPhotoContestProcess = (req, res, fields, files) => {
  console.log('field data: ', fields)
  console.log('files: ', files)
  res.redirect(303, '/contest/vacation-photo-thank-you')
}
```

（年份和月份被指定为*路由参数*，你会在第十四章学到有关路由参数的知识。）请继续运行它并检查控制台日志。你会看到你的表单字段以你预期的方式传递过来：作为一个对象，拥有与字段名对应的属性。`files`对象包含更多数据，但相对来说比较简单。对于每个上传的文件，你会看到有关大小、上传路径（通常是临时目录中的随机名称）以及用户上传的文件的原始名称（只是文件名，而不是整个路径，出于安全和隐私考虑）的属性。

你对这个文件的处理现在取决于你：你可以将它存储在数据库中，复制到更永久的位置，或者上传到基于云的文件存储系统。请记住，如果你依赖于本地存储来保存文件，你的应用程序将无法很好地扩展，这对基于云的主机来说是一个不好的选择。我们将在第十三章再次讨论这个例子。

## 使用 Fetch 进行文件上传

令人高兴的是，使用`fetch`进行文件上传几乎与让浏览器处理一样。文件上传的辛苦工作实际上在于编码，而这些都在中间件中被处理。

考虑使用以下 JavaScript 使用`fetch`发送我们的表单内容：

```
<script>
  document.getElementById('vacationPhotoContestForm')
    .addEventListener('submit', evt => {
      evt.preventDefault()
      const body = new FormData(evt.target)
      const container =
        document.getElementById('vacationPhotoContestFormContainer')
      const url = '/api/vacation-photo-contest/{{year}}/{{month}}'
      fetch(url, { method: 'post', body })
        .then(resp => {
          if(resp.status < 200 || resp.status >= 300)
            throw new Error(`Request failed with status ${resp.status}`)
          return resp.json()
        })
        .then(json => {
          container.innerHTML = '<b>Thank you for submitting your photo!</b>'
        })
        .catch(err => {
          container.innerHTML = `<b>We're sorry, we had a problem processing ` +
            `your submission.  Please <a href="/newsletter">try again</a>`
        })
    })
</script>
```

这里需要注意的重要细节是我们将表单元素转换为[`FormData`](https://mzl.la/2CErVzb)对象，而`fetch`可以直接接受该对象作为请求体。就是这么简单！因为编码方式与我们让浏览器处理时完全相同，所以我们的处理程序几乎完全相同。我们只希望返回一个 JSON 响应，而不是重定向：

```
exports.api.vacationPhotoContest = (req, res, fields, files) => {
  console.log('field data: ', fields)
  console.log('files: ', files)
  res.send({ result: 'success' })
}
```

# 改善文件上传 UI

从 UI 角度来看，浏览器内置的文件上传控件可以说有些欠缺。你可能已经看到过拖放界面和样式更有吸引力的文件上传按钮。

好消息是，你在这里学到的技术几乎都适用于大多数流行的“花哨”文件上传组件。归根结底，大部分都是在同一个表单上传机制上面打了一个漂亮的外壳。

一些最受欢迎的文件上传前端如下所示：

+   [jQuery 文件上传](http://bit.ly/2Qbcd6I)

+   [Uppy](http://bit.ly/2rEFWeb)（这款产品的好处是支持许多热门的上传目标）

+   [带预览的文件上传](http://bit.ly/2X5fS7F)（这个可以让你完全控制；你可以访问文件对象数组，然后用它们构建`FormData`对象，再与`fetch`一起使用）

# 结论

在本章中，您学习了用于处理表单的各种技术。我们探讨了传统的浏览器处理表单的方式（让浏览器向服务器发出`POST`请求，包含表单内容并渲染来自服务器的响应，通常是重定向），以及越来越普遍的方法，即防止浏览器提交表单并使用`fetch`自行处理。

我们学习了表单的常见编码方式：

`application/x-www-form-urlencoded`

默认且易于使用的编码方式，通常与传统表单处理相关联

`application/json`

用于使用`fetch`发送的（非文件）数据的常见方式

`multipart/form-data`

用于传输文件时使用的编码方式

现在我们已经介绍了如何将用户数据发送到服务器，让我们把注意力转向*cookies*和*sessions*，它们也有助于同步服务器和客户端。
