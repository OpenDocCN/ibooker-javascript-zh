# 第五章：函数

> 函数参数
> 
> 从一端进，从另一端出
> 
> 作为返回类型

在 第二章，“类型系统” 中，你了解了如何使用类型注解注释变量的值。现在，你将看到如何对函数参数和返回类型进行相同操作——以及为什么这样做很有用。

# 函数参数

看下面的 `sing` 函数，它接受一个 `song` 参数并将其记录下来：

```
function sing(song) {
  console.log(`Singing: ${song}!`);
}
```

开发 `sing` 函数的开发者意图为 `song` 参数提供什么值类型？

这是一个 `string` 吗？还是一个带有覆盖的 `toString()` 方法的对象？这段代码有 bug 吗？*谁知道呢？！*

如果没有显式声明类型信息，我们可能永远不会知道——TypeScript 将其视为 `any` 类型，这意味着参数的类型可以是任何东西。

与变量类似，TypeScript 允许你使用类型注解声明函数参数的类型。现在我们可以使用 `: string` 告诉 TypeScript `song` 参数是 `string` 类型：

```
function sing(song: string) {
  console.log(`Singing: ${song}!`);
}
```

更好：现在我们知道 `song` 是什么类型了！

注意，你不需要为函数参数添加正确的类型注解使代码成为有效的 TypeScript 语法。TypeScript 可能会报错，但生成的 JavaScript 仍然会运行。前面的代码片段缺少对 `song` 参数的类型声明仍会从 TypeScript 转换为 JavaScript。第十三章，“配置选项” 将讨论如何配置 TypeScript 对隐式为 `any` 类型的参数（如 `song`）的投诉方式。

## 必需参数

与 JavaScript 不同，它允许调用带有任意数量参数的函数，TypeScript 假定在函数上声明的所有参数都是必需的。如果使用错误数量的参数调用函数，TypeScript 将以类型错误的形式抗议。TypeScript 的参数计数将在函数调用时出现参数数目不足或过多的情况。

这个 `singTwo` 函数需要两个参数，因此传递一个参数和传递三个参数都是不允许的：

```
function singTwo(first: string, second: string) {
  console.log(`${first} / ${second}`);
}

// Logs: "Ball and Chain / undefined"
singTwo("Ball and Chain");
//      ~~~~~~~~~~~~~~~~
// Error: Expected 2 arguments, but got 1.

// Logs: "I Will Survive / Higher Love"
singTwo("I Will Survive", "Higher Love"); // Ok

// Logs: "Go Your Own Way / The Chain"
singTwo("Go Your Own Way", "The Chain", "Dreams");
//                                      ~~~~~~~~
// Error: Expected 2 arguments, but got 3.
```

强制要求函数提供必需的参数有助于通过确保所有预期的参数值存在于函数内部来强制执行类型安全性。未能确保这些值存在可能会导致代码中的意外行为，例如前面的 `singTwo` 函数记录 `undefined` 或忽略参数。

###### 注意

*参数* 指的是函数声明期望接收的参数。*参数值* 是在函数调用中提供给参数的值。在前面的例子中，`first` 和 `second` 是参数，而像 `"Dreams"` 这样的字符串是参数值。

## 可选参数

请记住，在 JavaScript 中，如果未提供函数参数，则函数内部的参数值默认为 `undefined`。有时不需要提供函数参数，函数的预期用途是该 `undefined` 值。我们不希望 TypeScript 报告未提供这些可选参数的参数错误。 TypeScript 允许通过在类型标注中的 `:` 前面添加 `?` 将参数标注为可选，这与可选对象类型属性类似。

可选参数不需要在函数调用时提供。因此，它们的类型总是添加 `| undefined` 作为联合类型。

在下面的 `announceSong` 函数中，`singer` 参数被标记为可选。它的类型是 `string | undefined`，并且不需要由函数调用者提供。如果提供了 `singer`，它可以是 `string` 类型或 `undefined`：

```
function announceSong(song: string, singer?: string) {
  console.log(`Song: ${song}`);

  if (singer) {
    console.log(`Singer: ${singer}`);
  }
}

announceSong("Greensleeves"); // Ok
announceSong("Greensleeves", undefined); // Ok
announceSong("Chandelier", "Sia"); // Ok
```

