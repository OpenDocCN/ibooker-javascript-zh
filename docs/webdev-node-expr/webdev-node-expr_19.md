# 第十九章：与第三方 API 集成

越来越成功的网站不再是完全独立的。与社交网络的集成对于吸引现有用户并找到新用户至关重要。为了提供门店定位器或其他位置感知服务，使用地理定位和地图服务至关重要。事情并不止于此：越来越多的组织意识到提供 API 有助于扩展其服务并使其更有用。

在本章中，我们将讨论两种最常见的集成需求：社交媒体和地理定位。

# 社交媒体

社交媒体是促销产品或服务的绝佳方式：如果这是您的目标，那么使用户能够轻松在社交媒体网站上分享您的内容至关重要。我写这篇文章时，主要的社交网络服务包括 Facebook、Twitter、Instagram 和 YouTube。像 Pinterest 和 Flickr 这样的网站也有它们的用途，但它们通常更加针对特定受众（例如，如果您的网站是关于 DIY 手工艺品的，您绝对需要支持 Pinterest）。笑吧，但我预测 MySpace 将会复兴。它的网站重新设计很有启发性，值得注意的是，MySpace 是建立在 Node 上的。

## 社交媒体插件与网站性能

大多数社交媒体集成是前端的事务。您在页面中引用适当的 JavaScript 文件，它会使得入站内容（例如来自您的 Facebook 页面的前三篇文章）和出站内容（例如在您所在页面上发布推文的能力）都变得可能。虽然这通常代表了社交媒体集成的最简单路径，但也伴随着成本：由于额外的 HTTP 请求，页面加载时间可能会增加一倍甚至三倍。如果页面性能对您很重要（尤其是对移动用户而言），您应该仔细考虑如何集成社交媒体。

话虽如此，启用 Facebook 点赞按钮或推特按钮的代码利用了浏览器中的 cookie 来代表用户发布内容，将这些功能移至后端将会很困难（有时甚至是不可能的）。因此，如果这是您需要的功能，链接适当的第三方库是最佳选择，尽管它可能会影响页面性能。

## 搜索推文

假设我们想要提及包含标签#Oregon #travel 的最近十条推文。我们可以使用前端组件来实现这一点，但这将涉及额外的 HTTP 请求。此外，如果我们在后端进行操作，我们可以选择缓存推文以提升性能。此外，如果我们在后端进行搜索，我们可以对“恶意推文”进行黑名单处理，这在前端会更为困难。

Twitter，类似于 Facebook，允许您创建*apps*。这有点误导：Twitter 应用程序并不*做*任何事情（传统意义上）。它更像是一组您可以在站点上使用来创建实际应用程序的凭据。访问 Twitter API 的最简单和最可移植的方式是创建一个应用程序并使用它来获取访问令牌。

