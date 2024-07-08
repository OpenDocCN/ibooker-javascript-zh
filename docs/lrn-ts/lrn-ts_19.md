# 第十五章：类型操作

> 条件语句，映射
> 
> 拥有强大的类型控制能力
> 
> 伴随着巨大的混淆而来

TypeScript 在类型系统中为我们提供了强大的类型定义能力。即使是来自第十章，“泛型”的逻辑修饰符，也无法与本章中类型操作的能力相比。完成本章后，您将能够基于其他类型混合、匹配和修改类型，从而为类型系统提供强大的表示方式。

###### 警告

这些花里胡哨的类型技术通常不是您经常使用的技术。您需要理解它们在有用情况下的用法，但请注意：如果过度使用，它们可能会难以阅读。祝您愉快！

# 映射类型

TypeScript 提供了一种语法，用于基于另一个类型的属性创建一个新类型：换句话说，*映射*从一个类型到另一个类型。在 TypeScript 中，*映射类型*是一种接受另一个类型并对该类型的每个属性执行某些操作的类型。

映射类型通过在一组键的每个键下创建一个新属性来创建一个新类型。它们使用类似于索引签名的语法，但是不是使用像`[i: string]`那样的静态键类型，而是使用与`in`一起的来自其他类型的计算类型，比如`[K in OriginalType]`：

```
type NewType = {
    [K in OriginalType]: NewProperty;
};
```

映射类型的一个常见用例是创建一个对象，其键是现有联合类型中的每个字符串字面量。这个`AnimalCounts`类型创建了一个新的对象类型，其中键是`Animals`联合类型中的每个值，而每个值都是`number`：

```
type Animals = "alligator" | "baboon" | "cat";

type AnimalCounts = {
    [K in Animals]: number;
};
// Equivalent to:
// {
//   alligator: number;
//   baboon: number;
//   cat: number;
// }
```

基于联合类型现有文字的映射类型是声明大型接口时节省空间的便捷方式。但是映射类型真正发光的时候，是它们能够作用于其他类型，甚至添加或删除成员修饰符时。

## 从类型映射类型

映射类型通常使用`keyof`运算符对现有类型进行操作，以抓取该现有类型的键。通过指示一个类型在现有类型的键上进行映射，我们可以从该现有类型映射到一个新类型。

这个`AnimalCounts`类型最终与之前的`AnimalCounts`类型相同，通过从`AnimalVariants`类型映射到一个新的等效类型：

```
interface AnimalVariants {
    alligator: boolean;
    baboon: number;
    cat: string;
}

type AnimalCounts = {
    [K in keyof AnimalVariants]: number;
};
// Equivalent to:
// {
//   alligator: number;
//   baboon: number;
//   cat: number;
// }
```

在前面的片段中，`keyof`命名为`K`的新类型键已知是原始类型的键。这意味着每个映射类型成员值都允许引用原始类型相同键下的成员值。

如果原始对象是`SomeName`，映射是`[K in keyof SomeName]`，那么映射类型中的每个成员都可以引用等效的`SomeName`成员值，作为`SomeName[K]`。

这个`NullableBirdVariants`类型接受一个原始的`BirdVariants`类型，并为每个成员添加`| null`：

```
interface BirdVariants {
    dove: string;
    eagle: boolean;
}

type NullableBirdVariants = {
    [K in keyof BirdVariants]: BirdVariants[K] | null,
};
// Equivalent to:
// {
//   dove: string | null;
//   eagle: boolean | null;
// }
```

与手动复制原始类型的每个字段相比，映射类型允许您一次定义一组成员，并根据需要批量重新创建它们的新版本。

### 映射类型和签名

在 第七章，“接口” 中，我介绍了 TypeScript 提供的两种将接口成员声明为函数的方式：

+   *Method* 语法，如 `member(): void`：声明接口成员是一个函数，预期作为对象的成员进行调用

+   *Property* 语法，如 `member: () => void`：声明接口成员等同于一个独立的函数

Mapped types don’t distinguish between method and property syntaxes on object types. Mapped types treat methods as properties on original types.

这种 `ResearcherProperties` 类型包含了 `Researcher` 的 `property` 和 `method` 成员：

