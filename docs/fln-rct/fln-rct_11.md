# 第十章：React 的替代方案

在上一章中，我们深入讨论了 React 服务器组件（RSCs）的新兴主题。我们探讨了它们的工作原理、使用时机以及为什么它们需要像下一代捆绑器、路由器等强大的工具。我们进一步区分了服务器组件和服务器渲染，并甚至从头开始实现了一个简单的 RSC 渲染器，以便理解其底层机制。

随着我们开始探索 React 的替代方案，对框架和服务器组件的角色和功能的理解将提供宝贵的背景信息。本章中讨论的每个库都附带其关联的框架，并且我们在 React 中涵盖的原则和权衡将同样适用于这些生态系统。

当我们将注意力从 React 及其生态系统转向时，让我们深入了解前端开发生态系统中的一些流行替代品：Vue.js、Angular、Svelte、Solid 和 Qwik。每个库和框架都介绍了其自己的响应性模型和 UI 开发思路。了解这些不同的模型可以拓宽我们的视野，并为解决项目中的问题提供更多工具。

# Vue.js

Vue.js 是一个流行的用于构建用户界面的 JavaScript 框架。由 Evan You 开发，他是一位曾在 AngularJS 项目上工作的前 Google 工程师。Vue.js 旨在提取 Angular 的优点，但在更轻量、更易维护和少观点化的包中实现。

Vue 的一个最显著特点是非侵入式的响应式系统。组件状态由响应式 JavaScript 对象组成。当您修改它们时，视图会更新。它使状态管理简单直观，但理解其工作原理也很重要，以避免一些常见的陷阱。

在 Vue 的响应性模型中，它拦截对象属性的读取和写入。Vue 2 由于浏览器支持的限制，仅使用 getter/setter，但在 Vue 3 中，响应式对象使用代理，而 ref 使用 getter/setter。从 Vue 文档中，以下是一些伪代码，说明了它们的工作原理：

```
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key) {
      track(target, key)
      return target[key]
    },
    set(target, key, value) {
      target[key] = value
      trigger(target, key)
    }
  })
}

function ref(value) {
  const refObject = {
    get value() {
      track(refObject, 'value')
      return value
    },
    set value(newValue) {
      value = newValue
      trigger(refObject, 'value')
    }
  }
  return refObject
}
```

这有点过于简化，但在这里我们演示了一个简单的响应式系统，利用代理实现。`reactive` 函数接受一个对象并返回该对象的代理，拦截 `get` 和 `set` 操作。在 `get` 操作时，它调用 `track` 函数并返回请求的属性。在 `set` 操作时，它更新值并调用 `trigger` 函数。

另一方面，`ref` 函数将一个值封装在对象中，并为该值提供响应式的 `get` 和 `set` 操作，类似于代理，但具有不同的结构，确保在访问或修改期间适当地调用 `track` 和 `trigger` 函数。

这是一个非常简单的反应性系统示例，但它展示了 Vue 响应性模型的基本原则。这种响应性模型甚至可以用于更新 DOM。我们可以实现简单的“反应式渲染”如下：

```
import { ref, watchEffect } from "vue";

const count = ref(0);

watchEffect(() => {
  document.body.innerHTML = `count is: ${count.value}`;
});

// updates the DOM
count.value++;
```

实际上，这与 Vue 组件保持状态和 DOM 同步的方式非常接近——每个组件实例创建一个反应性效果来渲染和更新 DOM。当然，Vue 组件使用比`innerHTML`更高效的方法来更新 DOM，但这应该足以让你对它的工作原理有一个基本的了解。

`ref()`、`computed()` 和 `watchEffect()` API 都是 Vue Composition API 的一部分。

## 信号

还有许多其他框架引入了类似 Vue Composition API 中`refs`的响应性原语，称之为“信号”，我们将在本章讨论。

从根本上说，信号与 Vue 的`refs`是同一种响应性原语。它是一个值容器，提供访问时的依赖跟踪，以及在变异时触发副作用。这种基于响应性原语的范式在前端世界并不新鲜：它可以追溯到十多年前的 Knockout observables 和 Meteor Tracker 的实现。Vue Options API 和 React 状态管理库 MobX 也是基于相同原理，但是隐藏在对象属性背后。

尽管信号并非作为某种东西的必要特征，但今天这个概念经常与渲染模型一起讨论，其中更新通过精细的订阅进行。由于使用虚拟 DOM，Vue 目前依赖编译器来实现类似的优化。然而，Vue 还在探索一种新的受 Solid 启发的编译策略（Vapor Mode），不依赖于虚拟 DOM，并更充分利用 Vue 内置的响应性系统。

## 简单性

Vue 最大的优势在于它的简单性。使用 Vue 很容易入门：你可以简单地在 HTML 文件中使用`<script>`标签引入 Vue 库，然后开始编写 Vue 组件。Vue 还提供了一个 CLI 工具来搭建新项目，这可以是开始更复杂应用的好方法。

