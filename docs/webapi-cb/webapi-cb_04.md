# 第四章：网络请求

# 介绍

如今，几乎找不到不发送任何网络请求的 Web 应用程序。自 Web 2.0 以来，通过 Ajax（异步 JavaScript 和 XML）的新方法，Web 应用程序一直在发送异步请求以获取新数据，而无需重新加载整个页面。XMLHttpRequest API 开启了交互式 JavaScript 应用程序的新时代。尽管名称如此，XMLHttpRequest（有时称为 XHR）也可以处理 JSON 和表单数据负载。

XMLHttpRequest 改变了游戏规则，但其 API 使用起来可能很痛苦。最终，第三方库如 Axios 和 jQuery 添加了更简化的 API 来包装核心 XHR API。

2015 年，基于`Promise`的新 API Fetch 成为新标准，并逐渐获得浏览器支持。今天，Fetch 是从 Web 应用程序发起异步请求的标准方式。

本章探讨了 XHR 和 Fetch 以及一些其他用于网络通信的 API：

Beacons

一个简单的单向 POST 请求，非常适合发送分析数据。

服务器发送事件

用于接收实时事件的单向持久连接服务器

WebSockets

用于双向持久连接的服务器通信

# 使用 XMLHttpRequest 发送请求

## 问题

您想向公共 API 发送 GET 请求，并支持不实现 Fetch API 的旧版浏览器。

## 解决方案

使用 XMLHttpRequest API。XMLHttpRequest 是用于发起网络请求的异步、事件驱动 API。XMLHttpRequest 的一般用法是这样的：

1.  创建一个新的`XMLHttpRequest`对象。

1.  为`load`事件添加监听器，接收响应数据。

1.  在请求上调用`open`，传递 HTTP 方法和 URL。

1.  最后，在请求上调用`send`。这会触发发送 HTTP 请求。

示例 4-1 展示了如何使用 XHR 处理 JSON 数据的简单示例。

##### 示例 4-1\. 使用 XMLHttpRequest 进行 GET 请求

```
/**
 * Loads user data from the URL /api/users, then prints them
 * to the console
 */
function getUsers() {
  const request = new XMLHttpRequest();

  request.addEventListener('load', event => {
    // The event target is the XHR itself; it contains a
    // responseText property that we can use to create a JavaScript object from
    // the JSON text.
    const users = JSON.parse(event.target.responseText);
    console.log('Got users:', users);
  });

  // Handle any potential errors with the request.
  // This only handles network errors. If the request
  // returns an error status like 404, the 'load' event still fires
  // where you can inspect the status code.
  request.addEventListener('error', err => {
    console.log('Error!', err);
  });

  request.open('GET', '/api/users');
  request.send();
}
```

## 讨论

XMLHttpRequest API 是一个事件驱动的 API。当接收到响应时，会触发`load`事件。在示例 4-1 中，`load`事件处理程序将原始响应文本传递给`JSON.parse`。它期望响应体是 JSON，并使用`JSON.parse`将 JSON 字符串转换为对象。

如果在加载数据时发生错误，将触发`error`事件。这处理连接或网络错误，但被视为“错误”的 HTTP 状态码（如 404 或 500）不会触发此事件。而是也触发`load`事件。

为了防止此类错误，您需要检查响应的`status`属性，以确定是否存在此类错误情况。可以通过引用`event.target.status`来访问它。

Fetch 已经得到长时间的支持，所以除非必须支持非常老旧的浏览器，否则大多数时候您将使用 Fetch API。

# 使用 Fetch API 发送 GET 请求

## 问题

你想要使用现代浏览器向公共 API 发送 GET 请求。

## 解决方案

使用 Fetch API。Fetch 是一个使用 `Promise` 的较新请求 API。它非常灵活，可以发送各种数据，但是 Example 4-2 发送一个基本的 GET 请求到 API。

##### Example 4-2\. 使用 Fetch API 发送 GET 请求

```
/**
 * Loads users by calling the /api/users API, and parses the
 * response JSON.
 * @returns a Promise that resolves to an array of users returned by the API
 */
function loadUsers() {
  // Make the request.
  return fetch('/api/users')
    // Parse the response body to an object.
    .then(response => response.json())
    // Handle errors, including network and JSON parsing errors.
    .catch(error => console.error('Unable to fetch:', error.message));
}

loadUsers().then(users => {
  console.log('Got users:', users);
});
```

## 讨论

Fetch API 更加简洁。它返回一个`Promise`，解析为表示 HTTP 响应的对象。`response` 对象包含诸如状态码、头部和正文等数据。

要获取 JSON 响应正文，你需要调用响应的 `json` 方法。该方法从流中读取正文，并返回解析为对象的 JSON 正文的 `Promise`。如果响应正文不是有效的 JSON，则 `Promise` 将被拒绝。

响应还有其他方法以其他格式读取正文，例如 `FormData` 或纯文本字符串。

因为 Fetch 使用 `Promise`，你也可以使用 `await`，如 Example 4-3 所示。

