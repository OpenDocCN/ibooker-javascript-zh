# 第十二章：Web 组件

# 介绍

Web 组件是一种构建具有自身行为的新 HTML 元素的方法。这种行为被封装在一个*自定义元素*中。

## 创建一个组件

你可以通过定义一个扩展了`HTMLElement`的类来创建一个 Web 组件，如示例 12-1 所示。

##### 示例 12-1\. 一个简单的 Web 组件

```
class MyComponent extends HTMLElement {
  connectedCallback() {
    this.textContent = 'Hello from MyComponent';
  }
}
```

当你将自定义元素添加到 DOM 中时，浏览器会调用`connectedCallback`方法。这通常是大多数组件逻辑的所在地。这是其中一个*生命周期回调*。其他一些生命周期回调包括：

`disconnectedCallback`

在从 DOM 中移除自定义元素时调用。这是一个执行清理工作（如移除事件监听器）的好地方。

`attributeChangedCallback`

当你改变元素的一个监视属性时调用。

## 注册一个自定义元素

创建完自定义元素类之后，必须在 HTML 文档中使用它之前向浏览器注册它。你可以通过在全局的`customElements`对象上调用`define`来注册你的自定义元素，如示例 12-2 所示。

##### 示例 12-2\. 在浏览器中注册自定义元素

```
customElements.define('my-component', MyComponent);
```

###### 注意

如果尝试定义一个已经定义过的自定义元素，浏览器会抛出一个错误。如果这对你可能是一个可能性，你可以调用`customElements.get('my-component')`来检查它是否已经定义。如果返回`undefined`，则可以安全地调用`customElements.define`。

注册了元素后，你可以像使用任何其他 HTML 元素一样使用它，如示例 12-3 所示。

##### 示例 12-3\. 使用自定义元素

```
<my-component></my-component>
```

###### 注意

自定义元素的名称必须始终使用连字符命名。这是规范要求的。它们也必须始终有一个闭合标记，即使没有子内容。

## 模板

有几种将 HTML 标记引入 Web 组件中的方法。例如，在`connectedCallback`中，你可以通过调用`document.createElement`手动创建元素并手动附加它们。

你还可以使用`<template>`元素指定组件的标记。它包含一些 HTML，在`connectedCallback`期间用来为你的组件提供内容。这些模板非常简单，它们不支持数据绑定、变量插值或任何类型的逻辑。它们只作为 HTML 内容的起点。在`connectedCallback`中，你可以选择元素，设置动态值，并根据需要添加事件监听器。

## 插槽

`<slot>` 是一个特殊的元素，你可以在模板中使用它。插槽是一种用于传递某些子内容的占位符。组件可以有一个默认插槽以及一个或多个*命名*插槽。你可以使用命名插槽在组件内放置多个内容片段。

示例 12-4 展示了一个具有命名和默认插槽的简单模板。

##### 示例 12-4\. 带有插槽的模板

```
<template>
  <h2><slot name="name"></slot></h2>
  <slot></slot>
</template>
```

假设这个模板用于 `<author-bio>` 组件中，如 示例 12-5 所示。

##### 示例 12-5\. 为插槽指定内容

```
<author-bio>
  <span slot="name">John Doe</span>
  <p>John is a great author who has written many books.</p>
</author-bio>
```

在组件的子内容中，你可以指定一个 `slot` 属性，对应组件模板中的一个具名插槽。包含文本“John Doe”的 `span` 元素将被放置在组件的 `name` 插槽中，即 `h2` 元素内。任何其他没有 `slot` 元素的子内容都会被放置在默认插槽中（没有名称的插槽）。

## 影子 DOM

影子 DOM 是一组与主 DOM 隔离的元素集合。Web 组件广泛使用影子 DOM。使用影子 DOM 的一个主要优势是可以进行作用域 CSS 样式。你在影子 DOM 中定义的任何样式 *仅* 应用于该影子 DOM 内部的元素。文档中的其他元素，即使通常会匹配 CSS 规则的选择器，也不会应用这些 CSS 样式。

这种样式作用范围是双向的。如果你在页面上有全局样式，它们将不会应用于影子 DOM 中的任何元素。

