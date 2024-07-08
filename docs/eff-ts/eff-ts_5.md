# 第五章：使用 `any` 类型

类型系统传统上是二进制的：语言要么具有完全静态的类型系统，要么具有完全动态的类型系统。TypeScript 模糊了这条界线，因为它的类型系统是*可选的*和*逐步的*。您可以向程序的某些部分添加类型，而另一些部分则不添加。

对于逐步将现有 JavaScript 代码库迁移到 TypeScript 非常重要（参见第八章）。其中的关键在于 `any` 类型，它有效地禁用了代码的部分类型检查。这既强大又容易滥用。学会如何明智地使用 `any` 对于编写有效的 TypeScript 至关重要。本章将指导您如何在保留其优点的同时限制 `any` 的缺点。

# 条款 38: 尽可能使用最狭隘的范围来处理 `any` 类型

考虑这段代码：

```
function processBar(b: Bar) { /* ... */ }

function f() {
  const x = expressionReturningFoo();
  processBar(x);
  //         ~ Argument of type 'Foo' is not assignable to
  //           parameter of type 'Bar'
}
```

如果您从上下文中某种方式知道 `x` 既可分配给 `Foo`，也可分配给 `Bar`，则可以通过两种方式强制 TypeScript 接受此代码：

```
function f1() {
  const x: any = expressionReturningFoo();  // Don't do this
  processBar(x);
}

function f2() {
  const x = expressionReturningFoo();
  processBar(x as any);  // Prefer this
}
```

其中，第二种形式更可取。为什么？因为 `any` 类型仅限于函数参数中的单个表达式。它不会影响到该参数或该行之外的代码。如果 `processBar` 调用后的代码引用 `x`，它的类型仍然是 `Foo`，仍然能触发类型错误，而在第一个示例中，其类型为 `any`，直到函数结束时才会超出范围。

如果从该函数中*返回* `x`，风险会显著增加。看看会发生什么：

```
function f1() {
  const x: any = expressionReturningFoo();
  processBar(x);
  return x;
}

function g() {
  const foo = f1();  // Type is any
  foo.fooMethod();  // This call is unchecked!
}
```

`any` 返回类型是“传染性”的，它可以在代码库中传播开来。由于我们对 `f` 的更改，`g` 中悄然出现了 `any` 类型。如果使用更狭隘范围的 `f2`，这种情况就不会发生。

（这是考虑包括显式返回类型注释的一个好理由，即使返回类型可以推断出来。它防止 `any` 类型“逃逸”。参见 条款 19 中的讨论。）

我们在这里使用 `any` 来消除我们认为不正确的错误。另一种方法是使用 `@ts-ignore`：

```
function f1() {
  const x = expressionReturningFoo();
  // @ts-ignore
  processBar(x);
  return x;
}
```

这将消除下一行的错误，保持 `x` 的类型不变。尽量不要过多依赖 `@ts-ignore`：类型检查器通常有充分的理由抱怨。这也意味着，如果下一行的错误变得更加严重，您将无法得知。

您可能还会遇到仅在较大对象的一个属性上出现类型错误的情况：

```
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value
 // ~~~ Property ... missing in type 'Bar' but required in type 'Foo'
  }
};
```

您可以通过在整个 `config` 对象周围加上 `as any` 来消除此类错误：

```
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value
  }
} as any;  // Don't do this!
```

但这样做的副作用是禁用了其他属性（`a` 和 `b`）的类型检查。使用更狭隘范围的 `any` 可以限制损害：

```
const config: Config = {
  a: 1,
  b: 2,  // These properties are still checked
  c: {
    key: value as any
  }
};
```

## 要记住的事情

+   尽可能将您对 `any` 的使用范围尽可能狭隘，以避免在代码的其他地方意外损失类型安全性。

+   永远不要从函数中返回 `any` 类型。这将悄悄导致任何调用该函数的客户端失去类型安全性。

+   如果需要消除一个错误，可以考虑`@ts-ignore`作为`any`的替代方案。

# 项目 39：更喜欢任意类型的更精确变体而不是纯粹的任意类型

`any`类型包括 JavaScript 中可以表达的所有值。这是一个广泛的集合！它不仅包括所有数字和字符串，还包括所有数组、对象、正则表达式、函数、类和 DOM 元素，更不用说`null`和`undefined`了。当您使用`any`类型时，请问自己是否真的考虑了更具体的东西。是否可以传递正则表达式或函数？

