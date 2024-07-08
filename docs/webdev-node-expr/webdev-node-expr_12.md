# 第十二章：生产关注点

尽管现在开始讨论生产关注点可能感觉为时过早，但如果您从早期就开始思考生产，您将节省大量时间和痛苦。发布日将在您意识到之前就来临。

在本章中，您将了解 Express 对不同执行环境的支持，网站扩展的方法，以及如何监控网站的健康状况。您将看到如何模拟生产环境进行测试和开发，以及如何进行压力测试，以便在问题发生之前识别生产问题。

# 执行环境

Express 支持*执行环境*的概念：在生产、开发或测试模式下运行应用程序的一种方法。实际上，您可以拥有尽可能多的不同环境。例如，您可以有一个分段环境或培训环境。但请记住，开发、生产和测试是“标准”环境，Express 和第三方中间件通常根据这些环境做出决策。换句话说，如果您有一个“分段”环境，没有办法让它自动继承生产环境的属性。因此，我建议您坚持使用生产、开发和测试的标准。

虽然可以通过调用`app.set('env', \'production')`来指定执行环境，但这样做是不明智的；这意味着您的应用程序将始终在该环境中运行，无论情况如何。更糟糕的是，它可能会在一个环境中开始运行，然后切换到另一个环境。

最好通过环境变量`NODE_ENV`来指定执行环境。让我们修改我们的应用程序，通过调用`app.get('env’)`报告它正在运行的模式：

```
const port = process.env.PORT || 3000
app.listen(port, () => console.log(`Express started in ` +
  `${app.get('env')} mode at http://localhost:${port}` +
  `; press Ctrl-C to terminate.`))
```

如果您现在启动服务器，您将看到它正在开发模式下运行；如果您不指定其他模式，这是默认值。让我们尝试将其置于生产模式下：

```
$ export NODE_ENV=production
$ node meadowlark.js
```

如果您使用 Unix/BSD，有一种便捷的语法可以仅在该命令的执行期间修改环境：

```
$ NODE_ENV=production node meadowlark.js
```

这将以生产模式运行服务器，但一旦服务器终止，`NODE_ENV`环境变量将不会被修改。我特别喜欢这个快捷方式，它减少了我意外留下环境变量设置为我不一定想要的值的机会。

###### 注意

如果您以生产模式启动 Express，则可能会注意到有关不适合在生产模式下使用的组件的警告。如果您一直在本书的示例中跟进，您会看到`connect.session`正在使用内存存储，这在生产环境中是不合适的。一旦我们在第十三章中切换到数据库存储，这个警告将消失。

# 环境特定配置

尽管 Express 在生产模式下会在控制台上记录更多警告（例如，通知您即将删除并将来会移除的模块），但仅仅改变执行环境并不能做太多事情。此外，在生产模式下，默认情况下启用了视图缓存（参见第七章）。

主要的执行环境是一个工具，让你能够轻松地决定你的应用在不同环境下的行为。作为一种注意，你应该尽量减少开发、测试和生产环境之间的差异。也就是说，你应该谨慎使用这个功能。如果你的开发或测试环境与生产环境差异很大，那么在生产中就可能会出现不同的行为，这会导致更多的缺陷（或更难找到的缺陷）。一些差异是不可避免的；例如，如果您的应用程序高度依赖于数据库驱动，您可能不希望在开发过程中操作生产数据库，这将是适合特定环境配置的好选择。另一个低影响领域是更详细的日志记录。在开发中可能想要记录的许多事情在生产中是不必要记录的。

让我们给我们的服务器添加一些日志记录。关键在于我们希望在生产和开发环境下有不同的行为。对于开发环境，我们可以保留默认设置，但是对于生产环境，我们希望将日志记录到文件中。我们将使用`morgan`（别忘了`npm install morgan`），这是最常见的日志中间件（伴随库中的*ch12/00-logging.js*）：

```
const morgan = require('morgan')
const fs = require('fs')

