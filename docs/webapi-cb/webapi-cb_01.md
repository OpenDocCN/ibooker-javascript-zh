# 第一章：异步 API

# 简介

本书涵盖的许多 API 都是*异步*的。当你调用其中一个函数或方法时，可能不会立即得到结果。不同的 API 有不同的机制，在准备好结果时将结果返回给你。

## 回调函数

最基本的异步模式是*回调函数*。这是一个你传递给异步 API 的函数。当工作完成时，它调用你的回调函数并传递结果。回调函数可以单独使用，也可以作为其他异步模式的一部分使用。

## 事件

许多浏览器 API 是基于*事件*的。事件是异步发生的事情。一些事件的例子包括：

+   按钮被点击了。

+   鼠标移动了。

+   网络请求完成了。

+   发生了错误。

事件有一个名称，比如 `click` 或 `mouseover`，以及一个包含有关事件发生的数据的对象。这可能包括点击了哪个元素或者 HTTP 状态码等信息。当你监听事件时，你提供一个回调函数，该函数接收事件对象作为参数。

实现事件的对象实现了 `EventTarget` 接口，该接口提供了 `addEventListener` 和 `removeEventListener` 方法。要监听元素或其他对象上的事件，你可以在其上调用 `addEventListener`，传递事件名称和处理函数。每次触发事件时都会调用回调函数，直到它被移除。可以通过调用 `removeEventListener` 手动移除侦听器，或者在许多情况下，当对象被销毁或从 DOM 中移除时，浏览器会自动移除侦听器。

## Promises

许多较新的 API 使用 `Promise`。`Promise` 是一个从函数返回的对象，它是异步操作最终结果的占位符。与监听事件不同，你在 `Promise` 对象上调用 `then`。你将一个回调函数传递给 `then`，该函数最终将以结果作为其参数调用。为了处理错误，你可以将另一个回调函数传递给 `Promise` 的 `catch` 方法。

当操作成功完成时，`Promise` 将*完成*，当出现错误时，`Promise` 将*拒绝*。完成的值作为参数传递给 `then` 回调，拒绝的值作为参数传递给 `catch` 回调。

事件和 `Promise` 之间有一些关键区别：

+   事件处理程序会多次触发，而 `then` 回调只会执行一次。你可以把 `Promise` 想象成一次性操作。

+   如果你在 `Promise` 上调用 `then` 方法，你总会得到结果（如果有的话）。这与事件不同，如果事件在你添加监听器之前发生，那么该事件将会丢失。

+   `Promise` 具有内置的错误处理机制。在事件中，通常需要监听错误事件来处理错误情况。

# 使用 Promises

## 问题

你想调用一个使用 `Promise` 的 API，并获取结果。

## 解决方案

调用`Promise`对象的`then`方法来处理回调函数中的结果。为了处理可能的错误，添加一个调用`catch`。

假设您有一个函数`getUsers`，它会发出网络请求来加载用户列表。此函数返回一个`Promise`，最终解析为用户列表（参见示例 1-1）。

##### 示例 1-1\. 使用基于`Promise`的 API

```
getUsers()
  .then(
    // This function is called when the user list has been loaded.
    userList => {
      console.log('User List:');
      userList.forEach(user => {
        console.log(user.name);
      });
    }
  ).catch(error => {
    console.error('Failed to load the user list:', error);
  });
```

## 讨论

从`getUsers`返回的`Promise`是一个带有`then`方法的对象。当用户列表加载完成时，执行传递给`then`的回调函数，并将用户列表作为其参数。

此`Promise`还具有用于处理错误的`catch`方法。如果在加载用户列表时出现错误，则调用传递给`catch`的回调函数以处理错误对象。根据结果只调用其中一个回调函数。

# 使用备用加载图片

## 问题

您希望加载一张图片以在页面上显示。如果加载图片时出现错误，您希望使用已知的良好图片 URL 作为备用。

## 解决方案