通过将 *shadow root* 附加到 Web 组件来创建影子 DOM，这个影子 DOM 可以是开放的或者关闭的。当影子 DOM 是开放的时候，你可以使用 JavaScript 访问和修改它的元素。当它是关闭的时候，Web 组件的 `shadowRoot` 属性为 `null`，因此无法访问其内容。

## Light DOM

然而，使用影子 DOM 是完全可选的。*Light DOM* 指的是 Web 组件内部的常规、非封装的 DOM。由于 Light DOM 不会从页面其余部分封装起来，全局样式将应用于其子元素。

# 创建一个显示今天日期的组件

## 问题

你希望一个 Web 组件能够在浏览器的本地化环境中格式化并展示今天的日期。

## 解决方案

在 Web 组件内部使用 `Intl.DateTimeFormat` 来格式化当前日期（参见 示例 12-6）。

##### 示例 12-6\. 格式化当前日期的自定义元素

```
class TodaysDate extends HTMLElement {
  connectedCallback() {
    const formatter = new Intl.DateTimeFormat(
      navigator.language,
      { dateStyle: 'full' }
    );

    this.textContent = formatter.format(new Date());
  }
}

customElements.define('todays-date', TodaysDate);
```

现在你可以使用这个 Web 组件来展示今天的日期，不需要任何属性或子内容，如 示例 12-7 所示。

##### 示例 12-7\. 展示当前日期

```
<p>
  Today's date is: <todays-date></todays-date>
</p>
```

## 讨论

当一个 `<todays-date>` 元素进入 DOM 时，浏览器调用 `connectedCallback` 方法。在 `connectedCallback` 中，`TodaysDate` 类使用 `Intl.DateTimeFormat` 对象格式化当前日期，你可能还记得这个对象来自 第十一章。`connectedCallback` 将这个格式化的日期字符串设置为元素的 `textContent`，这个属性是从 `Node`（`HTMLElement` 的祖先）继承而来。

# 创建一个格式化自定义日期的组件

## 问题

你希望一个 Web 组件能够格式化任意日期值。

## 解决方案

为 Web 组件添加一个 `date` 属性，并使用它来生成格式化的日期（参见 示例 12-8）。你可以监听这个属性的变化，并在日期属性变化时重新格式化日期。

##### 示例 12-8\. 自定义日期组件

```
class DateFormatter extends HTMLElement {
  // The browser will only notify the component about changes, via the
  // attributeChangedCallback, for attributes that are listed here.
  static observedAttributes = ['date'];

  constructor() {
    super();

    // Create the format here so you don't have to
    // re-create it every time the date changes.
    this.formatter = new Intl.DateTimeFormat(
      navigator.language,
      { dateStyle: 'full' }
    );
  }

  /**
 * Formats the date represented by the current value of the 'date'
 * attribute, if any.
 */
  formatDate() {
    if (this.hasAttribute('date')) {
      this.textContent = this.formatter.format(
        new Date(this.getAttribute('date'))
      );
    } else {
      // If no date specified, show nothing.
      this.textContent = '';
    }
  }

  attributeChangedCallback() {
    // Only watching one attribute, so this must be a change
    // to the date attribute. Update the formatted date, if any.
    this.formatDate();
  }

  connectedCallback() {
    // The element was just added. Show the initial formatted date, if any.
    this.formatDate();
  }
}

customElements.define('date-formatter', DateFormatter);
```

现在您可以将日期传递给`date`属性，以便以用户的区域设置格式化它（参见示例 12-9）。

##### 示例 12-9\. 使用`date-formatter`元素

```
<date-formatter date="2023-10-16T03:52:49.955Z"></date-formatter>
```

## 讨论

此配方在“创建一个显示今天日期的组件”基础上增加了通过属性指定自定义日期的功能。

默认情况下，如果更改传递给自定义元素的属性值，什么也不会发生。`connectedCallback`中的逻辑仅在首次将组件添加到 DOM 时运行。要使组件对属性更改做出响应，可以实现`attributeChangedCallback`方法。在`date-formatter`组件中，此方法接收更新后的`date`属性并创建新的格式化日期。当属性发生更改时，浏览器会调用此方法，并传递属性名称、旧值和新值。