switch(app.get('env')) {
  case 'development':
    app.use(morgan('dev'))
    break
  case 'production':
    const stream = fs.createWriteStream(__dirname + '/access.log',
      { flags: 'a' })
    app.use(morgan('combined', { stream }))
    break
}
```

如果你像平常一样启动服务器（`node meadowlark.js`）并访问站点，你会看到活动被记录到控制台。要查看应用在生产模式下的行为，请使用`NODE_ENV=production`运行它。现在，如果你访问应用程序，你将不会在终端上看到任何活动（这可能是我们希望在生产服务器上的情况），但所有活动都记录在[Apache 的组合日志格式](http://bit.ly/2NGC592)中，这是许多服务器工具的基本组成部分。

我们通过创建一个可追加的写入流（`{ flags: *a* }`）并将其传递给 morgan 配置来实现这一点。Morgan 有许多选项；要查看所有选项，请查阅[morgan 文档](http://bit.ly/32H5wMr)。

###### 注意

在上一个例子中，我们使用`__dirname`来将请求日志存储在项目本身的子目录中。如果你采用这种方法，你会希望将`log`添加到你的*.gitignore*文件中。或者，你可以采用更类似 Unix 的方法，将日志保存在*/var/log*的子目录中，就像 Apache 默认的做法一样。

我再次强调，当您做出特定于环境的配置选择时，应该用最佳判断力。请记住，当您的网站上线时，您的生产实例将在`生产`模式下运行（或应该如此）。每当您想要进行开发特定修改时，您应首先考虑这可能对生产中的质量保证造成的影响。我们将在第十三章看到一个更强大的环境特定配置的示例。

# 运行您的 Node 进程

到目前为止，我们一直在直接调用 `node` 运行我们的应用程序（例如，`node meadowlark.js`）。这对开发和测试来说是可以的，但在生产中有其缺点。特别是，如果您的应用程序崩溃或被终止，将没有任何保护措施。一个强大的*进程管理器*可以解决这个问题。

根据您的托管解决方案，如果提供了进程管理器，则可能不需要额外的进程管理器。也就是说，托管提供商将为您提供一个配置选项来指向您的应用程序文件，并将处理进程管理。

但是，如果您需要自己管理进程，则有两种流行的选项可供选择的进程管理器：

+   [永不停止](https://github.com/foreversd/forever)

+   [PM2](https://github.com/Unitech/pm2)

由于生产环境可能存在差异，我们不会详细讨论设置和配置进程管理器的具体事项。永不停止和 PM2 都有出色的文档，您可以在开发机器上安装和使用它们来学习如何配置。

我都用过，没有特别偏好。永不停止更为直接且易于入门，而 PM2 则提供了更多功能。

如果您想尝试一个进程管理器而又不想投入大量时间，我建议尝试使用永不停止。您可以分两步尝试它。首先，安装永不停止：

```
npm install -g forever
```

然后，使用永不停止启动您的应用程序（从应用程序根目录运行此命令）：

```
forever start meadowlark.js
```

您的应用现在正在运行……即使您关闭终端窗口，它也会继续运行！您可以使用 `forever restart meadowlark.js` 重新启动进程，并使用 `forever stop meadowlark.js` 停止它。

使用 PM2 起步会比较复杂，但如果您需要在生产环境中使用自己的进程管理器，它是值得一试的。

# 扩展您的网站

当前，扩展通常意味着两种情况之一：扩展上升或扩展外扩。*扩展上升* 指的是使服务器更强大：更快的 CPU、更好的架构、更多核心、更多内存等等。*扩展外扩* 则意味着更多的服务器。随着云计算的普及和虚拟化的普及，服务器计算能力变得不那么重要，根据您的需求，扩展外扩通常是最具成本效益的网站扩展方法。

在开发 Node 网站时，你应该始终考虑扩展的可能性。即使你的应用程序很小（也许它甚至是一个总是有限观众的内部网络应用程序），永远不会需要扩展，养成这种习惯也是个好主意。毕竟，也许你的下一个 Node 项目将成为下一个 Twitter，扩展将是必不可少的。幸运的是，Node 对扩展的支持非常好，考虑到这一点编写你的应用程序是毫不费力的。

在构建旨在扩展的网站时最重要的事情是持久性。如果你习惯于依赖基于文件的存储来保持持久性，*停下来*。这条路只会导致疯狂。

我第一次面对这个问题的经历几乎是灾难性的。我们的一个客户正在进行基于 Web 的比赛，该 Web 应用程序旨在通知前 50 名获奖者他们将获得奖品。由于某些企业 IT 限制，我们无法轻松使用数据库，因此大部分持久性通过编写平面文件来实现。我按照往常的做法继续进行，将每个条目保存到一个文件中。一旦文件记录了 50 名获奖者，就不再通知更多人他们中奖了。问题在于服务器是负载平衡的，因此一半的请求由一个服务器处理，另一半由另一个服务器处理。一台服务器通知了 50 名获奖者……另一台服务器也通知了。幸运的是，奖品不是很贵重（抱毯），而不是像 iPad 之类的昂贵物品，客户承担了损失，分发了 100 个奖品而不是 50 个（我提出自掏腰包支付额外的 50 条毯子，但他们慷慨地拒绝了我的提议）。

这个故事的寓意是，除非你有一个对*所有*服务器都可访问的文件系统，否则不应该依赖本地文件系统来保持持久性。有些例外是只读数据，如日志和备份。例如，我通常会将表单提交数据备份到本地平面文件中，以防数据库连接失败。在数据库故障的情况下，去每台服务器收集文件是一件麻烦事，但至少没有造成损失。

## 扩展应用集群

Node 本身支持*应用集群*，这是一种简单的单服务器扩展形式。通过应用集群，你可以为系统上的每个核心（CPU）创建一个独立的服务器（如果服务器数量超过核心数，不会提高应用程序的性能）。应用集群有两个好处：首先，它们可以帮助最大化给定服务器（硬件或虚拟机）的性能，其次，这是一种低开销的方式，在并行条件下测试你的应用程序。

让我们继续为我们的网站添加集群支持。虽然在主应用程序文件中完成所有这些工作非常常见，但我们将创建一个第二个应用程序文件，在集群中运行应用程序，使用我们一直在使用的非集群应用程序文件。为了启用这一点，我们首先必须对*meadowlark.js*进行轻微修改（参见伴侣库中的*ch12/01-server.js*的简化示例）：

```
function startServer(port) {
  app.listen(port, function() {
    console.log(`Express started in ${app.get('env')} ` +
      `mode on http://localhost:${port}` +
      `; press Ctrl-C to terminate.`)
  })
}