使用编程方式创建`Image`元素，并监听其`load`和`error`事件。如果触发了`error`事件，则用备用图片替换它。一旦请求的图片或占位图片加载完成，根据需要将其添加到 DOM 中。

为了更清晰的 API，您可以将其封装在一个`Promise`中。该`Promise`要么解析为要添加的`Image`，要么由于无法加载图片或备用图片而拒绝（参见示例 1-2）。

##### 示例 1-2\. 使用备用加载图片

```
/**
 * Loads an image. If there's an error loading the image, uses a fallback
 * image URL instead.
 *
 * @param url The image URL to load
 * @param fallbackUrl The fallback image to load if there's an error
 * @returns a Promise that resolves to an Image element to insert into the DOM
 */
function loadImage(url, fallbackUrl) {
  return new Promise((resolve, reject) => {
    const image = new Image();

    // Attempt to load the image from the given URL
    image.src = url;

    // The image triggers the 'load' event when it is successfully loaded.
    image.addEventListener('load', () => {
      // The now-loaded image is used to resolve the Promise
      resolve(image);
    });

    // If an image failed to load, it triggers the 'error' event.
    image.addEventListener('error', error => {
      // Reject the Promise in one of two scenarios:
      // (1) There is no fallback URL.
      // (2) The fallback URL is the one that failed.
      if (!fallbackUrl || image.src === fallbackUrl) {
        reject(error);
      } else {
        // If this is executed, it means the original image failed to load.
        // Try to load the fallback.
        image.src = fallbackUrl;
      }
    });
  });
}
```

## 讨论

`loadImage`函数接受一个 URL 和一个备用 URL，并返回一个`Promise`。然后它创建一个新的`Image`，并将其`src`属性设置为给定的 URL。浏览器尝试加载图片。

有三种可能的结果：

成功情况

如果图片成功加载，则触发`load`事件。事件处理程序使用该图片解析`Promise`，然后可以将其插入到 DOM 中。

备用情况

如果图片加载失败，则触发`error`事件。错误处理程序将`src`属性设置为备用 URL，并尝试加载备用图片。如果*成功*，则`load`事件触发并使用备用`Image`解析`Promise`。

失败情况

如果无法加载图片或备用图片，则错误处理程序拒绝带有`error`事件的`Promise`。

每次加载错误时都会触发`error`事件。处理程序首先检查是否是备用 URL 加载失败。如果是，则表示原始 URL 和备用 URL 都无法加载。这是失败的情况，因此拒绝`Promise`。

如果不是备用 URL，则表示请求的 URL 加载失败。现在设置备用 URL 并尝试加载它。

这里检查的顺序很重要。如果缺少第一次检查，如果后备加载失败，错误处理程序将触发设置（无效）后备 URL、请求它并再次触发`error`事件的无限循环。

示例 1-3 展示了如何使用`loadImage`函数。

##### 示例 1-3\. 使用`loadImage`函数

```
loadImage('https://example.com/profile.jpg', 'https://example.com/fallback.jpg')
  .then(image => {
    // container is an element in the DOM where the image will go
    container.appendChild(image);
  }).catch(error => {
    console.error('Image load failed');
  });
```

# 链接`Promise`

## 问题

您希望按顺序调用多个基于`Promise`的 API。每个操作都依赖于前一个操作的结果。

## 解决方案

使用`Promise`链来顺序执行异步任务。想象一个博客应用程序，其中有两个 API，都返回`Promise`：

`getUser(id)`

根据给定的用户 ID 加载用户

`getPosts(user)`

加载给定用户的所有博客帖子

如果要加载用户的帖子，首先需要加载`user`对象——在加载用户详细信息之前无法调用`getPosts`。您可以通过将这两个`Promise`链在一起来实现，如示例 1-4 所示。

##### 示例 1-4\. 使用`Promise`链

