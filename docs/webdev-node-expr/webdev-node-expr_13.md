# 第十三章：持久性

几乎所有除了最简单的网站和网络应用程序外，都需要某种形式的**持久性**；也就是说，一些比易失性内存更持久的数据存储方式，以便在服务器崩溃、停电、升级和迁移时数据能够存活下来。在本章中，我们将讨论持久性的可用选项，并演示文档数据库和关系数据库。然而，在进入数据库之前，我们将从最基本的持久性形式开始：文件系统持久性。

# 文件系统持久性

实现持久性的一种方式是简单地将数据保存到所谓的平面文件中（*平面*因为文件中没有固有的结构；它只是一系列字节的序列）。Node 通过`fs`（文件系统）模块实现了文件系统持久性。

文件系统持久性具有一些缺点。特别是，它不易扩展。一旦您需要多台服务器来满足流量需求，您将会遇到文件系统持久性的问题，除非所有服务器都可以访问共享文件系统。此外，由于平面文件没有固有结构，定位、排序和过滤数据的负担将落在您的应用程序上。因此，出于这些原因，您应优先考虑使用数据库而不是文件系统来存储数据。唯一的例外是存储二进制文件，如图像、音频文件或视频。虽然许多数据库可以处理这类数据，但它们很少比文件系统更有效地处理（尽管通常在数据库中存储有关二进制文件的信息以便于搜索、排序和过滤）。

如果您确实需要存储二进制数据，请记住文件系统存储仍然存在不易扩展的问题。如果您的主机没有访问共享文件系统的权限（通常是这种情况），您应考虑将二进制文件存储在数据库中（通常需要一些配置，以防止数据库停滞）或云存储服务，如 Amazon S3 或 Microsoft Azure Storage。

现在我们解决了注意事项，让我们来看看 Node 的文件系统支持。我们将回顾来自第八章的度假照片竞赛。在我们的应用程序文件中，让我们填写处理该表单的处理程序（*ch13/00-mongodb/lib/handlers.js*在伴随代码库中）：

```
const pathUtils = require('path')
const fs = require('fs')

// create directory to store vacation photos (if it doesn't already exist)
const dataDir = pathUtils.resolve(__dirname, '..', 'data')
const vacationPhotosDir = pathUtils.join(dataDir, 'vacation-photos')
if(!fs.existsSync(dataDir)) fs.mkdirSync(dataDir)
if(!fs.existsSync(vacationPhotosDir)) fs.mkdirSync(vacationPhotosDir)

function saveContestEntry(contestName, email, year, month, photoPath) {
  // TODO...this will come later
}

// we'll want these promise-based versions of fs functions later
const { promisify } = require('util')
const mkdir = promisify(fs.mkdir)
const rename = promisify(fs.rename)

exports.api.vacationPhotoContest = async (req, res, fields, files) => {
  const photo = files.photo[0]
  const dir = vacationPhotosDir + '/' + Date.now()
  const path = dir + '/' + photo.originalFilename
  await mkdir(dir)
  await rename(photo.path, path)
  saveContestEntry('vacation-photo', fields.email,
    req.params.year, req.params.month, path)
  res.send({ result: 'success' })
}
```

这里涉及很多内容，让我们来分解一下。首先，我们创建一个目录来存储上传的文件（如果目录不存在）。您可能希望将 *data* 目录添加到您的 *.gitignore* 文件中，以防意外提交上传的文件。回想一下 第八章 中，我们在 *meadowlark.js* 中处理实际文件上传，并已解码调用我们的处理程序。我们得到的是一个包含有关上传文件信息的对象 (`files`)。因为我们希望避免碰撞，所以不能仅仅使用用户上传的文件名（以防两个用户都上传 *portland.jpg*）。为了避免这个问题，我们基于时间戳创建一个唯一的目录；两个用户在同一毫秒内都上传 *portland.jpg* 的可能性非常小！然后我们将上传的文件重命名（移动）到我们构造的名称（我们的文件处理器会给它一个临时名称，我们可以从 `path` 属性中获取）。

最后，我们需要一种方法将用户上传的文件与他们的电子邮件地址（以及提交的月份和年份）关联起来。我们可以将这些信息编码到文件或目录名称中，但我们更倾向于将这些信息存储在数据库中。因为我们还没有学会如何做到这一点，我们将在本章后面的 `vacationPhotoContest` 函数中封装该功能并完成它。

###### 注意

一般情况下，你绝对不应该信任用户上传的任何内容，因为这是攻击你的网站的一个潜在途径。例如，恶意用户可以轻易地将一个有害的可执行文件改名为 *.jpg* 扩展名，并上传它作为攻击的第一步（希望以后能找到某种方法来执行它）。同样，我们在这里使用浏览器提供的 `name` 属性来命名文件，也存在一定的风险；某些人也可以通过在文件名中插入特殊字符来滥用这一点。为了让这段代码完全安全，我们将为文件命名随机生成一个名称，仅保留扩展名（确保它只包含字母和数字字符）。

尽管文件系统持久性具有其缺点，但它经常用于中间文件存储，了解如何使用 Node 文件系统库是很有用的。然而，为了解决文件系统存储的缺陷，让我们将注意力转向云持久性。

# 云持久性

云存储变得越来越流行，我强烈建议您利用其中一种价格便宜且稳健的服务。

使用云服务时，您必须进行一定量的前期工作。显然，您需要创建一个帐户，但您还需要了解您的应用程序如何与云服务进行身份验证，了解一些基本术语也很有帮助（例如，AWS 将其文件存储机制称为*存储桶*，而 Azure 称之为*容器*）。详细信息超出本书的范围，并且有充分的文档支持：