这些可选参数始终隐式可以是 `undefined`。在前面的代码中，`singer` 最初被视为 `string | undefined` 类型，然后由 `if` 语句缩小为仅 `string` 类型。

可选参数与包括 `| undefined` 的联合类型的参数不同。没有用 `?` 标记为可选的参数必须始终提供，即使值明确为 `undefined`。

在 `announceSongBy` 函数中，`singer` 参数必须显式提供。它可以是 `string` 类型或 `undefined`：

```
function announceSongBy(song: string, singer: string | undefined) { /* ... */ }

announceSongBy("Greensleeves");
// Error: Expected 2 arguments, but got 1.

announceSongBy("Greensleeves", undefined); // Ok
announceSongBy("Chandelier", "Sia"); // Ok
```

函数的任何可选参数必须是最后的参数。将可选参数放在必需参数之前会触发 TypeScript 语法错误：

```
function announceSinger(singer?: string, song: string) {}
//                                       ~~~~
// Error: A required parameter cannot follow an optional parameter.
```

## 默认参数

JavaScript 中的可选参数可以在其声明中用 `=` 和一个值给出默认值。对于这些可选参数，因为默认情况下提供了一个值，它们的 TypeScript 类型在函数内部不会隐式添加 `| undefined` 联合。 TypeScript 仍然允许以缺少或 `undefined` 参数调用这些参数的函数。

TypeScript 的类型推断对默认函数参数值的工作方式与对初始变量值的工作方式类似。如果参数有默认值并且没有类型标注，TypeScript 将根据默认值推断参数的类型。

在下面的 `rateSong` 函数中，`rating` 被推断为 `number` 类型，但在调用函数的代码中是可选的 `number | undefined`：

```
function rateSong(song: string, rating = 0) {
  console.log(`${song} gets ${rating}/5 stars!`);
}

rateSong("Photograph"); // Ok
rateSong("Set Fire to the Rain", 5); // Ok
rateSong("Set Fire to the Rain", undefined); // Ok

rateSong("At Last!", "100");
//                   ~~~~~
// Error: Argument of type '"100"' is not assignable
// to parameter of type 'number | undefined'.
```

## 剩余参数

JavaScript 中的某些函数可以用任意数量的参数调用。可以在函数声明的最后一个参数上放置 `...` 展开操作符，以指示从该参数开始传递给函数的所有“剩余”参数应该存储在一个单独的数组中。

TypeScript 允许声明这些剩余参数的类型方式与常规参数类似，只是在末尾添加了 `[]` 语法，以指示它是一个参数数组。

在这里，`singAllTheSongs`可以接受零个或多个类型为`string`的参数作为它的`songs`剩余参数：

```
function singAllTheSongs(singer: string, ...songs: string[]) {
  for (const song of songs) {
    console.log(`${song}, by ${singer}`);
  }
}

singAllTheSongs("Alicia Keys"); // Ok
singAllTheSongs("Lady Gaga", "Bad Romance", "Just Dance", "Poker Face"); // Ok

singAllTheSongs("Ella Fitzgerald", 2000);
//                                 ~~~~
// Error: Argument of type 'number' is not
// assignable to parameter of type 'string'.
```

我将在第六章，“数组”中介绍在 TypeScript 中处理数组的方法。

# 返回类型

TypeScript 是有洞察力的：如果它理解函数可能返回的所有可能值，它将知道函数的返回类型。在这个例子中，TypeScript 理解`singSongs`返回一个`number`：

```
// Type: (songs: string[]) => number
function singSongs(songs: string[]) {
  for (const song of songs) {
    console.log(`${song}`);
  }

  return songs.length;
}
```

如果一个函数包含多个不同值的`return`语句，TypeScript 将推断返回类型为所有可能返回类型的联合。

这个`getSongAt`函数将被推断返回`string | undefined`，因为它的两个可能返回值分别是`string`和`undefined`：

```
// Type: (songs: string[], index: number) => string | undefined
function getSongAt(songs: string[], index: number) {
  return index < songs.length
    ? songs[index]
    : undefined;
}
```

## 显式返回类型

与变量一样，我通常建议不必为具有类型注释的函数显式声明返回类型。然而，有几种情况下这样做可能会有用：

+   你可能希望强制函数返回多种可能值时始终返回相同类型的值。

+   TypeScript 将拒绝尝试推理递归函数的返回类型。

