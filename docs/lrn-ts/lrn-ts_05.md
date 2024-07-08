# 第四章：对象

> 对象字面量
> 
> 一组键和值
> 
> 每个都有自己的类型

第三章，“联合类型和字面量”详细介绍了联合类型和字面量类型：处理诸如 `boolean` 这样的原始类型及其值的字面量，例如 `true`。这些原始类型只是 JavaScript 代码中常用的复杂对象形状的冰山一角。如果 TypeScript 不能表示这些对象，它将显得相当无用。本章将介绍如何描述复杂的对象形状以及 TypeScript 如何检查它们的可赋值性。

# 对象类型

当您使用 `{...}` 语法创建对象字面量时，TypeScript 将视其为一个新的对象类型或类型形状，基于其属性。该对象类型将具有与对象值相同的属性名称和原始类型。可以通过 `value.member` 或等效的 `value['member']` 语法访问值的属性。

TypeScript 理解以下 `poet` 变量的类型是一个具有两个属性 `born`（类型为 `number`）和 `name`（类型为 `string`）的对象。允许访问这些成员，但尝试访问任何其他成员名称将导致该名称不存在的类型错误：

```
const poet = {
    born: 1935,
    name: "Mary Oliver",
};

poet['born']; // Type: number
poet.name; // Type: string

poet.end;
//   ~~~
// Error: Property 'end' does not exist on
// type '{ name: string; start: number; }'.
```

对象类型是 TypeScript 理解 JavaScript 代码的核心概念。除了 `null` 和 `undefined` 外的每个值都具有其后台类型形状中的一组成员，因此 TypeScript 必须理解每个值的对象类型以进行类型检查。

## 声明对象类型

直接从现有对象推断类型很好，但最终您会想要显式声明对象的类型。您需要一种方法来单独描述对象形状，而不是满足它的对象。

可以使用类似对象字面量但字段为类型的语法来描述对象类型。这是 TypeScript 在类型可赋值性错误消息中展示的相同语法。

此 `poetLater` 变量与之前的 `name: string` 和 `born: number` 相同类型：

```
let poetLater: {
    born: number;
    name: string;
};

// Ok
poetLater = {
    born: 1935,
    name: "Mary Oliver",
};

poetLater = "Sappho";
// Error: Type 'string' is not assignable to
// type '{ born: number; name: string; }'
```

## 别名对象类型

不断地像 `{ born: number; name: string; }` 这样写出对象类型会很快变得乏味。更常见的做法是使用类型别名为每个类型形状分配一个名称。

前面的代码片段可以用 `type Poet` 重写，这样做的额外好处是使 TypeScript 的可赋值性错误消息更加直接和可读。

```
type Poet = {
    born: number;
    name: string;
};

let poetLater: Poet;

// Ok
poetLater = {
    born: 1935,
    name: "Sara Teasdale",
};

poetLater = "Emily Dickinson";
// Error: Type 'string' is not assignable to 'Poet'.
```

###### 注意

大多数 TypeScript 项目更喜欢使用 `interface` 关键字来描述对象类型，这是一个我不会在 第七章，“接口” 之前介绍的特性。别名对象类型和接口几乎是相同的：本章的所有内容同样适用于接口。

我现在提到这些对象类型，因为理解 TypeScript 如何解释对象文字是学习 TypeScript 类型系统的重要部分。一旦我们转向本书下一节的功能，这些概念将继续非常重要。

# 结构化类型

TypeScript 的类型系统是*结构化类型*的：意味着任何满足某个类型的值都允许用作该类型的值。换句话说，当您声明参数或变量为特定对象类型时，您告诉 TypeScript，无论使用哪个对象（们），它们都必须具有这些属性。

以下`WithFirstName`和`WithLastName`别名对象类型仅声明了一个名为`string`的成员。`hasBoth`变量恰好具有这两个成员，即使它没有明确声明为这样，因此可以提供给声明为两个别名对象类型之一的变量：

```
type WithFirstName = {
  firstName: string;
};

type WithLastName = {
  lastName: string;
};

const hasBoth = {
  firstName: "Lucille",
  lastName: "Clifton",
};

// Ok: `hasBoth` contains a `firstName` property of type `string`
let withFirstName: WithFirstName = hasBoth;

// Ok: `hasBoth` contains a `lastName` property of type `string`
let withLastName: WithLastName = hasBoth;
```

结构化类型并不等同于*鸭子类型*，后者源自短语“如果它看起来像鸭子，叫起来像鸭子，那么它可能是鸭子”。