通过访问[*http://dev.twitter.com*](http://dev.twitter.com)来创建 Twitter 应用程序。确保已登录，然后点击导航栏中的用户名，然后选择“Apps”。点击“创建应用程序”，按照说明进行操作。创建应用程序后，您将看到现在有一个*consumer API key*和一个*API secret key*。API secret key 应保持秘密：绝不要将其包含在发送给客户端的响应中。如果第三方获取了此秘密，他们可以代表您的应用程序进行请求，如果使用是恶意的，对您可能会产生不利后果。

现在我们有了一个 consumer API key 和 secret key，我们可以与 Twitter REST API 进行通信。

为了保持代码整洁，我们将我们的 Twitter 代码放在一个名为*lib/twitter.js*的模块中：

```
const https = require('https')

module.exports = twitterOptions => {

 return {

  search: async (query, count) => {
    // TODO
  }
 }

}
```

这种模式应该开始变得熟悉了。我们的模块将一个配置对象传递给调用者导出一个函数。返回的是一个包含方法的对象。通过这种方式，我们可以为我们的模块添加功能。目前，我们只提供了一个`search`方法。以下是我们将如何使用这个库的方法：

```
const twitter = require('./lib/twitter')({
  consumerApiKey: credentials.twitter.consumerApiKey,
  apiSecretKey: credentials.twitter.apiSecretKey,
})

const tweets = await twitter.search('#Oregon #travel', 10)
// tweets will be in result.statuses
```

（在*.credentials.development.json*文件中不要忘记添加`twitter`属性，包括`consumerApiKey`和`apiSecretKey`。）

在我们实现`search`方法之前，我们必须提供一些功能来对我们自己进行 Twitter 认证。这个过程很简单：我们使用 HTTPS 请求基于我们的 consumer key 和 consumer secret 的访问令牌。我们只需做一次：目前，Twitter 不会过期访问令牌（尽管您可以手动使其失效）。因为我们不想每次都请求访问令牌，所以我们将缓存访问令牌以便重复使用。

我们构建模块的方式允许我们创建对调用者不可见的私有功能。具体来说，对调用者可见的仅有`module.exports`。因为我们返回的是一个函数，所以只有该函数对调用者可见。调用该函数将返回一个对象，而只有该对象的属性对调用者可见。因此，我们将创建一个名为`accessToken`的变量，用于缓存访问令牌，以及一个名为`getAccessToken`的函数，用于获取访问令牌。首次调用时，它将发出 Twitter API 请求以获取访问令牌。后续调用将简单地返回`accessToken`的值：

```
const https = require('https')

module.exports = function(twitterOptions) {

  // this variable will be invisible outside of this module
  let accessToken = null

  // this function will be invisible outside of this module
  const getAccessToken = async () => {
    if(accessToken) return accessToken
    // TODO: get access token
  }

  return {
    search: async (query, count) => {
      // TODO
    }
  }

}
```

我们将`getAccessToken`标记为异步，因为我们可能需要向 Twitter API 发出 HTTP 请求（如果没有缓存的令牌）。既然我们已经建立了基本结构，让我们实现`getAccessToken`：

```
const getAccessToken = async () => {
  if(accessToken) return accessToken

  const bearerToken = Buffer(
    encodeURIComponent(twitterOptions.consumerApiKey) + ':' +
    encodeURIComponent(twitterOptions.apiSecretKey)
  ).toString('base64')

  const options = {
    hostname: 'api.twitter.com',
    port: 443,
    method: 'POST',
    path: '/oauth2/token?grant_type=client_credentials',
    headers: {
      'Authorization': 'Basic ' + bearerToken,
    },
  }

  return new Promise((resolve, reject) =>
    https.request(options, res => {
      let data = ''
      res.on('data', chunk => data += chunk)
      res.on('end', () => {
        const auth = JSON.parse(data)
        if(auth.token_type !== 'bearer')
          return reject(new Error('Twitter auth failed.'))
        accessToken = auth.access_token
        return resolve(accessToken)
      })
    }).end()
  )
}
```

构建此调用的详细信息可在[Twitter 的应用程序仅身份验证开发文档页面](http://bit.ly/2KcJ4EA)中找到。基本上，我们必须构造一个基于 base64 编码的消费者密钥和消费者密钥的令牌。构造了该令牌后，我们可以使用包含该令牌的`Authorization`头部调用`/oauth2/token` API 请求访问令牌。请注意，我们必须使用 HTTPS：如果尝试通过 HTTP 进行此调用，则会未加密地传输您的密钥，API 将简单地挂断您的连接。

一旦我们从 API 接收到完整的响应（我们监听响应流的`end`事件），我们可以解析 JSON，确保令牌类型为`bearer`，然后继续我们的工作。我们缓存访问令牌，然后调用回调函数。

现在我们有了获取访问令牌的机制，我们可以进行 API 调用了。让我们实现我们的`search`方法：

```
search: async (query, count) => {
  const accessToken = await getAccessToken()
  const options = {
    hostname: 'api.twitter.com',
    port: 443,
    method: 'GET',
    path: '/1.1/search/tweets.json?q=' +
      encodeURIComponent(query) +
      '&count=' + (count || 10),
    headers: {
      'Authorization': 'Bearer ' + accessToken,
    },
  }
  return new Promise((resolve, reject) =>
    https.request(options, res => {
      let data = ''
      res.on('data', chunk => data += chunk)
      res.on('end', () => resolve(JSON.parse(data)))
    }).end()
  )
},
```

## 渲染推文

现在我们有了搜索推文的能力……那么我们如何在网站上显示它们？在很大程度上取决于您，但是有一些需要考虑的事项。Twitter 希望确保其数据的使用符合品牌的一致性。为此，它确实有[显示要求](http://bit.ly/32ET4N2)，其中包含您必须包含的功能元素来显示推文。

在需求方面有一些灵活性（例如，如果您在不支持图像的设备上显示，则无需包含头像图像），但大部分时间，您将最终得到非常类似于嵌入式推文的东西。这是一项很大的工作，但是有一种方法可以绕过它……但这涉及到链接到 Twitter 的小部件库，这正是我们试图避免的 HTTP 请求。

如果您需要显示推文，最好使用 Twitter 小部件库，即使这会产生额外的 HTTP 请求。对于更复杂的 API 使用，您仍然需要从后端访问 REST API，因此您可能最终会与前端脚本一起使用 REST API。

让我们继续我们的示例：我们希望显示提到标签#Oregon #travel 的前 10 条推文。我们将使用 REST API 搜索推文，并使用 Twitter 小部件库显示它们。由于我们不希望超出使用限制（或减慢服务器速度），我们将缓存这些推文和用于显示它们的 HTML 15 分钟。

我们将首先修改我们的 Twitter 库，以包含一个 `embed` 方法，该方法获取用于显示推文的 HTML。请注意，我们正在使用 npm 库 `querystringify` 来从对象构建查询字符串，因此不要忘记 `npm install querystringify` 并导入它（`const qs = require( ‘querystringify ’)`），然后将以下函数添加到 *lib/twitter.js* 的导出中：

```
embed: async (url, options = {}) => {
  options.url = url
  const accessToken = await getAccessToken()
  const requestOptions = {
    hostname: 'api.twitter.com',
    port: 443,
    method: 'GET',
    path: '/1.1/statuses/oembed.json?' + qs.stringify(options),
    headers: {
      'Authorization': 'Bearer ' + accessToken,
    },
  }
  return new Promise((resolve, reject) =>
    https.request(requestOptions, res => {
      let data = ''
      res.on('data', chunk => data += chunk)
      res.on('end', () => resolve(JSON.parse(data)))
    }).end()
  )
},
```

现在我们准备搜索并缓存推文了。在我们的主应用程序文件中，创建以下函数 `getTopTweets`：

```
const twitterClient = createTwitterClient(credentials.twitter)

const getTopTweets = ((twitterClient, search) => {
  const topTweets = {
    count: 10,
    lastRefreshed: 0,
    refreshInterval: 15 * 60 * 1000,
    tweets: [],
  }
  return async () => {
    if(Date.now() > topTweets.lastRefreshed + topTweets.refreshInterval) {
      const tweets =
       await twitterClient.search('#Oregon #travel', topTweets.count)
      const formattedTweets = await Promise.all(
        tweets.statuses.map(async ({ id_str, user }) => {
          const url = `https://twitter.com/${user.id_str}/statuses/${id_str}`
          const embeddedTweet =
           await twitterClient.embed(url, { omit_script: 1 })
          return embeddedTweet.html
        })
      )
      topTweets.lastRefreshed = Date.now()
      topTweets.tweets = formattedTweets
    }
    return topTweets.tweets
  }
})(twitterClient, '#Oregon #travel')
```

`getTopTweets` 函数的核心不仅仅是搜索具有指定标签的推文，而是为一段合理时间内的这些推文进行缓存。请注意，我们创建了一个立即调用的函数表达式（IIFE）：这是因为我们希望 `topTweets` 缓存在闭包内部，以防止被篡改。从 IIFE 返回的异步函数在必要时刷新缓存，然后返回缓存内容。

最后，让我们创建一个视图，`views/social.handlebars`，作为我们社交媒体存在的主页（目前仅包括我们选择的推文）：

```
<h2>Oregon Travel in Social Media</h2>

