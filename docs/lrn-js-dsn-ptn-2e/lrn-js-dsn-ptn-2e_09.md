# 第九章：异步编程模式

异步 JavaScript 编程允许您在后台执行长时间运行的任务，同时允许浏览器响应事件并运行其他代码来处理这些事件。在 JavaScript 中，异步编程相对较新，当本书第一版发布时，并没有支持它的语法。

JavaScript 的概念，如`promise`、`async`和`await`，使您的代码更整洁，易于阅读，而不会阻塞主线程。`async`函数作为 ES7 的一部分在 2016 年被引入，并且现在所有浏览器都支持。让我们看一些使用这些特性来结构应用流程的模式。

# 异步编程

在 JavaScript 中，同步代码以阻塞方式执行，这意味着代码按顺序一个语句一个语句地执行。当前语句的执行完成后，才能运行下面的代码。调用同步函数时，该函数内部的代码将从头到尾执行，然后控制权返回给调用者。

另一方面，异步代码以非阻塞方式执行，这意味着 JavaScript 引擎可以在当前运行的代码等待时切换到后台执行此代码。调用异步函数时，函数内部的代码将在后台执行，控制立即返回给调用者。

下面是 JavaScript 中同步代码的一个示例：

```
function synchronousFunction() {
  // do something
}

synchronousFunction();
// the code inside the function is executed before this line
```

下面是 JavaScript 中异步代码的一个示例：

```
function asynchronousFunction() {
  // do something
}

asynchronousFunction();
// the code inside the function is executed in the background
// while control returns to this line
```

您通常可以使用异步代码来执行长时间运行的操作，而不会阻塞您的其他代码。当进行网络请求、读写数据库或执行任何其他类型的输入/输出（I/O）操作时，异步代码非常合适。

使用`async`、`await`和`promise`等语言特性可以更轻松地在 JavaScript 中编写异步代码。它们允许您以看起来和行为类似于同步代码的方式编写异步代码，使其更易于阅读和理解。

让我们简要地看一下回调、promise 和`async`/`await`之间的区别，然后深入探讨每一种：

```
// using callbacks
function makeRequest(url, callback) {
  fetch(url)
    .then(response => response.json())
    .then(data => callback(null, data))
    .catch(error => callback(error));
}

makeRequest('http://example.com/', (error, data) => {
  if (error) {
    console.error(error);
  } else {
    console.log(data);
  }
});
```

在第一个示例中，`makeRequest`函数使用回调来返回网络请求的结果。调用者向`makeRequest`传递一个`callback`函数，该函数在结果（`data`）或错误时回调：

```
// using promises
function makeRequest(url) {
  return new Promise((resolve, reject) => {
    fetch(url)
      .then(response => response.json())
      .then(data => resolve(data))
      .catch(error => reject(error));
  });
}

makeRequest('http://example.com/')
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

在第二个示例中，`makeRequest`函数返回一个`promise`，该`promise`在网络请求的结果解析后解析，或在出现错误时拒绝。调用者可以使用返回的`promise`的`then`和`catch`方法来处理请求的结果：

```
// using async/await
async function makeRequest(url) {
  try {
    const response = await fetch(url);
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error(error);
  }
}

makeRequest('http://example.com/');
```

在第三个示例中，`makeRequest`函数使用`async`关键字声明，允许它使用`await`关键字等待网络请求的结果。调用者可以使用`try`和`catch`关键字来处理函数执行过程中可能发生的任何错误。

# 背景

JavaScript 中的回调函数可以作为参数传递给另一个函数，并在某些异步操作完成后执行。回调函数通常用于处理异步操作的结果，例如网络请求或用户输入。

使用回调函数的一个主要缺点是可能导致所谓的“回调地狱”——嵌套回调变得难以阅读和维护。考虑以下示例：

```
function makeRequest1(url, callback) {
  // make network request
  callback(null, response);
}

function makeRequest2(url, callback) {
  // make network request
  callback(null, response);
}

function makeRequest3(url, callback) {
  // make network request
  callback(null, response);
}