+   它可以加快在非常庞大的项目中 TypeScript 的类型检查速度——例如那些包含数百个 TypeScript 文件或更多的项目。

函数声明的返回类型注释放置在参数列表后的`)`之后。

对于函数声明，这是在`{`之前：

```
function singSongsRecursive(songs: string[], count = 0): number {
  return songs.length ? singSongsRecursive(songs.slice(1), count + 1) : count;
}
```

对于箭头函数（也称为 lambda 函数），这是在`=>`之前：

```
const singSongsRecursive = (songs: string[], count = 0): number =>
  songs.length ? singSongsRecursive(songs.slice(1), count + 1) : count;
```

如果函数中的`return`语句返回一个不可分配给函数返回类型的值，TypeScript 将会给出一个可分配性的投诉。

在这里，`getSongRecordingDate`函数明确声明为返回`Date | undefined`，但其中一个返回语句错误地提供了一个`string`：

```
function getSongRecordingDate(song: string): Date | undefined {
  switch (song) {
    case "Strange Fruit":
      return new Date('April 20, 1939'); // Ok

    case "Greensleeves":
      return "unknown";
      // Error: Type 'string' is not assignable to type 'Date'.

    default:
      return undefined; // Ok
  }
}
```

# 函数类型

JavaScript 允许我们将函数作为值传递。这意味着我们需要一种声明参数或变量类型的方式，以便保存函数。

函数类型的语法看起来类似于箭头函数，但其主体是一个类型而不是一个实现。

这个`nothingInGivesString`变量的类型描述了一个没有参数并返回`string`值的函数：

```
let nothingInGivesString: () => string;
```

这个`inputAndOutput`变量的类型描述了一个带有`string[]`参数、可选的`count`参数以及返回`number`值的函数：

```
let inputAndOutput: (songs: string[], count?: number) => number;
```

函数类型经常用于描述回调参数（意味着将作为函数调用的参数）。

例如，以下`runOnSongs`片段声明了它的`getSongAt`参数的类型为一个接受`index: number`并返回`string`的函数。传递`getSongAt`符合该类型，但`logSong`因为接受了一个`string`而不是`number`作为其参数而失败：

```
const songs = ["Juice", "Shake It Off", "What's Up"];

function runOnSongs(getSongAt: (index: number) => string) {
  for (let i = 0; i < songs.length; i += 1) {
    console.log(getSongAt(i));
  }
}

function getSongAt(index: number) {
  return `${songs[index]}`;
}

runOnSongs(getSongAt); // Ok

function logSong(song: string) {
  return `${song}`;
}

runOnSongs(logSong);
//         ~~~~~~~
// Error: Argument of type '(song: string) => string' is not
// assignable to parameter of type '(index: number) => string'.
//   Types of parameters 'song' and 'index' are incompatible.
//     Type 'number' is not assignable to type 'string'.
```

`runOnSongs(logSong)`的错误消息是一个可赋值性错误的例子，其中包含几个层次的细节。当投诉两个函数类型不可分配给彼此时，TypeScript 通常会给出三个层次的详细信息，具体性逐级增加：

1.  第一个缩进级别打印出这两种函数类型。

1.  下一个缩进级别指定了哪个部分不匹配。

1.  最后一个缩进级别是不匹配部分的精确可赋性投诉。

在上一个代码片段中，这些层次是：

1.  `logSong`的`(strong: string) => string`是被赋予的类型，被分配给`getSongAt: (index: number) => string`的接收者

1.  `logSong`的`song`参数被分配给`getSongAt`的`index`参数

1.  `song`的`number`类型不可赋给`index`的`string`类型

###### 提示

TypeScript 的多行错误一开始可能看起来令人生畏。逐行阅读并理解每个部分所传达的内容对于理解错误有很大帮助。

## 函数类型括号

函数类型可以放置在其他任何类型可以使用的地方。这包括联合类型。

在联合类型中，括号可以用来指示注释的哪一部分是函数返回还是周围的联合类型：

```
// Type is a function that returns a union: string | undefined
let returnsStringOrUndefined: () => string | undefined;

// Type is either undefined or a function that returns a string
let maybeReturnsString: (() => string) | undefined;
```

后续介绍更多类型语法的章节将展示函数类型必须用括号包裹的其他位置。

## 参数类型推断