虽然我们在这里只是浅尝辄止 Vue.js，但很明显，Vue 强大的响应性系统、基于模板的语法和良好结构的组件模型使它成为许多开发者的一个吸引人的选择。

# Angular

Angular，由 Google 开发和维护，是 JavaScript 框架领域中另一个著名的参与者。Angular 是一个完整且具有主见的框架，为前端各种问题提供了自己的解决方案，从渲染和状态管理到路由和表单处理。

Angular 引入了与 React 不同的响应性模型。Angular 不使用虚拟 DOM 的差异化和调和过程，而是使用称为变更检测的系统。

在 Angular 中，每个组件都有一个变更检测器，使用一个名为 Zone.js 的库来检查组件视图中的变更。在我们继续之前，让我们稍微详细讨论一下这个。

## 变更检测

变更检测是 Angular 用来检查应用程序状态是否已更改以及是否需要更新任何 DOM 的过程。在高层次上，Angular 从上到下遍历组件，寻找变更。Angular 定期运行其变更检测机制，以便将数据模型的更改反映在应用程序的视图中。变更检测可以通过手动触发或通过异步事件触发。

变更检测高度优化和高性能，但如果应用程序频繁运行变更检测，仍可能导致减速。这种变更检测系统是一个强大且灵活的工具，Angular 提供了几种策略来优化其行为，以适应不同场景的性能需求。

Angular 也使用模板语法，类似于 Vue，但它提供了更强大的指令和结构来操作 DOM，比如 `*ngIf` 条件性地渲染元素和 `*ngFor` 渲染列表。这与使用 JSX 的 React 不同，后者使用内置 JavaScript 表达式来渲染动态数据。

## 信号

Angular 正在进行一些根本性的变更，放弃脏检查，并引入自己的响应性原语实现。Angular 信号 API 如下所示：

```
const count = signal(0);

count(); // access the value
count.set(1); // set new value
count.update((v) => v + 1); // update based on previous value

// mutate deep objects with same identity
const state = signal({ count: 0 });
state.mutate((o) => {
  o.count++;
});
```

与 Vue 的引用不同，Angular 的基于 getter 的 API 风格在 Vue 组件中使用时提供了一些有趣的权衡。

+   `()` 比 `.value` 略少冗长，但更新值更为冗长。

+   无需解包引用：访问值始终需要 `()`。这样可以确保在任何地方访问值的一致性。这也意味着可以将原始信号传递为组件的属性。

Angular 是一种类似瑞士军刀的工具，提供了广泛的工具来构建复杂的应用程序。它的固执己见既是其代码库一致性和结构的优点，也是新开发人员面临的学习曲线和灵活性的限制。

# Svelte

Svelte 是构建用户界面的一种激进新方法。与传统框架不同，Svelte 是一个编译器，将您的声明式组件转换为高效的命令式代码，精确地更新 DOM。因此，您可以用更少的代码编写高性能、响应式的 Web 应用程序。

Svelte 的响应模型非常简单但功能强大。在 Svelte 中，响应式语句使用简单的语法编写，这让人想起电子表格公式。这里是一个基本的 Svelte 组件：

```
<script>
let count = 0;

function increment() {
    count += 1;
}
</script>

<div>{count}</div>
<button on:click={increment}>
  Click me
</button>
```

在这个例子中，标记中的`{count}`语法将在`count`变量更改时自动更新。这类似于 React 的 JSX，但有一个关键的区别：在 Svelte 中，这种响应性是自动的。您不需要调用设置函数或使用任何特殊的 API 来更新 DOM；您只需分配给变量，Svelte 就会处理其余的工作。

Svelte 还提供了一种响应式语句语法，允许您根据您的响应式数据计算值：

```
<script>
let count = 0;
let doubleCount = 0;

$: doubleCount = count * 2;

function increment() {
    count += 1;
}
</script>

<div>{doubleCount}</div>
<button on:click={increment}>
  Click me
</button>
```

在这个例子中，当`count`变化时，`doubleCount`会自动更新。这类似于 Vue 中的计算属性，但语法可能更简单。

Svelte 采用的编译器方法具有几个优势。通常情况下，这会导致更快的运行时性能，因为没有虚拟 DOM 的差异和修补步骤。相反，Svelte 生成直接更新 DOM 的代码。

然而，这种方法也存在一些权衡。Svelte 的编译器中心性意味着一些基于虚拟 DOM 的框架提供的动态能力（如动态组件类型）可能更加笨拙或冗长地表达出来。此外，由于 Svelte 生态系统比 React、Vue 和 Angular 更小且更年轻，因此可用的资源、库和社区解决方案可能较少。

## 符文

符文是影响 Svelte 编译器的符号。虽然今天 Svelte 使用`let`、`= `、`export`关键字和`$:`标签来指代特定事物，符文则使用*函数语法*来实现相同的功能，甚至更多。

