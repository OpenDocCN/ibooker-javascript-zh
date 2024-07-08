# 第二十章：远程数据

数据围绕着我们。我们在日常生活中创建和交互的数据往往以有趣且意想不到的方式存在。在构建 Node 应用程序时，我们经常与数据交互。有时，这些数据可能是我们为应用程序创建的，或者是用户输入到我们系统中的数据。然而，通常需要与来自应用程序外部的数据进行交互。本章介绍了在 Node 应用程序中处理远程数据的最佳实践和技术。

# 获取远程数据

## 问题

您希望在 Node 应用程序中向远程服务器发出请求。

## 解决方案

使用 `node-fetch`，这是最受欢迎和广泛使用的模块之一，将浏览器的 `window.fetch` 带到 Node 中。它通过 npm 安装：

```
$ npm install node-fetch
```

并且可以简单地使用：

```
const fetch = require('node-fetch');

fetch('https://oreilly.com')
  .then(res => res.text())
  .then(body => console.log(body));
```

## 讨论

`node-fetch` 提供了一个 API，紧密模仿了浏览器的 `window.fetch`，允许我们的 Node 程序访问远程资源。与 `window.fetch` 类似，它支持 GET、POST、DELETE 和 PUT 的 HTTP 方法。在 GET 的情况下，如果响应表示成功（状态码为 200），则可以按照您希望的方式处理返回的数据（在此示例中格式为 HTML）。

您可以请求 JSON 资源：

```
fetch('https://swapi.dev/api/people/1')
  .then(res => res.json())
  .then(json => console.log(json));
```

还可以使用 `async/await` 语法，包括 `try/catch` 块进行错误处理：

```
(async () => {
  try {
    const response = await fetch('https://swapi.dev/api/people/3');
    const json = await response.json();
    console.log(json);
  } catch (error) {
    console.log(error);
  }
})();
```

你还可以使用文件系统模块将结果流到文件中：

```
const fs = require('fs');
const fetch = require('node-fetch');

fetch('https://example.com/image.png')
  .then(res => {
    const dest = fs.createWriteStream('image.png');
    res.body.pipe(dest);
  });
```

`node-fetch` 还可以处理 POST、DELETE 和 PUT 方法，允许您向服务器发送数据。在以下示例中，我们进行了 `POST` 请求：

```
// example body for the request
const body = {
  id: 1,
  title: "Example"
};

fetch('https://example.com/post', {
    method: 'post',
    body:    JSON.stringify(body),
    headers: { 'Content-Type': 'application/json' },
  })
  .then(res => res.json())
  .then(json => console.log(json));
```

###### 注意

`node-fetch` 是用于获取远程数据的常见和有用的库，但不是唯一的选择。流行的替代方案包括 Request（虽然仍然流行，但不再活跃维护），Got，Axios 和 Superagent。

# 屏幕抓取

## 问题

您希望从 Node 应用程序内部访问 web 资源的特定内容。

## 解决方案

使用 `node-fetch` 和 Cheerio 模块进行*屏幕抓取*网站。

首先安装所需的模块：

```
$ npm install node-fetch cheerio
```

要抓取页面，利用 `node-fetch` 检索内容，然后使用 Cheerio 查询检索到的内容：

```
const fetch = require('node-fetch');
const cheerio = require('cheerio');

fetch('https://example.com')
  .then(res => res.text())
  .then(body => {
    const $ = cheerio.load(body);
    $('h1').each((i, element) => {
      console.log(element.children[0].data);
    });
  });
```

## 讨论

一个有趣的 Node 使用方法是*抓取*网站或资源，然后使用其他功能查询返回材料中的特定信息。用于查询的流行模块是 Cheerio，这是一个专为服务器使用的 jQuery 核心的微型实现。在以下示例中，创建了一个简单的应用程序来提取 O’Reilly Radar 博客页面上的所有文章标题。为了选择这些标题，我们使用 Cheerio 来查找位于 `main` 内容中的 `h2` 元素内部的链接 (`a`)。然后，将链接的文本列出到单独的输出中：

```
const fetch = require('node-fetch');
const cheerio = require('cheerio');

fetch('https://www.oreilly.com/radar/posts/')
  .then(res => res.text())
  .then(body => {
    const $ = cheerio.load(body);
    $('main h2 a').each((i, element) => {
      console.log(element.children[0].data);
    });
  });
```

成功请求后，通过`load()`方法将返回的 HTML 传递给 Cheerio，并将结果赋值给一个美元符号变量（`$`），以便我们可以类似于 jQuery 库的方式选择结果中的元素。

然后使用`main h2 a`元素模式查询所有匹配项，并使用`each`方法处理结果，访问每个标题的文本。控制台的输出应该是博客主页上所有文章的标题。

一个常见的用例是在没有提供 API 的情况下下载数据。在以下示例中，我们正在定位页面上的特定链接，并将链接的资源传输到本地文件。我还使用了`async/await`语法来演示如何使用它：

