# 第十一章：发送电子邮件

你的应用程序与世界沟通的主要方式之一是电子邮件。从用户注册到密码重置说明再到促销电子邮件，发送电子邮件是一个重要的功能。在本章中，你将学习如何使用 Node 和 Express 格式化并发送电子邮件，以帮助与你的用户进行沟通。

Node 和 Express 都没有内置的发送电子邮件的方法，因此我们必须使用第三方模块。我推荐的包是 Andris Reinman 的优秀 [*Nodemailer*](http://bit.ly/2Ked7vy)。在我们深入配置 Nodemailer 之前，让我们先了解一些电子邮件的基础知识。

# SMTP、MSA 和 MTA

发送电子邮件的通用语言是简单邮件传输协议（SMTP）。虽然可以使用 SMTP 直接将电子邮件发送到收件人的邮件服务器，但这通常不是一个好主意：除非你是像 Google 或 Yahoo! 这样的“受信任发送者”，否则你的邮件很可能会直接被投入垃圾箱。最好使用邮件提交代理（MSA），它会通过可信任的渠道传递邮件，从而减少邮件被标记为垃圾邮件的机会。除了确保你的邮件能送达，MSA 还处理像临时中断和退信等问题。整个过程的最后一部分是邮件传输代理（MTA），它是实际将邮件发送到最终目的地的服务。在本书中，*MSA*、*MTA* 和 *SMTP 服务器* 本质上是等效的。

因此，你将需要访问一个 MSA。虽然可以使用像 Gmail、Outlook 或 Yahoo! 这样的免费消费者电子邮件服务开始工作，但这些服务不再像以前那样友好地支持自动化邮件（为了减少滥用）。幸运的是，有几个优秀的电子邮件服务可供选择，适合低频使用并提供免费选项：[Sendgrid](https://sendgrid.com) 和 [Mailgun](https://www.mailgun.com)。我使用过这两个服务，都很喜欢。本书的示例将使用 SendGrid。

如果你在一个组织工作，组织本身可能有一个 MSA；你可以联系你的 IT 部门询问是否有可用于发送自动化电子邮件的 SMTP 中继。

如果你在使用 SendGrid 或 Mailgun，请立即设置你的账户。对于 SendGrid，你需要创建一个 API 密钥（它将作为你的 SMTP 密码）。

# 接收电子邮件

大多数网站只需要能够 *发送* 电子邮件，如密码重置说明和促销电子邮件。然而，有些应用程序也需要接收电子邮件。一个很好的例子是问题追踪系统，当有人更新问题时会发送电子邮件，如果你回复该邮件，则问题会自动更新为你的回复。

不幸的是，接收电子邮件涉及的内容要多得多，这本书不会涉及这方面。 如果这是你需要的功能，你应该允许你的邮件提供商维护邮箱，并定期使用像[imap-simple](http://bit.ly/2qQK0r5)这样的 IMAP 代理来访问它。

# 电子邮件标题

电子邮件消息由两部分组成：头部和正文（非常类似于 HTTP 请求）。 *头部* 包含有关电子邮件的信息：谁发送的、发给谁、收到的日期、主题等等。 这些头部通常在电子邮件应用程序中显示给用户，但还有许多其他头部。 大多数电子邮件客户端允许你查看头部； 如果你从未这样做过，我建议你试试。 头部提供了有关电子邮件如何到达你手中的所有信息； 每个经过的服务器和 MTA 都将在头部中列出。

人们经常会感到惊讶的是，某些头部，如“发件人”地址，可以由发件人任意设置。 当你指定一个与发送账户不同的“发件人”地址时，通常称为*伪造*。 除非你有合理的理由这样做，否则不要滥用。 有时这样做是合理的，但你不应该滥用它。

你发送的电子邮件*必须*有一个“发件人”地址。 但是，当发送自动化电子邮件时，有时可能会出现问题，这就是为什么你经常看到带有返回地址的电子邮件，如 DO NOT REPLY <*do-not-reply@meadowlarktravel.com*>。 是否采用这种方式或者让自动化邮件来自 Meadowlark Travel <*info@meadowlarktravel.com*> 是由你决定的；但是，如果你选择后者，你应该准备好回复发送到*info@meadowlarktravel.com*的电子邮件。

# 电子邮件格式

当互联网刚刚出现时，所有的电子邮件都是简单的 ASCII 文本。 自那时以来，世界发生了很大变化，人们希望用不同的语言发送电子邮件，并做更复杂的事情，比如包含格式化文本、图片和附件。 这就是事情开始变得混乱的地方：电子邮件的格式和编码是一堆可怕的技术和标准的混合。

幸运的是，我们不必真正去解决这些复杂性。 Nodemailer 会为我们处理这一切。 对你来说重要的是，你的电子邮件可以是纯文本（Unicode）或 HTML。

几乎所有现代电子邮件应用程序都支持 HTML 电子邮件，因此通常很安全地格式化你的电子邮件为 HTML。 尽管如此，还有一些“文本纯洁主义者”不喜欢 HTML 电子邮件，因此我建议始终包含文本和 HTML 电子邮件。 如果你不想写文本和 HTML 电子邮件，Nodemailer 支持一种快捷方式，可以从 HTML 自动生成纯文本版本。

# HTML 电子邮件

HTML 邮件是一个可以填满整本书的话题。不幸的是，它并不像为您的网站编写 HTML 那样简单：大多数邮件客户端仅支持 HTML 的一个小子集。大多数时候，您必须像在 1996 年一样编写 HTML；这并不好玩。特别是，您必须重新使用表格来进行布局（播放悲伤的音乐）。

如果您有处理 HTML 浏览器兼容性问题的经验，您就知道它可能会让人头疼。电子邮件兼容性问题要糟糕得多。幸运的是，有一些东西可以帮助解决。

首先，我鼓励您阅读 MailChimp 关于[撰写 HTML 邮件的文章](http://bit.ly/33CsaXs)。它很好地涵盖了基础知识，并解释了撰写 HTML 邮件时需要牢记的事项。

接下来是真正的时间节省器：[HTML Email Boilerplate](http://bit.ly/2qJ1XIe)。它本质上是一个非常好的、经过严格测试的 HTML 邮件模板。

最后，有测试。您已经学习了如何编写 HTML 邮件，并且正在使用 HTML Email Boilerplate，但测试是确保您的电子邮件不会在 Lotus Notes 7 上爆炸的唯一方法（是的，人们仍在使用它）。感觉要安装 30 种不同的邮件客户端来测试一个电子邮件吗？我不这么认为。幸运的是，有一个很棒的服务可以为您完成：[Litmus](http://bit.ly/2NI6JPo)。这并不是一项便宜的服务；计划每月大约从 $100 起步。但是，如果您发送大量促销电子邮件，这是无法超越的。

另一方面，如果您的格式较为简单，就不需要像 Litmus 这样昂贵的测试服务了。如果您坚持使用标题、粗体/斜体文本、水平规则和一些图像链接等内容，您就非常安全了。

# Nodemailer

首先，我们需要安装 Nodemailer 包：

```
npm install nodemailer
```

然后，需要引用 `nodemailer` 包并创建一个 Nodemailer 实例（在 Nodemailer 的术语中称为 *transport*）：

```
const nodemailer = require('nodemailer')

const mailTransport = nodemailer.createTransport({

  auth: {
    user: credentials.sendgrid.user,
    pass: credentials.sendgrid.password,
  }
})
```

注意我们正在使用我们在 第九章 中设置的凭据模块。您需要相应地更新您的 *.credentials.development.json* 文件：

```
{
  "cookieSecret": "your cookie secret goes here",
  "sendgrid": {
    "user": "your sendgrid username",
    "password": "your sendgrid password"
  }
}
```

SMTP 的常见配置选项包括端口、认证类型和 TLS 选项。但是，大多数主要的邮件服务使用默认选项。要找出要使用的设置，请参阅您的邮件服务文档（尝试搜索 *sending SMTP email* 或 *SMTP configuration* 或 *SMTP relay*）。如果您在发送 SMTP 邮件时遇到问题，您可能需要检查选项；请参阅 [Nodemailer documentation](https://nodemailer.com/smtp) 获取完整的支持选项列表。

###### 注意

如果您在跟随伴随的仓库，您会发现凭据文件中没有任何设置。过去，我有很多读者联系我，问为什么文件丢失或为空。出于同样的原因，我故意不提供有效的凭据，就像您需要小心自己的凭据一样！亲爱的读者，我非常信任您，但不至于给您我的电子邮件密码！

## 发送邮件

现在我们有了邮件传输实例，我们可以开始发送邮件了。我们将从一个简单的例子开始，只向一个收件人发送文本邮件（*ch11/00-smtp.js*在伴随代码库中）。

```
try {
  const result = await mailTransport.sendMail({
    from: '"Meadowlark Travel" <info@meadowlarktravel.com>',
    to: 'joecustomer@gmail.com',
    subject: 'Your Meadowlark Travel Tour',
    text: 'Thank you for booking your trip with Meadowlark Travel.  ' +
      'We look forward to your visit!',
  })
  console.log('mail sent successfully: ', result)
} catch(err) {
  console.log('could not send mail: ' + err.message)
}
```

###### 注意

在本节的代码示例中，我使用类似*joecustomer@gmail.com*的虚假电子邮件地址。为了验证目的，您可能需要将这些电子邮件地址更改为您控制的电子邮件地址，以便查看发生的情况。否则，可怜的*joecustomer@gmail.com*将会收到大量无意义的电子邮件！

您会注意到我们在这里处理了错误，但重要的是要理解，没有错误并不一定意味着您的电子邮件已成功发送给*收件人*。如果存在与 MSA 通信的问题（如网络或身份验证错误），则回调的`error`参数将被设置。如果 MSA 无法将电子邮件发送给收件人（例如由于无效的电子邮件地址或未知用户），则您将需要检查您邮件服务中的账户活动，您可以通过管理界面或 API 进行此操作。

如果您的系统需要自动确定电子邮件是否成功发送，请使用您邮件服务的 API。查阅您邮件服务的 API 文档获取更多信息。

## 向多个收件人发送邮件

Nodemail 支持使用逗号将邮件发送给多个收件人（*ch11/01-multiple-recipients.js*在伴随代码库中）：

```
try {
  const result = await mailTransport.sendMail({
    from: '"Meadowlark Travel" <info@meadowlarktravel.com>',
    to: 'joe@gmail.com, "Jane Customer" <jane@yahoo.com>, ' +
      'fred@hotmail.com',
    subject: 'Your Meadowlark Travel Tour',
    text: 'Thank you for booking your trip with Meadowlark Travel.  ' +
      'We look forward to your visit!',
  })
  console.log('mail sent successfully: ', result)
} catch(err) {
  console.log('could not send mail: ' + err.message)
}
```

请注意，在此示例中，我们混合了普通电子邮件地址（*joe@gmail.com*）和指定收件人姓名的电子邮件地址（“Jane Customer” < *jane@yahoo.com*>）。这是允许的语法。

当向多个收件人发送电子邮件时，您必须注意观察您的邮件发送代理（MSA）的限制。例如，SendGrid 建议限制收件人数量（SendGrid 建议一封邮件中不超过一千个收件人）。如果您正在发送批量邮件，则可能希望发送多个消息，每个消息都有多个收件人（*ch11/02-many-recipients.js*在伴随代码库中）：

```
// largeRecipientList is an array of email addresses
const recipientLimit = 100
const batches = largeRecipientList.reduce((batches, r) => {
  const lastBatch = batches[batches.length - 1]
  if(lastBatch.length < recipientLimit)
    lastBatch.push(r)
  else
    batches.push([r])
  return batches
}, [[]])
try {
  const results = await Promise.all(batches.map(batch =>
    mailTransport.sendMail({
      from: '"Meadowlark Travel", <info@meadowlarktravel.com>',
      to: batch.join(', '),
      subject: 'Special price on Hood River travel package!',
      text: 'Book your trip to scenic Hood River now!',
    })
  ))
  console.log(results)
} catch(err) {
  console.log('at least one email batch failed: ' + err.message)
}
```

# 更好的批量电子邮件选项

虽然您可以使用 Nodemailer 和适当的 MSA 发送批量邮件，但在选择这条路线之前应仔细考虑。负责任的电子邮件营销活动必须提供取消订阅的途径，这并不是一个微不足道的任务。乘以您维护的每个订阅列表（也许您有每周通讯和特别公告活动，例如）。这是一个最好不要重复造轮子的领域。像[Emma](https://myemma.com)、[Mailchimp](http://mailchimp.com)和[Campaign Monitor](http://www.campaignmonitor.com)等服务提供了您所需的一切，包括监控电子邮件营销活动成功的优秀工具。它们价格合理，我强烈推荐在促销邮件、通讯等方面使用它们。

# 发送 HTML 电子邮件

到目前为止，我们一直在发送纯文本电子邮件，但现在大多数人都希望看到一些更漂亮的东西。Nodemailer 允许您在同一封电子邮件中发送 HTML 和纯文本版本，允许电子邮件客户端选择显示哪个版本（通常是 HTML）（伴随存储库中的*ch11/03-html-email.js*）：

```
const result = await mailTransport.sendMail({
  from: '"Meadowlark Travel" <info@meadowlarktravel.com>',
  to: 'joe@gmail.com, "Jane Customer" <jane@yahoo.com>, ' +
    'fred@hotmail.com',
  subject: 'Your Meadowlark Travel Tour',
  html: '<h1>Meadowlark Travel</h1>\n<p>Thanks for book your trip with ' +
    'Meadowlark Travel.  <b>We look forward to your visit!</b>',
  text: 'Thank you for booking your trip with Meadowlark Travel.  ' +
    'We look forward to your visit!',
})
```

提供 HTML 和纯文本版本会增加很多工作量，特别是如果您的用户中很少有人喜欢纯文本电子邮件。如果您想节省一些时间，可以在 HTML 中编写电子邮件，并使用像[html-to-formatted-text](http://bit.ly/34RX8Lq)这样的包自动生成文本。 （请记住，它的质量可能不如手工编写的文本高；HTML 并非总是能够干净地转换。）

## HTML 电子邮件中的图像

虽然可以将图像嵌入 HTML 电子邮件，但我强烈不建议这样做。它们会使您的电子邮件消息变得臃肿，通常不被视为良好的做法。相反，您应该将要在电子邮件中使用的图像放在您的 Web 服务器上，并适当地进行链接。

最好在您的静态资产文件夹中有一个专门用于电子邮件图像的位置。您甚至应将在网站和电子邮件中都使用的资产分开。这样可以减少影响电子邮件布局的机会。

让我们在 Meadowlark Travel 项目中添加一些电子邮件资源。在*public*目录下创建一个名为*email*的子目录。您可以将*logo.png*放在这里，以及您想在电子邮件中使用的任何其他图像。然后，在您的电子邮件中，您可以直接使用这些图像：

```
<img src="//meadowlarktravel.com/email/logo.png"
  alt="Meadowlark Travel Logo">
```

###### 注

显而易见，当向其他人发送电子邮件时，不应使用*localhost*；他们可能甚至没有运行服务器，更不用说在 3000 端口上了！根据您的邮件客户端，您可能可以在电子邮件中使用*localhost*进行测试，但它在您的计算机之外是无法工作的。在第十七章中，我们将讨论一些技术，以平稳地从开发过渡到生产。

## 使用视图发送 HTML 电子邮件

到目前为止，我们一直在 JavaScript 中的字符串中放置我们的 HTML，这是一个应尽量避免的做法。虽然我们的 HTML 足够简单，但看看[HTML Email Boilerplate](http://bit.ly/2qJ1XIe)：您想把所有这些样板放在一个字符串中吗？绝对不。

幸运的是，我们可以利用视图来处理这个问题。让我们考虑一下我们的“感谢您与 Meadowlark Travel 预订旅行”的电子邮件示例，稍微扩展一下。假设我们有一个包含订单信息的购物车对象。该购物车对象将存储在会话中。假设我们订购过程的最后一步是一个由*/cart/checkout*处理的表单，该表单发送确认电子邮件。让我们首先为感谢页面创建一个视图，*views/cart-thank-you.handlebars*：

```
<p>Thank you for booking your trip with Meadowlark Travel,
  {{cart.billing.name}}!</p>
<p>Your reservation number is {{cart.number}}, and an email has been
sent to {{cart.billing.email}} for your records.</p>
```

然后，我们将为电子邮件创建一个电子邮件模板。下载 HTML 电子邮件样板，并放入*views/email/cart-thank-you.handlebars*。编辑文件并修改正文：

```
<table cellpadding="0" cellspacing="0" border="0" id="backgroundTable">
  <tr>
    <td valign="top">
      <table cellpadding="0" cellspacing="0" border="0" align="center">
        <tr>
          <td width="200" valign="top"><img class="image_fix"
            src="//placehold.it/100x100"
            alt="Meadowlark Travel" title="Meadowlark Travel"
            width="180" height="220" /></td>
        </tr>
        <tr>
          <td width="200" valign="top"><p>
            Thank you for booking your trip with Meadowlark Travel,
            {{cart.billing.name}}.</p><p>Your reservation number
            is {{cart.number}}.</p></td>
        </tr>
        <tr>
          <td width="200" valign="top">Problems with your reservation?
          Contact Meadowlark Travel at
          <span class="mobile_link">555-555-0123</span>.</td>
        </tr>
      </table>
    </td>
  </tr>
</table>
```

###### 小贴士

由于电子邮件中无法使用*localhost*地址，如果您的站点尚未上线，您可以使用占位符服务来获取任何图形。例如，[*http://placehold.it/100x100*](http://placehold.it/100x100)*动态提供一个您可以使用的 100 像素正方形图形。这种技术经常用于仅用于位置的（FPO）图像和布局目的。

现在我们可以为我们的购物车感谢页面创建一个路由（伴随仓库中的*ch11/04-rendering-html-email.js*）：

```
app.post('/cart/checkout', (req, res, next) => {
  const cart = req.session.cart
  if(!cart) next(new Error('Cart does not exist.'))
  const name = req.body.name || '', email = req.body.email || ''
  // input validation
  if(!email.match(VALID_EMAIL_REGEX))
    return res.next(new Error('Invalid email address.'))
  // assign a random cart ID; normally we would use a database ID here
  cart.number = Math.random().toString().replace(/⁰\.0*/, '')
  cart.billing = {
    name: name,
    email: email,
  }
  res.render('email/cart-thank-you', { layout: null, cart: cart },
    (err,html) => {
        console.log('rendered email: ', html)
        if(err) console.log('error in email template')
        mailTransport.sendMail({
          from: '"Meadowlark Travel": info@meadowlarktravel.com',
          to: cart.billing.email,
          subject: 'Thank You for Book your Trip with Meadowlark Travel',
          html: html,
          text: htmlToFormattedText(html),
        })
          .then(info => {
            console.log('sent! ', info)
            res.render('cart-thank-you', { cart: cart })
          })
          .catch(err => {
            console.error('Unable to send confirmation: ' + err.message)
          })
    }
  )
})
```

注意我们调用了`res.render`两次。通常情况下，您只调用一次（调用两次将仅显示第一次调用的结果）。但在这种情况下，我们绕过了第一次调用时的正常渲染过程：请注意我们提供了一个回调函数。这样做可以防止将视图的结果呈现给浏览器。相反，回调函数在参数`html`中接收呈现的视图：我们只需获取呈现的 HTML 并发送电子邮件！我们指定`layout: null`以防止使用我们的布局文件，因为所有内容都在电子邮件模板中（另一种方法是为电子邮件创建单独的布局文件并使用该文件）。最后，我们再次调用`res.render`。这次，结果将像往常一样呈现为 HTML 响应。

## 封装电子邮件功能

如果您在整个站点上经常使用电子邮件，您可能希望封装电子邮件功能。让我们假设您始终希望您的站点从同一发送者（“草地雀旅行” <*info@meadowlarktravel.com*>) 发送电子邮件，并且您始终希望以 HTML 格式发送自动生成的文本电子邮件。创建一个名为*lib/email.js*（伴随仓库中的*ch11/lib/email.js*）的模块：

```
const nodemailer = require('nodemailer')
const htmlToFormattedText = require('html-to-formatted-text')

module.exports = credentials => {

  const mailTransport = nodemailer.createTransport({
    host: 'smtp.sendgrid.net',
    auth: {
      user: credentials.sendgrid.user,
      pass: credentials.sendgrid.password,
    },
  })

  const from = '"Meadowlark Travel" <info@meadowlarktravel.com>'
  const errorRecipient = 'youremail@gmail.com'

  return {
    send: (to, subject, html) =>
      mailTransport.sendMail({
        from,
        to,
        subject,
        html,
        text: htmlToFormattedText(html),
      }),
  }

}
```

现在我们只需执行以下操作即可发送电子邮件（伴随仓库中的*ch11/05-email-library.js*）：

```
const emailService = require('./lib/email')(credentials)

emailService.send(email, "Hood River tours on sale today!",
  "Get 'em while they're hot!")
```

# 结论

在本章中，您了解了互联网上电子邮件传递的基础知识。如果您在跟随操作，您已设置了一个免费的电子邮件服务（很可能是 SendGrid 或 Mailgun），并使用该服务发送了文本和 HTML 电子邮件。您还了解了我们如何使用与在 Express 应用程序中渲染 HTML 相同的模板渲染机制来为电子邮件渲染 HTML。

电子邮件仍然是您的应用程序与用户沟通的重要方式。请注意不要滥用这种权力！如果您像我一样，收件箱里充斥着大量的自动化电子邮件，您大部分时间可能会忽略它们。在涉及自动化电子邮件时，少则多。您的应用程序向用户发送电子邮件可能有合法且有用的原因，但您应该始终问自己，“我的用户*真的*想要这封电子邮件吗？是否有其他方式来传达这些信息？”

现在，我们已经介绍了一些创建应用程序所需的基础设施，接下来我们将花一些时间讨论应用程序最终的生产发布，并需要考虑的各种因素，以确保发布成功。
