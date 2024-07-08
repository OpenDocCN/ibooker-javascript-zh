# 第六章：数组

> 数组和元组
> 
> 一个灵活一个固定
> 
> 选择您的冒险

JavaScript 数组非常灵活，可以容纳任意混合的值：

```
const elements = [true, null, undefined, 42];

elements.push("even", ["more"]);
// Value of elements: [true, null, undefined, 42, "even", ["more"]]
```

大多数情况下，单个 JavaScript 数组的目的是只存储特定类型的值。添加不同类型的值可能会令读者困惑，或者更糟糕的是，可能导致程序错误。

TypeScript 遵循保持每个数组一个数据类型的最佳实践，通过记住数组初始时包含的数据类型，并仅允许数组在该数据类型上操作来实现。

在这个例子中，TypeScript 知道 `warriors` 数组最初包含 `string` 类型的值，因此允许添加更多的 `string` 类型的值，但不允许添加其他类型的数据：

```
const warriors = ["Artemisia", "Boudica"];

// Ok: "Zenobia" is a string
warriors.push("Zenobia");

warriors.push(true);
//            ~~~~
// Argument of type 'boolean' is not assignable to parameter of type 'string'.
```

您可以将 TypeScript 对数组类型的推断视为类似于它如何从初始成员理解变量类型。TypeScript 通常尝试从分配值的方式理解代码的预期类型，数组也不例外。

# 数组类型

与其他变量声明一样，用于存储数组的变量不需要具有初始值。变量可以从 `undefined` 开始，并稍后接收数组值。

TypeScript 希望您通过给变量添加类型注解来告知它应将什么类型的值放入数组中。数组的类型注解要求数组中元素的类型，后跟一个 `[]`：

```
let arrayOfNumbers: number[];

arrayOfNumbers = [4, 8, 15, 16, 23, 42];
```

###### 注意

数组类型也可以像 `Array<number>` 这样的语法写成 *类泛型*。大多数开发者更喜欢简单的 `number[]`。类在 第八章，“类” 中讨论，泛型在 第九章，“类型修饰符” 中讨论。

## 数组和函数类型

数组类型是一个语法容器的示例，其中函数类型可能需要使用括号来区分函数类型中的内容。括号可以用来指示注解的哪一部分是函数返回或周围的数组类型。

此处的 `createStrings` 类型是一个函数类型，与 `stringCreators`（数组类型）不同：

```
// Type is a function that returns an array of strings
let createStrings: () => string[];

// Type is an array of functions that each return a string
let stringCreators: (() => string)[];
```

## 联合类型数组

您可以使用联合类型来表示数组的每个元素可以是多种选择类型中的一种。

在使用联合类型数组时，可能需要使用括号来指示注解的哪一部分是数组的内容或周围的联合类型。在数组联合类型中使用括号很重要——以下两种类型不同：

```
// Type is either a number or an array of strings
let stringOrArrayOfNumbers: string | number[];

// Type is an array of elements that are each either a number or a string
let arrayOfStringOrNumbers: (string | number)[];
```

TypeScript 将从数组的声明中理解为联合类型数组，如果数组中包含多种类型的元素。换句话说，数组元素的类型是数组中所有可能元素类型的并集。

在这里，`namesMaybe` 是 `(string | undefined)[]`，因为它既有 `string` 值，也有 `undefined` 值：

```
// Type is (string | undefined)[]
const namesMaybe = [
  "Aqualtune",
  "Blenda",
  undefined,
];
```

## 发展任意数组

如果你在初始设置为空数组的变量上没有包含类型注解，TypeScript 将把该数组视为逐步演化为`any[]`，这意味着它可以接收任何内容。和逐步演化的`any`变量一样，我们不喜欢逐步演化的`any[]`数组。它们部分抵消了 TypeScript 类型检查器的好处，因为允许你添加潜在不正确的值。

这个`values`数组最初包含`any`元素，然后演变为包含`string`元素，然后再次演变为包含`number | string`元素：

```
// Type: any[]
let values = [];

// Type: string[]
values.push('');

// Type: (number | string)[]
values[0] = 0;
```

和变量一样，允许数组成为逐步演化的`any`类型——以及通常使用`any`类型——部分地削弱了 TypeScript 类型检查的目的。当 TypeScript 知道你的值应该是什么类型时，它的工作效果最佳。

## 多维数组

一个 2D 数组，或者说是数组的数组，会有两个“[]”：

```
let arrayOfArraysOfNumbers: number[][];

arrayOfArraysOfNumbers = [
  [1, 2, 3],
  [2, 4, 6],
  [3, 6, 9],
];
```

