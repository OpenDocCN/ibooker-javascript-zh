# 第十五章：测量性能

# 介绍

JavaScript 应用程序中有许多第三方工具可用于性能测量，但浏览器还内置了一些方便的工具来捕获性能指标。

导航时间 API 用于捕获有关初始页面加载的性能数据。您可以检查页面加载所需的时间、DOM 变得可交互所需的时间等。它返回一组时间戳，指示页面加载期间每个事件发生的时间。

资源定时 API 允许您检查下载资源和进行网络请求所花费的时间。这涵盖页面资源，如 HTML 文件、CSS 文件、JavaScript 文件和图像，还包括使用 Fetch API 进行的异步请求。

用户定时 API 是一种计算任意操作经过的时间的方法。您可以创建性能 *标记*，这些是时间点，以及 *度量*，这是两个标记之间的计算持续时间。

所有这些 API 在页面上创建性能条目缓冲区。这是所有类型性能条目的单一集合。您可以随时检查此缓冲区，也可以使用 `PerformanceObserver` 异步监听新的性能条目添加。

性能条目使用高精度时间戳。时间以毫秒为单位测量，但在某些浏览器中还可以包含微秒精度的小数部分。在浏览器中，这些时间戳存储为 `DOMHighResTimeStamp` 对象。这些是数字，从页面加载时开始计时，表示自页面加载以来给定条目发生的时间。

本章的示例探讨了收集性能指标的解决方案。如何处理这些指标取决于您。您可以使用 Fetch 或 Beacon API 将性能指标发送到 API 进行收集和后续分析。

在开发过程中，您可以使用这些性能指标进行调试，或者保留它们以收集用户的实际性能指标。这些数据可以发送到分析服务进行聚合和分析。

# 测量页面加载性能

## 问题

您想要收集有关页面加载事件时间的信息。

## 解决方案

查找类型为 `navigation` 的单个性能条目，并从性能条目对象中检索导航时间戳（参见 示例 15-1）。然后可以计算这些时间戳之间的间隔，以了解各种页面加载事件所需的时间。

##### 示例 15-1\. 查找导航时间性能条目

```
// There is only one navigation performance entry.
const [navigation] = window.performance.getEntriesByType('navigation');
```

此对象具有许多属性。表 15-1 列出了一些有用的计算示例。

表 15-1\. 导航时间计算

| 指标 | 开始时间 | 结束时间 |
| --- | --- | --- |
| 首字节时间 | `startTime` | `responseStart` |
| DOM 交互时间 | `startTime` | `domInteractive` |
| 总加载时间 | `startTime` | `loadEventEnd` |

## 讨论

导航性能条目的`startTime`属性始终为 0。

此条目不仅包含时间信息，还包括数据传输量、HTTP 响应代码和页面 URL 等信息。这些信息对于确定应用程序在首次加载时的响应速度非常有用。

# 测量资源性能

## 问题

想要获取加载在页面上的资源的信息。

## 解决方案

在性能缓冲区中查找资源性能条目（见示例 15-2）。

##### 示例 15-2\. 获取资源性能条目

```
const entries = window.performance.getEntriesByType('resource');
```

每个页面资源都会生成一个条目。资源包括 CSS 文件、JavaScript 文件、图像以及页面发出的任何其他请求。

您可以通过计算`startTime`和`responseEnd`属性之间的差来确定每个资源的加载时间。资源的 URL 可在`name`属性中找到。

## 讨论

使用 Fetch API 进行的任何网络请求都会显示为一个资源。这使得该 API 对于分析您的 REST API 端点的实际性能非常有用。

页面首次加载时，性能缓冲区包含初始页面加载期间请求的所有资源的条目。随后的请求在生成时会添加到性能缓冲区中。

# 找出加载最慢的资源

## 问题

想要获取加载时间最长的资源列表。

## 解决方案

对资源性能条目列表进行排序和过滤。由于这个列表只是一个数组，因此可以对其调用诸如`sort`和`slice`之类的方法。要找出资源加载所花费的时间，可以通过`responseEnd`和`startTime`时间戳之间的差来计算。

示例 15-3 展示了如何查找加载最慢的五个资源。