<script id="twitter-wjs" type="text/javascript"
  async defer src="//platform.twitter.com/widgets.js"></script>

{{{tweets}}}
```

还有一个处理它的路由：

```
app.get('/social', async (req, res) => {
  res.render('social', { tweets: await getTopTweets() })
})
```

请注意，我们引用了外部脚本，Twitter 的 `widgets.js`。这个脚本将为页面上嵌入的推文格式化并提供功能。默认情况下，`oembed` API 将在 HTML 中包含对此脚本的引用，但由于我们要显示 10 条推文，这会比必要的引用该脚本多九次！因此，请回想一下，当我们调用 `oembed` API 时，我们传入了选项 `{ omit_script: 1 }`。由于我们这样做了，我们在视图中提供了它。尝试从视图中删除脚本。您仍然会看到推文，但它们将没有任何格式或功能。

现在我们有了一个不错的社交媒体信息流！让我们将注意力转向另一个重要应用程序：在我们的应用程序中显示地图。

# 地理编码

*地理编码* 指的是将街道地址或地名（Bletchley Park, Sherwood Drive, Bletchley, Milton Keynes MK3 6EB, UK）转换为地理坐标（纬度 51.9976597，经度 –0.7406863）的过程。如果您的应用程序需要进行任何形式的地理计算（距离或方向）或显示地图，则需要地理坐标。

###### 注意

您可能习惯于看到使用度分秒（DMS）指定的地理坐标。地理编码 API 和地图服务使用单个浮点数表示纬度和经度。如果您需要显示 DMS 坐标，请参阅 [此维基百科文章](http://bit.ly/2Xc5IlM)。

## 使用 Google 进行地理编码

Google 和 Bing 都提供优秀的地理编码 REST 服务。我们将在示例中使用 Google，但是 Bing 的服务非常类似。

如果您未将计费账户附加到您的 Google 账户，则您的地理编码请求将每天限制为一次，这将导致测试周期非常缓慢！在这本书中尽可能地，我试图避免推荐您至少在开发阶段不能免费使用的服务，并且我确实尝试了一些免费地理编码服务，并发现了足够大的可用性差距，因此我继续推荐 Google 地理编码。但是，就我所写的而言，使用 Google 进行开发量地理编码的成本是免费的：您的账户会获得每月 200 美元的信用额度，您需要做 40,000 次请求才能用尽！如果您想跟着本章进行，请转到 [您的 Google 控制台](http://bit.ly/2KcY1X0)，从主菜单中选择计费，并输入您的计费信息。

一旦设置了计费，您将需要 Google 地理编码 API 的 API 密钥。转到 [控制台](http://bit.ly/2KcY1X0)，从导航栏中选择您的项目，然后单击 API。如果地理编码 API 不在您启用的 API 列表中，请在附加 API 列表中找到并添加它。大多数 Google API 共享相同的 API 凭据，因此请单击左上角的导航菜单，返回到您的仪表板。单击凭据，如果您还没有合适的 API 密钥，则创建一个新的 API 密钥。请注意，API 密钥可以受限以防止滥用，因此请确保您的 API 密钥可以从您的应用程序中使用。如果您需要用于开发的密钥，您可以按 IP 地址限制密钥，并选择您的 IP 地址（如果您不知道它是什么，可以询问 Google，“我的 IP 地址是多少？”）。

一旦获得 API 密钥，请将其添加到 *.credentials.development.json*：

```
"google": {
  "apiKey": "<YOUR API KEY>"
}
```

然后创建一个模块 *lib/geocode.js*：

```
const https = require('https')
const { credentials } = require('../config')