一个 3D 数组，或者说是数组的数组的数组，会有三个“[]”。4D 数组有四个“[]”。5D 数组有五个“[]”。你可以猜想，对于 6D 数组及以上情况也是如此。

这些多维数组类型并没有为数组类型引入任何新的概念。将 2D 数组视为接受原始类型，只是在末尾添加了`[]`。

这个`arrayOfArraysOfNumbers`数组的类型是`number[][]`，也可以表示为`(number[])[]`：

```
// Type: number[][]
let arrayOfArraysOfNumbers: (number[])[];
```

# 数组成员

TypeScript 理解通过典型的基于索引的访问来检索数组成员，以返回该数组类型的元素。

这个`defenders`数组的类型是`string[]`，所以`defender`是一个`string`：

```
const defenders = ["Clarenza", "Dina"];

// Type: string
const defender = defenders[0];
```

联合类型数组的成员本身是相同的联合类型。

在这里，`soldiersOrDates`的类型是`(string | Date)[]`，因此`soldierOrDate`变量的类型是`string | Date`：

```
const soldiersOrDates = ["Deborah Sampson", new Date(1782, 6, 3)];

// Type: Date | string
const soldierOrDate = soldiersOrDates[0];
```

## 注意：不完备成员

TypeScript 类型系统被认为在技术上是*不完备*的：它可以大部分时间得到正确的类型，但有时它对值的类型理解可能是错误的。特别是数组在类型系统中是不完备性的源头。默认情况下，TypeScript 假设所有数组成员访问都会返回该数组的成员，即使在 JavaScript 中，使用大于数组长度的索引访问数组元素会返回`undefined`。

这段代码在默认的 TypeScript 编译器设置下没有任何投诉：

```
function withElements(elements: string[]) {
  console.log(elements[9001].length); // No type error
}

withElements(["It's", "over"]);
```

作为读者，我们可以推断它会在运行时崩溃，显示“`Cannot read property 'length' of undefined`”，但 TypeScript 故意不确保检索到的数组成员存在。它在代码片段中看到`elements[9001]`被视为`string`类型，而不是`undefined`类型。

###### 注意

TypeScript 确实有一个`--noUncheckedIndexedAccess`标志，使得数组查找更加受限和类型安全，但它相当严格，大多数项目不使用它。我在本书中没有涵盖它。第十二章，“使用 IDE 功能”链接到解释 TypeScript 所有配置选项深入的资源。

# 展开和剩余参数

还记得来自第五章，“函数”的`...` rest 参数吗？`...`操作符的 rest 参数和数组展开是与 JavaScript 中数组交互的关键方式。 TypeScript 理解这两者。

## Spreads

使用`...`展开操作符可以将数组连接在一起。TypeScript 理解结果数组将包含可以来自任一输入数组的值。

如果输入数组是相同类型，则输出数组将是相同类型。如果将两个不同类型的数组一起展开以创建新数组，则新数组将被理解为原始两种类型的并集数组。

在这里，`conjoined`数组已知包含类型为`string`和`number`的值，因此其类型被推断为`(string | number)[]`：

```
// Type: string[]
const soldiers = ["Harriet Tubman", "Joan of Arc", "Khutulun"];

// Type: number[]
const soldierAges = [90, 19, 45];

// Type: (string | number)[]
const conjoined = [...soldiers, ...soldierAges];
```

## 展开 rest 参数

TypeScript 识别并将对 JavaScript 中`...`展开数组作为 rest 参数的做法进行类型检查。用作 rest 参数的数组必须具有与 rest 参数相同的数组类型。

下面的`logWarriors`函数仅接受其`...names` rest 参数的`string`值。允许展开`string[]`类型的数组，但不允许`number[]`：

```
function logWarriors(greeting: string, ...names: string[]) {
  for (const name of names) {
    console.log(`${greeting}, ${name}!`);
  }
}

const warriors = ["Cathay Williams", "Lozen", "Nzinga"];

logWarriors("Hello", ...warriors);

const birthYears = [1844, 1840, 1583];

logWarriors("Born in", ...birthYears);
//                     ~~~~~~~~~~~~~
// Error: Argument of type 'number' is not
// assignable to parameter of type 'string'.
```

# 元组

虽然 JavaScript 数组在理论上可以是任意大小，但有时使用固定大小的数组也很有用，也称为*元组*。元组数组在每个索引处具有特定的已知类型，可能比数组所有可能成员的联合类型更具体。声明元组类型的语法类似于数组文字，但在元素值的位置上有类型。

