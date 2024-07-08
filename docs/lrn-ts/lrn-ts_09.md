# 第七章：接口

> 为什么仅使用
> 
> 无聊的内置类型形状何时
> 
> 我们可以自己创建！

我在第四章，“对象”中提到，虽然 `{ ... }` 对象类型的类型别名是描述对象形状的一种方式，TypeScript 还包括了许多开发人员更喜欢的“接口”特性。接口是另一种使用相关名称声明对象形状的方式。接口在很多方面与别名对象类型类似，但通常更受欢迎，因为它们具有更可读的错误消息、更快的编译器性能以及与类更好的互操作性。

# 类型别名与接口

这里是一个快速回顾如何使用类型别名描述具有 `born: number` 和 `name: string` 的对象的语法：

```
type Poet = {
  born: number;
  name: string;
};
```

这是接口的等效语法：

```
interface Poet {
  born: number;
  name: string;
}
```

这两种语法几乎是相同的。

###### 提示

喜欢分号的 TypeScript 开发人员通常将它们放在类型别名后面，而不是接口后面。这种偏好反映了使用 `;` 声明变量与声明类或函数时的差异。

TypeScript 的可赋值性检查和接口的错误消息在对象类型上的工作和外观几乎相同。如果 `Poet` 是一个接口或类型别名，则对 `valueLater` 变量进行赋值的以下可赋值性错误大致相同：

```
let valueLater: Poet;

// Ok
valueLater = {
  born: 1935,
  name: 'Sara Teasdale',
};

valueLater = "Emily Dickinson";
// Error: Type 'string' is not assignable to 'Poet'.

valueLater = {
  born: true,
  // Error: Type 'boolean' is not assignable to type 'number'.
  name: 'Sappho'
};
```

然而，接口和类型别名之间存在一些关键差异：

+   正如您稍后在本章中看到的，接口可以“合并”在一起进行增强——这在处理第三方代码（如内置全局或 npm 包）时特别有用。

+   正如您将在下一章第八章，“类”中看到的那样，接口可以用于对类声明的结构进行类型检查，而类型别名则不能。

+   接口通常更适合 TypeScript 类型检查器使用：它们声明了一个可以更轻松地在内部缓存的命名类型，而不是像类型别名那样动态复制和粘贴新的对象文字。

+   因为接口被认为是具有命名对象而不是未命名对象文本别名的对象，它们的错误消息在硬边缘情况下更可能是可读的。

基于后两个原因并为保持一致性，本书及其相关项目默认使用接口而不是别名对象形状。我通常建议尽可能使用接口（即，直到需要类型别名的联合类型等功能）。

# 属性类型

JavaScript 对象在实际使用中可能非常复杂和奇特，包括 getter 和 setter、有时仅存在的属性或接受任意属性名称。TypeScript 提供了一套接口类型系统工具，帮助我们建模这种奇特性。

###### 提示

由于接口和类型别名的行为非常相似，本章介绍的以下类型属性也适用于别名对象类型。

## 可选属性

与对象类型一样，接口属性并非都必须是对象中必需的。您可以通过在类型注释中的`:`之前包含`?`来指示接口的属性是可选的。

此`Book`接口仅需要一个`required`属性，并可选地允许一个`optional`属性。符合此接口的对象可以提供`optional`或者将其省略，只要它们提供`required`即可：

```
interface Book {
  author?: string;
  pages: number;
};

// Ok
const ok: Book = {
    author: "Rita Dove",
    pages: 80,
};

const missing: Book = {
    pages: 80
};
// Error: Property 'author' is missing in type
// '{ pages: number; }' but required in type 'Book'.
```

在接口和对象类型中，关于可选属性和类型联合中包含`undefined`的属性之间的差异同样适用。第十三章，“配置选项”将描述 TypeScript 在可选属性周围的严格设置。

## 只读属性

有时，您可能希望阻止接口的用户重新分配符合接口的对象的属性。TypeScript 允许您在属性名称之前添加`readonly`修饰符，以指示一旦设置，该属性不应该被设置为其他值。这些`readonly`属性可以正常读取，但不能重新分配为新值。

例如，下面`Page`接口中的`text`属性在访问时返回一个`string`，但如果分配一个新值则会导致类型错误：

