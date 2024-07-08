# 附录 D. 为第三方 JavaScript 模块编写声明文件的配方

本附录涵盖了一些关键的构建块和模式，这些在输入第三方模块时反复出现。要深入讨论输入第三方代码，请转到“在 DefinitelyTyped 上没有类型声明的 JavaScript”。

因为模块声明文件必须存在于 *.d.ts* 文件中，不能包含值，所以在声明模块类型时，需要使用`declare`关键字来确认给定类型的值确实被模块导出。表 D-1 提供了常规声明及其类型声明等效的简短摘要。

表 D-1. TypeScript 及其仅类型等效项

| *.ts* | *.d.ts* |
| --- | --- |
| `var a = 1` | `declare var a: number` |
| `let a = 1` | `declare let a: number` |
| `const a = 1` | `declare const a: 1` |
| `function a(b) { return b.toFixed() }` | `declare function a(b: number): string` |
| `class A { b() { return 3 } }` | `declare class A { b(): number }` |
| `namespace A {}` | `declare namespace A {}` |
| `type A = number` | `type A = number` |
| `interface A { b?: string }` | `interface A { b?: string }` |

# 导出类型

模块使用全局、ES2015 或 CommonJS 导出会影响你编写声明文件的方式。

## 全局

如果你的模块只是将值分配给全局命名空间，并且实际上并没有导出任何东西，你可以创建一个脚本模式文件（参见“模块模式与脚本模式”），并使用`declare`前缀你的变量、函数和类声明（其他类型的声明如`enum`、`type`等保持不变）：

```
// Global variable
declare let someGlobal: GlobalType

// Global class
declare class GlobalClass {}

// Global function
declare function globalFunction(): string

// Global enum
enum GlobalEnum {A, B, C}

// Global namespace
namespace GlobalNamespace {}

// Global type alias
type GlobalType = number

// Global interface
interface GlobalInterface {}
```

这些声明都将全局可用，无需显式导入到项目中的每个文件。在此，你可以在项目中的任何文件中使用`someGlobal`，而无需首先导入它，但在运行时，`someGlobal`需要分配给全局命名空间（浏览器中的`window`或 NodeJS 中的`global`）。

要保持文件在脚本模式下，请小心避免在声明文件中使用`import`和`export`。

## ES2015 导出

如果你的模块使用 ES2015 导出，即`export`关键字，只需用`export`替换`declare`（它确认全局变量已定义）：

```
// Default export
declare let defaultExport: SomeType
export default defaultExport

// Named export
export class SomeExport {
  a: SomeOtherType
}

// Class export
export class ExportedClass {}

// Function export
export function exportedFunction(): string

// Enum export
enum ExportedEnum {A, B, C}

// Namespace export
export namespace SomeNamespace {
  let someNamespacedExport: number
}

// Type export
export type SomeType = {
  a: number
}

// Interface export
export interface SomeOtherType {
  b: string
}
```

## CommonJS 导出

在 ES2015 之前，CommonJS 是事实上的模块标准，并且仍然是 NodeJS 的标准。它也使用`export`关键字，但语法有些不同：

```
declare let defaultExport: SomeType
export = defaultExport
```

注意我们是如何将我们的导出分配给`export`，而不是像我们对 ES2015 导出所做的那样使用`export`作为修饰符。

第三方 CommonJS 模块的类型声明可以只包含一个导出。要导出多个内容，我们利用声明合并（参见附录 C）。

例如，要为多个导出定义类型而没有默认导出，我们导出一个单一的`namespace`：

```
declare namespace MyNamedExports {
  export let someExport: SomeType
  export type SomeType = number
  export class OtherExport {
    otherType: string
  }
}
export = MyNamedExports
```

如果一个 CommonJS 模块同时具有默认导出和命名导出呢？我们可以利用声明合并的功能：

```
declare namespace MyExports {
  export let someExport: SomeType
  export type SomeType = number
}
declare function MyExports(a: number): string
export = MyExports
```

## UMD 导出

类型化 UMD 模块几乎与类型化 ES2015 模块相同。唯一的区别在于，如果你想让你的模块在脚本模式文件中全局可用（参见“模块模式与脚本模式”），你需要使用特殊的`export as namespace`语法。例如：

```
// Default export
declare let defaultExport: SomeType
export default defaultExport

// Named export
export class SomeExport {
  a: SomeType
}

// Type export
export type SomeType = {
  a: number
}

export as namespace MyModule
```

注意最后一行——如果你在项目中有一个脚本模式文件，你现在可以直接在全局`MyModule`命名空间下使用该模块（无需先导入它）：

```
let a = new MyModule.SomeExport
```

# 扩展模块

扩展模块类型声明不如给模块加类型常见，但如果你编写了一个 JQuery 插件或 Lodash 混合功能，可能会遇到这种情况。在可能的情况下，尽量避免这样做；相反，考虑使用一个单独的模块。也就是说，不要使用 Lodash 混合功能，而应使用常规函数，不要使用 JQuery 插件——等等，你为什么还在用 JQuery？

## 全局

