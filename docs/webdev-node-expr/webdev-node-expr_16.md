# 第十六章：单页应用

“单页应用”（SPA）这个术语在某种程度上有点误导，或者至少混淆了“页面”这个词的两种含义。从用户的角度来看，SPA 可以（而且通常会）看起来像有不同页面：主页、度假页面、关于页面等。事实上，你可以创建一个对用户来说无法区分的传统服务器端渲染应用程序和 SPA。

“单页”更多地涉及 HTML 如何以及在哪里构建，而不是用户的体验。在 SPA 中，当用户首次加载应用时，服务器会提供一个单个的 HTML 捆绑包，^(1) UI 中的任何变化（对用户来说可能看起来像不同的页面）都是 JavaScript 响应用户活动或网络事件而操作 DOM 的结果。

SPA 仍然需要频繁地与服务器通信，但通常只有 HTML 作为第一次请求的一部分发送。之后，客户端和服务器之间仅传输 JSON 数据和静态资源。

理解这种现在主导的 Web 应用程序开发方法需要一点历史......

# 网页应用程序开发的简短历史。

我们在过去的 10 年里对待 Web 开发的方式发生了巨大的变化，但有一件事情保持相对一致：网站或 Web 应用程序中涉及的组件。具体来说：

+   HTML 和文档对象模型（DOM）。

+   JavaScript

+   CSS。

+   静态资源（通常是多媒体：图像和视频等）。

由浏览器组合在一起的这些组件构成了用户体验。

然而，这种体验是如何构建的在 2012 年左右发生了显著变化。如今，Web 开发的主导范式是“单页应用”（SPA）。

要理解单页应用（SPA），我们需要了解与之对比的内容，因此我们将回溯到更早的时间，即 1998 年，这是“Web 2.0”一词首次被提出之前的一年，也是 jQuery 推出八年之前。

在 1998 年，交付 Web 应用程序的主流方法是在每次请求时由 Web 服务器发送 HTML、CSS、JavaScript 和多媒体资源。想象一下你在看电视，想换频道。在这里的比喻相当于你需要扔掉你的电视，买另一个，搬到家里并设置它——只是为了换台（即切换到不同页面，即使是同一站点内部）。

这种方法的问题在于涉及大量的开销。有时 HTML——或者大部分 HTML——根本不会改变。CSS 的变化更少。浏览器通过缓存资源来缓解一些开销，但 Web 应用程序的创新速度正在使这种模型变得紧张。

1999 年，"Web 2.0"这个术语被创造出来，试图描述人们开始期望从网站上获得的丰富体验。1999 年至 2012 年间，技术进步为单页面应用奠定了基础。

聪明的 Web 开发人员开始意识到，如果他们要让用户保持参与，那么每次用户想要（比喻性地）切换频道时，发送整个网站的开销都是不可接受的。这些开发人员意识到，并非应用程序中的每个变化都需要从服务器获取信息，而且并非每个需要从服务器获取信息的变化都需要整个应用程序才能实现小变化的交付。

在这个 1999 年至 2012 年的时期，页面仍然通常是页面：当您第一次访问网站时，您得到的是 HTML、CSS 和静态资产。当您导航到另一个页面时，您会得到不同的 HTML、不同的静态资产，有时候还有不同的 CSS。然而，每个页面上，页面本身可能会因用户交互而改变，而不是向服务器请求全新的应用，JavaScript 会直接改变 DOM。如果需要从服务器获取信息，这些信息会以 XML 或 JSON 的形式发送，没有所有相关的 HTML。再次说起，需要 JavaScript 解释数据并相应地更改用户界面。2006 年，jQuery 被引入，大大减轻了 DOM 操作的负担*和*处理网络请求。

这些变化中的许多是由于计算机和——由此推断——浏览器的增强能力，Web 开发人员发现要使网站或 Web 应用程序看起来更漂亮，越来越多的工作可以直接在用户的计算机上完成，而不是在服务器上完成，然后发送给用户。

这种方法的转变在 21 世纪初进入了高速发展阶段，当时智能手机开始引入。现在，不仅浏览器能够做更多的事情，而且人们希望在*无线网络*上访问 Web 应用程序。突然之间，发送数据的开销上升，使尽可能少地通过网络发送数据变得更有吸引力，让浏览器尽可能多地完成工作。

到 2012 年，试图通过网络发送尽可能少的信息，并在浏览器中尽可能多地进行操作成为常见做法。就像原始汤浓缩成了第一个生命体一样，这种丰富的环境为这种技术的自然演化提供了条件：单页面应用。

这个想法很简单：对于任何给定的 web 应用程序，HTML、JavaScript 和 CSS（如果有的话）只需要*发送一次*。一旦浏览器得到了 HTML，就由 JavaScript 来对 DOM 进行所有更改，使用户感觉自己正在导航到一个不同的页面。例如，当用户从主页导航到"假期"页面时，服务器不再需要发送不同的 HTML。

当然，服务器仍然参与其中：它仍然负责提供最新的数据，并在多用户应用程序中作为“单一真相来源”。但在 SPA 架构中，应用程序对用户的呈现方式不再是服务器的关注点：这是 JavaScript 和支持这一巧妙幻觉的框架的关注点。

