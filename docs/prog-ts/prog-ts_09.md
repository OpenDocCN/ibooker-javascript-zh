# 第九章：前端和后端框架

虽然你可以从头开始构建应用程序的每个部分——在服务器上的网络和数据库层，前端的用户界面框架和状态管理解决方案——但你可能不应该这样做。很难把细节做好，幸运的是，许多这些前端和后端的难题已经被其他工程师解决了。通过利用现有的工具、库和框架来构建前端和后端的东西，我们可以在构建自己的应用程序时快速迭代并站在稳定的基础上。

在本章中，我们将介绍一些在客户端和服务器上解决常见问题的最流行的工具和框架。我们将讨论您可能会使用每个框架以及如何安全地将其集成到您的 TypeScript 应用程序中。

# 前端框架

TypeScript 自然适合于前端应用程序的世界。凭借其对 JSX 的丰富支持以及安全建模可变性的能力，TypeScript 为您的应用程序提供结构和安全性，并使得在快节奏的前端开发环境中编写正确、可维护的代码变得更加容易。

当然，所有内置的 DOM API 都是类型安全的。要从 TypeScript 中使用它们，只需在项目的 *tsconfig.json* 中包含它们的类型声明：

```
{
  "compilerOptions": {
    "lib": ["dom", "es2015"]
  }
}
```

这将告诉 TypeScript 在检查代码时包含*lib.dom.d.ts*——其内置的浏览器和 DOM 类型声明。

###### 注意

`lib` *tsconfig.json* 选项只是告诉 TypeScript 在处理项目中的代码时包含一组特定的类型声明；它不会生成任何额外的代码，或者生成任何在运行时存在的 JavaScript。例如，它不会让 DOM 在 NodeJS 环境中奇迹般地工作（您的代码会编译，但在运行时会失败）——您需要确保您的类型声明与您的 JavaScript 环境在运行时实际支持的内容匹配。跳到 “构建您的 TypeScript 项目” 了解更多信息。

启用 DOM 类型声明后，您可以安全地使用 DOM 和浏览器 API 来执行诸如以下操作：

```
// Read properties from the global window object
let model = {
  url: window.location.href
}

// Create an <input /> element
let input = document.createElement('input')

// Give it some CSS classes
input.classList.add('Input', 'URLInput')

// When the user types, update the model
input.addEventListener('change', () =>
  model.url = input.value.toUpperCase()
)

// Inject the <input /> into the DOM
document.body.appendChild(input)
```

当然，所有这些代码都经过类型检查，并且具有像编辑器中的自动完成之类的常规好处。例如，考虑类似于这样的内容：

```
document.querySelector('.Element').value // Error TS2339: Property 'value' does
                                         // not exist on type 'Element'.
```

TypeScript 将抛出一个错误，因为`querySelector`的返回类型是可空的。

对于简单的前端应用程序，这些低级 DOM API 已经足够，并且将为您提供所需的内容，以便在浏览器中进行安全的、类型引导的编程。大多数真实世界的前端应用程序使用框架来抽象掉 DOM 渲染和重新渲染、数据绑定和事件处理的工作方式。以下部分将为您提供如何有效地使用 TypeScript 与一些最流行的浏览器框架的一些指针。

## React

React 是当今最受欢迎的前端框架之一，在类型安全方面非常出色。

React 如此安全的原因在于 React 组件——React 应用程序的基本构建块——在 TypeScript 中既定义又消费。这种属性在前端框架中很难找到，意味着组件定义和消费者都经过了类型检查。您可以使用类型来表达诸如“此组件接受用户 ID 和颜色”或“此组件只能具有列表项作为子元素”的内容。然后，TypeScript 会强制执行这些约束，验证您的组件是否按照其所说的方式运行。

