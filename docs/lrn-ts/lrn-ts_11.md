# 第九章：类型修饰符

> 类型的类型来自于类型。
> 
> “世界之大无奇不有。”
> 
> Anders 喜欢说。

到现在为止，你已经详细了解了 TypeScript 类型系统如何与现有的 JavaScript 构造（例如数组、类和对象）配合工作。对于本章和第十章，“泛型”，我将进一步深入到类型系统本身，并展示侧重于编写更精确类型以及基于其他类型的特性。

# 顶部类型

在第四章，“对象”中，我提到了*底部类型*的概念，用来描述一种不能有可能值且不能被访问的类型。有道理认为类型理论中可能也存在相反的情况。确实存在！

*顶部类型*，或通用类型，是一种可以表示系统中任何可能值的类型。所有其他类型的值都可以提供给类型为顶部类型的位置。换句话说，所有类型都可以分配给顶部类型。

## 再次提到 `any`

`any` 类型可以充当顶部类型，因为任何类型都可以提供给类型为 `any` 的位置。通常情况下，当位置允许接受任何类型的数据时，如 `console.log` 的参数：

```
let anyValue: any;
anyValue = "Lucille Ball"; // Ok
anyValue = 123; // Ok

console.log(anyValue); // Ok
```

`any` 的问题在于它明确告诉 TypeScript 不要对该值的可分配性或成员进行类型检查。如果你想快速绕过 TypeScript 的类型检查器，这种缺乏安全性很有用，但类型检查的禁用会降低 TypeScript 对该值的实用性。

例如，下面的 `name.toUpperCase()` 调用肯定会崩溃，但因为 `name` 声明为 `any`，TypeScript 不会报类型错误：

```
function greetComedian(name: any) {
    // No type error...
    console.log(`Announcing ${name.toUpperCase()}!`);
}

greetComedian({ name: "Bea Arthur" });
    // Runtime error: name.toUpperCase is not a function
```

如果你想表示一个值可以是任何东西，`unknown` 类型会更安全。

## unknown

TypeScript 中的 `unknown` 类型是其真正的顶部类型。`unknown` 类似于 `any`，因为所有对象都可以传递给类型为 `unknown` 的位置。与 `any` 的关键区别在于，TypeScript 对 `unknown` 类型的值更加严格：

+   TypeScript 不允许直接访问 `unknown` 类型值的属性。

+   `unknown` 不能分配给非顶部类型（如 `any` 或 `unknown`）的类型。

尝试访问 `unknown` 类型值的属性，例如以下片段，将导致 TypeScript 报告类型错误：

```
function greetComedian(name: unknown) {
    console.log(`Announcing ${name.toUpperCase()}!`);
    //                        ~~~~
    // Error: Object is of type 'unknown'.
}
```

TypeScript 只有在值的类型被缩小（例如使用 `instanceof` 或 `typeof`，或使用类型断言）时，才允许对 `unknown` 类型的名称访问成员。

此代码片段使用 `typeof` 将 `name` 从 `unknown` 缩小为 `string`：

```
function greetComedianSafety(name: unknown) {
    if (typeof value === "string") {
        console.log(`Announcing ${name.toUpperCase()}!`); // Ok
    } else {
        console.log("Well, I'm off.");
    }
}

greetComedianSafety("Betty White"); // Logs: 4
greetComedianSafety({}); // Does not log
```

这两个限制使得 `unknown` 比 `any` 更安全。通常情况下，应尽可能使用 `unknown` 而不是 `any`。

# 类型谓词

我之前向你展示过 JavaScript 构造（例如 `instanceof` 和 `typeof`）如何用于缩小类型。直接使用有限的这组检查是完全没问题的，但如果将逻辑封装在函数中，就会丢失这些检查。

例如，这个`isNumberOrString`函数接受一个值并返回一个布尔值，指示该值是`number`还是`string`。我们作为人类可以推断`if`语句中的`value`必须是这两种类型之一，因为`isNumberOrString(value)`返回 true，但 TypeScript 不知道。它只知道`isNumberOrString`返回一个布尔值，而不知道它意味着缩小参数的类型：

```
function isNumberOrString(value: unknown) {
    return ['number', 'string'].includes(typeof value);
}

function logValueIfExists(value: number | string | null | undefined) {
    if (isNumberOrString(value)) {
        // Type of value: number | string | null | undefined
        value.toString();
        // Error: Object is possibly undefined.
    } else {
        console.log("Value does not exist:", value);
    }
}
```