##### Example 4-3\. 使用 Fetch 和 `async`/`await`

```
async function loadUsers() {
  try {
    const response = await fetch('/api/users');
    return response.json();
  } catch (error) {
    console.error('Error loading users:', error);
  }
}

async function printUsers() {
  const users = await loadUsers();
  console.log('Got users:', users);
}
```

###### 注意

记住，在函数中使用 `await` 之前，该函数必须带有 `async` 关键字。

# 使用 Fetch API 发送 POST 请求

## 问题

你想要向期望 JSON 请求正文的 API 发送 POST 请求。

## 解决方案

使用 Fetch API，指定方法（POST）、JSON 正文和内容类型（参见 Example 4-4）。

##### Example 4-4\. 使用 Fetch API 发送 JSON 负载的 POST 请求

```
/**
 * Creates a new user by sending a POST request to /api/users.
 * @param firstName The user's first name
 * @param lastName The user's last name
 * @param department The user's department
 * @returns a Promise that resolves to the API response body
 */
function createUser(firstName, lastName, department) {
  return fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify({ firstName, lastName, department }),
    headers: {
      'Content-Type': 'application/json'
    }
  })
    .then(response => response.json());
}

createUser('John', 'Doe', 'Engineering')
  .then(() => console.log('Created user!'))
  .catch(error => console.error('Error creating user:', error));
```

## 讨论

Example 4-4 发送一些 JSON 数据的 `POST` 请求。调用 `JSON.stringify` 将用户对象转换为 JSON *字符串*，这是将其作为正文与 `fetch` 一起发送所需的。你还需要设置 `Content-Type` 头部，以便服务器知道如何解释正文。

Fetch 还允许你发送其他内容类型作为正文。Example 4-5 展示了如何使用表单数据发送 `POST` 请求。

##### Example 4-5\. 发送表单数据的 POST 请求

```
fetch('/login', {
  method: 'POST',
  body: 'username=sysadmin&password=password',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded;charset=UTF-8'
  }
})
  .then(response => response.json())
  .then(data => console.log('Logged in!', data))
  .catch(error => console.error('Request failed:', error));
```

# 使用 Fetch API 上传文件

## 问题

你想要使用 Fetch API 发送文件数据的 POST 请求。

## 解决方案

使用 `<input type="file">` 元素，并将文件内容作为请求正文发送（参见 Example 4-6）。

##### Example 4-6\. 使用 Fetch API 发送文件数据

```
/**
 * Given a form with a 'file' input, sends a POST request containing
 * the file data in its body.
 * @param form the form object (should have a file input with the name 'file')
 * @returns a Promise that resolves when the response JSON is received
 */
function uploadFile(form) {
  const formData = new FormData(form);
  const fileData = formData.get('file');
  return fetch('https://httpbin.org/post', {
    method: 'POST',
    body: fileData
  })
    .then(response => response.json());
}
```

## 讨论

使用现代浏览器 API 上传文件并不复杂。`<input type="file">` 通过 FormData API 提供文件数据，并包含在 POST 请求的正文中。浏览器会处理剩余部分。

# 发送信标

## 问题

你想要发送一个快速请求，而不必等待响应，例如发送分析数据。

## 解决方案

使用 Beacon API 以 POST 请求发送数据。使用 Fetch API 的常规 POST 请求可能在页面卸载之前无法完成。使用 beacon 更有可能成功（参见示例 4-7）。浏览器不等待响应，并且在用户离开您的站点时发送请求更有可能成功。

##### 示例 4-7\. 发送 beacon

```
const currentUser = {
  username: 'sysadmin'
};

// Some analytics data we want to capture
const data = {
  user: currentUser.username,
  lastVisited: new Date()
};

// Send the data before unload.
document.addEventListener('visibilitychange', () => {
  // If the visibility state is 'hidden', that means the page just became hidden.
  if (document.visibilityState === 'hidden') {
    navigator.sendBeacon('/api/analytics', data);
  }
});
```

## 讨论

使用 XMLHttpRequest 或 fetch 调用时，浏览器等待响应并将其返回（带有事件或 Promise）。通常，您不需要等待响应来处理单向请求，例如发送分析数据。

而不是`Promise`，`navigator.sendBeacon`返回一个布尔值，指示是否已安排发送操作。没有进一步的事件或通知。

`navigator.sendBeacon`总是发送一个`POST`请求。如果您想发送多组分析数据，例如用户与页面交互的集合，请在用户与页面交互时将它们收集到数组中，然后将数组作为`POST`主体与 beacon 一起发送。

# 使用 Server-Sent Events 监听远程事件

## 问题

您希望从后端服务器接收通知，而无需重复轮询。

## 解决方案

使用 EventSource API 接收服务器推送的事件（SSE）。

要开始监听 SSE，请创建`EventSource`的新实例，并将 URL 作为第一个参数传递（参见示例 4-8）。

##### 示例 4-8\. 打开 SSE 连接