```
interface Page {
    readonly text: string;
}

function read(page: Page) {
    // Ok: reading the text property doesn't attempt to modify it
    console.log(page.text);

    page.text += "!";
    //   ~~~~
    // Error: Cannot assign to 'text'
    // because it is a read-only property.
}
```

注意，`readonly`修饰符仅存在于类型系统中，并且仅适用于该接口的使用。除非该对象在使用位置声明为该接口，否则不会应用于对象。

在这个`exclaim`示例的延续中，`text`属性可以在函数外部修改，因为其父对象直到函数内部才显式用作`Text`。`pageIsh`可以用作`Page`，因为可写属性可分配给`readonly`属性（可变属性可以从中读取，这正是`readonly`属性所需的）：

```
const pageIsh = {
  text: "Hello, world!",
};

// Ok: messengerIsh is an inferred object type with text, not a Page
page.text += "!";

// Ok: read takes in Page, which happens to
// be a more specific version of pageIsh's type
read(messengerIsh);
```

使用显式类型注释`: Page`声明变量`pageIsh`将表明其`text`属性是`readonly`的。然而，其推断类型却不是`readonly`的。

只读接口成员是一种确保代码区域不会意外修改其不应修改的对象的便捷方式。但请记住，它们仅仅是类型系统的构造，不存在于编译后的 JavaScript 输出代码中。它们仅在 TypeScript 类型检查期间保护不被修改。

## 函数和方法

在 JavaScript 中，对象成员为函数是非常常见的。因此，TypeScript 允许声明接口成员为之前在第五章，“函数”中介绍的函数类型。

TypeScript 提供了两种将接口成员声明为函数的方法：

+   *方法*语法：声明接口成员是作为对象的成员调用的函数，如`member(): void`

+   *属性*语法：声明接口成员等于独立函数，如`member: () => void`

这两种声明形式是声明 JavaScript 对象具有函数的两种方式的类比。

这里显示的`method`和`property`成员都是可以不带参数调用并返回`string`的函数：

```
interface HasBothFunctionTypes {
  property: () => string;
  method(): string;
}

const hasBoth: HasBothFunctionTypes = {
  property: () => "",
  method() {
    return "";
  }
};

hasBoth.property(); // Ok
hasBoth.method(); // Ok
```

两种形式都可以接收`?`可选修饰符，表示它们不需要提供：

```
interface OptionalReadonlyFunctions {
  optionalProperty?: () => string;
  optionalMethod?(): string;
}
```

方法和属性声明大部分可以互换使用。我将在本书中介绍它们之间的主要区别是：

+   方法不能声明为`readonly`；属性可以。

+   接口合并（本章后面介绍）会对它们进行不同处理。

+   在第十五章，“类型操作”中涵盖的类型上执行的一些操作会有所不同。

未来的 TypeScript 版本可能会增加更严格地区分方法和属性函数的选项。

目前，我推荐的一般风格指南是：

+   如果您知道底层函数可能引用`this`，最常见的是类的实例（在第八章，“类”中介绍），请使用方法函数。

+   否则使用属性函数。

如果你混淆了这两种方式，或者不理解它们之间的区别，不要担心。除非你有意关注`this`作用域和你选择的形式，否则它很少会影响你的代码。

## 调用签名

接口和对象类型可以声明*调用签名*，这是一个描述值如何被调用的类型系统描述，就像一个函数一样。只有符合调用签名声明的方式调用的值才能赋值给接口，即具有可赋值参数和返回类型的函数。调用签名看起来类似于函数类型，但使用`:`冒号而不是`=>`箭头。

下面的`FunctionAlias`和`CallSignature`类型都描述了相同的函数参数和返回类型：

```
type FunctionAlias = (input: string) => number;

interface CallSignature {
  (input: string): number;
}

// Type: (input: string) => number
const typedFunctionAlias: FunctionAlias = (input) => input.length; // Ok

// Type: (input: string) => number
const typedCallSignature: CallSignature = (input) => input.length; // Ok
```

调用签名可用于描述具有一些用户定义属性的函数。TypeScript 将识别添加到函数声明的属性作为增加到该函数声明类型的属性。

下面的`keepsTrackOfCalls`函数声明具有类型为`number`的`count`属性，使其可以赋值给`FunctionWithCount`接口。因此，它可以被赋值给类型为`FunctionWithCount`的`hasCallCount`参数。代码片段末尾的函数没有给出`count`：

