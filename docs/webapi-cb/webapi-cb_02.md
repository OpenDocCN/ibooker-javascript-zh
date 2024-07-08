# 第二章：使用 Web Storage API 进行简单持久化

# 介绍

Web Storage API 在用户的浏览器中本地持久化简单数据。即使在关闭并重新打开浏览器后，你仍然可以检索这些数据。

此 API 具有提供数据访问和持久性的`Storage`接口。你不直接创建 `Storage` 实例；有两个全局实例：`window.localStorage` 和 `window.sessionStorage`。它们唯一的区别在于数据保留的时长。

`sessionStorage` 数据与特定的浏览器会话相关联。如果页面重新加载，它会保留数据，但完全关闭浏览器会导致数据丢失。相同起源的不同标签页不共享相同的持久化数据。

另一方面，`localStorage` 在同一起源的所有标签页和会话中共享相同的存储空间。即使关闭浏览器后，浏览器仍保留这些数据。总体而言，如果你想存储一些暂时的或敏感的内容，并希望在关闭浏览器后销毁它们，会话存储是一个不错的选择。

在两种情况下，存储空间都是特定于给定起源的。

## 获取和设置项

Web Storage 只能存储字符串值。每个值都有一个键，可以用来查找它。API 很简单：

`getItem(key)`

返回与键绑定的字符串，如果键不存在则返回`null`。

`setItem(key, value)`

在给定键下存储一个字符串值。如果键已存在，你将覆盖它。

`clear()`

删除当前起源的所有存储数据。

## 缺点

Web Storage 非常有用，但也有一些缺点：

数据存储限制

Web Storage 只能存储字符串数据。你可以存储简单对象，但不能直接存储 —— 需要先将其转换为 JavaScript 对象表示法（JSON）字符串。

大小限制

每个起源对于存储都有限定的空间。在大多数浏览器中，这是 5 兆字节。如果一个起源的存储空间已满，如果尝试添加更多数据，浏览器将抛出异常。

安全问题

即使浏览器分别存储每个起源的数据，它仍然容易受到跨站点脚本（XSS）攻击的影响。攻击者可以通过 XSS 攻击注入代码，窃取本地持久化数据。注意存储敏感数据时的风险。

###### 注意

本章中的所有示例都使用*本地*存储，但同样适用于*会话*存储，因为这两种对象都实现了相同的`Storage`接口。

# 检查 Web Storage 支持

## 问题

你想在使用本地存储之前检查其是否可用，以避免应用程序崩溃。你还希望处理本地存储可用但被用户设置阻止的情况。

## 解决方案

检查全局 `window` 对象的 `localStorage` 属性，以验证浏览器是否支持本地存储。如果检查通过，则本地存储可用（见 示例 2-1）。

##### 示例 2-1。检查本地存储是否可用

```
/**
 * Determines if local storage is available.
 * @returns true if the browser can use local storage, false if not
 */
function isLocalStorageAvailable() {
  try {
    // Local storage is available if the property exists.
    return typeof window.localStorage !== 'undefined';
  } catch (error) {
    // If window.localStorage exists but the user is blocking local
    // storage, the attempt to read the property throws an exception.
    // If this happens, consider local storage not available.
    return false;
  }
}
```

## 讨论

示例 2-1 中的函数处理了两种情况：如果本地存储根本不支持，以及如果它存在且没有被用户设置阻止。

它检查`window.localStorage`属性是否不是`undefined`。如果这个检查通过，这意味着浏览器支持本地存储。如果用户*阻止*了本地存储，仅仅引用`window.localStorage`属性就会抛出一个异常，错误信息显示为访问被拒绝。

通过用`try`/`catch`块包围属性检查，你还可以处理这种情况。当捕获到异常时，会考虑到本地存储不可用并返回`false`。

# 持久化字符串数据

## 问题

你想要将一个字符串值持久化到本地存储中，并在稍后读取它。

## 解决方案

使用`localStorage.getItem`和`localStorage.setItem`来读取和写入数据。示例 2-2 展示了如何使用本地存储来记住颜色选择器的值。

##### 示例 2-2\. 将数据持久化到本地存储

```
// A reference to the color picker input element
const colorPicker = document.querySelector('#colorPicker');

// Load the saved color, if any, and set it on the color picker.
const storedValue = localStorage.getItem('savedColor');
if (storedValue) {
  console.log('Found saved color:', storedValue);
  colorPicker.value = storedValue;
}

// Update the saved color whenever the value changes.
colorPicker.addEventListener('change', event => {
  localStorage.setItem('savedColor', event.target.value);
  console.log('Saving new color:', colorPicker.value);
});
```