module.exports = async query => {

  const options = {
    hostname: 'maps.googleapis.com',
    path: '/maps/api/geocode/json?address=' +
      encodeURIComponent(query) + '&key=' +
      credentials.google.apiKey,
  }

  return new Promise((resolve, reject) =>
    https.request(options, res => {
      let data = ''
      res.on('data', chunk => data += chunk)
      res.on('end', () => {
        data = JSON.parse(data)
        if(!data.results.length)
          return reject(new Error(`no results for "${query}"`))
        resolve(data.results[0].geometry.location)
      })
    }).end()
  )

}
```

现在我们有一个函数，将联系 Google API 对地址进行地理编码。如果找不到地址（或由于任何其他原因而失败），将返回错误。API 可以返回多个地址。例如，如果您搜索“10 Main Street”而没有指定城市、州或邮政编码，它将返回数十个结果。我们的实现只是选择第一个。API 返回大量信息，但目前我们感兴趣的只是坐标。您可以轻松修改此接口以返回更多信息。请参阅 [Google 地理编码 API 文档](http://bit.ly/2O4EE3t) 以了解 API 返回的更多信息。

### 使用限制

目前 Google 地理编码 API 每月有使用限制，但您每次地理编码请求支付 0.005 美元。因此，如果您在任何给定月份进行了百万次请求，您将从 Google 收到 5,000 美元的账单……因此，对您来说可能存在一个实际的限制！

###### 提示

如果您担心发生不必要的费用——如果您意外地让一个服务运行，或者如果一个不良分子获取了您的凭证，这可能会发生——您可以添加预算并配置警报，以在接近预算时通知您。转到您的 Google 开发者控制台，并从计费菜单中选择“预算和警报”。

在撰写本文时，Google 限制您在 100 秒内进行 5000 次请求，以防止滥用，这是很难超过的。Google 的 API 还要求，如果您在网站上使用地图，您必须使用 Google Maps。也就是说，如果您使用 Google 的服务来进行地理编码您的数据，您不能反过来在 Bing 地图上显示这些信息，否则将违反服务条款。一般来说，这不是一个繁琐的限制，因为您可能不会进行地理编码，除非您打算在地图上显示位置。但是，如果您更喜欢 Bing 的地图还是 Google 的地图，您应该注意服务条款，并使用适当的 API。

## 编码您的数据

我们拥有一个关于俄勒冈周围度假套餐的很好的数据库，我们可能决定要显示一个带有标注的地图，显示各种度假的位置，这就是地理编码的用武之地。

我们已经在数据库中有度假数据，每个度假都有一个位置搜索字符串，可以用于地理编码，但我们还没有坐标。

现在的问题是何时以及如何进行地理编码？总体而言，我们有三个选择：

+   当向数据库添加新的假期时进行地理编码。当我们向系统添加允许供应商动态添加假期到数据库的管理员界面时，这可能是一个很好的选择。然而，由于我们没有完成这个功能，因此我们将放弃这个选项。

+   检索数据库中的度假时根据需要进行地理编码。这种方法会在每次从数据库获取度假信息时进行检查：如果有任何度假信息缺少坐标，我们将对其进行地理编码。这个选项听起来很吸引人，可能是三个选项中最简单的一个，但它有一些重大的缺点，使其不适用。首先是性能问题：如果您向数据库添加了一千个新的度假信息，查看度假列表的第一个人将不得不等待所有这些地理编码请求成功并写入数据库。此外，可以想象一个负载测试套件向数据库添加了一千个度假信息，然后执行了一千个请求。由于它们都在同时运行，每一个请求都会导致一千个地理编码请求，因为数据还没有写入数据库……导致一百万个地理编码请求和来自 Google 的 5000 美元账单！因此，我们将这个选项划掉。

+   编写一个脚本来查找缺少坐标日期的假期，并对它们进行地理编码。这种方法为我们当前的情况提供了最佳解决方案。为了开发目的，我们一次性填充假期数据库，但我们还没有为添加新假期设计管理界面。此外，如果以后决定添加管理界面，这种方法与此并不冲突：事实上，我们只需在添加新假期后运行此流程，它就会正常工作。

首先，我们需要添加一种方法来更新 *db.js* 中的现有假期（我们还将添加一个关闭数据库连接的方法，在脚本中会很方便）：

```
module.exports = {
  //...
  updateVacationBySku: async (sku, data) => Vacation.updateOne({ sku }, data),
  close: () => mongoose.connection.close(),
}
```

然后我们可以编写一个脚本 *db-geocode.js*：

```
const db = require('./db')
const geocode = require('./lib/geocode')