```
interface Researcher {
    researchMethod(): void;
    researchProperty: () => string;
}

type JustProperties<T> = {
    [K in keyof T]: T[K];
};

type ResearcherProperties = JustProperties<Researcher>;
// Equivalent to:
// {
//   researchMethod: () => void;
//   researchProperty: () => string;
// }
```

在大多数实际的 TypeScript 代码中，方法和属性之间的区别并不常见。很少能找到一个将类类型作为输入的映射类型的实际用途。

## 修改修饰符

映射类型还可以改变原始类型成员的访问控制修饰符——`readonly` 和 `?` 可选性。可以使用与典型接口相同的语法将 `readonly` 或 `?` 放在映射类型的成员上。

下面的 `ReadonlyEnvironmentalist` 类型创建了一个带有所有成员 `readonly` 的 `Environmentalist` 接口的版本，而 `OptionalReadonlyConservationist` 更进一步，生成另一个版本，并对所有 `ReadonlyEnvironmentalist` 成员添加了 `?`：

```
interface Environmentalist {
    area: string;
    name: string;
}

type ReadonlyEnvironmentalist = {
    readonly [K in keyof Environmentalist]: Environmentalist[K];
};
// Equivalent to:
// {
//   readonly area: string;
//   readonly name: string;
// }

type OptionalReadonlyEnvironmentalist = {
    [K in keyof ReadonlyEnvironmentalist]?: ReadonlyEnvironmentalist[K];
};
// Equivalent to:
// {
//   readonly area?: string;
//   readonly name?: string;
// }
```

###### 注意

`OptionalReadonlyEnvironmentalist` 类型可以使用 `readonly [K in keyof Environmentalist]?: Environmentalist[K]` 的方式进行替代。

通过在新类型中修饰符之前添加 `-` 来删除修饰符。可以分别使用 `-readonly` 或 `-?:` 来代替 `readonly` 或 `?:`。

这种 `Conservationist` 类型包含了一些 `?` 可选和/或 `readonly` 成员，在 `WritableConservationist` 中这些成员变为可写，并在 `RequiredWritableConservationist` 中也被要求：

```
interface Conservationist {
    name: string;
    catchphrase?: string;
    readonly born: number;
    readonly died?: number;
}

type WritableConservationist = {
    -readonly [K in keyof Conservationist]: Conservationist[K];
};
// Equivalent to:
// {
//   name: string;
//   catchphrase?: string;
//   born: number;
//   died?: number;
// }

type RequiredWritableConservationist = {
    [K in keyof WritableConservationist]-?: WritableConservationist[K];
};
// Equivalent to:
// {
//   name: string;
//   catchphrase: string;
//   born: number;
//   died: number;
// }
```

###### 注意

另一种写法是使用 `-readonly [K in keyof Conservationist]-?:` `Conservationist[K]` 来替代 `RequiredWritableConservationist` 类型。

## 通用映射类型

映射类型的全部威力来自与泛型的结合，允许在不同类型之间重复使用单一映射类型。映射类型能够访问其范围内的任何类型名称的 `keyof`，包括映射类型本身的类型参数。

泛型映射类型经常用于表示数据在应用程序中流动时的变形方式。例如，可能希望应用程序的某个区域能够接收现有类型的值，但不允许修改数据。

这种 `MakeReadonly` 泛型类型接收任何类型，并创建一个在其所有成员上添加 `readonly` 修饰符的新版本。

```
type MakeReadonly<T> = {
    readonly [K in keyof T]: T[K];
}

interface Species {
    genus: string;
    name: string;
}

type ReadonlySpecies = MakeReadonly<Species>;
// Equivalent to:
// {
//   readonly genus: string;
//   readonly name: string;
// }
```

另一个开发人员通常需要表示的转换是接受接口的任意数量并返回该接口的完全填充实例的函数。

下面的`MakeOptional`类型和`createGenusData`函数允许提供任意数量的`GenusData`接口，并返回一个填充了默认值的对象：

```
interface GenusData {
    family: string;
    name: string;
}

type MakeOptional<T> = {
    [K in keyof T]?: T[K];
}
// Equivalent to:
// {
//   family?: string;
//   name?: string;
// }

/**
 * Spreads any {overrides} on top of default values for GenusData.
 */
function createGenusData(overrides?: MakeOptional<GenusData>): GenusData {
    return {
        family: 'unknown',
        name: 'unknown',
        ...overrides,
    }
}
```

