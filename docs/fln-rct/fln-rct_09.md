# 第八章：框架

在我们迄今为止的 React 之旅中，我们已经揭示了一系列功能和原则，这些功能和原则为其功能和多功能性做出了贡献。前一章深入探讨了异步 React 的迷人世界，这使我们能够使用`useTransition`和`useDeferredValue`等工具来创建高度响应和用户友好的界面。我们探索了这些工具如何利用 React 的复杂调度和优先级机制，这是由 Fiber reconciler 可能实现的，以实现最佳性能。在本章中，理解这些异步模式对我们进入 React 框架的领域至关重要。

单独使用 React 本身非常强大，但随着应用程序复杂度的增加，我们经常发现自己重复使用类似的模式或需要更简化的解决方案来应对常见挑战。这就是框架的作用。React 框架是建立在 React 之上的软件库或工具包，提供额外的抽象以更有效地处理常见任务，并强制执行最佳实践。

# 为什么我们需要一个框架

虽然 React 提供了创建交互式用户界面的构建块，但它在许多重要的架构决策上留给开发人员。在这方面，React 并没有明确的意见，使开发人员能够以他们认为合适的方式组织他们的应用程序。然而，随着应用程序的扩展，这种自由可能会成为负担。您可能会发现自己一遍又一遍地重新发明轮子，处理诸如路由、数据获取和服务器端渲染等常见挑战。

这就是 React 框架的作用。它们提供了一个预定义的结构和常见问题的解决方案，使开发人员能够专注于其应用程序的独特之处，而不是陷入样板代码中。这可以显著加速开发过程，并通过遵循框架强制执行的最佳实践来改善代码库的质量。

要完全理解这一点，让我们试着编写我们自己的最小框架。为了做到这一点，我们需要确定我们从框架中获得而不容易从纯 React 中获得的一些关键功能。为简洁起见，我们将确定这些行中的三个关键特征，这些特征将帮助我们形成一个很好的讨论基础：

+   服务器渲染

+   路由

+   数据获取

让我们从一个现有的想象中的 React 应用程序开始，并逐步添加这些功能，以了解框架对我们的意义。我们“框架化”的 React 应用程序的结构如下：

```
- index.js
- List.js
- Detail.js
- dist/
  - clientBundle.js
```

这是每个文件的样子：

```
// index.js

import React from "react";
import { createRoot } from "react-dom/client";
import Router from "./Router";

const root = createRoot(document);

const params = new URLSearchParams();
const thingId = params.get("id");

root.render(
  window.location.pathname === "/" ? <List /> : <Detail thingId={thingId} />
);

// List.js

export const List = () => {
  const [things, setThings] = useState([]);
  const [requestState, setRequestState] = useState("initial");
  const [error, setError] = useState(null);

  useEffect(() => {
    setRequestState("loading");
    fetch("https://api.com/get-list")
      .then((r) => r.json())
      .then(setThings)
      .then(() => {
        setRequestState("success");
      })
      .catch((e) => {
        setRequestState("error");
        setError(e);
      });
  }, []);

  return (
    <div>
      <ul>
        {things.map((thing) => (
          <li key={thing.id}>{thing.label}</li>
        ))}
      </ul>
    </div>
  );
};

// Detail.js

export const Detail = ({ thingId }) => {
  const [thing, setThing] = useState([]);
  const [requestState, setRequestState] = useState("initial");
  const [error, setError] = useState(null);

  useEffect(() => {
    setRequestState("loading");
    fetch("https://api.com/get-thing/" + thingId)
      .then((r) => r.json())
      .then(setThings)
      .then(() => {
        setRequestState("success");
      })
      .catch((e) => {
        setRequestState("error");
        setError(e);
      });
  }, []);

  return (
    <div>
      <h1>The thing!</h1>
      {thing}
    </div>
  );
};
```

这对所有仅客户端渲染的 React 应用程序都存在一些问题：

我们向用户发送一个只加载代码的空白页面，然后解析和执行 JavaScript。

用户在 JavaScript 启动之前会下载一个空白页面，然后他们会得到我们的应用程序。如果用户是搜索引擎，则可能看不到任何内容。如果搜索引擎爬虫不支持 JavaScript，则搜索引擎将不会索引我们的网站。

我们开始太晚获取数据。

我们的应用程序遭遇了用户体验的诅咒，称为*网络瀑布*：这种现象发生在网络请求连续进行并减慢应用程序速度时。我们的应用程序必须对服务器进行多次请求以实现基本功能。例如，它执行如下：

+   下载、解析和执行 JavaScript。

+   渲染和提交 React 组件。

+   `useEffect`开始获取数据。

+   渲染和提交加载器等。

+   `useEffect`完成数据获取。

+   渲染和提交数据。如果我们直接向浏览器提供带有数据的页面，那么所有这些都可以避免：如果我们像第七章中介绍的那样，在服务器端渲染 React 时发送 HTML 标记。

我们的路由器完全基于客户端。