```
interface FunctionWithCount {
  count: number;
  (): void;
}

let hasCallCount: FunctionWithCount;

function keepsTrackOfCalls() {
  keepsTrackOfCalls.count += 1;
  console.log(`I've been called ${keepsTrackOfCalls.count} times!`);
}

keepsTrackOfCalls.count = 0;

hasCallCount = keepsTrackOfCalls; // Ok

function doesNotHaveCount() {
  console.log("No idea!");
}

hasCallCount = doesNotHaveCount;
// Error: Property 'count' is missing in type
// '() => void' but required in type 'FunctionWithCalls'
```

## 索引签名

一些 JavaScript 项目创建的对象旨在将值存储在任意`string`键下。对于这些“容器”对象，声明一个接口并为每个可能的键添加一个字段是不切实际或不可能的。

TypeScript 提供了一种称为 *索引签名* 的语法，指示接口的对象允许接收任何键并返回该键下的特定类型。它们最常与字符串键一起使用，因为 JavaScript 对象属性查找会隐式将键转换为字符串。索引签名看起来像常规的属性定义，但在键后面有一个类型，并用大括号括起来，例如 `{ [i: string]: ... }`。

此 `WordCounts` 接口声明允许任何 `string` 键和 `number` 值。该类型的对象不限于接收任何特定的键，只要值是 `number`：

```
interface WordCounts {
  [i: string]: number;
}

const counts: WordCounts = {};

counts.apple = 0; // Ok
counts.banana = 1; // Ok

counts.cherry = false;
// Error: Type 'boolean' is not assignable to type 'number'.
```

索引签名方便为对象分配值，但并不完全类型安全。它们指示对象无论访问哪个属性，都应返回一个值。

此 `publishDates` 值安全地返回 `Frankenstein` 作为 `Date`，但会让 TypeScript 认为其 `Beloved` 已定义，尽管实际上是 `undefined`。

```
interface DatesByName {
  [i: string]: Date;
}

const publishDates: DatesByName = {
  Frankenstein: new Date("1 January 1818"),
};

publishDates.Frankenstein; // Type: Date
console.log(publishDates.Frankenstein.toString()); // Ok

publishDates.Beloved; // Type: Date, but runtime value of undefined!
console.log(publishDates.Beloved.toString()); // Ok in the type system, but...
// Runtime error: Cannot read property 'toString'
// of undefined (reading publishDates.Beloved)
```

可能的话，如果您希望存储键值对并且事先不知道键，通常更安全的做法是使用 `Map`。其 `.get` 方法总是返回带有 `| undefined` 的类型，以指示可能不存在该键。第九章，“类型修饰符” 将讨论如何使用通用容器类，如 `Map` 和 `Set`。

### 混合属性和索引签名

接口能够包含显式命名属性和通用的 `string` 索引签名，但有一个限制：每个命名属性的类型必须可分配给其通用索引签名的类型。您可以将它们混合使用，告诉 TypeScript 命名属性提供更具体的类型，而任何其他属性则回退到索引签名的类型。

在这里，`HistoricalNovels` 声明所有属性的类型都是 `number`，并且另外 `Oroonoko` 属性必须首先存在：

```
interface HistoricalNovels {
  Oroonoko: number;
  [i: string]: number;
}

// Ok
const novels: HistoricalNovels = {
  Outlander: 1991,
  Oroonoko: 1688,
};

const missingOroonoko: HistoricalNovels = {
  Outlander: 1991,
};
// Error: Property 'Oroonoko' is missing in type
// '{ Outlander: number; }' but required in type 'HistoricalNovels'.
```

一种常见的类型系统技巧，混合属性和索引签名的是，使用比索引签名的原始类型更具体的属性类型文字。只要命名属性的类型可分配给索引签名的类型（分别对于文字和原始类型），TypeScript 就会允许它。

在这里，`ChapterStarts` 声明 `preface` 下的属性必须是 `0`，而所有其他属性都是更一般的 `number`。这意味着任何符合 `ChapterStarts` 的对象必须具有 `preface` 属性等于 `0`：

```
interface ChapterStarts {
  preface: 0;
  [i: string]: number;
}