如果我们必须为我们编写的每个函数声明参数类型，包括用作参数的内联函数，那将是很麻烦的。幸运的是，TypeScript 可以推断函数中参数的类型，前提是这些函数被提供给带有声明类型的位置。

这个`singer`变量被知道是一个接受`string`类型参数的函数，所以后来分配给`singer`的函数中的`song`参数被知道是一个`string`：

```
let singer: (song: string) => string;

singer = function (song) {
  // Type of song: string
  return `Singing: ${song.toUpperCase()}!`; // Ok
};
```

函数作为具有函数参数类型的参数传递时，它们的参数类型也将被推断。

例如，在这里，TypeScript 推断`song`和`index`参数分别为`string`和`number`：

```
const songs = ["Call Me", "Jolene", "The Chain"];

// song: string
// index: number
songs.forEach((song, index) => {
  console.log(`${song} is at index ${index}`);
});
```

## 函数类型别名

记得第三章“联合和字面值”中的类型别名吗？它们也可以用于函数类型。

这个`StringToNumber`类型别名了一个接受`string`并返回`number`的函数，这意味着它可以用来描述变量的类型：

```
type StringToNumber = (input: string) => number;

let stringToNumber: StringToNumber;

stringToNumber = (input) => input.length; // Ok

stringToNumber = (input) => input.toUpperCase();
//                          ~~~~~~~~~~~~~~~~~~~
// Error: Type 'string' is not assignable to type 'number'.
```

同样，函数参数本身可以用别名进行类型化，这些别名碰巧指向一个函数类型。

这个`usesNumberToString`函数有一个参数，它本身是`NumberToString`别名的函数类型：

```
type NumberToString = (input: number) => string;

function usesNumberToString(numberToString: NumberToString) {
  console.log(`The string is: ${numberToString(1234)}`);
}

usesNumberToString((input) => `${input}! Hooray!`); // Ok

usesNumberToString((input) => input * 2);
//                            ~~~~~~~~~
// Error: Type 'number' is not assignable to type 'string'.
```

类型别名在函数类型中特别有用。它们可以节省大量水平空间，而无需重复编写参数和/或返回类型。

# 更多返回类型

现在，让我们看看两种更多的返回类型：`void`和`never`。

## 空返回

有些函数不打算返回任何值。它们要么没有`return`语句，要么只有不返回值的`return`语句。TypeScript 允许使用`void`关键字来指代返回什么都没有的函数的返回类型。

返回类型为`void`的函数可能不会返回值。这个`logSong`函数被声明为返回`void`，因此不允许返回值：

```
function logSong(song: string | undefined): void {
  if (!song) {
    return; // Ok
  }

  console.log(`${song}`);

  return true;
  // Error: Type 'boolean' is not assignable to type 'void'.
}
```

`void`可以作为函数类型声明中的返回类型很有用。在函数类型声明中使用时，`void`表示函数的任何返回值都将被忽略。

例如，这个`songLogger`变量表示一个接受`song: string`并且不返回值的函数：

```
let songLogger: (song: string) => void;

songLogger = (song) => {
  console.log(`${songs}`);
};

songLogger("Heart of Glass"); // Ok
```

请注意，尽管 JavaScript 函数如果没有返回真正的值，默认都会返回`undefined`，但`void`并不等同于`undefined`。`void`表示函数的返回类型将被忽略，而`undefined`是要返回的字面值。试图将`void`类型的值分配给其类型包括`undefined`的值是一种类型错误：

```
function returnsVoid() {
  return;
}

let lazyValue: string | undefined;

lazyValue = returnsVoid();
// Error: Type 'void' is not assignable to type 'string | undefined'.
```

`undefined`和`void`返回之间的区别对于忽略从传递给返回类型声明为`void`的位置的函数返回的任何值特别有用。例如，数组上的内置`forEach`方法接受一个返回`void`的回调。提供给`forEach`的函数可以返回任何值。以下`saveRecords`函数中的`records.push(record)`返回一个`number`（从数组的`.push()`返回的值），但仍然允许作为传递给`newRecords.forEach`的箭头函数的返回值：

```
const records: string[] = [];

function saveRecords(newRecords: string[]) {
  newRecords.forEach(record => records.push(record));
}

saveRecords(['21', 'Come On Over', 'The Bodyguard'])
```