if(require.main === module) {
  // application run directly; start app server
  startServer(process.env.PORT || 3000)
} else {
  // application imported as a module via "require": export
  // function to create server
  module.exports = startServer
}
```

如果你还记得来自第五章，如果`require.main === module`，这意味着脚本直接运行；否则，它已经被另一个脚本通过`require`调用。

然后，我们创建一个新的脚本，*meadowlark-cluster.js*（请参见伴侣库中的*ch12/01-cluster*的简化示例）：

```
const cluster = require('cluster')

function startWorker() {
  const worker = cluster.fork()
  console.log(`CLUSTER: Worker ${worker.id} started`)
}

if(cluster.isMaster){

  require('os').cpus().forEach(startWorker)

  // log any workers that disconnect; if a worker disconnects, it
  // should then exit, so we'll wait for the exit event to spawn
  // a new worker to replace it
  cluster.on('disconnect', worker => console.log(
    `CLUSTER: Worker ${worker.id} disconnected from the cluster.`
  ))

  // when a worker dies (exits), create a worker to replace it
  cluster.on('exit', (worker, code, signal) => {
    console.log(
      `CLUSTER: Worker ${worker.id} died with exit ` +
      `code ${code} (${signal})`
    )
    startWorker()
  })

} else {

    const port = process.env.PORT || 3000
    // start our app on worker; see meadowlark.js
    require('./meadowlark.js')(port)

}
```

当执行此 JavaScript 时，它将处于主（直接使用`node meadowlark-cluster.js`运行时）或工作进程的上下文中，当 Node 的集群系统执行它时。属性`cluster.isMaster`和`cluster.isWorker`确定你正在运行的上下文。当我们运行此脚本时，它正在主模式下执行，并使用`cluster.fork`为系统中的每个 CPU 启动一个工作进程。此外，我们通过监听来自工作进程的`exit`事件来重新启动任何死掉的工作进程。

最后，在`else`子句中，我们处理工作进程。由于我们将*meadowlark.js*配置为作为一个模块使用，我们只需导入它并立即调用它（请记住，我们将其导出为一个启动服务器的函数）。

现在启动你的新集群服务器：

```
node meadowlark-cluster.js
```

###### 注意

如果你使用虚拟化（如 Oracle 的 VirtualBox），你可能需要配置你的虚拟机以具有多个 CPU。默认情况下，虚拟机通常只有一个 CPU。

假设你正在使用多核系统，你应该看到一些工作进程启动。如果你想看到不同的工作进程处理不同的请求证据，请在你的路由之前添加以下中间件：

```
const cluster = require('cluster')