然而，仅此还不足以解决问题。如果仅实现`attributeChangedCallback`，则仍无法收到属性更改的通知。这是因为浏览器仅为*观察到*的属性调用`attributeChangedCallback`。这允许您定义属性的子集，以便浏览器仅为您感兴趣的那些属性调用`attributeChangedCallback`。要定义这些属性，请将静态的`observedAttributes`属性添加到您的组件类中。这应该是一个属性名称数组。

在`date-formatter`组件中，您仅监视一个属性（`date`属性）。因此，在`attributeChangedCallback`中，您无需检查`name`参数，因为您已经知道更改的是`date`属性。对于具有多个监视属性的组件，您可以检查`name`以查找已更改的属性。

如果您使用 JavaScript 更改`date`属性的值，则`attributeChangedCallback`将运行并更新格式化日期。

# 创建反馈组件

## 问题

您希望创建一个可重用的组件，用户可以在其中提供关于页面是否有帮助的反馈。

## 解决方案

创建一个网络组件来呈现反馈按钮，并在用户单击其中一个按钮时分发自定义事件。

首先，您需要创建一个模板元素，其中包含此组件使用的标记，如示例 12-10 所示。

##### 示例 12-10\. 创建模板

```
const template = document.createElement('template');
template.innerHTML = `
 <style>
 .feedback-prompt {
 display: flex;
 align-items: center;
 gap: 0.5em;
 }

 button {
 padding: 0.5em 1em;
 }
 </style>

 <div class="feedback-prompt">
 <p>Was this helpful?</p>
 <button type="button" data-helpful="true">Yes</button>
 <button type="button" data-helpful="false">No</button>
 </div>
`;
```

此组件使用包含模板标记的影子 DOM（参见示例 12-11）。CSS 样式规则仅限于此组件。

##### 示例 12-11\. 组件实现

```
class FeedbackRating extends HTMLElement {
  constructor() {
    super();

    // Create the shadow DOM and render the template into it.
    const shadowRoot = this.attachShadow({ mode: 'open' });
    shadowRoot.appendChild(template.content.cloneNode(true));
  }

  connectedCallback() {
    this.shadowRoot.querySelector('.feedback-prompt').addEventListener('click',
    event => {
      const { helpful } = event.target.dataset;

      if (typeof helpful !== 'undefined') {
        // Once a feedback option is chosen, hide the buttons and show a
        // confirmation.
        this.shadowRoot.querySelector('.feedback-prompt').remove();
        this.shadowRoot.textContent = 'Thanks for your feedback!';

        // JavaScript doesn't have a 'parseBoolean' type function, so convert the
        // string value to the corresponding boolean value.
        this.helpful = helpful === 'true';

        // Dispatch a custom event, so your app can be notified when a feedback
        // button is clicked.
        this.shadowRoot.dispatchEvent(new CustomEvent('feedback', {
          composed: true, // This is needed to "escape" the shadow DOM boundary.
          bubbles: true // This is needed to propagate up the DOM.
        }));
      }
    });
  }
}

customElements.define('feedback-rating', FeedbackRating);
```

现在您可以将此反馈组件添加到您的应用程序中（参见示例 12-12）。

##### 示例 12-12\. 使用反馈评分组件

```
<h2>Feedback</h2>
<feedback-rating></feedback-rating>
```

您可以监听自定义`feedback`事件，以便在用户选择反馈选项时收到通知（参见示例 12-13）。如何处理这些信息由您决定；也许您想要使用 Fetch API 将数据发送到分析端点。

##### 示例 12-13\. 监听反馈事件

```
document.querySelector('feedback-rating').addEventListener('feedback', event => {
  // Get the value of the feedback component's "helpful" property and send it to an
  // endpoint with a POST request.
  fetch('/api/analytics/feedback', {
    method: 'POST',
    body: JSON.stringify({ helpful: event.target.helpful }),
    headers: {
      'Content-Type': 'application/json'
    }
  });
});
```

## 讨论

`feedback-rating`组件显示提示和两个按钮。用户根据他们认为网站内容是否有帮助来点击两个按钮中的一个。

