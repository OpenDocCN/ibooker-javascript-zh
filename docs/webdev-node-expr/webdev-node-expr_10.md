# 第十章：中间件

到目前为止，我们已经接触了一些中间件：我们使用了现有的中间件（`body-parser`、`cookie-parser`、`static` 和 `express-session` 等），甚至编写了一些自己的中间件（用于向模板上下文添加天气数据、配置闪存消息以及我们的 404 处理程序）。但是，中间件到底是什么？

从概念上讲，*中间件*是一种封装功能的方式——具体来说，是对应用程序的 HTTP 请求操作的功能。实际上，中间件只是一个接受三个参数的函数：一个请求对象，一个响应对象和一个 `next()` 函数，后者将在稍后解释。（还有一种形式接受四个参数，用于错误处理，将在本章末尾介绍。）

中间件是在所谓的*管道*中执行的。你可以想象一个物理管道，里面装着水。水从一端泵入，然后在水到达目的地之前会经过仪表和阀门。这个类比的重要之处在于*顺序很重要*；如果你在阀门前放置压力计，与在阀门后放置压力计效果不同。同样地，如果你有一个向水中注入东西的阀门，那么从该阀门“下游”的所有内容都将包含添加的成分。在 Express 应用中，你可以通过调用 `app.use` 将中间件插入到管道中。

在 Express 4.0 之前，通过显式链接路由器来复杂化管道。根据你在何处链接路由器，当你混合中间件和路由处理程序时，路由可能会链接进出顺序不一致，使得管道序列不太清晰。在 Express 4.0 中，中间件和路由处理程序按照链接的顺序调用，使得顺序更加清晰。

通常的做法是在你的管道中将最后一个中间件设置为一个捕获所有请求的处理程序，即任何不匹配其他路由的请求。这个中间件通常返回状态码 404（未找到）。

那么请求如何在管道中“终止”呢？这就是传递给每个中间件的 `next` 函数所做的事情：如果你*不*调用 `next()`，请求将在那个中间件中终止。

# 中间件原则

灵活思考中间件和路由处理程序的方式是理解 Express 工作原理的关键。以下是你应该记住的几点：

+   路由处理程序（`app.get`、`app.post` 等——通常统称为 `app.METHOD`）可以被视为仅处理特定 HTTP 动词（`GET`、`POST` 等）的中间件。相反，中间件可以被视为处理所有 HTTP 动词的路由处理程序（这基本上等同于 `app.all`，处理任何 HTTP 动词；对于像 `PURGE` 这样的特殊动词，有一些细微差别，但对于常见动词来说效果相同）。

+   路由处理程序需要作为第一个参数的路径。如果你希望该路径匹配任何路由，请简单地使用`\*`。中间件也可以接受路径作为其第一个参数，但这是可选的（如果省略，则将匹配任何路径，就像你指定了`*`一样）。

+   路由处理程序和中间件接受一个回调函数，该函数接受两个、三个或四个参数（技术上，也可以有零个或一个参数，但这些形式没有合理的用途）。如果有两个或三个参数，第一个参数是请求和响应对象，第三个参数是`next`函数。如果有四个参数，它就成为*错误处理*中间件，第一个参数是错误对象，后面是请求、响应和下一个对象。

+   如果你没有调用`next()`，管道将被终止，不会再处理更多的路由处理程序或中间件。如果你没有调用`next()`，应向客户端发送响应（`res.send`、`res.json`、`res.render` 等）；如果不这样做，客户端将挂起并最终超时。

+   如果你调用`next()`，向客户端发送响应通常是不可取的。如果这样做，后续的中间件或路由处理程序将被执行，但它们发送的任何客户端响应将被忽略。

# 中间件示例

如果你想看看它的实际效果，让我们试试一些非常简单的中间件（*ch10/00-simple-middleware.js* 在伴随库中）：

```
app.use((req, res, next) => {
  console.log(`processing request for ${req.url}....`)
  next()
})

app.use((req, res, next) => {
  console.log('terminating request')
  res.send('thanks for playing!')
  // note that we do NOT call next() here...this terminates the request
})

app.use((req, res, next) => {
  console.log(`whoops, i'll never get called!`)
})
```

这里有三个中间件示例。第一个只是将消息记录到控制台，然后通过调用`next()`将请求传递给管道中的下一个中间件。然后，下一个中间件实际处理请求。注意，如果这里省略了`res.send`，将不会向客户端返回任何响应。最终，客户端会超时。最后一个中间件永远不会执行，因为在前一个中间件中终止了所有请求。

现在让我们考虑一个更复杂的完整示例（*ch10/01-routing-example.js* 在伴随库中）：

```
const express = require('express')
const app = express()

