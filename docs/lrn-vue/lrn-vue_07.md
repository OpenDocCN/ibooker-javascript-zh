# 第七章：高级渲染、动态组件和插件组合

在前几章中，您已经了解了 Vue 的工作原理，如何使用选项 API 和组合 API 组合组件，以及如何使用 Axios 将外部资源数据整合到 Vue 应用程序中。

本章将介绍 Vue 渲染的更高级方面。我们将探讨如何使用渲染函数和 JSX 计算功能组件，以及如何使用 Vue 组件标记动态和条件渲染元素。我们还将学习如何注册自定义插件以在应用程序中使用。

# 渲染函数和 JSX

使用 Vue 编译器 API，Vue 在渲染时将所有用于 Vue 组件的 HTML 模板处理并编译成虚拟 DOM。当 Vue 组件的数据更新时，Vue 触发内部渲染函数，将最新值发送到虚拟 DOM。

在特定场景下，如优化性能、工作在服务器端渲染应用程序或工作在动态组件库中，我们需要绕过 HTML 模板解析器过程。通过直接返回虚拟 DOM 中的渲染虚拟节点，并跳过模板编译过程，`render()` 是解决这类问题的解决方案。

## 使用渲染函数

在 Vue 2 中，`render()` 函数属性接收一个 `createElement` 回调参数。通过适当的参数触发 `createElement`，它返回一个有效的 VNode^(1)。我们通常将 `createElement` 表示为 `h` 函数。^(2)

示例 7-1 展示了在 Vue 2 语法中创建组件的方式。

##### 示例 7-1\. 在 Vue 2 中使用渲染函数

```
const App = {
 render(h) {
  return h(
   'div',
   { id: 'test-id' },
   'This is a render function test with Vue'
  )
 }
}
```

这段代码等同于编写以下模板代码：

```
const App = {
 template: `<div id='test-id'>This is a render function test with Vue</div>`
}
```

在 Vue 3 中，`render` 的语法发生了显著变化。它不再接受 `h` 函数作为参数。相反，`vue` 包公开了一个全局函数 `h`，用于创建 VNodes。因此，我们可以将 示例 7-1 中的代码重写为 示例 7-2 中所示。

##### 示例 7-2\. 在 Vue 3 中使用渲染函数

```
import { createApp, h } from 'vue'

const App = {
 render() {
  return h(
   'div',
   { id: 'test-id' },
   'This is a render function test with Vue'
  )
 }
}
```

输出保持不变。

# 使用渲染函数支持多根节点

由于 Vue 3 支持组件模板的多个根节点，`render()` 可以返回一个包含多个 VNode 的数组，每个节点将以同级方式插入到 DOM 中。

## 使用 `h` 函数创建 VNode

Vue 设计 `h` 函数非常灵活，有三个不同类型的输入参数，详见 表格 7-1。

表格 7-1\. `h` 函数的不同参数

| 参数 | 是否必需？ | 可接受的数据类型 | 描述 |
| --- | --- | --- | --- |
| 组件 | 是 | 字符串、对象或函数 | 它接受字符串作为文本或 HTML 标签元素，组件函数或选项对象。 |
| props | No | Object | 此对象包含从其父级接收的所有组件 `props`、属性和事件，与我们在 `template` 中编写的方式类似。 |
| 嵌套子节点 | No | String、array 或 object | 此参数包括一组 VNodes，或者仅包含文本的字符串组件，或者具有不同 `slots`（参见第三章）作为组件子节点的对象。 |

`h` 函数的语法如下：

```
h(component, { /*props*/ }, children)
```

例如，我们想创建一个组件，其根元素使用 `div` 标签，并具有 `id`、内联边框样式以及一个输入子元素。我们可以像这样调用 `h` 函数：

```
const inputElem = h(
 'input',
 {
  placeholder: 'Enter some text',
  type: 'text',
  id: 'text-input'
 })

const comp = h(
 'div',
 {
  id: 'my-test-comp',
  style: { border: '1px solid blue' }
 },
 inputElem
)
```

在实际 DOM 中，组件的输出将是：

```
<div id="my-test-comp" style="border: 1px solid blue;">
 Text input
 <input placeholder="Enter some text" type="text" id="text-input">
</div>
```

您可以使用以下完整的工作代码进行实验，并尝试不同的 `h` 函数配置：

```
import { createApp, h } from 'vue'

const inputElem = h(
 'input',
 {
  placeholder: 'Enter some text',
  type: 'text',
  id: 'text-input'
 })

const comp = h(
 'div',
 {
  id: 'my-test-comp',
  style: { border: '1px solid blue' }
 },
 inputElem
)

const App = {
 render() {
  return comp
 }
}

const app = createApp(App)

app.mount("#app")
```