+   结构化类型是静态系统检查类型的一种情况，在 TypeScript 的情况下是类型检查器。

+   鸭子类型是指在运行时使用对象类型之前没有任何检查。

总结：*JavaScript*是*鸭子类型*，而*TypeScript*是*结构化类型*。

## 使用检查

当向带有对象类型注释的位置提供值时，TypeScript 会检查该值是否可以分配给该对象类型。首先，该值必须具有对象类型的必需属性。如果对象中缺少对象类型所需的任何成员，则 TypeScript 将发出类型错误。

以下`FirstAndLastNames`别名对象类型要求`first`和`last`属性都存在。包含这两者的对象允许在声明为`FirstAndLastNames`类型的变量中使用，但没有它们的对象则不允许：

```
type FirstAndLastNames = {
  first: string;
  last: string;
};

// Ok
const hasBoth: FirstAndLastNames = {
  first: "Sarojini",
  last: "Naidu",
};

const hasOnlyOne: FirstAndLastNames = {
  first: "Sappho"
};
// Property 'last' is missing in type '{ first: string; }'
// but required in type 'FirstAndLastNames'.
```

两者之间不允许类型不匹配。对象类型同时指定必需属性的名称和这些属性预期的类型。如果对象的属性不匹配，TypeScript 将报告类型错误。

以下`TimeRange`类型期望`start`成员为`Date`类型。`hasStartString`对象引起类型错误，因为其`start`类型为`string`：

```
type TimeRange = {
  start: Date;
};

const hasStartString: TimeRange = {
  start: "1879-02-13",
  // Error: Type 'string' is not assignable to type 'Date'.
};
```

## 过量属性检查

TypeScript 会在变量声明为对象类型并且其初始值的字段数超过其描述的类型时报告类型错误。因此，声明变量为对象类型是让类型检查器确保该类型仅具有预期字段的一种方式。

以下`poetMatch`变量具有恰好与由`Poet`别名的对象类型描述的字段，而`extraProperty`因具有额外属性而导致类型错误：

```
type Poet = {
    born: number;
    name: string;
}

// Ok: all fields match what's expected in Poet
const poetMatch: Poet = {
  born: 1928,
  name: "Maya Angelou"
};

const extraProperty: Poet = {
    activity: "walking",
    born: 1935,
    name: "Mary Oliver",
};
// Error: Type '{ activity: string; born: number; name: string; }'
// is not assignable to type 'Poet'.
//   Object literal may only specify known properties,
//   and 'activity' does not exist in type 'Poet'.
```

请注意，多余属性检查仅在被声明为对象类型的位置创建对象文字时触发。提供现有对象文字会绕过多余属性检查。

此 `extraPropertyButOk` 变量不会触发前面示例中的 `Poet` 类型的类型错误，因为其初始值恰好与 `Poet` 结构匹配：

```
const existingObject = {
    activity: "walking",
    born: 1935,
    name: "Mary Oliver",
};

const extraPropertyButOk: Poet = existingObject; // Ok
```

多余属性检查将在任何创建新对象的位置触发，该位置预期该对象与对象类型匹配——正如您将在后面的章节中看到的那样，包括数组成员、类字段和函数参数。禁止多余属性是 TypeScript 帮助确保代码清洁且符合您期望的另一种方式。未在其对象类型中声明的多余属性通常是拼写错误的属性名称或未使用的代码。

## 嵌套对象类型

由于 JavaScript 对象可以嵌套为其他对象的成员，因此 TypeScript 的对象类型必须能够在类型系统中表示嵌套对象类型。做到这一点的语法与之前相同，但使用 `{ ... }` 对象类型而不是基本名称。

`Poem` 类型被声明为一个对象，其 `author` 属性具有 `firstName: string` 和 `lastName: string`。`poemMatch` 变量可以分配给 `Poem`，因为它匹配该结构，而 `poemMismatch` 不行，因为其 `author` 属性包含 `name` 而不是 `firstName` 和 `lastName`：

```
type Poem = {
    author: {
        firstName: string;
        lastName: string;
    };
    name: string;
};

// Ok
const poemMatch: Poem = {
    author: {
        firstName: "Sylvia",
        lastName: "Plath",
    },
    name: "Lady Lazarus",
};

const poemMismatch: Poem = {
    author: {
        name: "Sylvia Plath",
    },
    // Error: Type '{ name: string; }' is not assignable
    // to type '{ firstName: string; lastName: string; }'.
    //   Object literal may only specify known properties, and 'name'
    //   does not exist in type '{ firstName: string; lastName: string; }'.
    name: "Tulips",
};
```