通常答案是“不”，在这种情况下，您可能可以通过使用更具体的类型来保留一些类型安全性：

```
function getLengthBad(array: any) {  // Don't do this!
  return array.length;
}

function getLength(array: any[]) {
  return array.length;
}
```

较后一种版本，使用`any[]`而不是`any`，在三个方面都更好：

+   函数体中对`array.length`的引用经过类型检查。

+   函数的返回类型被推断为`number`而不是`any`。

+   对`getLength`的调用将被检查以确保参数是一个数组：

```
getLengthBad(/123/);  // No error, returns undefined
getLength(/123/);
       // ~~~~~ Argument of type 'RegExp' is not assignable
       //       to parameter of type 'any[]'
```

如果您期望参数是数组的数组，但不关心其类型，可以使用`any[][]`。如果您期望某种对象但不知道其值将是什么，可以使用`{[key: string]: any}`：

```
function hasTwelveLetterKey(o: {[key: string]: any}) {
  for (const key in o) {
    if (key.length === 12) {
      return true;
    }
  }
  return false;
}
```

在这种情况下，您还可以使用`object`类型，该类型包括所有非原始类型。这略有不同，因为虽然您仍然可以枚举键，但无法访问其中任何一个的值：

```
function hasTwelveLetterKey(o: object) {
  for (const key in o) {
    if (key.length === 12) {
      console.log(key, o[key]);
                   //  ~~~~~~ Element implicitly has an 'any' type
                   //         because type '{}' has no index signature
      return true;
    }
  }
  return false;
}
```

如果这种类型符合您的需求，您可能还会对`unknown`类型感兴趣。请参见项目 42。

如果期望一个函数类型，请避免使用`any`。在这里您有几个选项，具体取决于您希望有多精确：

```
type Fn0 = () => any;  // any function callable with no params
type Fn1 = (arg: any) => any;  // With one param
type FnN = (...args: any[]) => any;  // With any number of params
                                     // same as "Function" type
```

所有这些比`any`更精确，因此更可取。请注意在最后一个示例中使用`any[]`作为剩余参数的类型。`any`也可以在这里工作，但不够精确：

```
const numArgsBad = (...args: any) => args.length; // Returns any
const numArgsGood = (...args: any[]) => args.length;  // Returns number
```

这可能是`any[]`类型最常见的用法。

## 要记住的事情

+   当您使用`any`时，请考虑任何 JavaScript 值是否真的是允许的。

+   优先使用更精确的形式，如`any[]`或`{[id: string]: any}`或`() => any`，如果它们更准确地模拟您的数据。

# 项目 40：在类型正确的函数中隐藏不安全类型断言

有许多函数，其类型签名易于编写，但其类型安全的实现却相当困难。尽管编写类型安全的实现是一个高尚的目标，但处理你的代码中不会出现的边缘情况可能并不值得。如果尝试编写类型安全的实现不起作用，可以在具有正确类型签名的函数内部使用隐藏的不安全类型断言。隐藏在类型正确的函数内部的不安全断言比散布在代码中的不安全断言要好得多。

假设你想让一个函数缓存其最后一次调用。这是一种用于消除使用像 React 这样的框架时昂贵函数调用的常见技术。[¹] 编写一个通用的`cacheLast`包装器为任何函数添加此行为将是一个好主意。其声明很容易写出来：

```
declare function cacheLast<T extends Function>(fn: T): T;
```

这是一个实现的尝试：

```
function cacheLast<T extends Function>(fn: T): T {
  let lastArgs: any[]|null = null;
  let lastResult: any;
  return function(...args: any[]) {
      // ~~~~~~~~~~~~~~~~~~~~~~~~~~
      //          Type '(...args: any[]) => any' is not assignable to type 'T'
    if (!lastArgs || !shallowEqual(lastArgs, args)) {
      lastResult = fn(...args);
      lastArgs = args;
    }
    return lastResult;
  };
}
```

错误是有道理的：TypeScript 没有理由相信这个非常宽松的函数与`T`有任何关系。但你知道类型系统将强制要求它以正确的参数调用，并且其返回值给予正确的类型。所以如果在这里添加类型断言，你不应该期望出现太多问题：

```
function cacheLast<T extends Function>(fn: T): T {
  let lastArgs: any[]|null = null;
  let lastResult: any;
  return function(...args: any[]) {
    if (!lastArgs || !shallowEqual(lastArgs, args)) {
      lastResult = fn(...args);
      lastArgs = args;
    }
    return lastResult;
  } as unknown as T;
}
```

