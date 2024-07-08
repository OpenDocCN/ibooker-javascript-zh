# 第十三章：获取远程数据

在浏览器中接收和处理数据的能力，无需刷新页面，是 JavaScript 的超级能力之一。实时数据跟踪器、聊天应用程序、社交媒体动态更新等，都是通过 JavaScript 发出请求到服务器并更新页面内容来实现的。在本章中，我们将介绍如何发出和处理这些请求。

###### 注意

您可能也会听到 “AJAX” 这个术语，它是 Asynchronous JavaScript and XML 的缩写。虽然最初是指检索 XML，但 AJAX 已经成为从 Web 浏览器向远程服务器检索和发送数据的泛化术语。

# 使用 Fetch 请求远程数据

## 问题

您需要从服务器请求远程数据。

## 解决方案

使用 Fetch API，允许您发出请求并处理响应。要发出简单请求，请将 URL 作为 `fetch` 参数传递，该方法将返回一个 promise。以下示例请求 URL，解析 JSON 响应，并将响应记录到控制台：

```
const url = 'https://api.nasa.gov/planetary/apod?api_key=DEMO_KEY';
fetch(url)
  .then(response => response.json())
  .then(data => console.log(data));
```

或者，使用 `fetch` 的 `async/await` 语法：

```
const url = 'https://api.nasa.gov/planetary/apod?api_key=DEMO_KEY';

async function fetchRequest() {
  const response = await fetch(url);
  const data = await response.json();
  console.log(data);
}

fetchRequest();
```

## 讨论

Fetch API 提供了从远程源发送和检索数据的方法。在 Web 浏览器环境中工作时，这意味着可以在不刷新页面的情况下检索数据。作为 Web 用户，您可能经常遇到这些类型的请求。Fetch API 可用于：

+   在社交媒体动态中加载额外的条目

+   表单自动完成建议

+   “喜欢” 一个社交媒体帖子

+   基于先前响应更新表单字段值

+   在不离开页面的情况下提交表单

+   将商品添加到购物车

正如您所想象的那样，列表可以无限延续。

`fetch()` 方法接受两个参数：

`url`（必填）

正在进行请求的 URL

`options`

发出请求时的选项对象

可能的 `options` 包括：

`body`

请求的请求体内容

`cache`

请求的缓存模式 (`default`、`no-store`、`reload`、`no-cache`、`force-cache` 或 `only-if-cached`)

`credentials`

请求的凭证 (`omit`、`same-origin` 或 `include`)

`headers`

请求中包含的头部信息

`integrity`

用于验证资源的子资源完整性值

`keepalive`

将其设置为 `true`，使请求在页面生存期内继续存在

`method`

请求方法 (`GET`、`POST`、`PUT` 或 `DELETE`)

`mode`

请求的模式 (`cors`、`no-cors` 或 `same-origin`)

`redirect`

设置重定向的行为 (`follow`、`error` 或 `manual`)

`referrer`

设置引用者头部的值 (`about:client`、当前 URL 或空字符串)

`referrerPolicy`

指定引用者策略 (`no-referrer`、`no-referrer-when-downgrade`、`same-origin`、`origin`、`strict-origin`、`origin-when-cross-origin`、`strict-origin-when-cross-origin` 或 `unsafe-url`)

`signal`

`AbortController` 对象用于中止请求

如前例所示，仅需 `url` 参数即可。当仅传递 URL 时，`fetch` 方法将执行 `GET` 请求。以下示例演示如何使用选项对象：

```
const response = await fetch(url, {
  method: 'GET',
  mode: 'cors',
  credentials: 'omit',
  redirect: 'follow',
  referrerPolicy: 'no-referrer'
});
```

`fetch` 使用 JavaScript promises。最初的 promise 返回一个 `Response` 对象，其中包含完整的 HTTP 响应，包括主体、头部、状态码、重定向信息、cors 类型和 URL。使用返回的响应，然后可以使用额外的解析方法来解析请求的主体。在示例中，我使用 `json()` 方法将响应体解析为 JSON。以下是可能的解析方法：

`arrayBuffer()`

