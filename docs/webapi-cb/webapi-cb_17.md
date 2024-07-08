# 第十七章：CSS

# 介绍

在现代浏览器环境中，CSS 不仅允许您编写样式规则，还具有一组可以用来进一步增强应用程序的 API。

CSS 对象模型（CSSOM）允许您从 JavaScript 代码中以编程方式设置内联样式。不仅如此，您甚至可以在运行时更改 CSS 变量的值。

在 第八章 中，您看到了一个示例，使用 `window.matchMedia` 来程序化地检查媒体查询是否在当前页面上匹配。

本章包含一些使用这些 CSS 相关 API 的实用示例。在撰写时，一些这些 API 的浏览器支持并不理想。始终在使用之前检查浏览器兼容性。

# 文本范围的突出显示

## 问题

您想要在文档中的一段文本范围上应用突出显示效果。

## 解决方案

围绕所需文本创建一个 `Range` 对象，然后使用 CSS 自定义高亮 API 将高亮样式应用于该范围。

第一步是创建一个 `Range` 对象。此对象表示文档中的文本区域。示例 17-1 展示了一个通用的实用函数，用于根据文本节点和要突出显示的文本创建范围。

##### 示例 17-1\. 创建一个范围

```
/**
 * Given a text node and a substring to highlight, creates a Range object covering
 * the desired text.
 */
function getRange(textNode, textToHighlight) {
  const startOffset = textNode.textContent.indexOf(textToHighlight);
  const endOffset = startOffset + textToHighlight.length;

  // Create a Range for the text to highlight.
  const range = new Range();
  range.setStart(textNode, startOffset);
  range.setEnd(textNode, endOffset);

  return range;
}
```

###### 注意