## 讨论

当页面首次加载时，会检查本地存储是否已经保存了以前的颜色。如果使用一个不存在的键调用`getItem`，它会返回`null`。只有当返回值不为 null 或空时，才会在颜色选择器中设置该值。

当颜色选择器的值发生变化时，事件处理程序将新值保存到本地存储中。如果已经有一个保存的颜色，它会被覆盖。

# 持久化简单对象

## 问题

你有一个 JavaScript 对象，比如用户配置文件，你想要将其持久化到本地存储中。由于本地存储仅支持字符串值，因此你不能直接这样做。

## 解决方案

使用`JSON.stringify`将对象转换为 JSON 字符串后再保存它。在稍后加载该值时，使用`JSON.parse`将其转回对象，如示例 2-3 所示。

##### 示例 2-3\. 使用`JSON.parse`和`JSON.stringify`

```
/**
 * Given a user profile object, serialize it to JSON and store it in local storage.
 * @param userProfile the profile object to save
 */
function saveProfile(userProfile) {
  localStorage.setItem('userProfile', JSON.stringify(userProfile));
}

/**
 * Loads the user profile from local storage and deserializes the JSON back to
 * an object. If there is no stored profile, an empty object is returned.
 * @returns the stored user profile or an empty object.
 */
function loadProfile() {
  // If there is no stored userProfile value, this will return null. In this case,
  // use the default value of an empty object.
  return JSON.parse(localStorage.getItem('userProfile')) || {};
}
```

## 讨论

直接将配置文件对象传递给`localStorage.setItem`不会产生预期的效果，如示例 2-4 所示。

##### 示例 2-4\. 尝试持久化一个数组

```
const userProfile = {
  firstName: 'Ava',
  lastName: 'Johnson'
};

localStorage.setItem('userProfile', userProfile);

// Prints [object Object]
console.log(localStorage.getItem('userProfile'));
```

保存的值是`[object Object]`。这是对配置文件对象调用`toString`的结果。

`JSON.stringify`接受一个对象，并返回表示该对象的 JSON 字符串。将用户配置文件对象传递给`JSON.stringify`将得到这个 JSON 字符串（为了可读性添加了空格）：

```
{
  "firstName": "Ava",
  "lastName": "Johnson"
}
```