这确实对于任何你传递给它的简单函数都非常有效。在这个实现中隐藏了相当多的`any`类型，但你已经把它们从类型签名中排除了，所以调用`cacheLast`的代码不会知道。

（这实际上安全吗？这个实现有一些真实的问题：它不检查连续调用的`this`值是否相同。如果原始函数有定义在其上的属性，那么包装函数将没有这些属性，因此它将不具有相同的类型。但如果你知道在你的代码中不会遇到这些情况，这个实现就可以接受。这个函数*可以*以类型安全的方式编写，但这是一个更复杂的练习，留给读者自行探索。）

前一个示例中的`shallowEqual`函数在两个数组上操作并且易于输入和实现。但对象变体更有趣。与`cacheLast`一样，编写其类型签名很容易：

```
declare function shallowObjectEqual<T extends object>(a: T, b: T): boolean;
```

实现需要小心，因为没有保证`a`和`b`具有相同的键（参见条款 54）：

```
function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
  for (const [k, aVal] of Object.entries(a)) {
    if (!(k in b) || aVal !== b[k]) {
                           // ~~~~ Element implicitly has an 'any' type
                           //      because type '{}' has no index signature
      return false;
    }
  }
  return Object.keys(a).length === Object.keys(b).length;
}
```

TypeScript 抱怨访问`b[k]`尽管你刚刚检查过`k in b`为真有点令人惊讶。但它确实如此，所以你别无选择，只能进行类型转换：

```
function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
  for (const [k, aVal] of Object.entries(a)) {
    if (!(k in b) || aVal !== (b as any)[k]) {
      return false;
    }
  }
  return Object.keys(a).length === Object.keys(b).length;
}
```

这种类型断言是无害的（因为你已经检查过`k in b`），你得到了一个带有清晰类型签名的正确函数。这比在代码中分散迭代和断言以检查对象相等性要好得多！

## 需要记住的事情：

+   有时不安全的类型断言是必要或方便的。当你需要使用时，将其隐藏在一个具有正确签名的函数内部。

# 条款 41：理解不断演化的`any`

在 TypeScript 中，变量的类型通常在声明时确定。之后，它可以通过*精化*（例如检查是否为`null`）来细化，但不能扩展以包含新值。然而，有一个显著的例外涉及`any`类型。

在 JavaScript 中，你可以像这样编写一个生成数字范围的函数：

```
function range(start, limit) {
  const out = [];
  for (let i = start; i < limit; i++) {
    out.push(i);
  }
  return out;
}
```

当你将这个转换成 TypeScript 时，它将按你预期的方式工作：

```
function range(start: number, limit: number) {
  const out = [];
  for (let i = start; i < limit; i++) {
    out.push(i);
  }
  return out;  // Return type inferred as number[]
}
```

然而仔细检查时，令人惊讶的是它竟然工作正常！TypeScript 如何知道 `out` 的类型是 `number[]`，当它被初始化为 `[]` 时，这可能是任何类型的数组？

检查 `out` 的三个出现来揭示其推断类型开始讲述这个故事：

```
function range(start: number, limit: number) {
  const out = [];  // Type is any[]
  for (let i = start; i < limit; i++) {
    out.push(i);  // Type of out is any[]
  }
  return out;  // Type is number[]
}
```

`out` 的类型开始是 `any[]`，一个未区分的数组。但随着我们推送 `number` 值进去，它的类型“演变”成为 `number[]`。

这与缩小范围（条目 22）是不同的。数组的类型可以通过推送不同的元素而扩展：

```
const result = [];  // Type is any[]
result.push('a');
result  // Type is string[]
result.push(1);
result  // Type is (string | number)[]
```

在条件语句中，类型甚至可以在不同分支间变化。这里我们展示了与简单值相同的行为，而不是数组：

```
let val;  // Type is any
if (Math.random() < 0.5) {
  val = /hello/;
  val  // Type is RegExp
} else {
  val = 12;
  val  // Type is number
}
val  // Type is number | RegExp
```

最后一个触发这种“演变 any”行为的情况是，如果变量最初是 `null`。在 `try`/`catch` 块中设置值时经常遇到这种情况：

```
let val = null;  // Type is any
try {
  somethingDangerous();
  val = 12;
  val  // Type is number
} catch (e) {
  console.warn('alas!');
}
val  // Type is number | null
```