app.use((req, res, next) => {
  if(cluster.isWorker)
    console.log(`Worker ${cluster.worker.id} received request`)
  next()
})
```

现在你可以用浏览器连接到你的应用程序。多次重新加载并查看如何在每个请求中从池中获取不同的工作进程（你可能无法；Node 被设计来处理大量连接，简单通过重新加载浏览器可能无法充分压力测试它；稍后我们将探讨压力测试，你将能更好地看到集群的运行情况）。

## 处理未捕获的异常

在 Node 的异步世界中，未捕获的异常是特别关注的问题。让我们从一个不会造成太多麻烦的简单例子开始（我鼓励你跟着这些例子一起学习）：

```
app.get('/fail', (req, res) => {
  throw new Error('Nope!')
})
```

当 Express 执行路由处理程序时，它将它们包装在 try/catch 块中，因此这实际上不是一个未捕获的异常。这不会造成太大问题：Express 会在服务器端记录异常，并且访问者会得到一个难看的堆栈转储。然而，您的服务器是稳定的，并且其他请求将继续正常提供服务。如果我们想要提供一个“友好”的错误页面，创建一个文件 *views/500.handlebars* 并在所有路由之后添加一个错误处理程序：

```
app.use((err, req, res, next) => {
  console.error(err.message, err.stack)
  app.status(500).render('500')
})
```

提供自定义错误页面始终是一个好习惯；当错误发生时，它不仅会使您的用户感到更专业，还允许您在错误发生时采取行动。例如，此错误处理程序将是通知开发团队发生错误的好地方。不幸的是，这仅对 Express 能够捕获的异常有效。让我们试试更糟糕的情况：

```
app.get('/epic-fail', (req, res) => {
  process.nextTick(() =>
    throw new Error('Kaboom!')
  )
})
```

请尝试一下。结果将会非常灾难性：它会使整个服务器都崩溃！除了不向用户显示友好的错误消息之外，现在您的服务器已经宕机，*没有*请求正在得到服务。这是因为 `setTimeout` 是*异步*执行的；带有异常的函数的执行被推迟直到 Node 空闲。问题是，当 Node 空闲并且准备执行函数时，它不再具有关于正在服务的请求的上下文，因此除了不体面地关闭整个服务器外，别无选择，因为现在它处于未定义状态。（Node 无法知道函数或其调用者的目的，因此它不再假定任何进一步的函数将正常工作。）

###### 注意

`process.nextTick` 类似于调用 `setTimeout` 参数为 0，但效率更高。我们在这里使用它进行演示目的；通常情况下，您不会在服务器端代码中使用它。然而，在接下来的章节中，我们将处理许多异步执行的事情，例如数据库访问、文件系统访问和网络访问等，它们都会遇到这个问题。

我们可以采取措施来处理未捕获的异常，但*如果 Node 无法确定您的应用程序的稳定性，那么您也无法确定*。换句话说，如果发生未捕获的异常，唯一的解决方法就是关闭服务器。在这种情况下，我们能做的最好的事情就是尽可能优雅地关闭并具备故障转移机制。最简单的故障转移机制是使用集群。如果您的应用程序在集群模式下运行并且一个工作进程死掉，主进程将会生成另一个工作进程来替代它。（甚至您不需要多个工作进程；一个带有一个工作进程的集群就足够了，尽管故障转移可能会稍慢一些。）

因此，在这种情况下，当我们面对未处理的异常时，如何以最优雅的方式关闭？Node 处理这种情况的机制是`uncaughtException`事件。（Node 还有一种称为*domains*的机制，但该模块已被弃用，不再推荐使用。）

```
process.on('uncaughtException', err => {
  console.error('UNCAUGHT EXCEPTION\n', err.stack);
  // do any cleanup you need to do here...close
  // database connections, etc.
  process.exit(1)
})
```

不能期望您的应用程序永远不会遇到未捕获的异常，但您应该有一个机制来记录异常并在发生时通知您，并且您应该认真对待它。尝试确定发生异常的原因，以便进行修复。像[Sentry](https://sentry.io)、[Rollbar](https://rollbar.com)、[Airbrake](https://airbrake.io/)和[New Relic](https://newrelic.com)这样的服务是记录这类错误以进行分析的好方法。例如，要使用 Sentry，首先您必须注册一个免费账户，然后您将收到一个数据源名称（DSN），然后您可以修改您的异常处理程序：

```
const Sentry = require('@sentry/node')
Sentry.init({ dsn: '** YOUR DSN GOES HERE **' })