```
const events = new EventSource('https://example.com/events');

// Fired once connected
events.addEventListener('open', () => {
  console.log('Connection is open');
});

// Fired if a connection error occurs
events.addEventListener('error', event => {
  console.log('An error occurred:', event);
});

// Fired when receiving an event with a type of 'heartbeat'
events.addEventListener('heartbeat', event => {
  console.log('got heartbeat:', event.data);
});

// Fired when receiving an event with a type of 'notice'
events.addEventListener('notice', event => {
  console.log('got notice:', event.data);
})

// The EventSource leaves the connection open. If we want to close the connection,
// we need to call close on the EventSource object.
function cleanup() {
  events.close();
}
```

## 讨论

`EventSource`必须连接到一个特殊的 HTTP 端点，该端点使用`text/event-stream`的`Content-Type`头保持连接开放。每当事件发生时，服务器可以通过打开的连接发送新消息。

###### 注意

根据[MDN](https://oreil.ly/MliFN)的指出，强烈建议使用 HTTP/2 与 SSE。否则，浏览器对每个域名的`EventSource`连接数量施加严格限制。在这种情况下，最多只能有六个连接。

此限制不是每个标签页的限制；它是在给定域中所有标签页上强加的。

当`EventSource`通过持久连接接收事件时，它是纯文本。您可以从接收到的事件对象的`data`属性访问事件文本。以下是`notice`类型事件的示例：

```
event: notice
data: Connection established at 10:51 PM, 2023-04-22
id: 3
```

要监听此事件，请在`EventSource`对象上调用`addEventListener('notice')`。事件对象有一个`data`属性，其值是事件中以`data:`前缀开头的任意字符串值。

如果事件没有事件类型，您可以监听通用的`message`事件来接收它。

# 使用 WebSocket 实时交换数据

## 问题

您希望实时发送和接收数据，而无需反复使用 Fetch 请求轮询服务器。

## 解决方案

使用 WebSocket API 打开与后端服务器的持久连接（参见示例 4-9）。

##### 示例 4-9\. 创建 WebSocket 连接

```
// Open the WebSocket connection (the URL scheme should be ws: or wss:).
const socket = new WebSocket(url);

socket.addEventListener('open', onSocketOpened);
socket.addEventListener('message', handleMessage);
socket.addEventListener('error', handleError);
socket.addEventListener('close', onSocketClosed);

function onSocketOpened() {
  console.log('Socket ready for messages');
}

function handleMessage(event) {
  console.log('Received message:', event.data);
}

function handleError(event) {
  console.log('Socket error:', event);
}

function onSocketClosed() {
  console.log('Connection was closed');
}
```

###### 注意

要使用 WebSocket，您的服务器必须具有可以连接的 WebSocket 启用的端点。MDN 有一个关于创建 WebSocket 服务器的[深入分析](https://oreil.ly/fzX67)。

一旦套接字触发 `open` 事件，您可以开始发送消息，如示例 4-10 所示。

##### 示例 4-10\. 发送 WebSocket 消息

```
// Messages are simple strings.
socket.send('Hello');

// The socket needs the data as a string, so you can use
// JSON.stringify to serialize objects to be sent.
socket.send(JSON.stringify({
  username: 'sysadmin',
  password: 'password'
}));
```

WebSocket 连接是双向连接。从服务器接收的数据触发 `message` 事件。您可以根据需要处理这些数据，甚至发送响应（参见示例 4-11）。

##### 示例 4-11\. 响应 WebSocket 消息

```
socket.addEventListener('message', event => {
  socket.send('ACKNOWLEDGED');
});
```

最后，完成时，您可以通过在 WebSocket 对象上调用 `close` 来关闭连接。

## 讨论

WebSocket 很适合需要实时能力的应用程序，例如聊天系统或事件监控。WebSocket 端点具有 `ws://` 或 `wss://` 方案。这与 `http://` 和 `https://` 类似 —— 一个不安全，一个使用加密。

要启动 WebSocket 连接，浏览器首先向 WebSocket 端点发送 `GET` 请求。URL `wss://example.com/websocket` 的请求有效负载如下所示：

```
GET /websocket HTTP/1.1
Host: example.com
Sec-WebSocket-Key: aSBjYW4gaGFzIHdzIHBsej8/
Sec-WebSocket-Version: 13
Connection: Upgrade
Upgrade: websocket
```

这启动了 WebSocket 握手。如果成功，服务器将以状态码 101（协议切换）响应：

```
HTTP/1.1 101 Switching Protocols
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Accept: bm8gcGVla2luZywgcGxlYXNlIQ==
```

WebSocket 协议指定了一种算法，根据请求的 `Sec-WebSocket-Key` 生成 `Sec-Websocket-Accept` 头。客户端验证此值后，双向 WebSocket 连接激活，套接字触发 `open` 事件。

连接打开后，您可以通过在套接字对象上调用 `send` 来监听消息并发送消息。稍后，您可以通过在套接字对象上调用 `close` 来终止 WebSocket 会话。