`void`类型不是 JavaScript。它是 TypeScript 中用于声明函数返回类型的关键字。记住，它表示函数的返回值不打算被使用，而不是可以被返回的值。

## 永不返回

有些函数不仅不返回值，而且根本不打算返回。永不返回的函数是那些总是抛出错误或运行无限循环的函数（希望是有意的！）。

如果一个函数永远不会返回，添加显式的`: never`类型注释表示调用该函数后的任何代码都不会运行。这个`fail`函数只会抛出错误，因此它可以帮助 TypeScript 的控制流分析将`param`缩小到`string`：

```
function fail(message: string): never {
    throw new Error(`Invariant failure: ${message}.`);
}

function workWithUnsafeParam(param: unknown) {
    if (typeof param !== "string") {
        fail(`param should be a string, not ${typeof param}`);
    }

    // Here, param is known to be type string
    param.toUpperCase(); // Ok
}
```

###### 注意

`never`不同于`void`。`void`用于不返回任何内容的函数。`never`用于永远不返回的函数。

# 函数重载

一些 JavaScript 函数可以使用完全不同的参数集调用，这些参数集不能仅通过可选和/或剩余参数表示。这些函数可以用 TypeScript 语法描述为*重载签名*：在最终的*实现签名*和函数体之前多次声明函数名称、参数和返回类型的不同版本。

在确定是否为重载函数调用发出语法错误时，TypeScript 只会查看函数的重载签名。实现签名仅由函数的内部逻辑使用。

此 `createDate` 函数旨在使用一个 `timestamp` 参数或三个参数 `month`、`day` 和 `year` 来调用。允许使用这些参数数量中的任何一个进行调用，但是如果使用两个参数进行调用，则会导致类型错误，因为没有重载签名允许两个参数。在此示例中，前两行是重载签名，第三行是实现签名：

```
function createDate(timestamp: number): Date;
function createDate(month: number, day: number, year: number): Date;
function createDate(monthOrTimestamp: number, day?: number, year?: number) {
  return day === undefined || year === undefined
    ? new Date(monthOrTimestamp)
    : new Date(year, monthOrTimestamp, day);
}

createDate(554356800); // Ok
createDate(7, 27, 1987); // Ok

createDate(4, 1);
// Error: No overload expects 2 arguments, but overloads
// do exist that expect either 1 or 3 arguments.
```

重载签名与其他类型系统语法一样，在将 TypeScript 编译为输出 JavaScript 时会被擦除。

上述代码片段的函数将编译为大致以下 JavaScript：

```
function createDate(monthOrTimestamp, day, year) {
  return day === undefined || year === undefined
    ? new Date(monthOrTimestamp)
    : new Date(year, monthOrTimestamp, day);
}
```

###### 警告。

函数重载通常用作复杂、难以描述的函数类型的最后手段。通常最好保持函数简单，并在可能时避免使用函数重载。

## 调用签名兼容性。

用于重载函数实现的实现签名是函数实现使用的参数类型和返回类型。因此，函数重载签名中的返回类型和每个参数必须可分配给其实现签名中相同索引处的参数。换句话说，实现签名必须与所有重载签名兼容。

此 `format` 函数的实现签名声明其第一个参数为 `string`。虽然前两个重载签名也兼容于类型 `string`，但第三个重载签名的 `() => string` 类型是不兼容的：

```
function format(data: string): string; // Ok
function format(data: string, needle: string, haystack: string): string; // Ok

function format(getData: () => string): string;
//       ~~~~~~
// This overload signature is not compatible with its implementation signature.

function format(data: string, needle?: string, haystack?: string) {
  return needle && haystack ? data.replace(needle, haystack) : data;
}
```

# 总结。

在本章中，您看到了如何在 TypeScript 中推断或显式声明函数的参数和返回类型：

+   声明函数参数类型时使用类型注解。

+   使用可选参数、默认值和剩余参数来改变类型系统行为。

+   使用类型注解声明函数返回类型。

+   使用 `void` 类型描述不返回可用值的函数。

+   使用 `never` 类型描述根本不返回的函数。

+   使用函数重载描述不同的函数调用签名。

###### 提示。

现在您已经完成了本章的阅读，请在 [*https://learningtypescript.com/functions*](https://learningtypescript.com/functions) 上练习您所学的内容。

> 什么使得 TypeScript 项目变得优秀？
> 
> 它的功能良好。