将响应体解析为 `ArrayBuffer`

`blob()`

将响应体解析为 `Blob`

`json()`

将响应体解析为 JSON

`text()`

将响应体解析为 UTF-8 字符串

`formData()`

将响应体解析为 `FormData()` 对象

在使用 `fetch` 时，可以根据服务器的状态响应处理错误。在 `async/await` 中：

```
async function fetchRequestWithError() {
  const response = await fetch(url);
  if (response.status >= 200 && response.status < 400) {
    const data = await response.json();
    console.log(data);
  } else {
    // Handle server error
    // example: INTERNAL SERVER ERROR: 500 error
    console.log(`${response.statusText}: ${response.status} error`);
  }
}
```

若要进行更健壮的错误处理，可以将整个 `fetch` 请求包装在 `try/catch` 块中，以便处理任何其他错误：

```
async function fetchRequestWithError() {
  try {
    const response = await fetch(url);
    if (response.status >= 200 && response.status < 400) {
      const data = await response.json();
      console.log(data);
    } else {
      // Handle server error
      // example: INTERNAL SERVER ERROR: 500 error
      console.log(`${response.statusText}: ${response.status} error`);
    }
  } catch (error) {
    // Generic error handler
    console.log(error);
  }
}
```

在使用 JavaScript 的 `then` promise 语法时，错误处理方式类似：

```
fetch(url)
  .then((response) => {
    if (response.status >= 200 && response.status < 400) {
      return response.json();
    } else {
      // Handle server error
      // example: INTERNAL SERVER ERROR: 500 error
      console.log(`${response.statusText}: ${response.status} error`);
    }
  })
  .then((data) => {
    console.log(data)
  }).catch(error) => {
    // Generic error handler
    console.log(error);
  };
```