app.use((req, res, next) => {
  console.log('\n\nALLWAYS')
  next()
})

app.get('/a', (req, res) => {
  console.log('/a: route terminated')
  res.send('a')
})
app.get('/a', (req, res) => {
  console.log('/a: never called');
})
app.get('/b', (req, res, next) => {
  console.log('/b: route not terminated')
  next()
})
app.use((req, res, next) => {
  console.log('SOMETIMES')
  next()
})
app.get('/b', (req, res, next) => {
  console.log('/b (part 2): error thrown' )
  throw new Error('b failed')
})
app.use('/b', (err, req, res, next) => {
  console.log('/b error detected and passed on')
  next(err)
})
app.get('/c', (err, req) => {
  console.log('/c: error thrown')
  throw new Error('c failed')
})
app.use('/c', (err, req, res, next) => {
  console.log('/c: error detected but not passed on')
  next()
})

app.use((err, req, res, next) => {
  console.log('unhandled error detected: ' + err.message)
  res.send('500 - server error')
})

app.use((req, res) => {
  console.log('route not handled')
  res.send('404 - not found')
})

const port = process.env.PORT || 3000
app.listen(port, () => console.log( `Express started on http://localhost:${port}` +
  '; press Ctrl-C to terminate.'))
```

在尝试此示例之前，请想象一下结果会是什么。有哪些不同的路由？客户端会看到什么？控制台上会打印什么？如果你能正确回答所有这些问题，你就掌握了 Express 中的路由！特别注意请求到 */b* 和 */c* 的区别；在两种情况下都有错误，但一个会导致 404，另一个会导致 500。

注意中间件*必须*是一个函数。请记住，在 JavaScript 中，从函数中返回一个函数非常容易（也很常见）。例如，你会注意到`express.static`是一个函数，但我们实际上调用它，所以它必须返回另一个函数。考虑以下示例：

```
app.use(express.static)         // this will NOT work as expected

console.log(express.static())   // will log "function", indicating
                                // that express.static is a function
                                // that itself returns a function
```

还要注意，一个模块可以导出一个函数，该函数可以直接用作中间件。例如，这里有一个名为 *lib/tourRequiresWaiver.js* 的模块（Meadowlark Travel 的攀岩套餐需要一份免责声明）：

```
module.exports = (req,res,next) => {
  const { cart } = req.session
  if(!cart) return next()
  if(cart.items.some(item => item.product.requiresWaiver)) {
    cart.warnings.push('One or more of your selected ' +
      'tours requires a waiver.')
  }
  next()
}
```

我们可以像这样链接这个中间件（在伴随代码库中的 *ch10/02-item-waiver.example.js* 中）：

```
const requiresWaiver = require('./lib/tourRequiresWaiver')
app.use(requiresWaiver)
```

更常见的做法是，你会导出一个包含中间件属性的对象。例如，让我们把所有购物车验证代码放在 *lib/cartValidation.js* 中：

```
module.exports = {

  resetValidation(req, res, next) {
    const { cart } = req.session
    if(cart) cart.warnings = cart.errors = []
    next()
  },

  checkWaivers(req, res, next) {
    const { cart } = req.session
    if(!cart) return next()
    if(cart.items.some(item => item.product.requiresWaiver)) {
      cart.warnings.push('One or more of your selected ' +
        'tours requires a waiver.')
    }
    next()
  },

  checkGuestCounts(req, res, next) {
    const { cart } = req.session
    if(!cart) return next()
    if(cart.items.some(item => item.guests > item.product.maxGuests )) {
      cart.errors.push('One or more of your selected tours ' +
        'cannot accommodate the number of guests you ' +
        'have selected.')
    }
    next()
  },

}
```

然后你可以像这样链接中间件（在伴随代码库中的 *ch10/03-more-cart-validation.js* 中）：

```
const cartValidation = require('./lib/cartValidation')