makeRequest1('http://example.com/1', (error, data1) => {
  if (error) {
    console.error(error);
    return;
  }

  makeRequest2('http://example.com/2', (error, data2) => {
    if (error) {
      console.error(error);
      return;
    }

    makeRequest3('http://example.com/3', (error, data3) => {
      if (error) {
        console.error(error);
        return;
      }

      // do something with data1, data2, data3
    });
  });
});
```

在这个例子中，`makeRequest1`函数发起网络请求，然后调用`callback`函数处理请求的结果。`callback`函数然后使用`makeRequest2`函数进行第二次网络请求，并使用其结果调用另一个`callback`函数。这种模式继续用于第三个网络请求。

# Promise 模式

Promise 是 JavaScript 中处理异步操作的更现代方法。Promise 是表示异步操作结果的对象。它可以处于三种状态之一：pending（进行中）、fulfilled（已成功）或 rejected（已失败）。Promise 就像是一个合同，可以通过成功或失败来解决。

您可以使用`Promise`构造函数创建一个 promise，该构造函数接受一个函数作为参数。该函数接收两个参数：`resolve`和`reject`。当异步操作成功完成时调用`resolve`函数，如果操作失败则调用`reject`函数。

下面是一个示例，展示了如何使用 promise 进行网络请求：

```
function makeRequest(url) {
  return new Promise((resolve, reject) => {
    fetch(url)
      .then(response => response.json())
      .then(data => resolve(data))
      .catch(error => reject(error));
  });
}

makeRequest('http://example.com/')
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

在这个例子中，`makeRequest` 函数返回一个表示网络请求结果的`promise`。函数内部使用`fetch`方法进行 HTTP 请求。如果请求成功，promise 将以响应数据完成。如果失败，则以错误拒绝。调用者可以在返回的 promise 上使用`then`和`catch`方法来处理请求的结果。

使用 promise 而不是回调函数的主要优势之一是它们提供了一种更结构化和可读的方法来处理异步操作。这使您可以避免“回调地狱”，编写更易于理解和维护的代码。

下面的章节提供了更多示例，这些示例将有助于您理解在 JavaScript 中可以使用的不同 promise 设计模式。

## Promise 链式调用

这种模式允许您链式连接多个 promise 以创建更复杂的`async`逻辑：

```
function makeRequest(url) {
  return new Promise((resolve, reject) => {
    fetch(url)
      .then(response => response.json())
      .then(data => resolve(data))
      .catch(error => reject(error));
  });
}

function processData(data) {
  // process data
  return processedData;
}

makeRequest('http://example.com/')
  .then(data => processData(data))
  .then(processedData => console.log(processedData))
  .catch(error => console.error(error));
```

## Promise 错误处理

这种模式使用`catch`方法来处理 promise 链执行过程中可能出现的错误：

```
makeRequest('http://example.com/')
  .then(data => processData(data))
  .then(processedData => console.log(processedData))
  .catch(error => console.error(error));
```

## Promise 并行

这种模式允许您使用`Promise.all`方法并行运行多个 promise：

```
Promise.all([
  makeRequest('http://example.com/1'),
  makeRequest('http://example.com/2')
]).then(([data1, data2]) => {
  console.log(data1, data2);
});
```

## Promise 顺序执行

这种模式允许您使用`Promise.resolve`方法按顺序运行 promise：

```
Promise.resolve()
  .then(() => makeRequest1())
  .then(() => makeRequest2())
  .then(() => makeRequest3())
  .then(() => {
    // all requests completed
  });
```

## Promise 记忆化

这种模式使用缓存存储 promise 函数调用的结果，避免重复请求：

```
const cache = new Map();

function memoizedMakeRequest(url) {
  if (cache.has(url)) {
    return cache.get(url);
  }

  return new Promise((resolve, reject) => {
    fetch(url)
      .then(response => response.json())
      .then(data => {
        cache.set(url, data);
        resolve(data);
      })
      .catch(error => reject(error));
  });
}
```

在此示例中，我们将演示如何使用`memoizedMakeRequest`函数避免重复请求：

```
const button = document.querySelector('button');
button.addEventListener('click', () => {
  memoizedMakeRequest('http://example.com/')
    .then(data => console.log(data))
    .catch(error => console.error(error));
});
```

现在，当按钮被点击时，将调用`memoizedMakeRequest`函数。如果请求的 URL 已经在缓存中，则返回缓存数据。否则，将进行新的请求，并将结果缓存以供将来使用。

## Promise 管道

这种模式使用 promise 和函数式编程技术来创建`async`转换的管道：

```
function transform1(data) {
  // transform data
  return transformedData;
}

function transform2(data) {
  // transform data
  return transformedData;
}

makeRequest('http://example.com/')
  .then(data => pipeline(data)
    .then(transform1)
    .then(transform2))
  .then(transformedData => console.log(transformedData))
  .catch(error => console.error(error));
```

