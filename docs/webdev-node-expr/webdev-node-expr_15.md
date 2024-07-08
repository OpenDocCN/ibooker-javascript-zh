# 第十五章：REST API 和 JSON

虽然我们在 第八章 中看到了一些 REST API 的例子，但到目前为止，我们的范式主要是“在服务器端处理数据并向客户端发送格式化的 HTML”。越来越多的现代 Web 应用程序不再采用这种默认操作模式。相反，大多数现代 Web 应用程序都是单页面应用程序（SPA），它们在一个静态捆绑包中接收所有的 HTML 和 CSS，然后依赖于接收 JSON 格式的非结构化数据，并直接操作 HTML。同样地，通过提交表单来通信并将更改发送到服务器的重要性正逐渐让位于通过 HTTP 请求直接与 API 进行通信。

现在是时候把注意力转向使用 Express 提供 API 端点，而不是预先格式化的 HTML 了。在 第十六章 中，我们将展示如何使用我们的 API 动态渲染应用程序。

在本章中，我们将简化我们的应用程序，提供一个“即将推出”的 HTML 界面：我们将在 第十六章 中填充内容。而我们将专注于一个 API，该 API 将提供对我们的假期数据库的访问，并支持注册“淡季”侦听器的 API 功能。

*Web 服务* 是一个泛指，指任何可以通过 HTTP 访问的应用程序编程接口（API）。Web 服务的概念已经存在了相当长的时间，但直到最近，使其能够运行的技术仍然是古板、拜占庭式和过于复杂的。仍然有一些系统使用这些技术（如 SOAP 和 WSDL），并且有 Node 包可以帮助您与这些系统进行接口。不过，我们不会涵盖这些内容。相反，我们将专注于提供所谓的 RESTful 服务，这些服务要简单得多，易于接口化。

缩写 *REST* 表示 *表述性状态转移*，而在语法上令人困扰的 *RESTful* 则用作描述符来描述满足 REST 原则的 Web 服务。REST 的正式描述复杂且充满计算机科学的形式化，但基本原理是 REST 是客户端和服务器之间的无状态连接。REST 的正式定义还指定服务可以被缓存，并且服务可以被分层（也就是说，当你使用 REST API 时，可能会有其他 REST API 在其下层）。

从实际角度来看，HTTP 的约束实际上使得创建非 RESTful API 变得困难；例如，你必须特意去建立状态。因此，我们的工作大部分已经为我们完成。

# JSON 和 XML

提供 API 至关重要的是有一个共同的语言来交流。通信的一部分是我们必须使用 HTTP 方法与服务器通信。但过了这一点，我们可以自由选择任何数据语言。传统上，XML 一直是一个流行的选择，并且仍然是一个重要的标记语言。虽然 XML 并不特别复杂，但 Douglas Crockford 发现有更轻量级的空间，于是 JavaScript 对象表示法（JSON）诞生了。除了友好于 JavaScript（尽管它绝不是专有的；任何语言都可以轻松解析它），它还具有比 XML 更容易手工编写的优势。

大多数应用程序我更喜欢 JSON 而不是 XML：因为它有更好的 JavaScript 支持，并且它是一种更简单、更紧凑的格式。如果现有系统需要 XML 来与您的应用程序通信，我建议重点关注 JSON，只提供 XML。

# 我们的 API

我们将在开始实施之前计划我们的 API。除了列出假期并订阅“在季节内”的通知外，我们还将添加“删除假期”端点。由于这是一个公共 API，我们实际上不会删除假期。我们只是将其标记为“请求删除”，以便管理员可以审查。例如，您可以使用此不安全端点允许供应商请求从网站中删除假期，稍后可以由管理员审查。以下是我们的 API 端点。

`GET /api/vacations`

检索假期

`GET /api/vacation/:sku`

根据其 SKU 返回假期

`POST /api/vacation/:sku/notify-when-in-season`

以 `email` 作为查询字符串参数，并为指定的假期添加通知侦听器

`DELETE /api/vacation/:sku`

请求删除一个假期；以 `email`（请求删除的人）和 `notes` 作为查询字符串参数

```
[NOTE]
```

##### 示例 15-1。