const correctPreface: ChapterStarts = {
  preface: 0,
  night: 1,
  shopping: 5
};

const wrongPreface: ChapterStarts = {
  preface: 1,
  // Error: Type '1' is not assignable to type '0'.
};
```

### 数字索引签名

尽管 JavaScript 隐式将对象属性查找键转换为字符串，有时仅允许数字作为对象的键是可取的。TypeScript 索引签名可以使用 `number` 类型而不是 `string`，但与命名属性相同的是，它们的类型必须可分配给通用的 `string` 索引签名。

以下`MoreNarrowNumbers`接口将被允许，因为`string`可以分配给`string | undefined`，但`MoreNarrowStrings`不会，因为`string | undefined`不能分配给`string`：

```
// Ok
interface MoreNarrowNumbers {
  [i: number]: string;
  [i: string]: string | undefined;
}

// Ok
const mixesNumbersAndStrings: MoreNarrowNumbers = {
  0: '',
  key1: '',
  key2: undefined,
}

interface MoreNarrowStrings {
  [i: number]: string | undefined;
  // Error: 'number' index type 'string | undefined'
  // is not assignable to 'string' index type 'string'.
  [i: string]: string;
}
```

## 嵌套接口

就像对象类型可以作为其他对象类型的属性进行嵌套一样，接口类型也可以具有作为接口类型（或对象类型）的属性。

这个`Novel`接口包含一个必须满足内联对象类型的`author`属性和一个必须满足`Setting`接口的`setting`属性：

```
interface Novel {
    author: {
        name: string;
    };
    setting: Setting;
}

interface Setting {
    place: string;
    year: number;
}

let myNovel: Novel;

// Ok
myNovel = {
    author: {
        name: 'Jane Austen',
    },
    setting: {
        place: 'England',
        year: 1812,
    }
};

myNovel = {
    author: {
        name: 'Emily Brontë',
    },
    setting: {
        place: 'West Yorkshire',
    },
    // Error: Property 'year' is missing in type
    // '{ place: string; }' but required in type 'Setting'.
};
```

# 接口扩展

有时您可能会得到多个看起来相似的接口。一个接口可能包含另一个接口的所有相同成员，并添加一些额外的成员。

TypeScript 允许一个接口*扩展*另一个接口，这表示它复制另一个接口的所有成员。通过在名称后添加`extends`关键字（“派生”接口）并在其后跟要扩展的接口的名称（“基”接口），可以将接口标记为扩展另一个接口。这样做告诉 TypeScript，所有符合派生接口的对象也必须具有基接口的所有成员。

在以下示例中，`Novella`接口从`Writing`扩展，因此要求对象至少具有`Novella`的`pages`和`Writing`的`title`成员：

```
interface Writing {
    title: string;
}

interface Novella extends Writing {
    pages: number;
}

// Ok
let myNovella: Novella = {
    pages: 195,
    title: "Ethan Frome",
};

let missingPages: Novella = {
 // ~~~~~~~~~~~~
 // Error: Property 'pages' is missing in type
 // '{ title: string; }' but required in type 'Novella'.
    title: "The Awakening",
}

let extraProperty: Novella = {
 // ~~~~~~~~~~~~~
 // Error: Type '{ genre: string; name: string; strategy: string; }'
 // is not assignable to type 'Novella'.
 //   Object literal may only specify known properties,
 //   and 'genre' does not exist in type 'Novella'.
    pages: 300,
    strategy: "baseline",
    style: "Naturalism"
};
```

接口扩展是一种巧妙的方式，表示项目中的一种实体类型是另一种实体类型的超集（它包含另一个实体的所有成员）。它们允许您避免在多个接口中重复键入相同的代码以表示该关系。

## 覆盖属性

派生接口可以通过使用不同类型重新声明属性来*覆盖*或替换其基接口的属性。TypeScript 的类型检查器将强制执行覆盖属性必须可分配给其基属性。它这样做是为了确保派生接口类型的实例仍然可以分配给基接口类型。

大多数重新声明属性的派生接口要么使这些属性成为类型联合的更具体子集，要么使属性成为扩展自基接口类型的类型。

例如，这个`WithNullableName`类型在`WithNonNullableName`中被正确地设置为非空。然而，`WithNumericName`不被允许作为`number | string`，并且不可分配给`string | null`：

```
interface WithNullableName {
    name: string | null;
}