##### 示例 15-3\. 查找加载最慢的五个资源

```
const slowestResources = window.performance.getEntriesByType('resource')
  .sort((a, b) =>
    (b.responseEnd - b.startTime) - (a.responseEnd - a.startTime))
  .slice(0, 5);
```

## 讨论

关键在于`sort`调用。此调用将每对加载时间进行比较，并按加载时间降序对整个列表进行排序。然后，`slice`调用只获取排序后数组的前五个元素。

如果要获取加载时间最快的五个资源列表，只需反转比较加载时间的顺序即可（见示例 15-4）。

##### 示例 15-4\. 查找加载最快的 5 个资源

```
const fastestResources = window.performance.getEntriesByType('resource')
  .sort((a, b) =>
    (a.responseEnd - a.startTime) - (b.responseEnd - b.startTime))
  .slice(0, 5);
```

反转比较意味着数组按升序而非降序排序。现在，`slice`调用返回加载时间最快的五个资源。

# 找到特定资源的时间信息

## 问题

想要查找特定资源请求的时间信息。

## 解决方案

使用方法`window.performance.getEntriesByName`按特定 URL 查找资源（见示例 15-5）。

##### 示例 15-5\. 查找特定 URL 的所有资源时间

```
// Look up all requests to the /api/users API
const entries = window.performance.getEntriesByName('https://localhost/api/users',
'resource');
```

## 讨论

资源条目的名称是其 URL。`getEntriesByName`的第一个参数是 URL，第二个参数表示您对资源时间信息感兴趣。

如果对给定 URL 有多个请求，则返回的数组中将有多个资源条目。

# 分析渲染性能

## 问题

您希望记录页面上渲染某些数据所需的时间。

## 解决方案

在渲染开始之前创建一个性能标记。一旦渲染完成，再创建另一个标记。然后，您可以在两个标记之间创建一个*度量*来记录渲染所需的时间。

假设您有一个`DataView`组件，可以用来在页面上渲染一些数据（参见示例 15-6）。

##### 示例 15-6\. 测量渲染性能

```
// Create the initial performance mark just before rendering.
window.performance.mark('render-start');

// Create the component and render the data.
const dataView = new DataView();
dataView.render(data);

// When rendering is done, create the ending performance mark.
window.performance.mark('render-end');

// Create a measure between the two marks.
const measure = window.performance.measure('render', 'render-start', 'render-end');
```

`measure`对象包含度量的开始时间和计算出的持续时间。

## 讨论

每当创建性能标记和度量时，它们都会添加到页面的性能缓冲区以供以后查找。例如，如果您想稍后查找`render`度量，可以使用`window.performance.getEntriesByName`（参见示例 15-7）。

##### 示例 15-7\. 按名称查找度量

```
// There is only one 'render' measure, so you can use
// array destructuring to get the first (and only) entry.
const [renderMeasure] = window.performance.getEntriesByName('render');
```

标记和度量也可以通过传递`detail`选项与它们关联的数据。例如，在渲染示例 15-6 中的数据时，您可以在创建度量时将数据本身作为元数据传递。

使用这种方式创建度量时，需要在选项对象内包含起始和结束标记（参见示例 15-8）。

##### 示例 15-8\. 使用数据测量渲染性能

```
// Create the initial performance mark just before rendering.
window.performance.mark('render-start');

// Create the component and render the data.
const dataView = new DataView();
dataView.render(data);

// When rendering is done, create the ending performance mark.
window.performance.mark('render-end');

// Create a measure between the two marks, passing the
// data being rendered as the measure detail.
const measure = window.performance.measure('render', {
  start: 'render-start',
  end: 'render-end',
  detail: data
});
```

稍后，当您查找此性能条目时，度量的`detail`元数据将可用。

# 分析多步骤任务

## 问题

您希望收集多步骤过程的性能数据。您希望获取整个序列的时间，但也需要各个步骤的时间。例如，您可能希望从 API 加载一些数据，然后对这些数据进行处理。在这种情况下，您希望知道 API 请求的时间，处理的时间以及总共花费的时间。

## 解决方案

创建多个标记和度量。您可以在多个度量计算中使用给定的标记。