例如，要声明一个响应式状态片段，我们可以使用`$state`符文：

```
<script>
`-   let count = 0;`
+   let count = $state(0);

    function increment() {
        count += 1;
    }
</script>

<button on:click={increment}>
    clicks: {count}
</button>
```

随着应用程序复杂度的增加，弄清哪些值是响应式的，哪些不是，可能会变得棘手。当前的启发式方法仅适用于组件顶层的`let`声明，这可能会引起混淆。例如，如果您需要将某些内容转换为存储以便在多个位置使用，则在*.svelte*文件内部和*.js*文件内部的代码行为不一致可能会使重构代码变得困难。

有了符文，响应性不再局限于您的*.svelte*文件的边界之内。假设我们想要封装计数器逻辑，以便在组件之间重复使用。今天，您可以在*.js*或*.ts*文件中使用自定义存储：

```
import { writable } from "svelte/store";

export function createCounter() {
  const { subscribe, update } = writable(0);

  return {
    subscribe,
    increment: () => update((n) => n + 1),
  };
}
```

因为这实现了*存储契约*——返回值具有`subscribe`方法——我们可以通过在存储名称前加`$`来引用存储值：

```
<script>
+   import { createCounter } from './counter.js';
+
+   const counter = createCounter();
`-   let count = 0;`
`-`
`-   function increment() {`
`-       count += 1;`
`-   }`
</script>

`-<button on:click={increment}>`
`-   clicks: {count}`
+<button on:click={counter.increment}>
+   clicks: {$counter}
</button>
```

这样做虽然有效，但感觉有点奇怪！当你开始做更复杂的事情时，存储 API 可能会变得非常难以管理。而有了符文，一切变得简单得多：

```
-import { writable } from 'svelte/store';

export function createCounter() {
`-   const { subscribe, update } = writable(0);`
+   let count = $state(0);

    return {
`-       subscribe,`
`-       increment: () => update((n) => n + 1)`
+       get count() { return count },
+       increment: () => count += 1
   };
}
```

```
<script>
    import { createCounter } from './counter.js';

    const counter = createCounter();
</script>

<button on:click={counter.increment}>
`-   clicks: {$counter}`
+   clicks: {counter.count}
</button>
```

注意，我们在返回对象中使用了`get`属性，以便`counter.count`始终指向当前值，而不是函数被调用时的值。

### 运行时响应性

今天，Svelte 使用*编译时响应性*。这意味着，如果你有一些使用`$:`标签的代码，以便在依赖项更改时自动重新运行，这些依赖项在 Svelte 编译组件时确定：

```
<script>
    export let width;
    export let height;

    // the compiler knows it should recalculate `area`
    // when either `width` or `height` change...
    $: area = width * height;

    // ...and that it should log the value of `area`
    // when _it_ changes
    $: console.log(area);
</script>
```

这运作良好……直到它不工作。假设我们将代码重构如下：

```
// @errors: 7006 2304
const multiplyByHeight = (width) => width * height;
$: area = multiplyByHeight(width);
```

因为`$: area = ...`声明只能看到`width`，所以当`height`变化时不会重新计算。因此，代码很难重构，理解 Svelte 何时选择更新哪些值在一定复杂程度之后可能变得相当棘手。

Svelte 5 引入了`$derived`和`$effect`符文，它们在评估表达式时确定其依赖关系：

```
<script>
    let { width, height } = $props(); // instead of `export let`

    const area = $derived(width * height);

    $effect(() => {
        console.log(area);
    });
</script>
```

与`$state`、`$derived`和`$effect`类似，您也可以在*.js*和*.ts*文件中使用它们。

### 信号放大

与其他框架一样，Svelte 也意识到 Knockout 一直是正确的。

Svelte 5 的响应性由*信号*驱动，这本质上是 Knockout 在 2010 年所做的。最近，信号已经被 Solid（稍后详细介绍）推广，并被多个其他框架采纳。在 Svelte 5 中，信号是一个底层实现细节，而不是直接与之交互的内容。

# Solid

Solid 是一个用于构建用户界面的声明性 JavaScript 库。它类似于 React，提供了一个组件模型基础，但 Solid 基于响应式原语。Solid 不使用虚拟 DOM，而是使用细粒度的响应性系统来自动跟踪依赖关系并直接更新 DOM，这可能会导致更有效的更新。

这是一个简单的 Solid 组件示例：

```
import { createSignal } from "solid-js";

function Component() {
  const [count, setCount] = createSignal(0);

  return (
    <>
      <div>{count()}</div>
      <button onClick={() => setCount(count() + 1)}>Increment</button>
    </>
  );
}
```