在这里，数组`yearAndWarrior`被声明为具有索引 0 处为`number`，索引 1 处为`string`的元组类型：

```
let yearAndWarrior: [number, string];

yearAndWarrior = [530, "Tomyris"]; // Ok

yearAndWarrior = [false, "Tomyris"];
//                ~~~~~
// Error: Type 'boolean' is not assignable to type 'number'.

yearAndWarrior = [530];
// Error: Type '[number]' is not assignable to type '[number, string]'.
//   Source has 1 element(s) but target requires 2.
```

元组经常与数组解构一起在 JavaScript 中使用，以便根据单个条件设置两个变量的初始值。

例如，TypeScript 在这里认识到`year`将始终是`number`，而`warrior`将始终是`string`：

```
// year type: number
// warrior type: string
let [year, warrior] = Math.random() > 0.5
  ? [340, "Archidamia"]
  : [1828, "Rani of Jhansi"];
```

## 元组可赋值性

TypeScript 将元组类型视为比可变长度数组类型更具体。这意味着可变长度数组类型不能赋值给元组类型。

在这里，虽然我们人类可能会认为`pairLoose`内部有`[boolean, number]`，但 TypeScript 推断它为更一般的`(boolean | number)[]`类型：

```
// Type: (boolean | number)[]
const pairLoose = [false, 123];

const pairTupleLoose: [boolean, number] = pairLoose;
//    ~~~~~~~~~~~~~~
// Error: Type '(number | boolean)[]' is not
// assignable to type '[boolean, number]'.
//   Target requires 2 element(s) but source may have fewer.
```

如果`pairLoose`本身被声明为`[boolean, number]`，则允许将其值分配给`pairTuple`。

不同长度的元组也不能相互赋值，因为 TypeScript 在元组类型中包含了知道元组中有多少成员。

在这里，`tupleTwoExtra`必须具有精确两个成员，因此虽然`tupleThree`以正确的成员开始，但其第三个成员阻止了其分配给`tupleTwoExtra`：

```
const tupleThree: [boolean, number, string] = [false, 1583, "Nzinga"];

const tupleTwoExact: [boolean, number] = [tupleThree[0], tupleThree[1]];

const tupleTwoExtra: [boolean, number] = tupleThree;
//    ~~~~~~~~~~~~~
// Error: Type '[boolean, number, string]' is
// not assignable to type '[boolean, number]'.
//   Source has 3 element(s) but target allows only 2.
```

### Tuples as rest parameters

因为元组被视为具有更多特定类型信息的数组，包括长度和元素类型，它们可以特别有用于存储要传递给函数的参数。对于作为 `...` 剩余参数传递的元组，TypeScript 能够提供准确的类型检查。

在这里，`logPair` 函数的参数被类型化为 `string` 和 `number`。尝试将类型为 `(string | number)[]` 的值作为参数传入是不安全的，因为内容可能不匹配：它们可能是相同类型，或者一个是每种类型的错误顺序。然而，如果 TypeScript 知道该值是 `[string, number]` 元组，它就理解值是匹配的：

```
function logPair(name: string, value: number) {
  console.log(`${name} has ${value}`);
}

const pairArray = ["Amage", 1];

logPair(...pairArray);
// Error: A spread argument must either have a
// tuple type or be passed to a rest parameter.

const pairTupleIncorrect: [number, string] = [1, "Amage"];

logPair(...pairTupleIncorrect);
// Error: Argument of type 'number' is not
// assignable to parameter of type 'string'.

const pairTupleCorrect: [string, number] = ["Amage", 1];

logPair(...pairTupleCorrect); // Ok
```

如果您确实希望对剩余参数元组进行大胆尝试，您可以将它们与数组混合使用，以存储多个函数调用的参数列表。在这里，`trios` 是一个元组数组，其中每个元组的第二个成员也是一个元组。`trios.forEach(trio => logTrio(...trio))` 是安全的，因为每个 `...trio` 恰好匹配 `logTrio` 的参数类型。然而，`trios.forEach(logTrio)` 是不可分配的，因为它试图将整个 `[string, [number, boolean]` 作为第一个参数传递，这是类型 `string`：