process.on('uncaughtException', err => {
  // do any cleanup you need to do here...close
  // database connections, etc.
  Sentry.captureException(err)
  process.exit(1)
})
```

## 使用多台服务器进行扩展

虽然使用集群可以最大化单个服务器的性能，但当您需要多台服务器时会发生什么呢？这时情况会变得有些复杂。要实现这种并行性，您需要一个*代理*服务器。（通常称为*反向代理*或*前置代理*以区分常用于访问外部网络的代理，但我发现这种语言令人困惑且不必要，因此我将简称其为代理。）

两个非常流行的选项是[NGINX](https://www.nginx.com)（发音为“engine X”）和[HAProxy](http://www.haproxy.org)。尤其是 NGINX 服务器如雨后春笋般普及。我最近为我的公司进行了竞争分析，发现我们的竞争对手中多达 80%正在使用 NGINX。NGINX 和 HAProxy 都是强大且高性能的代理服务器，能够应对最苛刻的应用场景。（如果你需要证据，考虑到 Netflix 占据了多达 15%的*全球互联网流量*，它使用的是 NGINX。）

还有一些基于 Node 的较小的代理服务器，例如[node-http-proxy](http://bit.ly/34RWyNN)。如果您的需求较为简单或用于开发，这是一个不错的选择。对于生产环境，我建议使用 NGINX 或 HAProxy（它们都是免费的，但提供付费支持）。

安装和配置代理服务器超出了本书的范围，但实际并不像你想象的那么难（特别是如果你使用 node-http-proxy 或其他轻量级代理）。目前，使用集群可以确保我们的网站已经准备好进行扩展。

如果您配置了代理服务器，请确保告知 Express 您正在使用代理，并且应该信任该代理：

```
app.enable('trust proxy')
```

这样做将确保`req.ip`，`req.protocol`和`req.secure`将反映客户端和代理之间的连接详细信息，而不是客户端和您的应用程序之间的连接。此外，`req.ips`将是一个数组，指示原始客户端 IP 和任何中间代理的名称或 IP 地址。

# 监控您的网站

监控您的网站是您可以采取的最重要的，也是最经常被忽视的质量保证措施之一。在凌晨 3 点修复破损的网站是一件糟糕的事情，被你的老板在凌晨 3 点吵醒因为网站挂了更糟糕（或者更糟糕的是，在早上到达时意识到您的客户因为网站整晚都挂了而损失了 1 万美元并且没有人注意到）。

失败是无法避免的：它们和死亡以及税收一样，是不可避免的。但是，如果有一件事情可以让你的老板和客户相信你很擅长你的工作，那就是*始终*在他们之前知道故障。

## 第三方正常运行时间监控程序

在您网站的服务器上运行一个正常运行时间监控程序，就像在没有人居住的房子里安装烟雾报警器一样有效。它可能能够在某个页面挂掉时捕捉错误，但是如果整个*服务器*挂掉，可能会在不发出 SOS 的情况下就挂掉。这就是为什么您的第一道防线应该是第三方的正常运行时间监控程序。[UptimeRobot](http://uptimerobot.com)免费提供 50 个监视器，并且配置简单。警报可以发送到电子邮件，短信（短信），Twitter 或 Slack（等等）。您可以监控单个页面返回的代码（除了 200 之外的任何内容都被视为错误）或者检查页面上的关键字的存在或缺失。请记住，如果您使用关键字监视器，可能会影响您的分析（您可以在大多数分析服务中排除正常运行时间监控的流量）。

如果您的需求更复杂，还有其他更昂贵的服务，比如[Pingdom](http://pingdom.com)和[Site24x7](http://www.site24x7.com)。

# 压力测试

*压力测试*（或*负载测试*）旨在让您有信心，您的服务器能够承受同一时间的数百或数千个请求。这是另一个深奥的领域，可以成为专门一本书的主题：压力测试可以任意复杂，您想要多复杂取决于您项目的性质。如果您有理由相信您的网站可能会非常受欢迎，您可能需要在压力测试上投入更多时间。

让我们使用[Artillery](https://artillery.io/)添加一个简单的压力测试。首先，通过运行`npm install -g artillery`来安装 Artillery；然后编辑您的*package.json*文件，并在`scripts`部分添加以下内容：

```
  "scripts": {
    "stress": "artillery quick --count 10 -n 20 http://localhost:3000/"
  }