在这个示例中，`createSignal`创建了一个响应式原语，类似于 React 中的`useState`。关键区别在于`count`是一个返回当前值并隐式注册依赖关系的函数。当调用`setCount`时，它会触发依赖于`count`的任何 UI 部分的更新，而不重新调用函数组件。

要与 React 对比，在 React 中，组件（在这种情况下是`Component`）会被重新调用，包括其块内的所有逻辑。因此，`count`值本身不是响应式的。在 Solid 中，`Component`函数永远不会被重新调用，但`count`值本身是响应式的，并在调用`setCount`时变化。这被称为细粒度响应性，与 React 的粗粒度响应性完全相反。

Solid 的细粒度响应性系统意味着它可以最小化不必要的更新，并避免需要差异化步骤，从而实现非常高的性能。然而，由于它是一个相对较新且使用较少的库，可能没有像一些更成熟选项那样多的资源和社区解决方案。

Solid 的`createSignal()` API 设计强调了读/写分离。信号作为只读 getter 和单独的 setter 公开：

```
const [count, setCount] = createSignal(0);

count(); // access the value
setCount(1); // update the value
```

注意 `count` 信号如何在不使用 setter 的情况下传递。这确保状态除非显式公开 setter，否则不能被修改。

Solid 重新点燃了关于信号的讨论，并且这个概念已被许多其他框架和库采纳，正如我们之前看到的。关于信号的所有前述内容都来自于 Solid 的作者 Ryan Carniato 的工作，他以某种方式凭借 2010 年的概念单枪匹马地改变了整个前端生态系统。

# Qwik

Qwik 是一个独特的框架，旨在优化 Web 页面的加载，并优先考虑用户交互和响应能力。与传统框架不同，它将 Web 页面视为可以独立通过网络加载并按需交互的组件集合。这种方法显著减少了页面的初始加载时间，提升了整体用户体验。

使用 Qwik 构建的 Web 应用和站点带有极小且恒定的初始 JavaScript（~1 kB）。Qwik 站点加载的初始 JavaScript 量是恒定的，因为它是 Qwik 加载器。这就是为什么在某些圈子中称 Qwik 为“O(1) 框架”的原因，意味着无论应用程序大小如何，它的加载时间都是恒定的。

起初，Qwik 只加载最少量的 JavaScript，然后根据需要加载组件和其他行为。这种方法使得 Qwik 能够优先加载最重要的组件，从而实现更快的初始加载时间和更响应的用户体验。

Qwik 的一个重要特性是可恢复性。我们在服务器端 React 的章节中粗略地介绍了可恢复性（第六章），但回顾一下：可恢复性是通过将页面的初始状态的服务器渲染快照发送到客户端来实现的。当用户打开页面时，他们与这个静态快照进行交互，直到他们需要更多的互动为止。然后，随着用户继续操作，各种行为按需加载。这种机制为用户提供了即时的交互机会，这是许多其他框架中不多见的特性。

可恢复性远远优于水合（也在第六章中讨论），因为它不需要两次渲染组件。它还避免了用户界面的“奇异谷”，即在初始服务器渲染标记到达浏览器并且 JavaScript 加载并水合页面之前的一段时间内，网站不具有交互性。Qwik 立即启动。

在将 Qwik 与其他流行的框架如 React、Vue、Svelte 或 Solid 进行比较时，会发现几个区别。虽然 React 和 Vue 也采用了基于组件的方法，但如果我们在代码拆分方面不小心或不有意识地进行处理，有时可能会一次性向客户端发送整个 JavaScript 捆绑包，有时会达到几兆字节。这个过程可能导致更长的初始加载时间，特别是对于大型应用程序而言。另一方面，Qwik 只在需要时加载组件和事件处理程序，从而实现更快的初始加载时间和更响应迅速的用户体验。Qwik 还精于预取，对于延迟加载的元素进行预取，使得所有内容在初始加载时都被预取，但只有在需求时才进行解析和执行。

Qwik，像 Svelte 和 Solid 一样，专注于性能，但通过不同的方式实现这一目标。Svelte 将组件编译为高效的命令式代码，直接操作 DOM，而 Solid 则使用响应式细粒度的响应模型来处理其组件。Qwik 使用响应式原语，专注于优化组件加载，确保尽快提供最重要的组件。

在开发者体验方面，Qwik 提供了一个简单直观的 API，使得定义和使用组件变得非常容易。Qwik 组件在语法和结构上几乎与 React 组件相同，因为它们也是使用 JSX（或 TSX）表示的。这种相似性使得开发者可以轻松开始使用 Qwik，特别是如果他们已经熟悉 React 的话。

此外，Qwik 与 React 具有互操作性，允许开发者通过 `qwikify` 实用程序在 Qwik 应用程序中使用 React 组件。这种互操作性对于希望使用 Qwik 但也想利用丰富的 React 库和工具生态系统的开发者来说是一个重大优势。