TypeScript 有一种专门的语法用于返回一个布尔值，表示参数是否是特定类型。这被称为*类型谓词*，有时也称为“用户定义的类型保护”：您作为开发人员正在创建自己的类型保护，类似于`instanceof`或`typeof`。类型谓词通常用于指示传递为参数的参数是否比参数的更具体类型。

类型谓词的返回类型可以声明为参数的名称、`is`关键字和某种类型：

```
function typePredicate(input: WideType): input is NarrowType;
```

我们可以将前面示例中的辅助函数更改为具有显式返回类型，明确说明`value is number | string`。TypeScript 将能够推断出仅在`value is number | string`为`true`时可达的代码块必须具有`number | string`类型的`value`。此外，仅在`value is number | string`为`false`时可达的代码块必须具有`null | undefined`类型的`value`：

```
function isNumberOrString(value: unknown): value is number | string {
    return ['number', 'string'].includes(typeof value);
}

function logValueIfExists(value: number | string | null | undefined) {
    if (isNumberOrString(value)) {
        // Type of value: number | string
        value.toString(); // Ok
    } else {
        // Type of value: null | undefined
        console.log("value does not exist:", value);
    }
}
```

您可以将类型谓词视为返回不仅是布尔值，而且还指示参数是更具体类型的指示。

类型谓词通常用于检查已知为一个接口实例的对象是否是更具体接口的实例。

在这里，`StandupComedian`接口包含了`Comedian`之上的附加信息。`isStandupComedian`类型守卫可用于检查一般的`Comedian`是否特别是`StandupComedian`：

```
interface Comedian {
    funny: boolean;
}

interface StandupComedian extends Comedian {
    routine: string;
}

function isStandupComedian(value: Comedian): value is StandupComedian {
    return 'routine' in value;
}

function workWithComedian(value: Comedian) {
    if (isStandupComedian(value)) {
        // Type of value: StandupComedian
        console.log(value.routine); // Ok
    }

    // Type of value: Comedian
    console.log(value.routine);
    //                ~~~~~~~
    // Error: Property 'routine' does not exist on type 'Comedian'.
}
```

警告：因为类型谓词在 false 情况下也会缩小类型，如果类型谓词检查的不仅仅是其输入的类型，可能会得到令人惊讶的结果。

这个`isLongString`类型谓词在其`input`参数为`undefined`或长度小于`7`的`string`时返回`false`。因此，`else`语句（其 false 情况）被缩小为认为`text`必须是`undefined`类型：

```
function isLongString(input: string | undefined): input is string {
    return !!(input && input.length >= 7);
}

function workWithText(text: string | undefined) {
    if (isLongString(text)) {
        // Type of text: string
        console.log("Long text:", text.length);
    } else {
        // Type of text: undefined
        console.log("Short text:", text?.length);
        //                               ~~~~~~
        // Error: Property 'length' does not exist on type 'never'.
    }
}
```

做更多事情而不仅仅是验证属性或值类型的类型谓词很容易被误用。我通常建议尽可能避免使用它们。对于大多数情况，简单的类型谓词就足够了。

# 类型运算符

并非所有类型都可以仅使用关键字或现有类型的名称来表示。有时可能需要创建一个结合两者的新类型，对现有类型的属性进行一些转换。

## 键名

JavaScript 对象可以使用动态值检索成员，这些成员通常（但不一定）是 `string` 类型。在类型系统中表示这些键可能会很棘手。使用类似 `string` 的通用类型将允许容器值的无效键。

这就是为什么在使用更严格的配置设置时，TypeScript 会在下一个示例中报告 `ratings[key]` 的错误。`string` 类型允许不在 `Ratings` 接口上声明的属性，而 `Ratings` 没有声明索引签名来允许任何 `string` 键：

```
interface Ratings {
    audience: number;
    critics: number;
}

function getRating(ratings: Ratings, key: string): number {
    return ratings[key];
    //     ~~~~~~~~~~~
    // Error: Element implicitly has an 'any' type because expression
    // of type 'string' can't be used to index type 'Ratings'.
    //   No index signature with a parameter of
    //   type 'string' was found on type 'Ratings'.
}

const ratings: Ratings = { audience: 66, critic: 84 };

getRating(ratings, 'audience'); // Ok

getRating(ratings, 'not valid'); // Ok, but shouldn't be
```