const geocodeVacations = async () => {
  const vacations = await db.getVacations()
  const vacationsWithoutCoordinates = vacations.filter(({ location }) =>
    !location.coordinates || typeof location.coordinates.lat !== 'number')
  console.log(`geocoding ${vacationsWithoutCoordinates.length} ` +
    `of ${vacations.length} vacations:`)
  return Promise.all(vacationsWithoutCoordinates.map(async ({ sku, location }) => {
    const { search } = location
    if(typeof search !== 'string' || !/\w/.test(search))
      return console.log(`  SKU ${sku} FAILED: does not have location.search`)
    try {
      const coordinates = await geocode(search)
      await db.updateVacationBySku(sku, { location: { search, coordinates } })
      console.log(`  SKU ${sku} SUCCEEDED: ${coordinates.lat}, ${coordinates.lng}`)
    } catch(err) {
      return console.log(`  SKU {sku} FAILED: ${err.message}`)
    }
  }))
}

geocodeVacations()
  .then(() => {
    console.log('DONE')
    db.close()
  })
  .catch(err => {
    console.error('ERROR: ' + err.message)
    db.close()
  })
```

当您运行脚本 (`node db-geocode.js`) 时，您应该看到所有假期都已成功地进行了地理编码！现在我们有了这些信息，让我们学习如何在地图上显示它……

## 显示地图

虽然在地图上显示假期实际上属于“前端”工作，但到达这一步却看不到我们劳动成果将会非常令人失望。因此，我们将稍微偏离本书的后端重点，看看如何在地图上显示我们新编码的经销商。

我们已经创建了一个谷歌 API 密钥来进行地理编码，但我们仍然需要启用地图 API。前往 [您的谷歌控制台](http://bit.ly/2KcY1X0)，点击 APIs，找到 Maps JavaScript API 并启用（如果尚未启用）。

现在我们可以创建一个视图来显示我们的假期地图，*views/vacations-map.handlebars*。我们将从仅显示地图开始，并继续添加假期：

```
<div id="map" style="width: 100%; height: 60vh;"></div>
<script>
  let map = undefined
  async function initMap() {
    map = new google.maps.Map(document.getElementById('map'), {
      // approximate geographic center of oregon
      center: { lat: 44.0978126, lng: -120.0963654 },
      // this zoom level covers most of the state
      zoom: 7,
    })
  }