另一种编写`type Poem`的方法是将`author`属性的形状提取为其自己的别名对象类型`Author`。将嵌套类型提取为它们自己的类型别名也有助于 TypeScript 提供更具信息性的类型错误消息。在这种情况下，它可以说 `'Author'` 而不是 `'{ firstName: string; lastName: string; }'`：

```
type Author = {
    firstName: string;
    lastName: string;
};

type Poem = {
    author: Author;
    name: string;
};

const poemMismatch: Poem = {
    author: {
        name: "Sylvia Plath",
    },
    // Error: Type '{ name: string; }' is not assignable to type 'Author'.
    //     Object literal may only specify known properties,
    //     and 'name' does not exist in type 'Author'.
    name: "Tulips",
};
```

###### 提示

通常最好像这样将嵌套对象类型移到它们自己的类型名称中，这样做不仅可以使代码更可读，还可以使 TypeScript 错误消息更易于理解。

您将在后面的章节中看到，对象类型成员可以是其他类型，如数组和函数。

## 可选属性

对象类型属性并不都必须在对象中是必需的。您可以在类型属性的类型注释中的`:`之前包含`?`来指示它是一个可选属性。

此 `Book` 类型仅需要一个 `pages` 属性，并可选允许一个 `author`。符合此类型的对象可以提供 `author` 或将其省略，只要它们提供 `pages`：

```
type Book = {
  author?: string;
  pages: number;
};

// Ok
const ok: Book = {
    author: "Rita Dove",
    pages: 80,
};

const missing: Book = {
    author: "Rita Dove",
};
// Error: Property 'pages' is missing in type
// '{ author: string; }' but required in type 'Book'.
```

请记住，可选属性和类型联合中包含 `undefined` 的属性之间存在差异。用 `?` 声明的可选属性允许不存在。用 `| undefined` 声明为必需的属性必须存在，即使值为 `undefined`。

在以下 `Writers` 类型中，`editor` 属性可以在声明变量时被省略，因为在其声明中有 `?`。`author` 属性没有 `?`，因此它必须存在，即使其值只是 `undefined`：

```
type Writers = {
  author: string | undefined;
  editor?: string;
};

// Ok: author is provided as undefined
const hasRequired: Writers = {
  author: undefined,
};

const missingRequired: Writers = {};
//    ~~~~~~~~~~~~~~~
// Error: Property 'author' is missing in type
// '{}' but required in type 'Writers'.
```

第七章，“接口” 将进一步讨论其他类型属性，而第十三章，“配置选项”将描述 TypeScript 关于可选属性的严格设置。

# 对象类型的联合

在 TypeScript 代码中，希望能够描述一种类型，它可以是一个或多个具有稍有不同属性的不同对象类型是合理的。此外，你的代码可能希望能够基于属性值来在这些对象类型之间进行类型缩小。

## 推断的对象类型联合

如果一个变量被赋予一个可能是多种对象类型之一的初始值，TypeScript 将推断其类型为对象类型的联合。该联合类型将为每种可能的对象形状包含一个成员。类型上的每种可能属性在这些成员中都将存在，尽管它们在任何没有初始值的类型上将是 `?` 可选类型。

这个 `poem` 值始终具有类型为 `string` 的 `name` 属性，并且可能具有 `pages` 和 `rhymes` 属性，也可能没有：

```
const poem = Math.random() > 0.5
  ? { name: "The Double Image", pages: 7 }
  : { name: "Her Kind", rhymes: true };
// Type:
// {
//   name: string;
//   pages: number;
//   rhymes?: undefined;
// }
// |
// {
//   name: string;
//   pages?: undefined;
//   rhymes: boolean;
// }

poem.name; // string
poem.pages; // number | undefined
poem.rhymes; // booleans | undefined
```

## 显式对象类型联合

或者，你可以通过显式地使用自己的对象类型联合更加明确地描述你的对象类型。这样做需要编写更多的代码，但带来的好处是更多地控制你的对象类型。尤其值得注意的是，如果一个值的类型是对象类型的联合，TypeScript 的类型系统将只允许访问所有这些联合类型上存在的属性。

先前 `poem` 变量的这个版本显式地被类型化为一个联合类型，它总是具有 `always` 属性以及 `pages` 或 `rhymes` 中的一个。允许访问 `names` 是因为它总是存在的，但不能保证 `pages` 和 `rhymes` 的存在：