在前端应用程序的*视图层*中，关于组件定义和消费者的这种安全性是致命的。传统上，视图是程序员们花费数千小时挠头并愤怒地刷新浏览器的地方，因为错别字、遗漏的属性、错误输入的参数和不正确的嵌套元素。当您开始使用 TypeScript 和 React 编写视图时，您和您的团队在前端的生产力将翻倍。

### JSX 入门指南

使用 React 时，您可以使用称为*JavaScript XML*（JSX）的特殊 DSL 来定义视图，直接嵌入到 JavaScript 代码中。它在您的 JavaScript 中看起来像 HTML。然后，您可以运行您的 JavaScript 通过 JSX 编译器，将那些奇特的 JSX 语法重写为常规的 JavaScript 函数调用。

这个过程看起来像这样。假设您正在为朋友的餐厅构建一个菜单应用程序，并列出早午餐菜单上的一些项目，使用以下 JSX：

```
<ul class='list'>
  <li>Homemade granola with yogurt</li>
  <li>Fantastic french toast with fruit</li>
  <li>Tortilla Española with salad</li>
</ul>
```

在像 Babel 的[`transform-react-jsx`插件](http://bit.ly/2uENY4M)这样的 JSX 编译器中运行该代码后，您将得到以下输出：

```
React.createElement(
  'ul',
  {'class': 'list'},
  React.createElement(
    'li',
    null,
    'Homemade granola with yogurt'
  ),
  React.createElement(
    'li',
    null,
    'Fantastic French toast with fruit'
  ),
  React.createElement(
    'li',
    null,
    'Tortilla Española with salad'
  )
);

```

# TSC 标志：esModuleInterop

因为 JSX 编译为对`React.createElement`的调用，请确保在每个使用 JSX 的文件中导入 React 库，以便在作用域中有名为`React`的变量：

```
import React from 'react'
```

别担心——如果您忘记了，TypeScript 会提醒您：

```
<ul /> // Error TS2304: Cannot find name 'React'.
```

还请注意，我已在我的*tsconfig.json*中设置了`{"esModuleInterop": true}`以支持导入`React`而无需通配符（`*`）导入。如果您在跟进，请在您自己的*tsconfig.json*中启用`esModuleInterop`，或者改用通配符导入：

```
import * as React from 'react'
```

JSX 的好处在于，您可以编写看起来非常像普通 HTML 的内容，然后自动将其编译为友好于 JavaScript 引擎的格式。作为一名工程师，您只需使用一种熟悉的、高级的、声明性的 DSL，而无需处理实现细节。

您不需要 JSX 来使用 React（您可以直接编写已编译的代码，它也能正常工作），并且您可以在没有 React 的情况下使用 JSX（JSX 标签编译为的特定函数调用——在前面的示例中是`React.createElement`——是可以配置的），但是 React 与 JSX 的组合是神奇的，让编写视图非常有趣，也非常安全。

### TSX = JSX + TypeScript

包含 JSX 的文件使用扩展名 *.jsx*。而包含 JSX 的 TypeScript 文件使用扩展名 *.tsx*。TSX 对 JSX 就像 TypeScript 对 JavaScript 一样—是一个编译时的安全和辅助层，帮助你更高效地生产代码，减少错误。要为你的项目启用 TSX 支持，请在你的 *tsconfig.json* 文件中添加以下行：

```
{
  "compilerOptions": {
    "jsx": "react"
  }
}

```

`jsx` 指令在撰写时有三种模式：

`react`

使用 JSX pragma 编译 JSX 到一个扩展名为 *.js* 的文件（默认为 `React.createElement`）。

`react-native`

不编译 JSX，但生成一个扩展名为 *.js* 的文件。

`preserve`

对 JSX 进行类型检查，但不要将其编译掉，并生成一个扩展名为 *.jsx* 的文件。

在幕后，TypeScript 以一种可插拔的方式暴露了几个用于 TSX 类型的钩子。这些是 `global.JSX` 命名空间上的特殊类型，TypeScript 在整个程序中查看它们作为 TSX 类型的真实来源。如果你只是使用 React，你不需要深入到这个低级别；但是如果你正在构建自己的 TypeScript 库，该库使用 TSX（而不是 React）—或者如果你好奇 React 类型声明是如何做到的—可以查看 附录 G。

### 使用 TSX 与 React

React 允许我们声明两种类型的组件：函数组件和类组件。无论是哪种类型的组件，都需要一些属性并渲染一些 TSX。从消费者的角度来看，它们是相同的。

声明和渲染一个函数组件看起来像这样：

```
import React from 'react' ![1](img/1.png)

type Props = { ![2](img/2.png)
  isDisabled?: boolean
  size: 'Big' | 'Small'
  text: string
  onClick(event: React.MouseEvent<HTMLButtonElement>): void ![3](img/3.png)
}

export function FancyButton(props: Props) { ![4](img/4.png)
  const [toggled, setToggled] = React.useState(false) ![5](img/5.png)
  return <button
    className={'Size-' + props.size}
    disabled={props.isDisabled || false}
    onClick={event => {
      setToggled(!toggled)
      props.onClick(event)
    }}
  >{props.text}</button>
}

let button = <FancyButton ![6](img/6.png)
  size='Big'
  text='Sign Up Now'
  onClick={() => console.log('Clicked!')}
/>
```

![1](img/#co_frontend_and_backend_frameworks_CO1-1)

为了在使用 React 和 TSX 时进行类型检查，我们必须将 `React` 变量引入当前作用域。由于 TSX 被编译为 `React.createElement` 函数调用，这意味着我们需要导入 `React`，以便在运行时定义它。

![2](img/#co_frontend_and_backend_frameworks_CO1-2)

我们首先声明可以传递给我们的 `FancyButton` 组件的具体 props 集合。按照约定，`Props` 总是一个对象类型，并且名为 `Props`。对于我们的 `FancyButton` 组件，`isDisabled` 是可选的，而其余的 props 都是必需的。

![3](img/#co_frontend_and_backend_frameworks_CO1-3)

当使用 React 事件时，React 有自己的一套包装器类型用于 DOM 事件。请确保使用 React 的事件类型，而不是常规的 DOM 事件类型。

![4](img/#co_frontend_and_backend_frameworks_CO1-4)

函数组件只是一个普通函数，最多有一个参数（`props` 对象），并返回一个可以被 React 渲染的类型。React 是宽松的，可以渲染多种类型：TSX、字符串、数字、布尔值、`null` 和 `undefined`。

![5](img/#co_frontend_and_backend_frameworks_CO1-5)

我们使用 React 的`useState`钩子来为函数组件声明本地状态。`useState`是 React 中可用的几个钩子之一，您可以组合它们来创建自己的自定义钩子。请注意，因为我们将初始值`false`传递给`useState`，TypeScript 能够推断出该状态片段是一个`boolean`；如果我们使用了 TypeScript 无法推断的类型，例如数组，我们将需要显式地绑定类型（例如，使用 useState<number[]>;([])）。

![6](img/#co_frontend_and_backend_frameworks_CO1-6)

我们使用 TSX 语法来创建`FancyButton`的实例。`<FancyButton />`语法与调用`FancyButton`几乎相同，但它让 React 来管理`FancyButton`的生命周期。

就是这样。TypeScript 强制执行以下规则：

+   JSX 格式良好。标签已关闭且嵌套正确，标签名称没有拼写错误。

+   当我们实例化一个`<FancyButton />`时，我们将所有必需的——以及任何可选的——props 传递给`FancyButton`（`size`，`text`和`onClick`），并确保 props 的类型是正确的。

+   我们不会向`FancyButton`传递任何多余的 props，只传递必需的 props。

类组件类似：

```
import React from 'react' ![1](img/1.png)
import {FancyButton} from './FancyButton'

type Props = { ![2](img/2.png)
  firstName: string
  userId: string
}

type State = { ![3](img/3.png)
  isLoading: boolean
}

class SignupForm extends React.Component<Props, State> { ![4](img/4.png)
  state = { ![5](img/5.png)
    isLoading: false
  }
  render() { ![6](img/6.png)
    return <> ![7](img/7.png)
      <h2>Sign up for a 7-day supply of our tasty
          toothpaste now, {this.props.firstName}.</h2>
      <FancyButton
        isDisabled={this.state.isLoading}
        size='Big'
        text='Sign Up Now'
        onClick={this.signUp}
      />
    </>
  }
  private signUp = async () => { ![8](img/8.png)
    this.setState({isLoading: true})
    try {
      await fetch('/api/signup?userId=' + this.props.userId)
    } finally {
      this.setState({isLoading: false})
    }
  }
}

let form = <SignupForm firstName='Albert' userId='13ab9g3' /> ![9](img/9.png)
```

![1](img/#co_frontend_and_backend_frameworks_CO2-1)

和之前一样，我们导入`React`以将其引入作用域。

![2](img/#co_frontend_and_backend_frameworks_CO2-2)

类似于之前，我们声明一个`Props`类型来定义在创建`<SignupForm />`实例时需要传递的数据。

![3](img/#co_frontend_and_backend_frameworks_CO2-3)

我们声明一个`State`类型来模拟组件的本地状态。

![4](img/#co_frontend_and_backend_frameworks_CO2-4)

要声明一个类组件，我们需要扩展`React.Component`基类。

![5](img/#co_frontend_and_backend_frameworks_CO2-5)

我们使用属性初始化器来为本地状态声明默认值。

![6](img/#co_frontend_and_backend_frameworks_CO2-6)

类似于函数组件，类组件的`render`方法返回可以由 React 渲染的内容：TSX，字符串，数字，布尔值，`null`或`undefined`。

![7](img/#co_frontend_and_backend_frameworks_CO2-7)

TSX 支持使用特殊的`<>...</>`语法来进行片段处理。片段是一个无名称的 TSX 元素，用于包装其他 TSX 元素，可以避免在需要返回单个 TSX 元素的地方渲染额外的 DOM 元素。例如，React 组件的`render`方法需要返回单个 TSX 元素；为此，我们可以使用`<div>`或任何其他元素来包装我们的代码，但这样做会在渲染时产生不必要的开销。

![8](img/#co_frontend_and_backend_frameworks_CO2-8)

我们使用箭头函数定义`signUp`，以确保函数中的`this`不会被重新绑定。

![9](img/#co_frontend_and_backend_frameworks_CO2-9)

最后，我们实例化我们的 `SignupForm`。就像实例化函数组件一样，我们也可以直接用 `new SignupForm({firstName: 'Albert', userId: '13ab9g3'})` 来实例化它，但这意味着 React 无法为我们管理 `SignupForm` 实例的生命周期。

注意在这个例子中我们如何混合匹配基于值的（`FancyButton`，`SignupForm`）和内置的（`section`，`h2`）组件。我们让 TypeScript 工作来验证以下几点：

+   所有必需的状态字段都在 `state` 初始化器或构造函数中定义了。

+   我们在 `props` 和 `state` 上访问的任何内容都确实存在，并且是我们认为的类型。

+   在 React 中，我们不直接写入 `this.state`，因为状态更新必须通过 `setState` API 进行。

+   调用 `render` 确实返回一些 JSX。

使用 TypeScript，您可以使您的 React 代码更安全，从而成为一个更好、更快乐的人。

###### 注意

我们没有使用 React 的 `PropTypes` 特性，这是一种在运行时声明和检查 props 类型的方式。因为 TypeScript 已经在编译时为我们检查了类型，所以我们不需要再次检查。

## Angular 6/7

由 Shyam Seshadri 贡献

Angular 是一个比 React 更全面的前端框架，不仅支持渲染视图，还支持发送和管理网络请求、路由和依赖注入。它从头开始与 TypeScript 协作（事实上，框架本身就是用 TypeScript 编写的！）。

Angular 的核心工作方式是 Angular CLI 中内置的 Ahead-of-Time（AoT）编译器，这是 Angular 命令行实用程序，它获取您用 TypeScript 注解提供的类型信息，并使用该信息将您的代码编译为常规 JavaScript。Angular 不直接调用 TypeScript，而是在最终委托给 TypeScript 之前对您的代码应用一系列优化和转换。

让我们看看 Angular 如何使用 TypeScript 及其 AoT 编译器来确保编写前端应用的安全性。

### 脚手架

要初始化一个新的 Angular 项目，首先使用 NPM 全局安装 Angular CLI：

```
npm install @angular/cli --global
```

然后，使用 Angular CLI 初始化一个新的 Angular 应用程序：

```
ng new my-angular-app
```

按照提示操作，Angular CLI 将为您设置一个简易的 Angular 应用程序。

在本书中，我们不会深入讲解 Angular 应用程序的结构，或者如何配置和运行它。有关详细信息，请参阅[官方 Angular 文档](https://angular.io/docs)。

### 组件

让我们构建一个 Angular 组件。Angular 组件类似于 React 组件，并包括一种描述组件 DOM 结构、样式和控制器的方法。使用 Angular，您可以使用 Angular CLI 生成组件样板，然后手动填写详细信息。Angular 组件包含几个不同的文件：

+   描述组件渲染的 DOM 的模板

+   一组 CSS 样式

+   组件类，这是一个 TypeScript 类，用于定义你的组件业务逻辑。

让我们从组件类开始：

```
import {Component, OnInit} from '@angular/core'

@Component({
  selector: 'simple-message',
  styleUrls: ['./simple-message.component.css'],
  templateUrl: './simple-message.component.html'
})
export class SimpleMessageComponent implements OnInit {
  message: string
  ngOnInit() {
    this.message = 'No messages, yet'
  }
}
```

总的来说，这是一个相当标准的 TypeScript 类，只是有几个不同点展示了 Angular 如何利用 TypeScript。具体来说：

+   Angular 的生命周期钩子作为 TypeScript 接口可用——只需声明你要 `implement` 的哪些接口（`ngOnChanges`、`ngOnInit` 等）。然后 TypeScript 强制要求你实现符合所需生命周期钩子的方法。在本例中，我们实现了 `OnInit` 接口，这要求我们实现 `ngOnInit` 方法。

+   Angular 大量使用 TypeScript 装饰器（见 “装饰器”）来声明与 Angular 组件、服务和模块相关的元数据。在这个例子中，我们使用了 `selector` 来声明如何消费我们的组件，并使用了 `templateUrls` 和 `styleUrl` 来链接 HTML 模板和 CSS 样式表到我们的组件。

# TSC 标志：fullTemplateTypeCheck

要为您的 Angular 模板启用类型检查（您应该！），请确保在您的 *tsconfig.json* 中启用 `fullTemplateTypeCheck`：

```
{
  "angularCompilerOptions": {
    "fullTemplateTypeCheck": true
  }
}
```

注意 `angularCompilerOptions` 并非指定 TSC 的选项。相反，它定义了特定于 Angular 的 AoT 编译器的编译器标志。

### 服务

Angular 自带内置的依赖注入器（DI），这是框架实例化服务并将它们作为参数传递给依赖于它们的组件和服务的一种方式。这可以使得实例化和测试服务和组件更加容易。

让我们更新 `SimpleMessageComponent` 来注入一个依赖项 `MessageService`，负责从服务器获取消息：

```
import {Component, OnInit} from '@angular/core'
`import` `{``MessageService``}` `from` `'../services/message.service'`

@Component({
  selector: 'simple-message',
  templateUrl: './simple-message.component.html',
  styleUrls: ['./simple-message.component.css']
})
export class SimpleMessageComponent implements OnInit {
  message: string
  `constructor``(`
    `private` `messageService``:` `MessageService`
  `)` `{``}`
  ngOnInit() {
    `this``.``messageService``.``getMessage``(``)``.``subscribe``(``response` `=``>`
      `this``.``message` `=` `response``.``message`
    `)`
  }
}

```

Angular 的 AoT 编译器查看您的组件 `constructor` 接受的参数，提取它们的类型（例如 `MessageService`），并搜索相关的依赖注入器的依赖映射以查找该特定类型的依赖项。如果尚未实例化该依赖项，则实例化它（使用 `new`），并将其传递给 `SimpleMessageComponent` 实例的构造函数。所有这些依赖注入的内容相当复杂，但随着应用程序的增长以及根据应用程序配置（例如 `ProductionAPIService` 与 `DevelopmentAPIService`）或测试时使用的依赖项（`MockAPIService`）的不同，它可能非常方便。

现在让我们快速看一下如何定义一个服务：

```
import {Injectable} from '@angular/core'
import {HttpClient} from '@angular/common/http'

@Injectable({
  providedIn: 'root'
})
export class MessageService {
  constructor(private http: HttpClient) {}
  getMessage() {
    return this.http.get('/api/message')
  }
}
```

每当我们在 Angular 中创建一个服务时，我们都会再次使用 TypeScript 装饰器将其注册为可注入的东西，并定义它是在应用程序的根级别提供还是在子模块中提供。在这里，我们注册了服务 `MessageService`，允许我们在应用程序的任何地方注入它。在任何组件或服务的构造函数中，我们只需请求一个 `MessageService`，Angular 将神奇地负责传递它。

既然我们已经讨论了如何安全地使用这两种流行的前端框架，现在让我们转向定义前端和后端之间的接口类型。

# 类型安全的 API

由 Nick Nance 贡献

不论你决定使用哪种前端和后端框架，你都需要一种安全的跨机器通信方式——从客户端到服务器、服务器到客户端、服务器到服务器，以及客户端到客户端。

在这个领域有几种竞争的工具和标准。但在我们探讨它们及其工作原理之前，让我们考虑一下如何构建我们自己的解决方案，以及它可能具有的优缺点（毕竟我们是工程师）。

我们要解决的问题是：尽管我们的客户端和服务器可能是 100%类型安全的——安全的堡垒——但它们在某个时候需要通过未经类型化的网络协议如 HTTP、TCP 或其他基于套接字的协议进行通信。我们如何使这种通信类型安全？

一个良好的起点可能是像我们在“类型安全协议”中开发的类型安全协议。它可能看起来像这样：

```
type Request =
  | {entity: 'user', data: User}
  | {entity: 'location', data: Location}

// client.ts
async function get<R extends Request>(entity: R['entity']): Promise<R['data']> {
  let res = await fetch(/api/${entity})
  let json = await res.json()
  if (!json) {
    throw ReferenceError('Empty response')
  }
  return json
}

// app.ts
async function startApp() {
  let user = await get('user')  // User
}
```

你可以构建对应的 `post` 和 `put` 函数来写回到你的 REST API，并为服务器支持的每种实体添加一个类型。在后端，你将为每种实体类型实现一组相应的处理程序，从数据库读取并返回客户端请求的实体。

但是，如果你的服务器不是用 TypeScript 编写的，或者你不能在客户端和服务器之间共享你的 `Request` 类型（导致它们随时间脱节），或者你不使用 REST（也许你使用的是 GraphQL）？或者如果你需要支持其他客户端，比如 iOS 上的 Swift 客户端或 Android 上的 Java 客户端？

这就是类型化、代码生成的 API 所能解决的问题。它们有许多不同的变体，每种都有库可以在许多语言中使用（包括 TypeScript）——例如：

+   [Swagger](https://github.com/swagger-api/swagger-codegen) 用于 RESTful APIs

+   [Apollo](https://www.npmjs.com/package/apollo) 和 [Relay](https://facebook.github.io/relay/) 用于 GraphQL

+   [gRPC](https://grpc.io/) 和 [Apache Thrift](https://thrift.apache.org/) 用于 RPC

这些工具依赖于一个共同的真实数据来源，用于服务器和客户端——Swagger 的数据模型、Apollo 的 GraphQL schema、gRPC 的 Protocol Buffers——然后将其编译成特定语言的绑定（在我们的案例中是 TypeScript）。

这种代码生成能防止你的客户端和服务器（或多个客户端）在互相不同步；因为每个平台共享一个公共模式，你不会遇到这样的情况，即你更新了 iOS 应用以支持一个字段，但忘记在你的拉取请求上按下合并以添加服务器对其的支持。

本书不涵盖每种框架的详细细节。为你的项目选择一个框架，然后转到其文档了解更多信息。

# 后端框架

当你构建一个与数据库交互的应用程序时，你可能从原始 SQL 或 API 调用开始，这些本质上是无类型的：

```
// PostgreSQL, using node-postgres
let client = new Client
let res = await client.query(
  'SELECT name FROM users where id = $1',
  [739311]
) // any

// MongoDB, using node-mongodb-native
db.collection('users')
  .find({id: 739311})
  .toArray((err, user) =>
    // user is any
  )
```

通过一点手工输入，你可以使这些 API 更安全，并摆脱大部分的`any`：

```
db.collection('users')
  .find({id: 739311})
  .toArray((err, user: User) =>
    // user is any
  )
```

然而，原始 SQL API 仍然相对较低级，并且很容易使用错误的类型，或者忘记类型而意外地得到 `any`。

这就是 *对象关系映射*（ORM）的用武之地。ORM 从你的数据库架构生成代码，为你提供高级 API 来表达查询、更新、删除等操作。在静态类型语言中，这些 API 是类型安全的，因此你不必担心正确地输入类型和手动绑定泛型类型参数。

在 TypeScript 中访问数据库时，考虑使用 ORM。在撰写本文时，Umed Khudoiberdiev 的优秀 [TypeORM](https://www.npmjs.com/package/typeorm) 是 TypeScript 最完整的 ORM，支持 MySQL、PostgreSQL、Microsoft SQL Server、Oracle 甚至 MongoDB。使用 TypeORM，获取用户名字的查询可能像这样：

```
let user = await UserRepository
  .findOne({id: 739311}) // User | undefined
```

注意高级 API，它既安全（防止诸如 SQL 注入攻击之类的问题），默认情况下也是类型安全的（我们知道 `findOne` 返回的类型，而不必手动注释）。在处理数据库时，始终使用 ORM——这更方便，可以避免你因为在凌晨四点更新 `saleAmount` 字段为 `orderAmount` 而被叫醒，而你的同事决定在你不在时为你运行数据库迁移，但是半夜你的拉取请求失败了，即使迁移成功，你的纽约销售团队也意识到所有客户的订单金额都为 `null` 美元（这发生在... 朋友身上）。

# 总结

在本章中，我们涵盖了很多内容：直接操作 DOM；使用 React 和 Angular；使用类似 Swagger、gRPC 和 GraphQL 的工具为你的 API 添加类型安全性；以及使用 TypeORM 安全地与数据库交互。

JavaScript 框架变化迅速，当你阅读本文时，这里描述的具体 API 和框架可能已经成为博物馆展品。利用你新获得的直觉来*解决类型安全框架解决的问题*，找出可以利用他人工作的地方，使你的代码更安全、更抽象、更模块化。从本章中带走的重要思想不是在 2019 年使用最佳框架是什么，而是什么样的问题可以通过框架更好地解决。

结合类型安全的 UI 代码、带类型的 API 层和类型安全的后端，你可以从应用程序中消除整类错误，结果是你可以晚上睡得更安稳。