有趣的是，这种行为只在变量的类型隐式为 `any` 并且设置了 `noImplicitAny` 时才会发生！增加 *显式* 的 `any` 可以保持类型恒定：

```
let val: any;  // Type is any
if (Math.random() < 0.5) {
  val = /hello/;
  val  // Type is any
} else {
  val = 12;
  val  // Type is any
}
val  // Type is any
```

###### 注意

这种行为可能在编辑器中跟踪起来会令人困惑，因为类型只在你分配或推送元素之后“演变”。检查赋值行上的类型仍将显示 `any` 或 `any[]`。

如果在赋值之前使用一个值，你将会遇到隐式的 any 错误：

```
function range(start: number, limit: number) {
  const out = [];
  //    ~~~ Variable 'out' implicitly has type 'any[]' in some
  //        locations where its type cannot be determined
  if (start === limit) {
    return out;
    //     ~~~ Variable 'out' implicitly has an 'any[]' type
  }
  for (let i = start; i < limit; i++) {
    out.push(i);
  }
  return out;
}
```

换句话说，“演变”的 `any` 类型只有在你 *写* 到它们时才是 `any`。如果你试图在它们仍然是 `any` 的情况下 *读* 取它们，你会得到一个错误。

隐式的 `any` 类型不会通过函数调用而演变。箭头函数在这里会导致推断出错：

```
function makeSquares(start: number, limit: number) {
  const out = [];
     // ~~~ Variable 'out' implicitly has type 'any[]' in some locations
  range(start, limit).forEach(i => {
    out.push(i * i);
  });
  return out;
      // ~~~ Variable 'out' implicitly has an 'any[]' type
}
```

在这种情况下，你可能希望考虑使用数组的 `map` 和 `filter` 方法来在单个语句中构建数组，避免迭代和完全避免演变的 `any`。参见条目 23 和 27。

`any` 的演变带来了关于类型推断的所有常规警告。你的数组的正确类型真的应该是 `(string|number)[]` 吗？或者它应该是 `number[]`，而你错误地推送了一个 `string`？你可能仍然希望提供显式类型注解以获得更好的错误检查，而不是使用演变的 `any`。

## 记住的事情

+   虽然 TypeScript 类型通常只是 *细化*，但隐式的 `any` 和 `any[]` 类型是允许 *演变* 的。当出现这种情况时，你应该能够识别和理解这种结构。

+   为了更好的错误检查，考虑提供显式的类型注解，而不是使用演变的 `any`。

# 条目 42：对于未知类型的值，使用 `unknown` 而不是 `any`

假设你想写一个 YAML 解析器（YAML 可以表示与 JSON 相同的值集，但允许 JSON 语法的超集）。你的 `parseYAML` 方法的返回类型应该是什么？像 `JSON.parse` 一样使用 `any` 是很诱人的：

```
function parseYAML(yaml: string): any {
  // ...
}
```

但这与 条目 38 避免“传染性” `any` 类型的建议相矛盾，特别是不要从函数中返回它们。

理想情况下，你希望用户立即将结果分配给另一种类型：

```
interface Book {
  name: string;
  author: string;
}
const book: Book = parseYAML(`
 name: Wuthering Heights
 author: Emily Brontë
`);
```

虽然没有类型声明，`book`变量将默认成为`any`类型，在使用它时会绕过类型检查：

```
const book = parseYAML(`
 name: Jane Eyre
 author: Charlotte Brontë
`);
alert(book.title);  // No error, alerts "undefined" at runtime
book('read');  // No error, throws "TypeError: book is not a
               // function" at runtime
```

更安全的选择是让`parseYAML`返回一个`unknown`类型：

```
function safeParseYAML(yaml: string): unknown {
  return parseYAML(yaml);
}
const book = safeParseYAML(`
 name: The Tenant of Wildfell Hall
 author: Anne Brontë
`);
alert(book.title);
   // ~~~~ Object is of type 'unknown'
book("read");
// ~~~~~~~~~~ Object is of type 'unknown'
```

要理解`unknown`类型，有助于从可分配性的角度思考`any`。`any`的力量和危险来自两个属性：

+   任何类型都可以赋给`any`类型。

+   `any`类型可以赋给任何其他类型。^(2)

