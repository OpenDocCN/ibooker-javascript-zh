# 第十四章：路由

路由是你的网站或网络服务中最重要的方面之一；幸运的是，在 Express 中进行路由是简单、灵活和强大的。*路由*是通过 URL 和 HTTP 方法指定的请求被路由到处理它们的代码的机制。正如我们已经指出的那样，路由过去是基于文件且简单的。例如，如果你在网站上放置了文件*foo/about.html*，你可以通过浏览器的路径*/foo/about.html*访问它。简单但不灵活。而且，如果你还没有注意到，在你的 URL 中有*html*这个词现在已经非常过时了。

在我们深入讨论使用 Express 进行路由的技术细节之前，我们应该先讨论*信息架构*（IA）的概念。IA 指的是你内容的概念性组织。在你开始考虑路由之前，拥有一个可扩展（但不过于复杂）的 IA 将会在以后带来巨大的回报。

有关信息架构（IA）的最聪明和永恒的文章之一是由实际上*发明互联网*的 Tim Berners-Lee 所写的。你现在可以（也应该）阅读它：[*http://www.w3.org/Provider/Style/URI.html*](http://www.w3.org/Provider/Style/URI.html)。这篇文章写于 1998 年。让你沉思片刻；在 1998 年关于互联网技术写的东西，到今天依然有不多的能像它那样真实。

从那篇文章中，我们被要求承担起的崇高责任是：

> 作为网站管理员，你有责任为 URI 分配能够在 2 年、20 年甚至 200 年后仍然坚持的 URI。这需要思考、组织和承诺。
> 
> Tim Berners-Lee

如果网页设计需要像其他工程一样进行专业许可，我喜欢想象我们会为此效果发誓。（那篇文章的敏锐读者会发现，该文章的网址以*.html*结尾，这一点很有幽默感。）

打个比方（这可能对年轻观众来说有些难以理解），想象一下，每两年你最喜欢的图书馆都会完全重新排列杜威十进制系统。你某一天走进图书馆，却发现找不到任何东西。这正是当你重新设计 URL 结构时所发生的事情。

认真考虑你的 URL。20 年后它们是否还有意义？（200 年可能有些牵强：谁知道那时我们是否还会使用 URL 呢。但是，我钦佩那种能够这么远见的决心。）仔细考虑你内容的分类方式。逻辑地分类，并且尽量避免把自己限制在一个角落里。这既是一门科学，也是一门艺术。

或许最重要的是，与他人合作设计您的网址。即使您是方圆数英里内最好的信息架构师，您可能会惊讶地发现人们对相同内容有不同的看法。我并不是说您应该尝试制定一个从*所有人*的角度来看都有意义的 IA（因为这通常是不可能的），但能够从多个角度看问题会给您带来更好的想法，并暴露出您自己 IA 中的缺陷。

这里有一些建议，可帮助您实现持久的 IA：

永远不要在网址中公开技术细节

您是否曾经访问过一个网站，注意到网址以*.asp*结尾，并认为该网站已经过时了？请记住，曾经有一段时间，ASP 技术是尖端的。虽然我很痛苦地说，JavaScript、JSON、Node 和 Express 也将如此。希望它们能够多年多年地发挥作用，但时间对技术往往并不宽容。

避免在网址中包含无意义的信息

仔细考虑您网址中的每个单词。如果没有意义，就不要包括在内。例如，当网站在网址中使用“*home*”一词时，我总是感到不舒服。您的根网址*就是*您的主页。您不需要额外拥有像*/home/directions*和*/home/contact*这样的网址。

避免过长的网址

在所有事情都相等的情况下，短网址比长网址更好。然而，你不应该试图以牺牲清晰度或 SEO 为代价来缩短网址。缩写很诱人，但需要仔细考虑。它们在成为网址之前应该是常见且无处不在的。

请一致使用单词分隔符

使用连字符分隔单词相当普遍，使用下划线稍少见一些。连字符通常被认为比下划线更美观，大多数 SEO 专家推荐使用它们。无论您选择连字符还是下划线，都要在使用上保持一致。

永远不要在网址中使用空格或不可输入字符

不建议在网址中使用空白。通常会被转换为加号(+)，导致混乱。显然，您应该避免不可输入字符，并强烈警告您不要在网址中使用除字母、数字、短横线和下划线以外的任何字符。这在当时可能会感觉很聪明，但是“聪明”往往经不起时间的考验。显然，如果您的网站不是针对英语受众的，您可以使用非英语字符（使用百分比编码），尽管这可能在需要本地化您的网站时带来麻烦。

使用小写字母来编写您的网址

这个问题可能会引发一些争论。有些人认为在 URL 中使用大小写混合不仅可以接受，而且更可取。我不想在这个问题上进行宗教性的辩论，但我要指出小写的优势在于它可以始终通过代码自动生成。如果你曾经不得不处理成千上万个链接或进行字符串比较，你会欣赏这个论点。我个人觉得小写 URL 更具审美感，但最终，这个决定由你来做。

# 路径和 SEO

如果你希望你的网站被发现（大多数人都希望如此），那么你需要考虑 SEO 以及你的 URL 如何影响它。特别是，如果某些关键词很重要——*而且有意义*——考虑将它们作为 URL 的一部分。例如，Meadowlark Travel 提供多个俄勒冈海岸度假选择。为了确保这些度假选项在搜索引擎中排名靠前，我们在标题、头部、正文和元描述中使用“Oregon Coast”这个字符串，并且 URL 以*/vacations/oregon-coast*开头。曼扎尼塔度假套餐可以在*/vacations/oregon-coast/manzanita*找到。如果我们简单地使用*/vacations/manzanita*来缩短 URL，我们将会失去宝贵的 SEO 机会。

尽管如此，不要贸然将关键词塞进 URL 中以试图提高排名。这是行不通的。例如，试图将曼扎尼塔度假的 URL 更改为*/vacations/oregon-coast-portland-and-hood-river/oregon-coast/manzanita*，以此来多说一次“Oregon Coast”，同时还加入“Portland”和“Hood River”关键词，这是不合适的。这与良好的信息架构相悖，很可能会适得其反。

# 子域名

除了路径之外，子域名是常用于路由请求的 URL 的另一部分。子域名最好保留给应用程序的显著不同部分——例如，一个 REST API (*api.meadowlarktravel.com*)或管理员界面 (*admin.meadowlarktravel.com*)。有时候出于技术原因使用子域名。例如，如果我们用 WordPress 来构建我们的博客（而其余的网站使用 Express），使用*blog.meadowlarktravel.com*可能更容易（更好的解决方案是使用代理服务器，比如 NGINX）。通常通过使用子域名来分隔内容会对 SEO 产生影响，这就是为什么你通常应该将它们保留给对 SEO 不重要的站点区域，比如管理区域和 API。记住这一点，并确保在将子域名用于对 SEO 计划重要的内容之前，没有其他更好的选择。

Express 中的路由机制默认不考虑子域：`app.get(*/about*)`将处理关于 *http://meadowlarktravel.com/about*、*http://www.meadowlarktravel.com/about* 和 *http://admin.meadowlarktravel.com/about* 的请求。如果你想单独处理子域，可以使用一个名为`vhost`（代表“虚拟主机”，这来自于经常用于处理子域的 Apache 机制）的包。首先，安装这个包（`npm install vhost`）。要在开发机上测试基于域的路由，你需要一种“伪造”域名的方法。幸运的是，这正是你的*hosts 文件*需要做的。在 macOS 和 Linux 机器上，它可以在*/etc/hosts*中找到，在 Windows 上，它位于*c:\windows\system32\drivers\etc\hosts*。在你的 hosts 文件中添加以下内容（你需要管理权限来编辑它）：

```
127.0.0.1 admin.meadowlark.local
127.0.0.1 meadowlark.local
```

这告诉你的计算机将`meadowlark.local`和`admin.meadowlark.local`视为常规的互联网域，但将它们映射到本地主机（127.0.0.1）。我们使用`.local`顶级域，这样就不会混淆（你可以使用`.com`或任何其他互联网域，但它会覆盖真实域名，这可能会导致挫折）。

然后你可以使用`vhost`中间件来使用基于域的路由（*伴随仓库中的 ch14/00-subdomains.js*）：

```
// create "admin" subdomain...this should appear
// before all your other routes
var admin = express.Router()
app.use(vhost('admin.meadowlark.local', admin))

// create admin routes; these can be defined anywhere
admin.get('*', (req, res) => res.send('Welcome, Admin!'))

// regular routes
app.get('*', (req, res) => res.send('Welcome, User!'))
```

`express.Router()`本质上创建了一个新的 Express 路由的实例。你可以像处理`app`一样处理这个实例。你可以添加路由和中间件，就像对`app`做的一样。然而，在你将其添加到`app`之前，它不会做任何事情。我们通过`vhost`来添加它，将该路由实例绑定到子域。

###### 提示

`express.Router`还可以用于分区你的路由，这样你可以一次链接多个路由处理程序。查看[Express 路由文档](http://bit.ly/2X8VC59)以获取更多信息。

# 路由处理程序就是中间件

我们已经看到了匹配给定路径的基本路由。但`app.get(\'/foo', ...)`实际上做了什么呢？正如我们在第十章中所看到的，它只是一种专门的中间件，甚至有一个传入的`next`方法。让我们看一些更复杂的例子（*伴随仓库中的 ch14/01-fifty-fifty.js*）：

```
app.get('/fifty-fifty', (req, res, next) => {
  if(Math.random() < 0.5) return next()
  res.send('sometimes this')
})
app.get('/fifty-fifty', (req,res) => {
  res.send('and sometimes that')
})
```

在上一个示例中，我们为相同的路由设置了两个处理程序。通常情况下，第一个处理程序会胜出，但在这种情况下，第一个处理程序将有大约一半的机会失败，给第二个处理程序一个机会。我们甚至不必两次使用 `app.get`：你可以为单个`app.get`调用使用多个处理程序。以下是一个示例，它有大约等概率的三种不同的响应（*伴随仓库中的 ch14/02-red-green-blue.js*）：

```
app.get('/rgb',
  (req, res, next) => {
    // about a third of the requests will return "red"
    if(Math.random() < 0.33) return next()
    res.send('red')
  },
  (req, res, next) => {
    // half of the remaining 2/3 of requests (so another third)
    // will return "green"
    if(Math.random() < 0.5) return next()
    res.send('green')
  },
  function(req, res){
    // and the last third returns "blue"
    res.send('blue')
  },
)
```

虽然这一开始看起来可能并不特别有用，但它允许你创建能够在任何路由中使用的通用函数。例如，假设我们有一种机制，在特定页面上显示特别优惠。这些特别优惠经常变动，并非所有页面都显示。我们可以创建一个函数将特别优惠注入到`res.locals`属性中（你会在第七章中记得此属性）（伴随版本库中的 *ch14/03-specials.js*）。

```
async function specials(req, res, next) {
  res.locals.special = await getSpecialsFromDatabase()
  next()
}

app.get('/page-with-specials', specials, (req, res) =>
  res.render('page-with-specials')
)
```

我们还可以使用这种方法实现授权机制。假设我们的用户授权代码设置了一个名为 `req.session.authorized` 的会话变量。我们可以使用以下方法来创建一个可重用的授权过滤器（伴随版本库中的 *ch14/04-authorizer.js*）。

```
function authorize(req, res, next) {
  if(req.session.authorized) return next()
  res.render('not-authorized')
}

app.get('/public', () => res.render('public'))

app.get('/secret', authorize, () => res.render('secret'))
```

# 路由路径和正则表达式

当你在路由中指定一个路径（比如 */foo*）时，Express 最终将其转换为正则表达式。路由路径中可用一些正则表达式元字符：`+`, `?`, `*`, `(` 和 `)`。让我们看一些示例。假设你希望处理 */user* 和 */username* 这两个 URL 的路由相同：

```
app.get('/user(name)?', (req, res) => res.render('user'))
```

我最喜欢的一个新奇网站——现在可惜已经倒闭了——是 *http://khaaan.com*。那只是每个人最喜欢的星际舰队长高呼他最具代表性的台词。毫无用处，但每次看到都会让我微笑。假设我们想要创建我们自己的“KHAAAAAAAAN”页面，但我们不想让用户记住是 2 个 *a* 还是 3 个或 10 个。下面的方法可以实现：

```
app.get('/khaa+n', (req, res) => res.render('khaaan'))
```

并非所有常规正则表达式元字符在路由路径中都有意义，只有之前列出的那些元字符才有意义。这一点很重要，因为通常情况下，点号作为正则表达式元字符表示“任何字符”，可以在路由中未经转义地使用。

最后，如果你确实需要路由的完整正则表达式的全部功能，Express 也支持。

```
app.get(/crazy|mad(ness)?|lunacy/, (req,res) =>
  res.render('madness')
)
```

我还没有找到在我的路由路径中使用正则表达式元字符，更不用说完整的正则表达式的充分理由，但了解这些功能是存在的是很好的。

# 路由参数

虽然在 Express 工具箱中，正则表达式路由可能很少在日常工作中使用，但你很可能会经常使用路由参数。简而言之，这是一种将路由的一部分转换为变量参数的方法。假设在我们的网站中，我们想为每位员工创建一个页面。我们有一个带有生平和照片的员工数据库。随着公司的不断发展，每增加一位员工，为其增加一个新的路由变得越来越困难。让我们看看路由参数如何帮助我们（伴随版本库中的 *ch14/05-staff.js*）。

```
const staff = {
  mitch: { name: "Mitch",
    bio: 'Mitch is the man to have at your back in a bar fight.' },
  madeline: { name: "Madeline", bio: 'Madeline is our Oregon expert.' },
  walt: { name: "Walt", bio: 'Walt is our Oregon Coast expert.' },
}

app.get('/staff/:name', (req, res, next) => {
  const info = staff[req.params.name]
  if(!info) return next()   // will eventually fall through to 404
  res.render('05-staffer', info)
})
```

注意我们在路由中使用了 *:name*。这将匹配任意字符串（不包括斜杠）并将其放入 `req.params` 对象的键 `name` 中。这是一个我们会经常使用的功能，特别是在创建 REST API 时。我们的路由中可以有多个参数。例如，如果我们想要按城市列出我们的员工名单，我们可以使用如下方式：

```
const staff = {
  portland: {
    mitch: { name: "Mitch", bio: 'Mitch is the man to have at your back.' },
    madeline: { name: "Madeline", bio: 'Madeline is our Oregon expert.' },
  },
  bend: {
    walt: { name: "Walt", bio: 'Walt is our Oregon Coast expert.' },
  },
}

app.get('/staff/:city/:name', (req, res, next) => {
  const cityStaff = staff[req.params.city]
  if(!cityStaff) return next()  // unrecognized city -> 404
  const info = cityStaff[req.params.name]
  if(!info) return next()       // unrecognized staffer -> 404
  res.render('staffer', info)
})
```

# 组织路由

对您来说可能已经很清楚，在主应用程序文件中定义所有路由将变得难以管理。不仅这个文件会随着时间的推移而增长，而且它也不是一个良好的功能分离，因为该文件中已经有很多内容。一个简单的网站可能只有十几个或更少的路由，但一个更大的网站可能有数百个路由。

那么如何组织您的路由呢？好吧，您想如何组织您的路由？Express 并不关心您如何组织您的路由，因此您可以做的只受您自己想象的限制。

在接下来的章节中，我将介绍一些处理路由的流行方法，但归根结底，我建议为确定如何组织您的路由提供四个指导原则：

使用命名函数作为路由处理程序

写内联的路由处理程序，通过直接定义处理路由的函数来进行实际定义，对于小应用程序或原型设计是可以的，但随着您的网站增长，它将很快变得难以管理。

路由不应是神秘的

这个原则有意地模糊，因为一个大型复杂的网站可能需要比一个 10 页网站更复杂的组织方案。在光谱的一端是将您网站的所有路由都放在一个单一文件中，这样您就知道它们在哪里。对于大型网站来说，这可能是不理想的，因此您可以按功能区域拆分路由。然而，即使这样，应该清楚应该去哪里查找特定的路由。当您需要修复某些内容时，您最不希望做的就是花一个小时去找出路由处理的位置。我在工作中有一个 ASP.NET MVC 项目在这方面是一个噩梦。路由在至少 10 个不同的地方处理，这既不合逻辑也不一致，经常是相互矛盾的。即使我对那个（非常大的）网站非常熟悉，我仍然不得不花费大量时间去追踪某些 URL 的处理位置。

路由组织应该是可扩展的

如果您现在有 20 或 30 个路由，将它们全部定义在一个文件中可能就可以了。但是三年后当您有 200 个路由时呢？这是有可能的。无论您选择哪种方法，都应确保您有足够的空间来扩展。

不要忽视自动视图路由处理程序

如果您的网站包含许多静态页面并且具有固定的 URL，那么您的所有路由最终都会看起来像这样：`app.get('/static/thing', (req, res) => res.render(\'static/thing'))`。为了减少不必要的代码重复，考虑使用自动视图路由处理程序。本章稍后将介绍这种方法，并可与自定义路由一起使用。

# 在一个模块中声明路由

组织路由的第一步是将它们全部放入它们自己的模块中。有多种方法可以做到这一点。一种方法是使您的模块返回一个包含方法和处理程序属性的对象数组。然后您可以在应用程序文件中定义路由如下：

```
const routes = require('./routes.js')

routes.forEach(route => approute.method)
```

这种方法有其优势，并且非常适合动态存储我们的路由，比如存储在数据库或 JSON 文件中。然而，如果你不需要这种功能，我建议将 `app` 实例传递给模块，让它添加路由。这是我们示例中采用的方法。创建一个名为 *routes.js* 的文件，并将所有现有的路由移到其中：

```
module.exports = app => {

  app.get('/', (req,res) => app.render('home'))

  //...

}
```

如果我们只是简单地复制粘贴，可能会遇到一些问题。例如，如果我们有使用新上下文中不可用的变量或方法的内联路由处理程序，那么这些引用现在将会失效。我们可以添加必要的导入，但先暂停。我们很快将把处理程序移到它们自己的模块中，并在那时解决这个问题。

如何将我们的路由链接起来？简单：在 *meadowlark.js* 中，我们只需导入我们的路由：

```
require('./routes')(app)
```

或者我们可以更加显式，添加一个命名导入（我们将其命名为 `addRoutes`，以更好地反映它作为函数的性质；如果需要，我们也可以将文件命名为这样）：

```
const addRoutes = require('./routes')

addRoutes(app)
```

# 逻辑分组处理程序

为了符合我们的第一个指导原则（为路由处理程序使用命名函数），我们需要一个地方来放置这些处理程序。一个相当极端的选择是为每个处理程序单独创建一个 JavaScript 文件。我很难想象这种方法会有什么好处。最好是以某种方式将相关功能组合在一起。这不仅使共享功能更容易利用，还使相关方法的更改更容易。

目前，让我们将功能分组到不同的文件中：*handlers/main.js*，我们将在其中放置主页处理程序，“关于”处理程序，以及通常没有其他逻辑归属的任何处理程序；*handlers/vacations.js*，用于处理与假期相关的处理程序；等等。

考虑 *handlers/main.js*：

```
const fortune = require('../lib/fortune')

exports.home = (req, res) => res.render('home')

exports.about = (req, res) => {
  const fortune = fortune.getFortune()
  res.render('about', { fortune })
}

//...
```

现在让我们修改 *routes.js* 来利用这一点：

```
const main = require('./handlers/main')

module.exports = function(app) {

  app.get('/', main.home)
  app.get('/about', main.about)
  //...

}
```

这满足了我们所有的指导原则。*/routes.js* 非常简单直接。一眼就能看到网站中的所有路由以及它们的处理位置。我们还留了足够的空间来扩展。我们可以将相关功能分组到任意数量的不同文件中。如果 *routes.js* 变得过于复杂，我们可以再次使用相同的技术，将 `app` 对象传递给另一个模块，然后注册更多的路由（尽管这开始变得“过于复杂”——确保你确实能够证明这样一个复杂的方法是有必要的！）。

# 自动渲染视图

如果你发现自己想要回到过去的日子，当你只需将 HTML 文件放入目录中，然后——神奇地！——你的网站就会提供它，那么你并不孤单。如果你的网站内容丰富，但功能不多，你可能会觉得为每个视图添加路由是一种不必要的麻烦。幸运的是，我们可以解决这个问题。

假设您想要添加文件*views/foo.handlebars*并在路由*/foo*上自动使其可用。让我们看看我们可以如何做到这一点。在我们的应用程序文件中，在 404 处理程序之前，添加以下中间件（在配套存储库中的*ch14/06-auto-views.js*）：

```
const autoViews = {}
const fs = require('fs')
const { promisify } = require('util')
const fileExists = promisify(fs.exists)

app.use(async (req, res, next) => {
  const path = req.path.toLowerCase()
  // check cache; if it's there, render the view
  if(autoViews[path]) return res.render(autoViews[path])
  // if it's not in the cache, see if there's
  // a .handlebars file that matches
  if(await fileExists(__dirname + '/views' + path + '.handlebars')) {
    autoViews[path] = path.replace(/^\//, '')
    return res.render(autoViews[path])
  }
  // no view found; pass on to 404 handler
  next()
})
```

现在我们只需在*view*目录中添加一个*.handlebars*文件，并在适当的路径上进行神奇渲染即可。请注意，常规路由将绕过此机制（因为我们将自动视图处理程序放置在所有其他路由之后），因此，如果您有一个为路由*/foo*渲染不同视图的路由，那么它将优先处理。

请注意，如果您*删除*了访问过的视图，这种方法将会遇到问题；它将被添加到`autoViews`对象中，因此后续的视图尝试渲染它，即使它已被删除，也会导致错误。可以通过在`try/catch`块中包装渲染并在发现错误时从`autoViews`中删除该视图来解决此问题；我将把这个增强功能留给读者来练习。

# 结论

路由是项目中的重要部分，比这里概述的组织路由处理程序的方法更多，所以请随意尝试并找到适合您和您的项目的技术。我鼓励您偏爱清晰且易于跟踪的技术。路由在很大程度上是从外部世界（通常是浏览器的客户端）到响应它的服务器端代码的地图。如果这张地图复杂混乱，将会使您难以追踪应用程序中的信息流，这将影响开发和调试。