另一个选项是使用文字的类型联合来允许的键。这将更准确地限制为仅容器值上存在的键：

```
function getRating(ratings: Ratings, key: 'audience' | 'critic'): number {
    return ratings[key]; // Ok
}

const ratings: Ratings = { audience: 66, critic: 84 };

getCountLiteral(ratings, 'audience'); // Ok

getCountLiteral(ratings, 'not valid');
//                       ~~~~~~~~~~~
// Error: Argument of type '"not valid"' is not
// assignable to parameter of type '"audience" | "critic"'.
```

但是，如果接口有几十个或更多成员怎么办？你必须将每个成员的键手动输入到联合类型中，并保持更新。真是太麻烦了。

TypeScript 提供了 `keyof` 操作符，它接受一个现有类型，并返回该类型上允许的所有键的联合。将其放置在类型名称的前面，可以在任何可能使用类型的地方，例如类型注释。

在这里，`keyof Ratings` 等同于 `'audience' | 'critic'`，但写出来更快，如果 `Ratings` 接口发生变化，也不需要手动更新：

```
function getCountKeyof(ratings: Ratings, key: keyof Ratings): number {
    return ratings[key]; // Ok
}

const ratings: Ratings = { audience: 66, critic: 84 };

getCountKeyof(ratings, 'audience'); // Ok

getCountKeyof(ratings, 'not valid');
//                     ~~~~~~~~~~~
// Error: Argument of type '"not valid"' is not
// assignable to parameter of type 'keyof Ratings'.
```

`keyof` 是一个很棒的功能，用于基于现有类型的键创建联合类型。它还与 TypeScript 中的其他类型操作符结合得很好，允许使用一些非常巧妙的模式，您将在本章和第十五章，“类型操作”中看到。

## typeof

TypeScript 提供的另一个类型操作符是 `typeof`。它返回提供值的类型。如果值的类型手动编写会很复杂，则这将非常有用。

在这里，`adaptation` 变量声明为与 `original` 相同的类型：

```
const original = {
    medium: "movie",
    title: "Mean Girls",
};

let adaptation: typeof original;

if (Math.random() > 0.5) {
    adaptation = { ...original, medium: "play" }; // Ok
} else {
    adaptation = { ...original, medium: 2 };
    //                          ~~~~~~
    // Error: Type 'number' is not assignable to type 'string'.
}
```

尽管 `typeof` *类型* 操作符在视觉上看起来像是 *运行时* 的 `typeof` 操作符，用于返回值类型的字符串描述，但它们是不同的。它们只是偶然使用相同的词。请记住：JavaScript 运算符是运行时运算符，返回值的类型字符串名称。因为 TypeScript 版本是类型运算符，只能用于类型，并不会出现在编译后的代码中。

### keyof typeof

`typeof` 可以检索值的类型，而 `keyof` 则检索类型上允许的键。TypeScript 允许这两个关键字链接在一起，以简洁地检索值类型的允许键。将它们结合起来，`typeof` 类型操作符在处理 `keyof` 类型操作时非常有用。

在此示例中，`logRating` 函数旨在接受 `ratings` 值类型的键之一。代码使用 `keyof typeof` 表示 `key` 必须是 `ratings` 值类型的键之一，而不是创建一个接口：

```
const ratings = {
    imdb: 8.4,
    metacritic: 82,
};

function logRating(key: keyof typeof ratings) {
    console.log(ratings[key]);
}

logRating("imdb"); // Ok

logRating("invalid");
//        ~~~~~~~~~
// Error: Argument of type '"missing"' is not assignable
// to parameter of type '"imdb" | "metacritic"'.
```

通过结合 `keyof` 和 `typeof`，我们可以避免编写并更新对象上允许的键的类型。

# 类型断言

当您的代码“强类型”时，TypeScript 的运作效果最佳：代码中的所有值都具有精确已知的类型。诸如顶级类型和类型守卫之类的功能提供了方法，将复杂的代码理顺，以便 TypeScript 的类型检查器能够理解。然而，有时不可能百分之百准确地告知类型系统代码的预期行为。

例如，`JSON.parse` 故意返回顶级类型 `any`。无法安全地通知类型系统给定给 `JSON.parse` 的特定字符串值应返回任何特定值类型。（正如我们将在第十章，“泛型”中看到的，为 `parse` 添加一个只用于返回类型的泛型类型将违反一个被称为泛型黄金法则的最佳实践。）

