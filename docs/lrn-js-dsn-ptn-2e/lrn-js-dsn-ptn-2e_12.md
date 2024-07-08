# 第十二章：React.js 设计模式

多年来，使用 JavaScript 组合 UI 的简单方法的需求不断增加。前端开发人员寻找由许多不同的库和框架提供的开箱即用的解决方案。React 在这个领域的流行程度已经持续很长一段时间，自从它在 2013 年首次发布以来。本章将讨论在 React 宇宙中有帮助的设计模式。

[React](https://oreil.ly/7Z-65)，也称为 React.js，是由 Facebook 设计的开源 JavaScript 库，用于构建 UI 或 UI 组件。当然，并不是唯一的 UI 库。[Preact](https://oreil.ly/jXmKM)、[Vue](https://oreil.ly/fMoMp)、[Angular](https://oreil.ly/G_Oyv)、[Svelte](https://oreil.ly/scSoT)、[Lit](https://oreil.ly/5UgxC) 等等也非常适合使用可重用元素来组成界面。然而，鉴于 React 的流行度，我们选择它来讨论当前十年的设计模式。

# React 入门

前端开发人员谈论代码时，通常是在设计 Web 界面的上下文中。我们将界面组合看作是按钮、列表、导航等元素。React 提供了一种优化和简化的方式来表达这些元素中的界面。它还通过将界面组织成组件、属性和状态三个关键概念，帮助构建复杂和棘手的界面。

由于 React 专注于组合，它可以完美地映射到设计系统的元素。因此，为 React 设计奖励您以模块化的方式思考。它允许您在将页面或视图组合在一起之前开发单独的组件，以便您充分理解每个组件的范围和目的——这个过程被称为组件化。

## 使用的术语

在本章中，我们将经常使用以下术语。让我们快速看看它们各自的含义：

React/React.js/ReactJS

由 Facebook 于 2013 年创建的 React 库

ReactDOM

提供面向客户端和服务器渲染的 DOM 特定方法的 `react-dom` 包

JSX

JavaScript 的语法扩展

Redux

集中式状态容器

Hooks

一种在不编写类的情况下使用状态和其他 React 特性的新方法

ReactNative

用 JavaScript 开发跨平台原生应用程序的库

webpack

JavaScript 模块打包工具，在 React 社区中很受欢迎

单页应用程序（SPA）

在同一页上加载新内容而无需进行完整的页面刷新/重新加载的 Web 应用程序

## 基本概念

在讨论 React 设计模式之前，了解一些在 React 中使用的基本概念将会很有帮助：

JSX

JSX 是 JavaScript 的扩展，它使用类似 XML 的语法将模板 HTML 嵌入到 JS 中。它意图被转换为有效的 JavaScript，虽然这种转换的语义是特定于实现的。JSX 随着 React 库的流行而兴起，但也看到了其他实现。

组件

组件是任何 React 应用程序的构建块。它们类似于接受任意输入（属性）并返回描述应该显示在屏幕上的 React 元素的 JavaScript 函数。React 应用程序中的所有内容都是组件的一部分。基本上，React 应用程序只是组件中的组件中的组件。因此，开发人员不是在 React 中构建页面；他们构建组件。组件让您将 UI 拆分为独立的、可重用的部分。如果您习惯设计页面，从组件的角度思考可能看起来像是一个重大变化。但如果您使用设计系统或样式指南，这可能比看起来的范式转变要小。

属性

属性是 React 组件的内部数据的简写形式。它们写在组件调用内部，并传递给组件。它们还使用与 HTML 属性相同的语法，例如，`prop = value`。关于属性值值得记住的两件事是：（1）我们确定属性值并在构建组件之前将其作为蓝图的一部分使用，（2）属性值永远不会改变，即一旦传递给组件，属性就是只读的。您可以通过每个组件都可以访问的 `this.props` 属性引用来访问属性。

状态

状态是一个保存可能随组件生命周期而变化的信息的对象。它是存储在组件属性中的数据的当前快照。数据随时间变化，因此需要技术来管理数据变化，以确保组件在工程师希望的时间看起来正确——这称为状态管理。

客户端渲染

在客户端渲染（CSR）中，服务器仅为页面渲染基本的 HTML 容器。显示页面内容所需的逻辑、数据获取、模板化和路由由在客户端执行的 JavaScript 代码处理。CSR 作为构建单页面应用程序的一种方法变得流行。它有助于模糊网站和安装应用程序之间的差异，并且最适用于高度交互式应用程序。默认情况下，大部分应用程序逻辑在客户端执行。它通过 API 调用与服务器交互以获取或保存数据。

服务器端渲染

SSR 是最古老的网页内容渲染方法之一。SSR 生成完整的 HTML，用于响应用户请求时渲染页面内容。内容可能包括来自数据存储或外部 API 的数据。React 可以进行同构渲染，这意味着它可以在浏览器和服务器等其他平台上运行。因此，可以使用 React 在服务器上渲染 UI 元素。

水合

在服务器渲染的应用程序中，当前导航的 HTML 是在服务器上生成并发送到客户端的。由于服务器生成了标记，客户端可以快速解析它并在屏幕上显示。UI 变得交互式所需的 JavaScript 在此之后加载。只有在 JavaScript 捆绑包加载和处理后，才会附加使按钮等 UI 元素变得交互式的事件处理程序。这个过程称为水合。React 检查当前的 DOM 节点并用相应的 JavaScript 进行水合处理。

创建一个新的应用程序

较旧的文档建议使用 Create React App（CRA）来构建一个新的仅客户端的单页面应用程序来学习 React。这是一个 CLI 工具，用于为启动项目创建 React 应用程序的脚手架。然而，CRA 提供了一种受限的开发体验，这对许多现代 Web 应用程序来说太过局限。React 建议使用生产级别的 React 驱动框架，如 Next.js 或 Remix 来构建新的 Web 应用程序或网站。这些框架提供了大多数应用程序和网站最终需要的功能，如静态 HTML 生成、基于文件的路由、SPA 导航和真正的客户端代码。

React 在多年来已经发生了变化。引入到库中的不同特性催生了解决常见问题的各种方式。以下是我们将在接下来的章节中详细探讨的 React 的一些流行设计模式：

+   “高阶组件”

+   “渲染属性模式”

+   “Hooks 模式”

+   “静态导入”

+   “动态导入”

+   “代码分割”

+   “PRPL 模式”

+   “加载优先级”

# 高阶组件

我们经常希望在应用程序中的多个组件中使用相同的逻辑。这种逻辑可以包括向组件应用特定样式、需要授权或添加全局状态。通过使用高阶组件（HOC）模式，我们可以在多个组件中重用相同的逻辑。这种模式允许我们在整个应用程序中重用组件逻辑。

高阶组件（HOC）是一个接收另一个组件的组件。HOC 可以包含一个特定功能，该功能可以应用于我们传递给它的组件。HOC 返回应用了附加功能的组件。

假设我们始终希望向应用程序中的多个组件添加特定样式。我们可以创建一个高阶组件（HOC），将样式对象添加到作为参数传递的组件中，而不是每次在本地创建样式对象：

```
function withStyles(Component) {
  return props => {
    const style = { padding: '0.2rem', margin: '1rem' }
    return <Component style={style} {...props} />
  }
}

const Button = () = <button>Click me!</button>
const Text = () => <p>Hello World!</p>

const StyledButton = withStyles(Button)
const StyledText = withStyles(Text)
```

我们刚刚创建了`StyledButton`和`StyledText`组件，这是`Button`和`Text`组件的修改版本。它们现在都包含了在`withStyles`高阶组件中添加的样式。

进一步来看，让我们看一个从 API 获取的狗图片列表的应用程序。在获取数据时，我们希望向用户显示“Loading…​”屏幕。我们可以使用一个高阶组件添加这个逻辑，而不是直接将其添加到`DogImages`组件中。

让我们创建一个名为`withLoader`的高阶组件。一个高阶组件应该接收一个组件并返回该组件。在这种情况下，`withLoader`高阶组件应该接收应该显示`Loading…`直到数据被获取的元素。为了使`withLoader`高阶组件非常可重用，我们不会在该组件中硬编码狗 API URL。相反，我们可以将 URL 作为参数传递给`withLoader`高阶组件，因此在从不同 API 端点获取数据时，可以使用此加载器在任何需要加载指示器的组件上：

```
function withLoader(Element, url) {
  return props => {};
}
```

高阶组件返回一个元素，一个函数组件`props ⇒ {}`在这种情况下，我们希望在数据仍在获取时显示一个带有`Loading…`文本的逻辑。一旦数据被获取，组件应该将获取的数据作为属性传递。完整的`withLoader`代码如下所示：

```
import React, { useEffect, useState } from "react";

export default function withLoader(Element, url) {
  return (props) => {
    const [data, setData] = useState(null);

    useEffect(() => {
      async function getData() {
        const res = await fetch(url);
        const data = await res.json();
        setData(data);
      }

      getData();
    }, []);

    if (!data) {
      return <div>Loading...</div>;
    }

    return <Element {...props} data={data} />;
  };
}
```

我们刚刚创建了一个可以接收任何组件和 URL 的高阶组件：

+   在`useEffect`钩子中，`withLoader`高阶组件从我们传递的 API 端点获取数据。在数据正在获取的时候，我们返回包含`Loading…`文本的元素。

+   一旦数据被获取，我们将数据设置为已获取的数据。由于数据不再为`null`，我们可以显示传递给高阶组件的元素。

现在，为了在`DogImages`列表上显示`Loading…`指示器，我们将“包装好的”`withLoading`高阶组件导出到`DogImages`组件周围。`withLoader`高阶组件还期望`url`，以了解从哪个端点获取数据。在这种情况下，我们想要添加狗 API 端点。由于`withLoader`高阶组件返回了带有额外数据属性的元素，因此在`DogImages`组件中，我们可以访问数据属性：

```
import React from "react";
import withLoader from "./withLoader";

function DogImages(props) {
  return props.data.message.map((dog, index) => (
    <img src={dog} alt="Dog" key={index} />
  ));
}

export default withLoader(
  DogImages,
  "https://dog.ceo/api/breed/labrador/images/random/6"
);
```

高阶组件模式允许我们为多个组件提供相同的逻辑，同时将所有逻辑保存在一个地方。`withLoader`高阶组件不关心接收的组件或 URL；只要是有效的组件和有效的 API 端点，它将简单地将来自该 API 端点的数据传递给我们传递的组件。

## 组合

我们还可以组合多个高阶组件。比如说，我们还想添加一个功能，当用户悬停在`DogImages`列表上时显示一个悬停文本框。

我们必须创建一个高阶组件（HOC），为我们传递的元素提供一个悬停属性。基于这个属性，我们可以根据用户是否悬停在`DogImages`列表上来有条件地渲染文本框。

现在我们可以将`withHover`高阶组件包装在`withLoader`高阶组件外部：

```
export default withHover(
  withLoader(DogImages, "https://dog.ceo/api/breed/labrador/images/random/6")
);
```

现在，`DogImages`元素包含了我们从`withHover`和`withLoader`传递的所有属性。

在某些情况下，我们还可以使用 Hooks 模式来实现类似的结果。我们将在本章稍后详细讨论这种模式，但现在，让我们简单地说，使用 Hooks 可以减少组件树的深度，而使用 HOC 模式则很容易导致组件树深度嵌套。适合使用 HOC 的最佳情况是以下情况为真：

+   应用程序中许多组件需要使用相同的未定制行为。

+   组件可以独立工作，无需添加自定义逻辑。

## 优点

使用 HOC 模式可以将我们想要重用的逻辑集中在一处。这样做可以减少通过重复复制代码而在应用程序中意外传播错误的风险，从而可能引入新的错误。通过将逻辑集中在一处，我们可以保持代码的 DRY，并有效地强制执行关注点的分离。

## 缺点

高阶组件可以向元素传递的道具名称可能会导致名称冲突。例如：

```
function withStyles(Component) {
  return props => {
    const style = { padding: '0.2rem', margin: '1rem' }
    return <Component style={style} {...props} />
  }
}

const Button = () = <button style={{ color: 'red' }}>Click me!</button>
const StyledButton = withStyles(Button)
```

在这种情况下，`withStyles` 高阶组件（HOC）向我们传递的元素添加了一个名为 `style` 的属性。然而，`Button` 组件已经有一个名为 `style` 的属性，这将被覆盖！确保高阶组件能够通过重命名或合并属性来处理意外的名称冲突：

```
function withStyles(Component) {
  return props => {
    const style = {
      padding: '0.2rem',
      margin: '1rem',
      ...props.style
    }

    return <Component style={style} {...props} />
  }
}

const Button = () = <button style={{ color: 'red' }}>Click me!</button>
const StyledButton = withStyles(Button)
```

当使用多个组合的高阶组件时，它们都将道具传递给包装在其中的元素时，找出哪个高阶组件负责哪个道具可能是具有挑战性的。这可能会阻碍调试和轻松扩展应用程序。

# 渲染道具模式

在高阶组件部分，我们看到如果多个组件需要访问相同的数据或包含相同的逻辑，重用组件逻辑将是方便的。

另一种使组件可重用的方法是使用渲染道具模式。渲染道具是组件上的一个属性，其值是一个返回 JSX 元素的函数。组件本身除了调用渲染道具而不实现自己的渲染逻辑外，不渲染任何内容。

假设我们有一个 `Title` 组件，应该只渲染我们传递的值。我们可以为此使用渲染道具。让我们将想要 `Title` 组件渲染的值传递给渲染道具：

```
<Title render={() => <h1>I am a render prop!</h1>} />
```

我们可以通过返回调用的渲染道具来在 `Title` 组件中渲染这些数据：

```
const Title = props => props.render();
```

我们必须向组件元素传递一个名为 `render` 的道具，这是一个返回 React 元素的函数：

```
import React from "react";
import { render } from "react-dom";

import "./styles.css";

const Title = (props) => props.render();

render(
  <div className="App">
    <Title
      render={() => (
        <h1>
          <span role="img" aria-label="emoji">
            ✨
          </span>
          I am a render prop!{" "}
          <span role="img" aria-label="emoji">
            ✨
          </span>
        </h1>
      )}
    />
  </div>,
  document.getElementById("root")
);
```

渲染道具的有趣之处在于接收该道具的组件是可重用的。我们可以多次使用它，每次传递不同的值给渲染道具。

尽管它们被称为渲染道具，但渲染道具不一定要被称为 `render`。任何渲染 JSX 的道具都被视为渲染道具。因此，在下面的示例中，我们有三个渲染道具：

```
const Title = (props) => (
  <>
    {props.renderFirstComponent()}
    {props.renderSecondComponent()}
    {props.renderThirdComponent()}
  </>
);

render(
  <div className="App">
    <Title
      renderFirstComponent={() => <h1>First render prop!</h1>}
      renderSecondComponent={() => <h2> Second render prop!</h2>}
      renderThirdComponent={() => <h3>Third render prop!</h3>}
    />
  </div>,
  document.getElementById("root")
);
```

我们刚刚看到，我们可以使用渲染道具使组件可重用，因为我们每次可以传递不同的数据给渲染道具。

使用渲染属性的组件通常不仅仅调用渲染属性。相反，我们通常希望从接受渲染属性的组件传递数据到作为渲染属性传递的元素：

```
function Component(props) {
  const data = { ... }

  return props.render(data)
}
```

现在渲染属性可以接收我们传递的这个值作为其参数：

```
<Component render={data => <ChildComponent data={data} />}
```

## 提升状态

在我们看另一个使用渲染属性模式的用例之前，让我们先了解在 React 中“状态提升”的概念。

假设我们有一个温度转换器，您可以在一个有状态的输入元素中提供摄氏度输入。两个其他组件中的对应的 `Fahrenheit` 和 `Kelvin` 值会立即反映出来。为了能够与其他组件共享其状态，我们将不得不将状态移至需要它的组件的最近公共祖先。这就是所谓的“状态提升”：

```
function Input({ value, handleChange }) {
  return <input value={value} onChange={e => handleChange(e.target.value)} />;
}
function Kelvin({ value = 0 }) {
  return <div className="temp">{value + 273.15}K</div>;
}

function Fahrenheit({ value = 0 }) {
  return <div className="temp">{(value * 9) / 5 + 32}°F</div>;
}

export default function App() {
  const [value, setValue] = useState("");

  return (
    <div className="App">
      <h1>Temperature Converter</h1>
      <Input value={value} handleChange={setValue} />
      <Kelvin value={value} />
      <Fahrenheit value={value} />
    </div>
  );
}
```

状态提升是一种有价值的 React 状态管理模式，因为有时我们希望一个组件能够与其兄弟组件共享其状态。在具有少量组件的小型应用中，我们可以避免使用 Redux 或 React Context 这样的状态管理库，而是使用这种模式将状态提升到最近的共同祖先组件中。

虽然这是一个有效的解决方案，但是在处理许多子组件的大型应用中提升状态可能会比较棘手。每次状态更改都可能导致所有子组件重新渲染，即使这些组件不处理数据，也可能对应用程序的性能产生负面影响。我们可以使用渲染属性模式来解决这个问题。我们将改变 `Input` 组件，使其能够接收渲染属性：

```
function Input(props) {
  const [value, setValue] = useState("");

  return (
    <>
      <input
        type="text"
        value={value}
        onChange={e => setValue(e.target.value)}
        placeholder="Temp in °C"
      />
      {props.render(value)}
    </>
  );
}

export default function App() {
  return (
    <div className="App">
      <h1>Temperature Converter</h1>
      <Input
        render={value => (
          <>
            <Kelvin value={value} />
            <Fahrenheit value={value} />
          </>
        )}
      />
    </div>
  );
}
```

## 将子元素作为函数

除了常规的 JSX 组件外，我们还可以将函数作为子元素传递给 React 组件。通过 `children` 属性，我们可以访问到这个函数，技术上也算是一个渲染属性。

让我们改变 `Input` 组件。我们不再显式地传递渲染属性，而是将一个函数作为 `Input` 组件的子元素：

```
export default function App() {
  return (
    <div className="App">
      <h1>Temperature Converter</h1>
      <Input>
        {value => (
          <>
            <Kelvin value={value} />
            <Fahrenheit value={value} />
          </>
        )}
      </Input>
    </div>
  );
}
```

我们可以通过 `props.children` 属性访问到这个函数，该属性在 `Input` 组件上是可用的。我们不再使用用户输入值调用 `props.render`，而是使用用户输入的值调用 `props.children`：

```
function Input(props) {
  const [value, setValue] = useState("");

  return (
    <>
      <input
        type="text"
        value={value}
        onChange={e => setValue(e.target.value)}
        placeholder="Temp in °C"
      />
      {props.children(value)}
    </>
  );
}
```

这样，`Kelvin` 和 `Fahrenheit` 组件可以访问该值，而不用担心渲染属性的名称。

## 优点

使用渲染属性模式可以轻松地在多个组件之间共享逻辑和数据。通过使用渲染属性或子元素属性，组件可以被设计为可重用。虽然高阶组件模式主要解决相同的问题，即可重用性和数据共享，但渲染属性模式解决了在使用高阶组件模式时可能遇到的一些问题。

通过使用渲染属性模式，我们不再自动合并属性，从而避免了使用高阶组件模式可能遇到的命名冲突问题。我们会显式地通过父组件提供的值将属性传递给子组件。

由于我们明确传递 props，我们解决了 HOC 的隐式 props 问题。应该传递到元素的 props 都在渲染属性的参数列表中是可见的。通过这种方式，我们确切地知道特定 props 来自何处。我们可以通过渲染属性将应用程序的逻辑与渲染组件分离。接收渲染属性的有状态组件可以将数据传递给仅仅渲染数据的无状态组件。

## 缺点

React Hooks 在很大程度上解决了我们尝试使用渲染属性解决的问题。由于 Hooks 改变了我们向组件添加可重用性和数据共享的方式，它们可以在许多情况下替代渲染属性模式。

由于我们无法向渲染属性添加生命周期方法，因此只能在不需要修改接收到的数据的组件上使用它。

# Hooks 模式

React 16.8 引入了一个名为[Hooks](https://oreil.ly/6qnHk)的新功能。Hooks 使得可以在不使用 ES2015 类组件的情况下使用 React 状态和生命周期方法。虽然 Hooks 不一定是一个设计模式，但 Hooks 在应用程序设计中扮演了重要角色。Hooks 可以替代许多传统的设计模式。

让我们看看类组件是如何实现添加状态和生命周期方法的。

## 类组件

在 React 引入 Hooks 之前，我们必须使用类组件向组件添加状态和生命周期方法。React 中的典型类组件可能如下所示：

```
class MyComponent extends React.Component {
  // Adding state and binding custom methods
  constructor() {
    super()
    this.state = { ... }

    this.customMethodOne = this.customMethodOne.bind(this)
    this.customMethodTwo = this.customMethodTwo.bind(this)
  }

  // Lifecycle Methods
  componentDidMount() { ...}
  componentWillUnmount() { ... }

  // Custom methods
  customMethodOne() { ... }
  customMethodTwo() { ... }

  render() { return { ... }}
}
```

类组件可以包含以下内容：

+   构造函数中的状态

+   生命周期方法如`componentDidMount`和`componentWillUnmount`用于根据组件的生命周期执行副作用。

+   添加额外逻辑到类的自定义方法

虽然在引入 React Hooks 后我们仍然可以使用类组件，但使用类组件可能会有一些缺点。例如，考虑以下示例，其中一个简单的`div`作为按钮：

```
function Button() {
  return <div className="btn">disabled</div>;
}
```

而不是始终显示为禁用状态，我们希望在用户单击按钮时将其更改为启用状态，并为按钮添加一些额外的 CSS 样式。为此，我们需要向组件添加状态来知道状态是启用还是禁用。这意味着我们必须完全重构功能组件，并使其成为一个保持按钮状态的类组件：

```
export default class Button extends React.Component {
  constructor() {
    super();
    this.state = { enabled: false };
  }

  render() {
    const { enabled } = this.state;
    const btnText = enabled ? "enabled" : "disabled";

    return (
      <div
        className={`btn enabled-${enabled}`}
        onClick={() => this.setState({ enabled: !enabled })}
      >
        {btnText}
      </div>
    );
  }
}
```

在这个例子中，组件很简单，重构并不需要太多的工作。然而，你的真实组件可能包含更多的代码行，这使得重构组件变得更加困难。

除了确保在重构组件时不要意外改变任何行为之外，还必须理解 ES2015+类的工作方式。在不意外更改数据流的情况下正确重构组件可能是具有挑战性的。

## 重构

在几个组件之间共享代码的标准方法是使用 HOC 或 Render Props 模式。虽然这两种模式都是有效的，并且使用它们是一种良好的实践，但在以后的某个时候添加这些模式需要重新构造应用程序。

除了重构应用程序外，组件越大，使用许多包装组件在更深层嵌套组件之间共享代码也会导致一种被称为“包装地狱”的情况。在开发工具中看到以下类似结构并不罕见：

```
<WrapperOne>
  <WrapperTwo>
    <WrapperThree>
      <WrapperFour>
        <WrapperFive>
          <Component>
            <h1>Finally in the component!</h1>
          </Component>
        </WrapperFive>
      </WrapperFour>
    </WrapperThree>
  </WrapperTwo>
</WrapperOne>
```

包装地狱可能会使您难以理解数据在应用程序中的流动方式，从而更难弄清为什么会发生意外行为。

## 复杂性

随着我们向类组件添加更多逻辑，组件的大小会迅速增加。组件内部的逻辑可能会变得混乱和无结构，使开发人员难以理解类组件中某些逻辑的使用位置。这可能会使调试和优化性能变得更加困难。生命周期方法在代码中也需要大量重复。

## 钩子

在 React 中，并非总是使用类组件是一个很好的特性。为了解决 React 开发人员在使用类组件时可能遇到的常见问题，React 引入了 React Hooks。React Hooks 是您可以用来管理组件状态和生命周期方法的函数。React Hooks 使得：

+   添加状态到函数组件

+   在不使用`componentDidMount`和`componentWillUnmount`等生命周期方法的情况下管理组件的生命周期

+   在整个应用程序中重用相同的有状态逻辑

首先，让我们看看如何使用 React Hooks 向函数组件添加状态。

# 状态钩子

React 提供了一个名为`useState`的 Hook，用于在函数组件内部管理状态。

让我们看看如何使用`useState`钩子将类组件重构为函数组件。我们有一个名为`Input`的类组件，它渲染一个输入字段。当用户在输入字段中键入任何内容时，输入状态中的值会更新：

```
class Input extends React.Component {
  constructor() {
    super();
    this.state = { input: "" };

    this.handleInput = this.handleInput.bind(this);
  }

  handleInput(e) {
    this.setState({ input: e.target.value });
  }

  render() {
    <input onChange={handleInput} value={this.state.input} />;
  }
}
```

要使用`useState`钩子，我们需要访问 React 的`useState`方法。`useState`方法期望一个参数：这是状态的初始值，在本例中是一个空字符串。

我们可以从`useState`方法中解构出两个值：

+   状态的*当前值*

+   *我们可以更新状态的方法*：

```
const [value, setValue] = React.useState(initialValue);
```

您可以将第一个值与类组件的`this.state.[value]`进行比较。第二个值可以与类组件的`this.setState`方法进行比较。

由于我们处理输入的值，让我们称状态的当前值为输入，用于更新状态的方法称为`setInput`。初始值应为空字符串：

```
const [input, setInput] = React.useState("");
```

现在我们可以将`Input`类组件重构为有状态的函数组件：

```
function Input() {
  const [input, setInput] = React.useState("");

  return <input onChange={(e) => setInput(e.target.value)} value={input} />;
}
```

输入字段的值等于输入状态的当前值，就像在类组件示例中一样。当用户在输入字段中输入时，输入状态的值相应地更新，使用`setInput`方法：

```
import React, { useState } from "react";

  export default function Input() {
    const [input, setInput] = useState("");

    return (
      <input
        onChange={e => setInput(e.target.value)}
        value={input}
        placeholder="Type something..."
      />
    );
  }
```

## Effect Hook

我们已经看到可以使用`useState`组件在函数组件内处理状态。但是，类组件的另一个好处是可以将生命周期方法添加到组件中。

使用`useEffect`钩子，我们可以“挂接”到组件的生命周期。`useEffect`钩子有效地结合了`componentDidMount`、`componentDidUpdate`和`componentWillUnmount`生命周期方法：

```
componentDidMount() { ... }
useEffect(() => { ... }, [])

componentWillUnmount() { ... }
useEffect(() => { return () => { ... } }, [])

componentDidUpdate() { ... }
useEffect(() => { ... })
```

让我们使用状态钩子部分中使用的输入示例。每当用户在输入字段中输入任何内容时，该值也应该记录到控制台中。

我们需要一个`useEffect`钩子来“监听”输入值。我们可以通过将输入添加到`useEffect`钩子的依赖数组中来实现这一点。依赖数组是`useEffect`钩子接收的第二个参数：

```
import React, { useState, useEffect } from "react";

export default function Input() {
  const [input, setInput] = useState("");

  useEffect(() => {
    console.log(`The user typed ${input}`);
  }, [input]);

  return (
    <input
      onChange={e => setInput(e.target.value)}
      value={input}
      placeholder="Type something..."
    />
  );
}
```

当用户键入值时，输入字段的值现在会记录到控制台中。

## 自定义钩子

除了 React 提供的内置钩子（`useState`，`useEffect`，`useReducer`，`useRef`，`useContext`，`useMemo`，`useImperativeHandle`，`useLayoutEffect`，`useDebugValue`，`useCallback`），我们还可以轻松创建自己的自定义钩子。

您可能已经注意到所有的钩子都以“use”开头。重要的是要以“use”开头，以便 React 检查是否违反了 Hooks 的规则。

假设我们想追踪用户在输入时可能按下的特定键。我们的自定义钩子应该能够接收我们想要定位的键作为参数。

我们希望为用户传递的参数添加`keydown`和`keyup`事件侦听器。如果用户按下该键，则`keydown`事件将被触发，Hook 内的状态应该切换为 true。否则，当用户停止按下该按钮时，将触发`keyup`事件，并且状态将切换为 false：

```
function useKeyPress(targetKey) {
  const [keyPressed, setKeyPressed] = React.useState(false);

  function handleDown({ key }) {
    if (key === targetKey) {
      setKeyPressed(true);
    }
  }

  function handleUp({ key }) {
    if (key === targetKey) {
      setKeyPressed(false);
    }
  }

  React.useEffect(() => {
    window.addEventListener("keydown", handleDown);
    window.addEventListener("keyup", handleUp);

    return () => {
      window.removeEventListener("keydown", handleDown);
      window.removeEventListener("keyup", handleUp);
    };
  }, []);

  return keyPressed;
}
```

我们可以在我们的输入应用程序中使用这个自定义钩子。当用户按下 q、l 或 w 键时，让我们将其记录到控制台中：

```
import React from "react";
import useKeyPress from "./useKeyPress";

export default function Input() {
  const [input, setInput] = React.useState("");
  const pressQ = useKeyPress("q");
  const pressW = useKeyPress("w");
  const pressL = useKeyPress("l");

  React.useEffect(() => {
    console.log(`The user pressed Q!`);
  }, [pressQ]);

  React.useEffect(() => {
    console.log(`The user pressed W!`);
  }, [pressW]);

  React.useEffect(() => {
    console.log(`The user pressed L!`);
  }, [pressL]);

  return (
    <input
      onChange={e => setInput(e.target.value)}
      value={input}
      placeholder="Type something..."
    />
  );
}
```

我们可以重复使用`useKeyPress`钩子，而不必重复编写相同的代码，而是将键按逻辑局限于`Input`组件。

Hooks 的另一个重要优势是社区可以构建和共享 Hooks。我们自己编写了`useKeyPress`钩子，但这并不是必要的。如果我们安装了它，别人已经构建并准备在我们的应用程序中使用该钩子。

以下是一些列出社区构建的所有钩子并准备在您的应用程序中使用的网站：

+   [React Use](https://oreil.ly/Ya94L)

+   [useHooks](https://oreil.ly/ZMTcR)

+   [React 钩子集合](https://oreil.ly/jlksC)

## 额外的钩子指导

就像其他组件一样，特殊函数是在你想要向编写的代码添加钩子时使用的。以下是一些常见钩子函数的简要概述：

`useState`

`useState` 钩子使开发人员能够在函数组件内更新和操作状态，而无需将其转换为类组件。该钩子的一个优点是它简单，并且不需要像其他 React Hooks 那样复杂。

`useEffect`

`useEffect` 钩子用于在函数组件中的主要生命周期事件期间运行代码。函数组件的主体不允许突变、订阅、计时器、日志记录和其他副作用。如果允许这些副作用，可能会导致 UI 内的混乱错误和一致性问题。`useEffect` 钩子防止所有这些“副作用”，并允许 UI 顺畅运行。它将 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 结合到一个地方。

`useContext`

`useContext` 钩子接受一个上下文对象，即 `React.createcontext` 返回的值，并返回该上下文的当前值。`useContext` 钩子还与 React 上下文 API 配合使用，通过各种层级共享数据，而无需将应用程序的 props 传递给其他组件。请注意，传递给 `useContext` 钩子的参数必须是上下文对象本身，任何调用 `useContext` 的组件在上下文值更改时都会重新渲染。

`useReducer`

`useReducer` 钩子提供了一种替代 `setState` 的方法。特别适合处理涉及多个子值或下一个状态依赖于上一个状态的复杂状态逻辑。它使用数组解构接受一个 reducer 函数和一个初始状态输入，并返回当前状态和一个调度函数作为输出。`useReducer` 还优化了触发深度更新的组件的性能。

## 使用 Hooks 的优缺点

使用 Hooks 的一些好处如下：

代码行数较少

Hooks 允许您按关注点和功能组织代码，而不是按生命周期组织。这不仅使代码更清晰简洁，而且更短。以下是使用 React 的可搜索产品数据表的简单有状态组件的比较，以及在使用 `useState` 关键字后使用 Hooks 的外观。

有状态组件：

```
class TweetSearchResults extends React.Component {
    constructor(props) {
      super(props);
      this.state = {
        filterText: '',
        inThisLocation: false
      };

      this.handleFilterTextChange =
                this.handleFilterTextChange.bind(this);
      this.handleInThisLocationChange =
                this.handleInThisLocationChange.bind(this);
    }

    handleFilterTextChange(filterText) {
      this.setState({
        filterText: filterText
      });
    }

    handleInThisLocationChange(inThisLocation) {
      this.setState({
        inThisLocation: inThisLocation
      })
    }

    render() {
      return (
        <div>
          <SearchBar
            filterText={this.state.filterText}
            inThisLocation={this.state.inThisLocation}
            onFilterTextChange={this.handleFilterTextChange}
            onInThisLocationChange={this.handleInThisLocationChange}
          />
          <TweetList
            tweets={this.props.tweets}
            filterText={this.state.filterText}
            inThisLocation={this.state.inThisLocation}
          />
        </div>
      );
    }
  }
```

下面是使用 Hooks 的同一组件：

```
const TweetSearchResults = ({tweets}) => {
  const [filterText, setFilterText] = useState('');
  const [inThisLocation, setInThisLocation] = useState(false);
  return (
    <div>
      <SearchBar
        filterText={filterText}
        inThisLocation={inThisLocation}
        setFilterText={setFilterText}
        setInThisLocation={setInThisLocation}
      />
      <TweetList
        tweets={tweets}
        filterText={filterText}
        inThisLocation={inThisLocation}
      />
    </div>
  );
}
```

简化复杂组件

JavaScript 类可能很难管理，与热重载结合使用可能会很困难，并且可能需要更好的缩小。React Hooks 解决了这些问题，并确保函数式编程变得更容易。使用 Hooks，我们不需要类组件。

重复使用有状态逻辑

在 JavaScript 中，类鼓励多级继承，这会快速增加整体复杂性和错误的潜在可能性。然而，Hooks 允许您在不编写类的情况下使用状态和其他 React 功能。使用 React，您始终可以重复使用有状态逻辑，而无需重复编写代码。这降低了错误发生的机会，并允许使用普通函数进行组合。

共享非可视逻辑

在 Hooks 实现之前，React 没有提取和共享非可视逻辑的方法。这最终导致了更多的复杂性，例如 HOC 模式和 Render Props，用于解决常见问题。引入 Hooks 解决了这个问题，因为它允许将有状态的逻辑提取到一个简单的 JavaScript 函数中。

当然，使用 Hooks 也有一些潜在的缺点需要牢记：

+   必须遵守其规则。使用 linter 插件，更容易知道哪个规则被破坏了。

+   需要花费相当多的时间练习以正确使用（例如 `useEffect`）。

+   需要注意错误的使用（例如 `useCallback`、`useMemo`）。

## React Hooks 与类比较

当 Hooks 被引入到 React 中时，它带来了一个新问题：我们如何知道何时使用带有 Hooks 和类组件的函数组件？借助 Hooks，即使在函数组件中，也可以获取状态和部分生命周期 Hooks。Hooks 允许您在不编写类的情况下使用本地状态和其他 React 功能。以下是帮助您做出决定的一些 Hooks 和类之间的区别：

+   Hooks 帮助避免多层次结构，并使代码更清晰。通常情况下，当您在 DevTools 中查看时，如果使用 HOC 或 Render Props，则必须重构应用程序以包含多个层次结构。

+   Hooks 提供了在 React 组件之间的统一性。类由于需要理解绑定和函数调用的上下文，使人类和机器感到困惑。

# 静态导入

`import` 关键字允许我们导入另一个模块导出的代码。默认情况下，我们静态导入的所有模块都会添加到初始捆绑包中。使用默认的 ES2015+ 导入语法 `import module from [module]` 导入的模块是静态导入的。在本节中，我们将学习在 React.js 上下文中使用静态导入的用法。

让我们看一个例子。一个简单的聊天应用包含一个 `Chat` 组件，在其中我们静态导入并渲染三个组件：`UserProfile`、`ChatList` 和 `ChatInput`，用于输入和发送消息。在 `ChatInput` 模块中，我们静态导入一个 `EmojiPicker` 组件，在用户切换表情符号时显示表情符号选择器。我们将使用 [webpack](https://oreil.ly/37e9F) 来捆绑我们的模块依赖项：

```
import React from "react";

// Statically import Chatlist, ChatInput and UserInfo
import UserInfo from "./components/UserInfo";
import ChatList from "./components/ChatList";
import ChatInput from "./components/ChatInput";

import "./styles.css";

console.log("App loading", Date.now());

const App = () => (
  <div className="App">
    <UserInfo />
    <ChatList />
    <ChatInput />
  </div>
);

export default App;
```

模块在引擎到达导入它们的行时立即执行。当您打开控制台时，您可以看到模块加载的顺序。

由于组件是静态导入的，webpack 将这些模块捆绑到初始捆绑包中。我们可以在构建应用程序后查看 webpack 创建的捆绑包：

| 资源 | main.bundle.js |
| --- | --- |
| 大小 | 1.5 MiB |
| 块 | 主 [已发布] |
| 块名称 | 主 |

我们的聊天应用程序源代码被捆绑成一个捆绑包：*main.bundle.js*。大捆绑包大小可以根据用户的设备和网络连接显著影响我们应用程序的加载时间。在 `App` 组件可以将其内容呈现到用户屏幕之前，必须首先加载和解析所有模块。

幸运的是，有许多方法可以加快加载时间！我们不必总是一次性导入所有模块：可能有基于用户交互的模块只应该在需要时才导入，就像本例中的 EmojiPicker 或在页面下方渲染的模块。与其静态导入所有组件，不如在 `App` 组件呈现其内容后动态导入模块，用户可以与我们的应用程序交互。

# 动态导入

在“静态导入”一节讨论的聊天应用程序中，有四个关键组件：`UserInfo`、`ChatList`、`ChatInput` 和 `EmojiPicker`。然而，只有这些组件中的三个在初始页面加载时立即使用：`UserInfo`、`ChatList` 和 `ChatInput`。EmojiPicker 不直接可见，只有在用户点击表情符号以切换 EmojiPicker 时才可能渲染。这意味着我们不必要地将 EmojiPicker 模块添加到初始捆绑包中，可能增加了加载时间。

为了解决这个问题，我们可以动态导入 `EmojiPicker` 组件。我们不再静态导入它，而是仅在想要显示 EmojiPicker 时才导入它。在 React 中动态导入组件的一种简单方法是使用 React Suspense。`React.Suspense` 组件接收应该动态加载的组件，使得 `App` 组件能够通过暂停 EmojiPicker 模块的导入来更快地渲染其内容。当用户点击表情符号时，EmojiPicker 组件首次被渲染。EmojiPicker 组件呈现一个 `Suspense` 组件，该组件接收懒加载的模块：在本例中是 EmojiPicker。`Suspense` 组件接受一个 fallback 属性，该属性接收在挂起组件加载时应显示的组件！

不必将 EmojiPicker 不必要地添加到初始捆绑包中，我们可以将其拆分成自己的捆绑包，并减少初始捆绑包的大小。

较小的初始捆绑包大小意味着更快的初始加载速度：用户不必在空白加载屏幕上等待太久。fallback 组件让用户知道我们的应用程序没有冻结：他们需要等待一小段时间，直到模块被处理和执行。

| 资源 | 大小 | 块 | 块名称 |
| --- | --- | --- | --- |
| *emoji-picker.bundle.js* | 1.48 KiB | 1 [emitted] | emoji-picker |
| *main.bundle.js* | 1.33 MiB | main [emitted] | main |
| *vendors~emoji-picker.bundle.js* | 171 KiB | 2 [emitted] | vendors~emoji-picker |

以前，初始捆绑包大小为 1.5 MiB，现在通过暂停 EmojiPicker 的导入，我们将其减小为 1.33 MiB。在控制台中，您可以看到一旦切换到 EmojiPicker，EmojiPicker 就会执行：

```
import React, { Suspense, lazy } from "react";
  // import Send from "./icons/Send";
  // import Emoji from "./icons/Emoji";
  const Send = lazy(() =>
    import(/*webpackChunkName: "send-icon" */ "./icons/Send")
  );
  const Emoji = lazy(() =>
    import(/*webpackChunkName: "emoji-icon" */ "./icons/Emoji")
  );
  // Lazy load EmojiPicker  when <EmojiPicker /> renders
  const Picker = lazy(() =>
    import(/*webpackChunkName: "emoji-picker" */ "./EmojiPicker")
  );

  const ChatInput = () => {
    const [pickerOpen, togglePicker] = React.useReducer(state => !state, false);

    return (
      <Suspense fallback={<p id="loading">Loading...</p>}>
        <div className="chat-input-container">
          <input type="text" placeholder="Type a message..." />
          <Emoji onClick={togglePicker} />
          {pickerOpen && <Picker />}
          <Send />
        </div>
      </Suspense>
    );
  };

  console.log("ChatInput loaded", Date.now());

  export default ChatInput;
```

构建应用程序时，我们可以看到 webpack 创建的不同捆绑包。通过动态导入 `EmojiPicker` 组件，我们将初始捆绑包大小从 1.5 MiB 减小到 1.33 MiB！虽然用户可能仍需等待 EmojiPicker 完全加载，但通过确保应用程序在用户等待组件加载时仍可呈现和交互，我们改善了用户体验。

## 可加载组件

SSR 尚不支持 React Suspense。React Suspense 的一个很好的替代方案是 *loadable-components* 库，您可以在 SSR 应用程序中使用它：

```
import React from "react";
import loadable from "@loadable/component";

import Send from "./icons/Send";
import Emoji from "./icons/Emoji";

const EmojiPicker = loadable(() => import("./EmojiPicker"), {
  fallback: <div id="loading">Loading...</div>
});

const ChatInput = () => {
  const [pickerOpen, togglePicker] = React.useReducer(state => !state, false);

  return (
    <div className="chat-input-container">
      <input type="text" placeholder="Type a message..." />
      <Emoji onClick={togglePicker} />
      {pickerOpen && <EmojiPicker />}
      <Send />
    </div>
  );
};

export default ChatInput;
```

像 React Suspense 一样，我们可以将懒加载的模块传递给可加载组件，它将在请求 EmojiPicker 模块时导入该模块。在模块加载期间，我们可以渲染一个回退组件。

尽管可加载组件在 SSR 应用程序中是 React Suspense 的一个很好的替代方案，但它们在 CSR 应用程序中也很有帮助，用于暂停模块的导入：

```
import React from "react";
  import Send from "./icons/Send";
  import Emoji from "./icons/Emoji";
  import loadable from "@loadable/component";

  const EmojiPicker = loadable(() => import("./components/EmojiPicker"), {
    fallback: <p id="loading">Loading...</p>
  });

  const ChatInput = () => {
    const [pickerOpen, togglePicker] = React.useReducer(state => !state, false);

    return (
      <div className="chat-input-container">
        <input type="text" placeholder="Type a message..." />
        <Emoji onClick={togglePicker} />
        {pickerOpen && <EmojiPicker />}
        <Send />
      </div>
    );
  };

  console.log("ChatInput loaded", Date.now());

  export default ChatInput;
```

## 交互导入

在聊天应用程序示例中，当用户点击表情符号时，我们动态导入了 `EmojiPicker` 组件。这种类型的动态导入称为 *交互导入*。我们通过用户的交互触发组件的导入。

## **可见性导入**

除了用户交互外，我们经常有一些组件在初始页面加载时不需要可见。一个很好的例子是延迟加载图像或在视口中直接不可见的组件，只有当用户向下滚动到组件并使其可见时才加载。当用户滚动到组件并使其可见时触发动态导入的操作称为 *可见性导入*。

要知道组件当前是否在我们的视口中，我们可以使用 IntersectionObserver API 或诸如 `react-loadable-visibility` 或 `react-lazyload` 这样的库，以快速将可见性导入添加到我们的应用程序中。现在我们可以看一下聊天应用程序示例，当用户看到 EmojiPicker 时，它会被导入和加载：

```
import React from "react";
import Send from "./icons/Send";
import Emoji from "./icons/Emoji";
import LoadableVisibility from "react-loadable-visibility/react-loadable";

const EmojiPicker = LoadableVisibility({
  loader: () => import("./EmojiPicker"),
  loading: <p id="loading">Loading</p>
});

const ChatInput = () => {
  const [pickerOpen, togglePicker] = React.useReducer(state => !state, false);

  return (
    <div className="chat-input-container">
      <input type="text" placeholder="Type a message..." />
      <Emoji onClick={togglePicker} />
      {pickerOpen && <EmojiPicker />}
      <Send />
    </div>
  );
};

console.log("ChatInput loading", Date.now());

export default ChatInput;
```

# 代码拆分

在前面的部分中，我们看到了如何在需要时动态导入组件。在具有多个路由和组件的复杂应用程序中，我们必须确保我们的代码在适当的时间进行优化捆绑和拆分，以允许静态和动态导入的混合使用。

您可以使用基于路由的拆分模式来拆分您的代码，或者依赖于现代打包工具如 webpack 或 Rollup 来拆分和捆绑您应用程序的源代码。

## 基于路由的拆分

特定资源可能仅在特定页面或路由上才需要，我们可以通过添加基于路由的分割来请求仅在特定路由上需要的资源。通过结合 React Suspense 或*loadable-components*等库与`react-router`，我们可以根据当前路由动态加载组件。例如：

```
import React, { lazy, Suspense } from "react";
import { render } from "react-dom";
import { Switch, Route, BrowserRouter as Router } from "react-router-dom";

const App = lazy(() => import(/* webpackChunkName: "home" */ "./App"));
const Overview = lazy(() =>
  import(/* webpackChunkName: "overview" */ "./Overview")
);
const Settings = lazy(() =>
  import(/* webpackChunkName: "settings" */ "./Settings")
);

render(
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Switch>
        <Route exact path="/">
          <App />
        </Route>
        <Route path="/overview">
          <Overview />
        </Route>
        <Route path="/settings">
          <Settings />
        </Route>
      </Switch>
    </Suspense>
  </Router>,
  document.getElementById("root")
);

module.hot.accept();
```

通过按路由懒加载组件，我们只请求包含当前路由所需代码的捆绑包。由于大多数人已经习惯在重定向期间可能有一些加载时间，这是延迟加载组件的理想场所。

## 捆绑拆分

在构建现代 Web 应用时，像 webpack 或 Rollup 这样的捆绑工具会将应用的源代码捆绑成一个或多个捆绑包。当用户访问网站时，会请求并加载用于向用户显示数据和功能的捆绑包。

JavaScript 引擎（如 V8）可以在用户请求数据时解析和编译数据。尽管现代浏览器已经进化到尽可能快速和高效地解析和编译代码，但开发人员仍然负责优化请求数据的加载和执行时间。我们希望尽可能缩短执行时间，以防止阻塞主线程。

虽然现代浏览器可以在捆绑包到达时即时流式传输，但在用户设备上绘制第一个像素点之前可能仍需相当长的时间。捆绑包越大，在引擎到达进行第一次渲染调用的行之前所需的时间就越长。在此之前，用户必须盯着空白屏幕，这可能会令人沮丧。

我们希望尽快向用户展示数据。较大的捆绑包会导致加载时间、处理时间和执行时间增加。如果能够减小捆绑包的大小以加快速度，那将是极好的。我们可以将一个包含不必要代码的巨型捆绑包分割成多个较小的捆绑包来代替。在确定我们捆绑包大小时，有一些关键指标是必须考虑的。

通过对应用程序进行捆绑拆分，我们可以减少加载、处理和执行捆绑包所需的时间。这反过来减少了在用户屏幕上绘制第一个内容（称为首次内容呈现，FCP）的时间，也减少了用于将最大组件呈现到屏幕上的时间或最大内容呈现时间（LCP）指标。

尽管在屏幕上看到数据很好，但我们希望看到的不仅仅是内容。为了拥有完全功能的应用程序，我们希望用户也能与其进行交互。只有在捆绑包加载和执行后，UI 才变得可交互。所有内容在屏幕上绘制并变得可交互所需的时间称为交互时间（TTI）。

更大的捆绑包不一定意味着更长的执行时间。可能发生的情况是，我们加载了用户甚至可能根本不使用的大量代码。捆绑包的某些部分将仅在特定用户交互上执行，用户可能会执行或不执行。

引擎仍然必须加载、解析和编译甚至在用户可以在屏幕上看到任何内容之前都没有使用的代码。尽管由于浏览器处理这两个步骤的高效方式，解析和编译成本实际上可以忽略不计，但获取比必要更大的捆绑包可能会影响应用程序的性能。在低端设备或较慢网络上的用户在捆绑包被获取之前将看到加载时间显著增加。

不要最初请求当前导航中优先级不高的代码部分，我们可以将这些代码与渲染初始页面所需的代码分开。

# PRPL 模式

使应用程序对全球用户群体可访问可能是一个挑战。应用程序应在低端设备和网络连接质量差的地区表现出色。为了确保我们的应用程序在恶劣条件下能够尽可能高效地加载，我们可以使用推送渲染预缓存懒加载（PRPL）模式。

PRPL 模式专注于四个主要性能考虑因素：

+   *高效推送*关键资源可以最小化与服务器之间的往返次数并减少加载时间。

+   尽快*渲染*初始路由以改善用户体验。

+   在后台*预缓存*经常访问的路由的资产，以最小化对服务器的请求次数并实现更好的离线体验。

+   *懒加载*不经常请求的路由或资源。

当我们访问一个网站时，浏览器会向服务器请求所需的资源。入口点指向的文件通常是从服务器返回的，通常是我们应用程序的初始 HTML 文件。浏览器的 HTML 解析器在从服务器接收数据时立即开始解析这些数据。如果解析器发现需要更多资源，比如样式表或脚本，将向服务器发送额外的 HTTP 请求以获取这些资源。

反复请求资源并不理想，因为我们试图最小化客户端和服务器之间的往返次数。

我们长时间使用 HTTP/1.1 在客户端和服务器之间通信。尽管与 HTTP/1.0 相比，HTTP/1.1 引入了许多改进，比如在发送带有保持活动标头的新 HTTP 请求之前保持客户端和服务器之间的 TCP 连接活动，仍然存在一些问题需要解决。HTTP/2 相对于 HTTP/1.1 引入了重大变化，使我们能够优化客户端和服务器之间的消息交换。

虽然 HTTP/1.1 在请求和响应中使用换行分隔的纯文本协议，但 HTTP/2 将请求和响应拆分为更小的帧。包含头部和主体字段的 HTTP 请求至少分为两个帧：头部帧和数据帧。

HTTP/1.1 在客户端和服务器之间最多允许六个 TCP 连接。在可以通过同一 TCP 连接发送新请求之前，必须解决上一个请求。如果最后一个请求解决时间较长，这个请求将阻塞其他请求的发送。这种常见问题称为先行线阻塞，会增加特定资源的加载时间。

HTTP/2 使用双向流。单个 TCP 连接可以在客户端和服务器之间传输多个请求和响应帧。一旦服务器接收到该特定请求的所有请求帧，它将重新组装它们并生成响应帧。这些响应帧被发送回客户端，客户端重新组装它们。由于流是双向的，我们可以通过同一流发送请求和响应帧。

HTTP/2 通过允许在上一个请求解决之前在同一 TCP 连接上发送多个请求来解决先行线阻塞问题！HTTP/2 还引入了一种更优化的数据获取方式，称为服务器推送。服务器可以通过“推送”这些资源自动发送额外资源，而不是每次通过发送 HTTP 请求显式请求资源。

客户端接收到额外资源后，这些资源将被存储在浏览器缓存中。当解析入口文件时发现这些资源时，浏览器可以快速从缓存获取资源，而不是向服务器发出 HTTP 请求。

虽然推送资源减少了接收额外资源的时间，但服务器推送并不具备 HTTP 缓存感知性。推送的资源在下次访问网站时将不可用，必须再次请求。为了解决这个问题，PRPL 模式在初始加载后使用服务工作线程缓存这些资源，以确保客户端不会发出不必要的请求。

作为网站作者，我们通常清楚哪些资源是在早期获取的关键资源，而浏览器则尽力猜测。通过添加预加载资源提示到关键资源，我们可以帮助浏览器。

通过告诉浏览器您希望预加载特定资源，您正在告诉浏览器您希望比浏览器正常发现时更早获取它。预加载是优化加载当前路由关键资源所需时间的一个很好的方式。

尽管预加载资源是减少往返次数和优化加载时间的好方法，但过多地推送文件可能会有害。浏览器的缓存是有限的，通过请求客户端不需要的资源，您可能会不必要地使用带宽。PRPL 模式侧重于优化初始加载。在完全呈现初始路由之前，不会加载其他任何资源。

我们可以通过将应用程序拆分为小型、高效的捆包来实现这一点。这些捆包应允许用户在需要时仅加载他们所需的资源，同时最大化缓存使用。

缓存较大的捆包可能会成为问题。多个捆包可能共享相同的资源。

浏览器需要帮助识别哪些捆绑资源在多个路由之间共享，因此无法缓存这些资源。缓存资源对于减少与服务器的往返次数以及使我们的应用支持离线至关重要。

在使用 PRPL 模式时，我们需要确保请求的捆包包含我们在该时刻需要的最小资源量，并且浏览器可以缓存这些资源。在某些情况下，这可能意味着根本不使用任何捆包会更高效，我们可以简单地使用未捆绑的模块。

通过将应用程序捆绑为最小资源动态请求的好处可以通过配置浏览器和服务器以支持 HTTP/2 推送和有效缓存资源轻松模拟。对于不支持 HTTP/2 服务器推送的浏览器，我们可以创建优化的构建以最小化往返次数。客户端无需知道它是否接收到捆绑或未捆绑的资源：服务器会为每个浏览器提供适当的构建。

PRPL 模式通常将应用外壳作为其主要入口点，这是一个包含大部分应用逻辑并在多个路由之间共享的最小文件。它还包括应用的路由器，可以动态请求所需资源。

PRPL 模式确保在用户设备上看到初始路由之前不会请求或呈现任何其他资源。一旦成功加载初始路由，服务工作线程可以安装以在后台获取其他经常访问路由的资源。

由于这些数据是在后台获取的，用户不会遇到任何延迟。如果用户想要访问由服务工作线程缓存的经常访问路由，则服务工作线程可以快速从缓存中获取所需资源，而无需向服务器发送请求。

对于不经常访问的路由，可以动态导入资源。

# 加载优先级

加载优先级模式鼓励您明确优先处理特定资源的请求，这些资源您知道将较早需要。

预加载（`<link rel="preload">`）是浏览器的优化，允许较晚发现的关键资源提前请求。如果您习惯手动思考如何排序关键资源的加载，它可能对核心 Web Vitals（CWV）中的加载性能和指标产生积极影响。然而，预加载并非万能药，需要考虑一些权衡：

```
<link rel="preload" href="emoji-picker.js" as="script">
  ...
  </head>
  <body>
    ...
    <script src="stickers.js" defer></script>
    <script src="video-sharing.js" defer></script>
    <script src="emoji-picker.js" defer></script>
```

当优化像交互时间 (TTI) 或首次输入延迟 (FID) 这样的指标时，预加载对于加载 JavaScript 包（或块）以实现交互性是有帮助的。请记住，在使用预加载时需要格外小心，因为您希望在改善交互性的同时避免延迟必要的资源（如主要图像或字体），这些资源对于首次内容绘制 (FCP) 或最大内容绘制 (LCP) 是必要的。

如果您试图优化第一方 JavaScript 的加载，请考虑在文档`<head>`中使用`<script defer>`而不是`<body>`，以帮助提前发现这些资源。

## 单页应用中的预加载

虽然预取是缓存即将请求的资源的好方法，但我们可以预加载需要立即使用的资源。这可能是在初始渲染中使用的特定字体或用户立即看到的某些图片。

假设我们的`EmojiPicker`组件在初始渲染时应立即可见。虽然您不应将其包含在主包中，但应并行加载它。与预取一样，我们可以添加魔法注释，告知 webpack 此模块应预加载：

```
const EmojiPicker = import(/* webpackPreload: true */ "./EmojiPicker");
```

在构建应用程序后，我们可以看到`EmojiPicker`将被预取。实际输出在我们文档头部具有`rel="preload"`的`link`标签中可见：

```
<link rel="prefetch" href="emoji-picker.bundle.js" as="script" />
<link rel="prefetch" href="vendors~emoji-picker.bundle.js" as="script" />
```

预加载的`EmojiPicker`可以与初始包并行加载。与预取不同，浏览器在预取资源时仍需判断其网络连接和带宽是否足够，而预加载的资源将无论如何都会预加载。

而不是在初始渲染后等待`EmojiPicker`加载完成，资源将立即可用于我们。随着我们以更考虑周到的顺序加载资产，初始加载时间可能会显著增加，这取决于用户的设备和互联网连接。仅预加载那些在初始渲染后约 1 秒钟必须可见的资源。

## 预加载 + 异步 Hack

如果你希望浏览器高优先级下载脚本但不阻塞解析器等待脚本，你可以利用此处展示的预加载 + `async` 技巧。在这种情况下，预加载可能会延迟其他资源的下载，但这是开发者需要权衡的一个折衷：

```
<link rel="preload" href="emoji-picker.js" as="script" />

<script src="emoji-picker.js" async></script>
```

## Chrome 95+中的预加载

由于 Chrome 95+中预加载队列跳过行为的一些修复，该功能现在稍微更安全。Chrome 关于预加载的新建议由 Chrome 的 Pat Meenan 提出：

+   将其放在 HTTP 标头中将超越其他一切。

+   一般来说，预加载将按照解析器处理它们的顺序进行加载，适用于中等及以上的任何情况，因此在 HTML 开头放置预加载时要小心。

+   字体预加载可能最好放在头部的末尾或者主体的开头。

+   导入预加载应该在需要导入的 `script` 标签之后进行（这样实际脚本会首先加载/解析）。

+   图像预加载的优先级较低，应相对于 `async` 脚本和其他低/最低优先级标签进行排序。

# 列表虚拟化

列表虚拟化有助于提高大型数据列表的渲染性能。在动态列表中，您仅渲染可见的内容行，而不是整个列表。渲染的行仅是完整列表的一个小子集，而可见的内容（窗口）随用户滚动而移动。在 React 中，您可以通过 [`react-virtualized`](https://oreil.ly/3XThZ) 来实现这一点。这是由 [Brian Vaughn](https://oreil.ly/hCqQq) 开发的窗口化库，仅在滚动视口中渲染当前可见的项目。这意味着您无需支付一次性渲染成千上万行数据的成本。

## 窗口化/虚拟化如何工作？

虚拟化项目列表涉及维护和移动列表周围的窗口。`react-virtualized` 中的窗口化通过以下方式工作：

+   拥有一个小的容器 DOM 元素（例如，`<ul>`），具有相对定位（窗口）。

+   拥有一个用于滚动的大 DOM 元素。

+   绝对定位容器内的子元素，设置它们的样式为 top、left、width 和 height。

+   而不是一次性渲染列表中的成千上万个元素（可能导致较慢的初始渲染或影响滚动性能），虚拟化专注于仅渲染对用户可见的项目。

这可以帮助在中低端设备上保持列表渲染速度快。随着用户滚动，您可以获取/显示更多项目，卸载先前的条目并替换为新条目。

[`react-window`](https://oreil.ly/H_Rx7) 是由相同作者重写的 `react-virtualized`，旨在更小、更快且更易于树抖动。在可树抖动库中，尺寸取决于您选择使用的哪些 API 表面。使用它代替 `react-virtualized`，我看到大约 20 到 30 KB（经过压缩）的节省。这两个包的 API 大致相似；在它们不同的地方，`react-window` 倾向于更简单。

在 `react-window` 案例中，以下是主要组件。

## 列表

列表渲染一个窗口化列表（行）的元素，这意味着仅显示给用户可见的行。列表使用一个网格（内部）来渲染行，将属性传递给该内部网格。以下代码片段展示了在 React 中渲染列表与使用 `react-window` 的区别。

使用 React 渲染简单数据的列表（`itemsArray`）：

```
import React from "react";
import ReactDOM from "react-dom";

const itemsArray = [
  { name: "Drake" },
  { name: "Halsey" },
  { name: "Camila Cabello" },
  { name: "Travis Scott" },
  { name: "Bazzi" },
  { name: "Flume" },
  { name: "Nicki Minaj" },
  { name: "Kodak Black" },
  { name: "Tyga" },
  { name: "Bruno Mars" },
  { name: "Lil Wayne" }, ...
]; // our data

const Row = ({ index, style }) => (
  <div className={index % 2 ? "ListItemOdd" : "ListItemEven"} style={style}>
    {itemsArray[index].name}
  </div>
);

const Example = () => (
  <div
    style=
    class="List"
  >
    {itemsArray.map((item, index) => Row({ index }))}
  </div>
);

ReactDOM.render(<Example />, document.getElementById("root"));
```

使用 `react-window` 渲染列表：

```
import React from "react";
import ReactDOM from "react-dom";
import { FixedSizeList as List } from "react-window";

const itemsArray = [...]; // our data

const Row = ({ index, style }) => (
  <div className={index % 2 ? "ListItemOdd" : "ListItemEven"} style={style}>
    {itemsArray[index].name}
  </div>
);

const Example = () => (
  <List
    className="List"
    height={150}
    itemCount={itemsArray.length}
    itemSize={35}
    width={300}
  >
    {Row}
  </List>
);

ReactDOM.render(<Example />, document.getElementById("root"));
```

## 网格

Grid 在垂直和水平轴向上使用虚拟化渲染表格数据。它只根据当前水平/垂直滚动位置需要渲染填充自身的 Grid 单元格。

如果我们想要使用网格布局渲染与之前相同的列表，假设我们的输入是多维数组，我们可以使用`FixedSizeGrid`来实现如下：

```
import React from 'react';
import ReactDOM from 'react-dom';
import { FixedSizeGrid as Grid } from 'react-window';

const itemsArray = [
  [{},{},{},...],
  [{},{},{},...],
  [{},{},{},...],
  [{},{},{},...],
];

const Cell = ({ columnIndex, rowIndex, style }) => (
  <div
    className={
      columnIndex % 2
        ? rowIndex % 2 === 0
          ? 'GridItemOdd'
          : 'GridItemEven'
        : rowIndex % 2
          ? 'GridItemOdd'
          : 'GridItemEven'
    }
    style={style}
  >
    {itemsArray[rowIndex][columnIndex].name}
  </div>
);

const Example = () => (
  <Grid
    className="Grid"
    columnCount={5}
    columnWidth={100}
    height={150}
    rowCount={5}
    rowHeight={35}
    width={300}
  >
    {Cell}
  </Grid>
);

ReactDOM.render(<Example />, document.getElementById('root'));
```

## Web 平台的改进

一些现代浏览器现在支持[CSS `content-visibility`](https://oreil.ly/l-B70)。`content-visibility:auto`允许您在需要之前跳过渲染和绘制屏幕外内容。如果您有一个包含昂贵渲染的长 HTML 文档，请考虑尝试此属性。

对于渲染动态内容列表，我仍然建议使用像`react-window`这样的库。要比使用`content-visibility:hidden`版本或像许多列表虚拟化库今天可能做的那样，依据屏幕外条件使用`display:none`或移除 DOM 节点更具侵略性的版本。

# 结论

再次强调，使用预加载要谨慎，并始终评估其在生产环境中的影响。如果您的图片预加载早于其他资源，这可以帮助浏览器发现它（并相对其他资源进行排序）。当使用不当时，预加载可能会导致图片延迟 FCP（例如 CSS、字体），这与您的意图相反。此外，请注意，为了使这种重新排序效果有效，还取决于服务器正确地优先处理请求。

在需要获取但不执行脚本的情况下，您可能会发现`<link rel="preload">`有帮助。以下的[web.dev 文章](https://oreil.ly/kgnfa)讨论了如何使用预加载来：

+   [加载交互所需的关键脚本](https://oreil.ly/bwZC9)

+   [预加载您的 LCP 图像](https://oreil.ly/4N3VO)

+   [在防止布局移位的同时加载字体](https://oreil.ly/Up2iQ)

# 摘要

在本章中，我们讨论了推动现代 Web 应用程序架构和设计的一些关键考虑因素。我们还看到了 React.js 设计模式如何解决这些问题的不同方式。

本章前面，我们介绍了 CSR、SSR 和 hydration 的概念。JavaScript 对页面性能有重大影响。渲染技术的选择会影响 JavaScript 在页面生命周期中的加载或执行时间点。因此，在讨论 JavaScript 模式和下一章主题时，讨论渲染模式非常重要。