interface WithNonNullableName extends WithNullableName {
    name: string;
}

interface WithNumericName extends WithNullableName {
    name: number | string;
}
// Error: Interface 'WithNumericName' incorrectly
// extends interface 'WithNullableName'.
//   Types of property 'name' are incompatible.
//     Type 'string | number' is not assignable to type 'string | null'.
//       Type 'number' is not assignable to type 'string'.
```

## 扩展多个接口

在 TypeScript 中，允许将接口声明为扩展多个其他接口。在派生接口名称后的`extends`关键字之后可以使用任意数量的由逗号分隔的接口名称。派生接口将接收所有基接口的成员。

在这里，`GivesBothAndEither`有三种方法：一个是自己的，一个来自`GivesNumber`，一个来自`GivesString`：

```
interface GivesNumber {
  giveNumber(): number;
}

interface GivesString {
  giveString(): string;
}

interface GivesBothAndEither extends GivesNumber, GivesString {
  giveEither(): number | string;
}

function useGivesBoth(instance: GivesBothAndEither) {
  instance.giveEither(); // Type: number | string
  instance.giveNumber(); // Type: number
  instance.giveString(); // Type: string
}
```

通过将接口标记为扩展多个其他接口，可以减少代码重复，并使对象形状在不同代码区域中更易于重用。

# 接口合并

接口的重要特性之一是它们能够*合并*。接口合并意味着如果在同一作用域中声明了两个名称相同的接口，则它们将合并为一个更大的接口，在该名称下具有所有声明的字段。

此片段声明了一个具有两个属性`fromFirst`和`fromSecond`的`Merged`接口：

```
interface Merged {
  fromFirst: string;
}

interface Merged {
  fromSecond: number;
}

// Equivalent to:
// interface Merged {
//   fromFirst: string;
//   fromSecond: number;
// }
```

接口合并在日常 TypeScript 开发中并不经常使用。我建议在可能的情况下避免使用它，因为在一个接口在多个位置声明时，代码理解起来可能会比较困难。

然而，接口合并特别适用于增强来自外部包或内置全局接口（如`Window`）的接口。例如，在使用默认的 TypeScript 编译器选项时，在一个文件中声明了一个带有`myEnvironmentVariable`属性的`Window`接口，会使`window.myEnvironmentVariable`可用：

```
interface Window {
  myEnvironmentVariable: string;
}

window.myEnvironmentVariable; // Type: string
```

我将在第十一章，“声明文件”中更深入地讨论类型定义，以及在第十三章，“配置选项”中讨论 TypeScript 全局类型选项。

## 成员命名冲突

请注意，合并接口可能不会多次使用不同类型声明同一属性名称。如果在接口中已经声明了一个属性，则稍后合并的接口必须使用相同的类型。

在这个`MergedProperties`接口中，`same`属性是允许的，因为它在两个声明中是相同的，但`different`因为类型不同而报错：

```
interface MergedProperties {
  same: (input: boolean) => string;
  different: (input: string) => string;
}

interface MergedProperties {
  same: (input: boolean) => string; // Ok

  different: (input: number) => string;
  // Error: Subsequent property declarations must have the same type.
  // Property 'different' must be of type '(input: string) => string',
  // but here has type '(input: number) => string'.
}
```

然而，合并接口可以定义具有相同名称但不同签名的方法。这样做会为方法创建一个函数重载。

此`MergedMethods`接口创建了一个具有两个重载的`different`方法：

```
interface MergedMethods {
  different(input: string): string;
}

interface MergedMethods {
  different(input: number): string; // Ok
}
```

# 摘要

本章介绍了如何通过接口描述对象类型：

+   使用接口而不是类型别名来声明对象类型

+   各种接口属性类型：可选的、只读的、函数和方法

+   使用索引签名捕获对象属性

+   使用嵌套接口和`extends`继承重用接口

+   如何合并具有相同名称的接口

接下来将是设置多个对象具有相同属性的本机 JavaScript 语法：类。

###### 提示

现在您已经完成了本章的阅读，请在[*https://learningtypescript.com/objects-and-interfaces*](https://learningtypescript.com/objects-and-interfaces)上练习所学内容。

> 接口为什么是好的驱动程序？
> 
> 它们在合并时非常出色。