## Promise 重试

这种模式允许您在 promise 失败时重试：

```
function makeRequestWithRetry(url) {
  let attempts = 0;

  const makeRequest = () => new Promise((resolve, reject) => {
    fetch(url)
      .then(response => response.json())
      .then(data => resolve(data))
      .catch(error => reject(error));
  });

  const retry = error => {
    attempts++;
    if (attempts >= 3) {
      throw new Error('Request failed after 3 attempts.');
    }
    console.log(`Retrying request: attempt ${attempts}`);
    return makeRequest();
  };

  return makeRequest().catch(retry);
}
```

## Promise 装饰器

这种模式使用高阶函数创建一个装饰器，可应用于 promise 以添加额外的行为：

```
function logger(fn) {
  return function (...args) {
    console.log('Starting function...');
    return fn(...args).then(result => {
      console.log('Function completed.');
      return result;
    });
  };
}

const makeRequestWithLogger = logger(makeRequest);

makeRequestWithLogger('http://example.com/')
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

## Promise 竞速

这种模式允许您并行运行多个 promise，并返回首个解决的结果：

```
Promise.race([
  makeRequest('http://example.com/1'),
  makeRequest('http://example.com/2')
]).then(data => {
  console.log(data);
});
```

# async/await 模式

`async`/`await`是一种语言特性，允许程序员像编写同步代码一样编写异步代码。它建立在 promises 之上，使得处理异步代码更加简单和清晰。

这是一个示例，展示了如何使用`async`/`await`进行异步 HTTP 请求：

```
async function makeRequest() {
  try {
    const response = await fetch('http://example.com/');
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error(error);
  }
}
```

在此示例中，`makeRequest`函数是异步的，因为它使用了`async`关键字。在函数内部，使用`await`关键字暂停函数的执行，直到`fetch`调用解析。如果调用成功，则将数据记录到控制台。如果失败，则捕获错误并记录到控制台。

现在让我们看看其他一些使用`async`的模式。

## async 函数组合

这种模式涉及将多个`async`函数组合在一起，以创建更复杂的`async`逻辑：

```
async function makeRequest(url) {
  const response = await fetch(url);
  const data = await response.json();
  return data;
}

async function processData(data) {
  // process data
  return processedData;
}

async function main() {
  const data = await makeRequest('http://example.com/');
  const processedData = await processData(data);
  console.log(processedData);
}
```

## async 迭代

这种模式允许您使用`for-await-of`循环迭代`async`可迭代对象：

```
async function* createAsyncIterable() {
  yield 1;
  yield 2;
  yield 3;
}

async function main() {
  for await (const value of createAsyncIterable()) {
    console.log(value);
  }
}
```

## async 错误处理

这种模式使用`try-catch`块来处理`async`函数执行过程中可能出现的错误：

```
async function main() {
  try {
    const data = await makeRequest('http://example.com/');
    console.log(data);
  } catch (error) {
    console.error(error);
  }
}
```

## async 并行处理

这种模式允许您使用`Promise.all`方法并行运行多个`async`任务：

```
async function main() {
  const [data1, data2] = await Promise.all([
    makeRequest('http://example.com/1'),
    makeRequest('http://example.com/2')
  ]);

  console.log(data1, data2);
}
```

## async 顺序执行

这种模式允许您使用`Promise.resolve`方法按顺序运行`async`任务：

```
async function main() {
  let result = await Promise.resolve();

  result = await makeRequest1(result);
  result = await makeRequest2(result);
  result = await makeRequest3(result);

  console.log(result);
}
```

## async 记忆化

这种模式使用缓存存储`async`函数调用的结果，避免重复请求：

```
const cache = new Map();

async function memoizedMakeRequest(url) {
  if (cache.has(url)) {
    return cache.get(url);
  }

  const response = await fetch(url);
  const data = await response.json();

  cache.set(url, data);
  return data;
}
```

## async 事件处理

这种模式允许您使用`async`函数处理事件：

```
const button = document.querySelector('button');

async function handleClick() {
  const response = await makeRequest('http://example.com/');
  console.log(response);
}

button.addEventListener('click', handleClick);
```

## async/await 管道

这种模式使用`async`/`await`和函数式编程技术创建`async`转换的管道：

```
async function transform1(data) {
  // transform data
  return transformedData;
}