在“将类型视为值集合”(Item 7)的背景下，`any`显然不适合于类型系统，因为一个集合不能同时是所有其他集合的子集和超集。这是`any`强大之处，但也是其问题所在。由于类型检查器是基于集合的，使用`any`实际上会使其失效。

`unknown`类型是`any`的替代品，*可以*适应类型系统。它具有第一个属性（任何类型都可以赋给`unknown`），但不具备第二个属性（`unknown`只能赋给`unknown`和当然`any`）。相反的是`never`类型：它具有第二个属性（可以赋给任何其他类型），但不具备第一个属性（没有类型可以赋给`never`）。

试图访问具有`unknown`类型值的属性是错误的。尝试调用它或进行算术运算也是错误的。你不能对`unknown`做太多事情，这正是关键所在。关于`unknown`类型的错误将促使你添加适当的类型：

```
const book = safeParseYAML(`
 name: Villette
 author: Charlotte Brontë
`) as Book;
alert(book.title);
        // ~~~~~ Property 'title' does not exist on type 'Book'
book('read');
// ~~~~~~~~~ this expression is not callable
```

这些错误更合理。因为`unknown`不能赋值给其他类型，所以需要类型断言。但这也是合适的：我们确实比 TypeScript 更了解结果对象的类型。

当你知道会有一个值但不知道其类型时，适合使用`unknown`。`parseYAML`的结果就是一个例子，但还有其他情况。例如，在 GeoJSON 规范中，Feature 的`properties`属性是任何可 JSON 序列化的东西的集合。因此`unknown`是有意义的：

```
interface Feature {
  id?: string | number;
  geometry: Geometry;
  properties: unknown;
}
```

类型断言并不是从`unknown`对象中恢复类型的唯一方法。`instanceof`检查也可以：

```
function processValue(val: unknown) {
  if (val instanceof Date) {
    val  // Type is Date
  }
}
```

你也可以使用用户定义的类型保护：

```
function isBook(val: unknown): val is Book {
  return (
      typeof(val) === 'object' && val !== null &&
      'name' in val && 'author' in val
  );
}
function processValue(val: unknown) {
  if (isBook(val)) {
    val;  // Type is Book
  }
}
```

TypeScript 需要相当多的证据来缩小`unknown`类型：为了避免在`in`检查中出现错误，首先必须证明`val`是对象类型并且非`null`（因为`typeof null === 'object'`）。

有时你会看到使用泛型参数而不是`unknown`。你可以这样声明`safeParseYAML`函数：

```
function safeParseYAML<T>(yaml: string): T {
  return parseYAML(yaml);
}
```

然而，这在 TypeScript 中通常被认为是不好的风格。它看起来与类型断言不同，但实际上功能上是相同的。最好只返回`unknown`并强制用户使用断言或缩小到他们想要的类型。

`unknown`在“双重断言”中也可以替代`any`使用：

```
declare const foo: Foo;
let barAny = foo as any as Bar;
let barUnk = foo as unknown as Bar;
```

这两种方法在功能上是等效的，但是 `unknown` 形式在进行重构并拆分两个断言时风险较小。在这种情况下，`any` 可能会逃逸和扩散。如果 `unknown` 类型逃逸，它可能只会产生一个错误。

最后一点，你可能会看到使用 `object` 或 `{}` 的代码，类似于本条目中描述 `unknown` 的方式。它们也是广义类型，但比 `unknown` 稍微窄一些：

+   `{}` 类型包含除了 `null` 和 `undefined` 之外的所有值。

+   `object` 类型包括所有非原始类型。这不包括 `true` 或 `12` 或 `"foo"`，但包括对象和数组。

在引入 `unknown` 类型之前，使用 `{}` 更为常见。今天的使用情况有些罕见：只有在你确实知道 `null` 和 `undefined` 不可能存在时，才使用 `{}` 而不是 `unknown`。

## 需要记住的事情

+   `unknown` 类型是 `any` 的类型安全替代方案。当你知道有一个值但不知道其类型时，请使用它。

+   使用 `unknown` 强制用户进行类型断言或进行类型检查。

+   理解 `{}`、`object` 和 `unknown` 之间的区别。

# 项目 43：更喜欢类型安全的方法来进行 Monkey Patching

JavaScript 最著名的特性之一是其对象和类是“开放的”，这意味着你可以向它们添加任意属性。有时会使用这种方式在网页上创建全局变量，通过分配给 `window` 或 `document`：

```
window.monkey = 'Tamarin';
document.monkey = 'Howler';
```

