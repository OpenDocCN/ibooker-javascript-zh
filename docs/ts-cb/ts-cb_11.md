# 第十章：TypeScript 和 React

近年来，React 可以说是最受欢迎的 JavaScript 库之一。其简单的组件组合方法改变了我们编写前端（以及在某种程度上后端）应用程序的方式，允许你使用一种称为 JSX 的 JavaScript 语法扩展来声明性地编写 UI 代码。这个简单的原则不仅易于掌握和理解，而且还影响了其他数十个库。

JSX 无疑是 JavaScript 世界的一个重大变革，并且随着 TypeScript 致力于服务于所有 JavaScript 开发者的目标，JSX 也进入了 TypeScript 的领域。事实上，TypeScript 是一个功能齐全的 JSX 编译器。如果你不需要额外的捆绑或额外的工具，TypeScript 就足以启动你的 React 应用程序。在写作时，React 的类型定义每周在 NPM 上下载量达到 2000 万次。VS Code 提供的出色工具和优秀的类型使得 TypeScript 成为全球 React 开发者的首选。

尽管 TypeScript 在 React 开发者中的流行度依然不减，但有一个情况使得与 React 一起使用 TypeScript 略显困难：TypeScript 并不是 React 团队的首选。尽管现在其他基于 JSX 的库大多都是用 TypeScript 编写的，因此提供了出色的类型支持，但 React 团队使用他们自己的静态类型检查器[Flow](https://flow.org)，与 TypeScript 有些类似，但最终不兼容。这意味着数百万开发者依赖的 React 类型是由社区贡献者组成的小组后续创建，并发布在 Definitely Typed 上。虽然`@types/react`被认为是非常优秀的，但它们仍然只是尽力去为像 React 这样复杂的库提供类型。这不可避免地导致一些缺陷。对于那些缺陷显现的地方，本章将为你提供指引。

在本章中，我们将探讨 React 在理论上应该很容易，但 TypeScript 通过抛出复杂的错误信息让你感到困扰的情况。我们将弄清楚这些消息的含义，如何绕过它们，以及哪些解决方案能够从长远来看帮助你。你还将了解各种开发模式及其好处，以及如何使用 TypeScript 内置的 JSX 支持。

你不会得到 React 和 TypeScript 的基本设置指南。生态系统如此广泛丰富，有很多条路可以通往罗马。选择你的框架文档页面，并寻找 TypeScript 相关内容。同时请注意，我假设你有一些 React 的使用经验。在本章中，我们主要讨论如何为 React 进行类型编写。

虽然本章倾向于使用 React，但你也能够将某些学习内容应用到其他基于 JSX 的框架和库中。

# 10.1 编写代理组件

## 问题

你编写了很多标准的 HTML 组件，但不想一直设置所有必要的属性。

## 解决方案

创建代理组件并应用一些模式，使它们适用于您的情况。

## 讨论

大多数 Web 应用程序使用按钮。按钮具有默认为`submit`的`type`属性。这是表单的明智默认设置，在此类表单中，您通过 HTTP 执行操作，将内容 POST 到服务器端 API。但是当您只是想在您的网站上有交互元素时，按钮的正确类型是`button`。这不仅是一种美学选择，而且对于可访问性也非常重要：

```
<button type="button">Click me!</button>
```

当您编写 React 时，很少会将表单提交到具有`submit`类型的服务器，而是与许多`button`类型的按钮交互。处理这类情况的一种好方法是编写代理组件。它们模仿 HTML 元素，但预设了一些属性：

```
function Button(props) {
  return <button type="button" {...props} />;
}
```

思想是`Button`接受与 HTML `button`相同的属性，并将属性展开到 HTML 元素中。将属性展开到 HTML 元素是一个很好的功能，您可以确保能够设置所有 HTML 元素具有的属性，而无需预先知道要设置哪些属性。但是我们如何对它们进行类型化？

可以在 JSX 中使用的所有 HTML 元素都通过`JSX`命名空间中的内部元素定义。当您加载 React 时，`JSX`命名空间将出现为您文件中的全局命名空间，并且您可以通过索引访问所有元素。因此，`Button`的正确属性类型在`JSX.IntrinsicElements`中定义。

###### 注意

替代`JSX.IntrinsicElements`的是`React.ElementType`，这是 React 包中的泛型类型，也包括类和函数组件。对于代理组件，`JSX⁠.Int⁠rin⁠sic​Ele⁠ments`已经足够，并且带来了额外的好处：您的组件与其他类似 React 的框架（如 Preact）保持兼容。

`JSX.IntrinsicElements`是全局`JSX`命名空间中的一种类型。一旦这个命名空间在作用域内，TypeScript 就能够捕捉与您的基于 JSX 的框架兼容的基本元素：

```
type ButtonProps = JSX.IntrinsicElements["button"];

function Button(props: ButtonProps) {
  return <button type="button" {...props} />;
}
```

这包括子元素：我们将它们展开！正如您所见，我们将按钮的类型设置为`button`。由于 props 只是 JavaScript 对象，因此可以通过将其设置为 props 中的属性来覆盖`type`。如果定义了两个同名的键，那么后者将覆盖前者。这可能是期望的行为，但您也可能希望阻止您和您的同事覆盖`type`。使用`Omit<T, K>`辅助类型，您可以从 JSX `button`中获取所有属性，但删除您不想覆盖的键：

```
type ButtonProps = Omit<JSX.IntrinsicElements["button"], "type">;

function Button(props: ButtonProps) {
  return <button type="button" {...props} />;
}

const aButton = <Button type="button">Hi</Button>;
//                      ^
// Type '{ children: string; type: string; }' is not
// assignable to type 'IntrinsicAttributes & ButtonProps'.
// Property 'type' does not exist on type
// 'IntrinsicAttributes & ButtonProps'.(2322)
```

如果您需要`type`为`submit`，您可以创建另一个代理组件：

```
type SubmitButtonProps = Omit<JSX.IntrinsicElements["button"], "type">;

function SubmitButton(props: SubmitButtonProps) {
  return <button type="submit" {...props} />;
}
```

可以根据需要省略属性来扩展这个想法。也许您遵循设计系统，并不希望类名随意设置：

```
type StyledButton = Omit<
  JSX.IntrinsicElements["button"],
  "type" | "className" | "style"
> & {
  type: "primary" | "secondary";
};

function StyledButton({ type, ...allProps }: StyledButton) {
  return <Button type="button" className={`btn-${type}`} {...allProps}/>;
}
```

这甚至允许您重复使用`type`属性名。

我们从类型定义中删除了一些 props，并将它们预设为合理的默认值。现在我们希望确保用户不要忘记设置一些 props，比如图片的`alt`属性或`src`属性。

为此，我们创建一个`MakeRequired`辅助类型，移除可选标志：

```
type MakeRequired<T, K extends keyof T> = Omit<T, K> & Required<Pick<T, K>;
```

并构建我们自己的 props：

```
type ImgProps
  = MakeRequired<
    JSX.IntrinsicElements["img"],
    "alt" | "src"
  >;

export function Img(props: ImgProps) {
  return <img {...props} />;
}

const anImage = <Img />;
//               ^
// Type '{}' is missing the following properties from type
// 'Required<Pick<DetailedHTMLProps<ImgHTMLAttributes<HTMLImageElement>,
//  HTMLImageElement>, "alt" | "src">>': alt, src (2739)
```

通过对原始内置元素类型和代理组件进行一些更改，我们可以确保我们的代码变得更加健壮、更易访问且错误更少。

# 10.2 编写受控组件

## 问题

像输入框这样的表单元素增加了另一种复杂性，因为我们需要决定在哪里管理状态：在浏览器中还是在 React 中。

## 解决方案

写一个代理组件，使用区分联合和可选永不技巧，以确保在运行时不会从未受控切换到受控。

## 讨论

React 区分表单元素之间的 *受控组件* 和 *未受控组件*。当你使用像`input`、`textarea`或`select`这样的常规表单元素时，需要记住底层的 HTML 元素控制它们自己的状态。而在 React 中，元素的状态也是通过 React 来定义的 *。

如果你设置了`value`属性，React 会认为该元素的值也由 React 的状态管理控制，这意味着除非你使用`useState`和相关的 setter 函数维护元素的状态，否则无法修改这个值。

有两种方法可以处理这一问题。首先，你可以选择`defaultValue`作为属性而不是`value`。这将只在第一次渲染时设置输入的`value`，随后将所有控制权交给浏览器：

```
function Input({
  value = "", ...allProps
}: Props) {
  return (
    <input
      defaultValue={value}
      {...allProps}
    />
  );
}
```

或者，你通过 React 的状态管理内部管理`value`。通常，只需将原始输入元素的 props 与我们自己的类型相交集就足够了。我们从内置元素中删除`value`，并将其添加为必需的`string`：

```
type ControlledProps =
  Omit<JSX.IntrinsicElements["input"], "value"> & {
    value: string;
  };
```

然后，我们将`input`元素包装在一个代理组件中。将状态保存在代理组件内部并不是最佳实践；相反，你应该使用`useState`从外部管理它。我们还将从原始输入 props 传递的`onChange`处理器进行转发：

```
function Input({
  value = "", onChange, ...allProps
}: ControlledProps) {
  return (
    <input
      value={value}
      {...allProps}
      onChange={onChange}
    />
  );
}

function AComponentUsingInput() {
  const [val, setVal] = useState("");
  return <Input
    value={val}
    onChange={(e) => {
      setVal(e.target.value);
    }}
  />
}
```

React 在运行时处理从未受控到受控的切换时会发出一个有趣的警告：

> 一个组件正在将未受控输入转换为受控输入。这很可能是由于值从未定义变为已定义值，应该避免这种情况。决定在组件的生命周期内使用受控还是未受控输入元素。

我们可以通过在编译时确保我们要么总是提供一个定义了的字符串`value`，要么提供一个`defaultValue`来防止这个警告，但不能两者都提供。这可以通过使用可选 never 技术来使用歧视联合类型来解决（如 Recipe 3.8 中所示），并且使用`OnlyRequired`帮助类型从 Recipe 8.1 派生可能的属性来控制`JSX.IntrinsicElements["input"]`：

```
import React, { useState } from "react";

// A helper type setting a few properties to be required
type OnlyRequired<T, K extends keyof T = keyof T> = Required<Pick<T, K>> &
  Partial<Omit<T, K>>;

// Branch 1: Make "value" and "onChange" required, drop `defaultValue`
type ControlledProps = OnlyRequired<
  JSX.IntrinsicElements["input"],
  "value" | "onChange"
> & {
  defaultValue?: never;
};

// Branch 2: Drop `value` and `onChange`, make `defaultValue` required
type UncontrolledProps = Omit<
  JSX.IntrinsicElements["input"],
  "value" | "onChange"
> & {
  defaultValue: string;
  value?: never;
  onChange?: never;
};

type InputProps = ControlledProps | UncontrolledProps;

function Input({ ...allProps }: InputProps) {
  return <input {...allProps} />;
}

function Controlled() {
  const [val, setVal] = useState("");
  return <Input value={val} onChange={(e) => setVal(e.target.value)} />;
}

function Uncontrolled() {
  return <Input defaultValue="Hello" />;
}
```

在所有其他情况下，具有可选的`value`或具有`defaultValue`并尝试控制值将被类型系统禁止。

# 10.3 类型化自定义钩子

## 问题

您希望定义自定义钩子并获得适当的类型。

## 解决方案

使用元组类型或*const 上下文*。

## 讨论

让我们在 React 中创建一个自定义钩子，并遵循常规 React 钩子的命名约定：返回一个可以解构的数组（或元组）。例如，`useState`：

```
const [state, setState] = useState(0);
```

为什么我们要使用数组？因为数组的字段没有名称，你可以设置自己的名称：

```
const [count, setCount] = useState(0);
const [darkMode, setDarkMode] = useState(true);
```

因此，如果您有类似的模式，您也想返回一个数组。一个自定义切换钩子可能看起来像这样：

```
export const useToggle = (initialValue: boolean) => {
  const [value, setValue] = useState(initialValue);
  const toggleValue = () => setValue(!value);
  return [value, toggleValue];
}
```

没有什么特别的。我们唯一需要设置的类型是输入参数的类型。让我们试试吧：

```
export const Body = () => {
  const [isVisible, toggleVisible] = useToggle(false)
  return (
    <>
      <button onClick={toggleVisible}></button>
    { /* Error. See below */ }
      {isVisible && <div>World</div>}>}
    </>
  )
}
// Error: Type 'boolean | (() => void)' is not assignable to
// type 'MouseEventHandler<HTMLButtonElement> | undefined'.
// Type 'boolean' is not assignable to type
// 'MouseEventHandler<HTMLButtonElement>'.(2322)
```

那么为什么会失败？错误消息可能会很神秘，但我们应该注意的是第一种类型，它声明为不兼容：`boolean | (() => void)`。这是因为返回一个数组：一个可以容纳尽可能多元素的任意长度列表。从`useToggle`的返回值中，TypeScript 推断出一个数组类型。由于`value`的类型是`boolean`（太好了！），而`toggleValue`的类型是`(() => void)`（一个预期返回空的函数），TypeScript 告诉我们在这个数组中可能存在这两种类型。

这就是破坏与`onClick`兼容性的地方。`onClick`期望一个函数。这没问题，但`toggleValue`（或`toggleVisible`）是一个函数。然而，根据 TypeScript 的说法，它也可以是一个布尔值！TypeScript 告诉你要明确，或至少要进行类型检查。

但我们不应该需要进行额外的类型检查。我们的代码非常清晰。问题在于类型不正确。因为我们不处理一个数组，让我们换一个名字：元组。虽然数组是一个可以有任意长度值的值列表，但我们在元组中确切地知道有多少个值。通常，我们还知道元组中每个元素的类型。

因此，我们不应该返回一个数组而是一个元组在`useToggle`。问题是：在 JavaScript 中，数组和元组是无法区分的。在 TypeScript 的类型系统中，我们可以区分它们。

第一个选择：让我们对返回类型有意识。由于 TypeScript——正确地！——推断出一个数组，我们必须告诉 TypeScript，我们期望一个元组：

```
// add a return type here
export const useToggle = (initialValue: boolean): [boolean, () => void] => {
  const [value, setValue] = useState(initialValue);
  const toggleValue = () => setValue(!value);
  return [value, toggleValue];
};
```

以`[boolean, () => void]`作为返回类型，TypeScript 检查我们在此函数中返回一个元组。TypeScript 不会推断，而是确保您的预期返回类型与实际值匹配。这样一来，您的代码就不会再抛出错误了。

第二个选项：使用*const context*。通过元组，我们知道我们期望的元素数量，并且知道这些元素的类型。这听起来像是使用`const`断言冻结类型的工作：

```
export const useToggle = (initialValue: boolean) => {
  const [value, setValue] = useState(initialValue);
  const toggleValue = () => setValue(!value);
  // here, we freeze the array to a tuple
  return [value, toggleValue] as const;
}
```

现在返回类型是`readonly [boolean, () => void]`，因为`as const`确保您的值是常量且不可更改。从语义上讲，这种类型略有不同，但实际上您无法在`useToggle`外部更改返回的值。因此，使用`readonly`会稍微更加正确。

# 10.4 Typing Generic forwardRef Components

## 问题

您为组件使用`forwardRef`，但需要它们是泛型的。

## 解决方案

对于这个问题有几种解决方案。

## 讨论

如果您在 React 中创建组件库和设计系统，可能已经将`ref`转发到组件内部的 DOM 元素。

特别是当您包装基本组件或叶子节点在*代理组件*（参见 Recipe 10.1）中时，这尤其有用，但您希望像往常一样使用`ref`属性：

```
const Button = React.forwardRef((props, ref) => (
  <button type="button" {...props} ref={ref}>
    {props.children}
  </button>
));

// Usage: You can use your proxy just like you use
// a regular button!
const reference = React.createRef();
<Button className="primary" ref={reference}>Hello</Button>
```

通常为`React.forwardRef`提供类型通常非常简单。`@types/react`中提供的类型具有可以在调用`React.forwardRef`时设置的泛型类型变量。在这种情况下，显式注释您的类型是正确的做法：

```
type ButtonProps = JSX.IntrinsicElements["button"];

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  (props, ref) => (
    <button type="button" {...props} ref={ref}>
      {props.children}
    </button>
  )
);

// Usage
const reference = React.createRef<HTMLButtonElement>();
<Button className="primary" ref={reference}>Hello</Button>
```

到目前为止一切都很好。但是，如果您有一个接受泛型属性的组件，情况会变得有些复杂。以下组件生成了一个列表项的列表，您可以使用`button`元素选择每一行：

```
type ClickableListProps<T> = {
  items: T[];
  onSelect: (item: T) => void;
};

function ClickableList<T>(props: ClickableListProps<T>) {
  return (
    <ul>
      {props.items.map((item, idx) => (
        <li>
          <button key={idx} onClick={() => props.onSelect(item)}>
            Choose
          </button>
          {item}
        </li>
      ))}
    </ul>
  );
}

// Usage
const items = [1, 2, 3, 4];
<ClickableList items={items}
  onSelect={(item) => {
    // item is of type number
    console.log(item);
  } } />
```

您希望具有额外的类型安全性，以便在`on​Sel⁠ect`回调中使用类型安全的`item`。假设您想要创建一个指向内部`ul`元素的`ref`：该如何操作？让我们将`ClickableList`组件更改为一个接受`ForwardRef`的内部函数组件，并将其作为参数传递给`React.forwardRef`函数：

```
// The original component extended with a `ref`
function ClickableListInner<T>(
  props: ClickableListProps<T>,
  ref: React.ForwardedRef<HTMLUListElement>
) {
  return (
    <ul ref={ref}>
      {props.items.map((item, i) => (
        <li key={i}>
          <button onClick={(el) => props.onSelect(item)}>Select</button>
          {item}
        </li>
      ))}
    </ul>
  );
}

// As an argument in `React.forwardRef`
const ClickableList = React.forwardRef(ClickableListInner)
```

这样编译没有问题，但有一个缺点：我们无法为`Cl⁠ick⁠ab⁠le​Li⁠st⁠Prop⁠s`分配泛型类型变量。它默认变为`unknown`。与`any`相比这是好的，但也略微让人烦恼。当我们使用`ClickableList`时，我们知道要传递哪些项目，并且希望根据类型进行类型化！那么我们该如何实现呢？答案有些棘手……而且您有几个选项。

第一个选项是进行类型断言，恢复原始的函数签名：

```
const ClickableList = React.forwardRef(ClickableListInner) as <T>(
  props: ClickableListProps<T> & { ref?: React.ForwardedRef<HTMLUListElement> }
) => ReturnType<typeof ClickableListInner>;
```

如果您只有少数情况需要泛型`forwardRef`组件，类型断言效果很好，但当您处理大量这些组件时可能会显得笨拙。此外，您为应该是默认行为的事物引入了一个不安全的运算符。

第二个选择是使用包装组件创建自定义引用。虽然`ref`是 React 组件的保留字，但你可以使用自定义属性来模拟类似的行为。这同样有效：

```
type ClickableListProps<T> = {
  items: T[];
  onSelect: (item: T) => void;
  mRef?: React.Ref<HTMLUListElement> | null;
};

export function ClickableList<T>(
  props: ClickableListProps<T>
) {
  return (
    <ul ref={props.mRef}>
      {props.items.map((item, i) => (
        <li key={i}>
          <button onClick={(el) => props.onSelect(item)}>Select</button>
          {item}
        </li>
      ))}
    </ul>
  );
}
```

然而，你引入了一个新的 API。需要注意的是，还有使用包装组件的可能性，允许你在*内部*组件中使用`forwardRef`并向外部暴露自定义的`ref`属性：

```
function ClickableListInner<T>(
  props: ClickableListProps<T>,
  ref: React.ForwardedRef<HTMLUListElement>
) {
  return (
    <ul ref={ref}>
      {props.items.map((item, i) => (
        <li key={i}>
          <button onClick={(el) => props.onSelect(item)}>Select</button>
          {item}
        </li>
      ))}
    </ul>
  );
}

const ClickableListWithRef = forwardRef(ClickableListInner);

type ClickableListWithRefProps<T> = ClickableListProps<T> & {
  mRef?: React.Ref<HTMLUListElement>;
};

export function ClickableList<T>({
  mRef,
  ...props
}: ClickableListWithRefProps<T>) {
  return <ClickableListWithRef ref={mRef} {...props} />;
}
```

如果你只想要传递那个引用，两者都是有效的解决方案。如果你想要一个一致的 API，你可能会寻找其他的解决方案。

第三种选择是用你自己的类型定义增强`forwardRef`。TypeScript 提供了一种称为[*higher-order function type inference*](https://oreil.ly/rVsq9)的功能，允许将自由类型参数传播到外部函数。

这听起来很像我们最初想要的`forwardRef`，但它与我们当前的类型定义不兼容。原因是高阶函数类型推断仅适用于普通函数类型。`forwardRef`内部的函数声明还会为`defaultProps`等添加属性。这些都是来自类组件时代的遗留物，也许你并不想使用它们。

因此，不需要额外的属性，应该可以使用高阶函数类型推断！

我们正在使用 TypeScript，因此可以重新声明和重新定义自己的全局`module`、`namespace`和`interface`声明。声明合并是一个强大的工具，我们将要使用它：

```
// Redecalare forwardRef
declare module "react" {
  function forwardRef<T, P = {}>(
    render: (props: P, ref: React.Ref<T>) => React.ReactElement | null
  ): (props: P & React.RefAttributes<T>) => React.ReactElement | null;
}

// Just write your components like you're used to!

type ClickableListProps<T> = {
  items: T[];
  onSelect: (item: T) => void;
};
function ClickableListInner<T>(
  props: ClickableListProps<T>,
  ref: React.ForwardedRef<HTMLUListElement>
) {
  return (
    <ul ref={ref}>
      {props.items.map((item, i) => (
        <li key={i}>
          <button onClick={(el) => props.onSelect(item)}>Select</button>
          {item}
        </li>
      ))}
    </ul>
  );
}

export const ClickableList = React.forwardRef(ClickableListInner);
```

这个解决方案的好处是，你可以再次编写常规的 JavaScript，并且完全在类型层面上进行工作。另外，重新声明是模块作用域的：不会干扰来自其他模块的任何`forwardRef`调用！

# 10.5 为上下文 API 提供类型

## 问题

你希望在应用程序中使用上下文 API 进行全局管理，但你不知道如何处理类型定义的最佳方式。

## 解决方案

要么为上下文设置默认属性并让类型推断，要么创建你上下文属性的部分并显式实例化泛型类型参数。如果你不想提供默认值，但希望确保所有属性都被提供，可以创建一个辅助函数。

## 讨论

React 的上下文 API 允许你在全局级别共享数据。为了使用它，你需要两件事：

提供者

提供者将数据传递给子树。

消费者

消费者是在渲染属性内部*消费*传递的数据的组件。

使用 React 的类型定义，大多数情况下可以直接使用上下文。一切都是通过类型推断和泛型完成的。

首先，我们创建一个上下文。在这里，我们想要存储全局应用程序设置，如主题和应用程序的语言，以及全局状态。在创建 React 上下文时，我们希望传递默认属性：

```
import React from "react";

const AppContext = React.createContext({
  authenticated: true,
  lang: "en",
  theme: "dark",
});
```

因此，您所需的所有类型操作都已完成。我们有三个属性：`authenticated`，`lang`和`theme`；它们的类型分别是`boolean`和`string`。当您使用它们时，React 的类型信息将提供正确的类型。

接下来，组件树中高层次的组件需要提供上下文，例如应用程序的根组件。此提供程序将向下传播你设置的值到每个消费者：

```
function App() {
  return (
    <AppContext.Provider
      value={{
        authenticated: true,
        lang: "de",
        theme: "light",
      }}
    >
      <Header />
    </AppContext.Provider>
  );
}
```

现在，此树中的每个组件都可以消费此上下文。当您忘记一个属性或使用错误类型时，您将立即收到类型错误：

```
function App() {
// Property 'theme' is missing in type '{ lang: string; }' but required
// in type '{ lang: string; theme: string; authenticated: boolean }'.(2741)
  return (
    <AppContext.Provider
      value={{
        lang: "de",
      }}
    >
      <Header />
    </AppContext.Provider>
  );
}
```

现在，让我们消费我们的全局状态。可以通过渲染道具来消费上下文。您可以深度解构您的渲染道具，以获取您想要处理的属性：

```
function Header() {
  return (
    <AppContext.Consumer>
      {({ authenticated }) => {
        if (authenticated) {
          return <h1>Logged in!</h1>;
        }
        return <h1>You need to sign in</h1>;
      }}
    </AppContext.Consumer>
  );
}
```

使用上下文的另一种方法是通过相应的`useContext`钩子：

```
function Header() {
  const { authenticated } = useContext(AppContext);
  if (authenticated) {
    return <h1>Logged in!</h1>;
  }
  return <h1>You need to sign in</h1>;
}
```

因为我们早些时候定义了正确类型的属性，此时`authenticated`是布尔类型。再次强调，我们不必采取任何措施来获得此额外类型安全性。

前面的整个示例在我们拥有默认属性和值时效果最佳。有时您没有默认值或需要更灵活地设置属性。

与从默认值推断一切不同，我们显式地注释泛型类型参数，而不是使用完整类型，而是使用`Partial`。

我们为上下文的 props 创建了一个类型：

```
type ContextProps = {
  authenticated: boolean;
  lang: string;
  theme: string;
};
```

并初始化新的上下文：

```
const AppContext = React.createContext<Partial<ContextProps>>({});
```

更改上下文默认属性的语义也会对您的组件产生一些副作用。现在您不需要提供每个值；空上下文对象也可以达到相同效果！您的所有属性都是可选的：

```
function App() {
  return (
    <AppContext.Provider
      value={{
        authenticated: true,
      }}
    >
      <Header />
    </AppContext.Provider>
  );
}
```

这也意味着您需要检查每个属性是否已定义。这不会更改您依赖`boolean`值的代码，但是其他每个属性都需要另一个`undefined`检查：

```
function Header() {
  const { authenticated, lang } = useContext(AppContext);
  if (authenticated && lang) {
    return <>
      <h1>Logged in!</h1>
      <p>Your language setting is set to {lang}</p>
    </> ;
  }
  return <h1>You need to sign in (or don't you have a language setting?)</h1>;
}
```

如果无法提供默认值并希望确保所有属性由上下文提供者提供，则可以使用辅助函数来帮助自己。在这里，我们希望显式泛型实例化以提供类型，但提供正确的类型保护，以便在消费上下文时正确设置所有可能的未定义值。

```
function createContext<Props extends {}>() { ![1](img/1.png)
  const ctx = React.createContext<Props | undefined>(undefined); ![2](img/2.png)
  function useInnerCtx() { ![3](img/3.png)
    const c = useContext(ctx);
    if (c === undefined) ![4](img/4.png)
      throw new Error("Context must be consumed within a Provider");
    return c; ![5](img/5.png)
  }
  return [useInnerCtx, ctx.Provider as React.Provider<Props>] as const; ![6](img/6.png)
}
```

`createContext`中发生了什么？

![1](img/#co_typescript_and_react_CO1-1)

我们创建了一个没有函数参数但有泛型类型参数的函数。如果没有与函数参数的连接，我们无法通过推断实例化`Props`。这意味着为了`createContext`提供正确的类型，我们需要显式实例化它。

![2](img/#co_typescript_and_react_CO1-2)

我们创建了一个允许`Props`或`undefined`的上下文。添加了`undefined`类型后，我们可以将`undefined`作为值传递。不设默认值！

![3](img/#co_typescript_and_react_CO1-3)

在`createContext`内部，我们创建了一个自定义钩子。此钩子使用新创建的上下文`ctx`包装`useContext`。

![4](img/#co_typescript_and_react_CO1-4)

然后我们进行类型守卫，检查返回的 `Props` 是否包含 `undefined`。记住，调用 `createContext` 时，我们用 `Props | undefined` 实例化泛型类型参数。此行再次从联合类型中移除 `undefined`。

![5](img/#co_typescript_and_react_CO1-5)

这意味着在这里，`c` 就是 `Props`。

![6](img/#co_typescript_and_react_CO1-6)

我们断言 `ctx.Provider` 不接受 `undefined` 值。我们使用 `as const` 来返回 `[useInnerContext, ctx.Provider]` 作为元组类型。

使用类似于 `React.createContext` 的 `createContext`：

```
const [useAppContext, AppContextProvider] = createContext<ContextProps>();
```

当使用 `AppContextProvider` 时，我们需要提供所有值：

```
function App() {
  return (
    <AppContextProvider
      value={{ lang: "en", theme: "dark", authenticated: true }}
    >
      <Header />
    </AppContextProvider>
  );
}

function Header() {
  // consuming Context doesn't change much
  const { authenticated } = useAppContext();
  if (authenticated) {
    return <h1>Logged in!</h1>;
  }
  return <h1>You need to sign in</h1>;
}

```

根据您的用例，您可以在没有太多开销的情况下获得精确的类型。

# 10.6 类型化高阶组件

## 问题

您正在编写*高阶组件*以预设其他组件的某些属性，但不知道如何对其进行类型化。

## 解决方案

使用 `@types/react` 中的 `React.ComponentType<P>` 类型来定义一个组件，该组件扩展了您的预设属性。

## 讨论

React 受函数式编程的影响，这在组件设计（通过函数）、组合（通过组合）和更新（无状态，单向数据流）方式中可以看出。函数式编程技术和范式迅速在 React 开发中找到了应用。其中一种技术是高阶组件，它从*高阶函数*中汲取灵感。

高阶函数接受一个或多个参数来返回一个新的函数。有时这些参数用于预填充其他参数，例如我们在第七章中看到的柯里化示例。高阶组件类似：它们接受一个或多个组件，并返回另一个组件。通常情况下，您创建它们来预填充某些属性，以确保稍后不会更改它们。

想象一个通用的 `Card` 组件，它以字符串形式接受 `title` 和 `content`：

```
type CardProps = {
  title: string;
  content: string;
};

function Card({ title, content }: CardProps) {
  return (
    <>
      <h2>{title}</h2>
      <div>{content}</div>
    </>
  );
}
```

您可以使用此卡来显示某些事件，如警告、信息气泡和错误消息。最基本的信息卡将其标题设置为 `"Info"`：

```
<Card title="Info" content="Your task has been processed" />;
```

您可以对 `Card` 的属性进行子集化处理，以允许仅设置 `title` 的某个特定子集字符串，但另一方面，您希望尽可能地重用 `Card`。因此，您创建了一个新组件，它已将 `title` 设置为 `"Info"`，并且只允许设置其他属性：

```
const Info = withInjectedProps({ title: "Info" }, Card);

// This should work
<Info content="Your task has been processed" />;

// This should throw an error
<Info content="Your task has been processed" title="Warning" />;
```

换句话说，您*注入*了一部分属性，并使用新创建的组件设置了剩余的属性。函数 `withInjectedProps` 可以轻松编写：

```
function withInjectedProps(injected, Component) {
  return function (props) {
    const newProps = { ...injected, ...props };
    return <Component {...newProps} />;
  };
}
```

它接受 `injected` 属性和 `Component` 作为参数，返回一个新的函数组件，该函数组件接受剩余的属性作为参数，并使用合并的属性实例化原始组件。

那么我们如何对 `withInjectedProps` 进行类型化呢？让我们看看结果，并了解其中的内容：

```
function withInjectedProps<T extends {}, U extends T>( ![1](img/1.png)
  injected: T,
  Component: React.ComponentType<U> ![2](img/2.png)
) {
  return function (props: Omit<U, keyof T>) { ![3](img/3.png)
    const newProps = { ...injected, ...props } as U; ![4](img/4.png)
    return <Component {...newProps} />;
  };
}
```

下面是发生的情况：

![1](img/#co_typescript_and_react_CO2-1)

我们需要定义两个通用类型参数。`T` 是我们已经注入的 props，它扩展自 `{}` 以确保我们只传递对象。`U` 是 `Component` 的所有 props 的通用类型参数。`U` *扩展* `T`，这意味着 `U` 是 `T` 的一个子集。这表示 `U` 拥有比 `T` 更多的属性，但需要包括 `T` 已经定义的内容。

![2](img/#co_typescript_and_react_CO2-2)

我们将 `Component` 定义为类型 `React.ComponentType<U>`。这包括类组件和函数组件，并表示 props 将设置为 `U`。通过 `T` 和 `U` 的关系以及我们定义 `withInjectedProps` 的参数方式，我们确保传递给 `Component` 的所有内容都定义了带有 `injected` 的 `Component` 属性的子集。如果我们打字错误，我们很快就会收到第一个错误信息！

![3](img/#co_typescript_and_react_CO2-3)

返回的函数组件将接受剩余的 props。通过 `Omit<U, keyof T>`，我们确保不允许再次设置预填充的属性。

![4](img/#co_typescript_and_react_CO2-4)

合并 `T` 和 `Omit<U, keyof T>` 应该再次得到 `U`，但由于通用类型参数可以显式实例化为不同的内容，它们可能不再适合 `Component`。类型断言有助于确保 props 实际上是我们想要的。

就是这样！有了这些新类型，我们可以得到适当的自动完成和错误提示：

```
const Info = withInjectedProps({ title: "Info" }, Card);

<Info content="Your task has been processed" />;
<Info content="Your task has been processed" title="Warning" />;
//                                           ^
// Type '{ content: string; title: string; }' is not assignable
// to type 'IntrinsicAttributes & Omit<CardProps, "title">'.
// Property 'title' does not exist on type
// 'IntrinsicAttributes & Omit<CardProps, "title">'.(2322)
```

`withInjectedProps` 如此灵活，我们可以推导出创建各种情况下的高阶组件的高阶函数，比如 `withTitle`，它在这里用于预填充类型为 `string` 的 `title` 属性：

```
function withTitle<U extends { title: string }>(
  title: string,
  Component: React.ComponentType<U>
) {
  return withInjectedProps({ title }, Component);
}
```

您的函数式编程技能无限制。

# 10.7 React 合成事件系统中的回调函数类型化

## 问题

您希望为 React 中所有浏览器事件获得最佳可能的类型，并使用类型系统将您的回调限制在兼容的元素上。

## 解决方案

使用 `@types/react` 的事件类型，并使用通用类型参数专门化组件。

## 讨论

Web 应用程序通过用户交互变得生动起来。每次用户交互都会触发一个事件。事件至关重要，而 TypeScript 的 React 类型支持很好，但要求您不要使用 *lib.dom.d.ts* 中的原生事件。如果使用，React 会抛出错误：

```
type WithChildren<T = {}> = T & { children?: React.ReactNode };

type ButtonProps = {
  onClick: (event: MouseEvent) => void;
} & WithChildren;

function Button({ onClick, children }: ButtonProps) {
  return <button onClick={onClick}>{children}</button>;
//               ^
// Type '(event: MouseEvent) => void' is not assignable to
// type 'MouseEventHandler<HTMLButtonElement>'.
// Types of parameters 'event' and 'event' are incompatible.
// Type 'MouseEvent<HTMLButtonElement, MouseEvent>' is missing the following
// properties from type 'MouseEvent': offsetX, offsetY, x, y,
// and 14 more.(2322)
}
```

React 使用自己的事件系统，我们称之为*合成事件*。合成事件是浏览器原生事件的跨浏览器包装器，具有与其原生对应物相同的接口，但为了兼容性进行了调整。从 `@types/react` 的类型更改可以使您的回调函数再次兼容：

```
import React from "react";

type WithChildren<T = {}> = T & { children?: React.ReactNode };

type ButtonProps = {
  onClick: (event: React.MouseEvent) => void;
} & WithChildren;

function Button({ onClick, children }: ButtonProps) {
  return <button onClick={onClick}>{children}</button>;
}
```

浏览器的 `MouseEvent` 和 `React.MouseEvent` 在 TypeScript 的*结构*类型系统中有足够的差异，这意味着合成的对应物中缺少一些属性。您可以在前面的错误消息中看到，原始的 `MouseEvent` 比 `React.MouseEvent` 多了 18 个属性，其中一些可能很重要，比如坐标和偏移量，例如，如果您想在画布上绘图。

如果您想要访问原始事件的属性，可以使用 `nativeEvent` 属性：

```
function handleClick(event: React.MouseEvent) {
  console.log(event.nativeEvent.offsetX, event.nativeEvent.offsetY);
}

const btn = <Button onClick={handleClick}>Hello</Button>};
```

支持的事件有：`AnimationEvent`、`ChangeEvent`、`ClipboardEvent`、`Com⁠pos⁠iti⁠on​Ev⁠ent`、`DragEvent`、`FocusEvent`、`FormEvent`、`KeyboardEvent`、`MouseEvent`、`Poi⁠nt⁠er​Ev⁠ent`、`TouchEvent`、`TransitionEvent` 和 `WheelEvent`，以及对所有其他事件使用 `SyntheticEvent`。

到目前为止，我们应用了正确的类型以确保没有任何编译器错误。足够简单。但我们使用 TypeScript 不仅仅是为了执行类型应用的仪式以防止编译器投诉，而且是为了防止可能会有问题的情况发生。

让我们再次考虑一个按钮。或者一个链接（`a` 元素）。这些元素被设计为可点击；这是它们的目的。但在浏览器中，点击事件可以被每个元素接收。没有任何东西阻止您向 `div` 元素添加 `onClick`，这是所有元素中语义含义最少的元素，也没有辅助技术会告诉您 `div` 可以接收 `MouseEvent`，除非您向其添加大量属性。

如果我们能阻止我们的同事（和我们自己）在*错误*的元素上使用定义的事件处理程序，那不是很棒吗？`React.MouseEvent` 是一个泛型类型，它将兼容的元素作为其第一个类型。这被设置为 `Element`，这是浏览器中所有元素的基本类型。但您可以通过子类型化此泛型参数来定义一个更小的兼容元素集合：

```
type WithChildren<T = {}> = T & { children?: React.ReactNode };

// Button maps to an HTMLButtonElement
type ButtonProps = {
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void;
} & WithChildren;

function Button({ onClick, children }: ButtonProps) {
  return <button onClick={onClick}>{children}</button>;
}

// handleClick accepts events from HTMLButtonElement or HTMLAnchorElement
function handleClick(
  event: React.MouseEvent<HTMLButtonElement | HTMLAnchorElement>
) {
  console.log(event.currentTarget.tagName);
}

let button = <Button onClick={handleClick}>Works</Button>;
let link = <a href="/" onClick={handleClick}>Works</a>;

let broken = <div onClick={handleClick}>Does not work</div>;
//                ^
// Type '(event: MouseEvent<HTMLButtonElement | HTMLAnchorElement,
// MouseEvent>) => void' is not assignable to type
//'MouseEventHandler<HTMLDivElement>'.
// Types of parameters 'event' and 'event' are incompatible.
// Type 'MouseEvent<HTMLDivElement, MouseEvent>' is not assignable to
// type 'MouseEvent<HTMLButtonElement | HTMLAnchorElement, MouseEvent>'.
// Type 'HTMLDivElement' is not assignable to type #
// 'HTMLButtonElement | HTMLAnchorElement'.
```

尽管 React 的类型在某些领域给您更多的灵活性，但在其他方面缺少一些功能。例如，浏览器原生的 `InputEvent` 在 `@types/react` 中不受支持。合成事件系统旨在成为跨浏览器的解决方案，而一些 React 兼容的浏览器仍然缺少对 `InputEvent` 的实现。在它们赶上之前，您可以安全地使用基础事件 `SyntheticEvent`：

```
function onInput(event: React.SyntheticEvent) {
  event.preventDefault();
  // do something
}

const inp = <input type="text" onInput={onInput} />;
```

现在，您至少获得了*某种*类型安全性。

# 10.8 类型化多态组件

## 问题

您创建了一个代理组件（参见 Recipe 10.1），它需要作为许多不同 HTML 元素之一的行为。很难获得正确的类型。

## 解决方案

断言转发的属性为 `any` 或直接使用 JSX 工厂 `React.createElement`。

## 讨论

React 中的一个常见模式是定义多态（或`as`）组件，它们预定义行为，但可以作为不同的元素操作。想象一下调用到行动按钮或 CTA，它可以是指向网站的链接或实际的 HTML 按钮。如果你想要将它们类似地样式化，它们应该行为相似，但根据上下文它们应该有适合正确操作的正确 HTML 元素。

###### 注意

选择正确的元素是一个重要的辅助功能因素。`a`和`button`元素表示用户可以单击的东西，但`a`的语义与`button`的语义基本不同。`a`是锚点的缩写，需要有一个指向目标的引用(`href`)。`button`可以被点击，但通常通过 JavaScript 脚本化。这两个元素可能看起来相同，但它们的操作方式不同。它们不仅操作方式不同，而且在使用辅助技术（如屏幕阅读器）时也会以不同的方式进行通告。考虑你的用户，为合适的目的选择正确的元素。

这个想法是你在组件中有一个`as`属性，选择元素类型。根据`as`的元素类型，你可以转发适合该元素类型的属性。当然，你可以将这种模式与我们在[Recipe 10.1](https://wiki.example.org/ch10_proxy_components)中看到的所有内容结合起来。

```
<Cta as="a" href="https://typescript-cookbook.com">
  Hey hey
</Cta>

<Cta as="button" type="button" onClick={(e) => { /* do something */ }}>
  My my
</Cta>
```

在混合使用 TypeScript 时，你希望确保你获得了正确的属性自动补全以及错误的属性报错。如果你向`button`添加了`href`，TypeScript 应该给你正确的波浪线：

```
// Type '{ children: string; as: "button"; type: "button"; href: string; }'
// is not assignable to type 'IntrinsicAttributes & { as: "button"; } &
// ClassAttributes<HTMLButtonElement> &
// ButtonHTMLAttributes<HTMLButtonElement> & { ...; }'.
// Property 'href' does not exist on type ... (2322)
//                             v
<Cta as="button" type="button" href="" ref={(el) => el?.id}>
  My my
</Cta>
```

让我们试着输入`Cta`。首先，我们在完全没有类型的情况下开发组件。在 JavaScript 中，事情看起来并不复杂：

```
function Cta({ as: Component, ...props }) {
  return <Component {...props} />;
}
```

我们提取`as`属性并将其重命名为`Component`。这是 JavaScript 中的解构机制，语法与 TypeScript 注解类似，但适用于解构属性而不是对象本身（在对象本身上，你需要类型注解）。我们将其重命名为大写组件，以便我们可以通过 JSX 实例化它。其余的属性将被收集到`...props`中，并在创建组件时展开。请注意，你也可以用`...props`展开子元素，这是 JSX 的一个不错的副作用。

当我们想要为`Cta`添加类型时，我们创建一个`CtaProps`类型，它适用于`"a"`元素或`"button"`元素，并从`JSX.IntrinsicElements`中获取剩余的属性，与我们在[Recipe 10.1](https://wiki.example.org/ch10_proxy_components)中看到的类似。

```
type CtaElements = "a" | "button";

type CtaProps<T extends CtaElements> = {
  as: T;
} & JSX.IntrinsicElements[T];
```

当我们将类型与`Cta`连接起来时，我们看到函数签名仅需少量额外的注解即可很好地工作。但是在实例化组件时，我们会得到一个非常复杂的错误，告诉我们出了多少问题：

```
function Cta<T extends CtaElements>({
  as: Component,
  ...props
}: CtaProps<T>) {
  return <Component {...props} />;
//        ^
// Type 'Omit<CtaProps<T>, "as" | "children"> & { children: ReactNode; }'
// is not assignable to type 'IntrinsicAttributes &
// LibraryManagedAttributes<T, ClassAttributes<HTMLAnchorElement> &
// AnchorHTMLAttributes<HTMLAnchorElement> & ClassAttributes<...> &
// ButtonHTMLAttributes<...>>'.
// Type 'Omit<CtaProps<T>, "as" | "children"> & { children: ReactNode; }' is not
//  assignable to type
//  'LibraryManagedAttributes<T, ClassAttributes<HTMLAnchorElement>
//  & AnchorHTMLAttributes<HTMLAnchorElement> & ClassAttributes<...>
//  & ButtonHTMLAttributes<...>>'.(2322)
}
```

那么这个消息是从哪里来的呢？为了 TypeScript 正确地处理 JSX，我们需要在名为 `JSX` 的全局命名空间中使用类型定义。如果此命名空间在作用域内，TypeScript 就知道哪些不是组件的元素可以被实例化，以及它们可以接受哪些属性。这些是我们在本例中使用的 `JS⁠X.I⁠ntr⁠ins⁠ic​Ele⁠men⁠ts`，以及在第 10.1 节中使用的内容。

还需要定义的一个类型是 `LibraryManagedAttributes`。这种类型用于提供由框架本身定义的属性（如 `key`）或通过诸如 `defaultProps` 的方式定义的属性：

```
export interface Props {
  name: string;
}

function Greet({ name }: Props) {
  return <div>Hello {name.toUpperCase()}!</div>;
}
// Goes into LibraryManagedAttributes
Greet.defaultProps = { name: "world" };

// Type-checks! No type assertions needed!
let el = <Greet key={1} />;
```

React 的类型定义通过使用条件类型解决了 `LibraryManagedAttributes`。正如我们在第 12.7 节中看到的那样，条件类型在被评估时不会扩展所有可能的联合类型变体。这意味着 TypeScript 将无法检查你的类型定义是否适合组件，因为它无法评估 `Lib⁠rary​Man⁠age⁠dAt⁠trib⁠utes`。

解决此问题的一种方法是将 props 断言为 `any`：

```
function Cta<T extends CtaElements>({
  as: Component,
  ...props
}: CtaProps<T>) {
  return <Component {...(props as any)} />;
}
```

那样虽然可行，但它是一个不安全操作的标志，不应该是不安全的。另一种方法是在这种情况下不使用 JSX，而是使用 JSX 工厂函数 `React.createElement`。

每个 JSX 调用都是对 JSX 工厂调用的语法糖：

```
<h1 className="headline">Hello World</h1>

// will be transformed to
React.createElement("h1", { className: "headline" }, ["Hello World"]);
```

如果您使用嵌套组件，`createElement` 的第三个参数将包含嵌套的工厂函数调用。与 JSX 相比，`React.createElement` 调用起来更加简单，当创建新元素时 TypeScript 不会使用全局的 `JSX` 命名空间。这听起来像是我们需要的一个完美的解决方案。

`React.createElement` 需要三个参数：组件、props 和 children。现在，我们已经通过 `props` 携带了所有的子组件，但是对于 `React.createElement`，我们需要更加明确。这也意味着我们需要明确地定义 `children`。

为此，我们创建了一个 `WithChildren<T>` 辅助类型。它接受一个现有的类型，并以 `React.ReactNode` 的形式添加可选的子组件：

```
type WithChildren<T = {}> = T & { children?: React.ReactNode };
```

`WithChildren` 非常灵活。我们可以用它包装 props 的类型：

```
type CtaProps<T extends CtaElements> = WithChildren<{
  as: T;
} & JSX.IntrinsicElements[T]>;
```

或者我们可以创建一个联合体：

```
type CtaProps<T extends CtaElements> = {
  as: T;
} & JSX.IntrinsicElements[T] & WithChildren;
```

由于 `T` 默认设置为 `{}`，该类型变得通用可用。这使得您在需要时更容易附加 `children`。作为下一步，我们从 `props` 中解构 `children` 并将所有参数传递给 `React.createElement`：

```
function Cta<T extends CtaElements>({
  as: Component,
  children,
  ...props
}: CtaProps<T>) {
  return React.createElement(Component, props, children);
}
```

有了这个，你的多态组件就可以接受正确的参数而没有任何错误。
