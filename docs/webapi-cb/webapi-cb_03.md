# 第三章：URL 和路由

# 简介

大多数网页和应用程序都以某种方式处理 URL。这可能是像使用特定查询参数创建链接，或在单页应用程序（SPA）中基于 URL 进行路由等操作。

URL 只是遵循[RFC 3986, “统一资源标识符（URI）：通用语法”](https://oreil.ly/SUziR)中定义的一些语法规则的字符串。URL 有几个组成部分，您可能需要解析或操作这些部分。使用正则表达式或字符串连接等技术并不总是可靠的。

今天，浏览器支持 URL API。此 API 提供了一个`URL`构造函数，可以创建、派生和操作 URL。最初此 API 功能有限，但后来的更新添加了类似`URLSearchParams`接口的实用工具，简化了构建和读取查询字符串的过程。

## URL 的各个组成部分

当您使用表示有效 URL 的字符串调用`URL`构造函数时，生成的对象包含表示 URL 不同组成部分的属性。 图 3-1 显示了其中最常用的部分：

`protocol`（1）

对于 Web URL，通常为`http:`或`https:`（请注意包括冒号，但不包括斜杠）。其他协议也可能存在，如`file:`（用于本地文件，而不是服务器上托管的文件）或`ftp:`（FTP 服务器上的资源）。

`hostname`（2）

域名或主机名（`example.com`）。

`pathname`（3）

相对于根目录的资源路径，以斜杠开头（`/admin/login`）。

`search`（4）

任何查询参数。包括`?`字符（`?username=sysadmin`）。

![带有其组成部分突出显示的示例 URL](img/wacb_0301.png)

###### 图 3-1\. 带有其组成部分突出显示的示例 URL

`URL`的其他一些部分包括：

`hash`

如果 URL 包含散列标识符，则返回散列部分（包括散列符号`#`）。这有时用于较旧的 SPA 的内部导航。对于 URL *https://example.com/app#profile*，`hash`的值将是`#profile`。

`host`

类似于`hostname`，但也包括端口号（如果指定），例如`localhost:8443`。

`origin`

URL 的来源。这通常包括协议、主机名和端口（如果指定）。

您可以通过调用其`toString`方法或访问其`href`属性来获取整个 URL 字符串。

如果将无效的 URL 字符串传递给`URL`构造函数，则会抛出异常。

# 解析相对 URL

## 问题

如果您有一个部分或相对 URL，例如`/api/users`，您想要将其解析为完整的绝对 URL，如*https://example.com/api/users*。

## 解决方案

创建`URL`对象，传递相对 URL 和所需的基本 URL，如示例 3-1 所示。

##### 示例 3-1\. 创建相对 URL

```
/**
 * Given a relative path and a base URL, resolves a full absolute URL.
 * @param relativePath The relative path for the URL
 * @param baseUrl A valid URL to use as the base
 */
function resolveUrl(relativePath, baseUrl) {
  return new URL(relativePath, baseUrl).href;
}

// https://example.com/api/users
console.log(resolveUrl('/api/users', 'https://example.com'));
```

没有第二个参数，`URL`构造函数会因为`/api/users`不是有效的 URL 而抛出错误。第二个参数是构造新 URL 的基础。它通过假定给定的路径相对于基本 URL 来构造 URL。

## 讨论

第二个参数必须是有效的 URL。根据第一个参数，应用有效相对 URL 的典型规则来构造最终 URL。

如果第一个参数以斜杠开头，则忽略基本 URL 的路径名，并且新 URL 相对于基本 URL 的根目录：

```
// https://example.com/api/v1/users
console.log(resolveUrl('/api/v1/users', 'https://example.com'));

// https://example.com/api/v1/users
// Note that /api/v2 is discarded due to the leading slash in /api/v1/users
console.log(resolveUrl('/api/v1/users', 'https://example.com/api/v2'));
```

否则，计算相对于基本 URL 的 URL：

```
// https://example.com/api/v1/users
console.log(resolveUrl('../v1/users/', 'https://example.com/api/v2'));

// https://example.com/api/v1/users
console.log(resolveUrl('users', 'https://example.com/api/v1/groups'));
```

如果第一个参数本身就是有效的 URL，则忽略基本 URL。

如果构造函数的第二个参数不是字符串，则在其上调用`toString`，并使用生成的字符串。这意味着你可以传递其他类似于`URL`的`URL`对象，甚至其他类似的对象。你甚至可以传递`window.location`（一个`Location`对象，其属性类似于`URL`）以生成当前来源（参见示例 3-2）的新 URL。

##### 示例 3-2\. 在相同来源上创建相对 URL

```
const usersApiUrl = new URL('/api/users', window.location);
```

# 从 URL 中删除查询参数

## 问题

您希望从 URL 中删除所有查询参数。

## 解决方案

创建一个`URL`对象并将其`search`属性设置为空字符串，如示例 3-3 中所示。

##### 示例 3-3\. 删除 URL 的查询参数

```
/**
 * Removes all parameters from an input URL.
 *
 * @param inputUrl a URL string containing query parameters
 * @returns a new URL string with all query parameters removed
 */
function removeAllQueryParameters(inputUrl) {
  const url = new URL(inputUrl);
  url.search = '';
  return url.toString();
}

// Results in 'https://example.com/api/users'
removeAllQueryParams('https://example.com/api/users?user=sysadmin&q=user');
```

## 讨论

URL 中的查询参数以两种方式表示：使用`search`属性和`searchParams`属性。

`search`属性是一个单一字符串，包含所有查询参数以及前导的`?`字符。如果要删除整个查询字符串，可以将其设置为空字符串。

注意`search`属性被设置为空字符串。如果设置为`null`，则在查询字符串中得到字面字符串`null`（参见示例 3-4）。

##### 示例 3-4\. 错误地尝试删除所有查询参数

```
const url = new URL('https://example.com/api/users?user=sysadmin&q=user');

url.search = null;
console.log(url.toString()); // https://example.com/api/users?null
```

`searchParams`属性是一个`URLSearchParams`对象。它具有查看、添加和删除查询参数的方法。在添加查询参数时，它会自动处理编码字符。如果您只想删除一个查询参数，可以在该对象上调用`delete`，如示例 3-5 所示。

##### 示例 3-5\. 删除单个查询参数

```
/**
 * Removes a single parameter from an input URL
 *
 * @param inputUrl a URL string containing query parameters
 * @param paramName the name of the parameter to remove
 * @returns a new URL string with the given query parameter removed
 */
function removeQueryParameter(inputUrl, paramName) {
  const url = new URL(inputUrl);
  url.searchParams.delete(paramName);
  return url.toString();
}

console.log(
  removeQueryParameter(
    'https://example.com/api/users?user=sysadmin&q=user',
    'q'
  )
); // https://example.com/api/users?user=sysadmin
```

# 向 URL 添加查询参数

## 问题

您有一个现有的 URL，它可能已经包含一些查询参数，并且您想要添加额外的查询参数。

## 解决方案

使用`URLSearchParams`对象，可通过`searchParams`属性访问，以添加额外的参数（参见示例 3-6）。

##### 示例 3-6\. 添加额外的查询参数

```
const url = new URL('https://example.com/api/search?objectType=user');

url.searchParams.append('userRole', 'admin');
url.searchParams.append('userRole', 'user');
url.searchParams.append('name', 'luke');

// Prints
"https://example.com/api/search?objectType=user&userRole=admin&userRole=user
&name=luke"
console.log(url.toString());
```

## 讨论

此 URL 已经有一个查询参数（`objectType=user`）。代码使用解析后的 URL 的`searchParams`属性来追加更多查询参数。添加了两个`userRole`参数。使用`append`时，会添加新值并保留现有值。要替换所有同名参数的值为新值，可以使用`set`。

带有新参数的完整 URL 现在是：

```
https://example.com/api/search?objectType=user&userRole=admin&userRole=user
&name=luke
```

如果调用`append`时参数名存在但没有值，则会引发异常，如示例 3-7 所示。

##### 示例 3-7\. 尝试调用`append`而没有提供值

```
const url = new URL('https://example.com/api/search?objectType=user');

// TypeError: Failed to execute 'append' on 'URLSearchParams':
// 2 arguments required, but only 1 present.
url.searchParams.append('name');
```

该方法优雅地处理其他参数类型。如果未收到字符串值，则将该值转换为字符串（参见示例 3-8）。

##### 示例 3-8\. 添加非字符串参数

```
const url = new URL('https://example.com/api/search?objectType=user');

// The resulting URL has the query string:
//  ?objectType=user&name=null&role=undefined
url.searchParams.append('name', null);
url.searchParams.append('role', undefined);
```

使用`URLSearchParams`自动添加查询参数会处理任何潜在的编码问题。如果要添加具有保留字符（如 RFC 3986 定义的）的参数，例如`&`或`?`，`URLSearchParams`会自动对这些字符进行编码，以确保有效的 URL。它使用*百分比编码*，即添加一个百分号后跟表示该字符的十六进制数字。例如，`&`变成`%26`，因为`0x26`是和号的十六进制代码。

您可以通过将包含一些保留字符的查询参数追加在一起来查看此编码的实际效果，如示例 3-9 所示：

##### 示例 3-9\. 在查询参数中编码保留字符

```
const url = new URL('https://example.com/api/search');

// Contrived example string demonstrating several reserved characters
url.searchParams.append('q', 'admin&user?luke');
```

最终的 URL 变为：

```
https://example.com/api/search?q=admin%26user%3Fluke
```

URL 中包含`%26`代替`&`，以及`%3F`代替`?`。这些字符在 URL 中具有特殊含义。`?`表示查询字符串的开始，`&`是参数之间的分隔符。

如示例 3-6 所示，多次使用相同键调用`append`会添加具有给定键的新查询参数。当您调用`.append('userRole', 'user')`时，它会添加参数`userRole=user`并保留先前的`userRole=admin`。`URLSearchParams`还有一个`set`方法。`set`也会添加查询参数，但行为不同。`set`会用新参数替换给定键下的所有现有参数（参见示例 3-10）。如果您再次使用`set`构造相同的 URL，则结果将不同。

##### 示例 3-10\. 使用`set`添加查询参数

```
const url = new URL('https://example.com/api/search?objectType=user');

url.searchParams.set('userRole', 'admin');
url.searchParams.set('userRole', 'user');
url.searchParams.set('name', 'luke');
```

使用`set`而不是`append`时，第二个`userRole`参数将覆盖第一个，最终的 URL 为：

```
https://example.com/api/search?objectType=user&userRole=user&name=luke
```

注意只有一个`userRole`参数——最后添加的那个。

# 读取查询参数

## 问题

您想解析并列出 URL 中的查询参数。

## 解决方案

使用`URLSearchParams`的`forEach`方法列出键和值（参见示例 3-11）。

##### 示例 3-11\. 读取查询参数

```
/**
 * Takes a URL and returns an array of its query parameters
 *
 * @param inputUrl A URL string
 * @returns An array of objects with key and value properties
 */
function getQueryParameters(inputUrl) {
  // Can't use an object here because there may be multiple
  // parameters with the same key, and we want to return all parameters.
  const result = [];

  const url = new URL(inputUrl);

  // Add each key/value pair to the result array.
  url.searchParams.forEach((value, key) => {
    result.push({ key, value });
  });

  // Results are ready!
  return result;
}
```

## 讨论

当列出 URL 上的查询参数时，任何百分号编码的保留字符都会解码回它们的原始值（参见示例 3-12）。

##### 示例 3-12\. 使用`getQueryParameters`函数

```
getQueryParameters('https://example.com/api/search?name=luke%26ben'); ![1](img/1.png)
```

![1](img/#co_urls_and_routing_CO1-1)

`name` 参数包含一个百分号编码的和字符 (`%26`)。

此代码打印具有原始未编码值的参数 `name=luke%26ben`：

```
name: luke&ben
```

`forEach` 遍历每个唯一的键/值对组合。即使 URL 具有多个具有相同键的查询参数，这也会分别打印每个唯一的键/值对。

# 创建一个简单的客户端路由器

## 问题

您有一个单页应用程序，并希望添加客户端路由。这使用户能够在不进行新网络请求的情况下在不同的 URL 之间导航，并在客户端上替换内容。

## 解决方案

使用 `history.pushState` 和 `popstate` 事件来实现一个简单的路由器。这个简单的路由器在 URL 匹配到已知路由时呈现模板的内容（参见示例 3-13）。

##### 示例 3-13\. 一个简单的客户端路由器

```
// Route definitions. Each route has a path and some content to render.
const routes = [
  { path: '/', content: '<h1>Home</h1>' },
  { path: '/about', content: '<h1>About</h1>' }
];

function navigate(path, pushState = true) {
  // Find the matching route and render its content.
  const route = this.routes.find(route => route.path === path);

  // Be careful using innerHTML in a real app, which can be a security risk.
  document.querySelector('#main').innerHTML = route.content;

  if (pushState) {
    // Change the URL to match the new route.
    history.pushState({}, '', path);
  }
}
```

有了这个路由器，您可以添加链接：

```
<a href="/">Home</a>
<a href="/about">About</a>
```

当您单击这些链接时，浏览器会尝试导航到新页面，向服务器发出请求。这可能导致 404 错误，这并不是您想要的。要使用客户端路由器，您需要拦截点击事件，并与来自示例 3-13 的路由器集成，如示例 3-14 所示。

##### 示例 3-14\. 添加点击处理程序以路由链接

```
document.querySelectorAll('a').forEach(link => {
  link.addEventListener('click', event => {
    // Prevent the browser from trying to load the new URL from the server!
    event.preventDefault();
    navigate(link.getAttribute('href'));
  });
});
```

当您单击这些链接之一时，`preventDefault` 调用会阻止浏览器的默认行为（执行完整的页面导航）。相反，它获取 `href` 属性并将其传递给客户端路由器。如果找到匹配的路由，它会呈现该路由的内容。

要使其成为一个完整的解决方案，还有一个必要的部分。如果您单击其中一个客户端路由，然后单击浏览器的后退按钮，什么也不会发生。这是因为页面实际上没有导航，而只是从路由器中弹出先前的状态。为了处理这种情况，您还需要监听浏览器的 `popstate` 事件，并渲染正确的内容，如示例 3-15 所示。

##### 示例 3-15\. 监听 `popstate` 事件

```
window.addEventListener('popstate', () => {
  navigate(window.location.pathname, false);
});
```

当用户单击后退按钮时，浏览器会触发 `popstate` 事件。这会将页面 URL 变回，并且您只需要查找匹配 URL 的路由的内容。在这种情况下，您不希望调用 `pushState`，因为这会添加一个新的历史状态，这可能并不是您想要的，因为您刚刚从堆栈中弹出了旧的历史状态。

## 讨论

此客户端路由器正在运行，但存在一个问题。如果您单击“关于”链接，然后单击刷新按钮，浏览器将发起新的网络请求，这可能导致 404 错误。要解决此问题，服务器需要配置为返回主 HTML 和 JavaScript 内容，而不管 URL 的路径名是什么。这将加载路由器代码，并使用`window.location.pathname`的值调用。如果一切配置正确，客户端路由处理程序将执行并呈现正确的内容。

当使用客户端路由时，页面之间的导航可能更快，因为不需要与服务器进行往返。这使得导航更加流畅和响应更快。当然也有缺点。为了支持快速页面过渡，通常需要预先加载大量额外的 JavaScript，因此初始页面加载可能会较慢。

# 匹配 URL 与模式

## 问题

您希望定义一组有效 URL 的模式，可以将 URL 与之匹配。您可能还希望提取 URL 路径的一部分。例如，给定 URL *https://example.com/api/users/123/profile*，您希望获取用户 ID（`123`）。

## 解决方案

使用 URL 模式 API 定义预期模式并提取所需部分。

###### 注意

该 API 可能尚未得到所有浏览器的支持。请参阅[CanIUse](https://oreil.ly/Eb-k2)以获取最新的兼容性数据。

使用此 API，您可以创建一个`URLPattern`对象，该对象定义了一个可以用来匹配 URL 的模式（参见 Example 3-16）。它是使用定义要匹配的模式的字符串创建的。该字符串可以包含命名组，当与 URL 字符串匹配时，将会提取这些组。您可以按索引访问提取的值。这些组类似于正则表达式中的捕获组。

##### Example 3-16\. 创建`URLPattern`

```
const profilePattern = new URLPattern({ pathname: '/api/users/:userId/profile' });
```

Example 3-16 展示了一个带有单个命名组`userId`的简单 URL 模式。该组名称之前有一个冒号字符。您可以使用此模式对象匹配 URL，并从中提取用户 ID。Example 3-17 探讨了一些不同的 URL 及其如何使用`profilePattern`的`test`方法进行测试。

##### Example 3-17\. 测试模式对应的 URL

```
// The pattern won't match a pathname alone; it must be a valid URL.
console.log(profilePattern.test('/api/users/123/profile'));

// This URL matches because the pathname matches the pattern.
console.log(profilePattern.test('https://example.com/api/users/123/profile'));

// It also matches URL objects.
console.log(profilePattern.test(new URL
('https://example.com/api/users/123/profile')));

// The pathname must match exactly, so this won't match.
console.log(profilePattern.test('https://example.com/v1/api/users/123/profile'));
```

`profilePattern`指定了精确的路径名匹配，这就是为什么 Example 3-17 中的最后一个示例未能工作的原因。您可以定义一个不那么严格的版本，使用通配符（`*`）来匹配部分路径名。

##### Example 3-18\. 使用模式中的通配符

```
const wildcardProfilePattern = new URLPattern
({ pathname: '/*/api/users/:userId/profile' });

// This matches now because the /v1 portion of the URL matches the wildcard.
console.log(wildcardProfilePattern.test
('https://example.com/v1/api/users/123/profile'));
```

您可以使用模式的`exec`方法获取有关匹配的更多数据。如果模式与 URL 匹配，`exec`将返回一个包含 URL 各部分匹配的对象。每个嵌套对象都有一个`input`属性，指示 URL 的哪个部分匹配，并且有一个`groups`属性，其中包含模式中定义的任何命名组。

您可以使用`exec`从匹配的 URL 中提取用户 ID，详见 Example 3-19。

##### 示例 3-19\. 提取用户 ID

```
const profilePattern = new URLPattern({ pathname: '/api/users/:userId/profile' });

const match = profilePattern.exec('https://example.com/api/users/123/profile');
console.log(match.pathname.input); // '/api/users/123/profile'
console.log(match.pathname.groups.userId); // '123'
```

## 讨论

虽然它目前尚未获得所有浏览器的完全支持，但这是一个非常灵活的 API。您可以为 URL 的任何部分定义模式，匹配输入并提取组。