```
/**
 * Loads the post titles for a given user ID.
 * @param userId is the ID of the user whose posts you want to load
 * @returns a Promise that resolves to an array of post titles
 */
function getPostTitles(userId) {
  return getUser(userId)
    // Callback is called with the loaded user object
    .then(user => {
      console.log(`Getting posts for ${user.name}`);
      // This Promise is also returned from .then
      return getPosts(user);
    })
    // Calling then on the getPosts' Promise
    .then(posts => {
      // Returns another Promise that will resolve to an array of post titles
      return posts.map(post => post.title);
    })
    // Called if either getUser or getPosts are rejected
    .catch(error => {
      console.error('Error loading data:', error);
    });
}
```

## 讨论

`Promise`的`then`处理程序返回的值被包装在一个新的`Promise`中。这个`Promise`从`then`方法本身返回。这意味着`then`的返回值也是一个`Promise`，因此您可以链式调用另一个`then`。这就是如何创建`Promise`链的方式。

`getUser`返回一个解析为`user`对象的`Promise`。`then`处理程序调用`getPosts`并返回结果的`Promise`，然后再次从`then`返回，因此您可以再次调用`then`来获取最终结果，即帖子数组。

链的末尾调用`catch`来处理任何错误。这类似于`try`/`catch`块。如果链中的任何地方发生错误，`catch`处理程序将被调用，并且不会执行链的其余部分。

# 使用`async`和`await`关键字

## 问题

您正在使用返回`Promise`的 API，但希望代码以更线性或同步的方式阅读。

## 解决方案

使用`await`关键字与`Promise`一起，而不是在其上调用`then`（参见示例 1-5）。再次考虑来自“使用 Promise”的`getUsers`函数。此函数返回一个解析为用户列表的`Promise`。

##### 示例 1-5\. 使用`await`关键字

```
// A function must be declared with the async keyword
// in order to use await in its body.
async function listUsers() {
  try {
    // Equivalent to getUsers().then(...)
    const userList = await getUsers();
    console.log('User List:');
    userList.forEach(user => {
      console.log(user.name);
    });
  } catch (error) { // Equivalent to .catch(...)
    console.error('Failed to load the user list:', error);
  }
}
```

## 讨论

`await`是使用`Promise`的另一种语法。与使用接受结果作为其参数的回调函数调用`then`不同，表达式实际上“暂停”了函数余下的执行，并在`Promise`被履行时返回结果。

如果`Promise`被拒绝，`await`表达式会抛出被拒绝的值。这可以通过标准的`try`/`catch`块来处理。

# 并行使用`Promise`

## 问题

您希望使用`Promise`在并行执行一系列异步任务。

## 解决方案

收集所有`Promise`，并将它们传递给`Promise.all`。这个函数接受一个`Promise`数组，并等待它们全部完成。它返回一个新的`Promise`，一旦所有给定的`Promise`都完成，就会被实现；如果任何给定的`Promise`被拒绝，它就会被拒绝（参见示例 1-6）。

##### 示例 1-6\. 使用`Promise.all`加载多个用户

```
// Loading three users at once
Promise.all([
  getUser(1),
  getUser(2),
  getUser(3)
]).then(users => {
  // users is an array of user objects—the values returned from
  // the parallel getUser calls
}).catch(error => {
  // If any of the above Promises are rejected
  console.error('One of the users failed to load:', error);
});
```

## 讨论

如果您有多个不依赖于彼此的任务，`Promise.all`是一个很好的选择。示例 1-6 调用了三次`getUser`，每次传递不同的用户 ID。它将这些`Promise`收集到一个数组中，并将其传递给`Promise.all`。所有三个请求并行运行。

`Promise.all`返回另一个`Promise`。一旦所有三个用户成功加载，这个新的`Promise`将会被实现，其包含一个包含已加载用户的数组。每个结果的索引对应于输入数组中`Promise`的索引。在这种情况下，它按顺序返回用户`1`、`2`和`3`的数组。