`click`事件侦听器使用事件委托。它不是给每个按钮添加监听器，而是添加一个响应反馈提示任何位置的单个监听器。如果点击的元素没有`data-helpful`属性，则用户可能没有点击反馈按钮，因此不执行任何操作。否则，它将字符串值转换为布尔值，并将其设置为可以稍后检索的自定义元素属性。它还分发了一个事件，您可以在其他地方监听到。

要使此事件跨越影子 DOM 进入常规 DOM，必须设置`composed: true`选项。否则，添加到自定义元素的任何事件侦听器都不会被触发。

触发事件后，可以检查反馈元素本身（作为`event.target`属性可用）的`helpful`属性，以确定用户点击了哪个反馈按钮。

由于样式和标记包含在影子 DOM 中，因此 CSS 规则不会影响影子 DOM 外的任何元素。这一点很重要，否则像`button`这样的元素选择器会样式化页面上的每个按钮。由于样式是作用域的，它们仅应用于自定义元素内的按钮。

但是，传递到组件插槽的内容*可以*由全局 CSS 规则进行样式化。插槽内容不会移动到影子 DOM 中，而是保留在标准或轻 DOM 中。

# 创建个人资料卡组件

## 问题

您想要创建一个可重用的卡片组件来显示用户资料。

## 解决方案

在您的 Web 组件中使用插槽将内容传递到特定区域。

首先，按照示例 12-14 中显示的样式和标记定义模板。

##### 示例 12-14\. 个人资料卡模板

```
const template = document.createElement('template');
template.innerHTML = `
 <style>
 :host {
 display: grid;
 border: 1px solid #ccc;
 border-radius: 5px;
 padding: 8px;
 grid-template-columns: auto 1fr;
 column-gap: 16px;
 align-items: center;
 margin: 1rem;
 }

 .photo {
 border-radius: 50%;
 grid-row: 1 / span 3;
 }

 .name {
 font-size: 2rem;
 font-weight: bold;
 }

 .title {
 font-weight: bold;
 }
 </style>

 <div class="photo"><slot name="photo"></slot></div>
 <div class="name"><slot name="name"></slot></div>
 <div class="title"><slot name="title"></slot></div>
 <div class="bio"><slot></slot></div>
`;
```

此模板具有三个命名插槽（photo、name 和 title）和一个用于传记的默认插槽。组件实现本身相当简单；它只是创建并附加了一个包含模板的影子根（参见示例 12-15）。

##### 示例 12-15\. 组件实现

```
class ProfileCard extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.appendChild(template.content.cloneNode(true));
  }
}

customElements.define('profile-card', ProfileCard);
```

要使用该组件，可以在子元素上指定`slot`属性，以指定内容应放入哪个插槽（参见示例 12-16）。没有`slot`属性的传记元素放置在默认插槽中。

##### 示例 12-16\. 使用个人资料卡

```
<profile-card>
  <img slot="photo" src="/api/portraits/chavez.jpg" />
  <div slot="name">Phillip Chavez</div>
  <div slot="title">CEO</div>
  <p>Philip is a great CEO.</p>
</profile-card>

<profile-card>
  <img slot="photo" src="/api/portraits/lynch.jpg" />
  <div slot="name">Jamie Lynch</div>
  <div slot="title">Vice President</div>
  <p>Jamie is a great vice president.</p>
</profile-card>
```

图 12-1 显示了个人资料卡组件的渲染结果。

![渲染的个人资料卡](img/wacb_1201.png)

###### 图 12-1\. 渲染的个人资料卡

## 讨论

在 CSS 样式中，您可能已经注意到`:host`选择器，它表示应用于自定义元素*影子宿主*的样式。这是影子 DOM 附加到的元素。

通过此示例，您可以看到 Web 组件如何让您创建可重复使用的内容和布局。插槽是一个强大的工具，使您能够在需要的地方精确插入内容。

# 创建一个懒加载图像组件

## 问题

您需要一个可重用的组件，其中包含一个图像，直到滚动到视口中才加载。

## 解决方案

使用`IntersectionObserver`等待元素滚动到视图中，然后在包含的图像上设置`src`元素。

本配方调整了“滚动到视图时懒加载图像”（见示例 12-17 和 12-18）在 Web 组件中的解决方案。