TypeScript 提供了一种语法来覆盖值类型系统理解的方式：“类型断言”，也称为“类型转换”。对于希望是不同类型的值，您可以使用 `as` 关键字后跟一个类型。TypeScript 将遵循您的断言，并将该值视为该类型。

在此片段中，`JSON.parse` 返回的结果可能是诸如 `string[]`、`[string, string]` 或 `["grace", "frankie"]` 之类的类型。片段使用类型断言将代码中的三行类型从 `any` 转换为这些类型之一：

```
const rawData = `["grace", "frankie"]`;

// Type: any
JSON.parse(rawData);

// Type: string[]
JSON.parse(rawData) as string[];

// Type: [string, string]
JSON.parse(rawData) as [string, string];

// Type: ["grace", "frankie"]
JSON.parse(rawData) as ["grace", "frankie"];
```

类型断言仅存在于 TypeScript 类型系统中。它们与编译为 JavaScript 时移除的所有其他类型系统语法一起被移除。编译为 JavaScript 后，前述代码将如下所示：

```
const rawData = `["grace", "frankie"]`;

// Type: any
JSON.parse(rawData);

// Type: string[]
JSON.parse(rawData);

// Type: [string, string]
JSON.parse(rawData);

// Type: ["grace", "frankie"]
JSON.parse(rawData);
```

###### 注意

如果您使用旧版本库或代码，则可能会看到不同的类型转换语法，看起来像 `<type>item` 而不是 `item as type`。因为此语法与 JSX 语法不兼容，因此在 *.tsx* 文件中无法使用，不建议使用。

TypeScript 的最佳实践通常是尽量避免使用类型断言。最好使您的代码完全类型化，不需要使用断言干预 TypeScript 对其类型的理解。但偶尔会有必要使用类型断言的情况。

## 断言捕获的错误类型

错误处理是另一个可能需要类型断言的地方。通常不可能知道 `catch` 块中捕获的错误类型是什么，因为 `try` 块中的代码可能会意外地抛出与预期不同的任何对象。此外，尽管 JavaScript 的最佳实践是始终抛出 `Error` 类的实例，但某些项目可能会抛出字符串文字或其他令人惊讶的值。

如果您绝对确定代码区域只会抛出`Error`类的实例，可以使用类型断言将捕获的断言视为`Error`。此代码片段访问了假定为`Error`类实例的捕获`error`的`message`属性：

```
try {
    // (code that may throw an error)
} catch (error) {
    console.warn("Oh no!", (error as Error).message);
}
```

通常更安全的做法是使用类型缩小的形式，比如使用`instanceof`检查来确保抛出的错误是预期的错误类型。此代码片段检查抛出的错误是否是`Error`类的实例，以决定是记录该消息还是记录错误本身：

```
try {
    // (code that may throw an error)
} catch (error) {
    console.warn("Oh no!", error instanceof Error ? error.message : error);
}
```

## 非空断言

类型断言的另一个常见用例是从只在理论上而非实际上可能包含`null`和/或`undefined`的变量中去除它们。这种情况非常普遍，TypeScript 包含了一个简写形式。不必编写`as`和排除`null`和`undefined`的完整类型，您可以使用`!`来表示相同的意思。换句话说，`!`非空断言断言类型不是`null`或`undefined`。

下面的两个类型断言在结果上是相同的，它们都产生`Date`而不是`Date | undefined`：

```
// Inferred type: Date | undefined
let maybeDate = Math.random() > 0.5
    ? undefined
    : new Date();

// Asserted type: Date
maybeDate as Date;

// Asserted type: Date
maybeDate!;
```

非空断言在诸如`Map.get`这样的 API 中特别有用，如果不存在，则返回值或`undefined`。

在这里，`seasonCounts`是一个通用的`Map<string, number>`。我们知道它包含一个`"I Love Lucy"`键，所以`knownValue`变量可以使用`!`来从其类型中删除`| undefined`：

```
const seasonCounts = new Map([
    ["I Love Lucy", 6],
    ["The Golden Girls", 7],
]);

// Type: string | undefined
const maybeValue = seasonCounts.get("I Love Lucy");

console.log(maybeValue.toUpperCase());
//          ~~~~~~~~~~
// Error: Object is possibly 'undefined'.

// Type: string
const knownValue = seasonCounts.get("I Love Lucy")!;

console.log(knownValue.toUpperCase()); // Ok
```