```
type PoemWithPages = {
    name: string;
    pages: number;
};

type PoemWithRhymes = {
    name: string;
    rhymes: boolean;
};

type Poem = PoemWithPages | PoemWithRhymes;

const poem: Poem = Math.random() > 0.5
  ? { name: "The Double Image", pages: 7 }
  : { name: "Her Kind", rhymes: true };

poem.name; // Ok

poem.pages;
//   ~~~~~
// Property 'pages' does not exist on type 'Poem'.
//   Property 'pages' does not exist on type 'PoemWithRhymes'.

poem.rhymes;
//   ~~~~~~
// Property 'rhymes' does not exist on type 'Poem'.
//   Property 'rhymes' does not exist on type 'PoemWithPages'.
```

限制对对象可能不存在的成员的访问对于代码安全性来说是一件好事。如果一个值可能是多种类型之一，那么并非所有这些类型上都存在的属性在对象上并不保证存在。

正如文字和/或原始类型的联合必须被类型缩小以访问并非存在于所有类型成员上的属性一样，你需要缩小那些对象类型的联合。

## 缩小对象类型

如果类型检查器看到代码的某个区域只能在联合类型值包含某个属性时运行，它将缩小该值的类型，以仅包含包含该属性的成员。换句话说，如果在代码中检查它们的形状，TypeScript 的类型缩小将适用于对象。

继续使用显式类型化的 `poem` 示例，检查 `poem` 中的 `"pages" in poem` 是否作为 TypeScript 的类型守卫表明它是一个 `PoemWithPages`。如果 `poem` 不是 `PoemWithPages`，那么它必须是 `PoemWithRhymes`：

```
if ("pages" in poem) {
    poem.pages; // Ok: poem is narrowed to PoemWithPages
} else {
    poem.rhymes; // Ok: poem is narrowed to PoemWithRhymes
}
```

注意，TypeScript 不允许类似 `if (poem.pages)` 的真值存在检查。试图访问一个可能不存在的对象属性被视为类型错误，即使看起来像是类型保护的一种方式：

```
if (poem.pages) { /* ... */ }
//       ~~~~~
// Property 'pages' does not exist on type 'PoemWithPages | PoemWithRhymes'.
//   Property 'pages' does not exist on type 'PoemWithRhymes'.
```

## 辨别联合

JavaScript 和 TypeScript 中另一种流行的联合类型对象形式是在对象上有一个属性指示对象的形状。这种类型形状称为 *辨别联合*，其值指示对象类型的属性称为 *辨别符*。TypeScript 能够对辨别属性进行类型缩小，从而进行类型收窄。

例如，这个 `Poem` 类型描述了一个可以是 `PoemWithPages` 类型或 `PoemWithRhymes` 类型的对象，而 `type` 属性指示其中一个。如果 `poem.type` 是 `"pages"`，那么 TypeScript 能够推断出 `poem` 的类型必须是 `PoemWithPages`。如果没有这种类型收窄，值上既不保证任何一个属性的存在：

```
type PoemWithPages = {
    name: string;
    pages: number;
    type: 'pages';
};

type PoemWithRhymes = {
    name: string;
    rhymes: boolean;
    type: 'rhymes';
};

type Poem = PoemWithPages | PoemWithRhymes;

const poem: Poem = Math.random() > 0.5
  ? { name: "The Double Image", pages: 7, type: "pages" }
  : { name: "Her Kind", rhymes: true, type: "rhymes" };

if (poem.type === "pages") {
    console.log(`It's got pages: ${poem.pages}`); // Ok
} else {
    console.log(`It rhymes: ${poem.rhymes}`);
}

poem.type; // Type: 'pages' | 'rhymes'

poem.pages;
//   ~~~~~
// Error: Property 'pages' does not exist on type 'Poem'.
//   Property 'pages' does not exist on type 'PoemWithRhymes'.
```

辨别联合是我在 TypeScript 中最喜欢的功能，因为它们优雅地结合了 JavaScript 中常见的优雅模式与 TypeScript 的类型收窄。第十章，“泛型” 及其相关项目将展示更多关于如何使用辨别联合进行通用数据操作。

# 交集类型

TypeScript 的 `|` 联合类型代表一个值可以是两种或多种不同类型中的一种。正如 JavaScript 的运行时 `|` 运算符是其 `&` 运算符的对应物，TypeScript 允许表示一个同时为多种类型的类型：一个 `&` *交集类型*。交集类型通常与别名对象类型一起使用，以创建一个结合多个现有对象类型的新类型。

下面的 `Artwork` 和 `Writing` 类型用于形成一个合并的 `WrittenArt` 类型，具有 `genre`、`name` 和 `pages` 属性：

```
type Artwork = {
    genre: string;
    name: string;
};