如果您以前使用过 AJAX 请求，可能已经使用过 `XMLHttpRequest`（XHR）方法（在 “使用 XMLHttpRequest” 中介绍）。由于其基于 promise 的语法、简单的语法和广泛的浏览器支持，现在推荐使用 Fetch API 进行这些请求。`fetch` 在所有现代浏览器（Chrome、Edge、Firefox、Safari）中都受支持，但不支持 Internet Explorer。如果您的应用程序需要支持较旧版本的 Internet Explorer，可以选择使用 XHR（`XMLHttpRequest`）或使用 [fetch polyfill](https://github.com/github/fetch) 和 [promise polyfill](https://github.com/taylorhakes/promise-polyfill)。

# 使用 XMLHttpRequest

## 问题

您的应用程序需要请求远程数据，同时支持较旧的浏览器。

## 解决方案

使用 `XMLHttpRequest`（XHR）来替代 `fetch`。以下是一个 XHR `GET`请求示例，与 “使用 Fetch 请求远程数据” 的示例相似：

```
const url = 'https://api.nasa.gov/planetary/apod?api_key=DEMO_KEY';
const request = new XMLHttpRequest();
request.open('GET', url);
request.send();

request.onload = () => {
  if (request.status >= 200 && request.status < 400) {
    // successful request logs the returned JSON data
    const data = JSON.parse(request.response);
    console.log(data);
  } else {
    // server error
    // example: INTERNAL SERVER ERROR: 500 error
    console.log(`${request.statusText}: ${request.status} error`);
  }
};

// request error
request.onerror = () => console.log(request.statusText);
```

## 讨论

`XMLHttpRequest` 是发起远程数据请求的原始语法。尽管名称中含有 XML，但它可以用于请求各种数据。在前面的示例中，我正在请求 JSON 数据。那么 `XMLHttpRequest` 与 `fetch` 有何不同？

+   `fetch` 大量使用 JavaScript promises，而 `XMLHttpRequest` 则基于 `XMLHttpRequest()` 构造函数。

+   `XMLHttpRequest` 在所有浏览器中都受支持，包括较旧版本的 Internet Explorer。在 Internet Explorer 11 或更早的某些版本以及 2017 年或更早的某些版本的现代自动更新浏览器中，`fetch` 将无法正常工作，除非使用基于 `XMLHttpRequest` 的 polyfill。

+   `XMLHttpRequest` 默认将每个请求发送到服务器的 cookie，而 `fetch` 要求明确设置 `credentials` 选项。

+   `XMLHttpRequest` 支持跟踪上传进度，而 `fetch` 目前仅支持下载进度。

+   `fetch` 不支持超时，请求的长度由用户的浏览器决定。

尽管本章的其余部分将使用现代的 `fetch` 语法，但由于其浏览器支持和不同的特性，`XMLHttpRequest` 仍然是一个合理的选择，特别是在处理传统应用程序时。

# 提交表单

## 问题

您希望从客户端提交一个表单。

## 解决方案

使用 `fetch` 使一个 `FormData` 对象的 `POST` 请求：

```
const myForm = document.getElementById('my-form');
const url = 'http://localhost:8080/';

myForm.addEventListener('submit', async event => {
  event.preventDefault();

  const formData = new FormData(myForm);
  const response = await fetch(url, {
    method: 'post',
    body: formData
  });

  const result = await response.text();
  alert(result);
});
```

## 讨论

在示例代码中，我使用 `getElementById` 选择一个 HTML 表单元素，并将 `POST` 表单的 URL 存储为一个变量。在这种情况下，我将表单 POST 到本地开发服务器，如示例 13-1 所示。然后我为表单添加了一个事件监听器，并阻止了默认的表单提交行为，以便我可以使用 `fetch` 执行 JavaScript 的 `POST` 请求。

完整的 HTML 标记和 JavaScript 如下所示：

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Form POST</title>
  </head>
  <body>
    <h1>Form POST HTML</h1>

    <form id="my-form">
      <label for="name">Name:</label>
      <input type="text" id="name" name="name" />

      <label for="mail">E-mail:</label>
      <input type="email" id="mail" name="email" />

      <label for="msg">Message:</label>
      <textarea id="message" name="message"></textarea>

      <button>Submit</button>
    </form>

    <script>
      const myForm = document.getElementById('my-form');
      const url = 'http://localhost:8080/';

      myForm.addEventListener('submit', async event => {
      event.preventDefault();

      const formData = new FormData(myForm);
      const response = await fetch(url, {
      method: 'post',
      body: formData
      });

      const result = await response.text();
      alert(result);
      });

    </script>
  </body>
</html>
```

JavaScript 的 `FormData` 提供了一种简便的方式来创建所有表单数据的键/值对。这适用于基于文本的表单元素，如示例中所示，以及文件上传。首先，使用 `FormData` 构造函数：

```
const myForm = document.getElementById('my-form');
const formData = new FormData(myForm);
```

您还可以使用一些有用的方法来操作 `FormData` 中包含的数据：

`FormData.append(key, value)` 或 `FormData.append(key, blob, filename)`

向表单添加新数据

`FormData.delete(key)`

删除一个字段

`FormData.set(key, value)`

添加新数据，如果存在重复键，则删除

这是您如何向上一个示例中添加一个额外字段的方法：

```
const myForm = document.getElementById('my-form');
const url = 'http://localhost:8080/';

myForm.addEventListener('submit', async event => {
  event.preventDefault();

  const formData = new FormData(myForm);
  // add a new field using FormData.append
  formData.append('user', true);

  const response = await fetch(url, {
    method: 'post',
    body: formData
  });

  const result = await response.text();
  console.log(result);
});
```

现在 `POST` 请求的主体将是：

```
{
  name: 'Adam',
  email: 'adam@example.com',
  message: 'Hello',
  user: 'true'
}
```

也可以使用 `get` 和 `has` 方法处理表单值：

`FormData.get(key)`

获取特定键的值

`FormData.has(key)`

检查给定键的值并返回布尔值

虽然 `FormData` 非常有用，但它并不是 `POST` 请求体的唯一值类型。以下类型可以通过 `POST` 请求发送：

+   一个字符串

+   一个编码的字符串，例如 JSON 或 XML

+   一个 `URLSearchParams` 对象

+   二进制数据的 `Blob` 或 `BufferSource`

在 “从服务器填充选择列表” 中，我将演示如何使用 `fetch` 发送 JSON 的 `POST` 请求。

最后，示例 13-1 是一个 Node.js Express 服务器的示例，用于处理请求：

##### 示例 13-1\. Express 表单服务器示例

```
const express = require('express');
const formidable = require('formidable');
const cors = require('cors');

const app = express();
const port = 8080;

app.use(cors());

app.get('/', (req, res) =>
  res.send('Example server for receiving JS POST requests')
);

app.post('/', (req, res) => {
  const form = formidable();

  form.parse(req, (err, fields) => {
    if (err) {
      return;
    }
    console.log('POST body:', fields);
    res.sendStatus(200);
  });
});

app.listen(port, () =>
  console.log(`Example app listening at http://localhost:${port}`)
);
```

###### 注意

我们在 第二十一章 中详细介绍了 Express。

# 从服务器填充选择列表

## 问题

基于用户对另一个表单元素的操作，您希望用值填充一个选择列表。

## 解决方案

捕获表单元素的 `change` 事件：

```
const niceThings = document.getElementById('nice-thing');
niceThings.addEventListener('change', async () => {
  // GET request and events go here
});
```

在事件处理函数中，以 JSON 格式作为表单数据进行 `POST` 请求：

```
const niceThings = document.getElementById('nice-thing');
const url = 'http://localhost:8080/select';

// perform GET request when select value changes
niceThings.addEventListener('change', async () => {
  // object containing select value
  const selection = {
    niceThing: niceThings.value
  };

  // GET request to server
  const response = await fetch(url, {
    method: 'post',
    headers: {
      'Content-Type': 'application/json;charset=utf-8'
    },
    body: JSON.stringify(selection)
  });

});
```

用结果填充选择列表：

```
const select = document.getElementById('nicestuff');

if (response.ok) {
  const result = await response.json();
  // empty the select element
  select.length = 0;
  // add a default display option with text and no value
  select.options[0] = new Option('--Please choose an option--', '');
  // populate the select with the returned values
  for (let i = 0; i < result.length; i += 1) {
    select.options[select.length] = new Option(result[i], result[i]);
  }
  // display the select element
  select.style.display = 'block';
} else {
  // if there's a problem fetching the data, display an error
  alert('Error');
  }
```

## 讨论

基于用户的选择填充 `select` 或其他表单元素是常见的用户界面交互。您可以捕获用户在另一个表单元素中的选择，根据该值查询服务器应用程序，并基于该值构建其他表单元素，而无需离开页面，而不是填充带有许多选项的 `select` 元素，或者构建一组 10 或 20 个单选按钮。

示例 13-2 展示了一个简单的页面，捕获了选择元素的变化事件，使用所选值进行 fetch 请求，并通过解析返回的数据来填充新的选择列表。在示例中，数据作为数组返回，并且新选项的创建使用返回的文本，文本同时作为选项标签和选项值。在填充 `select` 元素之前，将其长度设置为 0\. 这是截断 `select` 元素的快速简便方法——删除所有现有选项并重新开始。

##### 示例 13-2\. 创建按需 `select` 列表

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Select List</title>
    <style>
      #nicestuff {
        display: none;
        margin: 10px 0;
      }

      label,
      legend {
        display: block;
        font-size: 1.6rem;
        font-weight: 700;
        margin-bottom: 0.5rem;
      }
    </style>
  </head>
  <body>
    <h1>Select List</h1>

    <form id="my-form">
      <label for="pet-select">Select a nice thing:</label>

      <select name="nicething" id="nice-thing">
        <option value="">--Please choose an option--</option>
        <option value="birds">Birds</option>
        <option value="flowers">Flowers</option>
        <option value="sweets">Sweets</option>
        <option value="critters">Cute Critters</option>
      </select>
      <select id="nicestuff">
        <option value="">--Please choose an option--</option>
      </select>
    </form>
    <script>
    const niceThings = document.getElementById('nice-thing');
    const select = document.getElementById('nicestuff');
    const url = 'http://localhost:8080/select';

    // perform GET request when select value changes
    niceThings.addEventListener('change', async () => {
    // object containing select value
    const selection = {
      niceThing: niceThings.value
    };

    // GET request to server
    const response = await fetch(url, {
      method: 'post',
      headers: {
        'Content-Type': 'application/json;charset=utf-8'
      },
      body: JSON.stringify(selection)
    });

    // if fetch is successful
    if (response.ok) {
      const result = await response.json();
      // empty the select element
      select.length = 0;
      // add a default display option with text and no value
      select.options[0] = new Option('--Please choose an option--', '');
      // populate the select with the returned values
      for (let i = 0; i < result.length; i += 1) {
        select.options[select.length] = new Option(result[i], result[i]);
      }
      // display the select element
      select.style.display = 'block';
    } else {
      // if there's a problem fetching the data, display an error
      alert('Error');
    }
    });

    </script>
  </body>
</html>
```

该示例使用 Node 应用程序来填充选择列表，但可以使用任何服务器端编程语言编写。有关详细信息，请参阅第 III 部分中的 Node 章节。

```
const express = require('express');
const formidable = require('formidable');
const cors = require('cors');

const app = express();
const port = 8080;

app.use(cors());

app.get('/', (req, res) =>
  res.send('Example server for receiving JS POST requests')
);

app.post('/select', (req, res) => {
  const form = formidable();

  form.parse(req, (err, fields) => {
    if (err) {
      return;
    }
    if (fields.niceThing === 'critters') {
      res.send(['puppies', 'kittens', 'guinea pigs']);
    } else if (fields.niceThing === 'sweets') {
      res.send(['licorice', 'cake', 'cookies', 'custard']);
    } else if (fields.niceThing === 'birds') {
      res.send(['robin', 'mockingbird', 'finch', 'dove']);
    } else if (fields.niceThing === 'flowers') {
      res.send(['roses', 'lilys', 'daffodils', 'pansies']);
    } else {
      res.send(['No Nice Things Found']);
    }
  });
});

app.listen(port, () =>
  console.log(`Example app listening at http://localhost:${port}`)
);
```

逐步构建表单元素并非所有应用都必要，但对于数据可能变化或表单复杂的情况下，这是确保更有效的表单的一个很好的方法。

# 解析返回的 JSON

## 问题

您希望安全地从 JSON 创建 JavaScript 对象。您还希望将 true 和 false 的数值表示（分别为 1 和 0）替换为它们的布尔对应值（`true` 和 `false`）。

## 解决方案

使用 `JSON.parse` 功能解析对象。要将数值转换为布尔对应值，创建一个*复苏*函数：

```
const jsonobj = '{"test" : "value1", "test2" : 3.44, "test3" : 0}';
const obj = JSON.parse(jsonobj, (key, value) => {
  if (typeof value === 'number') {
    if (value === 0) {
      value = false;
    } else if (value === 1) {
      value = true;
    }
  }
  return value;
});