有许多可用的 HTTP 动词。 `GET` 和 `POST` 是最常见的，接着是 `DELETE` 和 `PUT`。已经成为一种标准使用 `POST` 来 *创建* 某物，而 `PUT` 用于 *更新*（或修改）某物。这些词的英文含义并不以任何方式支持此区分，因此您可能希望考虑使用路径来区分这两个操作，以避免混淆。如果您想了解更多关于 HTTP 动词的信息，我建议从这篇[Tamas Piros 文章](http://bit.ly/32L4QWt)开始。

我们可以以许多方式描述我们的 API。在这里，我们选择使用 HTTP 方法和路径的组合来区分我们的 API 调用，以及通过查询字符串和主体参数的混合来传递数据。作为另一种选择，我们可以使用不同的路径（例如 */api/vacations/delete*）来使用相同的方法。^(1) 我们还可以以一种一致的方式传递数据。例如，我们可能选择在 URL 中传递检索参数所需的所有信息，而不是使用查询字符串：`DEL /api/vacation/:id/:email/:notes`。为了避免过长的 URL，我建议使用请求主体来传递大块数据（例如删除请求的备注）。

###### 提示

有一个广受欢迎且备受尊重的 JSON API 的约定，创意地命名为 JSON:API。对我来说，它有点冗长和重复，但我也相信，一个不完美的标准总比没有标准好。虽然本书不使用 JSON:API，但你将学到采纳 JSON:API 所制定的规范所需的一切。详见 [JSON:API 主页](https://jsonapi.org) 获取更多信息。

# API 错误报告

HTTP API 中的错误报告通常通过 HTTP 状态码实现。如果请求返回 200（OK），客户端知道请求成功。如果请求返回 500（内部服务器错误），请求失败。然而，在大多数应用程序中，并非所有事物都能（或应该）粗略地归类为“成功”或“失败”。例如，如果您按 ID 请求某些内容，但该 ID 不存在，这并不代表服务器错误。客户端请求了不存在的内容。通常情况下，错误可以分为以下几类：

严重错误

导致服务器处于不稳定或未知状态的错误。通常情况下，这是未处理异常的结果。从灾难性错误中安全恢复的唯一方法是重新启动服务器。理想情况下，任何待处理请求都应该收到 500 响应代码，但如果故障足够严重，服务器可能无法响应，请求将超时。

可恢复的服务器错误

可恢复错误不需要服务器重新启动或任何其他英勇行动。该错误是由服务器上的意外错误条件导致的（例如，数据库连接不可用）。问题可能是暂时的或永久的。在这种情况下，500 响应代码是合适的。

客户端错误

客户端错误是客户端犯的错误，通常是缺少或无效的参数。使用 500 响应代码是不合适的。毕竟，服务器并未发生故障。一切正常工作，只是客户端没有正确使用 API。在这里您有几个选择：可以用状态码 200 响应，并在响应体中描述错误，或者您还可以尝试使用适当的 HTTP 状态码描述错误。我建议采用后一种方法。在这种情况下，最有用的响应代码是 404（未找到）、400（错误请求）和 401（未授权）。此外，响应体应包含有关错误具体信息的解释。如果您想做得更多，错误消息甚至可以包含指向文档的链接。请注意，如果用户请求一系列内容但没有返回任何内容，这并不是错误条件。直接返回空列表是合适的。

在我们的应用程序中，我们将结合使用 HTTP 响应代码和响应体中的错误消息。

# 跨源资源共享

如果您发布一个 API，您可能希望将 API 提供给其他人使用。这将导致*跨站点 HTTP 请求*。跨站点 HTTP 请求一直是许多攻击的对象，因此受到*同源策略*的限制，该策略限制了脚本可以加载的位置。具体而言，协议、域名和端口必须匹配。这使得您的 API 无法被其他站点使用，这就是*跨源资源共享*（CORS）的用武之地。CORS 允许您逐案例解除此限制，甚至允许您列出特定允许访问脚本的域。CORS 通过`Access-Control-Allow-Origin`头部来实现。在 Express 应用程序中实现它的最简单方法是使用`cors`包（`npm install cors`）。要为您的应用程序启用 CORS，请使用以下方法：

```
const cors = require('cors')

app.use(cors())
```

因为同源 API 有其存在的理由（防止攻击），我建议仅在必要时应用 CORS。在我们的情况下，我们想要公开我们的整个 API（但只有 API），因此我们将 CORS 限制在以*/api*开头的路径上：

```
const cors = require('cors')

app.use('/api', cors())
```

有关更高级 CORS 用法的信息，请参阅[包文档](https://github.com/expressjs/cors)。

# 我们的测试

如果我们使用除了`GET`之外的 HTTP 动词，测试 API 可能会很麻烦，因为浏览器只知道如何发出`GET`请求（和表单的`POST`请求）。有办法解决这个问题，比如优秀的应用程序[Postman](https://www.getpostman.com)。但是，无论您是否使用这样的实用程序，编写自动化测试都是一个好习惯。在为我们的 API 编写测试之前，我们需要一种实际*调用*REST API 的方法。为此，我们将使用一个名为`node-fetch`的 Node 包，它复制了浏览器的*fetch* API：

```
npm install --save-dev node-fetch@2.6.0
```

我们将要实现的 API 调用的测试放在*tests/api/api.test.js*（伴随库中的*ch15/test/api/api.test.js*）中：

```
const fetch = require('node-fetch')

const baseUrl = 'http://localhost:3000'

const _fetch = async (method, path, body) => {
  body = typeof body === 'string' ? body : JSON.stringify(body)
  const headers = { 'Content-Type': 'application/json' }
  const res = await fetch(baseUrl + path, { method, body, headers })
  if(res.status < 200 || res.status > 299)
    throw new Error(`API returned status ${res.status}`)
  return res.json()
}

describe('API tests', () => {

  test('GET /api/vacations', async () => {
    const vacations = await _fetch('get', '/api/vacations')
    expect(vacations.length).not.toBe(0)
    const vacation0 = vacations[0]
    expect(vacation0.name).toMatch(/\w/)
    expect(typeof vacation0.price).toBe('number')
  })

  test('GET /api/vacation/:sku', async() => {
    const vacations = await _fetch('get', '/api/vacations')
    expect(vacations.length).not.toBe(0)
    const vacation0 = vacations[0]
    const vacation = await _fetch('get', '/api/vacation/' + vacation0.sku)
    expect(vacation.name).toBe(vacation0.name)
  })

  test('POST /api/vacation/:sku/notify-when-in-season', async() => {
    const vacations = await _fetch('get', '/api/vacations')
    expect(vacations.length).not.toBe(0)
    const vacation0 = vacations[0]
    // at this moment, all we can do is make sure the HTTP request is successful
    await _fetch('post', `/api/vacation/${vacation0.sku}/notify-when-in-season`,
      { email: 'test@meadowlarktravel.com' })
  })

  test('DELETE /api/vacation/:id', async() => {
    const vacations = await _fetch('get', '/api/vacations')
    expect(vacations.length).not.toBe(0)
    const vacation0 = vacations[0]
    // at this moment, all we can do is make sure the HTTP request is successful
    await _fetch('delete', `/api/vacation/${vacation0.sku}`)
  })

})
```

我们的测试套件以一个辅助函数`_fetch`开始，该函数处理一些常见的清理工作。如果尚未对其进行 JSON 编码，它将对请求体进行编码，添加适当的标头，并在响应状态码不在 200 范围内时抛出适当的错误。

对于我们的每个 API 端点，我们只有一个测试。我并不是在暗示这些测试是健壮或完整的；即使对于这个简单的 API，我们也可以（而且应该）为每个端点编写几个测试。我们这里的做法更像是一个起点，用来说明测试 API 的技术。

这些测试有几个重要的特征值得一提。其中之一是我们依赖于 API 已经启动并运行在 3000 端口上。一个更健壮的测试套件会找到一个空闲端口，在设置中启动 API，并在所有测试运行完成后停止它。其次，这个测试依赖于我们 API 中已经存在的数据。例如，第一个测试期望至少存在一个假期，并且该假期具有名称和价格。在真实应用中，您可能无法做出这些假设（例如，可能从无数据开始，并且可能希望测试允许缺少数据）。同样，一个更健壮的测试框架会有一种方法来设置和重置 API 中的初始数据，以便每次都从已知状态开始。例如，您可以编写脚本设置和填充测试数据库，将 API 连接到它，并在每次测试运行时将其清理掉。正如我们在第五章中所看到的，测试是一个广泛而复杂的主题，我们只能在这里探索其表面。

第一个测试涵盖了我们的`GET /api/vacations`端点。它获取所有的假期，验证至少有一个假期存在，并检查第一个假期是否具有名称和价格。我们还可以考虑测试其他数据属性。我将其留给读者来思考哪些属性是最重要的测试项目。

第二个测试涵盖了我们的`GET /api/vacation/:sku`端点。由于我们没有一致的测试数据，我们首先获取所有假期，从第一个假期中获取 SKU，以便测试这个端点。

我们最后两个测试涵盖了我们的`POST /api/vacation/:sku/notify-when-in-season`和`DELETE /api/vacation/:sku`端点。不幸的是，根据我们当前的 API 和测试框架，我们几乎无法验证这些端点是否按预期工作，所以我们默认调用它们，并信任 API 在不返回错误时执行正确的操作。如果我们想要使这些测试更加健壮，我们将不得不添加允许验证操作的端点（例如，确定特定假期是否已注册给定电子邮件的端点），或者以某种方式让测试可以“后门”访问我们的数据库。

如果现在运行测试，它们将超时并失败...因为我们尚未实施我们的 API 甚至启动我们的服务器。所以让我们开始吧！

# 使用 Express 提供 API

Express 非常适合提供 API。有许多 npm 模块可用，提供有用的功能（例如`connect-rest`和`json-api`），但我发现 Express 在开箱即用时就足够好，我们将坚持使用纯 Express 实现。

我们将从在*lib/handlers.js*中创建处理程序开始（我们可以创建一个单独的文件，比如*lib/api.js*，但现在让我们保持简单）：

```
exports.getVacationsApi = async (req, res) => {
  const vacations = await db.getVacations({ available: true })
  res.json(vacations)
}

exports.getVacationBySkuApi = async (req, res) => {
  const vacation = await db.getVacationBySku(req.params.sku)
  res.json(vacation)
}

exports.addVacationInSeasonListenerApi = async (req, res) => {
  await db.addVacationInSeasonListener(req.params.sku, req.body.email)
  res.json({ message: 'success' })
}

exports.requestDeleteVacationApi = async (req, res) => {
  const { email, notes } = req.body
  res.status(500).json({ message: 'not yet implemented' })
}
```

然后我们在*meadowlark.js*中连接 API：

```
app.get('/api/vacations', handlers.getVacationsApi)
app.get('/api/vacation/:sku', handlers.getVacationBySkuApi)
app.post('/api/vacation/:sku/notify-when-in-season',
  handlers.addVacationInSeasonListenerApi)
app.delete('/api/vacation/:sku', handlers.requestDeleteVacationApi)
```

现在应该没有什么特别令人惊讶的地方了。请注意，我们正在使用我们的数据库抽象层，因此使用我们的 MongoDB 实现或我们的 PostgreSQL 实现都没有关系（尽管您会发现根据实现的不同有些微不重要的额外字段，如果需要，我们可以删除它们）。

我将`requestDeleteVacationsApi`留给读者练习，主要是因为这种功能可以通过许多不同的方式实现。最简单的方法是只需修改我们的假期架构，添加“删除请求”字段，并在调用 API 时更新带有电子邮件和备注。更复杂的方法是拥有一个单独的表，比如一个审查队列，单独记录删除请求，引用相关的假期，这将更适合管理员使用。

假设您在第五章中正确设置了 Jest，您应该只需运行`npm test`，API 测试将被拾取（Jest 将查找以`.test.js`结尾的任何文件）。您会看到我们有三个通过的测试和一个失败的测试：未完成的`DELETE /api/vacation/:sku`。

# 结论

我希望这一章让你想问，“就这样？”到这一点上，你可能意识到 Express 的主要功能是响应 HTTP 请求。请求是什么以及它们如何响应完全取决于您。它们需要响应 HTML 吗？CSS？纯文本？JSON？使用 Express 都很容易实现。您甚至可以响应二进制文件类型。例如，动态构建并返回图像并不困难。从这个意义上说，API 只是 Express 可以响应的众多方式之一。

在下一章中，我们将通过构建单页应用程序来利用此 API，并以不同的方式复制我们在之前章节中所做的工作。

^(1) 如果您的客户端无法使用不同的 HTTP 方法，请参阅[此模块](http://bit.ly/2O7nr9E)，该模块允许您“伪造”不同的 HTTP 方法。