type Writing = {
    pages: number;
    name: string;
};

type WrittenArt = Artwork & Writing;
// Equivalent to:
// {
//   genre: string;
//   name: string;
//   pages: number;
// }
```

交集类型可以与联合类型结合使用，这在描述一个类型的辨别联合时有时是很有用的。

这种 `ShortPoem` 类型始终具有 `author` 属性，并且还是一个基于 `type` 属性的辨别联合：

```
type ShortPoem = { author: string } & (
    | { kigo: string; type: "haiku"; }
    | { meter: number; type: "villanelle"; }
);

// Ok
const morningGlory: ShortPoem = {
    author: "Fukuda Chiyo-ni",
    kigo: "Morning Glory",
    type: "haiku",
};

const oneArt: ShortPoem = {
    author: "Elizabeth Bishop",
    type: "villanelle",
};
// Error: Type '{ author: string; type: "villanelle"; }'
// is not assignable to type 'ShortPoem'.
//   Type '{ author: string; type: "villanelle"; }' is not assignable to
//   type '{ author: string; } & { meter: number; type: "villanelle"; }'.
//     Property 'meter' is missing in type '{ author: string; type: "villanelle"; }'
//     but required in type '{ meter: number; type: "villanelle"; }'.
```

## 交集类型的危险

交集类型是一个有用的概念，但是在使用它们时很容易让自己或 TypeScript 编译器感到困惑。我建议在使用它们时尽量保持代码简单。

### 长的可赋值性错误

当你创建复杂的交集类型（例如与联合类型结合）时，TypeScript 给出的可赋值性错误消息会变得更难理解。这将是 TypeScript 类型系统（以及一般类型化编程语言）的一个常见主题：你的代码越复杂，理解类型检查器消息的难度就越大。

在前面代码片段的 `ShortPoem` 情况下，将类型拆分成一系列别名对象类型将会更易读，允许 TypeScript 打印这些名称：

```
type ShortPoemBase = { author: string };
type Haiku = ShortPoemBase & { kigo: string; type: "haiku" };
type Villanelle = ShortPoemBase & { meter: number; type: "villanelle" };
type ShortPoem = Haiku | Villanelle;

const oneArt: ShortPoem = {
    author: "Elizabeth Bishop",
    type: "villanelle",
};
// Type '{ author: string; type: "villanelle"; }'
// is not assignable to type 'ShortPoem'.
//   Type '{ author: string; type: "villanelle"; }'
//   is not assignable to type 'Villanelle'.
//     Property 'meter' is missing in type
//     '{ author: string; type: "villanelle"; }'
//     but required in type '{ meter: number; type: "villanelle"; }'.
```

### 永不

交叉类型也很容易被误用，并创建一个不可能存在的类型。原始类型不能作为交叉类型中的组成部分，因为一个值不可能同时是多个原始类型。尝试将两个原始类型用`&`连接在一起将导致*never*类型，由关键字`never`表示：

```
type NotPossible = number & string;
// Type: never
```

`never`关键字和类型是编程语言中所谓的*底部类型*或空类型。底部类型是一种没有可能值且无法到达的类型。不能为类型为底部类型的位置提供任何类型：

```
let notNumber: NotPossible = 0;
//  ~~~~~~~~~
// Error: Type 'number' is not assignable to type 'never'.

let notString: never = "";
//  ~~~~~~~~~
// Error: Type 'string' is not assignable to type 'never'.
```

大多数 TypeScript 项目很少——甚至从不——使用`never`类型。它偶尔会出现在代码中表示不可能的状态。不过，大部分时间，这可能是误用交叉类型的错误。我将在第十五章，“类型操作”中更详细地介绍它。

# 总结

在本章中，您扩展了对 TypeScript 类型系统的理解，以便能够处理对象：

+   TypeScript 如何从对象类型字面量中解释类型

+   描述对象字面类型，包括嵌套和可选属性

+   使用对象字面类型的联合进行声明、推断和类型缩小

+   区分联合和鉴别联合

+   将对象类型与交叉类型结合在一起

###### 提示

现在您已经完成了本章的阅读，请在[*https://learningtypescript.com/objects*](https://learningtypescript.com/objects)上练习所学内容。

> 律师如何声明他们的 TypeScript 类型？
> 
> “我反对！”