##### 示例 12-17\. `LazyImage`组件

```
class LazyImage extends HTMLElement {
  constructor() {
    super();

    const shadowRoot = this.attachShadow({ mode: 'open' });
    this.image = document.createElement('img');
    shadowRoot.appendChild(this.image);
  }

  connectedCallback() {
    const observer = new IntersectionObserver(entries => {
      if (entries[0].isIntersecting) {
        console.log('Loading image');
        this.image.src = this.getAttribute('src');
        observer.disconnect();
      }
    });

    observer.observe(this);
  }
}

customElements.define('lazy-image', LazyImage);
```

##### 示例 12-18\. 使用`LazyImage`组件

```
<lazy-image src="https://placekitten.com/200/138"></lazy-image>
```

## 讨论

一旦元素滚动到视图中，`IntersectionObserver`回调获取`src`属性，并将其设置为图像的`src`属性，从而触发图像加载。

###### 注意

此示例说明如何创建扩展内置元素的自定义元素，但对于懒加载图像，您可能不需要它。较新的浏览器支持`img`标签上的`loading="lazy"`属性，具有相同的效果——直到滚动到视图中，图像才会加载。

# 创建一个披露组件

## 问题

您希望通过单击按钮显示或隐藏一些内容。例如，您可能有一个默认折叠的表单的“高级”部分，但可以通过单击按钮展开。

## 解决方案

构建一个披露网络组件。该组件分为两部分：切换内容的按钮和内容本身。这两部分将各自有一个插槽。默认插槽用于内容，按钮则有一个命名插槽。此组件还可以通过改变其`open`属性的值来以编程方式展开或折叠。

首先，定义披露组件的模板，如示例 12-19 所示。

##### 示例 12-19\. 披露组件模板

```
const template = document.createElement('template');
template.innerHTML = `
 <div>
 <button type="button" class="toggle-button">
 <slot name="title"></slot>
 </button>
 <div class="content">
 <slot></slot>
 </div>
 </div>
`;
```

组件的实现在示例 12-20 中显示。

##### 示例 12-20\. 披露组件的实现

```
class Disclosure extends HTMLElement {
  // Watch the 'open' attribute to react to changes.
  static observedAttributes = ['open'];

  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.appendChild(template.content.cloneNode(true));

    this.content = this.shadowRoot.querySelector('.content');
  }

  connectedCallback() {
    this.content.hidden = !this.hasAttribute('open');
    this.shadowRoot.querySelector('.toggle-button')
      .addEventListener('click', () => {
        if (this.hasAttribute('open')) {
          // Content is currently showing; remove the 'open'
          // attribute and hide the content.
          this.removeAttribute('open');
          this.content.hidden = true;
        } else {
          // Content is currently hidden; add the 'open' attribute
          // and show the content.
          this.setAttribute('open', '');
          this.content.hidden = false;
        }
      });
  }

  attributeChangedCallback(name, oldValue, newValue) {
    // Update the content's hidden state based on the new attribute value.
    if (newValue !== null) {
      this.content.hidden = false;
    } else {
      this.content.hidden = true;
    }
  }
}

// The element name must be hyphenated.
customElements.define('x-disclosure', Disclosure);
```

最后一件事——您需要在页面上添加一小段 CSS。否则，子内容将在页面上闪烁片刻，然后消失。这是因为在自定义元素注册之前，它没有行为，浏览器不知道其插槽。这意味着任何子内容将在页面中呈现。

然后，一旦自定义元素被定义，子内容移动到插槽中并消失。

要解决这个问题，您可以使用 CSS 来隐藏元素的内容，直到通过使用`:defined`伪类注册它。

##### 示例 12-21\. 修复闪烁问题

```
x-disclosure:not(:defined) {
  display: none;
}
```

这将最初隐藏内容。一旦自定义元素定义，元素将显示出来。您不会看到闪烁，因为内容已经移动到插槽中。

最后，您可以使用披露元素，如示例 12-22 所示。

##### 示例 12-22\. 使用披露元素

```
<x-disclosure>
  <div slot="title">Details</div>
  This is the detail child content that will be expanded or collapsed
  when clicking the title button.
</x-disclosure>
```