console.log(obj.test3); // false
```

## 讨论

要想知道如何创建 JSON，请考虑如何创建对象字面量，并将其转换为字符串（有一些注意事项）。

如果对象是一个数组：

```
const arr = new Array("one","two","three");
```

JSON 表示法等同于数组的字面表示法：

```
["one","two","three"];
```

注意使用双引号 (`""`) 而不是单引号，因为 JSON 不允许使用单引号。

如果您正在处理一个对象：

```
const obj3 = {
   prop1 : "test",
   result : true,
   num : 5.44,
   name : "Joe",
   cts : [45,62,13]
 };
```

JSON 表示法为：

```
{"prop1":"test","result":true,"num":5.44,"name":"Joe","cts":[45,62,13]}
```

注意在 JSON 中，属性名用引号括起来，但只有当值是字符串时才用引号括起来。此外，如果对象包含其他对象（如数组），它也会被转换为其 JSON 等效形式。然而，对象*不能*包含方法。如果有方法，则会抛出错误。JSON 只能处理数据。

JSON 静态对象并不复杂，因为它只提供两个方法：`stringify()` 和 `parse()`。`parse()` 方法接受两个参数：一个 JSON 格式的字符串和一个可选的 `reviver` 函数。该函数以键/值对作为参数，并返回原始值或修改后的结果。

在解决方案中，JSON 格式的字符串是一个包含三个属性的对象：一个字符串，一个数值，以及第三个属性，其数值为布尔型表示—0 为假，1 为真。

为了将所有的 0 和 1 值转换为`false`和`true`，第二个参数作为`JSON.parse()`的函数提供。它检查对象的每个属性，看它是否为数值。如果是，函数再检查值是否为 0 或 1。如果值为 0，则返回值设为`false`；如果为 1，则返回值设为`true`；否则，返回原始值。

能够转换传入的 JSON 格式数据非常重要，特别是在处理 AJAX 请求或 JSONP 响应结果时。您并不总能控制从服务获取的数据结构。

###### 注意

JSON 有一些限制：字符串必须用双引号括起来，不能包含十六进制值，也不能在字符串中包含制表符。

# 获取和解析 XML

## 问题

您需要获取远程 XML 文件并解析其内容。

## 解决方案

使用`fetch`以及提供从字符串解析 XML 的`DomParser`API 的能力。

首先，您需要使用`fetch`请求 XML 文件。在此示例中，我请求的是*纽约时报*主页的 XML 源：

```
const url = 'https://rss.nytimes.com/services/xml/rss/nyt/HomePage.xml';

