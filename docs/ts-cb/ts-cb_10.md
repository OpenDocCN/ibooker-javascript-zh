# 第九章：标准库和外部类型定义

TypeScript 的首席架构师 Anders Hejlsberg 曾说过，他设想“TypeScript 是 JavaScript 的瑞士”，这意味着它不偏爱或努力与单一框架兼容，而是试图迎合所有 JavaScript 框架和变体。过去，TypeScript 曾致力于实现装饰器，说服 Google 不再推广带有装饰器的 JavaScript 方言[AtScript](https://oreil.ly/ZrcKR)，后者是 TypeScript 加上装饰器。TypeScript 的装饰器实现也作为相应的[ECMAScript 装饰器提案](https://oreil.ly/76JuE)的模板。TypeScript 还理解 JSX 语法扩展，允许像 React 或 Preact 这样的框架无限制地使用 TypeScript。

但即使 TypeScript 试图迎合所有 JavaScript 开发者，并为众多框架整合新的有用特性做出巨大努力，仍有一些它无法或不会做到的事情。也许因为某个特性太小众，或者因为一个决策会对太多开发者产生巨大影响。

这就是为什么 TypeScript 被设计为默认可扩展的原因。像命名空间、模块和接口这样的许多 TypeScript 特性允许声明合并，这使你可以根据自己的喜好添加类型定义。

在本章中，我们将看到 TypeScript 如何处理标准 JavaScript 功能，如模块、数组和对象。我们将看到它们的一些限制，分析其背后的原因，并提供合理的解决方法。你将看到 TypeScript 被设计为对各种 JavaScript 变体非常灵活，从合理的默认值开始，并在需要时提供扩展的机会。

# 9.1 使用 Object.keys 迭代对象

## 问题

当你尝试通过迭代其键来访问对象属性时，TypeScript 会向你抛出红色波浪线，告诉你"`‘string’`不能用于索引类型。”

## 解决方案

使用`for-in`循环而不是`Object.keys`，并使用泛型类型参数锁定你的类型。

## 讨论

TypeScript 中一个引人注目的令人头疼的问题是尝试通过迭代其键来访问对象属性。这种模式在 JavaScript 中非常常见，然而 TypeScript 似乎竭尽全力阻止你使用它。我们使用这样一行简单的代码来迭代对象的属性：

```
Object.keys(person).map(k => person[k])
```

这导致 TypeScript 向你抛出红色波浪线，开发者翻桌：“元素隐式具有`'any'`类型，因为类型为`'string'`的表达式不能用于索引类型`'Person'`。”在这种情况下，经验丰富的 JavaScript 开发者感觉 TypeScript 在与他们作对。但就像 TypeScript 的所有决策一样，TypeScript 这样做有很好的理由。

让我们找出原因。看看这个函数：

```
type Person = {
  name: string;
  age: number;
};

function printPerson(p: Person) {
  Object.keys(p).forEach((k) => {
    console.log(k, p[k]);
//                ^
// Element implicitly has an 'any' type because expression
// of type 'string' can't be used to index type 'Person'.
  });
}
```

我们只想通过访问其键来打印`Person`的字段。TypeScript 不允许这样做。`Object.keys(p)` 返回一个`string[]`，这对于访问非常明确定义的对象结构`Person`来说太宽泛了。

但为什么会这样？难道我们只能访问可用的键不是显而易见的吗？这正是使用`Object.keys`的整个目的！是的，但我们也可以传递`Person`的子类型的对象，这些对象可能具有比在`Person`中定义的更多的属性：

```
const me = {
  name: "Stefan",
  age: 40,
  website: "https://fettblog.eu",
};

printPerson(me); // All good!
```

`printPerson` 仍然应该正确工作。它打印更多的属性，但不会中断。它仍然是`p`的键，因此应该可以访问每个属性。但是如果你不仅仅访问`p`呢？

假设`Object.keys` 给你`(keyof Person)[]`。你可以轻松地写出像这样的东西：

```
function printPerson(p: Person) {
  const you: Person = {
    name: "Reader",
    age: NaN,
  };

  Object.keys(p).forEach((k) => {
    console.log(k, you[k]);
  });
}

const me = {
  name: "Stefan",
  age: 40,
  website: "https://fettblog.eu",
};

printPerson(me);
```

如果 `Object.keys(p)` 返回一个`keyof Person[]` 类型的数组，你将能够访问其他`Person`对象。这可能不会累积。在我们的例子中，我们只是打印未定义的内容。但是如果你尝试对这些值做些什么，这将在运行时出错。

TypeScript 阻止你像这样的场景。虽然我们可能认为`Object.keys`是`keyof Person`，但实际上它可能会更多。

缓解这个问题的一种方法是使用类型保护：

```
function isKey<T>(x: T, k: PropertyKey): k is keyof T {
  return k in x;
}

function printPerson(p: Person) {
  Object.keys(p).forEach((k) => {
    if (isKey(p, k)) console.log(k, p[k]); // All fine!
  });
}
```

但这增加了一个本不应该存在的额外步骤。

还有另一种迭代对象的方法，使用`for-in`循环：

```
function printPerson(p: Person) {
  for (let k in p) {
    console.log(k, p[k]);
//                 ^
// Element implicitly has an 'any' type because expression
// of type 'string' can't be used to index type 'Person'.
  }
}
```

TypeScript 将因为同样的原因抛出同样的错误，因为你仍然可以做像这样的事情：

```
function printPerson(p: Person) {
  const you: Person = {
    name: "Reader",
    age: NaN,
  };

  for (let k in p) {
    console.log(k, you[k]);
  }
}

const me = {
  name: "Stefan",
  age: 40,
  website: "https://fettblog.eu",
};

printPerson(me);
```

但它将在运行时中断。但是，像这样写会给你比`Object.keys`版本更多的优势。如果你添加一个泛型，TypeScript 在这种情况下可以更加精确：

```
function printPerson<T extends Person>(p: T) {
  for (let k in p) {
    console.log(k, p[k]); // This works
  }
}
```

而不是要求`p`是`Person`（因此与所有`Person`的子类型兼容），我们添加一个新的泛型类型参数`T`，它是`Person`的子类型。这意味着所有兼容该函数签名的类型仍然兼容，但一旦我们使用`p`，我们处理的是一个显式的子类型，而不是更广泛的超类型`Person`。

我们用`T`替换了与`Person`兼容的东西，但 TypeScript 知道它足够具体，以防止错误。

前面的代码有效。`k`的类型是`keyof T`。这就是为什么我们可以访问`p`，它的类型是`T`。而这种技术仍然防止我们访问缺少特定属性的类型：

```
function printPerson<T extends Person>(p: T) {
  const you: Person = {
    name: "Reader",
    age: NaN,
  };
  for (let k in p) {
    console.log(k, you[k]);
//                 ^
//  Type 'Extract<keyof T, string>' cannot be used to index type 'Person'
  }
}
```

我们无法使用`keyof T`访问`Person`。它们可能不同。但由于`T`是`Person`的子类型，如果我们知道确切的属性名称，我们仍然可以分配属性。

```
p.age = you.age
```

这正是我们想要的。

TypeScript 在这里对其类型非常保守，这可能起初看起来有些奇怪，但它帮助你在你不会考虑到的情况下解决问题。我猜这是 JavaScript 开发人员通常会对编译器尖叫并认为他们在“与之战斗”的部分，但也许 TypeScript 在你不知情的情况下拯救了你。对于这种让人烦恼的情况，TypeScript 至少给了你解决方法。

# 9.2 明确通过类型断言和 `unknown` 强调不安全操作

## 问题

通过 JSON 操作解析任意数据可能会出错，如果数据不正确的话。TypeScript 的默认设置不提供这些不安全操作的任何保障。

## 解决方案

通过使用类型断言而不是类型注释明确突出不安全操作，并确保它们通过对 `unknown` 的原始类型进行修补来实施。

## 讨论

在 Recipe 3.9 中，我们讨论了如何有效地使用类型断言。类型断言是对类型系统的显式调用，以表明某些类型应该是不同的，并基于一些保护措施（例如，不将 `number` 实际上视为 `string`）使 TypeScript 将这个特定值视为新类型。

由于 TypeScript 强大且广泛的类型系统，有时不可避免地需要类型断言。有时甚至是我们想要的，就像在 Recipe 3.9 中展示的那样，我们使用 `fetch` API 从后端获取 JSON 数据。一种方法是调用 `fetch` 并将结果分配给注释类型：

```
type Person = {
  name: string;
  age: number;
};

const ppl: Person[] = await fetch("/api/people").then((res) => res.json());
```

`res.json()` 的结果是 `any`，^(1) 且一切都可以通过类型注释更改为任何其他类型。不能保证结果实际上是 `Person[]`。

另一种方法是使用类型断言而不是类型注释：

```
const ppl = await fetch("/api/people").then((res) => res.json()) as Person[];
```

对于类型系统而言，这是相同的事情，但我们可以轻松地扫描可能存在问题的情况。如果我们不对传入的值进行类型验证（例如使用 Zod 可参考 Recipe 12.5），那么在此处使用类型断言是突出不安全操作的有效方法。

在类型系统中，*不安全* 操作是指我们告诉类型系统我们期望值是某种类型，但是我们没有任何来自类型系统本身的保证，它实际上会成为真实情况。这在我们应用程序的边界处经常发生，我们从某个地方加载数据、处理用户输入或使用内置方法解析数据时。

通过使用特定的关键字来突出显示不安全操作，这些关键字表明类型系统中的显式类型更改。类型断言 (`as`)、类型预测 (`is`) 或断言签名 (`asserts`) 帮助我们找到这些情况。在某些情况下，TypeScript 甚至要求我们遵守其类型视图或根据我们的情况显式更改规则。但并非总是如此。

当我们从某个后端获取数据时，标注类型和写入类型断言一样容易。如果我们不强迫自己使用正确的技术，这些事情可能被忽视。

但我们可以借助 TypeScript 帮助我们做正确的事情。问题出在对 `res.json()` 的调用，它来自 *lib.dom.d.ts* 中的 `Body` 接口：

```
interface Body {
  readonly body: ReadableStream<Uint8Array> | null;
  readonly bodyUsed: boolean;
  arrayBuffer(): Promise<ArrayBuffer>;
  blob(): Promise<Blob>;
  formData(): Promise<FormData>;
  json(): Promise<any>;
  text(): Promise<string>;
}
```

`json()` 调用返回一个 `Promise<any>`，而 `any` 是 TypeScript 中一种松散的类型，TypeScript 在这种情况下完全忽略类型检查。我们需要 `any` 的谨慎兄弟 `unknown`。由于声明合并，我们可以重写 `Body` 类型定义，并将 `json()` 定义得更为严格：

```
interface Body {
  json(): Promise<unknown>;
}
```

当我们进行类型注释时，TypeScript 会警告我们不能将 `unknown` 分配给 `Person[]`：

```
const ppl: Person[] = await fetch("/api/people").then((res) => res.json());
//    ^
// Type 'unknown' is not assignable to type 'Person[]'.ts(2322)
```

但是如果我们进行类型断言，TypeScript 仍然会欣然接受：

```
const ppl = await fetch("/api/people").then((res) => res.json()) as Person[];
```

有了这个，我们可以强制 TypeScript 突出显示不安全的操作。^(2)

# 9.3 使用 defineProperty 进行操作

## 问题

您可以使用 `Object.defineProperty` 动态定义属性，但 TypeScript 不会检测到更改。

## 解决方案

创建一个包装函数，并使用断言签名来更改对象的类型。

## 讨论

在 JavaScript 中，您可以使用 `Ob⁠je⁠ct.de⁠fi⁠ne​Pr⁠op⁠er⁠ty` 动态定义对象属性。如果要求属性为只读，则此方法非常有用。想象一个存储对象，它具有不应被覆盖的最大值：

```
const storage = {
  currentValue: 0
};

Object.defineProperty(storage, 'maxValue', {
  value: 9001,
  writable: false
});

console.log(storage.maxValue); // 9001

storage.maxValue = 2;

console.log(storage.maxValue); // still 9001
```

`defineProperty` 和属性描述符非常复杂。它们允许您以通常保留给内置对象的方式处理属性，因此在较大的代码库中非常常见。TypeScript 在处理 `defineProperty` 时存在问题：

```
const storage = {
  currentValue: 0
};

Object.defineProperty(storage, 'maxValue', {
  value: 9001,
  writable: false
});

console.log(storage.maxValue);
//                  ^
// Property 'maxValue' does not exist on type '{ currentValue: number; }'.
```

如果我们不明确断言为新类型，那么 `maxValue` 将不会附加到 `storage` 的类型中。然而，对于简单的用例，我们可以使用断言签名来帮助自己。

###### 注意

虽然 TypeScript 在使用 `Object.defineProperty` 时可能不支持对象更改，但团队未来可能会为这类情况添加类型或特殊行为。例如，使用 `in` 关键字检查对象是否具有某个特定属性多年来未影响类型。这在 2022 年随着 [TypeScript 4.9](https://oreil.ly/YpyGG) 发生了改变。

想象一个 `assertIsNumber` 函数，您可以确保某个值是 `number` 类型。否则，它会抛出一个错误。这类似于 Node.js 中的 `assert` 函数：

```
function assertIsNumber(val: any) {
  if (typeof val !== "number") {
    throw new AssertionError("Not a number!");
  }
}

function multiply(x, y) {
  assertIsNumber(x);
  assertIsNumber(y);
  // at this point I'm sure x and y are numbers
  // if one assert condition is not true, this position
  // is never reached
  return x * y;
}
```

为了符合这种行为，我们可以添加一个断言签名，告诉 TypeScript 我们在此函数后对类型有更多了解：

```
function assertIsNumber(val: any) : asserts val is number
  if (typeof val !== "number") {
    throw new AssertionError("Not a number!");
  }
}
```

这与类型预测器（参见 Recipe 3.5）非常类似，但没有像 `if` 或 `switch` 这样基于条件的控制流：

```
function multiply(x, y) {
  assertIsNumber(x);
  assertIsNumber(y);
  // Now also TypeScript knows that both x and y are numbers
  return x * y;
}
```

如果您仔细观察它，您会发现这些断言签名可以 *即时更改参数或变量的类型*。这正是 `Object.defineProperty` 所做的。

下面的帮助函数并不力求完全准确或完整。它可能存在错误，可能无法处理 `defineProperty` 规范的每个边缘情况。但它将为我们提供基本功能。首先，我们定义一个名为 `de⁠fin⁠e​Pr⁠ope⁠rty` 的新函数，用作 `Object.defineProperty` 的包装函数：

```
function defineProperty<
  Obj extends object,
  Key extends PropertyKey,
  PDesc extends PropertyDescriptor>
  (obj: Obj, prop: Key, val: PDesc) {
  Object.defineProperty(obj, prop, val);
}
```

我们使用三种泛型：

+   我们想要修改的对象，类型为 `Obj`，是 `object` 的子类型。

+   `Key`类型，是`PropertyKey`的子类型（内置）：`string | number | ​symbol`。

+   `PDesc`，`PropertyDescriptor`的子类型（内置）。这允许我们定义带有所有特性的属性（可写性、可枚举性、可重配置性）。

我们使用泛型，因为 TypeScript 可以将它们缩小到非常具体的单元类型。例如，`PropertyKey` 就是所有数字、字符串和符号。但如果我们使用`Key extends PropertyKey`，我们可以精确定位`prop`为例如类型`"maxValue"`。如果我们想通过添加更多属性来改变原始类型，这是很有帮助的。

`Object.defineProperty`函数要么改变对象，要么在出现问题时抛出错误。这正是断言函数所做的事情。因此，我们的自定义辅助函数`defineProperty`也是如此。

让我们添加一个断言签名。一旦`defineProperty`成功执行，我们的对象就有了另一个属性。我们正在创建一些辅助类型来做这件事。首先是签名：

```
function defineProperty<
  Obj extends object,
  Key extends PropertyKey,
  PDesc extends PropertyDescriptor>
   (obj: Obj, prop: Key, val: PDesc):
     asserts obj is Obj & DefineProperty<Key, PDesc> {
  Object.defineProperty(obj, prop, val);
}
```

然后，`obj`的类型是`Obj`（通过泛型缩小）和我们新定义的属性。

这是`DefineProperty`辅助类型：

```
type DefineProperty<
  Prop extends PropertyKey,
  Desc extends PropertyDescriptor> =
    Desc extends { writable: any, set(val: any): any } ? never :
    Desc extends { writable: any, get(): any } ? never :
    Desc extends { writable: false } ? Readonly<InferValue<Prop, Desc>> :
    Desc extends { writable: true } ? InferValue<Prop, Desc> :
    Readonly<InferValue<Prop, Desc>>;
```

首先，我们处理`PropertyDescriptor`的`writable`属性。这是一组定义原始属性描述符如何工作的边界情况和条件：

+   如果我们设置了`writable`和任何属性访问器（`get`、`set`），我们失败了。`never`告诉我们发生了错误。

+   如果我们将`writable`设置为`false`，该属性是只读的。我们推迟到`InferValue`辅助类型。

+   如果我们将`writable`设置为`true`，该属性就不是只读的。我们同样推迟处理。

+   最后的默认情况与`writable: false`相同，因此是`Readonly<InferValue<Prop, Desc>>`。（`Readonly<T>`是内置的。）

这是处理设置`value`属性的`InferValue`辅助类型：

```
type InferValue<Prop extends PropertyKey, Desc> =
  Desc extends { get(): any, value: any } ? never :
  Desc extends { value: infer T } ? Record<Prop, T> :
  Desc extends { get(): infer T } ? Record<Prop, T> : never;
```

再次一组条件：

+   我们有一个 getter 和一个已设置的值吗？`Object.defineProperty`会抛出错误，所以`never`。

+   如果我们设置了一个值，让我们推断出这个值的类型，并创建一个带有我们定义的属性键和值类型的对象。

+   或者我们推断一个 getter 的返回类型。

+   我们还有其他遗漏的吗？TypeScript 不会让我们像它变成`never`那样处理对象。

有很多辅助类型，但大约 20 行代码可以完美解决：

```
type InferValue<Prop extends PropertyKey, Desc> =
  Desc extends { get(): any, value: any } ? never :
  Desc extends { value: infer T } ? Record<Prop, T> :
  Desc extends { get(): infer T } ? Record<Prop, T> : never;

type DefineProperty<
  Prop extends PropertyKey,
  Desc extends PropertyDescriptor> =
    Desc extends { writable: any, set(val: any): any } ? never :
    Desc extends { writable: any, get(): any } ? never :
    Desc extends { writable: false } ? Readonly<InferValue<Prop, Desc>> :
    Desc extends { writable: true } ? InferValue<Prop, Desc> :
    Readonly<InferValue<Prop, Desc>>

function defineProperty<
  Obj extends object,
  Key extends PropertyKey,
  PDesc extends PropertyDescriptor>
  (obj: Obj, prop: Key, val: PDesc):
    asserts  obj is Obj & DefineProperty<Key, PDesc> {
  Object.defineProperty(obj, prop, val)
}
```

让我们看看 TypeScript 如何处理我们的更改：

```
const storage = {
  currentValue: 0
};

defineProperty(storage, 'maxValue', {
  writable: false, value: 9001
});

storage.maxValue; // it's a number
storage.maxValue = 2; // Error! It's read-only

const storageName = 'My Storage';
defineProperty(storage, 'name', {
  get() {
    return storageName
  }
});

storage.name; // it's a string!

// it's not possible to assign a value and a getter
defineProperty(storage, 'broken', {
  get() {
    return storageName
  },
  value: 4000
});

// storage is never because we have a malicious
// property descriptor
storage;
```

虽然这可能不涵盖所有内容，但已经完成了简单属性定义的大部分工作。

# 9.4 扩展类型 Array.prototype.includes

## 问题

TypeScript 将无法在非常窄的元组或数组中查找广泛类型（如`string`或`number`）的元素。

## 解决方案

使用类型谓词创建泛型辅助函数，从而改变类型参数之间的关系。

## 讨论

我们创建了一个名为`actions`的数组，其中包含我们要执行的一组操作的字符串格式。这个`actions`数组的结果类型是`string[]`。

`execute` 函数以任何字符串作为参数。我们检查这是否是有效的操作，如果是，则执行某些操作：

```
// actions: string[]
const actions = ["CREATE", "READ", "UPDATE", "DELETE"];

function execute(action: string) {
  if (actions.includes(action)) {
    // do something with action
  }
}
```

如果我们想将 `string[]` 缩小为更具体的子集，这就变得有些棘手了。通过 `as const` 添加 *const 上下文*，我们可以将 `actions` 缩小为 `readonly ["CREATE", "READ", "UPDATE", "DELETE"]` 类型。

如果我们想要做详尽检查以确保我们为所有可用的操作都有案例，这非常方便。然而，`actions.includes` 却不认同我们：

```
// Adding const context
// actions: readonly ["CREATE", "READ", "UPDATE", "DELETE"]
const actions = ["CREATE", "READ", "UPDATE", "DELETE"] as const;

function execute(action: string) {
  if (actions.includes(action)) {
//                     ^
// Argument of type 'string' is not assignable to parameter of type
// '"CREATE" | "READ" | "UPDATE" | "DELETE"'.(2345)
  }
}
```

为什么会这样呢？让我们看一下 `Array<T>` 和 `ReadonlyArray<T>` 的类型定义（由于 *const 上下文*，我们使用后者）：

```
interface Array<T> {
  /**
 * Determines whether an array includes a certain element,
 * returning true or false as appropriate.
 * @param searchElement The element to search for.
 * @param fromIndex The position in this array at which
 *   to begin searching for searchElement.
 */
  includes(searchElement: T, fromIndex?: number): boolean;
}

interface ReadonlyArray<T> {
  /**
 * Determines whether an array includes a certain element,
 * returning true or false as appropriate.
 * @param searchElement The element to search for.
 * @param fromIndex The position in this array at which
 *   to begin searching for searchElement.
 */
  includes(searchElement: T, fromIndex?: number): boolean;
}
```

我们想要搜索的元素 (`searchElement`) 需要与数组本身的类型相同！因此，如果我们有 `Array<string>`（或 `string[]` 或 `Re⁠ad⁠on⁠ly​Ar⁠ra⁠y<s⁠tr⁠in⁠g>`），我们只能搜索字符串。在我们的情况下，这意味着 `action` 需要是 `"CREATE" | "READ" | "UPDATE" | "DELETE"` 类型。

突然间，我们的程序变得毫无意义。如果类型已经告诉我们它只能是四个字符串之一，为什么还要搜索某些内容呢？如果我们将 `action` 类型更改为 `"CREATE" | "READ" | "UPDATE" | "DELETE"`，`actions.includes` 就变得无用了。如果我们不改变它，TypeScript 将向我们抛出一个错误，这是理所当然的！

问题之一是，TypeScript 缺乏检查逆变类型的可能性，例如上界泛型。我们可以使用像 `extends` 这样的构造告诉一个类型应该是 `T` 类型的 *子集*，但我们无法检查一个类型是否是 `T` 类型的 *超集*。至少目前还不能！

那么我们可以做些什么呢？

### 选项 1：重新声明 ReadonlyArray

我们可以改变 `ReadonlyArray` 中 `includes` 的行为方式。由于声明合并，我们可以添加对于 `Re⁠ad⁠on⁠ly​Ar⁠ray` 的自定义定义，这些定义在参数上更宽松，在结果上更具体，就像这样：

```
interface ReadonlyArray<T> {
  includes(searchElement: any, fromIndex?: number): searchElement is T;
}
```

允许传递更广泛的 `searchElement` 值集合（字面上的任何值！），如果条件成立，我们通过 *类型预测* 告诉 TypeScript `se⁠ar⁠ch​Ele⁠men⁠t is⁠ T`（我们正在寻找的子集）。

结果表明，这非常有效：

```
const actions = ["CREATE", "READ", "UPDATE", "DELETE"] as const;

function execute(action: string) {
  if(actions.includes(action)) {
    // action: "CREATE" | "READ" | "UPDATE" | "DELETE"
  }
}
```

不过，这里有一个问题。虽然解决方案可以运行，但它假设了正确的内容和需要检查的内容。如果将 `action` 更改为 `number`，TypeScript 通常会抛出一个错误，指出不能搜索这种类型。`actions` 只包含 `string`，为什么还要查找 `number`？这是一个你想要捕捉的错误：

```
// type number has no relation to actions at all
function execute(action: number) {
  if(actions.includes(action)) {
    // do something
  }
}
```

通过对 `ReadonlyArray` 的修改，我们失去了这个检查，因为 `searchElement` 是 `any`。虽然 `action.includes` 的功能仍然按预期工作，但一旦我们在路途中更改函数签名，我们可能就看不到正确的 *问题* 了。

而且更重要的是，我们改变了内置类型的行为。这可能会影响到其他地方的类型检查，并可能在长期运行中造成问题！

###### 提示

如果你通过改变标准库的行为来进行 *类型补丁*，确保这是在模块范围内进行，而不是全局的。

还有另一种方法。

### 选项 2：带有类型断言的辅助函数

正如最初所述， TypeScript 缺乏检查一个值是否属于泛型参数的 *超集* 的可能性之一。通过一个辅助函数，我们可以改变这种关系：

```
function includes<T extends U, U>(coll: ReadonlyArray<T>, el: U): el is T {
  return coll.includes(el as T);
}
```

`includes` 函数以 `ReadonlyArray<T>` 作为参数，并搜索类型为 `U` 的元素。通过我们的泛型边界检查 `T extends U`，这意味着 `U` 是 *T* 的超集（或者 *T* 是 *U* 的子集）。如果方法返回 `true`，我们可以肯定 `el` 是更窄类型 `U`。

唯一需要让实现起作用的事情是在将 `el` 传递给 `Array.prototype.includes` 时进行一点类型断言。原始问题仍然存在！虽然类型断言 `el as T` 是可以的，因为我们已经在函数签名中检查可能的问题。

这意味着一旦我们将例如 `action` 更改为 `number`，我们就能在整个代码中获得正确的错误：

```
function execute(action: number) {
  if(includes(actions, action)) {
//            ^
// Argument of type 'readonly ["CREATE", "READ", "UPDATE", "DELETE"]'
// is not assignable to parameter of type 'readonly number[]'.
  }
}
```

这就是我们想要的行为。一个很好的点是 TypeScript 希望我们改变数组，而不是我们正在查找的元素。这是由泛型类型参数之间的关系所决定的。

###### 提示

如果你遇到与 `Array.prototype.indexOf` 类似的问题，同样的解决方案也适用。

TypeScript 的目标是正确处理所有标准 JavaScript 功能，但有时你必须做出权衡。这种情况需要权衡：你是允许比预期更宽松的参数列表，还是对你已经应该了解更多的类型抛出错误？

类型断言、声明合并和其他工具帮助我们在类型系统不能帮助我们的情况下解决问题。直到它变得比以前更好，通过允许我们在类型空间中进一步移动。

# 9.5 过滤 Nullish 值

## 问题

你希望使用 Boolean 构造函数从数组中过滤掉 nullish 值，但 TypeScript 仍然生成相同的类型，包括 `null` 和 `undefined`。

## 解决方案

使用声明合并重载 `Array` 的 `filter` 方法。

## 讨论

有时你有可能包含 *nullish* 值（`undefined` 或 `null`）的集合：

```
// const array: (number | null | undefined)[]
const array = [1, 2, 3, undefined, 4, null];
```

继续工作时，你希望从集合中删除这些 nullish 值。通常可以使用 `Array` 的 `filter` 方法来完成，也许通过检查值的 *真实性*。`null` 和 `undefined` 是 *假值*，因此它们被过滤掉：

```
const filtered = array.filter((val) => !!val);
```

检查值的真实性的一种便捷方法是将其传递给 Boolean 构造函数。这是简短、直接和非常优雅的阅读方式：

```
// const array: (number | null | undefined)[]
const filtered = array.filter(Boolean);
```

但遗憾的是，它并没有改变我们的类型。我们仍然有 `null` 和 `undefined` 作为过滤后数组的可能类型。

通过打开 `Array` 接口并为 `filter` 添加另一个声明，我们可以将这种特殊情况作为一种重载添加进去：

```
interface Array<T> {
  filter(predicate: BooleanConstructor): NonNullable<T>[]
}

interface ReadonlyArray<T> {
  filter(predicate: BooleanConstructor): NonNullable<T>[]
}
```

通过这种方式，我们摆脱了 nullish 类型，并更清楚地了解了数组内容的类型：

```
// const array: number[]
const filtered = array.filter(Boolean);
```

不错！有什么警告？字面元组和数组。`BooleanConstructor`不仅过滤 nullish 值，还过滤假值。为了获取正确的元素，我们不仅需要返回`NonNullable<T>`，还要引入一种检查真值的类型：

```
type Truthy<T> = T extends "" | false | 0 | 0n ? never : T;

interface Array<T> {
  filter(predicate: BooleanConstructor): Truthy<NonNullable<T>>[];
}

interface ReadonlyArray<T> {
  filter(predicate: BooleanConstructor): Truthy<NonNullable<T>>[];
}

// as const creates a readonly tuple
const array = [0, 1, 2, 3, ``, -0, 0n, false, undefined, null] as const;

// const filtered: (1 | 2 | 3)[]
const filtered = array.filter(Boolean);

const nullOrOne: Array<0 | 1> = [0, 1, 0, 1];

// const onlyOnes: 1[]
const onlyOnes = nullOrOne.filter(Boolean);
```

###### 注意

示例包括`0n`，这在`BigInt`类型中是 0。这种类型仅从 ECMAScript 2020 开始提供。

这给了我们关于预期类型的正确想法，但由于`ReadonlyArray<T>`使用元组的元素类型而不是元组类型本身，我们失去了元组内部类型顺序的信息。

与所有现有 TypeScript 类型扩展一样，请注意这可能会引起副作用。在本地范围内使用它们，并小心使用它们。

# 9.6 扩展模块

## 问题

您使用提供自己视图的库与 HTML 元素，如 Preact 或 React 一起工作。但有时它们的类型定义不包括最新功能。您希望对它们进行修补。

## 解决方案

在模块和接口级别使用声明合并。

## 讨论

*JSX*是 JavaScript 的语法扩展，引入了一种类似 XML 的方式来描述和嵌套组件。基本上，可以将任何可以描述为元素树的东西表达为 JSX。JSX 由流行的 React 框架的创建者引入，使得可以在 JavaScript 中以 HTML 样式编写和嵌套组件，实际上它被转译为一系列函数调用：

```
<button onClick={() => alert('YES')}>Click me</button>

// Transpiles to:

React.createElement("button", { onClick: () => alert('YES') }, 'Click me');
```

JSX 已经被许多框架采纳，即使与 React 没有或几乎没有联系。在第十章中有更多关于 JSX 的内容。

TypeScript 中的 React 类型附带了所有可能的 HTML 元素的大量接口。但有时您的浏览器、框架或代码可能比可能性更大。

假设您希望在 Chrome 中使用最新的图像功能并懒加载图像。这是渐进增强，因此只有理解正在发生的事情的浏览器才知道如何解释这一点。其他浏览器足够强大，不需要关心：

```
<img src="/awesome.jpg" loading="lazy" alt="What an awesome image" />
```

但是你的 TypeScript JSX 代码呢？错误：

```
function Image({ src, alt }) {
  // Property 'loading' does not exist.
  return <img src={src} alt={alt} loading="lazy" />;
}
```

为了防止这种情况，我们可以用我们自己的属性扩展可用的接口。这个 TypeScript 特性被称为*声明合并*。

创建一个*@types*文件夹，并在其中放置一个*jsx.d.ts*文件。更改您的 TypeScript 配置，以便您的编译器选项允许额外类型：

```
{
  "compilerOptions": {
    ...
    /* Type declaration files to be included in compilation. */
    "types": ["@types/**"],
  },
  ...
}
```

我们重新创建了确切的模块和接口结构：

+   模块称为`'react'`。

+   接口是`ImgHTMLAttributes<T>`扩展自 HTMLAttributes<T>。

我们从原始类型定义中知道这一点。在这里，我们添加我们想要的属性：

```
import "react";

declare module "react" {
  interface ImgHTMLAttributes<T> extends HTMLAttributes<T> {
    loading?: "lazy" | "eager" | "auto";
  }
}
```

并且，让我们确保不要忘记 alt 文本：

```
import "react";

declare module "react" {
  interface ImgHTMLAttributes<T> extends HTMLAttributes<T> {
    loading?: "lazy" | "eager" | "auto";
    alt: string;
  }
}
```

非常好！TypeScript 将采用原始定义并合并您的声明。您的自动完成可以提供所有可用选项，并在忘记 alt 文本时显示错误。

当使用[Preact](https://preactjs.com)时，情况变得有些复杂。原始的 HTML 类型定义非常宽泛，不像 React 的类型定义那样具体。这就是为什么在定义图像时我们必须更加明确的原因：

```
declare namespace JSX {
  interface IntrinsicElements {
    img: HTMLAttributes & {
      alt: string;
      src: string;
      loading?: "lazy" | "eager" | "auto";
    };
  }
}
```

这确保了`alt`和`src`都可用，并添加了一个名为`loading`的新属性。尽管技术是相同的：声明合并，它在命名空间、接口和模块级别上都有效。

# 9.7 扩展全局

## 问题

使用像`ResizeObserver`这样的浏览器特性，你会发现它在你当前的 TypeScript 配置中不可用。

## 解决方案

使用自定义类型定义扩展全局命名空间。

## 讨论

TypeScript 将所有 DOM API 的类型存储在*lib.dom.d.ts*中。这个文件是从 Web IDL 文件自动生成的。*Web IDL*代表*Web 接口定义语言*，是 W3C 和 WHATWG 用来定义 Web API 接口的格式。它大约在 2012 年发布，并且自 2016 年以来一直是一个标准。

当你阅读[W3C](https://www.w3.org)的标准时，比如在[Resize Observer](https://oreil.ly/XeSUG)上，你可以在规范的某个地方看到定义的部分或完整的定义。就像这样：

```
enum ResizeObserverBoxOptions {
  "border-box", "content-box", "device-pixel-content-box"
};

dictionary ResizeObserverOptions {
  ResizeObserverBoxOptions box = "content-box";
};

[Exposed=(Window)]
interface ResizeObserver {
  constructor(ResizeObserverCallback callback);
  void observe(Element target, optional ResizeObserverOptions options);
  void unobserve(Element target);
  void disconnect();
};

callback ResizeObserverCallback = void (
  sequence<ResizeObserverEntry> entries,
  ResizeObserver observer
);

[Exposed=Window]
interface ResizeObserverEntry {
  readonly attribute Element target;
  readonly attribute DOMRectReadOnly contentRect;
  readonly attribute FrozenArray<ResizeObserverSize> borderBoxSize;
  readonly attribute FrozenArray<ResizeObserverSize> contentBoxSize;
  readonly attribute FrozenArray<ResizeObserverSize> devicePixelContentBoxSize;
};

interface ResizeObserverSize {
  readonly attribute unrestricted double inlineSize;
  readonly attribute unrestricted double blockSize;
};

interface ResizeObservation {
  constructor(Element target);
  readonly attribute Element target;
  readonly attribute ResizeObserverBoxOptions observedBox;
  readonly attribute FrozenArray<ResizeObserverSize> lastReportedSizes;
};
```

浏览器使用这个作为实现相应 API 的指南。TypeScript 使用这些 IDL 文件来生成*lib.dom.d.ts*。[TypeScript 和 JavaScript 库生成器](https://oreil.ly/WLcLB)项目抓取 Web 标准并提取 IDL 信息。然后，IDL 到 TypeScript 生成器解析 IDL 文件并生成正确的类型定义。

手动维护要抓取的页面。一旦规范足够成熟并且被所有主要浏览器支持，人们会添加一个新资源，并且看到他们的更改将在即将发布的 TypeScript 版本中发布。所以只是时间问题，直到我们在*lib.dom.d.ts*中得到`ResizeObserver`。

如果我们等不及，我们可以自己为当前正在工作的项目添加类型定义。

假设我们生成了`ResizeObserver`的类型。我们将把输出存储在一个名为*resize-observer.d.ts*的文件中。以下是内容：

```
type ResizeObserverBoxOptions =
  "border-box" |
  "content-box" |
  "device-pixel-content-box";

interface ResizeObserverOptions {
  box?: ResizeObserverBoxOptions;
}

interface ResizeObservation {
  readonly lastReportedSizes: ReadonlyArray<ResizeObserverSize>;
  readonly observedBox: ResizeObserverBoxOptions;
  readonly target: Element;
}

declare var ResizeObservation: {
  prototype: ResizeObservation;
  new(target: Element): ResizeObservation;
};

interface ResizeObserver {
  disconnect(): void;
  observe(target: Element, options?: ResizeObserverOptions): void;
  unobserve(target: Element): void;
}

export declare var ResizeObserver: {
  prototype: ResizeObserver;
  new(callback: ResizeObserverCallback): ResizeObserver;
};

interface ResizeObserverEntry {
  readonly borderBoxSize: ReadonlyArray<ResizeObserverSize>;
  readonly contentBoxSize: ReadonlyArray<ResizeObserverSize>;
  readonly contentRect: DOMRectReadOnly;
  readonly devicePixelContentBoxSize: ReadonlyArray<ResizeObserverSize>;
  readonly target: Element;
}

declare var ResizeObserverEntry: {
  prototype: ResizeObserverEntry;
  new(): ResizeObserverEntry;
};

interface ResizeObserverSize {
  readonly blockSize: number;
  readonly inlineSize: number;
}

declare var ResizeObserverSize: {
  prototype: ResizeObserverSize;
  new(): ResizeObserverSize;
};

interface ResizeObserverCallback {
  (entries: ResizeObserverEntry[], observer: ResizeObserver): void;
}
```

我们声明了大量接口和一些实现我们接口的变量，比如`declare var ResizeObserver`，它是定义原型和构造函数的对象：

```
declare var ResizeObserver: {
  prototype: ResizeObserver;
  new(callback: ResizeObserverCallback): ResizeObserver;
};
```

这已经帮了很多忙。我们可以使用（可以说是）冗长的类型声明，并将它们直接放入我们需要它们的文件中。找到了`ResizeObserver`！虽然我们希望在任何地方都能使用它。

由于 TypeScript 的声明合并功能，我们可以根据需要扩展*命名空间*和*接口*。这一次，我们正在扩展*全局命名空间*。

全局命名空间包含所有全局可用的对象和接口。就像`window`对象（和`Window`接口）以及应该作为我们 JavaScript 执行上下文一部分的其他所有内容。我们扩展全局命名空间并将`ResizeObserver`对象添加到其中：

```
declare global { // opening up the namespace
  var ResizeObserver: { // merging ResizeObserver with it
    prototype: ResizeObserver;
    new(callback: ResizeObserverCallback): ResizeObserver;
  }
}
```

让我们将*resize-observer.d.ts*放在名为*@types*的文件夹中。不要忘记将该文件夹添加到 TypeScript 将解析的源列表以及*tsconfig.json*中的类型声明文件夹列表中：

```
{
  "compilerOptions": {
    //...
    "typeRoots": ["@types", "./node_modules/@types"],
    //...
  },
  "include": ["src", "@types"]
}
```

由于在目标浏览器中可能尚未提供`ResizeObserver`，请确保将`ResizeObserver`对象设置为`undefined`。这促使您检查对象是否可用：

```
declare global {
  var ResizeObserver: {
    prototype: ResizeObserver;
    new(callback: ResizeObserverCallback): ResizeObserver;
  } | undefined
}
```

在您的应用程序中：

```
if (typeof ResizeObserver !== 'undefined') {
  const x = new ResizeObserver((entries) => {});
}
```

这使得与`ResizeObserver`一起工作尽可能安全！

可能是 TypeScript 未捕捉到您的环境声明文件和全局扩展。如果发生这种情况，请确保：

+   通过`tsconfig.json`中的`include`属性解析*@types*文件夹。

+   通过将它们添加到*tsconfig.json*编译器选项中的`types`或`typeRoots`，您的环境类型声明文件将被识别为此类文件。

+   在环境声明文件的末尾添加`export {}`，以便 TypeScript 将此文件识别为模块。

# 9.8 将非 JS 模块添加到模块图中

## 问题

您可以使用类似 Webpack 的打包工具从 JavaScript 中加载*.css*或图像文件，但 TypeScript 不会识别这些文件。

## 解决方案

基于文件扩展名全局声明模块。

## 讨论

在 Web 开发中有一种趋势，即将 JavaScript 作为一切的默认入口点，并通过`import`语句处理所有相关资产。为此，您需要一个构建工具，即一个捆绑工具，分析您的代码并创建正确的构件。这方面的一个流行工具是[Webpack](https://webpack.js.org)，一个 JavaScript 捆绑工具，允许您捆绑*所有内容*——CSS、Markdown、SVG、JPEG 等等：

```
// like this
import "./Button.css";

// or this
import styles from "./Button.css";
```

Webpack 使用称为*loaders*的概念，它查看文件结尾并激活某些捆绑概念。在 JavaScript 中导入*.css*文件不是本地操作，这是 Webpack（或您使用的任何捆绑工具）的一部分。但是，我们可以教会 TypeScript 理解这类文件。

###### 注意

ECMAScript 标准委员会中有一个提案，允许导入除 JavaScript 以外的文件，并断言对此类特定内置格式的支持。这将最终影响到 TypeScript。您可以在[此处](https://oreil.ly/stAm5)阅读详细信息。

TypeScript 支持*环境模块声明*，即使是对于在环境中或通过工具可达但“物理上”不存在的模块。一个例子是 Node 的主要内置模块，如`url`、`http`或`path`，如 TypeScript 的文档中所述：

```
declare module "path" {
  export function normalize(p: string): string;
  export function join(...paths: any[]): string;
  export var sep: string;
}
```

这对于我们知道确切名称的模块非常有用。我们也可以对通配符模式使用相同的技术。让我们为所有*.css*文件声明一个通用环境模块：

```
declare module '*.css' {
  // to be done.
}
```

这个模式已经准备好了。这会监听我们想要导入的所有*.css*文件。我们期望的是一组可以添加到我们组件中的类名。由于我们不知道*.css*文件中定义了哪些类，让我们使用一个接受每个字符串键并返回字符串的对象：

```
declare module '*.css' {
  interface IClassNames {
    [className: string]: string
  }
  const classNames: IClassNames;
  export default classNames;
}
```

这就是我们需要做的一切来使我们的文件重新编译。唯一的缺点是我们无法使用确切的类名来获得自动完成和类似的好处。解决这个问题的一种方法是自动生成类型文件。有一些在 [NPM](https://oreil.ly/sDBv0) 上处理这个问题的包。请随意选择您喜欢的包。

如果我们想要将 MDX 类似的东西导入到我们的模块中，那会稍微容易些。MDX 允许我们编写 Markdown，它会解析为常规的 React（或 JSX）组件（更多关于 React 的信息请参见 第十章）。

我们期望一个函数组件（我们可以传递属性给它），返回一个 JSX 元素：

```
declare module '*.mdx' {
  let MDXComponent: (props) => JSX.Element;
  export default MDXComponent;
}
```

Voilà！我们可以在 JavaScript 中加载 *.mdx* 文件并将它们用作组件：

```
import About from '../articles/about.mdx';

function App() {
  return <>
    <About/>
  </>
}
```

如果您不知道会发生什么，请简化自己的生活。您所需做的只是声明该模块。不提供任何类型。TypeScript 允许加载但不会提供任何类型安全性：

```
declare module '*.svg';
```

要让环境模块在您的应用程序中可用，建议在项目的某个地方（可能是根目录）创建一个 *@types* 文件夹。您可以在这里放置任意数量的 *.d.ts* 文件，其中包含您的模块定义。向您的 *tsconfig.json* 添加引用，TypeScript 就知道该如何处理了：

```
{
  ...
  "compilerOptions": {
    ...
    "typeRoots": [
      "./node_modules/@types",
      "./@types"
    ],
    ...
  }
}
```

TypeScript 的主要功能之一是适应所有 JavaScript 的变体。有些是内置的，而其他的则需要您进行额外的补丁。

^(1) 在创建 API 定义时，`unknown` 不存在。此外，TypeScript 强调开发人员的生产力，`res.json()` 是一个广泛使用的方法，如果这样做将会破坏无数应用程序。

^(2) 感谢丹·范德坎姆的 [*Effective TypeScript* 博客](https://effectivetypescript.com) 在这个主题上的灵感。