尽管 Angular 通常被认为是第一个 SPA 框架，但现在已经有许多其他框架加入其中：React、Vue 和 Ember 是 Angular 竞争中最突出的几个。

如果你是新手开发者，SPA 可能是你唯一的参考框架，使这些内容仅仅是一些有趣的历史。但如果你是老手，可能会觉得这种转变令人困惑和不适应。无论你属于哪一类，本章旨在帮助你理解 Web 应用如何以 SPA 形式提供，并且 Express 在其中的角色。

这段历史对 Express 很重要，因为在 Web 开发技术转变期间，服务器的角色发生了变化。当本书的第一版出版时，Express 还常用于提供多页面应用程序（以及支持类似 Web 2.0 功能的 API）。现在，Express 几乎完全用于提供 SPA、开发服务器和 API，反映了 Web 开发性质的变化。

有趣的是，仍然有有效的理由让 Web 应用程序能够提供特定页面（而不是由浏览器重新格式化的“通用”页面）。虽然这看起来可能是一个全面的循环，或者是丢弃 SPA 的收益，但这种技术更好地反映了 SPA 的架构。称为*服务器端渲染*（SSR）的这种技术允许服务器使用与浏览器相同的代码来创建单个页面，以增强首次页面加载体验。关键在于服务器不需要做太多思考：它只需使用与浏览器相同的技术来生成特定页面。这种 SSR 通常用于增强首次页面加载体验和支持搜索引擎优化。这是一个更高级的主题，我们在这里不会详细讨论，但你应该了解这种实践的存在。

现在我们对 SPA 的起源及其原因有了一些了解，让我们来看看当前可用的 SPA 框架。

# SPA 技术

现在有许多选择的 SPA 技术：

React

目前，React 似乎是单页应用（SPA）领域的霸主，尽管它的一侧有昔日的巨头（Angular），另一侧有雄心勃勃的挑战者（Vue）。在 2018 年某个时刻，React 在使用统计上超过了 Angular。React 是一个开源库，但它起源于 Facebook 项目，而且 Facebook 仍然是其活跃的贡献者。我们将在 Meadowlark Travel 的重构中使用 React。

Angular

据大多数人说，作为“原始”的单页应用程序（SPA），Google 的 Angular 变得非常流行，但最终被 React 所取代。在 2014 年末，Angular 宣布了第 2 版，这是与第一版相比的巨大变化，使许多现有用户感到陌生，也吓跑了新用户。我相信这种转变（虽然可能是必要的）促使 React 最终超越了 Angular。另一个原因是 Angular 比 React 更庞大的框架。这既有优点也有缺点：Angular 提供了一个更完整的构建全应用程序的架构，始终有一个明确的“Angular 方式”去做事，而像 React 和 Vue 这样的框架则更多地由个人选择和创造力决定。无论哪种方式更好，更大的框架更笨重且演进缓慢，这给了 React 创新的优势。

Vue.js

作为 React 的一个新兴挑战者，由单一开发者 Evan You 的心血结晶。在非常短的时间内，它已经获得了令人印象深刻的追随者，其粉丝们非常喜欢它，但它仍然远远落后于 React 的独占鳌头。我对 Vue 有一些经验，我欣赏它清晰的文档和轻量级的方法，但我更喜欢 React 的架构和理念。

Ember

像 Angular 一样，Ember 提供了一个全面的应用程序框架。有一个庞大且活跃的开发社区，虽然不像 React 或 Vue 那样创新，但提供了很多功能和清晰度。我发现我更喜欢更轻量的框架，因此一直使用 React。

Polymer

我对 Polymer 没有经验，但它由 Google 支持，这使它具备了可信度。人们似乎对 Polymer 带来了什么很感兴趣，但我还没有看到有很多人急于采用它。

如果您正在寻找一个强大的开箱即用的框架，并且不介意在规定范围内操作，您应该考虑 Angular 或 Ember。如果您希望有创造性表达和创新的空间，我推荐使用 React 或 Vue。我还不知道 Polymer 的定位，但值得关注。

现在我们已经看到了这些竞争者，让我们继续使用 React，并将 Meadowlark Travel 重构为 SPA！

# 创建 React 应用程序

用 React 应用程序的最佳方式是使用 `create-react-app`（CRA）实用工具，它创建所有样板文件、开发工具，并提供一个最小的起始应用程序，供您构建。此外，`create-react-app` 将保持其配置最新化，因此您可以专注于构建应用程序，而不是框架工具。尽管如此，如果您需要配置工具的时候，可以“eject”应用程序：您将失去保持最新 CRA 工具的能力，但将完全控制应用程序的所有配置。

与迄今为止我们所做的不同，其中所有应用程序构件与我们的 Express 应用程序并排存在不同，SPA 最好被视为一个完全独立的应用程序。为此，我们将拥有*两个*应用程序根目录而不是一个。为了清晰起见，当我提到您的 Express 应用程序所在的目录时，我将称之为*服务器根*，而当我提到您的 React 应用程序所在的目录时，我将称之为*客户端根*。*应用程序根*是现在这两个目录都位于其中的位置。

