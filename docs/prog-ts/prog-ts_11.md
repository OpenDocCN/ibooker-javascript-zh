# 第十一章：与 JavaScript 的互操作

我们不生活在一个完美的世界。你的咖啡可能太烫，喝的时候会烫到嘴；你的父母可能会频繁地打电话给你留言；你家门口的坑洼无论你怎么打电话给市政府都还在那里；你的代码可能不完全涵盖静态类型。

大多数人都处于这种情况中：尽管偶尔你会有机会在 TypeScript 中启动一个全新的项目，但大多数情况下，它将作为一个安全的小岛，嵌入在一个更大、不太安全的代码库中。也许你有一个良好隔离的组件，想要尝试在上面使用 TypeScript，即使你的公司在其他地方都使用常规的 ES6 JavaScript，或者你因为重构了某些代码但忘记更新调用点而在凌晨 6 点被叫醒（现在已经 7 点了，你在同事们醒来之前正在快速合并 TypeScript 编译器到你的代码库中）。无论哪种情况，你可能会从一个 TypeScript 的孤岛开始。

到目前为止，在本书中我教你如何正确编写 TypeScript。本章是关于在真实代码库中以实用的方式编写 TypeScript，这些代码库正在迁移远离无类型语言，使用第三方的 JavaScript 库，并且有时为了快速修复生产问题而牺牲类型安全性。本章专注于与 JavaScript 的交互。我们将探讨：

+   使用类型声明

+   逐步从 JavaScript 迁移到 TypeScript

+   使用第三方的 JavaScript 和 TypeScript

# 类型声明

*类型声明* 是一个扩展名为 *.d.ts* 的文件。除了 JSDoc 注释（参见“第 2 步 b：添加 JSDoc 注释（可选）”）外，它是将 TypeScript 类型附加到本来无类型的 JavaScript 代码的一种方式。

类型声明的语法与常规的 TypeScript 类似，但也有一些区别：

+   类型声明只能包含类型，不能包含值。这意味着不能包括函数、类、对象或变量的实现，也不能为参数提供默认值。

+   虽然类型声明不能定义值，但它们可以声明在你的 JavaScript 中的某处存在一个值。我们使用特殊的`declare`关键字来实现这一点。

+   类型声明只声明对消费者可见的内容的类型。我们不包括未导出的内容，或者函数体内部的局部变量的类型。

让我们来看一个示例，看看 TypeScript (*.ts*) 代码片段及其相应的类型声明 (*.d.ts*)。这个示例是来自流行的 RxJS 库的一段相当复杂的代码；可以忽略它正在做什么的细节，而是关注它使用了哪些语言特性（导入、类、接口、类字段、函数重载等）：

```
import {Subscriber} from './Subscriber'
import {Subscription} from './Subscription'
import {PartialObserver, Subscribable, TeardownLogic} from './types'

export class Observable<T> implements Subscribable<T> {
  public _isScalar: boolean = false
  constructor(
    subscribe?: (
      this: Observable<T>,
      subscriber: Subscriber<T>
    ) => TeardownLogic
  ) {
    if (subscribe) {
      this._subscribe = subscribe
    }
  }
  static create<T>(subscribe?: (subscriber: Subscriber<T>) => TeardownLogic) {
    return new Observable<T>(subscribe)
  }
  subscribe(observer?: PartialObserver<T>): Subscription
  subscribe(
    next?: (value: T) => void,
    error?: (error: any) => void,
    complete?: () => void
  ): Subscription
  subscribe(
    observerOrNext?: PartialObserver<T> | ((value: T) => void),
    error?: (error: any) => void,
    complete?: () => void
  ): Subscription {
    // ...
  }
}
```

使用带有`declarations`标志的 TSC 运行此代码（`tsc -d Observable.ts`）会生成以下*Observable.d.ts*类型声明：

```
import {Subscriber} from './Subscriber'
import {Subscription} from './Subscription'
import {PartialObserver, Subscribable, TeardownLogic} from './types'

export declare class Observable<T> implements Subscribable<T> { ![1](img/1.png)
  _isScalar: boolean
  constructor(
    subscribe?: (
      this: Observable<T>,
      subscriber: Subscriber<T>
    ) => TeardownLogic
  );
  static create<T>(
    subscribe?: (subscriber: Subscriber<T>) => TeardownLogic
  ): Observable<T>
  subscribe(observer?: PartialObserver<T>): Subscription
  subscribe(
    next?: (value: T) => void,
    error?: (error: any) => void,
    complete?: () => void
  ): Subscription ![2](img/2.png)
}
```