一些由泛型映射类型完成的操作非常有用，TypeScript 提供了它们的实用类型。例如，使用内置的`Partial<T>`类型可以使所有属性变为可选。您可以在[*https://www.typescriptlang.org/docs/handbook/utility-types.html*](https://www.typescriptlang.org/docs/handbook/utility-types.html)上找到这些内置类型的列表。

# 条件类型

映射现有类型到其他类型是很巧妙的，但是我们尚未将逻辑条件加入类型系统。现在让我们来做这件事。

TypeScript 的类型系统是*逻辑编程语言*的一个例子。它允许基于逻辑检查先前的类型创建新的结构（类型）。它使用*条件类型*的概念来实现：一种根据现有类型解析为两种可能类型之一的类型。

条件类型的语法看起来像三元运算符：

```
LeftType extends RightType ? IfTrue : IfFalse
```

条件类型中的逻辑检查始终是左侧类型*是否扩展*或可分配给右侧类型。

下面的`CheckStringAgainstNumber`条件类型检查`string`是否*扩展*到`number`，换句话说，`string`类型是否可分配给`number`类型。它不行，因此结果类型是“如果为假”的情况：`false`：

```
// Type: false
type CheckStringAgainstNumber = string extends number ? true : false;
```

本章的大部分内容将涉及将其他类型系统特性与条件类型结合使用。随着代码片段变得更加复杂，请记住：每个条件类型都仅仅是布尔逻辑的一部分。每个条件类型接受某种类型并产生两种可能的结果之一。

## 泛型条件类型

条件类型能够检查其范围内的任何类型名称，包括条件类型本身的类型参数。这意味着您可以编写可重用的泛型类型来基于任何其他类型创建新类型。

将之前的`CheckStringAgainstNumber`类型转换为通用的`CheckAgainstNumber`类型，得到的类型是基于先前的类型是否可分配给`number`的`true`或`false`。`string`仍然是 false，而`number`和`0 | 1`则都是 true。

```
type CheckAgainstNumber<T> = T extends number ? true : false;

// Type: false
type CheckString = CheckAgainstNumber<'parakeet'>;

// Type: true
type CheckString = CheckAgainstNumber<1891>;

// Type: true
type CheckString = CheckAgainstNumber<number>;
```

下面的`CallableSetting`类型更实用一些。它接受一个泛型`T`并检查`T`是否为函数。如果是，则结果类型为`T`，就像`GetNumbersSetting`中`T`为`() => number[]`一样。否则，结果类型是返回`T`的函数，就像`StringSetting`中`T`为`string`一样，因此结果类型是`() => string`：

```
type CallableSetting<T> =
    T extends () => any
        ? T
        : () => T

// Type: () => number[]
type GetNumbersSetting = CallableSetting<() => number[]>;

// Type: () => string
type StringSetting = CallableSetting<string>;
```

条件类型也能够使用对象成员查找语法访问提供的类型的成员。它们可以在其`extends`子句中和/或在结果类型中使用这些信息。

JavaScript 库经常使用的一种模式非常适合条件泛型类型，即根据提供给函数的选项对象改变函数的返回类型。

例如，许多数据库函数或等效函数可能使用像`throwIfNotFound`这样的属性，通过更改函数在值未找到时抛出错误而不是返回`undefined`。以下的`QueryResult`类型通过特定情况下选项的`throwIfNotFound`知道是`true`，模拟了该行为，结果更窄的是`string`而不是`string | undefined`：

```
interface QueryOptions {
  throwIfNotFound: boolean;
}

type QueryResult<Options extends QueryOptions> =
  Options["throwIfNotFound"] extends true ? string : string | undefined;

declare function retrieve<Options extends QueryOptions>(
    key: string,
    options?: Options,
): Promise<QueryResult<Options>>;

// Returned type: string | undefined
await retrieve("Biruté Galdikas");

// Returned type: string | undefined
await retrieve("Jane Goodall", { throwIfNotFound: Math.random() > 0.5 });

// Returned type: string
await retrieve("Dian Fossey", { throwIfNotFound: true });
```

通过将条件类型与泛型类型参数结合，`retrieve`函数更精确地告诉类型系统它将如何改变程序的控制流。

## 类型分布性

条件类型在联合类型上*分布*，这意味着它们的结果类型将是将该条件类型应用于联合类型中的每个成员（类型在联合类型中）。换句话说，`ConditionalType<T | U>`与`Conditional<T> | Conditional<U>`是相同的。

类型分布性是一个解释起来比较啰嗦但对于条件类型在联合类型中的行为非常重要的概念。

考虑以下`ArrayifyUnlessString`类型，它将其类型参数`T`转换为一个数组，除非`T`扩展为`string`。`HalfArrayified`等同于`string | number[]`，因为`ArrayifyUnlessString<string | number>`与`ArrayifyUnlessString<string> | ArrayifyUnlessString<number>`相同：

```
type ArrayifyUnlessString<T> = T extends string ? T : T[];

// Type: string | number[]
type HalfArrayified = ArrayifyUnlessString<string | number>;
```

如果 TypeScript 的条件类型不能在联合类型中分布，`HalfArrayified`将会是`(string | number)[]`，因为`string | number`不能赋值给`string`。换句话说，条件类型将其逻辑应用于联合类型的每个成员，而不是整个联合类型。

## 推断类型

访问提供类型的成员对作为类型成员存储的信息很有效，但无法捕获其他信息，如函数参数或返回类型。条件类型能够通过在其扩展子句中使用`infer`关键字来访问其条件的任意部分。在扩展子句中放置`infer`关键字和一个新类型的名称意味着新类型将在条件类型的真实情况中可用。

这个`ArrayItems`类型接受一个类型参数`T`，并检查`T`是否是某种新的`Item`类型的数组。如果是，结果类型就是`Item`；如果不是，就是`T`：

```
type ArrayItems<T> =
    T extends (infer Item)[]
        ? Item
        : T;

// Type: string
type StringItem = ArrayItems<string>;

// Type: string
type StringArrayItem = ArrayItems<string[]>;

// Type: string[]
type String2DItem = ArrayItems<string[][]>;
```

推断类型可以用来创建递归条件类型。之前见过的`ArrayItems`类型可以扩展到递归地检索任意维度数组的项类型：

```
type ArrayItemsRecursive<T> =
    T extends (infer Item)[]
        ? ArrayItemsRecursive<Item>
        : T;

// Type: string
type StringItem = ArrayItemsRecursive<string>;

// Type: string
type StringArrayItem = ArrayItemsRecursive<string[]>;

// Type: string
type String2DItem = ArrayItemsRecursive<string[][]>;
```

注意，尽管`ArrayItems<string[][]>`的结果是`string[]`，但`ArrayItemsRecursive<string[][]>`的结果是`string`。泛型类型能够递归的能力使它们能够持续应用修改，比如在此处检索数组元素类型。

## 映射的条件类型

映射类型将对现有类型的每个成员应用更改。条件类型将对单个现有类型应用更改。将它们组合在一起，允许对泛型模板类型的每个成员应用条件逻辑。

此 `MakeAllMembersFunctions` 类型将类型的每个非函数成员转换为函数：

```
type MakeAllMembersFunctions<T> = {
    [K in keyof T]: T[K] extends (...args: any[]) => any
        ? T[K]
        : () => T[K]
};

type MemberFunctions = MakeAllMembersFunctions<{
    alreadyFunction: () => string,
    notYetFunction: number,
}>;
// Type:
// {
//   alreadyFunction: () => string,
//   notYetFunction: () => number,
// }
```

映射条件类型是一种方便的方法，用于使用一些逻辑检查修改现有类型的所有属性。

# never

在第四章，“对象”中，我介绍了 `never` 类型，即底部类型，这意味着它不能有可能的值，也不能被访问。在类型系统以及之前的运行时代码示例中添加 `never` 类型注释的合适位置可以告诉 TypeScript 更积极地检测到不可达的代码路径。

## never 和交集和并集

描述 `never` 底部类型的另一种方式是它是一个无法存在的类型。这使得 `never` 在与 `&` 交集和 `|` 联合类型一起具有一些有趣的行为：

+   在 `&` 交集类型中的 `never` 将交集类型减少到 `never`。

+   在 `|` 联合类型中的 `never` 被忽略。

这些 `NeverIntersection` 和 `NeverUnion` 类型说明了这些行为：

```
type NeverIntersection = never & string; // Type: never
type NeverUnion = never | string; // Type: string
```

特别是在联合类型中被忽略的行为使得 `never` 对从条件类型和映射类型中过滤值非常有用。

## never 和条件类型

泛型条件类型通常使用 `never` 从联合类型中过滤出类型。因为在联合中忽略 `never`，所以对于类型联合的泛型条件的结果将只包括那些不是 `never` 的类型。

此 `OnlyStrings` 泛型条件类型可过滤掉不是字符串的类型，因此 `RedOrBlue` 类型可从联合中过滤掉 `0` 和 `null`：

```
type OnlyStrings<T> = T extends string ? T : never;

type RedOrBlue = OnlyStrings<"red" | "blue" | 0 | false>;
// Equivalent to: "red" | "blue"
```

当为泛型类型制作类型工具时，`never` 通常与推断的条件类型结合使用。使用 `infer` 进行类型推断时必须位于条件类型的真实情况下，因此如果假设情况永远不会被使用，那么 `never` 就是适合放置在那里的类型。

此 `FirstParameter` 类型接受一个函数类型 `T`，检查它是否为带有 `arg: infer Arg` 的函数，并在是的情况下返回该 `Arg`：

```
type FirstParameter<T extends (...args: any[]) => any> =
    T extends (arg: infer Arg) => any
        ? Arg
        : never;

type GetsString = FirstParameter<
    (arg0: string) => void
>; // Type: string
```

在条件类型的假设情况中使用 `never` 允许 `FirstParameter` 提取函数第一个参数的类型。

## never 和映射类型

联合类型中的 `never` 行为也使其对在映射类型中过滤成员非常有用。通过以下三种类型系统特性可以过滤对象的键：

+   在联合类型中忽略 `never`。

+   映射类型可以映射类型的成员。

+   条件类型可用于在满足条件时将类型转换为 `never`。

将这三者组合在一起，我们可以创建一个映射类型，将原始类型的每个成员更改为原始键或 `never`。然后使用 `[keyof T]` 来获取该类型的成员，从而生成所有这些映射类型结果的联合，过滤掉 `never`。

下面的`OnlyStringProperties`类型将每个`T[K]`成员转换为如果该成员是字符串，则为`K`键，否则为`never`：

```
type OnlyStringProperties<T> = {
  [K in keyof T]: T[K] extends string ? K : never;
}[keyof T];

interface AllEventData {
    participants: string[];
    location: string;
    name: string;
    year: number;
}

type OnlyStringEventData = OnlyStringProperties<AllEventData>;
// Equivalent to: "location" | "name"
```

另一种解读`OnlyStringProperties<T>`类型的方法是过滤掉所有非`string`属性（将它们转换为`never`），然后返回所有剩余的键（`[keyof T]`）。

# 模板字面类型

我们已经讨论了很多关于条件和/或映射类型的内容。现在让我们转向逻辑较少的类型，专注于字符串一段时间。到目前为止，我提到了两种为字符串值编写类型的策略：

+   原始的`string`类型：当值可以是世界上的任何字符串时使用

+   字符串字面类型如`""`和`"abc"`：当值只能是这一种类型（或其联合体）时使用

然而，有时您可能希望指示字符串匹配某些字符串模式：部分字符串已知，但部分字符串未知。输入*模板字面类型*，一种 TypeScript 语法，用于指示字符串类型符合模式。它们看起来像模板字面字符串一样——因此它们的名字——但插入了原始类型或原始类型的联合体。

此模板字面类型指示字符串必须以`"Hello"`开头，但可以以任意字符串（`string`）结尾。以`"Hello"`开头的名称（如`"Hello, world!"`）匹配，但不匹配`"World! Hello!"`或`"hi"`：

```
type Greeting = `Hello${string}`;

let matches: Greeting = "Hello, world!"; // Ok

let outOfOrder: Greeting = "World! Hello!";
//  ~~~~~~~~~~
// Error: Type '"World! Hello!"' is not assignable to type '`Hello ${string}`'.

let missingAltogether: Greeting = "hi";
//  ~~~~~~~~~~~~~~~~~
// Error: Type '"hi"' is not assignable to type '`Hello ${string}`'.
```

字符串字面类型及其联合体可以在类型插值中代替通用的`string`原始类型，以限制模板字面类型匹配更窄的字符串模式。模板字面类型非常适合描述必须匹配受限制的允许字符串集合的字符串。

在这里，`BrightnessAndColor`仅匹配以`Brightness`开头、以`Color`结尾且中间有`-`连字符的字符串：

```
type Brightness = "dark" | "light";
type Color =  "blue" | "red";

type BrightnessAndColor = `${Brightness}-${Color}`;
// Equivalent to: "dark-red" | "light-red" | "dark-blue" | "light-blue"

let colorOk: BrightnessAndColor = "dark-blue"; // Ok

let colorWrongStart: BrightnessAndColor = "medium-blue";
//  ~~~~~~~~~~~~~~~
// Error: Type '"medium-blue"' is not assignable to type
// '"dark-blue" | "dark-red" | "light-blue" | "light-red"'.

let colorWrongEnd: BrightnessAndColor = "light-green";
//  ~~~~~~~~~~~~~
// Error: Type '"light-green"' is not assignable to type
// '"dark-blue" | "dark-red" | "light-blue" | "light-red"'.
```

如果没有模板字面类型，我们将不得不费力地写出所有`Brightness`和`Color`的四种组合。如果我们向其中任何一个添加更多的字符串字面类型，这将变得很繁琐！

TypeScript 允许模板字面类型包含任何原始类型（除了`symbol`）或其联合体：`string`、`number`、`bigint`、`boolean`、`null`或`undefined`。

此`ExtolNumber`类型允许任何以`"much "`开头、包含看起来像数字的字符串，并以`"wow"`结尾的字符串：

```
type ExtolNumber = `much ${number} wow`;

function extol(extolee: ExtolNumber) { /* ... */ }

extol('much 0 wow'); // Ok
extol('much -7 wow'); // Ok
extol('much 9.001 wow'); // Ok

extol('much false wow');
//    ~~~~~~~~~~~~~~~~
// Error: Argument of type '"much false wow"' is not
// assignable to parameter of type '`much ${number} wow`'.
```

## 内在的字符串操作类型

为了帮助处理字符串类型，TypeScript 提供了一小组内置的通用实用类型，它们接受一个字符串并对该字符串应用某些操作。截至 TypeScript 4.7.2，有四种类型：

+   `Uppercase`：将字符串字面类型转换为大写。

+   `Lowercase`：将字符串字面类型转换为小写。

+   `Capitalize`：将字符串字面类型的首字母转换为大写。

+   `Uncapitalize`：将字符串字面类型的首字母转换为小写。

这些都可以作为接受字符串的泛型类型使用。例如，使用 `Capitalize` 来将字符串的第一个字母大写：

```
type FormalGreeting = Capitalize<"hello.">; // Type: "Hello."
```

这些内置字符串操作类型在操作对象类型的属性键时非常有用。

## 模板文字键

模板文字类型介于原始的 `string` 和字符串文字之间，这意味着它们仍然是字符串。它们可以在任何其他可以使用字符串文字的地方使用。

例如，您可以将它们用作映射类型中的索引签名。此 `ExistenceChecks` 类型在 `DataKey` 中的每个字符串都有一个键，使用 `check${Capitalize<DataKey>}` 进行映射：

```
type DataKey = "location" | "name" | "year";

type ExistenceChecks = {
    [K in `check${Capitalize<DataKey>}`]: () => boolean;
};
// Equivalent to:
// {
//   checkLocation: () => boolean;
//   checkName: () => boolean;
//   checkYear: () => boolean;
// }

function checkExistence(checks: ExistenceChecks) {
    checks.checkLocation(); // Type: boolean
    checks.checkName(); // Type: boolean

    checks.checkWrong();
    //     ~~~~~~~~~~
    // Error: Property 'checkWrong' does not exist on type 'ExistenceChecks'.
}
```

## 重映射映射类型键

TypeScript 允许您基于原始成员使用模板文字类型为映射类型的成员创建新的键。在映射类型中使用 `as` 关键字，后跟模板文字类型作为索引签名，可以将结果类型的键修改为与模板文字类型匹配。这样做允许映射类型为每个映射属性具有不同的键，同时仍然引用原始值。

在这里，`DataEntryGetters` 是一种映射类型，其键是 `getLocation`、`getName` 和 `getYear`。每个键都映射到一个具有模板文字类型的新键。每个映射值是一个函数，其返回类型是使用原始 `K` 键作为类型参数的 `DataEntry`：

```
interface DataEntry<T> {
    key: T;
    value: string;
}

type DataKey = "location" | "name" | "year";

type DataEntryGetters = {
    [K in DataKey as `get${Capitalize<K>}`]: () => DataEntry<K>;
};
// Equivalent to:
// {
//   getLocation: () => DataEntry<"location">;
//   getName: () => DataEntry<"name">;
//   getYear: () => DataEntry<"year">;
// }
```

键重映可以与其他类型操作结合使用，以创建基于现有类型形状的映射类型。一个有趣的组合是在现有对象上使用 `keyof typeof` 来创建该对象类型的映射类型。

此 `ConfigGetter` 类型基于 `config` 类型，但每个字段都是返回原始配置的函数，并且键已从原始键修改：

```
const config = {
    location: "unknown",
    name: "anonymous",
    year: 0,
};

type LazyValues = {
    [K in keyof typeof config as `${K}Lazy`]: () => Promise<typeof config[K]>;
};
// Equivalent to:
// {
//   location: Promise<string>;
//   name: Promise<string>;
//   year: Promise<number>;
// }

async function withLazyValues(configGetter: LazyValues) {
    await configGetter.locationLazy; // Resultant type: string

    await configGetter.missingLazy();
    //                 ~~~~~~~~~~~
    // Error: Property 'missingLazy' does not exist on type 'LazyValues'.
};
```

请注意，在 JavaScript 中，对象键可以是 `string` 或 `Symbol` 类型——而 `Symbol` 键不能用作模板文字类型，因为它们不是原始类型。如果尝试在泛型类型中使用重映模板文字类型键，TypeScript 将会抱怨 `symbol` 不能用作模板文字类型：

```
type TurnIntoGettersDirect<T> = {
    [K in keyof T as `get${K}`]: () => T[K]
    //                     ~
    // Error: Type 'keyof T' is not assignable to type
    // 'string | number | bigint | boolean | null | undefined'.
    //   Type 'string | number | symbol' is not assignable to type
    //   'string | number | bigint | boolean | null | undefined'.
    //     Type 'symbol' is not assignable to type
    //     'string | number | bigint | boolean | null | undefined'.
};
```

要解决这个限制，您可以使用 `string &` 交集类型来强制只使用可以是字符串的类型。因为 `string & symbol` 的结果是 `never`，整个模板字符串将减少到 `never`，TypeScript 将忽略它：

```
const someSymbol = Symbol("");

interface HasStringAndSymbol {
    StringKey: string;
    [someSymbol]: number;
}

type TurnIntoGetters<T> = {
    [K in keyof T as `get${string & K}`]: () => T[K]
};

type GettersJustString = TurnIntoGetters<HasStringAndSymbol>;
// Equivalent to:
// {
//     getStringKey: () => string;
// }
```

TypeScript 从联合类型中过滤出 `never` 类型的行为再次证明其非常有用！

# 类型操作与复杂性

> 调试比一开始编写代码要难两倍。因此，如果您尽可能聪明地编写代码，那么您在定义上就不够聪明来调试它。
> 
> Brian Kernighan

本章描述的类型操作是当今任何编程语言中最强大、尖端的类型系统特性之一。大多数开发者对它们还不够熟悉，无法调试其复杂用法中的错误。像我在第十二章，“使用 IDE 功能”中介绍的行业标准开发工具通常并不适用于可视化多层次的相互使用的类型操作。

如果你确实需要使用类型操作，请为了任何阅读你代码的开发者（包括未来的你），尽量保持最少的使用。使用可读的名称帮助读者在阅读代码时理解。对于任何你认为未来的读者可能会遇到困难的地方，请留下描述性的注释。

# 总结

在这一章中，通过在其类型系统中操作类型，你揭示了 TypeScript 的真正力量：

+   使用映射类型将现有类型转换为新类型

+   在类型操作中引入逻辑，使用条件类型

+   学习`never`如何与交集、并集、条件类型和映射类型相互作用

+   使用模板字面类型表示字符串类型的模式

+   结合模板字面类型和映射类型来修改类型键

###### 提示

现在你已经完成了本章的阅读，请在[*https://learningtypescript.com/type-operations*](https://learningtypescript.com/type-operations)上练习你所学到的内容。

> 当你在类型系统中迷失时，你会使用什么？
> 
> 一个映射类型！
