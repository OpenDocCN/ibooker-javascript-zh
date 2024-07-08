# 第六章：字符串模板字面类型

在 TypeScript 的类型系统中，每个值也是一种类型。我们称之为字面类型，在与其他字面类型的联合中，您可以定义一个非常清晰的类型，指明它可以接受哪些值。让我们以`string`的子集为例。您可以确切地定义应包含在集合中的字符串，并排除大量错误。另一端的极端是字符串的整个集合。

但如果之间有什么呢？如果我们可以定义检查特定字符串模式是否可用的类型，并让其余部分更加灵活呢？*字符串模板字面类型*正是如此。它们允许我们定义类型，其中字符串的某些部分是预定义的；其余部分则是开放的，可以用于各种用途。

但更重要的是，与条件类型结合使用时，可以将字符串分割为各个部分并将相同的部分重复使用，这是一种非常强大的工具，特别是当您考虑 JavaScript 中多少代码依赖于字符串内的模式时。

在这一章中，我们将看到各种*字符串模板字面类型*的用例。从简单的字符串模式到根据格式字符串提取参数和类型，您将看到解析字符串作为类型的强大功能。

但我们保持现实。这里的一切都来自真实世界的例子。使用字符串模板字面类型可以实现的功能似乎是无穷无尽的。人们通过编写[拼写检查器](https://oreil.ly/63z2Y)或实现[SQL 解析器](https://oreil.ly/foSvx)将字符串模板字面类型的用法推向极限；看起来这个令人惊叹的功能可以做的事情没有极限。

# 6.1 定义自定义事件系统

## 问题

您正在创建一个自定义事件系统，并希望确保每个事件名称都遵循约定并以`"on"`开头。

## 解决方案

使用字符串模板字面类型描述字符串模式。

## 讨论

JavaScript 事件系统通常具有指示特定字符串是事件的前缀。通常，事件或事件处理程序字符串以`on`开头，但具体实现可能有所不同。

您希望创建自己的事件系统，并希望遵循这一约定。通过 TypeScript 的字符串类型，可以接受所有可能的字符串或子集作为字符串字面量类型的联合类型。尽管一个太广泛，另一个对我们的需求不够灵活。我们不想预先定义每个可能的事件名称；我们想遵循一种模式。

幸运的是，一种称为*字符串模板字面类型*或简称*模板字面类型*的类型正是我们所需要的。模板字面类型允许我们定义字符串字面量，但保留某些部分的灵活性。

例如，一个接受以`on`开头的所有字符串的类型可能如下所示：

```
type EventName = `on${string}`;
```

在语法上，模板文字类型借鉴了 JavaScript 的*模板字符串*。它们以反引号开始和结束，后跟任意字符串。

使用`${}`的特定语法允许向字符串添加 JavaScript 表达式，例如变量、函数调用等：

```
function greet(name: string) {
  return `Hi, ${name}!`;
}

greet("Stefan"); // "Hi, Stefan!"
```

TypeScript 中的模板文字类型非常相似。与 JavaScript 表达式不同，它们允许我们添加一组类型形式的值。定义 HTML 中所有可用标题元素的字符串表示的类型可以如下所示：

```
type Levels = 1 | 2 | 3 | 4 | 5 | 6;

// resolves to "H1" | "H2" | "H3" | "H4" | "H5" | "H6"
type Headings = `H${Levels}`;
```

`Levels`是`number`的一个子集，而`Headings`读作“以 H 开头，后跟与`Levels`兼容的值”。你不能在这里放入每一种类型，只能是那些有字符串表示的类型。

回到`EventName`：

```
type EventName = `on${string}`;
```

像这样定义，`EventName`读起来像“以`"on"`开头，后跟任意字符串”。这包括空字符串。让我们使用`EventName`来创建一个简单的事件系统。在第一步中，我们只想收集回调函数。

为此，我们定义了一个`Callback`类型，它是一个带有一个参数的函数类型：`EventObject`。`EventObject`是一个通用类型，包含事件信息的值：

```
type EventObject<T> = {
  val: T;
};

type Callback<T = any> = (ev: EventObject<T>) => void;
```

此外，我们需要一个类型来存储所有注册的事件回调，`Events`：

```
type Events = {
  [x: EventName]: Callback[] | undefined;
};
```

我们将`EventName`用作索引访问，因为它是`string`的有效子类型。每个索引指向一个回调函数数组。有了我们定义的类型，我们设置了一个`EventSystem`类：

```
class EventSystem {
  events: Events;
  constructor() {
    this.events = {};
  }

  defineEventHandler(ev: EventName, cb: Callback): void {
    this.events[ev] = this.events[ev] ?? [];
    this.events[ev]?.push(cb);
  }

  trigger(ev: EventName, value: any) {
    let callbacks = this.events[ev];
    if (callbacks) {
      callbacks.forEach((cb) => {
        cb({ val: value });
      });
    }
  }
}
```

构造函数创建了一个新的事件存储，`defineEventHandler`接受一个`Ev⁠en⁠t​Na⁠me`和`Callback`并将它们存储在所述的事件存储中。此外，`trigger`接受一个`Ev⁠ent​Na⁠me`，如果注册了回调，则执行每个注册的回调函数，并传递一个`Ev⁠ent​Obj⁠ect`。

第一步完成了。我们现在在定义事件时拥有了类型安全性：

```
const system = new EventSystem();
system.defineEventHandler("click", () => {});
// ^ Argument of type '"click"' is not assignable to parameter
//.  of type '`on${string}`'.(2345)
system.defineEventHandler("onClick", () => {});
system.defineEventHandler("onchange", () => {});
```

在配方 6.2 中，我们将看看如何使用字符串操作类型和键重映射来增强我们的系统。

# 6.2 使用字符串操作类型和键重映射创建事件回调

## 问题

您希望提供一个`watch`函数，该函数接受任何对象并为每个属性添加观察者函数，允许您定义事件回调。

## 解决方案

使用*键重映射*创建新的字符串属性键。使用*字符串操作类型*为观察函数设置适当的驼峰命名。

## 讨论

我们的事件系统在配方 6.1 中逐渐成形。我们能够注册事件处理程序并触发事件。现在我们想要添加观察功能。这个想法是扩展有效对象，使其具有注册回调的方法，每当属性更改时就会执行这些回调。例如，当我们定义一个`person`对象时，我们应该能够监听`onAgeChanged`和`onNameChanged`事件：

```
let person = {
  name: "Stefan",
  age: 40,
};

const watchedPerson = system.watch(person);

watchedPerson.onAgeChanged((ev) => {
  console.log(ev.val, "changed!!");
});

watchedPerson.age = 41; // triggers callbacks
```

因此，对于每个属性，将有一个方法以`on`开头，以`Changed`结尾，并接受带有事件对象参数的回调函数。

为了定义新的事件处理程序方法，我们创建了一个名为 `Wa⁠tch⁠ed​Ob⁠jec⁠t<T>` 的辅助类型，在其中添加定制方法：

```
type WatchedObject<T> = {
  [K in string & keyof T as `on${K}Changed`]: (
    ev: Callback<T[K]>
  ) => void;
};
```

有很多东西需要理清。让我们逐步进行：

1.  我们通过迭代 `T` 的所有键来定义一个*映射类型*。由于我们只关心 `string` 属性键，我们使用交集 `string & keyof T` 来摆脱潜在的符号或数字。

1.  接下来，我们将这个键*重映射*到一个新的字符串，由*字符串模板字面量类型*定义。它以 `on` 开头，然后取自我们映射过程的键 `K`，并附加 `Changed`。

1.  属性 `key` 指向一个接受回调函数的函数。回调函数本身将事件对象作为参数，并通过正确地替换其泛型，确保此事件对象包含我们观察对象的原始类型。这意味着当我们调用 `onAgeChanged` 时，事件对象实际上将包含一个 `number`。

这已经很棒了，但缺少重要的细节。当我们像这样在 `person` 上使用 `WatchedObject` 时，所有生成的事件处理方法在 `on` 后都缺少大写字母。为了解决这个问题，我们可以使用内置的*字符串操作类型*之一来将字符串类型大写：

```
type WatchedObject<T> = {
  [K in string & keyof T as `on${Capitalize<K>}Changed`]: (
    ev: Callback<T[K]>
  ) => void;
};
```

接下来是 `Capitalize`、`Lowercase`、`Uppercase` 和 `Uncapitalize`。如果我们悬停在 `WatchedObject<typeof person>` 上，我们可以看到生成的类型是什么样子的：

```
type WatchedPerson = {
  onNameChanged: (ev: Callback<string>) => void;
  onAgeChanged: (ev: Callback<number>) => void;
};
```

随着我们的类型设置，我们开始实施。首先，我们创建两个辅助函数：

```
function capitalize(inp: string) {
  return inp.charAt(0).toUpperCase() + inp.slice(1);
}

function handlerName(name: string): EventName {
  return `on${capitalize(name)}Changed` as EventName;
}
```

我们需要这两个辅助函数来模仿 TypeScript 重映射和操作字符串的行为。`capitalize` 将字符串的第一个字母更改为大写，并且 `handlerName` 在其前后添加前缀。使用 `handlerName` 我们需要进行一点类型断言，以向 TypeScript 指示类型已更改。通过 JavaScript 中可以转换字符串的多种方法，TypeScript 无法确定这将导致大写版本。

接下来，我们在事件系统中实现 `watch` 功能。我们创建一个通用函数，接受任何对象并返回一个包含原始属性和观察者属性的对象。

为了成功实现在属性更改时触发事件处理程序，我们使用 `Proxy` 对象来拦截 `get` 和 `set` 调用：

```
class EventSystem {
  // cut for brevity
  watch<T extends object>(obj: T): T & WatchedObject<T> {
    const self = this;
    return new Proxy(obj, {
      get(target, property) {
        // (1)
        if (
          typeof property === "string" &&
          property.startsWith("on") &&
          property.endsWith("Changed")
        ) {
          // (2)
          return (cb: Callback) => {
            self.defineEventHandler(property as EventName, cb);
          };
        }
        // (3)
        return target[property as keyof T];
      },
      // set to be done ...
    }) as T & WatchedObject<T>;
  }
}
```

我们想要拦截的 `get` 调用是每当我们访问 `WatchedObject<T>` 的属性时：

+   它们以 `on` 开头并以 `Changed` 结尾。

+   如果是这种情况，我们返回一个接受回调函数的函数。该函数本身通过 `defineEventHandler` 将回调函数添加到事件存储中。

+   在所有其他情况下，我们进行常规的属性访问。

现在，每当我们设置原始对象的值时，我们希望触发存储的事件。这就是为什么我们修改所有 `set` 调用的原因：

```
class EventSystem {
  // ... cut for brevity
  watch<T extends object>(obj: T): T & WatchedObject<T> {
    const self = this;
    return new Proxy(obj, {
      // get from above ...
      set(target, property, value) {
        if (property in target && typeof property === "string") {
          // (1)
          target[property as keyof T] = value;
          // (2)
          self.trigger(handlerName(property), value);
          return true;
        }
        return false;
      },
    }) as T & WatchedObject<T>;
  }
}
```

这个过程如下：

1.  设置值。无论如何，我们都需要更新对象。

1.  调用 `trigger` 函数执行所有已注册的回调。

请注意，我们需要进行一些类型断言来引导 TypeScript 朝正确的方向发展。毕竟，我们正在创建新对象。

就是这样！从头开始尝试示例，看看您的事件系统如何运行：

```
let person = {
  name: "Stefan",
  age: 40,
};

const watchedPerson = system.watch(person);

watchedPerson.onAgeChanged((ev) => {
  console.log(ev.val, "changed!!");
});

watchedPerson.age = 41; // logs "41 changed!!"
```

字符串模板字面类型与字符串操作类型和键重映射允许我们动态创建新对象的类型。这些强大的工具使得使用高级 JavaScript 对象创建更加健壮。

# 6.3 编写格式化函数

## 问题

您想为一个接受格式字符串并用实际值替换占位符的函数创建类型。

## 解决方案

创建一种条件类型，从字符串模板字面类型中推断占位符名称。

## 讨论

您的应用程序通过定义带有花括号占位符的格式字符串来定义格式字符串的方法。第二个参数接受一个带有替换值的对象，因此对于格式字符串中定义的每个占位符，都有一个相应值的属性键：

```
format("Hello {world}. My name is {you}.", {
  world: "World",
  you: "Stefan",
});
```

让我们为此函数创建类型定义，确保您的用户不会忘记添加所需的属性。作为第一步，我们使用一些非常广泛的类型定义函数接口。格式字符串的类型为 `string`，格式化参数在具有字面值的 `string` 键的 `Record` 中。我们首先关注类型；函数体的实现稍后处理：

```
function format(fmtString: string, params: Record<string, any>): string {
  throw "unimplemented";
}
```

作为下一步，我们希望通过添加泛型将函数参数锁定为具体值或字面类型。我们将 `fmtString` 的类型更改为泛型类型 `T`，它是 `string` 的子类型。这使我们仍然可以将字符串传递给函数，但是一旦我们传递字面字符串，我们就可以分析字面类型并查找模式（有关详细信息，请参见 Recipe 4.3）：

```
function format<T extends string>(
  fmtString: T,
  params: Record<string, any>
): string {
  throw "unimplemented";
}
```

现在我们已经确定了 `T`，我们可以将其作为泛型类型 `FormatKeys` 的类型参数传递。这是一种条件类型，用于扫描我们的格式字符串以查找花括号：

```
type FormatKeys<
  T extends string
> = T extends `${string}{${string}}${string}`
  ? T
  : never;
```

在这里，我们检查格式字符串是否：

+   以字符串开头；这也可以是空字符串

+   包含 `{`，跟随任何字符串，跟随 `}`。

+   再次由任何字符串跟随。

这实际上意味着我们检查格式字符串中是否恰好有一个占位符。如果是这样，我们返回整个格式字符串，如果不是，我们返回 `never`：

```
type A = FormatKeys<"Hello {world}">; // "Hello {world}"
type B = FormatKeys<"Hello">; // never
```

`FormatKeys` 可以告诉我们传入的字符串是否是格式字符串，但我们实际上对格式字符串的一个特定部分更感兴趣：即花括号之间的部分。使用 TypeScript 的 `infer` 关键字，我们可以告诉 TypeScript，如果格式字符串匹配此模式，那么获取花括号之间的任何文本类型并放入类型变量中：

```
type FormatKeys<
  T extends string
> = T extends `${string}{${infer Key}}${string}`
  ? Key
  : never;
```

这样，我们可以提取子字符串并根据需要重用它们：

```
type A = FormatKeys<"Hello {world}">; // "world"
type B = FormatKeys<"Hello">; // never
```

真棒！我们提取了第一个占位符名称。现在处理其余部分。由于可能会有后续的占位符，我们取第一个占位符之后的所有内容，并将其存储在名为`Rest`的类型变量中。这个条件总是成立，因为`Rest`要么是空字符串，要么包含我们可以再次分析的实际字符串。

我们取出`Rest`，在`true`分支中调用`FormatKeys<Rest>`，在`Key`的联合类型中：

```
type FormatKeys<
  T extends string
> = T extends `${string}{${infer Key}}${infer Rest}`
  ? Key | FormatKeys<Rest>
  : never;
```

这是一个*递归条件类型*。结果将是占位符的联合，我们可以用作格式化对象的键：

```
type A = FormatKeys<"Hello {world}">; // "world"
type B = FormatKeys<"Hello {world}. I'm {you}.">; // "world" | "you"
type C = FormatKeys<"Hello">; // never
```

现在是时候连接`FormatKeys`了。由于我们已经锁定了`T`，我们可以将其作为参数传递给`FormatKeys`，然后用作`Record`的参数：

```
function format<T extends string>(
  fmtString: T,
  params: Record<FormatKeys<T>, any>
): string {
  throw "unimplemented";
}
```

有了这个，我们的类型都准备好了。来实现吧！实现方式与我们定义的类型完美呼应。我们遍历`params`的所有键，并用括号内的相应值替换所有出现：

```
function format<T extends string>(
  fmtString: T,
  params: Record<FormatKeys<T>, any>
): string {
  let ret: string = fmtString;
  for (let k in params) {
    ret = ret.replaceAll(`{${k}}`, params[k as keyof typeof params]);
  }
  return ret;
}
```

注意两个特定的类型：

+   我们需要使用`string`对`ret`进行注释。`fmtString`是`T`的子类型，因此`ret`也将是`T`。这意味着我们无法更改值，因为`T`的类型将发生变化。通过将其注释为更广泛的`string`类型，帮助我们修改`ret`。

+   我们还需要断言对象键`k`实际上是`params`的一个键。这是一个不幸的解决方案，由于 TypeScript 的一些故障安全机制所致。有关此主题的更多信息，请参阅食谱 9.1。

通过食谱 9.1 中的信息，我们可以重新定义`format`，消除一些类型断言，以达到`format`函数的最终版本：

```
function format<T extends string, K extends Record<FormatKeys<T>, any>>(
  fmtString: T,
  params: K
): string {
  let ret: string = fmtString;
  for (let k in params) {
    ret = ret.replaceAll(`{${k}}`, params[k]);
  }
  return ret;
}
```

能够分割字符串并提取属性键非常强大。全球范围内的 TypeScript 开发人员都使用这种模式来加强类型，例如用于像[Express](https://expressjs.com)这样的 Web 服务器。我们将看到更多如何使用这个工具来获得更好类型的示例。

# 6.4 提取格式参数类型

## Problem

您希望扩展来自食谱 6.3 的格式化函数，使其能够为占位符定义类型。

## Solution

创建一个嵌套的条件类型，并使用类型映射查找类型。

## Discussion

让我们扩展上一课的例子。我们现在不仅想知道所有占位符，还想能够为这些占位符定义一定的类型集合。类型应该是可选的，并在占位符名称后面用冒号表示，类型应为 JavaScript 的基本类型之一。当我们传入类型不正确的值时，我们期望得到类型错误：

```
format("Hello {world:string}. I'm {you}, {age:number} years old.", {
  world: "World",
  age: 40,
  you: "Stefan",
});
```

供参考，让我们看一下来自食谱 6.3 的原始实现：

```
type FormatKeys<
  T extends string
> = T extends `${string}{${infer Key}}${infer Rest}`
  ? Key | FormatKeys<Rest>
  : never;

function format<T extends string>(
  fmtString: T,
  params: Record<FormatKeys<T>, any>
): string {
  let ret: string = fmtString;
  for (let k in params) {
    ret = ret.replace(`{${k}}`, params[k as keyof typeof params]);
  }
  return ret;
}
```

为了实现这一点，我们需要做两件事：

1.  将`params`的类型从`Record<FormatKeys<T>, any>`更改为一个实际对象类型，该对象类型具有与每个属性键相关联的适当类型。

1.  调整`FormatKeys`中的字符串模板字面类型，以便提取原始 JavaScript 类型。

对于第一步，我们引入了一个名为`FormatObj<T>`的新类型。它与`FormatKeys`的工作方式相同，但不是简单返回字符串键，而是将相同的键映射到新对象类型。这要求我们使用交集类型链式递归而不是联合类型（我们在每次递归中添加更多属性），并将打破条件从`never`更改为`{}`。如果我们使用`never`进行交集，整个返回类型变为`never`。这样，我们不会向返回类型添加任何新属性：

```
type FormatObj<
  T extends string
> = T extends `${string}{${infer Key}}${infer Rest}`
  ? { [K in Key]: any } & FormatObj<Rest>
  : {};
```

`FormatObj<T>`的工作方式与`Record<FormatKeys<T>, any>`相同。我们仍然没有提取任何占位符类型，但现在我们可以轻松设置每个占位符的类型，因为我们控制整个对象类型。

作为下一步，我们将在`FormatObj<T>`中更改解析条件，以便还要查找冒号分隔符。如果找到`:`字符，则推断`Type`中的后续字符串文字类型，并将其用作映射键的类型：

```
type FormatObj<
  T extends string
> = T extends `${string}{${infer Key}:${infer Type}}${infer Rest}`
  ? { [K in Key]: Type } & FormatObj<Rest>
  : {};
```

我们非常接近；只有一个警告。我们推断了*字符串*文字类型。这意味着，例如，解析`{age:number}`，`age`的类型将是字面字符串`"number"`。我们需要将此字符串转换为实际类型。我们可以使用另一个条件类型或使用映射类型作为查找：

```
type MapFormatType = {
  string: string;
  number: number;
  boolean: boolean;
  [x: string]: any;
};
```

这样，我们可以简单地检查每个键关联的类型，并为所有其他字符串提供一个出色的备选方案：

```
type A = MapFormatType["string"]; // string
type B = MapFormatType["number"]; // number
type C = MapFormatType["notavailable"]; // any
```

让我们将`MapFormatType`连接到`FormatObj<T>`：

```
type FormatObj<
  T extends string
> = T extends `${string}{${infer Key}:${infer Type}}${infer Rest}`
  ? { [K in Key]: MapFormatType[Type] } & FormatObj<Rest>
  : {};
```

我们快要成功了！现在的问题是我们期望每个占位符也定义类型。我们想要使类型是可选的。但是我们的解析条件明确要求`:`分隔符，因此每个没有定义类型的占位符也不会产生属性。

解决方案是在检查占位符之后*再次*检查类型：

```
type FormatObj<
  T extends string
> = T extends `${string}{${infer Key}}${infer Rest}`
  ? Key extends `${infer KeyPart}:${infer TypePart}`
    ? { [K in KeyPart]: MapFormatType[TypePart] } & FormatObj<Rest>
    : { [K in Key]: any } & FormatObj<Rest>
  : {};
```

类型读取如下：

1.  检查是否有可用的占位符。

1.  如果有可用的占位符，请检查是否有类型注释。如果有，则将键映射到格式类型；否则，将原始键映射到`any`。

1.  在所有其他情况下，返回空对象。

就是这样。我们可以添加一个故障安全保护。与其允许占位符没有类型定义，我们至少可以期望类型实现`toString()`。这确保我们始终获得字符串表示：

```
type FormatObj<
  T extends string
> = T extends `${string}{${infer Key}}${infer Rest}`
  ? Key extends `${infer KeyPart}:${infer TypePart}`
    ? { [K in KeyPart]: MapFormatType[TypePart] } & FormatObj<Rest>
    : { [K in Key]: { toString(): string } } & FormatObj<Rest>
  : {};
```

有了这个，让我们将新类型应用于`format`并更改实现：

```
function format<T extends string, K extends FormatObj<T>>(
  fmtString: T,
  params: K
): string {
  let ret: string = fmtString;
  for (let k in params) {
    let val = `${params[k]}`;
    let searchPattern = new RegExp(`{${k}:?.*?}`, "g");
    ret = ret.replaceAll(searchPattern, val);
  }
  return ret;
}
```

我们利用正则表达式替换可能带有类型注释的名称。在函数内部无需检查类型。在这种情况下，TypeScript 应该足够帮助我们。

我们看到的是，条件类型与字符串模板字面类型以及递归和类型查找等工具的结合，使我们能够用几行代码指定复杂的关系。我们的类型变得更好，我们的代码变得更加健壮，对开发者来说使用这样的 API 是一种享受。

# 6.5 处理递归限制

## 问题

您可以通过精心设计的字符串模板字面类型将任何字符串转换为有效的属性键。使用您设置的辅助类型，您可能会遇到递归限制。

## 解决方案

使用累积技术启用尾调用优化。

## 讨论

TypeScript 的字符串模板字面类型与条件类型结合使用，允许您动态创建新的字符串类型，这些类型可以作为属性键或检查程序的有效字符串。

它们使用递归工作，这意味着就像函数一样，您可以多次调用同一类型，直到达到某个限制。

例如，这种类型 `Trim<T>` 可以去除字符串类型开头和结尾的空格：

```
type Trim<T extends string> =
  T extends ` ${infer X}` ? Trim<X> :
  T extends `${infer X} ` ? Trim<X> :
  T;
```

它检查是否有起始空白，推断其余部分，并再次进行相同的检查。一旦所有起始空格都消失，相同的检查将发生在末尾的空格上。一旦起始和末尾的所有空格都消失，它就完成了，并跳到最后一个分支——返回剩余的字符串：

```
type Trimmed = Trim<"     key   ">; // "key"
```

调用类型多次就是递归，并且像这样写是合理的。TypeScript 可以从类型中看出递归调用是独立的，并且可以将其作为尾调用优化来评估，这意味着它可以在同一个调用堆栈帧内评估递归的下一步。

###### 注意

如果您想了解更多有关 JavaScript 中调用堆栈的信息，Thomas Hunter 的书籍 [*使用 Node.js 进行分布式系统*](https://learning.oreilly.com/library/view/distributed-systems-with/9781492077282)（O’Reilly 出版）提供了很好的介绍。

我们想要利用 TypeScript 的特性，通过递归调用条件类型，从任何字符串中创建一个有效的字符串标识符，方法是去除空白和无效字符。

首先，我们写一个类似于 `Trim<T>` 的辅助类型，它去掉找到的所有空格：

```
type RemoveWhiteSpace<T extends string> = T extends `${infer A} ${infer B}`
  ? RemoveWhiteSpace<`${Uncapitalize<A>}${Capitalize<B>}`>
  : T;
```

它检查是否有空白，推断空格前后的字符串（可以是空字符串），然后使用新形成的字符串类型再次调用相同的类型。它还将第一个推断小写化，并将第二个推断大写化，以创建类似驼峰命名的字符串标识符。

它一直这样做，直到所有空格都消失：

```
type Identifier = RemoveWhiteSpace<"Hello World!">; // "helloWorld!"
```

接下来，我们想要检查剩余字符是否有效。我们再次使用递归，将有效字符的字符串拆分为只包含一个字符的单字符串类型，并创建大写和小写版本：

```
type StringSplit<T extends string> = T extends `${infer Char}${infer Rest}`
  ? Capitalize<Char> | Uncapitalize<Char> | StringSplit<Rest>
  : never;

type Chars = StringSplit<"abcdefghijklmnopqrstuvwxyz">;
//  "a" | "A" | "b" | "B" | "c" | "C" | "d" | "D" | "e" | "E" |
//  "f" | "F" | "g" | "G" | "h" | "H" | "i" | "I" | "j" | "J" |
//  "k" | "K" | "l" | "L" | "m" | "M" | "n" | "N" | "o" | "O" |
//  "p" | "P" | "q" | "Q" | "r" | "R" | "s" | "S" | "t" | "T" |
//  "u" | "U" | "v" | "V" | "w" | "W" | "x" | "X" | "y" | "Y" |
//  "z" | "Z"
```

我们刮掉我们找到的第一个字符，将其大写，将其小写，然后对其余部分执行相同操作，直到没有更多的字符串为止。请注意，由于我们将递归调用放在联合类型中的结果中，这种递归无法进行尾调用优化。当我们达到 50 个字符时（TypeScript 编译器的硬限制），我们将达到递归限制。对于基本字符，我们没有问题！

但是当我们进行下一步，创建`Identifier`时，我们首先要检查有效字符。首先，我们调用`RemoveWhiteSpace<T>`类型，这允许我们摆脱空格并将其余部分改为驼峰式。然后我们检查结果是否符合有效字符。

就像在`StringSplit<T>`中一样，我们删掉了第一个字符，但在推断中进行了另一种类型检查。我们看看刚刚刮掉的字符是否是有效字符之一。然后我们获取剩下的部分。我们再次组合相同的字符串，但在剩余字符串上进行递归检查。如果第一个字符无效，则我们调用`Cr⁠ea⁠te​Id⁠en⁠ti⁠fie⁠r<T>`处理剩余部分：

```
type CreateIdentifier<T extends string> =
  RemoveWhiteSpace<T> extends `${infer A extends Chars}${infer Rest}`
  ? `${A}${CreateIdentifier<Rest>}`
//  ^ Type instantiation is excessively deep and possibly infinite.(2589)_.
  : RemoveWhiteSpace<T> extends `${infer A}${infer Rest}`
  ? CreateIdentifier<Rest>
  : T;
```

这里我们首次触及递归限制。TypeScript 通过错误警告我们，这种类型实例化可能是无限的且过于深层。似乎如果我们在字符串模板文字类型内使用递归调用，可能会导致调用栈错误并引发故障。因此，TypeScript 中断了。它无法在这里进行尾调用优化。

###### 注意

`CreateIdentifier<T>`可能仍然会生成正确的结果，即使在编写类型时 TypeScript 出错。这些是难以察觉的错误，因为它们可能在您不期望时出现。确保在错误发生时不要让 TypeScript 生成任何结果。

有一种解决方法。为了激活尾调用优化，递归调用需要独立存在。我们可以通过使用所谓的*累加器技术*来实现这一点。在这里，我们传递第二个类型参数称为`Acc`，它是`string`类型，并用空字符串进行实例化。我们将其用作累加器，存储中间结果，并一遍又一遍地将其传递给下一个调用：

```
type CreateIdentifier<T extends string, Acc extends string = ""> =
  RemoveWhiteSpace<T> extends `${infer A extends Chars}${infer Rest}`
  ? CreateIdentifier<Rest, `${Acc}${A}`>
  : RemoveWhiteSpace<T> extends `${infer A}${infer Rest}`
  ? CreateIdentifier<Rest, Acc>
  : Acc;
```

这样，递归调用再次独立存在，并且结果是第二个参数。当我们完成递归调用时，递归断开的分支，我们返回累加器，因为它是我们的最终结果：

```
type Identifier = CreateIdentifier<"Hello Wor!ld!">; // "helloWorld"
```

可能有更聪明的方法从任意字符串生成标识符，但请注意，在使用递归的复杂条件类型中，可能会遇到相同的问题。累加器技术是减轻此类问题的好方法。

# 6.6 使用字符串模板文字作为判别因子

## 问题

你的模型将对后端的请求作为状态机处理，从*pending*状态转换为*error*或*success*状态。这些状态应适用于不同的后端请求，但底层类型应该是相同的。

## 解决方案

使用字符串模板文字作为判别联合类型的判别因子。

## 讨论

从后端获取数据的方式始终遵循相同的结构。您发出请求，它可能是待处理的，要么实现并返回一些数据（成功），要么拒绝并返回错误。例如，要登录用户，所有可能的状态可能如下所示：

```
type UserRequest =
  | {
      state: "USER_PENDING";
    }
  | {
      state: "USER_ERROR";
      message: string;
    }
  | {
      state: "USER_SUCCESS";
      data: User;
    };
```

当我们获取用户订单时，我们可以使用相同的状态。唯一的区别在于成功载荷和每个状态的名称，这些名称根据请求类型进行了定制：

```
type OrderRequest =
  | {
      state: "ORDER_PENDING";
    }
  | {
      state: "ORDER_ERROR";
      message: string;
    }
  | {
      state: "ORDER_SUCCESS";
      data: Order;
    };
```

当我们处理全局状态处理机制，比如[Redux](https://redux.js.org)，我们希望通过像这样的标识符进行区分。我们仍然希望将其缩小到相应的状态类型！

TypeScript 允许您创建有区别的联合类型，其中辨别器是字符串模板文字类型。因此，我们可以使用相同的模式总结所有可能的后端请求：

```
type Pending = {
  state: `${Uppercase<string>}_PENDING`;
};

type Err = {
  state: `${Uppercase<string>}_ERROR`;
  message: string;
};

type Success = {
  state: `${Uppercase<string>}_SUCCESS`;
  data: any;
};

type BackendRequest = Pending | Err | Success;
```

这已经给了我们一个优势。我们知道每个联合类型成员的状态属性需要以大写字符串开头，后跟下划线和相应的状态字符串。我们可以像往常一样缩小到子类型：

```
function execute(req: BackendRequest) {
  switch (req.state) {
    case "USER_PENDING":
      // req: Pending
      console.log("Login pending...");
      break;
    case "USER_ERROR":
      // req: Err
      throw new Error(`Login failed: ${req.message}`);
    case "USER_SUCCESS":
      // req: Success
      login(req.data);
      break;
    case "ORDER_PENDING":
      // req: Pending
      console.log("Fetching orders pending");
      break;
    case "ORDER_ERROR":
      // req: Err
      throw new Error(`Fetching orders failed: ${req.message}`);
    case "ORDER_SUCCESS":
      // req: Success
      displayOrder(req.data);
      break;
  }
}
```

将整个字符串集作为辨别器的第一部分可能有点过于复杂。我们可以缩小到各种已知请求，并使用字符串操作类型来获取正确的子类型：

```
type RequestConstants = "user" | "order";

type Pending = {
  state: `${Uppercase<RequestConstants>}_PENDING`;
};

type Err = {
  state: `${Uppercase<RequestConstants>}_ERROR`;
  message: string;
};

type Success = {
  state: `${Uppercase<RequestConstants>}_SUCCESS`;
  data: any;
};
```

这就是摆脱拼写错误的方法！更好的是，假设我们将所有数据存储在类型为`Data`的全局状态对象中。我们可以从这里派生所有可能的`BackendRequest`类型。通过使用`keyof Data`，我们获得组成`BackendRequest`状态的字符串键：

```
type Data = {
  user: User | null;
  order: Order | null;
};

type RequestConstants = keyof Data;

type Pending = {
  state: `${Uppercase<RequestConstants>}_PENDING`;
};

type Err = {
  state: `${Uppercase<RequestConstants>}_ERROR`;
  message: string;
};
```

这已经对`Pending`和`Err`很有效，但在`Success`情况下，我们希望具有与`"user"`或`"order"`关联的实际数据类型。

第一种选择是使用索引访问从`Data`中获取`data`属性的正确类型：

```
type Success = {
  state: `${Uppercase<RequestConstants>}_SUCCESS`;
  data: NonNullable<Data[RequestConstants]>;
};
```

###### 提示

`NonNullable<T>`消除联合类型中的`null`和`undefined`。启用编译器标志`strictNullChecks`后，所有类型都排除了`null`和`undefined`。这意味着如果存在空值状态，则需要手动添加它们，并在想要确保它们不包含时手动排除它们。

但这意味着`data`对于所有后端请求可以是`User`或`Order`，如果添加新的请求，则更多。为了避免断开标识符与其关联的数据类型之间的连接，我们通过所有`RequestConstants`进行映射，创建状态对象，然后再次使用`RequestConstants`的索引访问来生成联合类型：

```
type Success = {
  [K in RequestConstants]: {
    state: `${Uppercase<K>}_SUCCESS`;
    data: NonNullable<Data[K]>;
  };
}[RequestConstants];
```

`Success`现在等于手动创建的联合类型：

```
type Success = {
    state: "USER_SUCCESS";
    data: User;
} | {
    state: "ORDER_SUCCESS";
    data: Order;
};
```