所以，进入您的应用程序根目录并创建一个名为*server*的目录；这将是您的 Express 服务器所在的位置。不要为客户端应用程序创建一个目录；CRA 将为我们完成这项工作。

在运行 CRA 之前，我们应该安装[Yarn](https://yarnpkg.com)。Yarn 是一个像 npm 一样的包管理器...实际上，yarn 在很大程度上是 npm 的替代品。对于 React 开发并非强制使用，但它是事实上的标准，不使用它将会很麻烦。在 Yarn 和 npm 的使用方式之间有一些细微差别，但您可能唯一会注意到的是，您应该运行`yarn add`而不是`npm install`。要安装 Yarn，只需按照[Yarn 安装说明](http://bit.ly/2xHZ2Cx)操作即可。

安装了 Yarn 后，请从应用程序根目录运行以下命令：

```
yarn create react-app client
```

现在进入您的客户端目录并输入`yarn start`。几秒钟后，您将看到一个新的浏览器窗口弹出，其中运行着您的 React 应用程序！

不要关闭终端窗口。CRA 对“热重载”支持非常好，因此当您在源代码中进行更改时，它会快速构建，*非常*迅速，浏览器将自动重新加载。

# React 基础

React 有出色的文档，这里不再重复。因此，如果您是 React 的新手，请从[React 入门](http://bit.ly/36VdKUq)教程开始，然后参阅[主要概念](http://bit.ly/2KgT939)指南。

您将发现 React 围绕*组件*组织，这些组件是 React 的主要构建块。用户看到或与之交互的所有内容通常都是 React 中的组件。让我们来看看*client/src/App.js*（您的内容可能略有不同——CRA 随时间变化）：

```
import React from 'react';
import logo from './logo.svg';
import './App.css';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
      </header>
    </div>
  );
}

export default App;
```

React 的一个核心概念是 UI 由*函数*生成。最简单的 React 组件只是返回 HTML 的函数，如我们在这里看到的。您可能正在看这个并认为它不是有效的 JavaScript；它看起来像混合了 HTML！事实上，情况稍微复杂一些。React 默认启用一种称为 JSX 的 JavaScript 超集。JSX 允许您编写看起来像 HTML 的内容。它不是*真正的*HTML；它创建 React 元素，而 React 元素的目的是（最终）对应于 DOM 元素。

但归根结底，您可以将其视为 HTML。这里，`App`是一个函数，将呈现与其返回的 JSX 相对应的 HTML。

需要注意的几点：由于 JSX 与 HTML 接近但不完全相同，存在一些细微的差异。您可能已经注意到我们使用`className`而不是`class`，这是因为`class`是 JavaScript 中的保留字。

要指定 HTML，你只需在任何表达式预期的地方启动 HTML 元素。你还可以在 HTML 中使用花括号“回到”JavaScript。例如：

```
const value = Math.floor(Math.random()*6) + 1
const html = <div>You rolled a {value}!</div>
```

在此示例中，`<div>`开始 HTML，并且围绕`value`的花括号回到 JavaScript，以提供存储在`value`中的数字。我们也可以轻松地内联计算：

```
const html = <div>You rolled a {Math.floor(Math.random()*6) + 1}!</div>
```

任何有效的 JavaScript 表达式都可以包含在 JSX 中的花括号内，包括其他 HTML 元素！这种常见用例是渲染列表：

```
const colors = ['red', 'green', 'blue']
const html = (
  <ul>
    {colors.map(color =>
      <li key={color}>{color}</li>
    )}
  </ul>
)
```

关于这个例子要注意的几点事项。首先，请注意，我们映射了我们的颜色来返回`<li>`元素。这是至关重要的：JSX 完全通过评估*表达式*来工作。因此，`<ul>`必须包含一个表达式或表达式数组。如果您将`map`更改为`forEach`，您会发现`<li>`元素不会被渲染。其次，请注意，`<li>`元素接收一个名为`key`的属性：这是性能的妥协。为了让 React 知道何时重新渲染数组中的元素，它需要每个元素的唯一键。由于我们的数组元素是唯一的，我们只使用了该值，但通常您会使用 ID 或者如果没有其他可用选项，数组中项目的索引。

我鼓励您在移动之前在*client/src/App.js*中尝试一些这些 JSX 示例。如果您保持`yarn start`运行，每次保存更改时，它们都会自动反映在浏览器中，这应该加快您的学习周期。

在我们从 React 基础知识中继续之前，还有一个主题需要涉及，那就是*状态*。每个组件都可以有自己的状态，这基本上意味着“与组件相关联的可以改变的数据”。购物车就是一个很好的例子。购物车组件的状态将包含一个项目列表；当您向购物车添加和删除项目时，组件的状态会发生变化。这似乎是一个过于简单或显而易见的概念，但是设计和管理组件状态的细节是制作 React 应用程序的大部分内容。当我们处理假期页面时，我们将看到状态的一个示例。

让我们继续并创建我们的 Meadowlark Travel 首页。

## 主页

从我们的 Handlebars 视图中回想起，我们有一个主要的“布局”文件，建立了我们网站的主要外观和感觉。让我们首先关注`<body>`标签中的内容（除了脚本）：

```
<div class="container">
  <header>
    <h1>Meadowlark Travel</h1>
    <a href="/"><img src="/img/logo.png" alt="Meadowlark Travel Logo"></a>
  </header>
  {{{body}}}
</div>
```

这将很容易重构为一个 React 组件。首先，我们将自己的 logo 复制到 *client/src* 目录下。为什么不放在 *public* 目录下？对于小型或常用的图形项，将它们嵌入 JavaScript 打包文件可能更有效，而 CRA 提供的打包工具会智能地做出选择。您从 CRA 获取的示例应用程序直接将其 logo 放在 *client/src* 目录下，但我仍然喜欢将图像资源收集到子目录中，因此将我们的 logo (*logo.png*) 放在 *client/src/img/logo.png* 中。

另一个棘手的地方是怎么处理 `{{{body}}}`？在我们的视图中，这是另一个视图将被呈现的地方——您所在页面的内容。我们可以在 React 中复制相同的基本思想。由于所有内容都以组件形式呈现，我们只需在此处呈现另一个组件。我们将从一个空的 Home 组件开始，并立即构建它：

```
import React from 'react'
import logo from './img/logo.png'
import './App.css'

function Home() {
  return (<i>coming soon</i>)
}

function App() {
  return (
    <div className="container">
      <header>
        <h1>Meadowlark Travel</h1>
        <img src={logo} alt="Meadowlark Travel Logo" />
      </header>
      <Home />
    </div>
  )
}

export default App
```

我们正在使用与样本应用程序相同的方法处理 CSS：我们可以简单地创建一个 CSS 文件并导入它。因此，我们可以编辑该文件并应用所需的任何样式。尽管在这个示例中我们保持基本设置，但在用 CSS 样式化 HTML 方面并没有根本性的变化，因此我们仍然拥有我们习惯使用的所有工具。

###### 注意

CRA 为您设置了 linting，在本章的进行过程中，您可能会看到警告（在 CRA 终端输出和浏览器的 JavaScript 控制台中都有）。这仅是因为我们逐步添加东西；到达本章末尾时，应该不会再有警告……如果有的话，请确保没有漏掉任何步骤！您还可以检查伴随的存储库。

## 路由

我们在 第十四章 中学到的路由的核心概念没有改变：我们仍然使用 URL 路径来确定用户看到界面的哪一部分。不同之处在于，由客户端应用程序负责处理这一点。根据路由更改 UI 是客户端应用程序的责任：如果导航需要来自服务器的新数据或更新数据，那很好，客户端应用程序负责从服务器请求。

对于在 React 应用中进行路由，有很多选择，以及很多关于此的强烈意见。然而，有一个主要的路由库：[React Router](http://bit.ly/32GvAXK)。我对 React Router 不太满意的地方很多，但它如此普遍，您肯定会遇到它。此外，它是一个很好的选择来快速启动基本项目，出于这两个原因，我们将在这里使用它。

我们将安装 React Router 的 DOM 版本开始（还有一个适用于 React Native 的版本，用于移动开发）：

```
yarn add react-router-dom
```

现在我们将连接路由器，并添加关于页面和未找到页面。我们还将把站点标志链接回主页：

```
import React from 'react'
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Link
} from 'react-router-dom'
import logo from './img/logo.png'
import './App.css'

function Home() {
  return (
    <div>
      <h2>Welcome to Meadowlark Travel</h2>
      <p>Check out our "<Link to="/about">About</Link>" page!</p>
    </div>
  )
}

function About() {
  return (<i>coming soon</i>)
}

function NotFound() {
  return (<i>Not Found</i>)
}

function App() {
  return (
    <Router>
      <div className="container">
        <header>
          <h1>Meadowlark Travel</h1>
          <Link to="/"><img src={logo} alt="Meadowlark Travel Logo" /></Link>
        </header>
        <Switch>
          <Route path="/" exact component={Home} />
          <Route path="/about" exact component={About} />
          <Route component={NotFound} />
        </Switch>
      </div>
    </Router>
  )
}

export default App
```

首先要注意的是，我们将整个应用程序包装在 `<Router>` 组件中。这是启用路由的关键。在 `<Router>` 内部，我们可以使用 `<Route>` 根据 URL 路径条件性地渲染组件。我们将内容路由放在 `<Switch>` 组件中：这确保其中包含的组件只会*一个*被渲染。

在 Express 和 React Router 中，我们完成的路由有一些微妙的差异。在 Express 中，我们会根据第一个成功匹配的路径来渲染页面（或者如果找不到则显示 404 页面）。而在 React Router 中，路径只是一个“提示”，用于确定应该显示哪些组件的组合。因此，它比在 Express 中的路由更加灵活。由于这一点，React Router 的路由默认行为就好像在路径末尾有一个星号 (`*`)。也就是说，默认情况下，路径 `/` 将匹配 *每个* 页面（因为它们都以斜杠开头）。因此，我们使用 `exact` 属性来使此路由行为更像是 Express 中的路由。类似地，如果没有 `exact` 属性，`/about` 路径也会匹配 `/about/contact`，这可能不是我们想要的结果。对于主要内容路由，很可能希望除了“未找到”路由之外，所有路由都使用 `exact`。否则，您需要确保在 `<Switch>` 内正确排列它们，以便以正确的顺序匹配。

第二点需要注意的是使用 `<Link>`。您可能会想知道为什么我们不直接使用 `<a>` 标签。`<a>` 标签的问题在于——即使在同一网站上，如果没有额外的工作，浏览器仍会如实处理它们“跳转到其他地方”，并且会导致向服务器发送新的 HTTP 请求……从而再次下载 HTML 和 CSS，破坏了单页面应用程序的目标。它会在页面加载时*工作*，React Router 会按预期执行正确的操作，但速度和效率不如预期，会引发不必要的网络请求。实际上，看到差异是一个有益的练习，可以帮助理解单页面应用程序的本质。作为一个实验，创建两个导航元素，一个使用 `<Link>`，另一个使用 `<a>`：

```
<Link to="/">Home (SPA)</Link>
<a href="/">Home (reload)</Link>
```

然后打开您的开发工具，切换到网络选项卡，清除流量，点击“保留日志”（在 Chrome 中）。现在点击“Home (SPA)”链接，并注意根本没有网络流量。点击“Home (reload)”链接，观察网络流量。简言之，这就是单页面应用程序的本质。

## 度假页面—视觉设计

到目前为止，我们只是在构建纯前端应用程序……那么 Express 又是如何介入的呢？我们的服务器仍然是真实数据的唯一来源。特别是，它维护我们希望在网站上显示的度假信息数据库。幸运的是，在第十五章中，我们已经完成了大部分工作：我们公开了一个可以返回 JSON 格式度假信息的 API，已准备好在 React 应用程序中使用。

不过，在我们连接这两件事之前，让我们继续构建我们的假期页面。我们不会有任何假期可渲染，但让我们不要因此而停下来。

在前面的部分中，我们在*client/src/App.js*中包含了所有内容页面，这通常被认为是不好的做法：每个组件通常应该存在于自己的文件中。因此，我们将花时间将我们的`Vacations`组件分离出来成为自己的组件。创建文件*client/src/Vacations.js*：

```
import React, { useState, useEffect } from 'react'
import { Link } from 'react-router-dom'

function Vacations() {
  const [vacations, setVacations] = useState([])
  return (
    <>
      <h2>Vacations</h2>
      <div className="vacations">
        {vacations.map(vacation =>
          <div key={vacation.sku}>
            <h3>{vacation.name}</h3>
            <p>{vacation.description}</p>
            <span className="price">{vacation.price}</span>
          </div>
        )}
      </div>
    </>
  )
}

export default Vacations
```

到目前为止，我们的做法相当简单：我们只是返回一个包含额外`<div>`元素的`<div>`，每个元素表示一个假期。那么这个`vacations`变量是从哪里来的呢？在这个示例中，我们使用了 React 的一个新特性，称为*React hooks*。在使用 hooks 之前，如果一个组件想要有自己的状态（在本例中是假期列表），你必须使用类实现。Hooks 使我们能够有自己状态的基于函数的组件。在我们的`Vacations`函数中，我们调用`useState`来设置我们的状态。注意，我们将一个空数组传递给`useState`：这将是状态中`vacations`的初始值（我们稍后将讨论如何填充它）。`setState`返回的是一个包含状态值本身（`vacations`）和更新状态的方法（`setVacations`）的数组。

也许你会想为什么我们不能直接修改`vacations`：它只是一个数组，所以我们不能调用`push`来添加假期吗？我们可以，但这将违背 React 状态管理系统的初衷，该系统确保组件之间的一致性、性能和通信。

您可能还想知道我们的假期周围看起来像一个空组件(`<>...</>`)的情况。这被称为[*fragment*](http://bit.ly/2ryneVj)。片段是必需的，因为每个组件必须渲染一个单一元素。在我们的情况下，我们有两个元素，`<h2>`和`<div>`。片段简单地提供了一个“透明”的根元素，用于包含这两个元素，以便我们可以渲染一个单一元素。

让我们将我们的`Vacations`组件添加到我们的应用程序中，即使现在还没有任何假期可显示。在*client/src/App.js*中首先导入您的假期页面：

```
import Vacations from './Vacations'
```

然后我们只需在我们路由器的`<Switch>`组件中创建一个路由：

```
<Switch>
  <Route path="/" exact component={Home} />
  <Route path="/about" exact component={About} />
  <Route path="/vacations" exact component={Vacations} />
  <Route component={NotFound} />
</Switch>
```

继续保存；您的应用程序应该会自动重新加载，您可以导航到*/vacations*页面，尽管现在还没有太多有趣的内容可见。现在我们已经大部分客户端基础设施就位，让我们转向与 Express 集成。

## 假期页面——服务器集成

我们已经完成了大部分假期页面所需的工作；我们有一个从数据库获取假期并以 JSON 格式返回它们的 API 端点。现在我们需要弄清楚如何使服务器和客户端进行通信。

我们可以从第十五章开始工作；我们不需要添加任何内容，但我们可以删除一些我们不再需要的东西。我们可以移除以下内容：

+   Handlebars 和视图支持（尽管我们会保留静态中间件，原因稍后会看到）。

+   Cookies 和 sessions（我们的 SPA 可能仍然使用 cookies，但它在这里不再需要服务器的帮助……而我们会以完全不同的方式思考 sessions）。

+   所有渲染视图的路由（显然我们保留 API 路由）。

现在我们留下了一个简化的服务器。那么现在我们该怎么处理它呢？我们首先要解决的问题是我们一直在使用 3000 端口，而 CRA 开发服务器默认也使用 3000 端口。我们可以任意更改其中一个，所以我建议将 Express 端口更改为 3033—只是因为我喜欢那个数字的音调。你会记得我们在*meadowlark.js*中设置了默认端口，所以我们只需要改变它：

```
const port = process.env.PORT || 3033
```

当然，我们可以使用环境变量来控制它，但由于我们经常与 SPA 开发服务器一起使用它，我们可能会改变代码。

现在两个服务器都在运行，我们可以在它们之间通信。但是怎么做呢？在我们的 React 应用中，我们可以做类似这样的事情：

```
fetch('http://localhost:3033/api/vacations')
```

这种方法的问题在于我们将在整个应用程序中频繁使用这样的请求……现在我们到处都嵌入了`[*http://localhost:3033*](http://localhost:3033)`，这在生产环境中行不通，可能在同事的电脑上也不行，因为可能需要使用不同的端口，测试服务器的端口可能也不同……这种方法会带来配置上的麻烦。当然，你可以将基础 URL 存储为一个变量，然后在所有地方使用它，但有更好的方法。

在理想情况下，从你的应用程序的角度来看，所有内容都是从同一个地方托管的：使用相同的协议、主机和端口获取 HTML、静态资产和 API。这简化了很多事情，并确保了源代码的一致性。如果所有内容都来自同一个地方，你可以简单地省略协议、主机和端口，只需调用`fetch(*/api/vacations*)`。这是一个很好的方法，幸运的是非常容易实现！

CRA 的配置支持*proxy*，允许你将 web 请求传递给你的 API。编辑你的*client/package.json*文件，并添加以下内容：

```
"proxy": "http://localhost:3033",
```

它添加在哪里并不重要。我通常将它放在`"private"`和`"dependencies"`之间，因为我喜欢在文件中尽量靠前看到它。现在，只要你的 Express 服务器运行在 3033 端口上，你的 CRA 开发服务器将通过 API 请求传递到你的 Express 服务器。

现在配置完成，让我们使用*effect*（另一个 React hook）来获取并更新假期数据。这里是整个`Vacations`组件与`useEffect` hook：

```
function Vacations() {
  // set up state
  const [vacations, setVacations] = useState([])

  // fetch initial data
  useEffect(() => {
    fetch('/api/vacations')
      .then(res => res.json())
      .then(setVacations)
  }, [])

  return (
    <>
      <h2>Vacations</h2>
      <div className="vacations">
        {vacations.map(vacation =>
          <div key={vacation.sku}>
            <h3>{vacation.name}</h3>
            <p>{vacation.description}</p>
            <span className="price">{vacation.price}</span>
          </div>
        )}
      </div>
    </>
  )
}
```

与之前一样，`useState`正在配置我们的组件状态，使其拥有一个`vacations`数组，并带有一个伴随的 setter。现在我们添加了`useEffect`，它调用我们的 API 来检索度假，并异步调用该 setter。请注意，我们将空数组作为`useEffect`的第二个参数传入；这是向 React 发出的一个信号，表示此效果应在组件挂载时仅运行一次。表面上看，这可能看起来是一种奇怪的信号方式，但一旦您更多地了解了 hooks，您就会发现它其实是相当一致的。要了解更多关于 hooks 的信息，请参阅[React hooks 文档](http://bit.ly/34MGSeK)。

Hooks 是相对较新的东西—它们是在 2019 年 2 月的 16.8 版本中添加的—因此，即使您对 React 有一些经验，您可能对 hooks 并不熟悉。我坚信 hooks 是 React 体系结构中的一个极好的创新，尽管它们一开始可能看起来很陌生，但您会发现它们实际上简化了您的组件并减少了人们普遍犯的一些棘手的与状态相关的错误。

现在我们已经学会了如何从服务器检索数据，让我们把注意力转向以相反方式发送信息。

## 向服务器发送信息

我们已经有一个 API 端点用于在服务器上进行更改；当（假期）回到季节时，我们有一个端点用于发送电子邮件通知。让我们继续修改我们的`Vacations`组件，使其显示一个针对不在季节内的度假的注册表单。按照 React 的风格，我们将创建两个新组件：我们将把单个假期视图拆分成`Vacation`和`NotifyWhenInSeason`组件。我们可以把它们都放在一个组件中，但 React 开发的推荐方法是拥有很多特定目的的组件，而不是庞大的通用组件（为了简洁起见，我们将不会把这些组件放在它们自己的文件中：我会把这留给读者做练习）：

```
import React, { useState, useEffect } from 'react'

function NotifyWhenInSeason({ sku }) {
  return (
    <>
      <i>Notify me when this vacation is in season:</i>
      <input type="email" placeholder="(your email)" />
      <button>OK</button>
    </>
  )
}

function Vacation({ vacation }) {
  return (
    <div key={vacation.sku}>
      <h3>{vacation.name}</h3>
      <p>{vacation.description}</p>
      <span className="price">{vacation.price}</span>
      {!vacation.inSeason &&
        <div>
          <p><i>This vacation is not currently in season.</i></p>
          <NotifyWhenInSeason sky={vacation.sku} />
        </div>
      }
    </div>
  )
}

function Vacations() {
  const [vacations, setVacations] = useState([])
  useEffect(() => {
    fetch('/api/vacations')
      .then(res => res.json())
      .then(setVacations)
  }, [])
  return (
    <>
      <h2>Vacations</h2>
      <div className="vacations">
        {vacations.map(vacation =>
          <Vacation key={vacation.sku} vacation={vacation} />
        )}
      </div>
    </>
  )
}

export default Vacations
```

现在，如果您有任何`inSeason`为`false`的假期（除非您更改了数据库或初始化脚本），您将更新表单。现在让我们连接按钮来进行 API 调用。修改`NotifyWhenInSeason`：

```
function NotifyWhenInSeason({ sku }) {
  const [registeredEmail, setRegisteredEmail] = useState(null)
  const [email, setEmail] = useState('')
  function onSubmit(event) {
    fetch(`/api/vacation/${sku}/notify-when-in-season`, {
        method: 'POST',
        body: JSON.stringify({ email }),
        headers: { 'Content-Type': 'application/json' },
      })
      .then(res => {
        if(res.status < 200 || res.status > 299)
          return alert('We had a problem processing this...please try again.')
        setRegisteredEmail(email)
      })
    event.preventDefault()
  }
  if(registeredEmail) return (
    <i>You will be notified at {registeredEmail} when
    this vacation is back in season!</i>
  )
  return (
    <form onSubmit={onSubmit}>
      <i>Notify me when this vacation is in season:</i>
      <input
        type="email"
        placeholder="(your email)"
        value={email}
        onChange={({ target: { value } }) => setEmail(value)}
        />
      <button type="submit">OK</button>
    </form>
  )
}
```

我们选择让该组件跟踪两个不同的值：用户在输入时的电子邮件地址，以及他们按下“确定”后的最终值。前者是一种被称为*受控组件*的技术，您可以在[React 表单文档](http://bit.ly/2X9P9qh)中了解更多信息。我们追踪后者是为了在用户执行了按下“确定”的操作时了解，以便我们可以相应地更改 UI。我们也可以简单地使用布尔值“registered”，但这样可以让我们的 UI 提醒用户他们注册时使用的电子邮件。

我们还需要与我们的 API 通信做更多的工作：我们必须指定方法（`POST`），将主体编码为 JSON，并指定内容类型。

注意，我们要决定返回哪种 UI。如果用户已经注册，我们返回一个简单的消息；如果他们没有，我们呈现表单。这在 React 中是非常常见的模式。

哎呀！为了那么少的功能而做那么多工作……而且功能也相当简陋。如果 API 调用出现问题，我们的错误处理功能虽然有效，但不够用户友好；而组件将只在我们停留在这个页面时记住我们注册的假期。如果我们离开再回来，我们将再次看到表单。

要使这段代码更易理解，我们可以采取一些步骤。首先，我们可以编写一个 API 包装器，处理编码输入和确定错误的混乱细节；随着我们使用更多的 API 端点，这将带来显著的回报。此外，React 还有许多流行的表单处理框架，可以大大减轻表单处理的负担。

解决“记住”用户注册了哪些假期的问题有点棘手。真正有用的是让我们的假期对象具备这些信息（无论用户是否注册）。然而，我们的专用组件对假期一无所知；它只知道 SKU。在下一节中，我们将讨论*状态管理*，这将指向解决这个问题的方案。

## 状态管理

大部分规划和设计 React 应用程序的架构工作都集中在状态管理上，通常不是单个组件的状态管理，而是它们如何共享和协调状态。我们的示例应用程序确实共享了一些状态：`Vacations`组件向下传递了一个假期对象给`Vacation`组件，而`Vacation`组件则将假期的 SKU 传递给`NotifyWhenInSeason`监听器。但到目前为止，我们的信息只在树中*向下*流动；当信息需要向*上*返回时会发生什么呢？

最常见的方法是传递负责更新状态的函数。例如，`Vacations`组件可能有一个用于修改假期的函数，它可以传递给`Vacation`，然后再传递给`NotifyWhenInSeason`。当`NotifyWhenInSeason`调用它来修改假期时，树的顶部的`Vacations`将意识到事情已经改变，这将导致它重新渲染，从而使其所有后代都重新渲染。

听起来很累人和复杂，并且有时确实如此，但有些技术可以帮助解决这些问题。它们如此多样和有时复杂，以至于我们无法在这里完全覆盖它们（这也不是一本关于 React 的书），但我可以指引你进一步阅读：

[Redux](https://redux.js.org)

Redux 通常是人们在考虑 React 应用程序的全面状态管理时首先想到的东西。它是最早形式化的状态管理架构之一，仍然非常流行。在概念上，它非常简单，这仍然是我喜欢的状态管理框架。即使最终你不选择 Redux，我建议你观看其创建者 Dan Abramov 的[免费教程视频](https://egghead.io/courses/getting-started-with-redux)。

[MobX](https://mobx.js.org)

MobX 在 Redux 之后出现。它在短时间内获得了令人印象深刻的追随者，并且可能是第二受欢迎的状态容器，仅次于 Redux。MobX 确实可以导致看起来更容易编写的代码，但我仍然觉得 Redux 在应用程序扩展时提供了更好的框架，即使它增加了样板代码。

[Apollo](https://www.apollographql.com)

Apollo 并不是一个状态管理库* per se*，但通常它的使用方式可以替代其中的一个。它本质上是一个前端接口，用于[GraphQL](https://graphql.org)，作为 REST API 的替代方案，与 React 有很多集成。如果你在使用 GraphQL（或者对它感兴趣），那么它绝对值得一试。

[React Context](https://reactjs.org/docs/context.html)

React 本身已经通过提供内置的 Context API 参与了这场游戏。它实现了 Redux 相同的一些功能，但是减少了样板代码。然而，我觉得 React Context 不够健壮，Redux 在应用程序不断增长时是更好的选择。

当你刚开始使用 React 时，你可以基本上忽略跨应用程序的状态管理的复杂性，但很快你会意识到需要一种更有组织的方式来管理状态。当你达到这一点时，你会想要研究一些选项，并选择一个适合你的选项。

## 部署选项

到目前为止，我们一直在使用 CRA 内置的开发服务器——这确实是开发的最佳选择，我建议坚持使用它。然而，当谈到部署时，它并不是一个合适的选择。幸运的是，CRA 带有一个构建脚本，用于创建一个针对生产优化的捆绑包，然后你有很多选择。当你准备创建部署捆绑包时，只需运行`yarn build`，然后将创建一个*build*目录。*build*目录中的所有资产都是静态的，可以部署到任何地方。

我目前首选的部署方式是将 CRA 构建放入带有[静态网站托管](https://amzn.to/3736fuT)的 AWS S3 存储桶中。这远非唯一选择：每个主要的云提供商和 CDN 都提供类似的功能。

在这种配置中，我们必须创建路由，以便将 API 调用路由到您的 Express 服务器，并从 CDN 提供您的静态 bundle。对于我的 AWS 部署，我使用[AWS CloudFront](https://amzn.to/2KglZRb)执行此路由；静态资产从前述的 S3 存储桶中提供，并且 API 请求被路由到 EC2 实例上的 Express 服务器或 Lambda 上。

另一个选择是让 Express 来完成所有工作。这样做的优点是能够将整个应用程序集中到一个单一的服务器上，这样部署会相当简单，管理也很方便。虽然这种方式可能不太适合可扩展性或性能，但对于小型应用程序来说是一个有效的选择。

要完全通过 Express 提供您的应用程序，只需将运行`yarn build`时创建的*build*目录中的内容复制到 Express 应用程序的*public*目录中。只要您链接了静态中间件，它将自动提供*index.html*文件，这就是您所需的一切。

不妨试试：如果你的 Express 服务器仍在 3033 端口运行，你应该能够访问*http://localhost:3033*，看到与 CRA 开发服务器提供的相同应用程序！

###### 注意

如果你想了解 CRA 的开发服务器是如何工作的，它使用了一个名为`webpack-dev-server`的包，其底层使用了 Express！所以最终一切都与 Express 有关！

# 结论

本章仅仅触及了 React 及其周围的技术表面。如果你想深入了解 React，[*学习 React*](https://oreil.ly/ROqku)（由 Alex Banks 和 Eve Porcello 编著，O'Reilly 出版）是一个很好的起点。这本书还涵盖了使用 Redux 进行状态管理（不过目前还不包括 hooks）。[官方 React 文档](http://bit.ly/37377Qb)也非常全面和详细。

单页应用程序（SPAs）确实改变了我们思考和交付 Web 应用程序的方式，并显著提升了性能，尤其是在移动设备上。即使 Express 是在大多数 HTML 仍然主要在服务器上渲染的时代编写的，它并没有使 Express 过时。相反，为单页应用程序提供 API 的需求使得 Express 焕发了新生命！

从阅读本章中应该也能清楚地看出，这确实都是相同的游戏：数据在浏览器和服务器之间来回传递。只是数据的性质发生了变化，我们要适应通过动态 DOM 操作改变 HTML 的方式。

^(1)出于性能考虑，bundle 可能会被拆分为按需加载的“chunk”（称为*lazy loading*），但其原理是一样的。