```
const path =
  'data-research/mortgage-performance-trends/mortgages-30-89-days-delinquent/';
const url = `https://www.consumerfinance.gov/${path}`;

(async () => {
  try {
    const response = await fetch(url);
    const body = await response.text();
    const $ = cheerio.load(body);
    $("a:contains('state')").each(async (i, element) => {
      const fetchFile = await fetch(element.attribs.href);
      const dest = fs.createWriteStream(`data-${i}.csv`);
      await fetchFile.body.pipe(dest);
    });
  } catch (error) {
    console.log(error);
  }
})();
```

我们首先获取特定 URL 上的页面，这个例子中是一个包含几个链接的美国政府网站的页面。然后我们使用 Cheerio 定位页面上所有包含“state”单词的链接。最后，我们获取链接的文件并将其传输到本地文件。

###### 警告

屏幕抓取可以是您工具箱中有用的工具，但请谨慎操作。在为生产应用程序抓取网站之前，请务必查阅其服务条款（ToS）或征得网站所有者的许可。同时，要小心不要通过过载主机的服务器而意外执行拒绝服务攻击（DDoS）。

# 通过 RESTful API 访问 JSON 格式化数据

## 问题

您想通过其 API 从服务中访问以 JSON 格式化的数据。

## 解决方案

在 Node 应用程序中，从 API 获取 JSON 格式化数据的最简单技术是使用 HTTP 请求库。

在以下示例中，我将再次使用`node-fetch`，就像在“获取远程数据”中一样：

```
const fetch = require('node-fetch');

(async () => {
  try {
    const response = await fetch('https://swapi.dev/api/people/1/');
    const json = await response.json();
    console.log(json);
  } catch (error) {
    console.log(error);
  }
})();
```

npm 模块`got`是`node-fetch`的一种流行替代品：

```
const got = require('got');

(async () => {
  try {
    const response = await got('https://swapi.dev/api/people/2/');
    console.log(JSON.parse(response.body));
  } catch (error) {
    console.log(error.response.body);
  }
})();
```

## 讨论

RESTful API 是一种无状态的 API，意味着每个客户端请求都包含服务器响应所需的一切（不涉及请求之间的存储状态）；它显式地使用 HTTP 方法。它支持类似目录结构的 URI，并以特定的方式传输数据（通常是 XML 或 JSON）。HTTP 方法包括：

+   GET: 获取资源数据

+   PUT: 更新资源

+   DELETE: 删除资源

+   POST: 创建资源

因为我们专注于获取数据，所以目前唯一感兴趣的方法是 GET。因为我们专注于 JSON，所以我们使用可以访问 JSON 格式数据并将其转换为我们可以在 JavaScript 应用程序中操作的对象的客户端方法。

让我们看另一个例子。

[Open Exchange Rate](https://openexchangerates.org) 提供了一个 API，我们可以用它来获取当前的汇率，将名称转换为不同类型的货币缩写，以及特定日期的汇率。它有一个[永久免费计划](https://oreil.ly/TjhFo)，可以免费访问有限的 API。

可以对系统进行两次查询（当前货币汇率和名称到缩写），当两个查询都完成时，将缩写作为键获取，并使用这些键在结果中查找长名称和汇率，然后将这些配对打印到控制台：

```
const fetch = require('node-fetch');
require('dotenv').config();

const id = process.env.APP_ID;

(async () => {
  try {
    const moneyAPI1 = await fetch(
      `https://openexchangerates.org/api/latest.json?app_id=${id}`
    );
    const moneyAPI2 = await fetch(
      `http://openexchangerates.org/api/currencies.json?app_id=${id}`
    );

    const latest = await moneyAPI1.json();
    const names = await moneyAPI2.json();
    const keys = Object.keys(latest.rates);

    keys.forEach((value, index) => {
      const rate = latest.rates[keys[index]];
      const name = names[keys[index]];
      console.log(`${name} ${rate}`);
    });
  } catch (error) {
    console.log(error);
  }
})();
```

###### 注意

请注意，`id` 值需要用您在创建账户时 API 提供商分配的唯一 ID 替换。在这个例子中，我使用 `dotenv` 模块从 *.env* 文件中加载存储的值。

基础货币是“USD”或美元，以下是结果的样本：

```
"Malawian Kwacha 394.899498"
"Mexican Peso 13.15711"
"Malaysian Ringgit 3.194393"
"Mozambican Metical 30.3662"
"Namibian Dollar 10.64314"
"Nigerian Naira 162.163699"
"Nicaraguan Córdoba 26.03978"
"Norwegian Krone 6.186976"
"Nepalese Rupee 98.07189"
"New Zealand Dollar 1.185493"
```

在代码片段中，我使用 `async/await` 进行查询，并在两个查询都完成后处理结果。在生产系统中，我们很可能会根据我们的计划允许的时间（免费 API 访问每小时一次）缓存结果。

## 参见

示例中不需要 *转义* 用作 API 请求参数的值，但如果需要转义值，可以使用 Node 的内置 `querystring.escape()` 方法。