## 在渲染函数中编写 JavaScript XML

JavaScript XML（JSX）是由 React 框架引入的 JavaScript 扩展，允许开发人员在 JavaScript 中编写 HTML 代码。JSX 格式中的 HTML 和 JavaScript 代码如下所示：

```
const JSXComp = <div>This is a JSX component</div>
```

上述代码输出一个渲染带有文本“这是一个 JSX 组件”的 `div` 标签的组件。剩下的工作就是在渲染函数中直接返回这个组件：

```
import { createApp, h } from 'vue'

const JSXComp = <div>This is a JSX component</div>

const App = {
 render() {
  return JSXComp
 }
}

const app = createApp(App)

app.mount("#app")
```

Vue 3.0 支持开箱即用的 JSX 编写。JSX 的语法与 Vue 模板不同。要绑定动态数据，我们使用单花括号 `{}`，如示例 7-3。

##### 示例 7-3\. 使用 JSX 编写简单的 Vue 组件

```
import { createApp, h } from 'vue'

const name = 'JSX'
const JSXComp = <div>This is a {name} component</div>

const App = {
 render() {
  return JSXComp
 }
}

const app = createApp(App)

app.mount("#app")
```

我们使用相同的方法绑定动态数据。无需用 `''` 包裹表达式。以下示例显示了如何将值绑定到 `div` 的 `id` 属性上：

```
/**... */
const id = 'jsx-comp'
const JSXComp = <div id={id}>This is a {name} component</div>
/**... */
```

然而，与 React 中的 JSX 不同，我们在 Vue 中不会将诸如 `class` 之类的属性转换为 `className`。相反，我们保留这些属性的原始语法。对于元素的事件监听器（在 React 中是 `onClick`，在 Vue 中仍然是 `onclick` 等）也是如此。

您还可以像 Options API 中的其他 Vue 组件一样注册 JSX 组件作为 `components` 的一部分。在编写动态组件时结合 `render` 函数使用非常方便，并在许多情况下提供更好的可读性。

接下来，我们将讨论如何编写函数组件。

# 函数组件

函数组件是一个无状态组件，绕过了典型的组件生命周期。与使用 Options API 的标准组件不同，函数组件是一个函数，用于表示该组件的渲染函数。

由于它是一个无状态组件，因此无法访问 `this` 实例。相反，Vue 将组件的外部 `props` 和 `context` 作为函数参数暴露出来。函数组件必须使用 `vue` 包中的全局函数 `h()` 创建并返回一个虚拟节点实例。因此，语法将是：

```
import { h } from 'vue'

export function MyFunctionComp(props, context) {
 return h(/* render function argument */)
}
```

`context` 公开了组件的上下文属性，包括用于组件的事件发射器 `emits`，从父组件传递给组件的 `attrs`，以及包含组件嵌套元素的 `slots`。

例如，功能组件 `myHeading` 会在标题元素内显示传递给它的任何文本。我们使用 `level` props 指定标题的级别。如果我们想要将文本 “Hello World” 显示为级别 2 的标题（`<h2>`），则使用 `myHeading` 如下所示：

```
<my-heading level="2">Hello World</my-heading>
```

并且输出应该是：

```
<h2>Hello World</h2>
```

为此，我们使用 `vue` 包中的 `h` 渲染函数，并执行 Example 7-4 中显示的代码。

##### 示例 7-4\. 使用 `h` 函数创建自定义标题组件

```
import { h } from 'vue';

export function MyHeading(props, context) {
 const heading = `h${props.level}`

 return h(heading, context.$attrs, context.$slots);
}
```

Vue 将跳过功能组件的模板渲染过程，并将虚拟节点声明直接添加到其渲染器管道中。这种机制导致功能组件不可用嵌套插槽或属性。

# 为功能组件定义 Props 和 Emits

您可以按照以下语法明确定义功能组件的可接受 `props` 和 `emits`：

```
MyFunctionComp.props = ['prop-one', 'prop-two']
MyFunctionComp.emits = ['event-one', 'event-two']
```

如果不定义，`context.props` 将与 `context.attrs` 具有相同的值，其中包含传递给组件的所有属性。

当您需要以编程方式控制组件渲染时，功能组件非常强大，特别适用于组件库作者，他们需要为用户需求提供组件的低级别灵活性。

###### 注意

Vue 3 提供了一种额外的方式，使用 `<script setup>` 编写组件。只有在以*SFC*格式编写组件时才相关，详见 “setup”。