如果其中一个用户未能加载怎么办？也许其中一个用户 ID 不存在或者存在临时网络错误。如果`Promise.all`传递的*任何*`Promise`被拒绝，那么新的`Promise`将立即被拒绝。拒绝值与被拒绝的`Promise`的拒绝值相同。

如果其中一个用户加载失败，由`Promise.all`返回的`Promise`将被拒绝，并且会带有出错的错误。其他`Promise`的结果将会丢失。

如果您仍然希望获取任何已解决`Promise`的结果（或来自其他被拒绝`Promise`的错误），可以改用`Promise.allSettled`。使用`Promise.allSettled`，会返回一个新的`Promise`，与`Promise.all`类似。但是，这个`Promise`始终会在所有`Promise`都已处理完毕（无论是已解决还是*已拒绝*）后被实现。

如示例 1-7 所示，解决的值是一个数组，其中每个元素都有一个`status`属性。这个属性可以是`fulfilled`或`rejected`，具体取决于该`Promise`的结果。如果状态是`fulfilled`，则对象还具有一个`value`属性，即解决的值。另一方面，如果状态是`rejected`，则对象有一个`reason`属性，即被拒绝的值。

##### 示例 1-7\. 使用`Promise.allSettled`

```
Promise.allSettled([
  getUser(1),
  getUser(2),
  getUser(3)
]).then(results => {
  results.forEach(result => {
    if (result.status === 'fulfilled') {
      console.log('- User:', result.value.name);
    } else {
      console.log('- Error:', result.reason);
    }
  });
});
// No catch necessary here because allSettled is always fulfilled.
```

# 使用`requestAnimationFrame`来为元素添加动画效果

## 问题

您想要使用 JavaScript 以高效的方式对元素进行动画处理。

## 解决方案

使用`requestAnimationFrame`函数来安排动画更新以在规律的间隔运行。

假设您有一个要使用淡出动画隐藏的`div`元素。这通过调整不透明度，在`requestAnimationFrame`中使用的回调函数来完成（参见示例 1-8）。每个间隔的持续时间取决于动画的期望每秒帧数（FPS）。

##### 示例 1-8\. 使用`requestAnimationFrame`进行淡出动画

```
const animationSeconds = 2; // Animate over 2 seconds
const fps = 60; // A nice, smooth animation

// The time interval between each frame
const frameInterval = 1000 / fps;

// The total number of frames for the animation
const frameCount = animationSeconds * fps;

// The amount to adjust the opacity by in each frame
const opacityIncrement = 1 / frameCount;

// The timestamp of the last frame
let lastTimestamp;

// The starting opacity value
let opacity = 1;

function fade(timestamp) {
  // Set the last timestamp to now if there isn't an existing one.
  if (!lastTimestamp) {
    lastTimestamp = timestamp;
  }

  // Calculate how much time has elapsed since the last frame.
  // If not enough time has passed yet, schedule another call of this
  // function and return.
  const elapsed = timestamp - lastTimestamp;
  if (elapsed < frameInterval) {
    requestAnimationFrame(animate);
    return;
  }

  // Time for a new animation frame. Remember this timestamp.
  lastTimestamp = timestamp;

  // Adjust the opacity value and make sure it doesn't go below 0.
  opacity = Math.max(0, opacity - opacityIncrement)
  box.style.opacity = opacity;

  // If the opacity hasn't reached the target value of 0, schedule another
  // call to this function.
  if (opacity > 0) {
    requestAnimationFrame(animate);
  }
}

// Schedule the first call to the animation function.
requestAnimationFrame(fade);
```

## 讨论

这是使用 JavaScript 进行元素动画的一种良好且高效的方法，具有良好的浏览器支持。因为它是异步完成的，这种动画不会阻塞浏览器的主线程。如果用户切换到另一个标签页，动画会暂停，并且不会不必要地调用`requestAnimationFrame`。

当您使用`requestAnimationFrame`调度函数运行时，该函数在下一次重绘操作之前被调用。这种情况发生的频率取决于浏览器和屏幕刷新率。