如果你想扩展另一个模块的全局命名空间或接口，只需创建一个脚本模式文件（参见“模块模式与脚本模式”），然后增加它。请注意，这仅适用于接口和命名空间，因为 TypeScript 会为您处理它们的合并。

例如，让我们给 JQuery 添加一个新的厉害的`marquee`方法。我们首先安装`jquery`本身：

```
npm install jquery --save
npm install @types/jquery --save-dev
```

然后我们会在项目中创建一个新文件，比如*jquery-extensions.d.ts*，并将`marquee`添加到 JQuery 的全局`JQuery`接口中（我发现 JQuery 在其类型声明中定义了其方法）：

```
interface JQuery {
  marquee(speed: number): JQuery<HTMLElement>
}
```

现在，在任何使用 JQuery 的文件中，我们都可以使用`marquee`（当然，我们还需要为`marquee`添加一个运行时实现）：

```
import $ from 'jquery'
$(myElement).marquee(3)
```

注意，这与我们在“安全扩展原型”中用来扩展内置全局变量的技术相同。

## 模块

扩展模块导出有点棘手，并且有更多陷阱：你需要正确地为你的扩展类型，运行时按正确的顺序加载你的模块，并确保当你扩展的模块类型声明结构发生变化时更新你的扩展类型。

例如，让我们为 React 类型化一个新的导出。我们首先安装 React 及其类型声明：

```
npm install react --save
npm install @types/react --save-dev
```

然后我们利用模块合并（参见“声明合并”），简单地声明一个与我们的 React 模块同名的模块：

```
import {ReactNode} from 'react'

declare module 'react' {
  export function inspect(element: ReactNode): void
}
```

注意，与我们用于扩展全局变量的示例不同，我们的扩展文件是否处于模块模式或脚本模式并不重要。

可以考虑从模块中扩展特定的导出功能吗？灵感来自[ReasonReact](https://reasonml.github.io/reason-react)，假设我们想为我们的 React 组件添加一个内置的 reducer（reducer 是为 React 组件声明显式状态转换的一种方式）。在撰写本文时，React 的类型声明将`React.Component`类型声明为一个接口和一个类，这两者会合并为单个 UMD 导出：

```
export = React
export as namespace React

declare namespace React {
  interface Component<P = {}, S = {}, SS = any>
    extends ComponentLifecycle<P, S, SS> {}
  class Component<P, S> {
    constructor(props: Readonly<P>)
    // ...
  }
  // ...
}
```

让我们通过在项目根目录中的*react-extensions.d.ts*文件中输入以下内容来扩展`Component`与我们的`reducer`方法：

```
import 'react' ![1](img/1.png)

declare module 'react' { ![2](img/2.png)
  interface Component<P, S> { ![3](img/3.png)
    reducer(action: object, state: S): S ![4](img/4.png)
  }
}
```

![1](img/#co_recipes_for_writing_declaration_files_for_third_party_javascript_modules_CO1-1)

我们导入`'react'`，将我们的扩展文件切换到脚本模式，这是我们需要以消耗 React 模块的方式。请注意，我们还可以通过导入其他内容、导出某些内容或导出空对象（`export {}`）等其他方式切换到脚本模式，不一定非要专门导入`'react'`。

![2](img/#co_recipes_for_writing_declaration_files_for_third_party_javascript_modules_CO1-2)

我们声明`'react'`模块，告诉 TypeScript 我们要为该特定的`import`路径声明类型。因为我们已经安装了`@types/react`（它为相同的`'react'`路径定义了一个导出），TypeScript 将会把这个模块声明与`@types/react`提供的合并。

![3](img/#co_recipes_for_writing_declaration_files_for_third_party_javascript_modules_CO1-3)

我们通过声明自己的`Component`接口来增强 React 提供的`Component`接口。根据接口合并的规则（“声明合并”），我们在声明中必须使用与`@types/react`中相同的确切签名。

![4](img/#co_recipes_for_writing_declaration_files_for_third_party_javascript_modules_CO1-4)

最后，我们声明我们的`reducer`方法。

在声明这些类型之后（并假设我们已经在某个地方实现了支持此更新的运行时行为），我们现在可以以类型安全的方式声明具有内置`reducers`的 React 组件：

```
import * as React from 'react'

type Props = {
  // ...
}

type State = {
  count: number
  item: string
}

type Action =
  | {type: 'SET_ITEM', value: string}
  | {type: 'INCREMENT_COUNT'}
  | {type: 'DECREMENT_COUNT'}

class ShoppingBasket extends React.Component<Props, State> {
  reducer(action: Action, state: State): State {
    switch (action.type) {
      case 'SET_ITEM':
        return {...state, item: action.value}
      case 'INCREMENT_COUNT':
        return {...state, count: state.count + 1}
      case 'DECREMENT_COUNT':
        return {...state, count: state.count - 1}
    }
  }
}
```

正如本节开头所述，尽量避免可能使您的模块变脆弱且依赖于加载顺序的模式（即使它很酷）。相反，尝试使用组合，使您的模块扩展消耗它们正在扩展的模块，并导出一个包装器，而不是修改该模块。