这个 API 可能尚未被所有浏览器支持。请参阅 [CanIUse](https://oreil.ly/wDJWH) 获取最新的兼容性数据。

假设您有在 示例 17-2 中显示的 HTML 元素。

##### 示例 17-2\. 一些 HTML 标记

```
<p id="text">
  This is some text. We're using the CSS Custom Highlight API to highlight some of
  the text.
</p>
```

如果您想要突出显示文本“highlight some of the text”，您可以使用 `getRange` 助手来创建围绕该文本的 `Range`（参见 示例 17-3）。

##### 示例 17-3\. 使用 `getRange` 助手

```
const node = document.querySelector('#text');
const range = getRange(node.firstChild, 'highlight some of the text');
```

现在您有了范围，需要向浏览器的高亮注册表注册一个新的高亮效果。通过使用该范围创建一个新的 `Highlight` 对象，然后将该 `Highlight` 对象传递给 `CSS.highlights.set` 函数（参见 示例 17-4）。

##### 示例 17-4\. 注册高亮

```
const highlight = new Highlight(range);
CSS.highlights.set('highlight-range', highlight);
```

这注册了高亮，但默认情况下没有视觉效果。接下来，您需要创建一些 CSS 样式，以便应用于高亮。通过使用 `::highlight` 伪元素完成此操作。您可以将此伪元素与在 示例 17-4 中注册高亮的关键字组合使用（参见 示例 17-5）。

##### 示例 17-5\. 设置高亮样式

```
::highlight(highlight-range) {
  background-color: #fef3c7;
}
```

应用此样式后，范围内的文本现在以浅琥珀色突出显示。

## 讨论

您还可以使用 `<mark>` 元素来突出显示内容。示例 17-6 展示了如何使用 `<mark>` 来突出显示某些文本。

##### 示例 17-6\. 使用 mark 元素进行突出显示

```
<p id="text">
  This is some text. We're using the mark element to
  <mark>highlight some of the text</mark>.
</p>
```

这与使用 CSS 自定义高亮 API 具有相同的视觉效果，但关键区别在于使用 `<mark>` 需要将新元素插入到 DOM 中。这可能取决于您添加新元素的位置。

例如，如果您想要高亮的文本跨越多个元素，可能无法仅使用 `<mark>` 元素来实现，同时保持有效的 HTML。考虑 示例 17-7 中的 HTML。

##### 示例 17-7\. 一些需要高亮的标记

```
<p>
  This is a paragraph, which is being highlighted.
</p>

<p>
  The highlight extends to this paragraph. This is not highlighted.
</p>
```

如果您想要高亮显示“正在进行高亮。高亮扩展到这一段落。”，您无法仅使用单个 `<mark>` 元素实现（参见 示例 17-8）。

##### 示例 17-8\. 无效的 HTML

```
<p>
  This is a paragraph, <mark>which is being highlighted.
</p>

<p>
  The highlight extends to this paragraph</mark>. This is not highlighted.
</p>
```

这不是有效的 HTML。解决方案是使用两个单独的 `<mark>` 元素，但这样就不是单个连续高亮区域了。

使用 CSS 自定义高亮 API 可以实现跨多个标签的高亮效果，通过创建跨多个标签的范围并应用高亮效果。

# 防止未样式化文本的闪烁

## 问题

您希望避免在使用 Web 字体时出现未样式化文本的闪烁。

## 解决方案

使用 CSS 字体加载 API 显式加载您希望在应用程序中使用的字体，并延迟渲染任何文本，直到字体加载完成。

要使用此 API 加载字体，首先创建一个包含有关要加载的字体面的 `FontFace` 对象。示例 17-9 使用了 Roboto 字体。

##### 示例 17-9\. 创建 Roboto 字体

```
const roboto = new FontFace(
  'Roboto',
  'url(https://fonts.gstatic.com/s/roboto/v30/KFOmCnqEu92Fr1Mu72xKKTU1Kvnz.woff2)', {
    style: 'normal',
    weight: 400
  });
```

文档具有全局 `fonts` 属性，它是一个 `FontFaceSet`，包含文档中使用的所有字体面。要使用此字体面，您需要将其添加到 `FontFaceSet` 中（参见 示例 17-10）。

##### 示例 17-10\. 将 Roboto 添加到全局 `FontFaceSet`

```
document.fonts.add(roboto);
```

到目前为止，您只是定义了字体，但尚未加载任何内容。您可以通过在 `FontFace` 对象上调用 `load` 来启动加载过程（参见 示例 17-11）。这会返回一个 `Promise`，一旦字体加载完成就会解析。

##### 示例 17-11\. 等待字体加载完成

```
roboto.load()
  .then(() => {
    // Font has been loaded and is ready for use.
  });
```

为了防止未样式化文本的闪烁，您需要隐藏使用该字体的文本，直到字体加载完成。例如，如果您的应用程序显示初始加载动画，则可以继续动画，直到必要的字体加载完成，然后删除加载程序并开始渲染应用程序。

如果您的应用程序使用多个字体，可以等待 `document.fonts.ready` 的 `Promise`。一旦所有字体加载并准备好，该 `Promise` 就会解析。

## 讨论

在使用 CSS 的 Web 字体时，字体是通过 `@font-face` 规则声明的，其中包含要下载的字体文件的 URL。如果在字体加载完成之前渲染文本，则会使用回退系统字体。一旦字体准备就绪，文本会重新以正确的字体进行渲染。这可能会导致不良效果，例如如果字体度量不同，则会出现布局转移。

使用 `@font-face` 的缺点是无法知道字体何时已加载并准备好使用。通过使用 CSS 字体加载 API，您可以更好地控制字体加载，并确切地知道何时可以安全地开始使用特定字体来渲染文本。

如果加载字体时出现错误——例如，可能是字体 URL 打错了——字体的 `load` 方法返回的 `Promise` 将会以错误拒绝。

# 动画化 DOM 过渡

## 问题

你希望在添加或移除 DOM 元素时展示一个动画过渡效果。

## 解决方案

使用视图过渡 API 提供两个状态之间的动画过渡效果。

###### 注意

该 API 可能尚未被所有浏览器支持。请查看 [CanIUse](https://oreil.ly/I8RFN) 获取最新的兼容性数据。

此 API 用于在两个 DOM 状态之间应用过渡效果。要启动视图过渡，请调用 `document.startViewTransition` 函数。此函数将回调函数作为其参数。您需要在此回调函数内执行您的 DOM 更改。

在 示例 17-12 中，假设您有一个单页面应用。该应用的每个视图都是具有唯一 ID 的顶级 HTML 元素。要在视图之间进行路由，可以移除当前视图并添加新视图。

##### 示例 17-12\. 一个简单的视图过渡

```
function showAboutPage() {
  document.startViewTransition(() => {
    document.querySelector('#home-page').style.display = 'none';
    document.querySelector('#about-page').style.display = 'block';
  });
}
```

这应用了一个基本的交叉淡入淡出过渡效果在两个视图之间。

如果要调整交叉淡入淡出过渡的速度，可以通过一些 CSS 来实现，如 示例 17-13 所示。

##### 示例 17-13\. 减缓过渡速度

```
::view-transition-old(root),
::view-transition-new(root) {
  animation-duration: 2s;
}
```

## 讨论

视图过渡效果通过有效地捕获当前 DOM 状态的屏幕截图来工作。一旦回调内进行的 DOM 更改完成，另一个截图就会被捕获。浏览器在页面上创建一些伪元素，并在它们之间应用动画过渡。

创建的伪元素包括：

`::view-transition`

一个包含所有视图过渡的顶级覆盖层

`::view-transition-group(<name>)`

单个视图过渡

`::view-transition-image-pair(<name>)`

包含正在过渡的两个图像

`::view-transition-old(<name>)`

旧 DOM 状态的图像

`::view-transition-new(<name>)`

新 DOM 状态的图像

一些伪元素需要一个 `name` 参数。可以是以下之一：

`*`

匹配所有视图过渡组

`root`

匹配 `root` 过渡组，如果没有提供自定义名称，则使用默认名称。

自定义标识符

您可以通过在要过渡的元素上设置 `view-transition-name` 属性来指定自定义标识符。

你可以使用 CSS 选择器来定位这些伪元素并应用不同的动画效果。可以通过创建 `@keyframes` 规则来实现这一点，并将该动画应用于 `::view-transition-old` 或 `::view-transition-new` 伪元素。

# 在运行时修改样式表

## 问题

你希望动态向页面样式表添加 CSS 规则。

## 解决方案

使用 `CSSStyleSheet` 的 `insertRule` 方法添加所需的规则（见 示例 17-14）。

##### 示例 17-14\. 添加 CSS 规则

```
const [stylesheet] = document.styleSheets;
stylesheet.insertRule(`
 .some-selector {
 background-color: red;
 }
`);
```

## 讨论

如果你有动态添加到页面的新 HTML 内容（如单页应用程序），可能会需要这样做。可以在添加新内容时动态添加样式规则。

# 条件性地设置 CSS 类

## 问题

你想要仅在满足某个条件时将 CSS 类应用于元素。

## 解决方案

使用元素的 `classList` 的 `toggle` 方法（参见 示例 17-15）。

##### 示例 17-15\. 条件性地切换类名

```
// Assume isExpanded is a variable with the current expanded
// state
element.classList.toggle('expanded', isExpanded);
```

## 讨论

如果在没有第二个参数的情况下调用 `toggle`，它会在当前未设置时添加类名，或者在已经设置时移除类名。

除了 `toggle`，你还可以使用 `add` 和 `remove` 来通过添加和移除给定的类名来操作类列表。如果在已经设置了类名的情况下调用 `add`，则没有任何效果。类似地，如果在未设置类名的情况下调用 `remove`，也没有任何效果。

# 匹配媒体查询

## 问题

你想要使用 JavaScript 检查特定的媒体查询是否满足。例如，你可以使用 `prefers-color-scheme` 媒体查询来确定用户操作系统是否设置为暗主题。

## 解决方案

使用 `window.matchMedia` 来评估媒体查询或监听其变化（参见 示例 17-16）。

##### 示例 17-16\. 检查暗色主题

```
const isDarkTheme = window.matchMedia('(prefers-color-scheme: dark)').matches;
```

## 讨论

`window.matchMedia` 返回一个 `MediaQueryList` 对象，不仅具有 `matches` 属性，还可以监听 `change` 事件。如果媒体查询的结果发生变化，将触发此事件。

例如，如果用户的操作系统颜色主题设置在应用程序打开时发生更改，则 `prefers-color-scheme` 查询的 `change` 事件将触发。然后，可以检查新的匹配状态（参见 示例 17-17）。

##### 示例 17-17\. 监听媒体查询变化

```
const query = window.matchMedia('(prefers-color-scheme: dark)');
query.addEventListener('change', () => {
  if (query.matches) {
    // switch to dark mode
  } else {
    // switch to light mode
  }
});
```

# 获取元素的计算样式

## 问题

你想要找到来自样式表（而不是内联样式）的特定 CSS 样式。

## 解决方案

使用 `window.getComputedStyle` 来计算元素的最终样式。

# 谨慎使用 getComputedStyle

调用 `getComputedStyle` 时，会强制浏览器重新计算样式和布局，这可能成为性能瓶颈。

考虑在 示例 17-18 中应用了一些样式的 HTML 元素。

##### 示例 17-18\. 带有样式的一些 HTML

```
<style>
  #content {
    background-color: blue;
  }

  .container {
    background-color: red;
    color: white;
  }
</style>

<div id="content" class="container">What color am I?</div>
```

要确定应用于元素的样式，请将元素传递给 `window.getComputedStyle`（参见 示例 17-19）。

##### 示例 17-19\. 获取计算样式

```
const content = document.querySelector('#content');
const styles = window.getComputedStyle(content);
console.log(styles.backgroundColor);
```

因为 ID 选择器的特异性高于类选择器，所以它会赢得冲突，并且 `styles.backgroundColor` 是蓝色。在某些浏览器上，它可能不是字符串“blue”，而是诸如 `rgb(0, 0, 255)` 的颜色表达式。

## 讨论

元素的 `style` 属性仅适用于*内联样式*。参考 示例 17-20。

##### 示例 17-20\. 具有内联样式的元素

```
<style>
  #content {
    background-color: blue;
  }
</style>

<div id="content" style="color: white;">Content</div>
```

此示例将 `color` 属性指定为内联样式，因此可以通过引用 `style` 属性访问。然而，背景颜色来自样式表，不能通过这种方式找到（参见 示例 17-21）。

##### 示例 17-21\. 检查内联样式

```
const content = document.querySelector('#content');
console.log(content.style.backgroundColor); // empty string
console.log(content.style.color); // 'white'
```

因为 `getComputedStyle` 计算元素的最终样式，它包含样式表样式和内联样式（参见 示例 17-22）。

##### 示例 17-22\. 检查计算样式

```
const content = document.querySelector('#content');
const styles = window.getComputedStyle(content);
console.log(styles.backgroundColor); // 'rgb(0, 0, 255)'
console.log(styles.color); // 'rgb(255, 255, 255)'
```