```

这将模拟 10 个“虚拟用户”(`--count 10`)，每个用户将向您的服务器发送 20 个请求(`-n 20`)。

确保你的应用正在运行（例如在单独的终端窗口中），然后运行`npm run stress`。你会看到像这样的统计数据：

```
Started phase 0, duration: 1s @ 16:43:37(-0700) 2019-04-14
Report @ 16:43:38(-0700) 2019-04-14
Elapsed time: 1 second
  Scenarios launched:  10
  Scenarios completed: 10
  Requests completed:  200
  RPS sent: 147.06
  Request latency:
    min: 1.8
    max: 10.3
    median: 2.5
    p95: 4.2
    p99: 5.4
  Codes:
    200: 200

All virtual users finished
Summary report @ 16:43:38(-0700) 2019-04-14
  Scenarios launched:  10
  Scenarios completed: 10
  Requests completed:  200
  RPS sent: 145.99
  Request latency:
    min: 1.8
    max: 10.3
    median: 2.5
    p95: 4.2
    p99: 5.4
  Scenario counts:
    0: 10 (100%)
  Codes:
    200: 200
```

这个测试是在我的开发笔记本上运行的。你可以看到 Express 在任何请求中的响应时间都不超过 10.3 毫秒，其中 99% 的请求在 5.4 毫秒以下完成。我无法提供具体的指导，告诉你应该寻找什么样的数字，但为了确保应用程序响应迅速，你应该寻找总连接时间在 50 毫秒以下的情况。（别忘了这仅仅是服务器传输数据给客户端所需的时间；客户端仍然需要渲染数据，这也需要时间，所以你在传输数据时花费的时间越少，越好。）

如果你定期对你的应用进行压力测试并进行基准测试，你就能够识别问题。如果你刚刚完成了一个功能，并发现你的连接时间增加了三倍，那么你可能需要对你的新功能进行一些性能调优！

# 结论

希望本章内容能够让你在接近应用程序发布时考虑到一些事项。生产应用程序涉及很多细节，虽然你无法预测到启动时可能发生的一切，但你越能预见，你就越能处于有利地位。借路易斯·巴斯德的话说，准备充足者多得利。