</script>
<script src="https://maps.googleapis.com/maps/api/js?key={{googleApiKey}}&callback=initMap"
    async defer></script>
```

现在是时候在地图上放置一些与我们的假期相对应的标记了。在 第十五章 中，我们创建了一个 API 端点 `/api/vacations`，现在将包含地理编码数据。我们将使用该端点获取我们的假期，并在地图上放置标记。修改 *views/vacations-map.handlebars.js* 中的 `initMap` 函数：

```
async function initMap() {
  map = new google.maps.Map(document.getElementById('map'), {
    // approximate geographic center of oregon
    center: { lat: 44.0978126, lng: -120.0963654 },
    // this zoom level covers most of the state
    zoom: 7,
  })
  const vacations = await fetch('/api/vacations').then(res => res.json())
  vacations.forEach(({ name, location }) => {
    const marker = new google.maps.Marker({
      position: location.coordinates,
      map,
      title: name,
    })
  })
}
```

现在我们有一张显示所有假期位置的地图了！我们可以通过很多方式来改进这个页面：可能最好的起点是链接标记与假期详细页面，这样您可以点击一个标记，它会带您到假期信息页面。我们还可以实现自定义标记或工具提示：谷歌地图 API 有很多功能，您可以从 [官方谷歌文档](https://developers.google.com/maps/documentation/javascript/tutorial) 中了解它们。

# 天气数据

还记得我们在 第七章 中的“当前天气”小部件吗？让我们连接一些实时数据！我们将使用美国国家气象局 (NWS) API 来获取预报信息。与我们的 Twitter 集成和地理编码一样，我们将缓存预报信息，以防止每次对我们网站的访问都通过 NWS（如果我们的网站变得流行可能会使我们被列入黑名单）。创建一个名为 *lib/weather.js* 的文件：

```
const https = require('https')
const { URL } = require('url')

const _fetch = url => new Promise((resolve, reject) => {
  const { hostname, pathname, search } = new URL(url)
  const options = {
    hostname,
    path: pathname + search,
    headers: {
      'User-Agent': 'Meadowlark Travel'
    },
  }
  https.get(options, res => {
    let data = ''
    res.on('data', chunk => data += chunk)
    res.on('end', () => resolve(JSON.parse(data)))
  }).end()
})