接下来，我们将探讨如何使用插件为 Vue 应用程序添加外部功能。

# 使用 Vue 插件全局添加自定义功能

我们使用插件在 Vue 应用程序中全局添加第三方库或额外的自定义功能。Vue 插件是一个对象，公开一个名为 `install()` 的方法，其中包含逻辑代码，并负责安装插件本身。以下是一个示例插件：

```
/* plugins/samplePlugin.ts */
import type { App  } from 'vue'

export default {
 install(app: App<Element>, options: Object) {
  // Installation logic
 }
}
```

在此代码中，我们在位于 `plugins` 目录中的 `samplePlugin` 文件中定义了我们的示例插件代码。`install()` 接收两个参数：一个 `app` 实例和一些作为插件配置的 `options`。

例如，让我们编写一个 `truncate` 插件，它将添加一个新的全局函数属性 `$truncate`。如果字符串长度超过 `options.limit` 字符，则 `$truncate` 将返回截断的字符串，如 Example 7-5 所示。

##### 示例 7-5\. 编写截断插件

```
/* plugins/truncate.ts */
import type { App } from 'vue';

export default {
  install(app: App<Element>, options: { limit: number }) {
    const truncate = (str: string) => {
      if (str.length > options.limit) {
        return `${str.slice(0, options.limit)}...`;
      }

      return str;
    }
    app.config.globalProperties.$truncate = truncate;
  }
}
```

要在我们的应用程序中使用此插件，我们在 `main.ts` 中创建的 `app` 实例上调用 `app.use()` 方法：

```
/* main.ts */
import { createApp } from 'vue'
import truncate from './plugins/truncate'

const App = {}

//1\. Create the app instance
const app = createApp(App);

//2\. Register the plugin
app.use(truncate, { limit: 10 })

app.mount('#app')
```

Vue 引擎将安装 `truncate` 插件，并初始化为 10 个字符的 `limit`。该插件将在 `app` 实例中的每个 Vue 组件中可用。您可以在 `script` 部分使用 `this.$truncate` 或在 `template` 部分直接使用 `$truncate` 来调用此插件：

```
import { createApp, defineComponent } from 'vue'
import truncate from './plugins/truncate'

const App = defineComponent({
 template: `
 <h1>{{ $truncate('My truncated long text') }}</h1>
 <h2>{{ truncatedText }}</h2>
 `,
 data() {
  return {
   truncatedText: this.$truncate('My 2nd truncated text')
  }
 }
});

const app = createApp(App);
app.use(truncate, { limit: 10 })
app.mount('#app')
```

输出应如 图 7-1。

![展示两个标题文本的显示，都作为截断文本的结果来自调用截断插件。](img/lvue_0701.png)

###### 图 7-1\. 组件输出文本被截断

然而，只有在 `<template>` 部分使用时或在 `script` 部分的 Options API 中作为 `this.$truncate`，才能使用 `$truncate`。在 `<script setup>` 或 `setup()` 中访问 `$truncate` 是*不可能*的。为此，我们需要使用提供/注入模式（见“使用提供/注入模式在组件之间通信”），从以下位置的插件 `install` 函数开始，位于 `plugins/truncate.ts` 文件中：

```
/* plugins/truncate.ts */
export default {
  install(app: App<Element>, options: { limit: number }) {
    //...
    app.provide("plugins", { truncate });
  }
}
```

Vue 将 `truncate` 作为 `plugins` 对象的一部分传递给所有应用程序的组件。通过这样，我们可以使用 `inject` 接收我们想要的插件 `truncate` 并继续计算 `truncatedText`：

```
<script setup lang="ts">
import { inject } from 'vue';

const { truncate } = inject('plugins');
const truncatedText = truncate('My 2nd truncated text');
</script>
```

插件在组织全局方法和在其他应用程序中重用时非常有帮助。在安装外部库时编写逻辑，例如*axios*用于获取外部数据，*i18n*用于本地化等，也是非常有益的。

# 在我们的应用程序中注册 Pinia 和 Vue Router

在我们的应用程序脚手架期间，Vite 使用与在 `main.ts` 中生成的原始代码中反映的相同方法添加 Pinia 和 Vue Router 作为应用程序插件。

下一节将介绍如何使用 Vue `<component>` 标签在运行时渲染动态组件。

# 使用 `<component>` 标签进行动态渲染

`<component>` 标签充当渲染 Vue 组件的占位符，根据传递给其 `is` props 的组件引用名称进行渲染，遵循此语法：

```
<component is="targetComponentName" />
```