app.use(cartValidation.resetValidation)
app.use(cartValidation.checkWaivers)
app.use(cartValidation.checkGuestCounts)
```

###### 注意

在前面的例子中，我们有一个中间件在语句 `return next()` 中提前终止。Express 不期望中间件返回一个值（也不会处理任何返回值），因此这只是写 `next(); return` 的一种简化方式。

# 常见中间件

虽然 npm 上有成千上万的中间件项目，但其中只有少数几个是常见且基础的，至少在每个非平凡的 Express 项目中都能找到一些。其中一些中间件曾经如此常见，以至于它们实际上被捆绑到 Express 中，但长久以来已被移至单独的包中。目前仅有与 Express 本身捆绑的中间件是 `static`。

这个列表试图涵盖最常见的中间件：

`basicauth-middleware`

提供基本的访问授权。请记住，基本认证只提供最基本的安全性，你应该仅在 HTTPS 下使用基本认证（否则用户名和密码将明文传输）。你应该仅在需要快速简单且使用 HTTPS 时才使用基本认证。

`body-parser`

提供 HTTP 请求体的解析。提供解析 URL 编码和 JSON 编码体以及其他类型的中间件。

`busboy`、`multiparty`、`formidable`、`multer`

所有这些中间件选项解析使用 `multipart/form-data` 编码的请求体。

`compression`

使用 gzip 或 deflate 压缩响应数据。这是一件好事，你的用户会感谢你，尤其是在慢速或移动连接下的用户。应该尽早链接此中间件，以防止其他可能发送响应的中间件。我唯一建议在 `compress` 之前链接的是调试或日志记录中间件（它们不发送响应）。请注意，在大多数生产环境中，压缩是由像 NGINX 这样的代理处理的，因此此中间件是不必要的。

`cookie-parser`

提供 cookie 支持。参见第九章。

`cookie-session`

提供基于 cookie 存储的会话支持。我一般不推荐这种会话方式。必须在 `cookie-parser` 之后链接此中间件。参见第九章。

`express-session`

提供会话 ID（存储在 cookie 中）的会话支持。默认使用内存存储，这在生产环境中并不适用，可以配置为使用数据库存储。参见第九章和第十三章。

`csurf`

提供对跨站点请求伪造（CSRF）攻击的防护。这个中间件使用会话，所以必须在`express-session`中间件之后链接。不幸的是，仅仅链接这个中间件并不能自动保护 against CSRF 攻击；更多信息请参见第十八章。

`serve-index`

为静态文件提供目录列表支持。除非你特别需要目录列表，否则不需要包含此中间件。

`errorhandler`

提供堆栈跟踪和错误消息给客户端。我不建议在生产服务器上链接此中间件，因为它暴露了实现细节，可能会出现安全或隐私问题。更多信息请参见第二十章。

`serve-favicon`

提供 *favicon*（出现在浏览器标题栏中的图标）。这并非绝对必要；你可以简单地在静态目录的根目录中放一个 *favicon.ico*，但这个中间件可以提高性能。如果使用，应该在中间件堆栈中放置得很靠前。它还允许你指定其他文件名而不是 *favicon.ico*。

`morgan`

提供自动记录支持；所有请求都将被记录。更多信息请参见第二十章。

`method-override`

提供对 `x-http-method-override` 请求头的支持，允许浏览器“伪装”使用 HTTP 方法而不是 `GET` 和 `POST`。这在调试时可能会有用。这只在你编写 API 时需要。

`response-time`

在响应中添加 `X-Response-Time` 头，提供响应时间（毫秒）。除非进行性能调优，你通常不需要这个中间件。

`static`

提供静态（公共）文件服务支持。你可以多次链接此中间件，指定不同的目录。更多细节请参见第十七章。

`vhost`

虚拟主机（vhosts），这个术语是从 Apache 借鉴来的，让 Express 中的子域名更容易管理。更多信息请参见第十四章。

# 第三方中间件

目前，还没有一个全面的“商店”或第三方中间件的索引。几乎所有的 Express 中间件都可以在 npm 上找到，所以如果你在 npm 上搜索“Express”和“中间件”，你会得到一个相当不错的列表。官方的 Express 文档还包含一个有用的[中间件列表](http://bit.ly/36UrbnL)。

# 结论

在本章中，我们深入探讨了中间件的定义，如何编写自己的中间件，以及它在 Express 应用程序中的处理过程。如果你开始认为 Express 应用程序只是一组中间件的集合，那你就开始理解 Express 了！甚至我们迄今为止使用的路由处理程序只是中间件的特殊案例。

在下一章中，我们将看到另一个常见的基础设施需求：发送电子邮件（你最好相信这里面会涉及到一些中间件！）。