如果浏览器请求[*https://our-app.com/detail?thingId=24*](https://our-app.com/detail?thingId=24)，服务器将返回 404 页面，因为服务器上没有这样的文件。为了解决这个问题，通常会使用一个常见的技巧，当遇到 404 时渲染一个 HTML 文件，该文件加载 JavaScript 并由客户端路由接管。但这种方法对搜索引擎或者 JavaScript 支持有限的环境不起作用。

框架有助于解决所有这些问题以及更多。让我们具体探讨它们是如何做到的。

## 服务器端渲染

首先，框架通常会为我们提供服务器端渲染。要将服务器端渲染添加到此应用程序中，我们需要一个服务器。我们可以使用像 Express.js 这样的包自己编写一个服务器，然后部署这个服务器，这样就可以使用了。让我们探讨一下将为这样一个服务器提供动力的代码。

在我们继续之前，请注意我们仅仅出于简化和说明框架如何实现这些特性的底层机制，才使用`renderToString`。在真实的生产用例中，通常更好地依赖于更强大的异步 API，比如像`renderToPipeableStream`，就像第六章中所介绍的那样。

说完这些，让我们开始吧：

```
// ./server.js

import express from "express";
import { renderToString } from "react-dom/server"; // Covered in Chapter 6

import { List } from "./List";
import { Detail } from "./Detail";

const app = express();

app.use(express.static("./dist")); // Get static files like client JS, etc.

const createLayout = (children) => `<html lang="en">
<head>
 <title>My page</title>
</head>
<body>
  ${children}
 <script src="/clientBundle.js"></script>
</body>
<html>`;

app.get("/", (req, res) => {
  res.setHeader("Content-Type", "text/html");
  res.end(createLayout(renderToString(<List />)));
});

app.get("/detail", (req, res) => {
  res.setHeader("Content-Type", "text/html");
  res.end(
    createLayout(renderToString(<Detail thingId={req.params.thingId} />))
  );
});

app.listen(3000, () => {
  console.info("App is listening!");
});
```

这段代码就是我们为应用程序添加服务器端渲染所需的全部内容。请注意，在客户端的`index.js`中有其自己的客户端路由，并且我们基本上只是为服务器添加了另一个路由。框架提供*同构路由*，这些路由可以在客户端和服务器上工作。

## 路由

尽管这个服务器*还可以*，但它的扩展性不佳：每添加一个路由，我们就得手动添加更多的 `req.get` 调用。让我们让这个更具可扩展性一点。我们可以通过多种方式解决这个问题，比如使用一个配置对象将路由映射到组件，或者基于文件系统的路由。为了教育的目的（实际上也很有趣），让我们来探索*基于文件系统的路由*。这是 Next.js 等框架约定和观点背后推理及机制变得更加清晰的地方。当我们强制规定所有页面必须放在*./pages*目录中，且该目录中的所有文件名都成为路由路径时，我们的服务器可以依赖这个约定作为一个假设，从而变得更具可扩展性。

让我们用一个例子来说明这一点。首先，我们将扩展我们的目录结构。新的目录结构如下所示：

```
- index.js
- pages/
  - list.js
  - detail.js
- dist/
  - clientBundle.js
```

现在，我们可以假设`pages`中的所有内容都变成了一个路由。让我们更新我们的服务器以匹配这个结构：

```
// ./server.js

import express from "express";
import { join } from "path";
import { renderToString } from "react-dom/server"; // Covered in Chapter 6

const app = express();

app.use(express.static("./dist")); // Get static files like client JS, etc.

const createLayout = (children) => `<html lang="en">
<head>
 <title>My page</title>
</head>
<body>
  ${children}
 <script src="/clientBundle.js"></script>
</body>
<html>`;

app.get("/:route", async (req, res) => {
  // Import the route's component from the pages directory
  const exportedStuff = await import(
    join(process.cwd(), "pages", req.params.route)
  );

  // We can no longer have named exports because we need predictability
  // so we opt for default exports instead.
  // `.default` is standardized and therefore we can rely on it.
  const Page = exportedStuff.default;

  // We can infer props from the query string maybe?
  const props = req.query;

  res.setHeader("Content-Type", "text/html");
  res.end(createLayout(renderToString(<Page {...props} />)));
});

app.listen(3000, () => {
  console.info("App is listening!");
});
```

现在，由于我们采用了新的*./pages*目录约定，我们的服务器的扩展性大大提高了！太棒了！但是，现在我们被迫让每个页面的组件成为默认导出，因为我们的方法更通用，否则无法预测导入的名称。这是使用框架时的一种权衡。在这种情况下，这种权衡似乎是值得的。

## 数据获取

很棒！我们完成了三个目标中的两个。我们已经实现了服务器渲染和基于文件系统的路由，但我们仍然受到网络瀑布效应的影响。让我们解决数据获取的问题。首先，我们将更新我们的组件以通过 props 接收初始数据。为了简单起见，我们将只处理 `List` 组件，把 `Detail` 组件留给你做作业：

```
// ./pages/list.jsx
// Note the default export for filesystem-based routing.

export default function List({ initialThings } /* <- adding initial prop */) {
  const [things, setThings] = useState(initialThings);
  const [requestState, setRequestState] = useState("initial");
  const [error, setError] = useState(null);

  // This can still run to fetch data if we need it to.
  useEffect(() => {
    if (initialThings) return;
    setRequestState("loading");
    fetch("https://api.com/get-list")
      .then((r) => r.json())
      .then(setThings)
      .then(() => {
        setRequestState("success");
      })
      .catch((e) => {
        setRequestState("error");
        setError(e);
      });
  }, [initialThings]);

  return (
    <div>
      <ul>
        {things.map((thing) => (
          <li key={thing.id}>{thing.label}</li>
        ))}
      </ul>
    </div>
  );
}
```

很好。现在我们添加了一个初始 prop，我们需要一种方法来在服务器上获取这个页面所需的数据，然后将其传递给组件以在渲染之前使用。让我们探讨一下我们可以如何做到这一点。理想情况下，我们想要做的是这样的：

```
// ./server.js

import express from "express";
import { join } from "path";
import { renderToString } from "react-dom/server"; // Covered in Chapter 6

const app = express();

app.use(express.static("./dist")); // Get static files like client JS, etc.

const createLayout = (children) => `<html lang="en">
<head>
 <title>My page</title>
</head>
<body>
  ${children}
 <script src="/clientBundle.js"></script>
</body>
<html>`;

app.get("/:route", async (req, res) => {
  const exportedStuff = await import(
    join(process.cwd(), "pages", req.params.route)
  );

  const Page = exportedStuff.default;

  // Get component's data
  const data = await exportedStuff.getData();
  const props = req.query;

  res.setHeader("Content-Type", "text/html");

  // Pass props and data
  res.end(createLayout(renderToString(<Page {...props} {...data.props} />)));
});

app.listen(3000, () => {
  console.info("App is listening!");
});
```

这意味着我们需要从任何需要数据的页面组件导出一个名为 `getData` 的获取器函数！让我们调整列表来完成这一点：

```
// ./pages/list.jsx

// We'll call this on the server and pass these props to the component
export const getData = async () => {
  return {
    props: {
      initialThings: await fetch("https://api.com/get-list").then((r) =>
        r.json()
      ),
    },
  };
};

export default function List({ initialThings } /* <- adding initial prop */) {
  const [things, setThings] = useState(initialThings);
  const [requestState, setRequestState] = useState("initial");
  const [error, setError] = useState(null);

  // This can still run to fetch data if we need it to.
  useEffect(() => {
    if (initialThings) return;
    setRequestState("loading");
    getData()
      .then(setThings)
      .then(() => {
        setRequestState("success");
      })
      .catch((e) => {
        setRequestState("error");
        setError(e);
      });
  }, [initialThings]);

  return (
    <div>
      <ul>
        {things.map((thing) => (
          <li key={thing.id}>{thing.label}</li>
        ))}
      </ul>
    </div>
  );
}
```

搞定！现在我们是这样的：

+   在每个文件的每个路由上尽可能早地获取数据

+   作为 HTML 字符串渲染完整页面

+   将其发送到客户端

我们已经成功地添加并理解了我们从各种框架中识别并实现的三个特性的基本版本。通过这样做，我们学会了并理解了框架实现其功能的基本机制。具体来说，我们学会了框架如何：

+   为我们提供服务器渲染

+   具有受文件系统影响的同构路由

+   通过导出的函数获取数据

如果您之前使用过 Next.js 版本低于 13，那么它各种模式背后的推理应该在这一点上变得非常清晰，特别是围绕：

+   *./pages*目录

+   所有页面的导出都是默认导出

+   `getServerSideProps` 和 `getStaticProps`

现在我们了解了框架在代码层面的机制以及一些约定背后的原因，让我们放大视野，总结使用框架的好处。

# 使用框架的好处

使用框架的好处包括：

抽象

框架强制执行一定的结构和模式来组织代码库。这导致一致性，使新开发人员更容易理解应用程序的流程。它还使我们能够专注于我们的产品和功能，而不必担心如何构建代码的细枝末节。

最佳实践

框架通常内置了最佳实践，鼓励开发人员遵循。这可以提高代码质量，减少错误。例如，框架可能鼓励您尽早获取数据—即，在服务器端—而不是等待客户端获取数据。这可以提高性能和用户体验。

性能优化

框架提供更高级别的抽象来处理常见任务，如路由、数据获取、服务器渲染等。这可以使您的代码更清晰、更易读，更易于维护，同时依赖更广泛的社区来确保这些抽象的质量。Next.js 提供的`useRouter`钩子就是一个例子，它使得在组件中访问路由变得容易。

社区和生态系统

许多框架都带有开箱即用的优化，如代码拆分、服务器端渲染和静态站点生成。这些可以显著提高应用程序的性能。例如，Next.js 自动对应用程序进行代码拆分，并在用户悬停在链接上时预加载下一页的代码，从而实现更快的页面转换。

流行的框架拥有庞大的社区和丰富的插件和库生态系统。这意味着如果遇到问题，你通常可以快速找到解决方案或获得帮助。

流程和一致性

# 使用框架的权衡

尽管框架有许多优点，但也不是没有权衡之处。了解这些可以帮助您就是否使用框架以及选择哪个框架做出明智的决定：

学习曲线

每个框架都有自己的一套概念、API 和约定，您需要学习。如果您是 React 的新手，同时尝试学习一个框架可能会让人不知所措，但仍然值得推荐。如果您已经熟悉 React，您需要投入时间学习框架的特定功能和 API。

灵活性与约定

尽管框架强制的结构和约定可以是一个福音，但它们也可能是限制性的。如果你的应用程序有独特的要求，不适合框架的模型，你可能会发现自己与框架作斗争，而不是得到它的帮助。有些情况下，你为特定用户群体建设，这些用户有快速的互联网和现代浏览器，并且不需要服务器端渲染或数据获取。在这些情况下，框架可能会显得过于笨重。

依赖性和承诺

选择一个框架就是一种承诺。你正在将你的应用程序与框架的命运联系在一起。如果框架停止维护，或者采取了与你需求不符的方向，你可能需要面临是否进行昂贵迁移或自行维护现有框架代码的困难决策。

抽象层的开销

尽管抽象层可以通过隐藏复杂性来简化开发，但它们也可能创建“魔法”，使得很难理解底层发生了什么。这可能会使调试和性能优化变得复杂。此外，每个抽象层都伴随着一些开销，可能会影响性能。例如，在 Next.js 中的服务器动作中，使用`"use server"`指令会以某种魔法方式使动作在服务器上运行。这是一个很好的抽象，但理解它的工作原理可能会有难度。

现在我们明白了为什么要使用 React 框架，以及涉及其中的利弊后，我们可以深入探讨 React 生态系统中具体的框架。在本章的接下来的部分中，我们将探索一些流行的选择，如 Next.js 和 Remix。每个框架都提供独特的特性和优势，了解它们将帮助你选择适合特定需求的正确工具。

# 流行的 React 框架

让我们探索一些流行的 React 框架，并讨论它们的特点、优势和权衡。我们将从每个框架的简要概述开始，然后详细比较它们的特性和性能。在选择项目框架时，我们还将讨论一些需要考虑的因素。

## Remix

Remix 是一个强大的现代 Web 框架，利用了 React 和 Web 平台的特性。让我们从一些实际例子开始，了解它的工作原理。

### 一个基本的 Remix 应用程序

首先，我们将设置一个基本的 Remix 应用程序。你可以使用`npm`来安装 Remix：

```
npm create remix@2.2.0
```

这将在你当前的目录中创建一个新的 Remix 项目。让我们四处看看里面有什么。首先，我们有一个*app*目录，其中包含*entry.client.tsx*和*entry.server.tsx*。还有一个*root.tsx*在这个目录中。

一开始，我们可以立即看到 Remix 默认支持客户端和服务器端入口点。此外，*root.tsx* 包含一个共享布局组件，该组件在每个页面上都会渲染。这是 Remix 提供的一个预定义结构的很好示例，帮助您快速入门。

### 服务器端渲染

Remix 通过其 *entry.server.tsx* 默认支持服务器端渲染。文件由框架为我们生成，但让我们稍微理解一下它。它看起来是这样的：

```
import { PassThrough } from "node:stream";

import type { AppLoadContext, EntryContext } from "@remix-run/node";
import { createReadableStreamFromReadable } from "@remix-run/node";
import { RemixServer } from "@remix-run/react";
import isbot from "isbot";
import { renderToPipeableStream } from "react-dom/server";

const ABORT_DELAY = 5_000;

export default function handleRequest(
  request: Request,
  responseStatusCode: number,
  responseHeaders: Headers,
  remixContext: EntryContext,
  loadContext: AppLoadContext
) {
  return isbot(request.headers.get("user-agent"))
    ? handleBotRequest(
        request,
        responseStatusCode,
        responseHeaders,
        remixContext
      )
    : handleBrowserRequest(
        request,
        responseStatusCode,
        responseHeaders,
        remixContext
      );
}

function handleBotRequest(
  request: Request,
  responseStatusCode: number,
  responseHeaders: Headers,
  remixContext: EntryContext
) {
  return new Promise((resolve, reject) => {
    let shellRendered = false;
    const { pipe, abort } = renderToPipeableStream(
      <RemixServer
        context={remixContext}
        url={request.url}
        abortDelay={ABORT_DELAY}
      />,
      {
        onAllReady() {
          shellRendered = true;
          const body = new PassThrough();
          const stream = createReadableStreamFromReadable(body);

          responseHeaders.set("Content-Type", "text/html");

          resolve(
            new Response(stream, {
              headers: responseHeaders,
              status: responseStatusCode,
            })
          );

          pipe(body);
        },
        onShellError(error: unknown) {
          reject(error);
        },
        onError(error: unknown) {
          responseStatusCode = 500;
          // Log streaming rendering errors from inside the shell.  Don't log
          // errors encountered during initial shell rendering since they'll
          // reject and get logged in handleDocumentRequest.
          if (shellRendered) {
            console.error(error);
          }
        },
      }
    );

    setTimeout(abort, ABORT_DELAY);
  });
}

function handleBrowserRequest(
  request: Request,
  responseStatusCode: number,
  responseHeaders: Headers,
  remixContext: EntryContext
) {
  return new Promise((resolve, reject) => {
    let shellRendered = false;
    const { pipe, abort } = renderToPipeableStream(
      <RemixServer
        context={remixContext}
        url={request.url}
        abortDelay={ABORT_DELAY}
      />,
      {
        onShellReady() {
          shellRendered = true;
          const body = new PassThrough();
          const stream = createReadableStreamFromReadable(body);

          responseHeaders.set("Content-Type", "text/html");

          resolve(
            new Response(stream, {
              headers: responseHeaders,
              status: responseStatusCode,
            })
          );

          pipe(body);
        },
        onShellError(error: unknown) {
          reject(error);
        },
        onError(error: unknown) {
          responseStatusCode = 500;
          // Log streaming rendering errors from inside the shell.  Don't log
          // errors encountered during initial shell rendering since they'll
          // reject and get logged in handleDocumentRequest.
          if (shellRendered) {
            console.error(error);
          }
        },
      }
    );

    setTimeout(abort, ABORT_DELAY);
  });
}
```

Remix 的一个很棒的特性是，这个文件在内部使用，但在此处暴露出来供我们定制。如果我们删除这个文件，Remix 将会使用其内部默认的实现。这是一个很好的逃生口，允许我们根据需要定制服务器渲染行为，而不会被框架的“魔法”所束缚。

该文件定义了在我们的 Remix 应用程序中生成和处理 HTTP 响应的方式，特别是关于如何处理来自机器人和常规浏览器的请求的不同方式。Remix 是一个用于构建现代 React 应用程序的框架，而这个文件是 Remix 应用程序的服务器端逻辑的一部分。

最初，文件从各种库中导入必要的模块和类型，例如 `node:stream`，`@remix-run/node`，`@remix-run/react`，`isbot` 和 `react-dom/server`。它定义了一个名为 `ABORT_DELAY` 的常量，其值为 5,000 毫秒，用作渲染操作的超时期限。

文件导出了一个名为 `handleRequest` 的默认函数，该函数接受多个参数，包括 HTTP 请求、响应状态码、响应头以及 Remix 和应用程序加载过程的上下文。在 `handleRequest` 内部，它检查传入请求的用户代理，以确定请求是否来自使用 `isbot` 库的机器人。根据请求来自机器人还是浏览器，它将处理委托给 `handleBotRequest` 或 `handleBrowserRequest` 函数。

这有助于 SEO 和性能。例如，如果请求来自机器人，则确保响应包含页面的渲染 HTML 内容非常重要，这正是 `handleBotRequest` 的作用。另一方面，如果请求来自常规浏览器，则确保响应包含页面的渲染 HTML 内容以及必要的 JavaScript 代码以水合页面非常重要，这正是 `handleBrowserRequest` 的作用。很酷的是，Remix 自动为我们处理了这些。

`handleBotRequest`和`handleBrowserRequest`函数在结构上非常相似，但在渲染外壳准备就绪或遇到错误时有不同的处理程序。它们返回一个解析为 HTTP 响应的承诺。他们通过`renderToPipeableStream`启动一个到可管道流的渲染操作，传入一个`RemixServer`组件以及来自请求的必要上下文和 URL。他们定义了一个超时时间，如果渲染操作时间超过`ABORT_DELAY`，则中止渲染操作。

在渲染操作的事件处理程序中，他们创建了一个`PassThrough`流和从中读取的可读流。他们为响应设置了`Content-Type`头为`text/html`。他们使用封装了流、响应头和状态码的新`Response`对象来解析承诺。在渲染过程中出现错误时，他们要么拒绝承诺，要么将错误记录到控制台，这取决于错误发生的渲染阶段。

该文件主要确保正确生成并返回 HTTP 响应，根据请求来自机器人还是常规浏览器的不同渲染逻辑应用，这对于现代 Web 应用程序的 SEO 和性能考虑至关重要。

如果我们没有自定义内容要添加，我们可以简单地删除这个文件，Remix 会为我们处理服务器渲染。现在让我们保留它，并探索一下 Remix 如何处理路由。

### 路由

在 Remix 中，每个路由都由*routes*目录中的一个文件表示。如果我们创建*./routes/cheese.tsx*，其默认导出为：

```
export default function CheesePage() {
  return <h1>This might sound cheesy, but I think you're really grate!</h1>;
}
```

然后用`npm run dev`运行本地开发服务器，我们会看到一个有趣标题的页面。再次看到 Remix 提供了一个预定义的结构，帮助您快速入门，而这种约定中默认导出的价值类似于我们之前基于文件系统的路由实现。当与*./app/root.tsx*中的共享布局组件以及服务器和客户端入口点结合使用时，这构成了大多数网站的基础。然而，对于现代 Web 应用程序，我们仍然缺少一个至关重要的组件：数据获取。让我们看看 Remix 如何处理这个问题。

### 数据获取

在撰写本文时，Remix 的数据获取故事涉及使用称为*loaders*的函数。当您导出一个名为`loader`的异步函数，返回某个值时，这个值通过`useLoaderData`钩子可在页面组件中使用。让我们通过一个示例看看这是如何工作的。

要回到我们的奶酪页面，假设我们想从 API 获取奶酪列表并在页面上显示它们。我们可以通过从*./routes/cheese.tsx*导出一个`loader`函数来实现这一点：

```
// Get some utils
import { json } from "@remix-run/node";
import { useLoaderData } from "@remix-run/react";

// The loader
export async function loader() {
  const data = await fetch("https://api.com/get-cheeses");
  return json(await data.json());
}

export default function CheesePage() {
  const cheeses = useLoaderData();
  return (
    <div>
      <h1>This might sound cheesy, but I think you're really grate!</h1>
      <ul>
        {cheeses.map((cheese) => (
          <li key={cheese.id}>{cheese.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

通过这个例子，我们可以看到我们之前实现的数据获取的一种重复。我们可以看到 Remix 的 `loader` 函数与我们自己的 `getData` 函数相似。我们还可以看到 `useLoaderData` hook 与我们自己的 `initialThings` prop 相似。理想情况下，此时我们能够了解框架如何实现这些功能背后的共同模式和基本机制。

到目前为止，我们已经涵盖了：

+   服务器渲染

+   路由

+   数据获取

但是还有一个 Remix 功能我们还没有涉及到：表单和服务器操作，或者说变异——即在服务器上改变数据，比如创建、更新或删除数据。让我们接着来探索这个。

### 数据变异

Remix 负责将网络带回基础，严重依赖于原生网络平台的约定和行为。这在数据变异和 Remix 对表单的使用上最为明显。让我们通过一个例子来探索这一点，扩展我们之前的奶酪例子：让奶酪列表可变。为此，让我们首先更新我们的 *./routes/cheese.tsx* 文件：

```
// Get some utils
import { json } from "@remix-run/node";
import { useLoaderData } from "@remix-run/react";

// The loader
export async function loader() {
  const data = await fetch("https://api.com/get-cheeses");
  return json(await data.json());
}

export default function CheesePage() {
  const cheeses = useLoaderData();
  return (
    <div>
      <h1>This might sound cheesy, but I think you're really grate!</h1>
      <ul>
        {cheeses.map((cheese) => (
          <li key={cheese.id}>{cheese.name}</li>
        ))}
      </ul>
      <form action="/cheese" method="post">
        <input type="text" name="cheese" />
        <button type="submit">Add Cheese</button>
      </form>
    </div>
  );
}
```

注意我们在页面中添加了一个新的 `form` 元素。这个表单的 action 是 `/cheese`，方法是 `post`。这是一个标准的 HTML 表单，将向 `/cheese` 路由提交一个 POST 请求。此外，`input` 元素有一个 `name` 属性，没有 `useState` 或 `onChange` 处理程序：Remix 让浏览器管理表单的状态和行为。这是 Remix 如何依赖网络平台提供出色开发者体验的一个很好的例子，而不是让 React 管理一切。

鉴于表单的 action 属性是 `/cheese`，而我们已经在 *./routes/cheese.tsx* 文件中，我们可以假设表单将提交到相同的路由。当这个路由使用 `POST` 方法访问时，我们知道表单已经提交。当这个路由默认使用 `GET` 方法访问时，我们知道表单尚未提交，而是显示初始 UI。

让我们更新我们的 *./routes/cheese.tsx* 文件来处理这个问题：

```
// Get some utils
import { json, ActionFunctionArgs, redirect } from "@remix-run/node";
import { useLoaderData } from "@remix-run/react";

// The loader
export async function loader() {
  const data = await fetch("https://api.com/get-cheeses");
  return json(await data.json());
}

// The form action
export async function action({ params, request }: ActionFunctionArgs) {
  const formData = await request.formData();

  await fetch("https://api.com/add-cheese", {
    method: "POST",
    body: JSON.stringify({
      name: formData.get("cheese"),
    }),
  });

  return redirect("/cheese"); // Come back to this page, but with GET this time.
}

export default function CheesePage() {
  const cheeses = useLoaderData();
  return (
    <div>
      <h1>This might sound cheesy, but I think you're really grate!</h1>
      <ul>
        {cheeses.map((cheese) => (
          <li key={cheese.id}>{cheese.name}</li>
        ))}
      </ul>
      <form action="/cheese" method="post">
        <input type="text" name="cheese" />
        <button type="submit">Add Cheese</button>
      </form>
    </div>
  );
}
```

注意我们添加了一个新的 `action` 函数，它接受 `params` 和 `request` 参数。`params` 参数是一个包含路由参数的对象。`request` 参数是一个包含请求对象的对象。我们可以使用这个对象从请求中获取表单数据，然后使用它向我们的 API 发送请求以添加新的奶酪。

然后我们返回一个重定向到相同的路由，但这次使用 `GET` 方法。这将导致页面重新加载，并且 `loader` 函数将再次被调用以获取更新后的奶酪列表。

这就是 Remix 如何充分依赖网络平台，使 JavaScript 能够在需要的地方使用，并让浏览器处理其余部分。如果这个页面没有使用 JavaScript 访问，它将正常工作，因为它依赖于网络平台。如果页面使用了 JavaScript，Remix 通过添加交互性和更好的用户体验逐步增强体验。

到目前为止，我们已经介绍了 Remix 如何：

+   提供服务器渲染

+   处理路由

+   处理数据获取

+   处理数据变化

此时，我们应该能够看到我们自己实现这些功能与 Remix 实现之间的强烈对比。这表明我们正在理解框架在实现这些功能背后的机制。

现在让我们考虑一下 Next.js，以及它如何与隔离这些功能背后的共同点做非常类似的事情。

## Next.js

Next.js 是由 Vercel 开发的流行 React 框架，以其丰富的功能和简单性在创建服务器端渲染（SSR）和静态网站方面享有盛誉。它遵循“约定优于配置”的原则，减少了启动项目所需的样板代码和决策。随着 Next.js 13 的发布，引入了 Next.js 应用程序路由器是一项重大的新增功能。

让我们通过一个基本的 Next.js 应用程序来了解其工作原理。首先，让我们运行以下命令创建一个新的 Next.js 项目：

```
npm create next-app@14
```

这将提出一些问题，但最终我们将得到一个基本的 Next.js 项目。让我们四处看看里面有什么。首先，我们有一个*app*目录，包括*page.tsx*、*layout.tsx*、*error.tsx*和*loading.tsx*。

我们立即注意到的一件事是，Next.js 不像 Remix 那样暴露服务器配置，而是隐藏了大量的复杂性，旨在“避让”，让开发者专注于构建他们的应用程序。这是不同框架在解决相同问题时具有不同哲学和方法的一个很好的例子。

让我们在之前确定的三个关键特性（服务器渲染、路由和数据获取）的背景下探索 Next.js。

### 服务器渲染

Next.js 不仅提供服务器渲染，而且是面向服务器的。Next.js 中的每个页面和组件都是服务器组件。我们将在第九章中深入探讨服务器组件，但现在可以简单地说，服务器组件是专门在服务器上渲染的组件。目前这种理解已经足够，因为重点在于 Next.js，而不是服务器组件。对于服务器组件，第九章应该已经足够了。

在 Next.js 的背景下，这意味着什么呢？实质上，我们要基于以下原则操作：除非在给定的路由或组件顶部添加`"use client"`指令，否则所有编写的代码都将在服务器上执行。没有这个指令，所有代码都被视为服务器端代码。

然而，Next.js 也是以静态为先：在构建时，服务器组件尽可能地渲染成静态内容，然后部署。这种服务器为先和静态为先的结合正是 Next.js 如此强大的地方，显著地优化了性能，因为静态内容可以说是最快到达用户的，因为不需要运行时或服务器端处理；它只是文本（HTML）。从静态内容进一步发展的是服务器渲染内容，可以高度优化和缓存，但仍需要服务器来渲染内容。最后一步是通过水合处理客户端渲染内容，用于页面的交互部分。

借助这种方法，Next.js 很适合将较小的 JavaScript 包发送给用户，大部分内容都是一些静态和服务器渲染的标记的混合体。不仅页面，而且组件在服务器上的渲染粒度是 Next.js 的一个强大功能，它支持一些非常强大的数据获取和渲染模式。在深入探讨这些模式之前，让我们先了解一下 Next.js 如何处理路由。

### 路由

在我们新的 Next.js 项目中看到的是一个 *app* 目录，包含 *layout.tsx* 和 *page.tsx*。Next.js 遵循这种模式：你在浏览器中看到的用户页面的 URL 是该页面所在目录的名称，*app* 相当于根目录（*/*），其下的每个目录都成为一个子路径。

要理解这一点，让我们创建一个名为 *cheese* 的目录，并在其中添加一个 *page.tsx* 文件。当 *./app* 下的目录有一个 *page.tsx* 文件时，该目录就成为一个路由。让我们向 *./app/cheese/page.tsx* 添加一些内容：

```
export default function CheesePage() {
  return <h1>This might sound cheesy, but I think you're really grate!</h1>;
}
```

现在，如果我们运行开发服务器并导航到 */cheese*，我们将看到一个具有有趣标题的页面。这里值得注意的是，Next.js 还有一个共享布局的概念，类似于 Remix，在这里你可以在 *./app/layout.tsx* 中定义一个布局组件，它将在每个页面上呈现。然后，*./app/cheese/layout.tsx* 将在 */cheese* 路由下的每个页面上呈现。布局通常是跨多个页面共享的路由部分，例如页眉、页脚或其他固定元素。

很好，这就是 Next.js 处理路由的方式。它有点类似于 Remix 和我们基于文件系统的路由实现，但有一个细微的差异，即不是单个文件成为页面，而是整个目录，实际页面总是要求命名为 *page.tsx*。除此之外，它们非常相似。

让我们谈谈数据获取。

### 数据获取

因为每个组件都是服务器组件，所以 Next.js 中的每个组件都能够是异步的，因此可以等待数据。让我们尝试像在前面的 Remix 示例中那样获取奶酪，但这次在 Next.js 中进行：

```
export default async function CheesePage() {
  const cheeses = await fetch("https://api.com/get-cheeses").then((r) =>
    r.json()
  );
  return (
    <div>
      <h1>This might sound cheesy, but I think you're really grate!</h1>
      <ul>
        {cheeses.map((cheese) => (
          <li key={cheese.id}>{cheese.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

如果你读到并且感到印象深刻——是的。这种语法是 React 工程师多年来一直希望的，而且非常自然地可以理解。这是可能的，因为这里的 `CheesePage` 是一个服务器组件：它不包含在客户端捆绑包中，而是在服务器上渲染。这意味着我们可以 `await` 数据并直接渲染到页面。

因为所有组件都是服务器组件，我们可以进一步增加粒度，不仅在页面级别获取数据，还可以在组件级别获取数据。考虑将这个页面拆分成更小的组件，其中 `CheeseList` 可以重复使用，也可以在其他地方使用。

我们的页面将变成这样：

```
import { CheeseList } from "./CheeseList";

export default function CheesePage() {
  return (
    <div>
      <h1>This might sound cheesy, but I think you're really grate!</h1>
      <CheeseList />
    </div>
  );
}
```

我们的 `CheeseList` 组件将是这样的：

```
export async function CheeseList() {
  const cheeses = await fetch("https://api.com/get-cheeses").then((r) =>
    r.json()
  );
  return (
    <ul>
      {cheeses.map((cheese) => (
        <li key={cheese.id}>{cheese.name}</li>
      ))}
    </ul>
  );
}
```

这种方法的真正威力在于我们可以在组件级别获取数据，然后将其渲染到页面上。我们不导出页面级别的函数，比如 `loader`、`getData`、`getServerSideProps`、`getStaticProps` 或类似的东西。相反，我们只是在组件级别获取数据并将其渲染到页面上。

这些数据会怎样呢？Next.js 在我们部署时用它来静态生成页面的第一次加载，并在后续加载时进行服务器渲染。Next.js 还有许多缓存和去重机制，确保数据的完整性和性能。

最后，让我们通过探索 Next.js 如何处理数据变更来完成比较。

### 数据变异

Next.js 有一个 *服务器动作* 的概念，这些是在服务器上运行的函数。这些函数在提交表单、用户点击按钮或用户导航到页面时调用。它们是在服务器上运行的函数，不包含在客户端捆绑包中。

让我们看看如何像在 Remix 示例中那样向列表中添加一种奶酪。为此，我们将在页面上添加一个表单：

```
import { CheeseList } from "./CheeseList";

import { redirect } from "next/navigation";
import { revalidatePath } from "next/cache";

export default function CheesePage() {
  return (
    <div>
      <h1>This might sound cheesy, but I think you're really grate!</h1>
      <CheeseList />
      <form
        action={async (formData) => {
          "use server";
          await fetch("https://api.com/add-cheese", {
            method: "POST",
            body: JSON.stringify({
              name: formData.get("cheese"),
            }),
          });
          revalidatePath("/cheese");
          return redirect("/cheese");
        }}
        method="post"
      >
        <input type="text" name="cheese" />
        <button type="submit">Add Cheese</button>
      </form>
    </div>
  );
}
```

这里我们使用了一个类似 Remix 的标准 HTML 表单，只是这次的 `action` 属性是一个函数。这个函数是一个服务器动作，在表单提交时调用。这个函数不包含在客户端捆绑包中，而是在服务器上运行。这是通过顶部的 `"use server"` 指令强制执行的。

我们可以把这个函数移到任何我们想要的地方，包括放入服务器组件的主体中，如下所示：

```
import { redirect } from "next/navigation";
import { revalidatePath } from "next/cache";

export default function CheesePage() {
  async function addCheese(formData) {
    "use server";
    await fetch("https://api.com/add-cheese", {
      method: "POST",
      body: JSON.stringify({
        name: formData.get("cheese"),
      }),
    });

    revalidatePath("/cheese");
    return redirect("/cheese");
  }

  return (
    <div>
      <h1>This might sound cheesy, but I think you're really grate!</h1>
      <CheeseList />
      <form action={addCheese} method="post">
        <input type="text" name="cheese" />
        <button type="submit">Add Cheese</button>
      </form>
    </div>
  );
}
```

或者甚至放入一个单独的模块，如下所示：

```
import { addCheeseAction } from "./addCheeseAction";

export default function CheesePage() {
  return (
    <div>
      <h1>This might sound cheesy, but I think you're really grate!</h1>
      <CheeseList />
      <form action={addCheese} method="post">
        <input type="text" name="cheese" />
        <button type="submit">Add Cheese</button>
      </form>
    </div>
  );
}
```

在这种情况下，`addCheeseAction` 将位于自己的文件中，并读取如下：

```
"use server";

import { redirect } from "next/navigation";
import { revalidatePath } from "next/cache";

export async function addCheeseAction(formData) {
  await fetch("https://api.com/add-cheese", {
    method: "POST",
    body: JSON.stringify({
      name: formData.get("cheese"),
    }),
  });

  revalidatePath("/cheese");
  return redirect("/cheese");
}
```

这里存在一个固有的问题，与 Remix 不同，所有组件都是客户端组件，服务器组件根本不支持交互，因为它们不包含在客户端捆绑包中，也从不被浏览器加载；因此，`onClick` 处理程序实际上从不传递给用户。为了解决这个问题，Next.js 有一个客户端组件的概念，这些组件包含在客户端捆绑包中，并由浏览器加载。这些组件不是服务器组件，因此不能是异步的，也不能有服务器动作。

让我们探讨如何添加一个奶酪，但这次使用一些服务器和客户端组件的混合方式。这也将帮助我们在表单提交时立即通过旋转器或类似方式提供反馈。为此，我们将创建一个新组件，*./app/AddCheeseForm.tsx*：

```
"use client";
import { addCheeseAction } from "./addCheeseAction";

export function AddCheeseForm() {
  return (
    <form action={addCheeseAction} method="post">
      <input type="text" name="cheese" />
      <button type="submit">Add Cheese</button>
    </form>
  );
}
```

现在它是一个客户端组件，我们可以做交互式的事情——比如响应表单状态的变化。让我们更新我们的`AddCheeseForm`来实现这一点：

```
"use client";
import { addCheeseAction } from "./addCheeseAction";
import { useFormStatus } from "react-dom";

export function AddCheeseForm() {
  const { pending } = useFormStatus();

  return (
    <form action={addCheeseAction} method="post">
      <input disabled={pending} type="text" name="cheese" />
      <button type="submit" disabled={pending}>
        {pending ? "Loading..." : "Add Cheese"}
      </button>
    </form>
  );
}
```

因为我们的`AddCheeseForm`是一个客户端组件，我们可以使用`useFormStatus`来获取表单的状态。这是 React 提供的一个 hook。这个 hook 返回一个对象，其中有一个`pending`属性，当表单正在提交时为`true`，当表单没有提交时为`false`。我们可以利用这一点在表单提交时禁用表单，并显示加载指示器。

现在，我们可以在我们的页面中使用这个表单，就像这样一个服务器组件：

```
import { CheeseList } from "./CheeseList";
import { AddCheeseForm } from "./AddCheeseForm";

export default function CheesePage() {
  return (
    <div>
      <h1>This might sound cheesy, but I think you're really grate!</h1>
      <CheeseList />
      <AddCheeseForm />
    </div>
  );
}
```

因此，我们有了一些服务器和客户端组件。`CheesePage`和`CheeseList`是服务器组件，而`AddCheeseForm`是客户端组件。这两个组件都是可重用的，可以在我们的应用程序的其他地方使用。关于客户端和服务器组件还有一些规则和考虑事项，但我们将在第九章中探讨这些内容。

现在，如果我们放大看，我们可以看到 Next.js 解决了与 Remix 和我们基于文件系统的路由、数据获取和数据变更类似的问题。它的方式略有不同，但底层机制有些相似。

理想情况下，通过探索这两个框架，我们能够理解为什么我们会选择框架，它们解决的问题以及它们如何为我们带来好处。

让我们通过讨论如何选择一个框架来结束。

# 选择一个框架

决定为您的项目选择哪个 React 框架可能是一个具有挑战性的决定，因为每个框架都提供了一套独特的特性、优势和权衡。在本节中，我们将尝试为您提供一些关于当今流行 React 框架为开发者提供可行选项的见解，并讨论学习曲线、灵活性和性能等因素，这些因素可以帮助您选择最适合您特定需求的框架。

值得注意的是，一个框架并不是比另一个框架更好或更差。每个框架都有其自身的优势和劣势，最适合您项目的框架将取决于您的具体要求和偏好。

## 理解您的项目需求

在我们深入探讨每个框架的细节之前，了解项目的具体需求非常重要。以下是一些需要考虑的关键问题：

+   您的项目范围是什么？它是一个小型个人项目，还是一个具有多个功能的中型应用程序，或者是一个大规模复杂的应用程序？

+   您希望在您的项目中包含哪些主要功能和特性？

+   您是否需要服务器端渲染（SSR）、站点生成（SSG）或两者的组合？

+   您是否正在构建像博客或电子商务站点这样的内容密集型站点，可能受益于优秀的 SEO？

+   实时数据或高度动态内容是否是您应用程序的关键部分？

+   在构建过程中，您对自定义和控制的需求有多大？

+   您的应用程序的性能和速度有多重要？

+   您在 React 和一般 Web 开发概念方面的熟练程度如何？

+   您的目标用户是谁？坐在桌子前拥有快速互联网的企业人士？还是拥有各种设备和互联网速度的广大公众？

了解这些问题的答案将为您提供一个更清晰的框架需求图景。

## Next.js

让我们在 Next.js 的背景下探讨一些这些参数：

学习曲线

Next.js 在内部使用 React 的最前沿技术，经常使用 React 的 canary 版本。这意味着 Next.js 经常走在潮流的前沿，可能会更具挑战性。然而，Next.js 团队在框架文档和各种功能的清晰指南方面做得非常出色，这可以帮助您快速入门。

灵活性

Next.js 设计时考虑了静态和服务器渲染内容之间的灵活性。它也支持完全客户端应用程序，尽管这不是它的主要用例。Next.js 还提供了丰富的插件和集成生态系统，可以显著加快开发过程。

性能

Next.js 强调性能优化，专注于静态生成和服务器端渲染，以及缓存。在撰写时，Next.js 发布了四种不同的用途驱动缓存，每种都旨在为多种用例提供最佳性能。然而，这种性能可能会在客户端/服务器边界之间引起摩擦，并且在何时使用哪种决策上需要考虑周到。

还值得注意的是，构建 React 的团队中的一些成员在 Vercel 工作，Next.js 的开发地，这表明 Next.js 和 React 之间存在非常紧密的开发反馈循环。

## Remix

相比于 Next.js，Remix 是 React 框架领域的较新参与者，大约早了 10 年左右创建。它由 React Router 的创作者构建，强调 Web 基础知识，做出较少假设并提供了很多灵活性：

学习曲线

Remix 可能有稍平缓的学习曲线，因为它更多地依赖于 Web 基础知识，并且以许多人在更重视服务器组件之前学习的方式使用 React。

直觉性

Remix 往往退到一边，允许 Web 平台的基础知识发挥作用。这可能是一把双刃剑：一方面很好，因为直观和熟悉，但另一方面可能有点令人沮丧，因为它没有其他框架那么“神奇”。

性能

Remix 独特的路由和数据加载方法使其高效且性能优越。由于数据获取与路由绑定，仅会获取特定路由所需的必要数据，从而减少了整体数据需求。此外，其乐观的 UI 更新和渐进增强策略提升了用户体验。

## 权衡

选择一个框架并不是没有权衡，其中最重要的是便利性与控制之间的权衡。所有框架通过传统化的方式，大大减少了我们在应用程序中的脑力劳动和决策过程。例如，默认情况下，框架已经为问题提供了答案，比如：

+   我们如何进行路由？

+   静态资源放在哪里？

+   我们应该使用服务器端渲染吗？

+   我们从哪里获取数据？

考虑到框架在这些主题及更多方面的强烈约定化，这使得开发者失去了一些控制权。作为交换，我们获得了相当多的便利性，可以更专注于应用程序的核心部分，比如业务逻辑本身。

大多数情况下，关于框架的权衡都围绕着这一连续性展开。

那么，如何选择合适的框架呢？关键在于你的项目需求和个人偏好：

+   如果你需要一个相对灵活的全栈框架，Next.js 可能更合适，因为它允许你在静态、服务器端或完全客户端的应用之间进行选择。

+   如果你偏好一种服务器端优先、渐进增强且坚持 Web 基础知识的方法，Remix 可能是你的最佳选择。

无论如何，尝试在一个较小的项目或应用程序的一部分中使用其中一个是个好主意。这将帮助你更好地理解它们的工作方式，以及哪个对你来说更舒适。

## 开发者体验

两种框架都提供世界级的开发者体验，专注于生产力和易用性。它们都提供丰富的功能和工具，帮助开发者构建高质量的应用程序，正如我们在本章中前面所见。

随着项目复杂性和规模的增长，构建性能变得越来越关键。Next.js 和 Remix 都进行了优化，以改善构建时间。

Next.js 默认使用静态生成，这意味着页面在构建时预渲染。这可以带来更快的页面加载速度，但对于页面数量较多的站点，构建时间可能较长。

为了解决这个问题，Next.js 引入了增量静态再生（Incremental Static Regeneration，ISR），允许开发者在不进行完整重建的情况下重新生成静态页面。这一功能可以显著提高大型动态网站的构建时间。

Remix 则采用独特的构建性能理念。它选择了一种以服务器为先的架构，这意味着页面会按需由服务器渲染，并将 HTML 发送到客户端。

## 运行时性能

Next.js 和 Remix 都是以性能为设计核心，并提供多种优化方式，以实现快速响应的应用程序。

Next.js 自带了几种内置的性能优化。它支持自动代码分割，确保每个页面只加载必要的代码。它还有一个内置的 Image 组件，优化图像加载以提升性能。

Next.js 中的混合 SSG/SSR 模型允许开发者为每个页面选择最佳的数据获取策略，平衡性能和实时性。不需要实时数据的页面可以在构建时预渲染，从而实现更快的页面加载。对于需要实时数据的页面，可以使用服务器端渲染或 ISR。

Next.js 还为没有阻塞数据需求的页面提供了自动静态优化，确保它们作为静态 HTML 文件提供，从而实现更快的首字节时间（TTFB）。

最后，Next.js 尽可能地利用 React Server Components，使得它能向客户端发送更少的 JavaScript，从而实现更快的页面加载和其他开销。

Remix 在性能方面采用了略有不同的方法。它选择了服务器渲染而不是预渲染页面，仅向客户端流式传输客户端需要的 HTML。这可能会导致更快的 TTFB，特别是对于动态内容而言。

Remix 的一个关键特性是其强大的缓存策略。它利用浏览器的原生 fetch 和 cache API，允许开发者为不同的资源指定缓存策略。这导致更快的页面加载和更具韧性的应用程序。

Next.js 和 Remix 都为大型、复杂项目提供了引人注目的优势。它们在开发者体验、构建性能和运行时性能方面都表现出色。如果你偏好成熟的生态系统、丰富的资源和插件以及混合的 SSG/SSR 模型，以及像 ISR 这样的创新功能，那么 Next.js 可能是一个更好的选择。另一方面，如果你更喜欢采用服务器渲染方法，支持即时部署，强调利用网络平台特性如 fetch 和 cache APIs，并且涉及高级 React 概念如 Suspense 和 Server Components，那么 Remix 可能更适合你。

对于你特定项目需求来说，最合适的框架最终取决于你团队的专业知识、项目需求以及你对某些架构模式的偏好。无论选择哪种，Next.js 和 Remix 都是构建高质量、高性能 React 应用程序的坚实基础。

# 章节回顾

在本章的过程中，我们深入探讨了 React 框架的概念。本章使我们能够探索使用框架的基本原理、推理以及实际应用的影响。

讨论从回顾并发 React 及其对高效渲染和用户交互的影响开始。然后，我们继续探讨 React 框架的“为什么”和“什么”：为什么它们是必要的，它们提供了哪些好处，以及它们涉及了哪些权衡。

我们通过实现我们自己的基本框架来做到这一点，这使我们能够理解 React 框架背后的基本机制和概念。然后，我们探讨了基于文件系统的路由概念，在许多 React 框架中是一个常见的特性。我们还研究了数据获取以及如何在框架中实现它。

接下来，我们深入比较了不同框架，主要集中在 Next.js 和 Remix 上。每个框架都提供其独特的功能和优势，选择往往取决于项目的具体需求。我们探讨了这些框架如何解决服务器渲染、路由、数据获取和数据变更，并且将它们与我们自己的实现进行了比较。

通过这一过程，我们通过我们自己实现与框架之间的共同点，获得了对机制的理解。我们还探讨了使用框架所涉及的权衡，并通过理解其基本机制来缓解这些问题。

最后，我们讨论了如何选择一个框架，并探讨了此决策所涉及的一些权衡。我们还研究了框架的开发者体验和运行时性能，并考虑了对我们的项目可能最好的选择。

# 复习问题

在本章结束时，这里有一些问题帮助您复习我们所涵盖的概念：

+   使用像 Next.js 或 Remix 这样的 React 框架的主要原因是什么，它们提供了哪些好处？

+   使用 React 框架存在哪些权衡或不利之处？

+   框架解决了哪些常见问题？

+   这些框架是如何解决这些问题的呢？

# 接下来

在本章中，我们简要提到了 React 服务器组件，并以一种粗略的方式开始探索它们。在下一章中，我们将更加深入地关注 React 服务器组件，理解它们的价值主张，并通过编写一个最小的服务器来理解它们的工作原理，以渲染和提供 React 服务器组件。

另外，我们将探讨为什么 React 服务器组件需要像捆绑器、路由器等新一代构建工具。最终，我们将更加深入地理解 React 服务器组件及其背后的机制，在这必定是一个富有信息和教育性的深入探讨中获益良多。