在示例 15-9 中，有一个 API 返回一些用户交易。一旦接收到交易数据，您希望对交易数据进行分析。最后，分析数据将发送到另一个 API。

##### 示例 15-9\. 分析多步骤过程

```
window.performance.mark('transactions-start');
const transactions = await fetch('/api/users/123/transactions');
window.performance.mark('transactions-end');
window.performance.mark('process-start');
const analytics = processAnalytics(transactions);
window.performance.mark('process-end');
window.performance.mark('upload-start');
await fetch('/api/analytics', {
  method: 'POST',
  body: JSON.stringify(analytics),
  headers: {
    'Content-Type': 'application/json'
  }
});
window.performance.mark('upload-end');
```

一旦进程完成并且标记已经获取，您可以使用这些标记生成多个度量，如示例 15-10 所示。

##### 示例 15-10\. 生成度量

```
console.log('Download transactions:',
  window.performance.measure(
    'transactions', 'transactions-start', 'transactions-end'
  ).duration
);

console.log('Process analytics:',
  window.performance.measure(
    'analytics', 'process-start', 'process-end'
  ).duration
);

console.log('Upload analytics:',
  window.performance.measure(
    'upload', 'upload-start', 'upload-end'
  ).duration
);

console.log('Total time:',
  window.performance.measure(
    'total', 'transactions-start', 'upload-end'
  ).duration
);
```

## 讨论

这个示例展示了如何创建多个标记和度量来收集一组任务的性能数据。给定的标记可以在多个度量中使用多次。示例 15-10 为每个步骤创建一个度量，然后生成整个任务持续时间的最终度量。这是通过获取下载任务的第一个标记和上传任务的最后一个标记，并计算它们之间的度量来完成的。

# 监听性能条目

## 问题

您希望在有新的性能条目时收到通知，以便将其报告给分析服务。例如，考虑您希望在每次发出 API 请求时通知性能统计数据的情况。

## 解决方案

使用`PerformanceObserver`监听所需类型的新性能条目。对于 API 请求，类型将是`resource`（参见 示例 15-11）。

##### 示例 15-11\. 使用`PerformanceObserver`

```
const analyticsEndpoint = 'https://example.com/api/analytics';

const observer = new PerformanceObserver(entries => {
  for (let entry of entries.getEntries()) {
    // Only interested in 'fetch' entries.
    // Use the Beacon API to send a quick request containing the performance
    // entry data.
    if (entry.initiatorType === 'fetch') {
      navigator.sendBeacon(analyticsEndpoint, entry);
    }
  }
});

observer.observe({ type: 'resource' });
```

## 讨论

`PerformanceObserver`会为每个网络请求触发，包括您发送到分析服务的请求。因此，示例 15-11 在发送请求之前会检查给定条目是否不是分析端点。如果没有这个检查，您会陷入无限的 POST 请求循环中。当发出网络请求时，观察者触发并发送 POST 请求。这会创建一个新的性能条目，进而再次调用观察者。每次向分析服务发送 POST 请求都会触发新的观察者回调。

为了防止在短时间内向分析服务发送大量请求，对于真实的应用程序，您可能希望将性能条目收集到缓冲区中。一旦缓冲区达到一定大小，您可以将缓冲区中的所有条目一次性发送（参见 示例 15-12）。

##### 示例 15-12\. 批量发送性能条目

```
const analyticsEndpoint = 'https://example.com/api/analytics';

// An array to hold buffered entries. Once the buffer reaches the desired size,
// all entries are sent in a single request.
const BUFFER_SIZE = 10;
let buffer = [];

const observer = new PerformanceObserver(entries => {
  for (let entry of entries.getEntries()) {
    if (entry.initiatorType === 'fetch' && entry.name !== analyticsEndpoint) {
      buffer.push(entry);
    }

    // If the buffer has reached its target size, send the analytics request.
    if (buffer.length === BUFFER_SIZE) {
      fetch(analyticsEndpoint, {
        method: 'POST',
        body: JSON.stringify(buffer),
        headers: {
          'Content-Type': 'application/json'
        }
      });

      // Reset the buffer now that the batched entries have been sent.
      buffer = [];
    }
  }
});

observer.observe({ type: 'resource' });
```