这种方法适用于像用户配置文件这样的对象，但是[JSON 规范](https://www.json.org)限制了可以序列化为字符串的内容。一般来说，这些是对象、数组、字符串、数字、布尔值和`null`。其他值，比如类实例或函数，不能以这种方式序列化。

# 持久化复杂对象

## 问题

你想要将一个无法直接序列化为 JSON 字符串的对象，例如用户配置文件中可能包含一个指定最后更新时间的`Date`对象，持久化到本地存储中。

## 解决方案

使用 `JSON.stringify` 和 `JSON.parse` 的 `replacer` 和 `reviver` 函数，为复杂数据提供自定义序列化。

考虑以下配置文件对象：

```
const userProfile = {
  firstName: 'Ava',
  lastName: 'Johnson',

  // This date represents June 2, 2025.
  // Months start with zero but days start with 1.
  lastUpdated: new Date(2025, 5, 2);
}
```

如果你使用 `JSON.stringify` 序列化这个对象，生成的字符串将以 ISO 日期字符串形式包含 `lastUpdated` 日期（参见 示例 2-5）。

##### 示例 2-5\. 尝试序列化带有 `Date` 对象的对象

```
const json = JSON.stringify(userProfile);
```

生成的 JSON 字符串如下所示：

```
{
  "firstName": "Ava",
  "lastName": "Johnson",
  "lastUpdated": '2025-06-02T04:00:00.000Z'
}
```

现在你有一个可以保存到本地存储的 JSON 字符串。然而，如果你使用这个 JSON 字符串调用 `JSON.parse`，生成的对象与原始对象略有不同。`lastUpdated` 属性仍然是一个字符串，而不是一个 `Date` 对象，因为 `JSON.parse` 不知道这应该是一个 `Date` 对象。

为了处理这些情况，`JSON.stringify` 和 `JSON.parse` 接受名为 `replacer` 和 `reviver` 的特殊函数。这些函数提供自定义逻辑，用于将非基本类型值转换为 JSON 以及从 JSON 转换回来。

### 使用 `replacer` 函数进行序列化

`JSON.stringify` 的 `replacer` 参数可以以几种不同的方式工作。MDN 提供了关于 `replacer` 函数的详细 [文档](https://oreil.ly/H56TM)。

`replacer` 函数接受两个参数：`key` 和 `value`（参见 示例 2-6）。`JSON.stringify` 首先以空字符串作为键，被字符串化的对象作为值调用此函数。你可以在这里通过调用 `getTime()` 将 `lastUpdated` 字段转换为 `Date` 对象的可序列化表示，`getTime()` 返回自纪元时（1970 年 1 月 1 日 UTC 午夜）以来的毫秒数。

##### 示例 2-6\. `replacer` 函数

```
function replacer(key, value) {
  if (key === '') {
    // First replacer call, "value" is the object itself.
    // Return all properties of the object, but transform lastUpdated.
    // This uses object spread syntax to make a copy of "value" before
    // adding the lastUpdated property.
    return {
      ...value,
      lastUpdated: value.lastUpdated.getTime()
    };
  }

  // After the initial transformation, the replacer is called once
  // for each key/value pair.
  // No more replacements are necessary, so return these as is.
  return value;
}
```

你可以将这个 `replacer` 函数传递给 `JSON.stringify`，将对象序列化为 JSON，如 示例 2-7 所示。

##### 示例 2-7\. 使用 `replacer` 进行字符串化

```
const json = JSON.stringify(userProfile, replacer);
```

这将生成以下 JSON 字符串：

```
{
  "firstName": "Ava",
  "lastName": "Johnson",
  "lastUpdated": 1748836800000
}
```

`lastUpdated` 属性中的数字是 2025 年 6 月 2 日的时间戳。

### 使用 `reviver` 函数进行反序列化

当你将这个 JSON 字符串传递给 `JSON.parse` 时，`lastUpdated` 属性仍然保持为一个数字（时间戳）。你可以使用 `reviver` 函数将这个序列化的数字值转换回一个 `Date` 对象。

`JSON.parse` 为 JSON 字符串中的每个属性调用 `reviver` 函数。对于每个键，函数返回的值是设置在最终对象中的值（参见 示例 2-8）。

##### 示例 2-8\. `reviver` 函数

```
function reviver(key, value) {
  // JSON.parse calls the reviver once for each key/value pair.
  // Watch for the lastUpdated key.
  // Only proceed if there's actually a value for lastUpdated.
  if (key === 'lastUpdated' && value) {
    // Here, the value is the timestamp. You can pass this to the Date constructor
    // to create a Date object referring to the proper time.
    return new Date(value);
  }

  // Restore all other values as is.
  return value;
}
```

要使用 `reviver`，将其作为第二个参数传递给 `JSON.parse`，如 示例 2-9 所示。

##### 示例 2-9\. 使用 `reviver` 进行解析

```
const object = JSON.parse(userProfile, reviver);
```

这将返回一个与我们最初的用户配置文件对象相等的对象：

```
{
  firstName: 'Ava',
  lastName: 'Johnson',
  lastUpdated: [Date object representing June 2, 2025]
}
```

## 讨论

通过这种可靠的方法将对象转换为 JSON，并保持 `Date` 属性完整，你可以将这些值持久化到本地存储中。

此处展示的方法只是使用 `replacer` 函数处理的一种方式。除了 `replacer` 函数外，还可以在要序列化为字符串的对象上定义一个 `toJSON` 函数。结合工厂函数，就不需要 `replacer` 函数。

##### 示例 2-10\. 使用添加了 `toJSON` 函数的工厂

```
/**
 * A factory function to create a user profile object,
 * with the lastUpdated property set to today and a toJSON method
 *
 * @param firstName The user's first name
 * @param lastName The user's last name
 */
function createUser(firstName, lastName) {
  return {
    firstName,
    lastName,
    lastUpdated: new Date(),
    toJSON() {
      return {
        firstName: this.firstName,
        lastName: this.lastName,
        lastUpdated: this.lastUpdated.getTime();
      }
    }
  }
}

const userProfile = createUser('Ava', 'Johnson');
```

使用 示例 2-10 中的对象调用 `JSON.stringify` 返回与之前相同的 JSON 字符串，`lastUpdated` 已适当转换为时间戳。

###### 注意

对于使用 `JSON.parse` 解析字符串为对象的操作，没有类似 `toJSON` 的机制。如果使用此处展示的 `toJSON` 方法，仍然需要编写一个 `reviver` 函数，以正确反序列化用户配置文件字符串。

由于函数无法序列化，因此生成的 JSON 字符串不会有 `toJSON` 属性。无论你选择哪种方法，生成的 JSON 都是相同的。

# 监听存储更改

## 问题

当同源的另一个标签页更改本地存储时，你希望收到通知。

## 解决方案

在 `window` 对象上监听 `storage` 事件。当同一来源的其他标签页或会话对本地存储中的任何数据进行更改时，此事件会触发（参见 示例 2-11）。

##### 示例 2-11\. 监听来自另一个标签页的存储更改

```
// Listen for the 'storage' event. If another tab changes the
// 'savedColor' item, update this page's color picker with the new value.
window.addEventListener('storage', event => {
  if (event.key === 'savedColor') {
    console.log('New color was chosen in another tab:', event.newValue);
    colorPicker.value = event.newValue;
  }
});
```

考虑来自“持久化字符串数据”的持久性颜色选择器。如果用户同时打开了多个标签页，并在另一个标签页中更改颜色，您可以收到通知并更新本地内存中的数据副本，以保持所有内容同步。

###### 注意

`storage` 事件不会在进行存储更改的标签页或页面上触发。它用于监听其他页面对本地存储所做的更改。

`storage` 事件指定了哪个键发生了更改以及新值是什么。它还包括旧值，以便进行比较。

## 讨论

`storage` 事件的主要用例是实时保持多个会话之间的同步。

###### 注意

`storage` 事件仅在同一设备上的同一浏览器中的其他标签页和会话中触发。

即使你不监听 `storage` 事件，同一来源的所有会话仍然共享同一本地存储数据。如果在任何时候调用 `localStorage.getItem`，你仍将获得最新的值。`storage` 事件只是在此类更改发生时提供实时通知，以便应用程序可以更新本地数据。

# 查找所有已知键

## 问题

你想知道当前原点的本地存储中目前存在的所有键。

## 解决方案

使用 `length` 属性与 `key` 函数构建所有已知键的列表。`Storage` 对象没有直接返回键列表的函数，但可以通过以下方式构建这样的列表：

+   `length` 属性返回键的数量。

+   给定索引，`key` 函数返回该索引处的键。

您可以结合 `for` 循环使用 示例 2-12 中显示的方法，构建所有键的数组。

##### 示例 2-12\. 构建键列表

```
/**
 * Generates an array of all keys found in the local storage area
 * @returns an array of keys
 */
function getAllKeys() {
  const keys = [];

  for (let i = 0; i < localStorage.length; i++) {
    keys.push(localStorage.key(i));
  }

  return keys;
}
```

## 讨论

您还可以结合 `length` 属性和 `key` 函数执行其他类型的查询。例如，这可能是一个函数，它接受一个键的数组并返回一个包含这些键/值对的对象（参见 示例 2-13）。

##### 示例 2-13\. 查询一组键/值对的子集

```
function getAll(keys) {
  const results = {};

  // Check each key in local storage.
  for (let i = 0; i < localStorage.length; i++) {

    // Get the ith key. If the keys array includes this key, add it and its value
    // to the results object.
    const key = localStorage.key(i);
    if (keys.includes(key)) {
      results[key] = localStorage.getItem(key);
    }
  }

  // results now has all key/value pairs that exist in local storage.
  return results;
}
```

###### 注意

使用 `key` 函数引用的键的排序在不同浏览器中可能不同。

# 移除数据

## 问题

您想从本地存储中删除一些或全部数据。

## 解决方案

根据需要使用 `removeItem` 和 `clear` 方法。

要从本地存储中删除特定键值对，请调用 `localStorage.remove​I⁠tem` 并传入键（参见 示例 2-14）。

##### 示例 2-14\. 从本地存储中移除一个项目

```
// This is a safe operation. If the key doesn't exist,
// no exception is thrown.
localStorage.removeItem('my-key');
```

调用 `localStorage.clear` 来移除当前来源（origin）的本地存储中的 *所有* 数据，如 示例 2-15 中所示。

##### 示例 2-15\. 从本地存储中移除所有项目

```
localStorage.clear();
```

## 讨论

浏览器限制可以存储在 Web 存储中的数据量。通常限制约为 5 MB。为了避免空间不足并引发错误，应在不再需要时删除项目。根据您使用 Web 存储的情况，您还可以提供一种方法让用户清除存储的数据。考虑一个表情选择器，它在本地存储中存储最近选择的表情。您可以添加一个“清除最近使用”按钮，以删除这些项目。