Qwik 通过其基于组件和事件驱动的架构提出了现代 Web 开发的新方法。它的重点在于可恢复性和优先加载，使其在与 React、Vue、Svelte 和 Solid 等其他框架相比显得与众不同。虽然每个工具都有其优势和使用案例，但 Qwik 的独特特性使其成为 Web 开发框架领域中令人兴奋的新选择。对于寻求高性能、以用户为中心和高效构建 Web 应用程序的开发者和团队来说，Qwik 可能是一个正确的选择。

Qwik 唯一的缺点是它仍然相对较新，并且没有像 React、Vue 或 Angular 那样成熟的生态系统。但是，它正在获得关注，并且拥有一个不断增长的开发者和贡献者社区。随着 Qwik 的持续发展，看到它与其他框架的比较及如何用于构建更强大的应用程序将是非常有趣的。

# 常见模式

所有这些技术——React、Angular、Qwik、Solid 和 Svelte——都是用于为 Web 创建丰富、交互式用户界面的解决方案。尽管它们在哲学、方法论和实现细节上各有不同，但它们共享几个反映它们共同目标的共同点。

## 基于组件的架构

这些框架和库之间的一个主要共同点是采用组件化架构。在组件化架构中，UI 被拆分成独立的组件，每个组件负责用户界面的特定部分。

组件封装了自己的状态和逻辑，并可以组合在一起构建复杂的 UI。这种模块化促进了代码重用、关注点分离和提高了可维护性。在这些框架中，组件可以是函数式的，并且通常可以组合、扩展或装饰以创建更复杂的组件。

## 声明式语法

React、Angular、Qwik、Solid 和 Svelte 都使用声明式语法来定义 UI。在声明式方法中，开发者指定了给定状态下 UI 的外观，框架负责更新 UI 以匹配该状态。这抽象了繁琐且容易出错的命令式 DOM 操作。

所有这些技术都提供了自己的模板语言来编写声明式 UI。React、Qwik 和 Solid 使用 JSX；Angular 使用其自己的基于 HTML 的模板语法；而 Svelte 则有其受 HTML 启发的语言。

## 更新

所有这些库和框架都提供了一种机制来响应应用程序状态的更新，并相应地更改 UI。React 和 Vue 使用虚拟 DOM 差异算法来进行这些更新。而 Svelte 则将组件编译为直接更新 DOM 的命令式代码。Angular 使用基于 Zones 和可观察对象的变更检测机制。

不久之后，几乎所有人都将使用 vDOM，而其他人将使用各种信号。

尽管方法不同，目标是相同的：在响应状态变化时有效地更新 UI，抽象出复杂的 DOM 操作，使开发者能够专注于应用程序逻辑。

## 生命周期方法

React、Angular、Solid 和 Svelte 提供了生命周期方法或钩子，这些方法在组件生命周期的不同阶段调用，比如组件首次创建、更新和即将从 DOM 中移除时。开发者可以利用这些方法来运行副作用、清理资源或根据 props 的变化进行更新。

## 生态系统与工具

这些框架和库都由丰富的工具、库和资源支持。它们都支持现代 JavaScript 特性和工具，包括 ES6 语法、模块以及 Webpack 和 Babel 等构建工具。它们还具有出色的 TypeScript 支持，允许开发者编写类型安全的代码并利用 TypeScript 强大的功能。

大多数这些技术还配备或可用复杂的开发者工具，可以帮助调试和应用程序分析。React 和 Angular 的开发者工具扩展是这类工具的绝佳例子。

虽然 React、Angular、Qwik、Solid 和 Svelte 各自有其独特的优势和理念，但它们都分享以下共同目标：提供基于组件的架构，实现声明式 UI 的创建，为状态变化提供响应性，简化事件处理，提供生命周期方法或类似概念，并支持丰富的生态系统和现代 JavaScript 工具链。这些共同的特性和概念体现了 Web 开发朝向更模块化、声明式和响应式范式的演变。

# React 不是响应式

“响应式”这个术语被用来描述编程世界中的许多事物，但通常用来描述自动根据数据变化更新的系统。响应式编程范式基本上是构建能响应变化并自动传播这些变化的系统。这就是为什么像 Vue.js 和 Svelte 这样的框架经常被描述为具有响应性。然而，React 并不遵循传统的响应式模型，它的方法是完全不同的。

React 是一个以声明方式构建用户界面的库——*声明式*意味着我们编写 React 的人只描述我们想要的*是什么*，React 处理*如何*。它允许开发者根据当前应用状态描述 UI，React 会在状态变化时更新 UI。这种描述听起来像 React 是响应式的，但当你深入了解其实现细节时，就会发现 React 的模型与传统的响应式编程模型有很大不同。

要理解为什么 React 在传统意义上不是响应式的，让我们首先看看系统中传统响应式看起来如何。在传统的响应式系统中，依赖关系在代码运行时会自动跟踪。当一个响应式依赖发生变化时，所有依赖于它的计算都会自动重新运行以反映这一变化。通常使用数据绑定、可观察对象或信号与插槽等技术来实现这一点。