切换按钮将显示文本“Details”，因为它放置在`title`插槽中。其余内容放置在默认插槽中。

## 讨论

披露组件使用其`open`属性来确定是否显示子内容。当点击切换按钮时，根据当前状态添加或删除属性，然后有条件地应用`hidden`属性到子内容。

您还可以通过添加或移除`open`属性来程序化地切换子内容的可见性。这是因为组件正在观察`open`属性。如果您使用 JavaScript 更改它，甚至在浏览器开发工具中更改它，浏览器会调用组件的`attributeChangedCallback`方法，并传递新的值。

`open`属性没有值。如果要默认打开内容，请简单地添加没有值的`open`属性，如示例 12-23 所示。

##### 示例 12-23\. 默认显示内容

```
<x-disclosure open>
  <div slot="title">Details</div>
  This is the detail child content that will be expanded or collapsed
  when clicking the title button.
</x-disclosure>
```

如果移除该属性，则`attributeChangedCallback`中的`newValue`参数将为`null`。在这种情况下，它将通过应用`hidden`属性来隐藏子内容。如果添加了没有值的属性，如示例 12-23，则`newValue`参数将为空字符串。在这种情况下，它将移除`hidden`属性。

# 创建一个样式化按钮组件

## 问题

您希望创建一个具有不同样式选项的可重用按钮组件。

## 解决方案

按钮将有三个变体：

+   默认变体，带有灰色背景

+   “primary”变体，带有蓝色背景

+   “danger”变体，带有红色背景

首先，创建带有自定义按钮样式的模板，以及“primary”和“danger”变体的 CSS 类，如示例 12-24 所示。

##### 示例 12-24\. 按钮模板

```
const template = document.createElement('template');
template.innerHTML = `
 <style>
 button {
 background: #333;
 padding: 0.5em 1.25em;
 font-size: 1rem;
 border: none;
 border-radius: 5px;
 color: white;
 }

 button.primary {
 background: #2563eb;
 }

 button.danger {
 background: #dc2626;
 }
 </style>

 <button>
 <slot></slot>
 </button>
`;
```

大部分模板都是 CSS。组件本身的实际标记非常简单：只是一个带有默认插槽的按钮元素。

组件本身将支持两个属性：

`variant`

按钮变体的名称（`primary`或`danger`）

`type`

传递到底层`button`元素的`type`属性。将其设置为`button`以防止提交表单（参见示例 12-25）。

##### 示例 12-25\. 按钮组件

```
class StyledButton extends HTMLElement {
  static observedAttributes = ['variant', 'type'];

  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.appendChild(template.content.cloneNode(true));
    this.button = this.shadowRoot.querySelector('button');
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (name === 'variant') {
      this.button.className = newValue;
    } else if (name === 'type') {
      this.button.type = newValue;
    }
  }
}

customElements.define('styled-button', StyledButton);
```

要添加点击侦听器，实际上您不必再做任何额外的工作。您可以向`styled-button`元素添加点击侦听器，当您单击底层按钮时将触发它，这要归功于事件委托。通过事件委托，您可以向父元素添加事件侦听器，其子元素的事件也会触发父元素的事件侦听器。

最后，这是如何使用`styled-button`组件（参见示例 12-26）。

##### 示例 12-26\. 使用`styled-button`组件

```
<styled-button id="default-button" type="button">Default</styled-button>
<styled-button id="primary-button" type="button" variant="primary">
  Primary
</styled-button>
<styled-button id="danger-button" type="button" variant="danger">
  Danger
</styled-button>
```

## 讨论

通过在按钮元素上设置与变体名称相等的类名来应用样式。这将导致相应的 CSS 规则应用所需的背景颜色。

您不需要在`connectedCallback`中添加任何代码来应用类，因为浏览器会调用`attributeChangedCallback`，包括初始值和后续更新的值。

您可以像处理普通按钮一样向`styled-button`添加点击事件监听器（参见示例 12-27）。

##### 示例 12-27\. 添加点击监听器

```
<script>
document.querySelector('#default-button').addEventListener('click', () => {
  console.log('Clicked the default button');
});
</script>

<styled-button id="default-button" type="button">Default</styled-button>
```