或将数据附加到 DOM 元素：

```
const el = document.getElementById('colobus');
el.home = 'tree';
```

这种风格在使用 jQuery 的代码中特别常见。

你甚至可以向内置原型附加属性，有时会产生令人惊讶的结果：

```
>  `RegExp``.``prototype``.``monkey` `=` `'Capuchin'`
"Capuchin"
> `/123/``.``monkey`
"Capuchin"
```

这些方法通常不是良好的设计。当你将数据附加到 `window` 或 DOM 节点时，你实际上是将其变成全局变量。这样做容易无意中在程序的各个部分之间引入依赖，并意味着每次调用函数时都需要考虑副作用。

添加 TypeScript 会引入另一个问题：虽然类型检查器知道 `Document` 和 `HTMLElement` 的内置属性，但它肯定不知道你添加的属性：

```
document.monkey = 'Tamarin';
      // ~~~~~~ Property 'monkey' does not exist on type 'Document'
```

修复这个错误的最直接方式是使用 `any` 断言：

```
(document as any).monkey = 'Tamarin';  // OK
```

这会满足类型检查器，但是，也应该不会让你感到惊讶，它也有一些缺点。与任何使用 `any` 的情况一样，你将失去类型安全和语言服务：

```
(document as any).monky = 'Tamarin';  // Also OK, misspelled
(document as any).monkey = /Tamarin/;  // Also OK, wrong type
```

最佳解决方案是将数据移出 `document` 或 DOM。但如果不能这样做（可能你正在使用需要它的库或正在迁移 JavaScript 应用程序），那么还有一些次优的选择可用。

一种方法是使用增强功能，这是 `interface` 的特殊能力之一（第 13 项）：

```
interface Document {
  /** Genus or species of monkey patch */
  monkey: string;
}

document.monkey = 'Tamarin';  // OK
```

这比在几个方面使用 `any` 要好：

+   你可以获得类型安全。类型检查器将标记拼写错误或错误类型的赋值。

+   你可以将文档附加到属性 (Item 48)。

+   您可以在属性上获得自动完成。

+   此增强的确切记录是有的。

在模块上下文中（即使用 `import` / `export` 的 TypeScript 文件），您需要添加 `declare global` 才能使其工作：

```
export {};
declare global {
  interface Document {
    /** Genus or species of monkey patch */
    monkey: string;
  }
}
document.monkey = 'Tamarin';  // OK
```

使用增强的主要问题与作用域有关。首先，增强是全局应用的。您无法将其隐藏在代码的其他部分或库中。其次，如果在应用程序运行时分配属性，则无法在此之后仅引入增强。当您补丁 HTML 元素时，这尤为问题，因为页面上的某些元素将具有属性，而其他元素则不会。因此，您可能希望声明该属性为 `string|undefined`。这更加准确，但使得类型使用起来不那么方便。

另一种方法是使用更精确的类型断言：

```
interface MonkeyDocument extends Document {
  /** Genus or species of monkey patch */
  monkey: string;
}

(document as MonkeyDocument).monkey = 'Macaque';
```

TypeScript 对类型断言可以接受，因为 `Document` 和 `MonkeyDocument` 共享属性 (Item 9)。并且在赋值时会获得类型安全。作用域问题也更容易管理：没有全局修改 `Document` 类型，只是引入了一个新类型（只有在引入时才在作用域内）。每次引用 Monkey patched 属性时，必须编写断言（或引入新变量）。但你可以把这视作重构为更结构化代码的鼓励。Monkey patching 不应该*太*容易！

## 需要记住的事情

+   更倾向于结构化代码，而不是在全局变量或 DOM 上存储数据。

+   如果必须在内置类型上存储数据，请使用其中一种类型安全的方法（增强或断言自定义接口）。

+   理解增强的作用域问题。

# Item 44: 跟踪类型覆盖以防止类型安全性回归

一旦为隐式 `any` 类型的值添加类型注释并启用 `noImplicitAny`，你是否免受与任何类型相关问题的影响？答案是否定的；`any` 类型仍然可以通过两种主要方式进入您的程序：

*显式* `any` 类型

即使您遵循 38 和 39 的建议，使您的 `any` 类型变得更窄和更具体，它们仍然是 `any` 类型。特别是像 `any[]` 和 `{[key: string]: any}` 这样的类型一旦索引进入，结果的 `any` 类型就可以流经您的代码。

来自第三方类型声明