例如，信号是一种可以用来创建响应式值的响应式原语：在读取时，信号的读取者订阅它，在写入时，所有订阅者都会收到通知。这就是响应式编程的基础。

React 使用了一种不同的方法来管理状态及其更新。它不会自动追踪依赖关系和传播更改，而是引入了一种更明确的机制来更新状态——`useState` 钩子。当状态发生变化时，React 不会立即进行渲染更新，而是安排重新渲染，在重新渲染期间，整个组件函数将会以新状态再次运行。

这意味着在这个计数器的情况下：

```
import React, { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  function increment() {
    setCount(count + 1);
  }

  return (
    <div>
      <p>{count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

export default Counter;
```

当调用 `setCount` 时，将重新调用 `Counter` 函数，包括 `useState` 钩子。这与传统的响应式模型不同，传统模型中只会更新 UI 的响应部分，而在这种情况下，更新的是 `<p>` 中的 `{count}`。这称为粗粒度的响应性，与信号的细粒度响应模型形成鲜明对比。

React 经常被认为符合以下等式：

```
v = f(s)
```

也就是说，视图等于其状态的函数。这个等式本身描述了 React 的非反应性本质：视图是状态的函数，但状态变化时不会自动更新视图。相反，只有当函数重新执行时，视图才会使用新状态更新。

这就是 React 的虚拟 DOM 差异化和协调过程的关键所在。当组件的状态或属性发生变化时，React 会重新渲染组件，创建一个新的虚拟 DOM 子树。然后，它会将这个新子树与旧子树进行比较，计算出需要的最小的实际 DOM 变化，并将这些变化应用到 DOM 上。

明确设置状态并重新渲染的这种模式，与自动的反应性变化传播相对，使得更加可预测：如果把 React 拟人化，它会说，“告诉我你对状态的期望，我会照顾好它。” 它支持批量状态更新等特性，并且使得在任何时刻都更容易推理应用程序的状态，因为状态更新和结果 UI 更新是一个单一的、原子性操作。

然而，这也意味着 React 组件在传统意义上不那么具有反应性。它们不会自动响应数据的变化。相反，它们明确描述了在给定状态下 UI 应该如何看起来，而 React 负责在状态变化时重新执行函数并应用任何必要的更新，而不是直接在原地更新相应的值。

虽然 React 的方法不是指自动跟踪和传播变化，但它仍然提供了一种高效的机制来构建动态、交互式的用户界面。使用状态和属性来控制渲染提供了一个清晰且可预测的模型，帮助理解变化如何在应用程序中传播，而虚拟 DOM 系统有效地管理更新到实际 DOM。

总的来说，无论 React 的方法是否被认为是响应式，最终都归结为语义。如果你定义响应性为系统中变化的自动传播，那么不，React 不是响应式的。但如果你定义响应性为系统能够以可预测和受控的方式响应状态变化，那么是的，React 确实可以被认为是响应式的。

查看 React 和其他框架/库，很明显，在 UI 开发中，没有一种大小合适的方法来管理状态和响应性。每种工具都有其优势和权衡，并适用于不同的用例。了解这些差异在选择合适的工具时至关重要，也可以帮助编写更有效和高效的代码，无论你使用的是哪种框架或库。

React 处理状态和更新的模型提供了控制和便利的极好平衡。显式状态更新机制使开发人员能更轻松地推理他们的应用状态，而协调和差异算法则高效地将更新应用到 DOM 中。尽管不是传统意义上的“响应式”，React 的方法已被证明在构建复杂用户界面时非常有效。

不可否认的是，响应式编程模型在自动管理依赖和更新方面提供了一些引人注目的好处。但正如我们所见，React 的方法提供了自己一套优势，提供了高度的控制和可预测性。

对于完美主义者来说，现在我们将看一下在 Solid 中，一个使用响应式模型的框架中相同计数器是如何表现的：

```
import { createSignal } from "solid";

function Counter() {
  const [count, setCount] = createSignal(0);

  function increment() {
    setCount(count + 1);
  }

  return (
    <div>
      <p>{count()}</p>
      <button onClick={increment()}>Increment</button>
    </div>
  );
}

export default Counter;
```

在这个例子中，`count`是组件数据的一个响应式属性。当我们通过在我们的`<p>`元素内部调用`count()`来首次读取`count`时，我们隐式订阅了`count`的响应式值的那部分 JSX。

然后，当我们稍后调用`increment()`时，这将调用`setCount`，`setCount`更新值并通知所有订阅者值已更改，促使它们进行更新。这有点类似于发布/订阅模式，其中订阅者订阅发布者，发布者通知所有订阅者。

结果是细粒度的响应性：即，函数组件本身`Counter`从未被调用超过一次，但是细粒度、小型的响应式值会被调用。