module.exports = locations => {

  const cache = {
    refreshFrequency: 15 * 60 * 1000,
    lastRefreshed: 0,
    refreshing: false,
    forecasts: locations.map(location => ({ location })),
  }

  const updateForecast = async forecast => {
    if(!forecast.url) {
      const { lat, lng } = forecast.location.coordinates
      const path = `/points/${lat.toFixed(4)},${lng.toFixed(4)}`
      const points = await _fetch('https://api.weather.gov' + path)
      forecast.url = points.properties.forecast
    }
    const { properties: { periods } } = await _fetch(forecast.url)
    const currentPeriod = periods[0]
    Object.assign(forecast, {
      iconUrl: currentPeriod.icon,
      weather: currentPeriod.shortForecast,
      temp: currentPeriod.temperature + ' ' + currentPeriod.temperatureUnit,
    })
    return forecast
  }

  const getForecasts = async () => {
    if(Date.now() > cache.lastRefreshed + cache.refreshFrequency) {
      console.log('updating cache')
      cache.refreshing = true
      cache.forecasts = await Promise.all(cache.forecasts.map(updateForecast))
      cache.refreshing = false
    }
    return cache.forecasts
  }

  return getForecasts

}
```

你会注意到，我们厌倦了直接使用 Node 内置的`https`库，而是创建了一个实用函数`_fetch`，使我们的天气功能更易读一些。可能会有一件事引起你的注意，那就是我们将`User-Agent`标头设置为`Meadowlark Travel`。这是 NWS 天气 API 的一个怪癖：它需要一个字符串作为`User-Agent`。他们声明最终将其替换为 API 密钥，但现在我们只需要在这里提供一个值。

从 NWS API 获取天气数据在这里是一个两步操作。有一个 API 端点叫做`points`，它接受一个带有精确四位小数的纬度和经度，并返回关于该位置的信息...包括获取预报的适当 URL。一旦我们对任何给定的坐标有了那个 URL，我们就不需要再次获取它。我们只需要调用那个 URL 来获取更新的预报。

注意，预报返回的数据比我们使用的要多得多；我们可以用这个特性做更多复杂的事情。特别是，预报的 URL 返回一个期间数组，第一个元素是当前期间（例如，“下午”或“晚上”），接着是延伸到下周的期间。可以随意查看`periods`数组中的数据，了解可用的数据类型。

我们的缓存中有一个名为`refreshing`的布尔属性，值得注意的细节。这是必需的，因为更新缓存需要一定的时间，并且是异步完成的。如果在第一次缓存刷新完成之前有多个请求进来，它们都会触发刷新缓存的工作。这不会造成实质性损害，但会多出一些不必要的 API 调用。这个布尔变量只是一个标志，用来告诉任何额外的请求：“我们正在处理中。”

我们设计它成为我们在第七章创建的虚拟函数的即插即用替代品。我们所要做的就是打开*lib/middleware/weather.js*，并替换`getWeatherData`函数：

```
const weatherData = require('../weather')

const getWeatherData = weatherData([
  {
    name: 'Portland',
    coordinates: { lat: 45.5154586, lng: -122.6793461 },
  },
  {
    name: 'Bend',
    coordinates: { lat: 44.0581728, lng: -121.3153096 },
  },
  {
    name: 'Manzanita',
    coordinates: { lat: 45.7184398, lng: -123.9351354 },
  },
])
```

现在我们的小部件中有了实时天气数据！

# 结论

我们实际上只是初步了解了与第三方 API 集成可能完成的事情的表面。无论你看哪里，新的 API 都在不断涌现，提供各种想得到的数据（甚至波特兰市现在也通过 REST API 提供了大量的公共数据）。虽然不可能涵盖你可以使用的 API 的一小部分，但本章涵盖了你需要了解的使用这些 API 的基础知识：`http.request`、`https.request`和解析 JSON。

现在我们掌握了很多知识。我们走了很长一段路！但是当事情出错时会发生什么呢？在下一章中，我们将讨论调试技术，帮助我们解决事情不如预期时的情况。