特别是从 `@types` 声明文件中，`any` 类型会悄无声息地进入：即使您已启用了 `noImplicitAny` 并且从未输入过 `any`，`any` 类型仍然会在代码中流动。

由于 `any` 类型对类型安全性和开发者体验的负面影响（条目 5），在代码库中跟踪它们的数量是个好主意。有许多方法可以做到这一点，包括 npm 上的 `type-coverage` 包：

```
$ npx type-coverage
9985 / 10117 98.69%
```

这意味着，在这个项目中的 10,117 个符号中，有 9,985 个（98.69%）具有 `any` 之外的类型或 `any` 的别名。如果一个变更无意中引入了 `any` 类型并且它在你的代码中传播，你会看到这个百分比相应下降。

在某种程度上，这个百分比是用来评估你在本章中遵循其他条目建议的程度。使用范围狭窄的 `any` 将减少具有 `any` 类型的符号数量，使用诸如 `any[]` 这样更具体的形式也是如此。通过数字跟踪这一点有助于确保随着时间的推移事情只会变得更好。

即使只收集一次类型覆盖信息也是有益的。使用 `--detail` 标志运行 `type-coverage` 将打印出你的代码中每个 `any` 类型出现的位置：

```
$ npx type-coverage --detail
path/to/code.ts:1:10 getColumnInfo
path/to/module.ts:7:1 pt2
...
```

这些值得调查，因为它们很可能会揭示你没有考虑到的 `any` 的来源。让我们看几个例子。

明确的 `any` 类型通常是你之前为了便利而做出的选择的结果。也许你得到了一个你不想花时间解决的类型错误。或者也许这个类型是你还没有写出来的。或者你可能只是匆忙之间。

使用 `any` 的类型断言可能会阻止类型流向其本应流向的地方。也许你构建了一个处理表格数据的应用程序，并且需要一个单参数函数来构建某种列描述：

```
function getColumnInfo(name: string): any {
  return utils.buildColumnInfo(appState.dataSchema, name);  // Returns any
}
```

`utils.buildColumnInfo` 函数在某个时候返回了 `any`。作为提醒，你在函数中添加了一个注释和一个显式的“: any”注解。

然而，在过去的几个月里，你还为 `ColumnInfo` 添加了一个类型，`utils.buildColumnInfo` 不再返回 `any`。现在的 `any` 注解正在丢弃宝贵的类型信息。去掉它！

第三方 `any` 类型可以有几种形式，但最极端的是当你给整个模块一个 `any` 类型时：

```
declare module 'my-module';
```

现在你可以从 `my-module` 导入任何东西而不会出错。这些符号都具有 `any` 类型，如果通过它们传递值，将导致更多的 `any` 类型：

```
import {someMethod, someSymbol} from 'my-module';  // OK

const pt1 = {
  x: 1,
  y: 2,
};  // type is {x: number, y: number}
const pt2 = someMethod(pt1, someSymbol);  // OK, pt2's type is any
```

由于使用看起来与类型良好的模块相同，很容易忘记你替换了模块。或者可能是一个同事这样做了，而你一开始根本不知道。值得不时重新审视这些。也许这个模块有官方的类型声明。或者也许你已经对模块有足够的了解，可以自己编写类型并将其贡献回社区。

第三方声明中另一个常见的 `any` 来源是类型存在错误时。也许声明没有遵循 Item 29 的建议，并且声明一个函数返回一个联合类型，而实际上它返回了更具体的东西。当你首次使用函数时，修复这个问题似乎不值得，所以你使用了 `any` 断言。但也许声明现在已经修复了。或者也许是时候自己修复它们了！

导致你使用 `any` 类型的考虑可能不再适用。也许现在你可以插入一个类型，而之前你使用了 `any`。也许不再需要一个不安全的类型断言。也许你曾经绕过的类型声明中的错误已经被修复。跟踪你的类型覆盖可以突出显示这些选择，并鼓励你不断重新审视它们。

## 要记住的事情

+   即使设置了 `noImplicitAny`，`any` 类型可能仍会通过显式的 `any` 或第三方类型声明 (`@types`) 进入你的代码。

+   考虑跟踪你的程序有多少是类型良好的。这将鼓励你重新考虑是否使用 `any` 并随着时间增加类型安全性。

^(1) 如果你正在使用 React，你应该使用内置的 `useMemo` 钩子，而不是自己编写。

^(2) 除了 `never`。