+   查看[AWS：Node.js 入门](https://amzn.to/2CCYk9s)获取更多信息。

+   [Azure 适用于 JavaScript 和 Node.js 开发人员](http://bit.ly/2NEkTku)

一旦您完成初始配置，使用云持久性就非常简单。以下是一个示例，展示了将文件保存到 Amazon S3 帐户有多么简单：

```
const filename = 'customerUpload.jpg'

s3.putObject({
  Bucket: 'uploads',
  Key: filename,
  Body: fs.readFileSync(__dirname + '/tmp/ + filename),
})
```

查看[AWS SDK 文档](https://amzn.to/2O3e1MA)获取更多信息。

以下是如何在 Microsoft Azure 上执行相同操作的示例：

```
const filename = 'customerUpload.jpg'

const blobService = azure.createBlobService()
blobService.createBlockBlobFromFile('uploads', filename, __dirname +
  '/tmp/' + filename)
```

查看[Microsoft Azure 文档](http://bit.ly/2Kd3rRK)获取更多信息。

现在我们已经了解了几种文件存储技术，让我们考虑使用数据库存储结构化数据的方法。

# 数据库持久性

几乎所有不是最简单的网站和 Web 应用程序都需要一个数据库。即使您的大部分数据是二进制的，而且您使用共享文件系统或云存储，您也很可能需要一个数据库来帮助目录化这些二进制数据。

传统上，“数据库”一词是“关系数据库管理系统”（RDBMS）的简称。关系数据库，如 Oracle、MySQL、PostgreSQL 或 SQL Server，基于数十年的研究和正式的数据库理论。这项技术现在已经非常成熟，这些数据库的强大是毋庸置疑的。然而，我们现在有幸扩展我们对数据库构成的理解。近年来，NoSQL 数据库变得流行起来，它们挑战了互联网数据存储的现状。

声称 NoSQL 数据库在某种程度上比关系数据库更好是愚蠢的，但它们确实具有某些优势（反之亦然）。虽然将关系数据库与 Node 应用程序集成非常容易，但也有些 NoSQL 数据库似乎几乎是为 Node 设计的。

最流行的两种 NoSQL 数据库类型是*文档数据库*和*键值数据库*。文档数据库擅长存储对象，这使它们非常适合 Node 和 JavaScript。键值数据库正如其名称所示，非常简单，非常适合具有易于映射为键值对的数据模式的应用程序。

我认为文档数据库代表了在关系数据库的约束和键值数据库的简单性之间找到的最佳折衷方案，因此，我们将在第一个示例中使用文档数据库。MongoDB 是主流的文档数据库，在目前是健壮和成熟的。

对于我们的第二个示例，我们将使用 PostgreSQL，这是一个流行且强大的开源关系型数据库管理系统。

## 关于性能的注记

NoSQL 数据库的简单性是双刃剑。仔细规划关系型数据库可能是一个复杂的任务，但仔细规划的好处是提供了性能优异的数据库。不要被误导认为，因为 NoSQL 数据库通常更简单，就不存在调整它们以获得最大性能的艺术和科学。

传统上，关系型数据库依赖其严格的数据结构和数十年的优化研究来实现高性能。另一方面，NoSQL 数据库采纳了互联网的分布式特性，并像 Node 一样，转而专注于并发以提升性能（关系型数据库也支持并发，但通常仅用于最苛刻的应用程序）。

规划数据库性能和可扩展性是一个庞大且复杂的主题，超出了本书的范围。如果您的应用程序需要高水平的数据库性能，我建议首先阅读 Kristina Chodorow 和 Michael Dirolf 的*[MongoDB 权威指南](http://bit.ly/Mongo_DB_Guide)*（O’Reilly）。

## 抽象化数据库层

在本书中，我们将实施相同的功能，并展示如何在两种数据库中执行（不仅仅是两种数据库，而是两种基本不同的数据库架构）。虽然本书的目标是涵盖两种流行的数据库架构选择，但它反映了现实场景：在项目进行中切换 Web 应用程序的主要组件。这可能出于许多原因。通常归结为发现不同技术能更具成本效益，或者允许您更快地实施必要的功能。

在可能的情况下，*抽象*您的技术选择是有价值的，这指的是编写某种 API 层以泛化底层技术选择。如果做得好，它会减少替换问题组件的成本。然而，这是有代价的：编写抽象层是您必须编写和维护的另一件事情。

幸运的是，我们的抽象化层将非常小，因为我们只支持本书目的的一小部分功能。目前，这些功能将如下：

+   从数据库返回一个活动度假列表

+   存储希望在特定度假季节通知时通知的用户的电子邮件地址

尽管这看起来足够简单，但这里有很多细节。度假是什么样子的？我们总是希望从数据库获取所有假期吗？还是希望能够过滤或分页它们？我们如何识别度假？等等。

我们将保持本书中抽象化层的简单性。我们将其包含在一个名为*db.js*的文件中，该文件将导出两个方法，我们将从简单提供虚拟实现开始：

```
module.exports = {
  getVacations: async (options = {}) => {
    // let's fake some vacation data:
    const vacations = [
      {
        name: 'Hood River Day Trip',
        slug: 'hood-river-day-trip',
        category: 'Day Trip',
        sku: 'HR199',
        description: 'Spend a day sailing on the Columbia and ' +
          'enjoying craft beers in Hood River!',
        location: {
          // we'll use this for geocoding later in the book
          search: 'Hood River, Oregon, USA',
        },
        price: 99.95,
        tags: ['day trip', 'hood river', 'sailing', 'windsurfing', 'breweries'],
        inSeason: true,
        maximumGuests: 16,
        available: true,
        packagesSold: 0,
      }
    ]
    // if the "available" option is specified, return only vacations that match
    if(options.available !== undefined)
      return vacations.filter(({ available }) => available === options.available)
    return vacations
  },
  addVacationInSeasonListener: async (email, sku) => {
    // we'll just pretend we did this...since this is
    // an async function, a new promise will automatically
    // be returned that simply resolves to undefined
  },
}
```

这为我们的数据库实现向应用程序展示了一个期望……而我们所要做的就是使我们的数据库符合这个接口。请注意，我们引入了“可用性”的概念；我们这样做是为了能够临时禁用假期而不是从数据库中删除它们。一个示例用例是一家小旅馆通知您他们关闭几个月进行翻新。我们将这与“旺季”概念分开，因为我们可能希望在网站上列出淡季假期，因为人们喜欢提前计划。

我们还包含一些非常通用的“位置”信息；我们将在第十九章中详细介绍这一点。

现在我们已经为我们的数据库层建立了一个抽象的基础，让我们看看如何使用 MongoDB 实现数据库存储。

## 设置 MongoDB

设置 MongoDB 实例的难度取决于您的操作系统。因此，我们将通过使用一个出色的免费 MongoDB 托管服务 mLab 来彻底避开这个问题。

###### 注意

mLab 并不是唯一的 MongoDB 服务提供商。MongoDB 公司现在通过其产品[MongoDB Atlas](https://www.mongodb.com)免费和低成本提供数据库托管服务。虽然免费账户不建议用于生产目的。mLab 和 MongoDB Atlas 都提供生产就绪的账户，因此在做选择之前应该了解它们的定价。当您转向生产环境时，与相同的托管服务保持一致将会更加方便。

使用 mLab 开始是很简单的。只需访问[*https://mlab.com*](https://mlab.com)，然后点击注册。填写注册表格并登录，您将会进入您的主页。在数据库下，您会看到“此时没有数据库”。点击“创建新数据库”，您将被带到一个包含一些选项的页面。您首先要选择一个云提供商。对于免费（沙盒）账户，选择大体上无关紧要，尽管您应该选择靠近您的数据中心（然而，并非每个数据中心都提供沙盒账户）。选择 SANDBOX，然后选择一个区域。然后选择一个数据库名称，点击提交订单（即使它是免费的，也还是一个订单！）。您将被带回到您的数据库列表，并在几秒钟后，您的数据库将可供使用。

拥有一个设置好的数据库是成功的一半。现在我们必须知道如何使用 Node 访问它，这就是 Mongoose 的用武之地。

## Mongoose

虽然有一个低级别的[MongoDB 驱动程序](http://bit.ly/2Kfw0hE)可用，但您可能希望使用对象文档映射器（ODM）。对于 MongoDB 来说，最流行的 ODM 是*Mongoose*。

JavaScript 的一个优点是其对象模型非常灵活。如果你想给一个对象添加属性或方法，你只需要这么做，而不需要担心修改类。不过，这种随意的灵活性可能会对你的数据库产生负面影响，因为它们可能变得碎片化并且难以优化。Mongoose 试图通过引入*模式*和*模型*来取得平衡（结合起来，模式和模型类似于传统面向对象编程中的类）。这些模式灵活但仍为你的数据库提供了一些必要的结构。

在我们开始之前，我们需要安装 Mongoose 模块：

```
npm install mongoose
```

然后我们将我们的数据库凭据添加到我们的 *.credentials.development.json* 文件中：

```
"mongo": {
    "connectionString": "your_dev_connection_string"
  }
}
```

你可以在 mLab 的数据库页面上找到连接字符串。从你的主屏幕上，点击相应的数据库。你会看到一个框，里面有你的 MongoDB 连接 URI（它以 *mongodb://* 开头）。你还需要一个数据库用户。要创建一个用户，点击 Users，然后选择“Add database user”。

注意，我们可以通过创建一个 *.credentials.production.js* 文件并使用 `NODE_ENV=production` 来为生产环境建立第二组凭据；在上线时你会需要这样做！

现在我们所有的配置都完成了，让我们实际连接到数据库并做一些有用的事情！

## 使用 Mongoose 进行数据库连接

我们将从创建到数据库的连接开始。我们将把我们的数据库初始化代码放在 *db.js* 中，与我们之前创建的虚拟 API 一起（在伴随代码库中为 *ch13/00-mongodb/db.js*）：

```
const mongoose = require('mongoose')
const { connectionString } = credentials.mongo
if(!connectionString) {
  console.error('MongoDB connection string missing!')
  process.exit(1)
}
mongoose.connect(connectionString)
const db = mongoose.connection
db.on('error' err => {
  console.error('MongoDB error: ' + err.message)
  process.exit(1)
})
db.once('open', () => console.log('MongoDB connection established'))

module.exports = {
  getVacations: async () => {
    //...return fake vacation data
  },
  addVacationInSeasonListener: async (email, sku) => {
    //...do nothing
  },
}
```

任何需要访问数据库的文件都可以简单地导入 *db.js*。然而，我们希望初始化尽快完成，即在我们需要 API 之前，所以我们会从 *meadowlark.js* 中导入它（在那里我们不需要对 API 做任何事情）：

```
require('./db')
```

现在我们正在连接到数据库，是时候考虑我们将如何结构化我们传输到数据库和从数据库传输的数据了。

## 创建模式和模型

让我们为 Meadowlark Travel 创建一个度假套餐数据库。我们首先定义一个模式并从中创建一个模型。创建文件 *models/vacation.js*（在伴随代码库中为 *ch13/00-mongodb/models/vacation.js*）：

```
const mongoose = require('mongoose')

const vacationSchema = mongoose.Schema({
  name: String,
  slug: String,
  category: String,
  sku: String,
  description: String,
  location: {
    search: String,
    coordinates: {
      lat: Number,
      lng: Number,
    },
  },
  price: Number,
  tags: [String],
  inSeason: Boolean,
  available: Boolean,
  requiresWaiver: Boolean,
  maximumGuests: Number,
  notes: String,
  packagesSold: Number,
})

const Vacation = mongoose.model('Vacation', vacationSchema)
module.exports = Vacation
```

这段代码声明了构成我们度假模型的属性及其类型。你会看到有几个字符串属性，一些数值属性，两个布尔属性以及一个字符串数组（用 `[String]` 表示）。在这一点上，我们还可以在我们的模式上定义方法。每个产品都有一个库存单位 (SKU)；即使我们不认为度假是“库存商品”，但 SKU 的概念在会计中是非常标准的，即使没有实体商品出售时也是如此。

一旦有了模式，我们就可以使用 `mongoose.model` 创建一个模型：此时，`Vacation` 就像传统面向对象编程中的类一样。请注意，在创建模型之前必须定义我们的方法。

###### 注意

由于浮点数的性质，在 JavaScript 中进行财务计算时一定要小心。我们可以将价格存储为分而不是美元，这会有所帮助，但并不能完全消除问题。对于我们旅行网站的适度目的，我们不打算担心这些，但是如果您的应用涉及非常大或非常小的财务金额（例如利息的分数美分或交易量），您应该考虑使用类似 [currency.js](https://currency.js.org) 或 [decimal.js-light](http://bit.ly/2X6kbQ5) 的库。此外，JavaScript 的 [BigInt](https://mzl.la/2Xhs45r) 内置对象，从 Node 10 开始可用（写作时具有有限的浏览器支持），可以用于此目的。

我们正在导出由 Mongoose 创建的 `Vacation` 模型对象。虽然我们可以直接使用这个模型，但这将削弱我们提供数据库抽象层的努力。因此，我们选择仅从 *db.js* 文件导入它，并让我们的应用程序的其余部分使用其方法。将 `Vacation` 模型添加到 *db.js* 中：

```
const Vacation = require('./models/vacation')
```

现在我们所有的结构都已经定义好了，但是我们的数据库并不是很有趣，因为里面实际上什么都没有。让我们通过一些数据来种植它，使它变得有用。

## 种植初始数据

我们的数据库中还没有任何度假套餐，所以我们将添加一些来启动我们。最终，您可能希望创建一种管理产品的方式，但是对于本书的目的，我们只打算在代码中执行它（伴随库中的 *ch13/00-mongodb/db.js*）:

```
Vacation.find((err, vacations) => {
  if(err) return console.error(err)
  if(vacations.length) return

  new Vacation({
    name: 'Hood River Day Trip',
    slug: 'hood-river-day-trip',
    category: 'Day Trip',
    sku: 'HR199',
    description: 'Spend a day sailing on the Columbia and ' +
      'enjoying craft beers in Hood River!',
    location: {
      search: 'Hood River, Oregon, USA',
    },
    price: 99.95,
    tags: ['day trip', 'hood river', 'sailing', 'windsurfing', 'breweries'],
    inSeason: true,
    maximumGuests: 16,
    available: true,
    packagesSold: 0,
  }).save()

  new Vacation({
    name: 'Oregon Coast Getaway',
    slug: 'oregon-coast-getaway',
    category: 'Weekend Getaway',
    sku: 'OC39',
    description: 'Enjoy the ocean air and quaint coastal towns!',
    location: {
      search: 'Cannon Beach, Oregon, USA',
    },
    price: 269.95,
    tags: ['weekend getaway', 'oregon coast', 'beachcombing'],
    inSeason: false,
    maximumGuests: 8,
    available: true,
    packagesSold: 0,
  }).save()

  new Vacation({
      name: 'Rock Climbing in Bend',
      slug: 'rock-climbing-in-bend',
      category: 'Adventure',
      sku: 'B99',
      description: 'Experience the thrill of climbing in the high desert.',
      location: {
        search: 'Bend, Oregon, USA',
      },
      price: 289.95,
      tags: ['weekend getaway', 'bend', 'high desert', 'rock climbing'],
      inSeason: true,
      requiresWaiver: true,
      maximumGuests: 4,
      available: false,
      packagesSold: 0,
      notes: 'The tour guide is currently recovering from a skiing accident.',
  }).save()
})
```

这里使用了两个 Mongoose 方法。首先是 `find`，它就像它的名字一样。在这种情况下，它在数据库中找到所有 `Vacation` 的实例，并调用回调函数返回这个列表。我们这样做是因为我们不想不断地重新添加我们的种子假期：如果数据库中已经有了假期，那么它已经被种子化了，我们可以继续进行。然而，第一次执行这个操作时，`find` 将返回一个空列表，因此我们继续创建两个假期，然后在它们上调用 `save` 方法，将这些新对象保存到数据库中。

现在数据已经存入数据库，是时候取回它了！

## 检索数据

我们已经看到了 `find` 方法，这是我们将用来显示假期列表的方法。但是，这次我们将向 `find` 传递一个选项来过滤数据。具体来说，我们只想显示当前可用的假期。

为产品页面创建一个视图，*views/vacations.handlebars*：

```
<h1>Vacations</h1>
{{#each vacations}}
  <div class="vacation">
    <h3>{{name}}</h3>
    <p>{{description}}</p>
    {{#if inSeason}}
      <span class="price">{{price}}</span>
      <a href="/cart/add?sku={{sku}}" class="btn btn-default">Buy Now!</a>
    {{else}}
      <span class="outOfSeason">We're sorry, this vacation is currently
      not in season.
      {{! The "notify me when this vacation is in season"
          page will be our next task. }}
      <a href="/notify-me-when-in-season?sku={{sku}}">Notify me when
      this vacation is in season.</a>
    {{/if}}
  </div>
{{/each}}
```

现在我们可以创建路由处理程序来连接所有这些。在 *lib/handlers.js* 中（不要忘记导入 `../db`），我们创建处理程序：

```
exports.listVacations = async (req, res) => {
  const vacations = await db.getVacations({ available: true })
  const context = {
    vacations: vacations.map(vacation => ({
      sku: vacation.sku,
      name: vacation.name,
      description: vacation.description,
      price: '$' + vacation.price.toFixed(2),
      inSeason: vacation.inSeason,
    }))
  }
  res.render('vacations', context)
}
```

我们添加一个调用处理程序的路由，在 *meadowlark.js* 中：

```
app.get('/vacations', handlers.listVacations)
```

如果您运行此示例，您将只看到我们虚拟数据库实现中的一个假期。这是因为我们已初始化了数据库并且播种了数据，但我们还没有用真实的数据库替换虚拟实现。所以现在让我们来做这个。打开 *db.js* 并修改 `getVacations`：

```
module.exports = {
  getVacations: async (options = {}) => Vacation.find(options),
  addVacationInSeasonListener: async (email, sku) => {
    //...
  },
}
```

这太容易了！只是一个一行代码。部分原因是因为 Mongoose 为我们做了很多繁重的工作，而且我们设计的 API 方式类似于 Mongoose 的工作方式。当我们稍后将其适应 PostgreSQL 时，您会看到我们需要做更多的工作。

###### 注意

机敏的读者可能会担心我们的数据库抽象层并没有太多的“保护”技术中立的目标。例如，开发者可能会阅读这段代码，并且发现他们可以将任何 Mongoose 选项传递给假期模型，这样应用程序就会使用特定于 Mongoose 的功能，这将使得切换数据库变得更加困难。我们可以采取一些措施来防止这种情况发生。我们不仅仅是将东西传递给 Mongoose，而是要寻找特定的选项并明确地处理它们，以表明任何实现都必须提供这些选项。但是出于本例的考虑，我们将放任这一点，并保持这段代码的简单性。

大多数内容应该看起来很熟悉，但可能会有一些令您惊讶的地方。例如，我们如何处理假期列表的视图上下文可能看起来有点奇怪。为什么我们要将从数据库返回的产品映射到一个几乎相同的对象？一个原因是我们想以整齐格式显示价格，所以我们必须将其转换为格式化的字符串。

我们本可以通过这样做来节省一些输入：

```
const context = {
  vacations: products.map(vacations => {
    vacation.price = '$' + vacation.price.toFixed(2)
    return vacation
  })
}
```

这肯定可以节省我们几行代码，但根据我的经验，有很多理由不直接将未映射的数据库对象传递给视图。视图获得了一堆可能不需要的属性，可能还以与其不兼容的格式。我们的示例目前还相当简单，但一旦开始变得更加复杂，您可能希望对传递给视图的数据进行更多定制。此外，这也很容易意外地暴露机密信息或可能危及网站安全的信息。基于这些理由，我建议映射从数据库返回的数据，并仅将必要的内容传递给视图（必要时进行转换，就像我们对 `price` 做的那样）。

###### 注意

在某些 MVC 架构的变体中，引入了一个称为 *视图模型* 的第三个组件。视图模型本质上是将一个（或多个）模型提炼和转换，使其更适合在视图中显示。我们在这里所做的是即时创建一个视图模型。

到目前为止，我们已经走了很长一段路。我们成功地使用数据库存储了关于我们假期的信息。但是，如果我们不能更新它们，数据库就不会太有用。让我们把注意力转向与数据库接口的这个方面。

## 添加数据

我们已经看到了如何添加数据（在种子化度假集合时添加了数据）以及如何更新数据（在预订度假时更新售出的套餐数量），但让我们看一个稍微复杂的场景，突显文档数据库的灵活性。

当度假不在季节时，我们显示一个链接，邀请客户在度假再次进入季节时通知他们。让我们连接这个功能。首先，我们创建模式和模型（*models/vacationInSeasonListener.js*）：

```
const mongoose = require('mongoose')

const vacationInSeasonListenerSchema = mongoose.Schema({
  email: String,
  skus: [String],
})
const VacationInSeasonListener = mongoose.model('VacationInSeasonListener',
  vacationInSeasonListenerSchema)

module.exports = VacationInSeasonListener
```

接下来我们将创建我们的视图，*views/notify-me-when-in-season.handlebars*：

```
<div class="formContainer">
  <form class="form-horizontal newsletterForm" role="form"
      action="/notify-me-when-in-season" method="POST">
    <input type="hidden" name="sku" value="{{sku}}">
    <div class="form-group">
      <label for="fieldEmail" class="col-sm-2 control-label">Email</label>
      <div class="col-sm-4">
        <input type="email" class="form-control" required
          id="fieldEmail" name="email">
      </div>
    </div>
    <div class="form-group">
      <div class="col-sm-offset-2 col-sm-4">
        <button type="submit" class="btn btn-default">Submit</button>
      </div>
    </div>
  </form>
</div>
```

然后是路由处理程序：

```
exports.notifyWhenInSeasonForm = (req, res) =>
  res.render('notify-me-when-in-season', { sku: req.query.sku })

exports.notifyWhenInSeasonProcess = (req, res) => {
  const { email, sku } = req.body
  await db.addVacationInSeasonListener(email, sku)
  return res.redirect(303, '/vacations')
}
```

最后，我们在 *db.js* 中添加了一个真实的实现：

```
const VacationInSeasonListener = require('./models/vacationInSeasonListener')

module.exports = {
  getVacations: async (options = {}) => Vacation.find(options),
  addVacationInSeasonListener: async (email, sku) => {
    await VacationInSeasonListener.updateOne(
      { email },
      { $push: { skus: sku } },
      { upsert: true }
    )
  },
}
```

这是什么魔法？我们如何在 `VacationInSeasonListener` 集合甚至不存在之前就“更新”记录？答案在于 Mongoose 的一个便利功能叫做 *upsert*（“update” 和 “insert” 的混成词）。基本上，如果不存在具有给定电子邮件地址的记录，它将被创建。如果记录已存在，则将进行更新。然后，我们使用魔法变量 `$push` 表示我们要向数组添加一个值。

###### 注意：

此代码不会阻止用户多次填写表单后将多个 SKU 添加到记录中。当度假季节到来时，我们找到所有想收到通知的客户时，必须小心不要多次通知他们。

我们现在肯定已经涵盖了重要的基础知识！我们学会了如何连接到 MongoDB 实例，向其种子化数据，读取数据，并对其进行更新！然而，你可能更喜欢使用关系数据库管理系统，因此让我们改变思路，看看如何使用 PostgreSQL 来完成同样的工作。

## PostgreSQL

像 MongoDB 这样的对象数据库非常棒，并且通常更快地开始使用，但如果你尝试构建一个强大的应用程序，你可能会像规划传统关系数据库一样多或更多地工作来构建你的对象数据库结构。此外，你可能已经对关系数据库有经验，或者你可能已经有一个现有的关系数据库需要连接。

幸运的是，在 JavaScript 生态系统中，每个主要的关系型数据库都有强大的支持，如果你需要使用关系型数据库，应该不会有任何问题。

让我们拿我们的度假数据库，并使用关系数据库重新实现它。在这个示例中，我们将使用 PostgreSQL，一个流行且复杂的开源关系数据库。我们将使用的技术和原则对任何关系数据库都是类似的。

与我们用于 MongoDB 的对象数据映射（ODM）类似，针对关系数据库也有对象关系映射（ORM）工具可用。然而，由于大多数对此主题感兴趣的读者可能已经熟悉关系数据库和 SQL，因此我们将直接使用 Node PostgreSQL 客户端。

和 MongoDB 一样，我们将使用一个免费的在线 PostgreSQL 服务。当然，如果你习惯于安装和配置自己的 PostgreSQL 数据库，你也可以这样做。所有要改变的只是连接字符串。如果你使用自己的 PostgreSQL 实例，请确保你使用的是 9.4 或更高版本，因为我们将使用 9.4 引入的 JSON 数据类型（我写这篇文章时，正在使用 11.3）。

有许多在线 PostgreSQL 的选择；在这个示例中，我将使用 [ElephantSQL](https://www.elephantsql.com)。开始使用简单至极：创建一个账户（你可以使用 GitHub 账号登录），然后点击创建新实例。你只需要给它一个名字（例如，“meadowlark”）并选择一个计划（你可以使用他们的免费计划）。你还需要指定一个区域（试着选择离你最近的那一个）。一旦你设置好了，你会在详情部分找到一些关于你实例的信息。复制 URL（连接字符串），里面包括了用户名、密码和实例位置，都在一个便捷的字符串中。

将该字符串放入你的 *.credentials.development.json* 文件中：

```
"postgres": {
  "connectionString": "your_dev_connection_string"
}
```

对象数据库和关系型数据库（RDBMSs）之间的一个区别是，你通常需要更多的前期工作来定义 RDBMS 的模式，并使用数据定义 SQL 来创建模式，然后再添加或检索数据。为了遵循这个范式，我们将把这作为一个单独的步骤来处理，而不是让我们的 ODM 或 ORM 来处理，就像我们在 MongoDB 中所做的那样。

我们可以创建 SQL 脚本，并使用命令行客户端执行数据定义脚本来创建我们的表，或者我们可以使用 PostgreSQL 客户端 API 在 JavaScript 中完成这项工作，但这是一个只做一次的独立步骤。因为这是关于 Node 和 Express 的书，我们会选择后者来完成这项工作。

首先，我们将需要安装 `pg` 客户端库 (`npm install pg`)。然后创建 *db-init.js*，这将只用于初始化我们的数据库，与我们的 *db.js* 文件有所区别，后者会在每次服务器启动时被使用（在 companion repo 中的 *ch13/01-postgres/db.js*）：

```
const { credentials } = require('./config')

const { Client } = require('pg')
const { connectionString } = credentials.postgres
const client = new Client({ connectionString })

const createScript = `
 CREATE TABLE IF NOT EXISTS vacations (
 name varchar(200) NOT NULL,
 slug varchar(200) NOT NULL UNIQUE,
 category varchar(50),
 sku varchar(20),
 description text,
 location_search varchar(100) NOT NULL,
 location_lat double precision,
 location_lng double precision,
 price money,
 tags jsonb,
 in_season boolean,
 available boolean,
 requires_waiver boolean,
 maximum_guests integer,
 notes text,
 packages_sold integer
 );
`

const getVacationCount = async client => {
  const { rows } = await client.query('SELECT COUNT(*) FROM VACATIONS')
  return Number(rows[0].count)
}

const seedVacations = async client => {
  const sql = `
 INSERT INTO vacations(
 name,
 slug,
 category,
 sku,
 description,
 location_search,
 price,
 tags,
 in_season,
 available,
 requires_waiver,
 maximum_guests,
 notes,
 packages_sold
 ) VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11, $12, $13, $14)
 `
  await client.query(sql, [
    'Hood River Day Trip',
    'hood-river-day-trip',
    'Day Trip',
    'HR199',
    'Spend a day sailing on the Columbia and enjoying craft beers in Hood River!',
    'Hood River, Oregon, USA',
    99.95,
    `["day trip", "hood river", "sailing", "windsurfing", "breweries"]`,
    true,
    true,
    false,
    16,
    null,
    0,
  ])
  // we can use the same pattern to insert other vacation data here...
}

client.connect().then(async () => {
  try {
    console.log('creating database schema')
    await client.query(createScript)
    const vacationCount = await getVacationCount(client)
    if(vacationCount === 0) {
      console.log('seeding vacations')
      await seedVacations(client)
    }
  } catch(err) {
    console.log('ERROR: could not initialize database')
    console.log(err.message)
  } finally {
    client.end()
  }
})
```

让我们从这个文件的底部开始。我们拿到我们的数据库客户端(`client`)并对其调用`connect()`，这将建立数据库连接并返回一个 promise。当 promise 解析后，我们就可以针对数据库采取行动。

首先我们会调用 `client.query(createScript)`，这将创建我们的 `vacations` 表（也称为*关系*）。如果我们查看 `createScript`，我们会看到这是数据定义的 SQL 语句。本书不涉及 SQL 的深入讨论，但如果你正在阅读这一部分，我假设你至少对 SQL 有基本的了解。你可能注意到的一件事是，我们使用蛇形命名法（snake_case）来命名字段，而不是驼峰命名法（camelCase）。也就是说，原来的“inSeason”变成了“in_season”。虽然在 PostgreSQL 中可以使用驼峰命名法来命名结构，但对于任何带有大写字母的标识符都必须加引号，这比它值得的麻烦更多。稍后我们会再次回到这个问题。

你会注意到，我们已经开始更深入地思考我们的模式。假期名称可以有多长？（这里我们随意将其限制在 200 个字符。）类别名称和 SKU 可以有多长？请注意，我们使用 PostgreSQL 的 `money` 类型来表示价格，并且将 slug 作为我们的主键（而不是添加一个单独的 ID）。

如果你已经熟悉关系数据库，这个简单的模式不会有什么意外。然而，我们处理“标签”的方式可能会引起你的注意。

在传统的数据库设计中，我们可能会创建一个新表来将假期与标签关联起来（这称为*规范化*）。我们可以在这里这样做。但是在这里，我们可能会决定在传统关系数据库设计与“JavaScript 方式”之间进行一些妥协。如果我们选择两个表（例如`vacations`和`vacation_tags`），我们将不得不从两个表中查询数据，以创建一个包含有关假期所有信息的单个对象，就像我们在 MongoDB 示例中所做的那样。可能存在性能原因需要增加这种额外的复杂性，但让我们假设没有，我们只想能够快速确定特定假期的标签。我们可以将其作为文本字段，并用逗号分隔我们的标签，但然后我们将不得不解析出我们的标签，而 PostgreSQL 给了我们一个更好的方法，即 JSON 数据类型。我们很快将看到，通过将其指定为 JSON（`jsonb`，通常更高性能的二进制表示），我们可以将其存储为 JavaScript 数组，JavaScript 数组与我们在 MongoDB 中看到的一样。

最后，我们通过使用与之前相同的基本概念将我们的种子数据插入到数据库中：如果 `vacations` 表为空，我们添加一些初始数据；否则，我们假设我们已经完成了这些操作。

你会注意到，插入我们的数据比在 MongoDB 中更加不便。有方法可以解决这个问题，但是对于这个示例，我想明确使用 SQL。我们可以编写一个函数来使插入语句更自然，或者我们可以使用 ORM（稍后详细介绍）。但是现在，SQL 完成了工作，并且对于任何已经了解 SQL 的人来说，应该是舒适的。

请注意，尽管此脚本设计为仅运行一次以初始化和填充我们的数据库，但我们已经以安全的方式编写它。我们包含了`IF NOT EXISTS`选项，并检查`vacations`表是否为空，然后再添加种子数据。

我们现在可以运行脚本来初始化我们的数据库：

```
$ node db-init.js
```

现在我们已经设置好了数据库，我们可以编写一些代码来在我们的网站中*使用*它。

数据库服务器通常只能同时处理有限数量的连接，因此 Web 服务器通常实现一种称为*连接池*的策略，以平衡建立连接的开销与长时间保持连接的危险，从而使服务器负载过重。幸运的是，这些细节由 PostgreSQL Node 客户端为您处理。

这次我们将采用稍微不同的策略处理我们的*db.js*文件。与其仅仅是一个我们需要导入以建立数据库连接的文件不同，它将返回一个 API，我们编写它来处理与数据库通信的详细信息。

我们在度假模型上还有一个决定要做。回想一下，当我们创建我们的模型时，我们在数据库模式中使用了 snake_case，但所有我们的 JavaScript 代码都使用 camelCase。总体来说，我们在这里有三个选项：

+   重构我们的模式以使用 camelCase。这将使我们的 SQL 更加丑陋，因为我们必须记得正确引用我们的属性名。

+   在我们的 JavaScript 中使用 snake_case。虽然这不是理想的，因为我们喜欢标准（对吧？）。

+   在数据库端使用 snake_case，而在 JavaScript 端转换为 camelCase。这是我们必须做的更多工作，但它可以保持我们的 SQL 和 JavaScript 的整洁。

幸运的是，第三个选项可以自动完成。我们可以编写自己的函数来进行翻译，但我们将依赖于一个流行的实用库称为[Lodash](https://lodash.com)，这使得操作变得非常简单。只需运行`npm install lodash`来安装它。

现在，我们的数据库需求非常简单。我们只需要获取所有可用的度假套餐，因此我们的*db.js*文件看起来像这样（*ch13/01-postgres/db.js*在配套存储库中）：

```
const { Pool } = require('pg')
const _ = require('lodash')

const { credentials } = require('./config')

const { connectionString } = credentials.postgres
const pool = new Pool({ connectionString })

module.exports = {
  getVacations: async () => {
    const { rows } = await pool.query('SELECT * FROM VACATIONS')
    return rows.map(row => {
      const vacation = _.mapKeys(row, (v, k) => _.camelCase(k))
      vacation.price = parseFloat(vacation.price.replace(/^\$/, ''))
      vacation.location = {
        search: vacation.locationSearch,
        coordinates: {
          lat: vacation.locationLat,
          lng: vacation.locationLng,
        },
      }
      return vacation
    })
  }
}
```

简单明了！我们导出一个名为`getVacations`的单一方法，它按照广告所述执行。它还使用了 Lodash 的`mapKeys`和`camelCase`函数将我们的数据库属性转换为 camelCase。

需要注意的一点是，我们必须小心处理 `price` 属性。PostgreSQL 的 `money` 类型通过 `pg` 库转换为已经格式化的字符串。这是有充分理由的：正如我们已经讨论过的，JavaScript 最近才添加了对任意精度数值类型（`BigInt`）的支持，但目前还没有一个 PostgreSQL 适配器能够利用它（并且在任何情况下这可能也不是最高效的数据类型）。我们可以改变我们的数据库模式，使用数值类型而不是 `money` 类型，但我们不应该让我们的前端选择来驱动我们的模式。我们也可以处理从 `pg` 返回的预格式化字符串，但这样做会导致我们所有现有的依赖于 `price` 为数字的代码都需要改变。此外，这种方法会削弱我们在前端执行数值计算的能力（例如对购物车中商品价格求和）。基于这些原因，我们选择在从数据库检索数据时将字符串解析为数字。

我们还需要将我们的位置信息（“平面”在表中）转换为更接近 JavaScript 结构的形式。我们这样做只是为了与我们的 MongoDB 示例保持一致；我们可以使用其现有的结构（或修改我们的 MongoDB 示例以具有平面结构）。

我们需要学习的最后一件事是如何使用 PostgreSQL 更新数据，所以让我们填写“旺季假期”侦听器功能。

## 添加数据

就像 MongoDB 示例一样，我们将使用我们的“旺季假期”侦听器示例。我们将首先在 *db-init.js* 的 `createScript` 字符串中添加以下数据定义：

```
CREATE TABLE IF NOT EXISTS vacation_in_season_listeners (
  email varchar(200) NOT NULL,
  sku varchar(20) NOT NULL,
  PRIMARY KEY (email, sku)
);
```

请记住，我们小心地以非破坏性的方式编写了 *db-init.js*，这样我们可以随时运行它。因此，我们可以再次运行它来创建 `vacation_in_season_listeners` 表。

现在我们可以修改 *db.js* 来包含一个更新这个表的方法：

```
module.exports = {
  //...
  addVacationInSeasonListener: async (email, sku) => {
    await pool.query(
      'INSERT INTO vacation_in_season_listeners (email, sku) ' +
      'VALUES ($1, $2) ' +
      'ON CONFLICT DO NOTHING',
      [email, sku]
    )
  },
}
```

PostgreSQL 的 `ON CONFLICT` 子句实际上启用了 upserts。在这种情况下，如果邮箱和 SKU 的确切组合已经存在，用户已经注册以便收到通知，因此我们无需采取任何行动。如果我们在表中有其他列（例如上次注册日期），我们可能需要使用更复杂的 `ON CONFLICT` 子句（有关更多信息，请参阅 [PostgreSQL INSERT documentation](http://bit.ly/3724FJI)）。还要注意，此行为取决于我们如何定义表格。我们将邮箱和 SKU 定义为复合主键，这意味着不能有重复，这进而需要 `ON CONFLICT` 子句（否则，当用户尝试在同一个假期上注册通知时，`INSERT` 命令会导致错误）。

现在我们已经看到了如何连接两种类型的数据库的完整示例，一个是对象数据库，另一个是关系数据库管理系统（RDBMS）。可以清楚地看到数据库的功能是一样的：以一种一致和可扩展的方式存储、检索和更新数据。由于功能相同，我们能够创建一个抽象层，以便选择不同的数据库技术。我们可能需要一个数据库的最后一件事是用于持久化会话存储，这在第九章中有所提及。

# 使用数据库进行会话存储

正如我们在第九章中讨论的那样，在生产环境中使用内存存储会话数据是不合适的。幸运的是，使用数据库作为会话存储是很容易的。

虽然我们可以使用现有的 MongoDB 或 PostgreSQL 数据库作为会话存储，但完整的数据库对于会话存储来说可能有些过度，对于键值数据库来说却是一个完美的使用案例。截至我写这篇文章时，用于会话存储的最流行的键值数据库是[Redis](https://redis.io)和[Memcached](https://memcached.org)。与本章中的其他示例保持一致，我们将使用一个免费的在线服务来提供 Redis 数据库。

首先，请访问[Redis Labs](https://redislabs.com)，创建一个账户。然后创建一个免费的订阅计划。选择缓存作为计划，并给数据库起个名字；其余的设置可以保持默认。

您将会看到一个查看数据库的屏幕，就我所知，关键信息需要几秒钟才能显示，请耐心等待。您需要的是端点字段和访问控制与安全下的 Redis 密码（默认情况下是隐藏的，但旁边有一个按钮可以显示它）。将它们放入您的*.credentials.development.json*文件中：

```
"redis": {
  "url": "redis://:<YOUR PASSWORD>@<YOUR ENDPOINT>"
}
```

注意这个稍微奇怪的 URL：通常在密码前面会有一个用户名，但 Redis 允许仅使用密码连接；然而，在密码前的冒号仍然是必需的。

我们将使用一个叫做`connect-redis`的包来提供 Redis 会话存储。一旦你安装了它（`npm install connect-redis`），我们就可以在主应用程序文件中设置它。我们仍然使用`express-session`，但现在我们传递一个新的属性`store`给它，这将配置它使用数据库。请注意，我们必须将`expressSession`传递给从`connect-redis`返回的函数，以获取构造函数：这是会话存储的一个常见特性（在伴随代码库的*ch13/00-mongodb/meadowlark.js*或*ch13/01-postgres/meadowlark.js*中）：

```
const expressSession = require('express-session')
const RedisStore = require('connect-redis')(expressSession)

app.use(cookieParser(credentials.cookieSecret))
app.use(expressSession({
  resave: false,
  saveUninitialized: false,
  secret: credentials.cookieSecret,
  store: new RedisStore({
    url: credentials.redis.url,
    logErrors: true,  // highly recommended!
  }),
}))
```

现在让我们将我们新建的会话存储用于实际的用途。假设我们希望能够以不同的货币显示度假价格。此外，我们希望网站记住用户的货币偏好。

我们将从在度假页面底部添加一个货币选择器开始：

```
<hr>
<p>Currency:
    <a href="/set-currency/USD" class="currency {{currencyUSD}}">USD</a> |
    <a href="/set-currency/GBP" class="currency {{currencyGBP}}">GBP</a> |
    <a href="/set-currency/BTC" class="currency {{currencyBTC}}">BTC</a>
</p>
```

现在来看一些 CSS 代码（你可以将其嵌入到 *views/layouts/main.handlebars* 文件中，或者链接到 *public* 目录下的 CSS 文件中）：

```
a.currency {
  text-decoration: none;
}
.currency.selected {
  font-weight: bold;
  font-size: 150%;
}
```

最后，我们将添加一个路由处理程序来设置货币，并修改我们的 */vacations* 路由处理程序以在当前货币中显示价格（*ch13/00-mongodb/lib/handlers.js* 或者 *ch13/01-postgres/lib/handlers.js* 在配套代码库中）：

```
exports.setCurrency = (req, res) => {
  req.session.currency = req.params.currency
  return res.redirect(303, '/vacations')
}

function convertFromUSD(value, currency) {
  switch(currency) {
    case 'USD': return value * 1
    case 'GBP': return value * 0.79
    case 'BTC': return value * 0.000078
    default: return NaN
  }
}

exports.listVacations = (req, res) => {
  Vacation.find({ available: true }, (err, vacations) => {
    const currency = req.session.currency || 'USD'
    const context = {
      currency: currency,
      vacations: vacations.map(vacation => {
        return {
          sku: vacation.sku,
          name: vacation.name,
          description: vacation.description,
          inSeason: vacation.inSeason,
          price: convertFromUSD(vacation.price, currency),
          qty: vacation.qty,
        }
      })
    }
    switch(currency){
      case 'USD': context.currencyUSD = 'selected'; break
      case 'GBP': context.currencyGBP = 'selected'; break
      case 'BTC': context.currencyBTC = 'selected'; break
    }
    res.render('vacations', context)
  })
}
```

你还需要在 *meadowlark.js* 中添加一个设置货币的路由：

```
app.get('/set-currency/:currency', handlers.setCurrency)
```

当然，这并不是执行货币转换的好方法。我们希望利用第三方货币转换 API 来确保我们的汇率是最新的。但这对于演示目的已经足够了。你现在可以在各种货币之间切换，并且——试试看——停止和重新启动你的服务器。你会发现它记住了你的货币偏好！如果清除了你的 Cookie，货币偏好将被遗忘。你会注意到，现在我们失去了我们漂亮的货币格式化；现在它更加复杂，我将把这留给读者作为一个练习。

另一个读者的练习是将 `set-currency` 路由通用化，使其更有用。目前，它总是重定向到度假页面，但如果你想在购物车页面上使用它呢？看看你能不能想出一两种解决这个问题的方法。

如果你查看你的数据库，你会发现有一个名为 *sessions* 的新集合。如果你探索该集合，你会找到一个带有你的会话 ID（属性 `sid`）和你的货币偏好的文档。

# 结论

在这一章中，我们确实涵盖了很多内容。对于大多数 Web 应用程序来说，数据库是使应用程序有用的核心。设计和调优数据库是一个广泛的主题，可能需要多本书来覆盖，但我希望这为你提供了连接两种类型数据库和移动数据所需的基本工具。

现在我们已经放置了这个基本的部分，我们将重新访问路由和它在 Web 应用中的重要性。