## 类型断言注意事项

类型断言，就像`any`类型一样，是 TypeScript 类型系统的一个必要的逃生口。因此，也像`any`类型一样，在合理的情况下应尽量避免使用。通常更好的做法是使用更准确的类型来表示您的代码，而不是为了断言值的类型而使其变得更容易。这些断言通常是错误的——要么在编写时已经错误，要么随着代码库的变化后来变得错误。

例如，假设`seasonCounts`示例随时间改变而在映射中具有不同的值。其非空断言可能仍然使代码通过 TypeScript 类型检查，但可能会导致运行时错误：

```
const seasonCounts = new Map([
    ["Broad City", 5],
    ["Community", 6],
]);

// Type: string
const knownValue = seasonCounts.get("I Love Lucy")!;

console.log(knownValue.toUpperCase()); // No type error, but...
// Runtime TypeError: Cannot read property 'toUpperCase' of undefined.
```

一般来说，应该节制地使用类型断言，只有在确保安全的情况下才使用。

### 断言与声明

在声明变量类型与使用类型断言来改变具有初始值的变量类型之间存在差异。当变量的初始值和变量的类型注释同时存在时，TypeScript 的类型检查器会执行可赋值性检查。然而，类型断言明确告诉 TypeScript 跳过一些类型检查。

下面的代码创建了两个具有相同缺陷的`Entertainer`类型的对象：缺少`acts`成员。 TypeScript 能够在`declared`变量中捕获错误，因为它具有`: Entertainer`类型注释。 由于类型断言，它无法在`asserted`变量上捕获错误：

```
interface Entertainer {
    acts: string[];
    name: string;
}

const declared: Entertainer = {
    name: "Moms Mabley",
};
// Error: Property 'acts' is missing in type
// '{ one: number; }' but required in type 'Entertainer'.

const asserted = {
    name: "Moms Mabley",
} as Entertainer; // Ok, but...

// Both of these statements would fail at runtime with:
// Runtime TypeError: Cannot read properties of undefined (reading 'toPrecision')
console.log(declared.acts.join(", "));
console.log(asserted.acts.join(", "));
```

因此，强烈建议要么使用类型注释，要么允许 TypeScript 从其初始值推断变量的类型。

### 断言可赋值性

类型断言仅用于一小部分情况，即某些值的类型略有错误。如果类型断言涉及两种完全不相关的类型，则 TypeScript 将会注意并报告类型错误。

例如，不允许从一个基本类型切换到另一个基本类型，因为基本类型彼此无关：

```
let myValue = "Stella!" as number;
//            ~~~~~~~~~~~~~~~~~~~
// Error: Conversion of type 'string' to type 'number'
// may be a mistake because neither type sufficiently
// overlaps with the other. If this was intentional,
// convert the expression to 'unknown' first.
```

如果您绝对必须将一个值从一种类型切换到完全不相关的另一种类型，则可以使用双重类型断言。 首先将值强制转换为顶级类型 —— `any`或`unknown` —— 然后将该结果转换为不相关类型：

```
let myValueDouble = "1337" as unknown as number; // Ok, but... eww.
```

`as unknown as...`双重类型断言是危险的，并且几乎总是代码类型错误的标志。在代码周围使用它们作为逃逸舱从类型系统中，意味着当周围代码发生变化导致先前工作的代码出现问题时，类型系统可能无法帮助您。我只是将双重类型断言作为一个警示故事来帮助解释类型系统，并不鼓励它们的使用。

# Const 断言

回到第四章，“对象”，我介绍了一个`as const`语法，用于将可变数组类型更改为只读元组类型，并承诺在书中的其他地方更多地使用它。 现在是时候了！

Const 断言通常可用于指示任何值 —— 数组、基本类型、值，你怎么称呼它 —— 应被视为不可变版本的常量。 具体来说，`as const`将以下三条规则应用于接收到的任何类型：

+   数组被视为`readonly`元组，而不是可变数组。

+   字面量被视为字面量，而不是它们的一般基本类型的等价物。

+   对象上的属性被视为`readonly`。

您已经看到数组变为元组，如将此数组断言为元组：

```
// Type: (number | string)[]
[0, ''];

// Type: readonly [0, '']
[0, ''] as const;
```

