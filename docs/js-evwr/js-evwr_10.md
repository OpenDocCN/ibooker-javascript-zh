# 第九章：细节

当现在几乎无处不在的空气清新剂 Febreze 首次发布时，它一无是处。最初的广告展示了人们使用该产品去除特定的恶臭，如香烟烟雾，导致销售疲软。面对这令人失望的结果，营销团队将焦点转移到将 Febreze 用作完美细节。现在，广告描绘了有人在清理房间，拍打枕头，并完成了使用 Febreze 给房间喷雾的任务。这种重新定义产品的方式使销售额飙升。

这是一个很好的例子，*细节很重要*。现在我们有一个工作中的 API，但缺少让它投入生产的最后修饰。在本章中，我们将实施一些 Web 和 GraphQL 应用程序安全和用户体验最佳实践。这些远远超过空气清新剂的喷雾的细节将对我们应用程序的安全性、可用性和易用性至关重要。

# Web 应用程序和 Express.js 最佳实践

Express.js 是支持我们 API 的底层 Web 应用程序框架。我们可以对 Express.js 代码进行一些小的调整，以为我们的应用程序提供坚实的基础。

## Express Helmet

Express [Helmet 中间件](https://oreil.ly/NGae1) 是一组小型的以安全为导向的中间件函数。这些函数将调整我们应用程序的 HTTP 头部以提升安全性。尽管其中许多是针对基于浏览器的应用程序的，启用 Helmet 是保护我们应用程序免受常见网络漏洞的简单步骤。

要启用 Helmet，我们将在我们的应用程序中引入中间件，并指示 Express 在我们的中间件堆栈中早期使用它。在 *./src/index.js* 文件中，添加以下内容：

```
// first require the package at the top of the file
const helmet = require('helmet')

// add the middleware at the top of the stack, after const app = express()
app.use(helmet());
```

通过添加 Helmet 中间件，我们可以快速为我们的应用启用常见的网络安全最佳实践。

## 跨域资源共享

跨域资源共享（CORS）是允许从另一个域请求资源的手段。因为我们的 API 和 UI 代码将分开存放，我们希望从其他来源启用凭据。如果你有兴趣了解 CORS 的方方面面，我强烈推荐 [Mozilla CORS Guide](https://oreil.ly/E1lXZ)。

要启用 CORS，我们将在我们的 *.src/index.js* 文件中使用 Express.js [CORS 中间件](https://oreil.ly/lYr7g) 包：

```
// first require the package at the top of the file
const cors = require('cors');

// add the middleware after app.use(helmet());
app.use(cors());
```

通过这种方式添加中间件，我们可以允许来自 *所有* 域的跨源请求。目前这对我们来说效果很好，因为我们处于开发模式，可能会使用由我们的托管提供商生成的域，但通过使用中间件，我们也可以限制请求来自特定来源。

# 分页

目前，我们的`notes`和`users`查询返回数据库中所有笔记和用户的完整列表。这在本地开发中运行良好，但随着应用程序的增长，这将变得难以维护，因为查询可能会返回数百（甚至数千）个笔记，这样的开销会拖慢我们的数据库、服务器和网络。因此，我们可以对这些查询进行分页，仅返回一定数量的结果。

我们可以实现两种常见的分页方式。第一种类型，*偏移分页*，通过客户端传递偏移数，并返回有限数量的数据。例如，如果每页数据限制为 10 条记录，我们想要请求第三页数据，我们可以传递偏移量为 20。尽管这在概念上是最直观的方法，但它可能会遇到扩展和性能问题。

第二种分页方式是*基于游标的分页*，其中传递一个基于时间或唯一标识的游标作为起始点。然后，我们请求跟随此记录的特定数据量。这种方法能够给我们最大程度上的分页控制。此外，因为 Mongo 的对象 ID 是有序的（它们以 4 字节的时间值开头），我们可以很容易地将它们用作游标。要了解更多关于 Mongo 对象 ID 的信息，建议阅读[对应的 MongoDB 文档](https://oreil.ly/GPE1c)。

如果这对你来说听起来过于概念化，没关系。让我们一起来实现一个基于 GraphQL 查询的分页笔记 Feed。首先，我们定义我们将要创建的内容，接着是我们的模式更新，最后是我们的解析器代码。对于我们的 Feed，我们希望在查询 API 时可选地传递一个游标作为参数。API 应返回有限的数据量，一个表示数据集中最后一项的游标点，并一个布尔值，表示是否有另一页数据可查询。

有了这个描述，我们可以更新我们的*src/schema.js*文件，定义这个新的查询。首先，我们需要在文件中添加一个`NoteFeed`类型：

```
type NoteFeed {
  notes: [Note]!
  cursor: String!
  hasNextPage: Boolean!
}
```

接下来，我们将添加我们的`noteFeed`查询：

```
type Query {
  # add noteFeed to our existing queries
  noteFeed(cursor: String): NoteFeed
}
```

在我们的模式更新后，我们可以为我们的查询编写解析器代码。在*./src/resolvers/query.js*中，向导出对象添加以下内容：

```
noteFeed: async (parent, { cursor }, { models }) => {
  // hardcode the limit to 10 items
  const limit = 10;
  // set the default hasNextPage value to false
  let hasNextPage = false;
  // if no cursor is passed the default query will be empty
  // this will pull the newest notes from the db
  let cursorQuery = {};

  // if there is a cursor
  // our query will look for notes with an ObjectId less than that of the cursor
  if (cursor) {
    cursorQuery = { _id: { $lt: cursor } };
  }

  // find the limit + 1 of notes in our db, sorted newest to oldest
  let notes = await models.Note.find(cursorQuery)
    .sort({ _id: -1 })
    .limit(limit + 1);

  // if the number of notes we find exceeds our limit
  // set hasNextPage to true and trim the notes to the limit
  if (notes.length > limit) {
    hasNextPage = true;
    notes = notes.slice(0, -1);
  }

  // the new cursor will be the Mongo object ID of the last item in the feed array
  const newCursor = notes[notes.length - 1]._id;

  return {
    notes,
    cursor: newCursor,
    hasNextPage
  };
}
```

有了这个解析器，我们可以查询我们的`noteFeed`，它将最多返回 10 个结果。在 GraphQL Playground 中，我们可以编写以下查询来接收笔记列表，它们的对象 ID，它们的“创建时间”时间戳，游标，以及下一页的布尔值：

```
query {
  noteFeed {
    notes {
      id
      createdAt
    }
    cursor
    hasNextPage
  }
}
```

由于我们的数据库中有超过 10 条笔记，这会返回一个游标以及`hasNextPage`值为`true`。通过这个游标，我们可以查询 Feed 的第二页：

```
query {
  noteFeed(cursor: "<YOUR OBJECT ID>") {
    notes {
      id
      createdAt
    }
    cursor
    hasNextPage
  }
}
```

我们可以继续为每个游标执行此操作，其中`hasNextPage`值为`true`。有了这个实现，我们已经创建了一个分页笔记 Feed。这不仅允许我们的 UI 请求特定数据 Feed，还可以减少服务器和数据库的负担。

# 数据限制

除了建立分页之外，我们还希望限制可以通过我们的 API 请求的数据量。这可以防止可能会使我们的服务器或数据库超载的查询。

在这个过程中的一个简单的第一步是限制查询可以返回的数据量。我们的两个查询，`users` 和 `notes`，返回数据库中所有匹配的数据。我们可以通过在数据库查询上设置 `limit()` 方法来解决这个问题。例如，在我们的 *.src/resolvers/query.js* 文件中，我们可以更新我们的 `notes` 查询如下：

```
notes: async (parent, args, { models }) => {
  return await models.Note.find().limit(100);
}
```

尽管限制数据是一个很好的开始，但是当前我们的查询可以写成无限深度。这意味着可以写一个单一查询来检索一系列笔记，每个笔记的作者信息，每个作者的收藏列表，每个收藏的作者信息，依此类推。这是一个很多数据的查询，我们还可以继续！为了防止这些过度嵌套的查询，我们可以*限制查询的深度*。

此外，我们可能有复杂的查询，虽然不是过于嵌套，但仍需要大量计算来返回数据。我们可以通过*限制查询复杂性*来防止这些类型的请求。

我们可以通过在我们的 *./src/index.js* 文件中使用 `graphql-depth-limit` 和 `graphql-validation-complexity` 包来实现这些限制：

```
// import the modules at the top of the file
const depthLimit = require('graphql-depth-limit');
const { createComplexityLimitRule } = require('graphql-validation-complexity');

// update our ApolloServer code to include validationRules
const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [depthLimit(5), createComplexityLimitRule(1000)],
  context: async ({ req }) => {
    // get the user token from the headers
    const token = req.headers.authorization;
    // try to retrieve a user with the token
    const user = await getUser(token);
    // add the db models and the user to the context
    return { models, user };
  }
});
```

通过添加这些包，我们为我们的 API 添加了额外的查询保护。有关如何保护 GraphQL API 免受恶意查询的更多信息，请查看 Spectrum 的首席技术官 Max Stoiber 的[精彩文章](https://oreil.ly/_r5tl)。

# 其他考虑事项

在构建完我们的 API 后，您应该对 GraphQL 开发的基础知识有了坚实的理解。如果您渴望深入了解更多主题，一些继续学习的好去处包括测试、GraphQL 订阅和 Apollo Engine。

## 测试

好吧，我承认：我没有在这本书中写关于测试，我感到有些内疚。测试我们的代码很重要，因为它使我们能够放心地进行更改，并改善与其他开发人员的协作。我们的 GraphQL 设置中的一大优势是解析器只是简单的函数，接受一些参数并返回数据。这使得我们的 GraphQL 逻辑易于测试。

## 订阅

订阅是 GraphQL 的一个非常强大的功能，它提供了一种简单的方法在我们的应用程序中集成发布-订阅模式。这意味着用户界面可以订阅在服务器上发布数据时进行通知或更新。这使得 GraphQL 服务器成为处理实时数据的理想解决方案。有关 GraphQL 订阅的更多信息，请查看[Apollo Server 文档](https://oreil.ly/YwI5_)。

## Apollo GraphQL 平台

在开发我们的 API 的整个过程中，我们一直在使用 Apollo GraphQL 库。在将来的章节中，我们还将使用 Apollo 客户端库与我们的 API 进行交互。我选择这些库是因为它们是行业标准，并且为使用 GraphQL 的开发者提供了极好的开发体验。如果你把你的应用程序投入生产，维护这些库的公司 Apollo 还提供了一个平台，为 GraphQL API 提供监控和工具支持。您可以在[Apollo 的网站](https://www.apollographql.com)上了解更多。

# 结论

在本章中，我们为我们的应用程序添加了一些最后的修饰。虽然我们可以实现许多其他选项，但在这一点上，我们已经开发出了一个坚实的 MVP（最小可行产品）。在这个状态下，我们已经准备好启动我们的 API！在下一章中，我们将把我们的 API 部署到一个公共 Web 服务器上。