## 示例 2：依赖值

考虑一个显示项目列表和项目计数的组件。在像 Svelte 这样的响应式系统中，每当列表变化时，计数会自动更新：

```
<script>
  let items = ['Apple', 'Banana', 'Cherry'];
  $: count = items.length;
</script>

<p>{count} items:</p>
<ul>
  {#each items as item (item)}
    <li>{item}</li>
  {/each}
</ul>
```

在这里，`$: count = items.length;` 声明了一个响应式语句。每当 `items` 发生变化时，`count` 会自动重新计算。

在 React 中，这看起来有些不同：

```
import React, { useState } from "react";

function ItemList() {
  const [items, setItems] = useState(["Apple", "Banana", "Cherry"]);
  const count = items.length;

  // ... update items somewhere ...

  return (
    <div>
      <p>{count} items:</p>
      <ul>
        {items.map((item) => (
          <li key={item}>{item}</li>
        ))}
      </ul>
    </div>
  );
}

export default ItemList;
```

在这个 React 组件中，`count` 不是一个在`items`改变时自动更新的响应式值。相反，它是在渲染阶段从当前状态派生出来的值。当`items`改变时，我们需要调用`setItems`来更新状态并引起重新渲染，在这个时候`count`被重新计算，不是因为`count`是响应式的，而是因为`ItemList`函数组件被重新调用。

# React 的未来

鉴于像信号这样的响应式原语在整个前端生态系统中的广泛采用，一些人可能会认为 React 最终会采纳类似的方法。然而，React 团队表示他们对信号“不感兴趣”，并选择了另一种方法来实现信号提供的类似性能优势。

为了更好地理解这一点，让我们通过一个例子回顾一些我们通过 React 学到的东西。考虑这个组件：

```
import React, { useState } from "react";
import
{ ComponentWithExpensiveChildren } from "./ExpensiveComponent";

function Counter() {
  const [count, setCount] = useState(0);

  function increment() {
    setCount(count + 1);
  }

  return (
    <div>
      <p>{count}</p>
      <button onClick={increment}>Increment</button>
      <ComponentWithExpensiveChildren />
    </div>
  );
}

export default Counter;
```

在这个非常，非常构造的例子中，我们有一个包含名为`Counter`状态的组件，带有一些子元素：

+   一个 `<p>` 元素，显示当前计数

+   一个 `<button>` 元素，用来增加计数

+   一个 `<ComponentWithExpensiveChildren>` 组件，渲染一些计算量大的昂贵子元素

现在，假设我们点击按钮增加计数。会发生什么？`Counter`函数会被调用/重新调用/重新渲染以及其所有的子元素。这是 React 的默认行为。这意味着即使其 props 或状态没有改变，`<ComponentWithExpensiveChildren>`组件也会重新渲染！

这种粗粒度的响应性使得 React 的性能比可能的更低。然而，这是一个相当容易的修复：我们只需在正确的时间和地点包含`memo`：

```
import React, { useState, memo } from "react";
import { ComponentWithExpensiveChildren } from "./ComponentWithExpensiveChildren";

function Counter() {
  const [count, setCount] = useState(0);

  function increment() {
    setCount(count + 1);
  }

  return (
    <div>
      <p>{count}</p>
      <button onClick={increment}>Increment</button>
      <MemoizedComponentWithExpensiveChildren />
    </div>
  );
}

const MemoizedComponentWithExpensiveChildren = memo(
  ComponentWithExpensiveChildren
);

export default Counter;
```

只要我们记得在需要的地方使用`memo`，这就能正常工作。实际上，这提供了与信号相同的细粒度响应性。但它不像信号那样方便，因为我们必须记得在需要的地方使用`memo`。

我们中的许多人可能认为，信号很容易解决这个问题，但 Meta 的 React 团队认为信号和`memo`可能是日常使用 React 的开发人员不必考虑的实现细节。他们回顾了 React 的最初价值主张：“声明性地描述你的 UI，让 React 去处理剩下的事情。” React 团队认为，更优越的方式是开发者不需要关心信号、`memo`或任何细节，而是让 React 能够找出渲染 UI 的最佳方式。

为此，团队正在开发一个新的软件来做到这一点：React Forget。

## React Forget

Forget 是一个针对 React 的工具链，类似于启用了其`--fix`标志的代码检查工具：它强制执行 React 的规则，然后通过智能记忆那些在应用程序生命周期中不会改变的值来自动优化 React 代码，例如`ComponentWithExpensiveChildren`。

由于 React 的这些规则，Forget 编译器可以预测这些值并为我们进行记忆化。这与 Svelte 的做法类似，但 Forget 编译的是更高性能的 React 代码，而不是编译为命令式代码。

这些 React 的规则是什么？让我们回顾一下：

1.  React 组件应该是纯函数。