![1](img/#co_interoperating_with_javascript_CO1-1)

注意在`class`之前的`declare`关键字。我们实际上不能在类型声明中定义类，但可以声明在*.d.ts*文件的对应 JavaScript 文件中定义了一个类。可以将`declare`想象成一种确认：“我保证我的 JavaScript 导出了这种类型的类。”

![2](img/#co_interoperating_with_javascript_CO1-2)

因为类型声明不包含实现，所以我们只保留了`subscribe`的两个重载，而没有保留其实现的签名。

注意*Observable.d.ts*只是*Observable.ts*的一个版本，去掉了实现部分。换句话说，它只包含了*Observable.ts*中的类型信息。

这种类型声明对于 RxJS 库中使用*Observable.ts*的其他文件并不有用，因为它们可以直接访问*Observable.ts*源 TypeScript 文件。但是，如果您在 TypeScript 应用程序中使用 RxJS，则这很有用。

想想看：如果 RxJS 的作者想要在 NPM 上为他们的 TypeScript 用户提供打包的类型信息（RxJS 可用于 TypeScript 和 JavaScript 应用程序），他们有两个选择：打包源 TypeScript 文件（供 TypeScript 用户使用）和已编译的 JavaScript 文件（供 JavaScript 用户使用），或者只提供已编译的 JavaScript 文件以及 TypeScript 用户的类型声明。后者可以减少文件大小，并明确正确的导入使用方式。它还有助于保持应用程序的编译时间快速，因为您的 TSC 实例不需要在每次编译应用程序时重新编译 RxJS（实际上，这就是我们在“项目引用”中介绍的优化策略的原因！）。

类型声明文件有几个用途：

1.  当其他人在他们的 TypeScript 应用程序中使用您编译的 TypeScript 时，他们的 TSC 实例将查找与您生成的 JavaScript 文件对应的*.d.ts*文件。这告诉 TypeScript 您项目的类型信息。

1.  带有 TypeScript 支持的代码编辑器（如 VSCode）将读取这些*.d.ts*文件，以在用户键入时为他们提供有用的类型提示，即使他们不使用 TypeScript。

1.  它们通过避免不必要地重新编译您的 TypeScript 代码显著加快了编译时间。

类型声明是告诉 TypeScript：“JavaScript 中有这样一个定义，我将描述给你。” 当我们谈论类型声明时，为了与包含值的常规声明区分开来，我们通常将它们称为 *环境* 声明；例如，*环境变量声明* 使用 `declare` 关键字声明变量在 JavaScript 中某处定义，而常规的非环境变量声明则是使用 `let` 或 `const` 声明一个变量，而无需使用 `declare` 关键字。

你可以为几件事情使用类型声明：

+   要告诉 TypeScript 有关某个在 JavaScript 中某处定义的全局变量。例如，如果你在浏览器环境中填充了 `Promise` 全局变量或者定义了 `process.env`，你可以使用 *环境变量声明* 来提醒 TypeScript。

+   定义一个在整个项目中都全局可用的类型，因此在使用时不需要先导入它（我们称之为环境类型声明）。

+   告诉 TypeScript 关于你使用 NPM 安装的第三方模块的信息（*环境模块声明*）。

无论你使用类型声明做什么，它都必须存在于脚本模式的 *.ts* 或 *.d.ts* 文件中（回想一下我们早前讨论的模块模式与脚本模式 “模块模式与脚本模式”）。按照惯例，如果文件有对应的 *.js* 文件，我们会给文件一个 *.d.ts* 扩展名；否则，我们使用 *.ts* 扩展名。文件的名称并不重要——例如，我喜欢将所有类型声明放在一个顶级文件 *types.ts* 中，直到文件变得难以管理——一个类型声明文件可以包含任意多个类型声明。

最后，虽然类型声明文件中的顶级值需要使用 `declare` 关键字（如 `declare let`、`declare function`、`declare class` 等），但顶级类型和接口则不需要。

在明确了这些基本规则后，让我们简要看一些每种类型声明的例子。

## 环境变量声明

环境变量声明是一种告诉 TypeScript 在项目中任何 *.ts* 或 *.d.ts* 文件中可以使用的全局变量的方式，而无需首先显式导入它。

假设你在浏览器中运行 NodeJS 程序，并且在某一时刻程序检查 `process.env.NODE_ENV`（可能是 `"development"` 或 `"production"`）。当你运行程序时，你会得到一个难看的运行时错误：

```
Uncaught ReferenceError: process is not defined.
```

你在 Stack Overflow 上搜寻了一下，意识到让程序运行起来的最快方法是自己填充 `process.env.NODE_ENV` 并将其硬编码。所以你创建了一个新文件 *polyfills.ts*，并定义了一个全局的 `process.env`：

```
process = {
  env: {
    NODE_ENV: 'production'
  }
}
```

当然，TypeScript 会挺身而出，用红色波浪线提醒你，试图通过增强全局 `window` 来阻止你显然正在犯的错误：

```
Error TS2304: Cannot find name 'process'.
```

但在这种情况下，TypeScript 显得过于保守了。实际上，你确实希望增强 `window`，并且希望安全地进行操作。

那么你该怎么办呢？你在 Vim 中打开*polyfills.ts*（你知道这将发生什么）并输入：

```
`declare` `let` `process``:` `{`
  `env``:` `{`
    `NODE_ENV``:` `'development'` `|` `'production'`
  `}`
`}`

process = {
  env: {
    NODE_ENV: 'production'
  }
}

```

你在告诉 TypeScript 存在一个全局对象`process`，它有一个名为`env`的属性，其中包含一个名为`NODE_ENV`的属性。一旦你告诉 TypeScript 这一点，红色波浪线消失了，你可以安全地定义你的`process`全局对象。

# TSC 设置：lib

TypeScript 自带一组类型声明，用于描述包括内置 JavaScript 类型（如`Array`和`Promise`）和内置类型的方法（如`''.toUpperCase`）在内的 JavaScript 标准库，还包括全局对象如`window`和`document`（在浏览器环境中），以及`onmessage`（在 Web Worker 环境中）。

你可以通过你的*tsconfig.json*文件的`lib`字段引入 TypeScript 的内置类型声明。查看`lib`以深入了解如何调整项目的`lib`设置。

## 环境类型声明

环境类型声明遵循环境变量声明相同的规则：声明必须位于脚本模式的*.ts*或*.d.ts*文件中，并且它将在项目中的其他文件中全局可用，无需显式导入。例如，让我们声明一个全局实用程序类型`ToArray<T>`，如果`T`尚不是数组，则将其提升为数组。我们可以在项目的任何脚本模式文件中定义这种类型——例如，让我们在顶层的*types.ts*文件中定义它：

```
type ToArray<T> = T extends unknown[] ? T : T[]
```

现在，我们可以在任何项目文件中使用这种类型，而无需显式导入：

```
function toArray<T>(a: T): ToArray<T> {
  // ...
}
```

考虑使用环境类型声明来建模应用程序中始终使用的数据类型。例如，你可以用它们来使我们在“模拟名义类型”中开发的`UserID`类型全局可用：

```
type UserID = string & {readonly brand: unique symbol}
```

现在，你可以在应用程序的任何地方使用`UserID`，而无需先显式导入它。

## 环境模块声明

当你使用 JavaScript 模块并希望快速为其声明一些类型，以便安全地使用它——而无需首先将类型声明贡献回 JavaScript 模块的 GitHub 存储库或 DefinitelyTyped——环境模块声明就是你要使用的工具。

环境模块声明是一个常规的类型声明，被包围在特殊的`declare module`语法中：

```
declare module 'module-name' {
  export type MyType = number
  export type MyDefaultType = {a: string}
  export let myExport: MyType
  let myDefaultExport: MyDefaultType
  export default myDefaultExport
}
```

在这个例子中，模块名（`'module-name'`）对应于一个确切的`import`路径。当你导入这个路径时，你的环境模块声明告诉 TypeScript 可用的内容：

```
import ModuleName from 'module-name'
ModuleName.a  // string
```

如果你有一个嵌套的模块，请确保在其声明中包含整个`import`路径：

```
declare module '@most/core' {
  // Type declaration
}
```

如果你只是想快速告诉 TypeScript “我正在导入这个模块——稍后我会为它添加类型，现在就假设它是`any`”，保留头部但省略实际声明：

```
// Declare a module that can be imported, where each of its imports are any
declare module 'unsafe-module-name'
```

现在，如果你使用这个模块，它就不太安全了：

```
import {x} from 'unsafe-module-name'
x  // any
```

模块声明支持通配符导入，因此你可以为任何匹配给定模式的`import`路径提供类型。使用通配符（`*`）来匹配一个`import`路径：^(1)

```
// Type JSON files imported with Webpack's json-loader
declare module 'json!*' {
  let value: object
  export default value
}

// Type CSS files imported with Webpack's style-loader
declare module '*.css' {
  let css: CSSRuleList
  export default css
}
```

现在，你可以加载 JSON 和 CSS 文件：

```
import a from 'json!myFile'
a  // object

import b from './widget.css'
b  // CSSRuleList
```

###### 注意

为了使最后两个示例工作，你需要配置你的构建系统来加载*.json*和*.css*文件。你可以告诉 TypeScript 这些路径模式是安全导入的，但 TypeScript 本身不能构建它们。

直接跳转到“没有在 DefinitelyTyped 上具有类型声明的 JavaScript”，了解如何使用环境模块声明为无类型第三方 JavaScript 声明类型的示例。

# 逐步从 JavaScript 迁移到 TypeScript

TypeScript 的设计考虑了与 JavaScript 的互操作性，并非事后补救。因此，尽管不是毫不费力，迁移到 TypeScript 是一种很好的体验，它允许你逐个文件地将代码库转换过来，随着迁移逐步选择更严格的安全级别，向老板和同事展示静态类型化代码可以有多么有影响力，一次提交一个。

从高层次来看，你希望的目标是：你的代码库完全用 TypeScript 编写，具有严格的类型覆盖，而你依赖的第三方 JavaScript 库应该具有高质量且严格的类型。任何可以在编译时捕获的错误都会被捕获，TypeScript 丰富的自动补全功能可以减少每行代码编写所需的时间。你可能需要采取一些小步骤来达到这个目标：

+   将 TSC 添加到你的项目中。

+   开始对你现有的 JavaScript 代码进行类型检查。

+   将你的 JavaScript 代码逐步迁移到 TypeScript。

+   为你的依赖安装类型声明，可以是为没有类型的依赖存根出类型，也可以是为无类型依赖编写类型声明，并将其贡献回 DefinitelyTyped。^(2)

+   为你的代码库打开`strict`模式。

这个过程可能需要一些时间，但你会立即看到安全性和生产力的提升，随着你的进展，还会发现更多的收益。让我们一步步来看看这些步骤。

## 步骤 1：添加 TSC

在处理结合了 TypeScript 和 JavaScript 的代码库时，首先让 TSC 编译 JavaScript 文件与你的 TypeScript 并行工作。在你的*tsconfig.json*中：

```
{
  "compilerOptions": {
  "allowJs": true
}
```

通过这一个改变，你现在可以使用 TSC 来编译你的 JavaScript。只需将 TSC 添加到你的构建流程中，并且通过 TSC 运行每个现有的 JavaScript 文件，^(3) 或者继续通过现有的构建流程运行传统的 JavaScript 文件，并通过 TSC 运行新的 TypeScript 文件。

设置`allowJs`为`true`后，TypeScript 不会对你现有的 JavaScript 代码进行类型检查，但它会根据你在*tsconfig.json*中设置的目标（ES3、ES5 或其他）转译它，使用你要求的模块系统（在*tsconfig.json*的`module`字段中）。第一步完成。提交它，并给自己一个鼓励——你的代码库现在使用 TypeScript 了！

## 步骤 2a：启用 JavaScript 的类型检查（可选）

现在 TSC 正在处理您的 JavaScript，为什么不也对其进行类型检查呢？虽然您的 JavaScript 中可能没有显式的类型注释，请记住 TypeScript 在推断类型方面有多么出色；它可以像在常规 TypeScript 代码中那样推断出您 JavaScript 中的类型。在您的*tsconfig.json*中启用此功能：

```
{
  "compilerOptions": {
  "allowJs": true,
  "checkJs": true
}

```

现在，每当 TypeScript 编译 JavaScript 文件时，它都会尽力推断类型并进行类型检查，就像对常规 TypeScript 代码一样。

如果您的代码库很大，并且打开`checkJs`会同时报告太多类型错误，请关闭它，而是通过在文件顶部添加`// @ts-check`指令（普通注释）来逐个文件启用 JavaScript 文件的检查。或者，如果一些大文件引发了大部分错误，而您暂时不想修复它们，请保持`checkJs`开启，并仅为这些文件添加`// @ts-nocheck`指令。

###### 注意

因为 TypeScript 无法推断一切（例如函数参数类型），它将在您的 JavaScript 代码中推断大量类型为`any`。如果您在*tsconfig.json*中启用了`strict`模式（应该！），您可能希望在迁移时暂时允许隐式`any`。在您的*tsconfig.json*中添加：

```
{
  "compilerOptions": {
  "allowJs": true,
  "checkJs": true,
  "noImplicitAny": false
}

```

不要忘记在迁移了大量代码到 TypeScript 后再次打开`noImplicitAny`！它可能会显示出您错过的一堆真实错误（当然，除非您是 Xenithar，巴思莫尔达的 JavaScript 巫师的弟子，能够仅凭一锅满满的艾蒿进行心眼中的类型检查）。

当 TypeScript 对 JavaScript 代码进行处理时，它会使用比对 TypeScript 代码更宽松的推断算法。具体而言：

+   所有函数参数都是可选的。

+   函数和类的属性类型是从使用中推断出来的（而不是必须一开始就声明的）：

```
 class A {
   x = 0 // number | string | string[], inferred from usage
   method() {
     this.x = 'foo'
   }
   otherMethod() {
     this.x = ['array', 'of', 'strings']
   }
 }
```

+   在声明对象、类或函数后，您可以为其分配额外的属性。在幕后，TypeScript 通过为每个类和函数声明生成相应的命名空间，并自动向每个对象字面量添加索引签名来实现这一点。

## 步骤 2b：添加 JSDoc 注释（可选）

也许您很忙，只需为添加到旧 JavaScript 文件中的新函数添加单个类型注释。在有机会将该文件转换为 TypeScript 之前，您可以使用 JSDoc 注释为您的新函数类型。

您可能以前见过 JSDoc；它是那些带有`@`注释的奇怪看起来的 JavaScript 和 TypeScript 代码上方的注释，如`@param`、`@returns`等。 TypeScript 理解 JSDoc，并将其作为输入传递给其类型检查器，方式与其在 TypeScript 代码中使用显式类型注释相同。

假设您有一个 3000 行的实用程序文件（是的，我知道您的“朋友”写的）。您向其中添加一个新的实用程序函数：

```
export function toPascalCase(word) {
  return word.replace(
    /\w+/g,
    ([a, ...b]) => a.toUpperCase() + b.join('').toLowerCase()
  )
}
```

不必将*utils.js*全面转换为 TypeScript，这样可能会捕捉到一堆你接下来需要修复的错误；你只需注释你的`toPascalCase`函数，就在未经类型定义的 JavaScript 海洋中创造出一个小小的安全岛：

```
*`/**  * @param word {string} An input string to convert  * @returns {string} The string in PascalCase  */`*
export function toPascalCase(word) {
  return word.replace(
    /\w+/g,
    ([a, ...b]) => a.toUpperCase() + b.join('').toLowerCase()
  )
}

```

没有 JSDoc 注释的话，TypeScript 会将`toPascalCase`的类型推断为`(word: any) => string`。现在，当 TypeScript 编译你的代码时，它知道`toPascalCase`的类型是`(word: string) => string`。而且你还得到了一些不错的文档！

前往[TypeScript Wiki](http://bit.ly/2YCTWBf)了解更多支持的 JSDoc 注释。

## 步骤 3：将文件重命名为.ts

一旦你将 TSC 添加到构建流程中，并可选择在可能的情况下开始对 JavaScript 进行类型检查和注释，现在是时候开始切换到 TypeScript 了。

逐个文件，将文件扩展名从*.js*（或*.coffee*、*.es6*等）更改为*.ts*。一旦你在代码编辑器中重命名文件，你会看到你的朋友红色的波浪线出现（TypeError，而不是儿童电视节目），揭示了类型错误、遗漏的情况、忘记的`null`检查和拼写错误的变量名。修复这些错误有两种策略：

1.  做正确的事情。花时间正确地为形状、字段和函数类型化，这样你可以捕捉到所有使用它们的文件中的错误。如果启用了`checkJs`，在*tsconfig.json*中打开`noImplicitAny`来揭示`any`并为其添加类型，然后关闭它以减少对余下 JavaScript 文件类型检查输出的干扰。

1.  快速行动。将你的 JavaScript 文件批量重命名为*.ts*扩展名，并保持*tsconfig.json*设置宽松（即将`strict`设置为`false`），以在重命名后尽可能少地抛出类型错误。将复杂类型定义为`any`以满足类型检查器的要求。修复剩下的类型错误并提交。完成后，逐个打开`strict`模式标志（如`noImplicitAny`、`noImplicitThis`、`strictNullChecks`等），并修复弹出的错误。（参见附录 F 获取完整的标志列表。）

###### 提示

如果你选择快速而不拘一格的方式，一个有用的技巧是将环境类型声明`TODO`定义为`any`类型的别名，并使用它而不是`any`，这样你可以更轻松地找到和跟踪缺少的类型。你也可以将其命名得更加具体，以便在项目范围的代码搜索中更容易找到：

```
// globals.ts
type TODO_FROM_JS_TO_TS_MIGRATION = any

// MyMigratedUtil.ts
export function mergeWidgets(
  widget1: TODO_FROM_JS_TO_TS_MIGRATION,
  widget2: TODO_FROM_JS_TO_TS_MIGRATION
): number {
  // ...
}
```

这两种方法都是可以接受的，你可以自行选择。因为 TypeScript 是一种逐渐类型化的语言，它从根本上构建起与未经类型定义的 JavaScript 代码尽可能安全地进行互操作。无论你是在严格类型化的 TypeScript 中与未经类型定义的 JavaScript 互操作，还是与松散类型化的 TypeScript 互操作，TypeScript 都会尽最大努力确保你的安全，并保证在你精心构建的严格类型化岛上，一切都尽可能安全。

## 步骤 4：使其严格

一旦你将大量 JavaScript 迁移到 TypeScript，你会希望通过逐步选择 TSC 更严格的标志来尽可能地使代码更安全（详见 附录 F 获取完整的标志列表）。

最后，您可以禁用 TSC 的 JavaScript 互操作标志，强制所有代码都以严格类型化的 TypeScript 编写：

```
{
  "compilerOptions": {
  "allowJs": false,
  "checkJs": false
}

```

这将显示与类型相关的最终错误。修复这些错误，您将得到一个干净且安全的代码库，即使是最顽固的 OCaml 工程师也会为您竖起大拇指（如果您礼貌地问的话）。

遵循这些步骤将有助于您在控制的 JavaScript 中添加类型，但对于您无法控制的 JavaScript（例如从 NPM 安装的代码），该如何处理呢？为了达到这个目标，我们首先需要稍微改变方向…

# JavaScript 的类型查找

当您从 TypeScript 文件导入 JavaScript 文件时，TypeScript 遵循以下类似的算法来查找您的 JavaScript 代码的类型声明（请记住，在 TypeScript 中我们谈论“文件”和“模块”时是可以互换的）：^(4)

1.  查找与您的 *.js* 文件同名的兄弟 *.d.ts* 文件。如果存在，则将其用作 *.js* 文件的类型声明。

    例如，假设您的文件夹结构如下所示：

    ```
    my-app/
    ├──src/
    │ ├──index.ts
    │ └──legacy/
    │   ├──old-file.js
    │   └──old-file.d.ts
    ```

    然后从 *index.ts* 中导入 *old-file*：

    ```
    // index.ts
    import './legacy/old-file'
    ```

    TypeScript 将使用 *src/legacy/old-file.d.ts* 作为 *./legacy/old-file* 的类型声明源。

1.  否则，如果 `allowJs` 和 `checkJs` 为 true，则根据 *.js* 文件中的任何 JSDoc 注释推断出类型。

1.  否则，将整个模块视为 `any`。

当导入第三方 JavaScript 模块——即您安装到 *node modules* 的 NPM 包时，TypeScript 使用稍微不同的算法：

1.  查找模块的本地类型声明。如果存在，则使用它。

    例如，假设您的应用程序文件夹结构如下所示：

    ```
    my-app/
    ├──node_modules/
    │ └──foo/
    ├──src/
    │ ├──index.ts
    │ └──types.d.ts
    ```

    并且 *types.d.ts* 看起来像这样：

    ```
    // types.d.ts
    declare module 'foo' {
      let bar: {}
      export default bar
    }
    ```

    如果接着导入 `foo`，TypeScript 将使用 *types.d.ts* 中的环境模块声明作为 `foo` 的类型源：

    ```
    // index.ts
    import bar from 'foo'
    ```

1.  否则，请查看模块的 *package.json*。如果它定义了一个名为 `types` 或 `typings` 的字段，请使用该字段指向的 *.d.ts* 文件作为模块类型声明的源。

1.  否则，逐级向外遍历目录，并查找具有模块类型声明的 *node modules/@types* 目录。

    例如，假设你安装了 React：

    ```
    npm install react --save
    npm install @types/react --save-dev
    ```

    ```
    my-app/
    ├──node_modules/
    │ ├──@types/
    │ │ └──react/
    │ └──react/
    ├──src/
    │ └──index.ts
    ```

    当您导入 React 时，TypeScript 将找到 *@types/react* 文件夹，并将其用作 React 的类型声明源：

    ```
    // index.ts
    import * as React from 'react'
    ```

1.  否则，按照本地类型查找算法的步骤 1–3 进行。

虽然步骤很多，但一旦你掌握了，它实际上非常直观。

# TSC 设置：types 和 typeRoots

默认情况下，TypeScript 在项目文件夹中的*node modules/@types*及其包含的文件夹（*../node modules/@types*等）中查找第三方类型声明。大多数情况下，你应该保留这种行为。

要覆盖全局类型声明的默认行为，请在你的*tsconfig.json*中配置`typeRoots`，并用一个包含文件夹路径的数组来查找类型声明。例如，你可以告诉 TypeScript 在*typings*文件夹和*node modules/@types*中查找类型声明：

```
{
  "compilerOptions": {
    "typeRoots" : ["./typings", "./node modules/@types"]
  }
}
```

要进行更精细的控制，可以在你的*tsconfig.json*中使用`types`选项来指定希望 TypeScript 查找类型的包。例如，以下配置忽略除了 React 外的所有第三方类型声明：

```
{
  "compilerOptions": {
    "types" : ["react"]
  }
}
```

# 使用第三方 JavaScript

###### 注意

我假设你正在使用像 NPM 或 Yarn 这样的包管理器来安装第三方 JavaScript。如果你是那些喜欢手动复制粘贴代码的人——你真是可耻。

当你通过`npm install`将第三方 JavaScript 代码安装到你的项目中时，可能会有三种情况：

1.  你安装的代码自带类型声明。

1.  你安装的代码没有类型声明，但在 DefinitelyTyped 上可以找到声明。

1.  你安装的代码没有类型声明，并且在 DefinitelyTyped 上也找不到声明。

让我们深入了解每种情况。

## 自带类型声明的 JavaScript

如果一个包默认带有类型声明，当你使用`{"noImplicitAny": true}`导入它时，TypeScript 不会给你抛出红色波浪线。

如果你安装的代码是从 TypeScript 编译而来的，或者其作者在其 NPM 包中包含了类型声明，那么你很幸运。只需安装该代码并开始使用它，就可以获得完整的类型支持。

一些带有内置类型声明的 NPM 包示例包括：

```
npm install rxjs
npm install ava
npm install @angular/cli
```

###### 警告

除非你安装的代码实际上是从 TypeScript 编译而来的，否则始终存在类型声明与描述该代码的实际代码不匹配的风险。当类型声明与源代码打包在一起时，这种情况发生的风险非常低（尤其是对于流行的包而言），但仍需注意。

## 在 DefinitelyTyped 上有类型声明的 JavaScript

即使你导入的第三方代码没有类型声明，也可能在[DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped)上找到它的声明，这是 TypeScript 的社区维护的用于开源项目环境模块声明的集中存储库。

要检查你安装的包是否在 DefinitelyTyped 上有可用的类型声明，可以在 [TypeSearch](https://microsoft.github.io/TypeSearch/) 上搜索，或者尝试安装声明。所有 DefinitelyTyped 类型声明都发布到 NPM 的 `@types` 范围下，因此你可以直接从该范围进行 `npm install`：

```
npm install lodash --save            # Install Lodash
npm install @types/lodash --save-dev # Install type declarations for Lodash
```

大多数情况下，你会想要使用 `npm install` 的 `--save-dev` 标志将已安装的类型声明添加到 *package.json* 的 `devDependencies` 字段中。

###### 注意

由于 DefinitelyTyped 上的类型声明是由社区维护的，它们可能存在不完整、不准确或过时的风险。虽然大多数流行的包都有良好维护的类型声明，但如果发现你使用的声明可以改进，花时间改进它们并将其贡献回 DefinitelyTyped，以便其他 TypeScript 用户可以利用你的辛勤工作。

## 在 DefinitelyTyped 上没有类型声明的 JavaScript

这是三种情况中最不常见的情况。在这里，你有几个选项，从最便宜且最不安全的到最耗时且最安全的：

1.  *通过添加 `// @ts-ignore` 指令在未经类型化的导入之上*，将特定导入列入白名单。TypeScript 将允许你使用未经类型化的模块，但该模块及其所有内容将被类型化为 `any`：

    ```
    // @ts-ignore
    import Unsafe from 'untyped-module'

    Unsafe  // any
    ```

1.  *通过创建一个空的类型声明文件并为该模块创建存根*，将该模块的所有用法列入白名单。例如，如果你安装了很少使用的包 `nearby-ferret-alerter`，你可以创建一个新的类型声明文件（例如 *types.d.ts*），并向其中添加环境类型声明：

    ```
    // types.d.ts
    declare module 'nearby-ferret-alerter'
    ```

    这告诉 TypeScript 存在一个模块，你可以导入（`import alert from 'nearby-ferret-alerter'`），但它并不告诉 TypeScript 有关该模块中包含的类型。这种方法略好于第一种，因为现在有一个中心的 *types.d.ts* 文件列举了应用程序中所有未经类型化的模块，但同样不安全，因为 `nearby-ferret-alerter` 及其所有导出仍将被类型化为 `any`。

1.  *创建一个环境模块声明*。就像在前一种方法中一样，创建一个名为 *types.d.ts* 的文件，并添加一个空的声明（`declare module 'nearby-ferret-alerter'`）。现在，填写类型声明。例如，结果可能如下所示：

    ```
    // types.d.ts
    declare module 'nearby-ferret-alerter' {
      export default function alert(loudness: 'soft' | 'loud'): Promise<void>
      export function getFerretCount(): Promise<number>
    }
    ```

    现在当你 `import alert from 'nearby-ferret-alerter'` 时，TypeScript 将准确知道 `alert` 的类型。它不再是 `any`，而是 `(loudness: 'quiet' | 'loud') => Promise<void>`。

1.  *创建类型声明并贡献给 NPM*。如果您已经为您的模块创建了本地类型声明，并且达到了第三个选项，考虑将其贡献回 NPM，以便下一个需要为 `nearby-ferret-alerter` 包获取类型声明的人也可以使用它。要实现这一点，您可以向 `nearby-ferret-alerter` 的 Git 存储库提交拉取请求，直接贡献类型声明，或者如果该存储库的维护者不想负责维护 TypeScript 类型声明，可以将您的声明贡献给 DefinitelyTyped。

为第三方 JavaScript 编写类型声明是直截了当的，但其具体操作方式取决于您正在为其编写类型的模块类型。在为不同类型的 JavaScript 模块（从 NodeJS 模块到 jQuery 增强，再到 Lodash 混入和 React、Angular 组件）编写类型时，会出现一些常见的模式。转到 附录 D 查看针对第三方 JavaScript 模块的各种类型编写方法。

###### 注

自动为未类型化的 JavaScript 生成类型声明是一个活跃的研究领域。查看 [`dts-gen`](https://www.npmjs.com/package/dts-gen) 以了解一种自动生成任何第三方 JavaScript 模块类型声明框架的方法。

# 总结

有几种方法可以从 TypeScript 中使用 JavaScript。表 11-1 总结了这些选项。

表 11-1\. 使用 JavaScript 从 TypeScript 的方法总结

| 方法 | tsconfig.json 标志 | 类型安全性 |
| --- | --- | --- |
| 导入未类型化的 JavaScript | `{"allowJs": true}` | 差 |
| 导入和检查 JavaScript | `{"allowJs": true, "checkJs": true}` | 良好 |
| 导入和检查带有 JSDoc 注释的 JavaScript | `{"allowJs": true, "checkJs": true, "strict": true}` | 优秀 |
| 导入具有类型声明的 JavaScript | `{"allowJs": false, "strict": true}` | 优秀 |
| 导入 TypeScript | `{"allowJs": false, "strict": true}` | 优秀 |

在本章中，我们涵盖了一些使用 JavaScript 和 TypeScript 结合的各个方面，从不同类型声明的种类及其使用方式，到逐步将现有 JavaScript 项目逐步迁移到 TypeScript，再到安全（以及不安全地）使用第三方 JavaScript。与 JavaScript 的互操作可能是 TypeScript 中最棘手的方面之一；通过手头的所有工具，您现在已经具备在自己的项目中进行操作的能力了。

^(1) 使用 `*` 进行通配符匹配遵循与常规 [glob 模式匹配](https://en.wikipedia.org/wiki/Glob_(programming)) 相同的规则。

^(2) DefinitelyTyped 是 JavaScript 类型声明的开源存储库。继续阅读以了解更多信息。

^(3) 对于非常大的项目，逐个文件通过 TSC 可能会很慢。有关如何提升大型项目性能的方法，请参见“项目引用”。

^(4) 严格来说，这对于模块模式而言是正确的，但对于脚本模式则不是。详细了解请参阅“模块模式与脚本模式”。