让我们深入了解`as const`产生的另外两个变化。

## 字面量到基本类型

让类型系统理解字面值为特定的字面量值，而不是扩展为其一般的基本类型，可能会很有用。

例如，类似于返回元组的函数，一个函数可能会返回一个特定的字面值，而不是一般的基本类型。 这些函数还返回可以更为具体的值 —— 在这里，`getNameConst`的返回类型是更具体的`"Maria Bamford"`，而不是一般的`string`：

```
// Type: () => string
const getName = () => "Maria Bamford";

// Type: () => "Maria Bamford"
const getNameConst = () => "Maria Bamford" as const;
```

在值上具体字段可能也是有用的。许多流行的库要求值上的判别字段是特定的文字，以便它们的代码类型可以更具体地推断值。在这里，`narrowJoke`变量具有类型`"one-liner"`的`style`，而不是`string`类型，因此它可以在需要`Joke`类型的位置提供：

```
interface Joke {
    quote: string;
    style: "story" | "one-liner";
}

function tellJoke(joke: Joke) {
    if (joke.style === "one-liner") {
        console.log(joke.quote);
    } else {
        console.log(joke.quote.split("\n"));
    }
}

// Type: { quote: string; style: "one-liner" }
const narrowJoke = {
    quote: "If you stay alive for no other reason do it for spite.",
    style: "one-liner" as const,
};

tellJoke(narrowJoke); // Ok

// Type: { quote: string; style: string }
const wideObject = {
    quote: "Time flies when you are anxious!",
    style: "one-liner",
};

tellJoke(wideObject);
// Error: Argument of type '{ quote: string; style: string; }'
// is not assignable to parameter of type 'LogAction'.
//   Types of property 'style' are incompatible.
//     Type 'string' is not assignable to type '"story" | "one-liner"'.
```

## 只读对象

对象文字，例如用作变量初始值的值通常会扩展属性的类型，就像`let`变量的初始值扩展一样。例如，字符串值如`'apple'`会变成基本类型`string`，数组被类型化为数组而不是元组，等等。当这些值中的一些或全部后来被用于需要其特定文字类型的地方时，这可能会不方便。

使用`as const`断言一个值的文字，然而，将推断类型转换为尽可能具体。所有成员属性变为`readonly`，文字被视为其自身的文字类型而不是它们的一般基本类型，数组变为只读元组，依此类推。换句话说，对值的文字应用 const 断言使得该值不可变，并递归地将相同的 const 断言逻辑应用于其所有成员。

例如，后续的`preferencesMutable`值是在没有`as const`声明的情况下声明的，因此其名称是原始类型`string`，并且允许修改。然而，`favoritesConst`使用`as const`声明，因此其成员值是文字类型而不允许修改：

```
function describePreference(preference: "maybe" | "no" | "yes") {
    switch (preference) {
        case "maybe":
            return "I suppose...";
        case "no":
            return "No thanks.";
        case "yes":
            return "Yes please!";
    }
}

// Type: { movie: string, standup: string }
const preferencesMutable = {
    movie: "maybe"
    standup: "yes",
};

describePreference(preferencesMutable.movie);
//                 ~~~~~~~~~~~~~~~~~~~~~~~~
// Error: Argument of type 'string' is not assignable
// to parameter of type '"maybe" | "no" | "yes"'.

preferencesMutable.movie = "no"; // Ok

// Type: readonly { readonly movie: "maybe", readonly standup: "yes" }
const preferencesReadonly = {
    movie: "maybe"
    standup: "yes",
} as const;

describePreference(preferencesReadonly.movie); // Ok

preferencesReadonly.movie = "no";
//                  ~~~~~
// Error: Cannot assign to 'movie' because it is a read-only property.
```

# 概要

在本章中，您使用类型修饰符将现有对象和/或类型转换为新类型：

+   顶级类型：高度宽容的`any`和高度限制的`unknown`

+   类型操作符：使用`keyof`来获取类型的键和/或`typeof`来获取值的类型

+   使用和不使用类型断言来偷偷地改变值的类型

+   使用`as const`断言来缩小类型

###### Tip

现在您已经完成了本章的阅读，请在[*https://learningtypescript.com/type-modifiers*](https://learningtypescript.com/type-modifiers)上练习所学内容。

> 为什么文字类型如此固执呢？
> 
> 它思想狭隘。