1.  一些钩子和自定义事件处理程序不需要是纯函数。

1.  纯函数中禁止的操作包括：

    +   在函数内部禁止变异变量/对象而不是新创建的。

    +   读取可能会改变的属性。

1.  允许的操作包括：

    +   读取 props 或 state

    +   抛出错误

    +   变异新创建的对象/绑定。

1.  惰性初始化是一个例外，允许为初始化目的进行变异。

1.  渲染期间创建的对象或闭包在渲染完成后不应变异，除了存储在状态中的可变对象。

正是由于这些 React 的规则，Forget 编译器可以预测应用程序生命周期中不会改变的值，并为我们进行记忆化。结果是？高度优化、性能卓越的 React 代码，可以与使用信号的其他库的性能相媲美。

在撰写本文时，Forget 在 Meta 公司正在评估中，并且在 Instagram 和 WhatsApp 上的使用超出了预期。它尚未开源，但 React 团队正在考虑在不久的将来发布它作为开源软件。

### Forget 与信号的对比

因为 Forget 尚未开源，所以很难以权威的方式评论它的权衡。然而，我们可以推测，如果 Forget 确实对所有不会改变的内容进行了记忆化，那么信号的细粒度反应性可能仍然优于 React Forget 的粗粒度反应性，因为信号存在于组件层次结构之外的平行宇宙。

因此，当更新发生时，React 仍然需要遍历整个组件树，并比较每个组件的 props 的新旧值，以确定哪些组件需要重新渲染。这在信号中并非如此，其中仅更新 UI 的响应部分而无需遍历树。初步数据表明，即使使用了 Forget，React 仍可能比默认使用信号的库慢，但现在还为时过早。

# 章节回顾

本章开始时回顾了第九章，我们详细讨论了 RSCs 的使用。然后我们深入探讨了 JavaScript 框架的广阔领域，超越了 React，包括 Angular、Vue、Svelte、Solid 和 Qwik，旨在理解这些库和框架之间的差异和相似之处。

我们从 Vue.js 入手，探讨了它如何采用声明式方法构建 UI，并通过其基于组件的架构促进了关注点的强分离。

接下来，我们深入研究了 Angular、Svelte、Solid 和 Qwik，探讨它们的独特特性和理念。我们看了它们如何使用响应式基元自动更新 UI，以响应数据变化，并探讨了它们在这方面与 React 的不同之处。

在个别考察之后，我们对这些 UI 库进行了比较，突出它们的优势、劣势和重叠之处。我们关注它们的响应模型、架构选择、开发体验和性能特征。通过代码示例，我们展示了每个库的独特特性，帮助我们更好地理解它们的差异。

我们还研究了响应性的概念以及在不同库中如何实现不同。有趣的是，我们讨论了 React 不同于传统意义上的响应式，因为它遵循一种更粗粒度的方法，其中状态变化会导致重新渲染，而不是像 Vue 或 Svelte 这样的库中的细粒度响应模型。

最后，我们看了一眼 React 未来的发展方向以及它未来几年可能的演变。我们讨论了 React 团队对响应性的方法，以及它与传统响应式编程模型的不同之处。我们还看了一下 Forget 编译器，这是一个用于 React 的工具链，通过记忆那些在应用程序生命周期中不变的值来自动优化 React 代码。

最后，让我们把飞机安全降落。

# 复习问题

这里有一些问题列表，帮助您跟踪本章讨论的概念理解程度。如果您能自信地回答所有问题，那很好！这表明您正在从这本书中学到东西。如果不能，可能值得重新阅读本章。

1.  React、Vue、Svelte、Solid 和 Angular 之间的响应模型有何不同？这些差异对这些库/框架的性能和开发体验有何影响？

1.  讨论 Qwik 在最大化性能方面的独特方法。这与我们讨论过的其他 UI 库/框架的方法有何不同？

1.  在本章讨论的每个 UI 库/框架中，它们各自的核心优势和劣势是什么？这些优势和劣势如何影响选择某个项目的库/框架？

1.  React 在传统意义上并非是响应式的。详细解释这个说法，并与像 Vue 或 Svelte 这样的“推送型”响应模型进行比较。

1.  什么是 React Forget？它如何工作？它与信号（signals）相比如何？

# 接下来

当我们接近总结对 React 及其生态系统的全面探索时，我们准备综合我们所学的一切。在下一章，也是最后一章中，我们将退后一步，反思整个局势。

我们将总结本书内容，全面展示我们今天的进展及明日的预期。在此过程中，我们将汲取本书中积累的所有技术知识和见解。

从理解 React 协调器的内部工作方式和深入异步性，到处理服务器组件和理解各种 React 框架，再到与同行比较 React —— 所有这些都有其目的。现在，我们准备好串联各点，看到更大的画面，并规划前进的道路。

那么，你准备好迈入 React 和前端开发的未来了吗？敬请期待壮丽的结局！