async function fetchAndParse() {
  const response = await fetch(url);
  const data = await response.text();
  console.log(data);
}

fetchAndParse();
```

接下来，使用`DOMParser`解析返回的 XML 字符串，然后使用 DOM 方法查询文档中的数据：

```
const url = 'https://rss.nytimes.com/services/xml/rss/nyt/HomePage.xml';

async function fetchAndParse() {
  const response = await fetch(url);
  const data = await response.text();
  const parser = new DOMParser();
  const XMLDocument = parser.parseFromString(data, 'text/xml');
  console.log(XMLDocument);
}

fetchAndParse();
```

## 讨论

使用`fetch`获取 XML 时，文档作为纯文本返回。然后您可以使用`DOMParser`API 启用 DOM 方法来查询文档并处理结果。

`DOMParser`使您能够使用像`getElementsByTagName`这样的 DOM 查询方法与 XML 内容交互。`DOMParser`需要两个参数。第一个参数是要解析的字符串。第二个参数是`mimeType`，指定文档类型。`mimeType`的选项包括：

+   `text/html`

+   `text/xml`

+   `application/xml`

+   `applicatiom/xhtml+html`

+   `image/svg+xml`

下面的例子扩展了 XML 解析器以使用 DOM 查询选择器将最新文章的名称输出到网页：

```
(async () => {
  const url = 'https://rss.nytimes.com/services/xml/rss/nyt/HomePage.xml';

  // fetch and parse the XML document
  async function fetchAndParse() {
    const response = await fetch(url);
    const data = await response.text();
    const parser = new DOMParser();
    const XMLDocument = parser.parseFromString(data, 'text/xml');
    return XMLDocument;
  }

  function displayTitles(xml) {
    // HTML element where the results will be displayed
    // the markup contains a ul with an id of "results"
    const listElem = document.getElementById('results');
    // get the article titles
    // each is wrapped in a <title> tag within an <item> tag
    const titles = xml.querySelectorAll('item title');
    // loop over each title in the XML; append its text content to the HTML list
    titles.forEach(title => {
      const listItem = document.createElement('li');
      listItem.innerText = title.textContent;
      listElem.appendChild(listItem);
    });
  }

  const xml = await fetchAndParse();
  displayTitles(xml);
})();
```

# 发送二进制数据并加载为图像

## 问题

您想要作为二进制数据请求服务器端的图像。

## 解决方案

通过`fetch`请求获取二进制数据只需将响应类型设置为*`blob`*，然后在返回时操作数据。在解决方案中，数据随后被转换并加载到`img`元素中：

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Binary Data</title>
  </head>
  <body>
    <h1>Binary Data</h1>

    <img id="result" />
    <script>
      async function fetchImage() {
      const url = 'logo.png';
      const response = await fetch(url);
      const blob = await response.blob();

      // add returned url to image element
      const img = document.getElementById('result');
      img.src = URL.createObjectURL(blob);
      }

      fetchImage();
    </script>
  </body>
</html>
```