```
function logTrio(name: string, value: [number, boolean]) {
  console.log(`${name} has ${value[0]} (${value[1]}`);
}

const trios: [string, [number, boolean]][] = [
  ["Amanitore", [1, true]],
  ["Æthelflæd", [2, false]],
  ["Ann E. Dunwoody", [3, false]]
];

trios.forEach(trio => logTrio(...trio)); // Ok

trios.forEach(logTrio);
//            ~~~~~~~
// Argument of type '(name: string, value: [number, boolean]) => void'
// is not assignable to parameter of type
// '(value: [string, [number, boolean]], ...) => void'.
//   Types of parameters 'name' and 'value' are incompatible.
//     Type '[string, [number, boolean]]' is not assignable to type 'string'.
```

## 元组推断

TypeScript 通常将创建的数组视为可变长度数组，而不是元组。如果看到一个数组被用作变量的初始值或函数的返回值，那么它将假定为灵活大小的数组，而不是固定大小的元组。

下面的 `firstCharAndSize` 函数被推断为返回 `(string | number)[]`，而不是 `[string, number]`，因为这是对其返回的数组文字推断的类型：

```
// Return type: (string | number)[]
function firstCharAndSize(input: string) {
  return [input[0], input.length];
}

// firstChar type: string | number
// size type: string | number
const [firstChar, size] = firstCharAndSize("Gudit");
```

在 TypeScript 中有两种常见的方法来指示一个值应该是一个更具体的元组类型而不是一般的数组类型：显式元组类型和 `const` 断言。

### 显式元组类型

元组类型可以在类型注释中使用，例如函数的返回类型注释。如果函数被声明为返回元组类型并返回数组文字，那么该数组文字将被推断为元组而不是更一般的可变长度数组。

此 `firstCharAndSizeExplicit` 函数版本明确声明它返回一个 `string` 和 `number` 的元组：

```
// Return type: [string, number]
function firstCharAndSizeExplicit(input: string): [string, number] {
  return [input[0], input.length];
}

// firstChar type: string
// size type: number
const [firstChar, size] = firstCharAndSizeExplicit("Cathay Williams");
```

### Const 断言元组

在显式类型注释中键入元组类型可能会因为与任何显式类型注释键入相同的原因而让人感到头疼。这是额外的语法需要您在代码更改时编写和更新。

作为替代方案，TypeScript 提供了一个称为 *const 断言* 的 `as const` 操作符，可以放置在一个值之后。Const 断言告诉 TypeScript 在推断其类型时使用最字面、最只读的可能形式。如果在数组文字之后放置一个 const 断言，它将指示该数组应该被视为元组：

```
// Type: (string | number)[]
const unionArray = [1157, "Tomoe"];

// Type: readonly [1157, "Tomoe"]
const readonlyTuple = [1157, "Tomoe"] as const;
```

请注意，`as const` 断言不仅仅是将灵活大小的数组转换为固定大小的元组：它们还指示 TypeScript 元组是只读的，不能在期望修改值的地方使用。

在这个例子中，`pairMutable` 允许修改，因为它具有传统的显式元组类型。然而，`as const` 使得不可修改的值不能赋给可变的 `pairAlsoMutable`，并且常量 `pairConst` 的成员也不允许修改：

```
const pairMutable: [number, string] = [1157, "Tomoe"];
pairMutable[0] = 1247; // Ok

const pairAlsoMutable: [number, string] = [1157, "Tomoe"] as const;
//    ~~~~~~~~~~~~~~~
// Error: The type 'readonly [1157, "Tomoe"]' is 'readonly'
// and cannot be assigned to the mutable type '[number, string]'.

const pairConst = [1157, "Tomoe"] as const;
pairConst[0] = 1247;
//        ~
// Error: Cannot assign to '0' because it is a read-only property.
```

在实践中，只读元组对于函数返回值非常方便。从返回元组的函数中返回的值通常会立即解构，因此只读元组不会妨碍函数的使用。

这个 `firstCharAndSizeAsConst` 返回一个 `readonly [string, number]`，但是消费代码只关心从元组中检索值：

```
// Return type: readonly [string, number]
function firstCharAndSizeAsConst(input: string) {
  return [input[0], input.length] as const;
}

// firstChar type: string
// size type: number
const [firstChar, size] = firstCharAndSizeAsConst("Ching Shih");
```

###### 注意

只读对象和 `as const` 断言在第九章，“类型修饰符”中有更详细的介绍。

# 摘要

在本章中，您将学习如何声明数组并检索其成员：

+   使用 `[]` 声明数组类型

+   使用括号声明函数数组或联合类型

+   TypeScript 如何理解数组元素作为数组类型的一部分

+   使用 `...` 展开和剩余参数

+   声明元组类型以表示固定大小的数组

+   使用类型注解或 `as const` 断言来创建元组

###### 提示

现在您已经完成了本章的阅读，请在[*https://learningtypescript.com/arrays*](https://learningtypescript.com/arrays)上实践所学内容。

> 海盗最喜欢的数据结构是什么？
> 
> 阵列！