假设您的目标组件可以从 Vue 实例访问（注册为应用程序的组件或在嵌套 `<component>` 时作为父组件），Vue 引擎将根据名称字符串查找目标组件，并用目标组件替换标签。目标组件还将继承传递给 `<component>` 的所有额外 props。

假设我们有一个 `HelloWorld` 组件，用于呈现文本“Hello World”：

```
<template>
  <div>Hello World</div>
</template>
```

我们将此组件注册到 `App`，然后使用 `<component>` 标签动态渲染，如下所示：

```
<template>
  <component is="HelloWorld" />
</template>
<script lang="ts">
import HelloWorld from "@/components/HelloWorld";
import { defineComponent } from "vue";

export defineComponent({
 components: { HelloWorld },
});
</script>
```

您还可以使用 `v-bind` 指令（简写为 `:`）将组件绑定为 `is` props 的引用。我们可以通过以下方式将前两个代码块缩短为单个 `App` 组件：

```
<template>
  <component :is="myComp" />
</template>
<script lang="ts">
import HelloWorld from "@/components/HelloWorld";
import { defineComponent } from "vue";

export defineComponent({
 data() {
  return {
   myComp: {
    template: '<div>Hello World</div>'
   }
  }
 }
});
</script>
```

注意，组件引用`myComp`遵循 Options API 语法。你也可以传递导入的 SFC 组件。两种情况的输出应该是相同的。

`<component>`标签非常强大。例如，如果你有一个画廊组件，可以选择将每个画廊项条件地渲染为一个`Card`组件或`Row`组件，使用`<component>`来进行部分切换。

然而，切换组件意味着 Vue 完全卸载当前元素并删除所有组件的当前数据状态。切换回该组件相当于创建一个具有新数据状态的新实例。为了防止这种行为并为未来的切换维护一个被动元素的状态，我们使用`<keep-alive>`组件。

# 使用<keep-alive>保持组件实例的存活状态

`<keep-alive>`是一个内置的 Vue 组件，用于包装动态元素，并在非活动模式下保留组件状态。

假设我们有两个组件，`StepOne`和`StepTwo`。在`StepOne`组件中，有一个字符串`input`字段，使用`v-model`与本地数据属性`name`进行双向绑定：

```
<!--StepOne.vue-->
<template>
  <div>
    <label for="name">Step one's input</label>
    <input v-model="name" id="name" />
  </div>
</template>
<script setup lang="ts">
import { ref } from 'vue';

const name = ref<string>("");
</script>
```

而`StepTwo`组件渲染一个静态字符串：

```
<!--StepTwo.vue-->
<template>
  <h2>{{ name }}</h2>
</template>
<script setup lang="ts">
const name = "Step 2";
</script>
```

在主`App`模板中，我们将使用`component`标签将本地数据属性`activeComp`作为组件引用进行渲染。`activeComp`的初始值是`StepOne`，我们有一个按钮可以在`StepOne`和`StepTwo`之间切换：

```
<template>
  <div>
    <keep-alive>
      <component :is="activeComp" />
    </keep-alive>
    <div>
      <button @click="activeComp = 'StepOne'" v-if="activeComp === 'StepTwo'">
      Go to Step Two
      </button>
      <button @click="activeComp = 'StepTwo'" v-else>Back to Step One</button>
    </div>
    </div>
</template>
<script lang="ts">
import { defineComponent } from "vue";
import StepOne from "./components/StepOne.vue";
import StepTwo from "./components/StepTwo.vue";

export default defineComponent({
  components: { StepTwo, StepOne },
  data() {
    return {
      activeComp: "StepOne",
    };
  },
});
</script>
```

每当你在`StepOne`和`StepTwo`之间切换时，Vue 都会保留从输入字段接收的`name`属性的任何值。当切换回`StepOne`时，你可以继续使用先前的值，而不是从初始值开始。

你还可以使用其`max`属性为`keep-alive`定义最大缓存实例数量：

```
<keep-alive max="2">
  <component :is="activeComp" />
 </keep-alive>
```

这段代码通过设置`max="2"`来定义`keep-alive`应保持的最大实例数。一旦缓存实例数超过限制，Vue 将从缓存列表中移除最近最少使用（LRU）的实例，以便缓存新的实例。

# 概要

本章探讨了如何使用 JSX 和函数组件控制组件渲染，全局注册 Vue 自定义插件，并使用`<component>`标签动态和条件地渲染组件。

下一章将介绍 Vue Router，Vue 的官方路由管理库，并讨论如何使用 Vue Router 在应用程序中处理不同路由之间的导航。

^(1) 虚拟节点

^(2) 代表超文本，意味着使用 JavaScript 代码创建 HTML