在动画之前，示例 1-8 根据给定的动画持续时间（2 秒）和帧率（每秒 60 帧）进行一些计算。它计算出总帧数，并使用持续时间计算每帧的运行时间。如果您想要与系统刷新率不匹配的不同帧率，这将跟踪上次动画更新执行的时间，以维持目标帧率。

然后，根据帧数，计算每一帧中的透明度调整。

通过将其传递给`requestAnimationFrame`调用，调度`fade`函数。每次浏览器调用此函数时，都会传递一个时间戳。`fade`函数计算自上一帧以来经过的时间。如果尚未经过足够的时间，则不执行任何操作，并要求浏览器在下次再次调用时执行。

一旦经过足够的时间，它执行动画步骤。它获取计算的不透明度调整，并将其应用于元素的样式。根据确切的时机，这可能导致不透明度小于 0，这是无效的。使用`Math.max`修复此问题，以设置最小值为 0。

如果不透明度尚未达到 0，则需要执行更多的动画帧。它再次调用`requestAnimationFrame`来调度下一次执行。

作为此方法的替代方案，较新的浏览器支持 Web 动画 API，您将在第八章中了解它。该 API 允许您指定带有 CSS 属性的关键帧，并且浏览器会处理为您更新中间值。

# 将事件 API 封装在 Promise 中

## 问题

您想要封装一个基于事件的 API 以返回`Promise`。

## 解决方案

创建一个新的`Promise`对象，并在其构造函数中注册事件侦听器。当接收到等待的事件时，使用值解析`Promise`。类似地，如果发生错误事件，则拒绝`Promise`。

有时这被称为“将函数转换为 Promise”。示例 1-9 演示了如何将`XMLHttpRequest`API 转换为 Promise。

##### 示例 1-9\. 将`XMLHttpRequest`API 转换为 Promise

```
/**
 * Sends a GET request to the specified URL. Returns a Promise that will resolve to
 * the JSON body parsed as an object, or will reject if there is an error or the
 * response is not valid JSON.
 *
 * @param url The URL to request
 * @returns a Promise that resolves to the response body
 */
function loadJSON(url) {
  // Create a new Promise object, performing the async work inside the
  // constructor function.
  return new Promise((resolve, reject) => {
    const request = new XMLHttpRequest();

    // If the request is successful, parse the JSON response and
    // resolve the Promise with the resulting object.
    request.addEventListener('load', event => {
      // Wrap the JSON.parse call in a try/catch block just in case
      // the response body is not valid JSON.
      try {
        resolve(JSON.parse(event.target.responseText));
      } catch (error) {
        // There was an error parsing the response body.
        // Reject the Promise with this error.
        reject(error);
      }
    });

    // If the request fails, reject the Promise with the
    // error that was emitted.
    request.addEventListener('error', error => {
      reject(error);
    });

    // Set the target URL and send the request.
    request.open('GET', url);
    request.send();
  });
}
```

示例 1-10 展示了如何使用转换为 Promise 的`loadJSON`函数。

##### Example 1-10\. 使用`loadJSON`辅助函数

```
// Using .then
loadJSON('/api/users/1').then(user => {
  console.log('Got user:', user);
})

// Using await
const user = await loadJSON('/api/users/1');
console.log('Got user:', user);
```

## 讨论

通过使用`new`运算符调用`Promise` *构造函数* 来创建一个`Promise`。此函数接收两个参数，一个`resolve`函数和一个`reject`函数。

`resolve` 和 `reject` 函数是由 JavaScript 引擎提供的。在`Promise`构造函数内部，你可以执行异步工作并监听事件。当调用`resolve`函数时，`Promise`立即以该值解析。调用`reject`也同样工作——它会用错误拒绝`Promise`。

创建自己的`Promise`可以帮助处理这类情况，但通常情况下，你不需要像这样手动创建它们。如果一个 API 已经返回了`Promise`，你不需要再将其包装在自己的`Promise`中，直接使用即可。