## 讨论

CORS 规范的一个好处是支持 fetch 请求中的二进制数据（也称为*类型化数组*）。二进制请求的关键要求是将响应类型设置为以下之一：

`arraybuffer`

固定长度的原始二进制数据缓冲区

`blob`

类似文件的不可变原始数据

在解决方案中，我使用了`URL.createObjectURL()`方法将`blob`转换为 DOMString（通常映射为 JavaScript 字符串），并将该 URL 赋给`img`元素的`src`属性。

当然，在首次为`src`属性分配 PNG 文件的 URL 时也很简单。但是，使用诸如 Web Workers 和 WebGL 等各种技术必须能够操作二进制数据。

# 在不同域之间共享 HTTP Cookies

## 问题

您希望作为*带凭证*请求从另一个域访问资源，包括 HTTP cookies 和任何身份验证信息。

## 解决方案

必须在客户端和服务器应用程序中进行更改以支持带凭证的请求。在以下示例中，客户端应用程序位于*somedomain.com*，而服务器位于*api.example.com*。由于这些是不同的域，默认情况下客户端到服务器的带凭证请求将不会共享。

在客户端，必须测试`fetch`请求上的`credentials`属性：

```
fetch('https://api.example.com', {
  credentials: "include"
})
```

在服务器中，`Access-Control-Allow-Controls`头的值必须设置为`true`：