async function transform2(data) {
  // transform data
  return transformedData;
}

async function main() {
  const data = await makeRequest('http://example.com/');
  const transformedData = await pipeline(data)
    .then(transform1)
    .then(transform2);

  console.log(transformedData);
}
```

## async 重试

这种模式允许您在`async`操作失败时重试：

```
async function makeRequestWithRetry(url) {
  let attempts = 0;

  while (attempts < 3) {
    try {
      const response = await fetch(url);
      const data = await response.json();
      return data;
    } catch (error) {
      attempts++;
      console.log(`Retrying request: attempt ${attempts}`);
    }
  }

  throw new Error('Request failed after 3 attempts.');
}
```

## async/await 装饰器

这种模式使用高阶函数创建一个装饰器，可以应用于`async`函数以添加额外的行为：

```
function asyncLogger(fn) {
  return async function (...args) {
    console.log('Starting async function...');
    const result = await fn(...args);
    console.log('Async function completed.');
    return result;
  };
}

@asyncLogger
async function main() {
  const data = await makeRequest('http://example.com/');
  console.log(data);
}
```

# 其他实际示例

除了前几节讨论的模式外，让我们看一些在 JavaScript 中使用`async`/`await`的实际示例。

## 发送 HTTP 请求

```
async function makeRequest(url) {
  try {
    const response = await fetch(url);
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error(error);
  }
}
```

## 从文件系统中读取文件

```
async function readFile(filePath) {
  try {
    const fileData = await fs.promises.readFile(filePath);
    console.log(fileData);
  } catch (error) {
    console.error(error);
  }
}
```

## 写入文件到文件系统

```
async function writeFile(filePath, data) {
  try {
    await fs.promises.writeFile(filePath, data);
    console.log('File written successfully.');
  } catch (error) {
    console.error(error);
  }
}
```

## 执行多个 async 操作

```
async function main() {
  try {
    const [data1, data2] = await Promise.all([
      makeRequest1(),
      makeRequest2()
    ]);
    console.log(data1, data2);
  } catch (error) {
    console.error(error);
  }
}
```

## 顺序执行多个 async 操作

```
async function main() {
  try {
    const data1 = await makeRequest1();
    const data2 = await makeRequest2();
    console.log(data1, data2);
  } catch (error) {
    console.error(error);
  }
}
```

## 缓存 async 操作的结果

```
const cache = new Map();

async function makeRequest(url) {
  if (cache.has(url)) {
    return cache.get(url);
  }

  try {
    const response = await fetch(url);
    const data = await response.json();
    cache.set(url, data);
    return data;
  } catch (error) {
    throw error;
  }
}
```

## 使用 async/await 处理事件

```
const button = document.querySelector('button');

button.addEventListener('click', async () => {
  try {
    const data = await makeRequest('http://example.com/');
    console.log(data);
  } catch (error) {
    console.error(error);
  }
});
```

## 在失败时重试 async 操作

```
async function makeRequest(url) {
  try {
    const response = await fetch(url);
    const data = await response.json();
    return data;
  } catch (error) {
    throw error;
  }
}

async function retry(fn, maxRetries = 3, retryDelay = 1000) {
  let retries = 0;

  while (retries <= maxRetries) {
    try {
      return await fn();
    } catch (error) {
      retries++;
      console.error(error);
      await new Promise(resolve => setTimeout(resolve, retryDelay));
    }
  }

  throw new Error(`Failed after ${retries} retries.`);
}

retry(() => makeRequest('http://example.com/')).then(data => {
  console.log(data);
});
```

## 创建一个 async/await 装饰器

```
function asyncDecorator(fn) {
  return async function(...args) {
    try {
      return await fn(...args);
    } catch (error) {
      throw error;
    }
  };
}
const makeRequest = asyncDecorator(async function(url) {
  const response = await fetch(url);
  const data = await response.json();
  return data;
});

makeRequest('http://example.com/').then(data => {
  console.log(data);
});
```

# 总结

本章涵盖了一系列广泛的模式和示例，这些模式在编写用于在后台执行长时间运行任务的异步代码时非常有用。我们看到`callback`函数为 promises 和`async`/`await`执行一个或多个`async`任务铺平了道路。

在下一章中，我们将看一看应用架构模式的另一个角度。我们将看看模块化开发的模式是如何随着时间的推移而演变的。