```
const http = require('http');
const Cookies = require('cookies');

const server = http.createServer((req,res) => {
  // Set CORS headers
  res.setHeader('Content-type', 'text/plain');
  res.setHeader('Access-Control-Allow-Origin', 'https://somedomain.com');
  res.setHeader('Access-Control-Allow-Credentials', true);

  const cookies = new Cookies (req, res);
  cookies.set("apple","red");

  res.writeHead(200);
  res.end("Hello cross-domain");

});

server.listen(8080);
```

###### 注意

在使用 Express 时，我建议使用[CORS 中间件](https://oreil.ly/vNPPC)。我们在第二十一章中详细讨论了 Express。

## 讨论

跨域资源共享（Cross-Origin Resource Sharing，CORS）是指在不同域之间共享信息，如 cookies 和凭据头信息，由于安全原因，浏览器对跨域共享信息进行限制。通过配置 CORS 扩展，可以在不同域之间发送 HTTP cookies 或身份验证头信息，只要客户端和服务器都信号同意。

如果在客户端使用`XMLHttpRequest`替代`fetch`，请设置`withCredentials`属性：

```
const request = new XMLHttpRequest();

request.onreadystatechange = function() {
    if (this.readyState == 4) {
        console.log(this.status);
        if (this.status == 200) {
            document.getElementById('result').innerHTML = this.responseText;
        }
    }
};
request.open('GET','http://localhost:8080/');
request.withCredentials = true;
request.send(null);
```

# 使用 Websockets 建立客户端和服务器之间的双向通信

## 问题

您希望在服务器和网页客户端之间建立双向实时通信。

## 解决方案

WebSockets 允许您支持客户端和服务器之间的双向通信。客户端创建一个新的 WebSockets 对象，传入 WebSockets 服务器的 URI。请注意，使用`ws:`协议代替`http`或`https`。当客户端收到消息时，它将消息文本转换为对象，检索计数器，增加它，然后在对象的字符串成员中使用它。

在以下示例中，客户端打印出每隔一个数字，从 2 开始。通过在消息中传递要打印的字符串，客户端和服务器之间保持状态：

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Using Websockets</title>
  </head>
  <body>
    <h1>Using Websockets</h1>

    <div id="output"></div>
    <script type="text/javascript">
      const socket = new WebSocket('ws://localhost:8080');
      socket.onmessage = event => {
        const msg = JSON.parse(event.data);
        msg.counter = Number(msg.counter) + 1;
        msg.strng += `${msg.counter}-`;
        const html = `<p> ${msg.strng} </p>`;
        document.getElementById('output').innerHTML = html;
        socket.send(JSON.stringify(msg));
      };
    </script>
  </body>
</html>
```

对于服务器，我正在使用`ws` Node 模块。一旦创建服务器，它通过发送具有两个成员的 JavaScript 对象与客户端开始通信：数字计数器和字符串。必须首先将对象转换为字符串。代码监听传入消息和`close`事件。当接收到传入消息时，它增加计数器并发送对象：

```
var wsServer = require('ws').Server;
var wss = new wsServer({port:8001});
wss.on('connection', (function (conn) {

    // object being passed back and forth between
    // client and server
    var counter = {counter: 1, strng: ''};

    // send first communication to client
    conn.send(JSON.stringify(counter));

    // on response back
    conn.on('message', function(message) {
        var ct = JSON.parse(message);
        ct.counter = parseInt(ct.counter) + 1;
        if (ct.counter < 100) {
           conn.send(JSON.stringify(ct));
        }
    });
}));
```

## 讨论

双向通信，也称为*全双工*通信，是可以同时进行的双向通信。将其视为一条双向道路，交通双向流动。所有现代浏览器都支持 WebSockets 规范，正如您所看到的，它非常容易使用。

WebSockets 的优势不仅仅在于在浏览器中使用非常简单，而且它能够穿越代理和防火墙，这是其他双向通信技术（如长轮询）无法轻松实现甚至不可能的。为了确保应用程序的安全性，诸如 Chrome 和 Firefox 之类的用户代理禁止混合内容（即同时使用 HTTP 和 HTTPS）。

WebSockets 支持二进制数据以及文本。正如示例所示，可以通过在发送之前调用`JSON.stringify()`将 JSON 对象传输，并在接收端对字符串调用`JSON.parse()`。

## 参见

参见有关[WebSockets](https://www.websocket.org)的更多信息。

# 长轮询远程数据源

## 问题

您希望与服务器保持连接，以便客户端能够立即更新新信息，但服务器不使用 WebSockets。

## 解决方案

使用长轮询，这是一种技术，客户端通过使用异步的`fetch`函数来维持与服务器的连接，并在响应后再次调用自身。在其最基本的形式下，客户端端的长轮询看起来像这样：

```
const url = 'http://localhost:8080/';

async function longPoll() {
  const response = await fetch(url);
  // if message received, log response to console and call polling function
  const message = await response.text();
  console.log(message);
  await longPoll();
}

longPoll();
```

通过添加一些错误处理，当接收到错误时将等待一定的时间，然后尝试轮询服务器可以改善这一情况：

```
const url = 'http://localhost:8080/';

async function longPoll() {
  try {
    // if message received, log response to console and call polling function
    const response = await fetch(url);
    const message = await response.text();
    console.log(message);
    await longPoll();
  } catch (error) {
    // if fetch returns an error, wait 1 second and try again
    console.log(`Request failed ${error}`);
    await new Promise(resolve => setTimeout(resolve, 1000));
    await longPoll();
  }
}

longPoll();
```

## 讨论

长轮询服务器涉及向服务器发出请求并保持连接，直到响应被发送。一旦客户端收到响应，它立即重新连接到服务器并等待新的响应。这个过程可以分解如下：

1.  客户端向服务器发送请求。

1.  客户端保持与服务器的连接，同时等待响应。

1.  服务器将响应发送给客户端。

1.  客户端重新连接到服务器，进程重复。

我发现一个聊天程序是思考长轮询的一个有帮助的方式。想象一个聊天程序，其中有两个用户在彼此聊天，Riley 和 Harlow。他们每个人都连接到服务器。当 Riley 发送消息时，服务器将向 Harlow 的浏览器发送响应，后者立即重新连接并等待下一条消息。

长轮询的局限性在于服务器能维持的开放连接数量。Node 被设计用来处理许多并发连接，而某些语言有其限制。所有语言都受限于服务器硬件本身。尽管长轮询是一种简单且有效地维持连接的方法，WebSockets（如 “使用 Websockets 建立客户端和服务器之间的双向通信” 中所述）是客户端和服务器之间更高效的双向通信手段。
