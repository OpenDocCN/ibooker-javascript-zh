# 第十四章：语法扩展

> “TypeScript 不会添加
> 
> 到 JavaScript 运行时的影响。”
> 
> …那一切都是谎言吗？！

当 TypeScript 在 2012 年首次发布时，Web 应用程序的复杂性增长速度远远快于普通 JavaScript 添加支持深层复杂性的功能。当时最流行的 JavaScript 语言变体，CoffeeScript，通过引入新颖的语法结构，与 JavaScript 趋向分歧。

如今，通过引入针对 TypeScript 等超集语言的新运行时特性扩展 JavaScript 语法被认为是不良实践，原因有几个：

+   最重要的是，运行时语法扩展可能与较新版本的 JavaScript 中的新语法冲突。

+   它们使得对于新接触该语言的程序员来说更难理解 JavaScript 的结束和其他语言的开始。

+   它们增加了将超集语言代码转译为 JavaScript 的转译器的复杂性。

因此，我怀着沉重的心情和深深的遗憾告诉你，早期的 TypeScript 设计者在 TypeScript 语言中引入了三个 JavaScript 语法扩展：

+   类，在规范确定的情况下与 JavaScript 类对齐

+   枚举，类似于键和值的普通对象的简单语法糖

+   命名空间，一种在现代模块之前结构化和排列代码的解决方案

###### 注意

TypeScript 在 JavaScript 的运行时语法扩展“原罪”在 TypeScript 早期并不是一种设计决策。TypeScript 不会在这些新的运行时语法构造通过严格的审议过程被添加到 JavaScript 本身之前引入它们。

TypeScript 类最终看起来和行为几乎与 JavaScript 类相同（哦！），除了 `useDefineForClassFields` 行为（本书未涵盖的配置选项）和参数属性（在此涵盖）。由于它们偶尔有用，枚举仍然在某些项目中使用。几乎没有新项目再使用命名空间。

TypeScript 也采纳了 JavaScript “装饰器”的实验性提案，我也会涵盖这个内容。

# 类参数属性

###### 提示

我建议避免使用类参数属性，除非你在一个大量使用类或从中受益的框架的项目中工作。

在 JavaScript 类中，通常希望在构造函数中接收一个参数并立即将其分配给类属性。

这个 `Engineer` 类接受一个 `area` 参数，类型为 `string`，并将其赋给类型为 `string` 的 `area` 属性：

```
class Engineer {
    readonly area: string;

    constructor(area: string) {
        this.area = area;
        console.log(`I work in the ${area} area.`);
    }
}

// Type: string
new Engineer("mechanical").area;
```

TypeScript 包括一种用于声明这些“参数属性”的快捷语法：在类构造函数的开头为同一类型的成员属性分配属性。在构造函数参数前面放置`readonly`和/或其中一个隐私修饰符——`public`、`protected`或`private`，告诉 TypeScript 还要声明一个同名和类型的属性。

前述的`Engineer`示例可以使用参数属性在 TypeScript 中重写`area`：

```
class Engineer {
    constructor(readonly area: string) {
        console.log(`I work in the ${area} area.`);
    }
}

// Type: string
new Engineer("mechanical").area;
```

参数属性在类构造函数的最开始（或在继承自基类的情况下`super()`调用之后）被分配。它们可以与类的其他参数和/或属性混合使用。

下面的`NamedEngineer`类声明了一个常规属性`fullName`，一个常规参数`name`和一个参数属性`area`：

```
class NamedEngineer {
    fullName: string;

    constructor(
        name: string,
        public area: string,
    ) {
        this.fullName = `${name}, ${area} engineer`;
    }
}
```

没有参数属性的等效 TypeScript 看起来类似，但需要更多代码行来显式赋值`area`：

```
class NamedEngineer {
    fullName: string;
    area: string;

    constructor(
        name: string,
        area: string,
    ) {
        this.area = area;
        this.fullName = `${name}, ${area} engineer`;
    }
}
```

参数属性在 TypeScript 社区中有时是一个争议性问题。大多数项目倾向于彻底避免它们，因为它们是运行时语法扩展，因此遭受我之前提到的相同缺点的影响。它们也无法与新的`#`类私有字段语法一起使用。

另一方面，在那些非常偏爱创建类的项目中，它们的使用效果非常好。参数属性解决了一个便利性问题，即需要两次声明参数属性名称和类型，这是 TypeScript 固有的，而不是 JavaScript。

# 实验性装饰器

###### 小贴士

我建议尽可能避免装饰器，直到 ECMAScript 的一个版本通过装饰器语法。如果您正在使用像 Angular 或 NestJS 这样的框架版本，该框架的文档将指导如何使用它们。

许多其他允许类注释或装饰这些类及其成员的语言，使用某种运行时逻辑修改它们。*装饰器*函数是 JavaScript 允许通过首先放置`@`和函数名称来注释类和成员的建议。

例如，以下代码段展示了如何在类`MyClass`上使用装饰器的语法：

```
@myDecorator
class MyClass { /* ... */ }
```

ECMAScript 尚未正式通过装饰器，因此截至版本 4.7.2，TypeScript 默认不支持它们。但是，TypeScript 包含一个`experimentalDecorators`编译选项，允许在代码中使用旧的实验版本。可以通过`tsc`命令行或 TSConfig 文件启用，如下所示，与其他编译选项一样：

```
{
    "compilerOptions": {
        "experimentalDecorators": true
    }
}
```

每次使用装饰器时，它都会在所装饰的实体创建时执行一次。每种装饰器——访问器、类、方法、参数和属性——都接收一组描述所装饰实体的不同参数。

例如，此`logOnCall`装饰器用于`Greeter`类方法上，接收`Greeter`类本身，属性键（`"log"`），以及描述该属性的`descriptor`对象。修改`descriptor.value`以在调用`Greeter`类上的原始`greet`方法之前记录“装饰”`greet`方法：

```
function logOnCall(target: any, key: string, descriptor: PropertyDescriptor) {
    const original = descriptor.value;
    console.log("[logOnCall] I am decorating", target.constructor.name);

    descriptor.value = function (...args: unknown[]) {
        console.log(`[descriptor.value] Calling '${key}' with:`, ...args);
        return original.call(this, ...args);
    }
}

class Greeter {
    @logOnCall
    greet(message: string) {
        console.log(`[greet] Hello, ${message}!`);
    }
}

new Greeter().greet("you");
// Output log:
// "[logOnCall] I am decorating", "Greeter"
// "[descriptor.value] Calling 'greet' with:", "you"
// "[greet] Hello, you!"
```

我不会深入探讨旧的`experimentalDecorators`如何为每种可能的装饰器类型工作的细微差别和具体信息。 TypeScript 的装饰器支持是实验性的，并且与 ECMAScript 提案的最新草案不一致。尤其是在任何 TypeScript 项目中编写自己的装饰器很少是合理的。

# 枚举

###### 提示

我建议不要使用枚举，除非您有一组经常重复的字面值，所有这些字面值都可以用一个常见名称描述，并且如果切换到枚举将更容易阅读其代码。

大多数编程语言包含“枚举”或枚举类型的概念，用于表示一组相关值。枚举可以被视为存储在对象中的一组字面值，并为每个值提供友好的名称。

JavaScript 不包括枚举语法，因为传统对象可以用来替代它们。例如，虽然 HTTP 状态码可以存储并用作数字，但许多开发人员发现将它们存储在由友好名称键入的对象中更易于阅读：

```
const StatusCodes = {
    InternalServerError: 500,
    NotFound: 404,
    Ok: 200,
    // ...
} as const;

StatusCodes.InternalServerError; // 500
```

TypeScript 中类似枚举对象的棘手之处在于没有很好的类型系统方法来表示值必须是它们的值之一。一种常见的方法是使用来自第九章，“类型修饰符”的`keyof`和`typeof`类型修改器来“拼凑”一个，但这需要一定的语法输入。

下面的`StatusCodeValue`类型使用先前的`StatusCodes`值创建其可能状态码数字值的类型联合：

```
// Type: 200 | 404 | 500
type StatusCodeValue = (typeof StatusCodes)[keyof typeof StatusCodes];

let statusCodeValue: StatusCodeValue;

statusCodeValue = 200; // Ok

statusCodeValue = -1;
// Error: Type '-1' is not assignable to type 'StatusCodeValue'.
```

TypeScript 提供了一个用于创建类型为`number`或`string`字面值的对象的`enum`语法。从`enum`关键字开始，然后是一个对象的名称——通常是 PascalCase——然后是包含枚举中逗号分隔键的`{}`对象。每个键可以选择在初始值前使用`=`。

前一个`StatusCodes`对象将类似于此`StatusCode`枚举：

```
enum StatusCode {
    InternalServerError = 500,
    NotFound = 404,
    Ok = 200,
}

StatusCode.InternalServerError; // 500
```

与类名一样，`StatusCode`等枚举名称可以用作类型注释中的类型名称。在此处，类型为`StatusCode`的`statusCode`变量可以给予`StatusCode.Ok`或一个数字值：

```
let statusCode: StatusCode;

statusCode = StatusCode.Ok; // Ok
statusCode = 200; // Ok
```

###### 警告

TypeScript 允许将任何数字分配给数值枚举值以方便，但会略微降低类型安全性。在先前的代码片段中，`statusCode = -1`也将被允许。

枚举编译为输出编译后的 JavaScript 中的等效对象。它们的每个成员都变成了具有相应值的对象成员键，反之亦然。

先前的`enum StatusCode`将大致创建以下 JavaScript：

```
var StatusCode;
(function (StatusCode) {
    StatusCode[StatusCode["InternalServerError"] = 500] = "InternalServerError";
    StatusCode[StatusCode["NotFound"] = 404] = "NotFound";
    StatusCode[StatusCode["Ok"] = 200] = "Ok";
})(StatusCode || (StatusCode = {}));
```

枚举是 TypeScript 社区中一个略具争议的话题。 一方面，它们违反了 TypeScript 不向 JavaScript 添加新的运行时语法结构的一般原则。 它们为开发人员学习提供了一种新的非 JavaScript 语法，并围绕选项（如 `preserveConstEnums`，稍后在本章中讨论）存在一些怪癖。

另一方面，它们对于显式声明已知值集合非常有用。 枚举在 TypeScript 和 VS Code 源代码库中广泛使用！

## 自动数值

枚举成员不需要具有显式初始值。 当省略值时，TypeScript 将从 `0` 开始第一个值，并递增每个后续值 `1`。 当值在只需唯一关联键名而不重要时，允许 TypeScript 选择枚举成员的值是一个不错的选择。

此 `VisualTheme` 枚举允许 TypeScript 完全选择值，结果是三个整数：

```
enum VisualTheme {
    Dark, // 0
    Light, // 1
    System, // 2
}
```

发出的 JavaScript 看起来与如果值已被显式设置时相同：

```
var VisualTheme;
(function (VisualTheme) {
    VisualTheme[VisualTheme["Dark"] = 0] = "Dark";
    VisualTheme[VisualTheme["Light"] = 1] = "Light";
    VisualTheme[VisualTheme["System"] = 2] = "System";
})(VisualTheme || (VisualTheme = {}));
```

在带有数值的枚举中，任何未显式设置值的成员都会比前一个值高`1`。

例如，`Direction` 枚举可能只关心其 `Top` 成员的值为 `1`，其余值也是正整数：

```
enum Direction {
  Top = 1,
  Right,
  Bottom,
  Left,
}
```

其输出的 JavaScript 也将与其余成员显式值 `2`、`3` 和 `4` 相同：

```
var Direction;
(function (Direction) {
    Direction[Direction["Top"] = 1] = "Top";
    Direction[Direction["Right"] = 2] = "Right";
    Direction[Direction["Bottom"] = 3] = "Bottom";
    Direction[Direction["Left"] = 4] = "Left";
})(Direction || (Direction = {}));
```

###### 警告

修改枚举的顺序将导致底层数值发生变化。 如果你将这些值持久化存储在某个地方（如数据库），请小心更改枚举顺序或删除条目。 因为保存的数值将不再代表你的代码所期望的内容，你的数据可能会突然损坏。

## 字符串值枚举

枚举也可以使用字符串作为其成员而不是数字。

此 `LoadStyle` 枚举使用友好的字符串值作为其成员：

```
enum LoadStyle {
    AsNeeded = "as-needed",
    Eager = "eager",
}
```

使用字符串成员值的枚举在输出 JavaScript 方面与使用数值成员值的枚举结构上看起来一样：

```
var LoadStyle;
(function (LoadStyle) {
    LoadStyle["AsNeeded"] = "as-needed";
    LoadStyle["Eager"] = "eager";
})(LoadStyle || (LoadStyle = {}));
```

字符串值枚举对于为共享常量起别名很有用。 而不是使用字符串文字的类型联合，字符串值枚举允许更强大的编辑器自动完成和重命名这些属性，正如在 第十二章，“使用 IDE 功能” 中所述。

字符串成员值的一个缺点是 TypeScript 不能自动计算它们。 仅允许跟随数值成员值的枚举成员可以自动计算。

TypeScript 可以为此枚举的 `ImplicitNumber` 隐式提供值 `9001`，因为前一个成员值是数字 `9000`，但其 `NotAllowed` 成员将会因其后跟字符串成员值而发出错误：

```
enum Wat {
    FirstString = "first",
    SomeNumber = 9000,
    ImplicitNumber, // Ok (value 9001)
    AnotherString = "another",

    NotAllowed,
    // Error: Enum member must have initializer.
}
```

###### 提示

理论上，你可以创建一个既有数值成员又有字符串成员值的枚举。 实际上，这种枚举可能会导致混淆，所以你可能不应该这样做。

## 常量枚举

因为枚举创建了一个运行时对象，使用它们会产生比使用字面值联合的常见替代策略更多的代码。TypeScript 允许在枚举前加上 `const` 修饰符来告诉 TypeScript 从编译的 JavaScript 代码中省略它们的对象定义和属性查找。

这个 `DisplayHint` 枚举用作 `displayHint` 变量的值：

```
const enum DisplayHint {
    Opaque = 0,
    Semitransparent,
    Transparent,
}

let displayHint = DisplayHint.Transparent;
```

输出的编译后 JavaScript 代码将完全省略枚举声明，并在枚举值上使用注释：

```
let displayHint = 2 /* DisplayHint.Transparent */;
```

对于仍希望创建枚举对象定义的项目，确实存在一个 `preserveConstEnums` 编译器选项，该选项将保留枚举声明本身的存在。值仍将直接使用字面量，而不是在枚举对象上访问它们。

前面的代码片段在其编译后的 JavaScript 输出中仍将省略属性查找：

```
var DisplayHint;
(function (DisplayHint) {
    DisplayHint[DisplayHint["Opaque"] = 0] = "Opaque";
    DisplayHint[DisplayHint["Semitransparent"] = 1] = "Semitransparent";
    DisplayHint[DisplayHint["Transparent"] = 2] = "Transparent";
})(DisplayHint || (DisplayHint = {}));

let displayHint = 2 /* Transparent */;
```

`preserveConstEnums` 可以帮助减少生成的 JavaScript 代码的大小，尽管并非所有的 TypeScript 代码转译方式都支持它。参见 第十三章，“配置选项” 获取关于 `isolatedModules` 编译器选项以及何时可能不支持 `const` 枚举的更多信息。

# 命名空间

###### 警告

除非您正在为现有包编写 DefinitelyTyped 类型定义文件，否则不要使用命名空间。命名空间不符合现代 JavaScript 模块语义。它们的自动成员赋值可能使代码难以阅读。我之所以提到它们，是因为您可能会在 *.d.ts* 文件中遇到它们。

在 ECMAScript 模块得到批准之前，将大量输出代码捆绑到浏览器加载的单个文件中并不罕见。这些巨大的单文件通常创建全局变量来保存项目不同区域重要数值的引用。将这个文件包含在页面中比设置旧的模块加载器（如 RequireJS）更为简单——在许多服务器尚未支持 HTTP/2 下载流时，这样做通常也更为高效。为单文件输出的项目需要一种组织代码部分和全局变量的方式。

TypeScript 语言通过“内部模块”的概念提供了一种解决方案，现在称为命名空间。*命名空间* 是一个全局可用的对象，其“导出”的内容可作为该对象的成员调用。命名空间使用 `namespace` 关键字后跟一个 `{}` 代码块来定义。该命名空间块中的所有内容在函数闭包内部评估。

这个 `Randomized` 命名空间创建一个 `value` 变量并在内部使用它：

```
namespace Randomized {
    const value = Math.random();
    console.log(`My value is ${value}`);
}
```

它的输出 JavaScript 创建了一个 `Randomized` 对象，并在函数内部评估了块的内容，因此 `value` 变量在命名空间外部不可用：

```
var Randomized;
(function (Randomized) {
    const value = Math.random();
    console.log(`My value is ${value}`);
})(Randomized || (Randomized = {}));
```

###### 警告

命名空间和`namespace`关键字最初在 TypeScript 中分别称为“模块”和“`module`”。考虑到现代模块加载器和 ECMAScript 模块的兴起，这是一个可惜的选择。在回顾中，`module`关键字在非常旧的项目中仍偶尔会出现，但可以——也应该——安全地替换为`namespace`。

## 命名空间导出

命名空间的关键特性是它可以通过将内容作为命名空间对象的成员来“导出”。然后，代码的其他部分可以通过名称引用该成员。

在这里，`Settings`命名空间导出了在命名空间内部和外部使用的`describe`、`name`和`version`值：

```
namespace Settings {
  export const name = "My Application";
  export const version = "1.2.3";

  export function describe() {
    return `${Settings.name} at version ${Settings.version}`;
  }

  console.log("Initializing", describe());
}

console.log("Initialized", Settings.describe());
```

输出的 JavaScript 显示，值始终作为`Settings`的成员（例如，`Settings.name`）在内部和外部使用：

```
var Settings;
(function (Settings) {
    Settings.name = "My Application";
    Settings.version = "1.2.3";
    function describe() {
        return `${Settings.name} at version ${Settings.version}`;
    }
    Settings.describe = describe;
    console.log("Initializing", describe());
})(Settings || (Settings = {}));
console.log("Initialized", Settings.describe());
```

通过使用`var`来输出对象，并将导出的内容引用为这些对象的成员，命名空间的设计在跨多个文件时工作良好。之前的`Settings`命名空间可以在多个文件中重写：

```
// settings/constants.ts
namespace Settings {
  export const name = "My Application";
  export const version = "1.2.3";
}
```

```
// settings/describe.ts
namespace Settings {
    export function describe() {
        return `${Settings.name} at version ${Settings.version}`;
    }

    console.log("Initializing", describe());
}
```

```
// index.ts
console.log("Initialized", Settings.describe());
```

输出的 JavaScript，拼接在一起，大致如下：

```
// settings/constants.ts
var Settings;
(function (Settings) {
    Settings.name = "My Application";
    Settings.version = "1.2.3";
})(Settings || (Settings = {}));
// settings/describe.ts
(function (Settings) {
    function describe() {
        return `${Settings.name} at version ${Settings.version}`;
    }
    Settings.describe = describe;
    console.log("Initialized", describe());
})(Settings || (Settings = {}));
console.log("Initialized", Settings.describe());
```

在单文件和多文件声明形式中，运行时输出对象是一个具有三个键的对象。大致上：

```
const Settings = {
    describe: function describe() {
        return `${Settings.name} at version ${Settings.version}`;
    },
    name: "My Application",
    version: "1.2.3",
};
```

使用命名空间的关键区别在于，它可以跨不同文件分割，并且成员仍然可以在命名空间的名称下相互引用。

## 嵌套命名空间

命名空间可以通过将命名空间从另一个命名空间导出或在名称中放置一个或多个`.`（点）来“嵌套”到无限级别。

以下两个命名空间声明的行为将是相同的：

```
namespace Root.Nested {
    export const value1 = true;
}

namespace Root {
    export namespace Nested {
        export const value2 = true;
    }
}
```

它们都编译为结构上相同的代码：

```
(function (Root) {
    let Nested;
    (function (Nested) {
        Nested.value2 = true;
    })(Nested || (Nested = {}));
})(Root || (Root = {}));
```

嵌套命名空间是强制在使用命名空间组织的大型项目中的各个部分之间更好地划分的一个方便方法。许多开发人员选择使用项目名称的根命名空间——也许在公司的和/或组织的命名空间内部——以及项目的每个主要区域的子命名空间。

## 类型定义中的命名空间

现在命名空间的唯一优点——也是我选择将其包括在本书中的唯一原因——是它们对于 DefinitelyTyped 类型定义非常有用。许多 JavaScript 库——特别是一些较旧的 Web 应用程序基础库，如 jQuery——是为了在 Web 浏览器中以传统的、非模块化的`<script>`标签包含而设置的。它们的类型定义需要指示它们创建一个对所有代码可用的全局变量——这种结构正是命名空间所完美捕捉的。

此外，许多支持浏览器的 JavaScript 库既可以在更现代的模块系统中导入，也可以创建一个全局命名空间。TypeScript 允许模块类型定义包括一个`export as namespace`，后跟全局名称，以指示该模块也在该名称下全局可用。

例如，这个模块的声明文件导出了一个 `value` 并且在全局可用：

```
// node_modules/@types/my-example-lib/index.d.ts
export const value: number;
export as namespace libExample;
```

类型系统将会知道 `import("my-example-lib")` 和 `window.libExample` 都将返回一个带有类型为 `number` 的 `value` 属性的模块：

```
// src/index.ts
import * as libExample from "my-example-lib"; // Ok
const value = window.libExample.value; // Ok
```

## 更倾向于使用模块而非命名空间

与其使用命名空间，前面示例中的 *settings/constants.ts* 文件和 *settings/describe.ts* 文件可以根据现代标准改为使用 ECMAScript 模块：

```
// settings/constants.ts
export const name = "My Application";
export const version = "1.2.3";
```

```
// settings/describe.ts
import { name, version } from "./constants";

export function describe() {
    return `${Settings.name} at version ${Settings.version}`;
}

console.log("Initializing", describe());
```

```
// index.ts
import { describe } from "./settings/describe";

console.log("Initialized", describe());
```

在使用命名空间结构化的 TypeScript 代码中，由于命名空间创建的是隐式的文件之间联系，而非 ECMAScript 模块那样显式声明的联系，因此现代构建工具（如 Webpack）很难对其进行树摇（即删除未使用的文件）。通常强烈建议使用 ECMAScript 模块而非 TypeScript 命名空间编写运行时代码。

###### 注意

截至 2022 年，TypeScript 本身是用命名空间编写的，但 TypeScript 团队正在努力转换为模块。谁知道，也许在您阅读本文时，他们已完成这一转换！让我们拭目以待。

# 仅类型导入和导出

我希望以积极的笔调结束这一章节。最后介绍的一组语法扩展——仅类型导入和导出，非常有用且不会增加生成的 JavaScript 的复杂性。

TypeScript 的编译器将从导入和导出的值中删除仅在类型系统中使用的值，因为它们不在运行时 JavaScript 中使用。

例如，以下的 *index.ts* 文件创建了一个 `action` 变量和一个 `ActivistArea` 类型，然后通过独立的导出声明导出它们。当编译到 *index.js* 时，TypeScript 的编译器会知道从独立导出声明中移除 `ActivistArea`：

```
// index.ts
const action = { area: "people", name: "Bella Abzug", role: "politician" };

type ActivistArea = "nature" | "people";

export { action, ActivistArea };
```

```
// index.js
const action = { area: "people", name: "Bella Abzug", role: "politician" };

export { action };
```

知道如何删除像 `ActivistArea` 这样的重新导出类型需要了解 TypeScript 类型系统。像 Babel 这样逐个文件操作的转译器无法访问 TypeScript 类型系统，无法知道每个名称是否仅在类型系统中使用。TypeScript 的 `isolatedModules` 编译选项，在除 TypeScript 外的其他工具中帮助确保代码将转译成功，详见 第十三章，“配置选项”。

TypeScript 允许在单个导入名称或整个 `{...}` 对象的 `export` 和 `import` 声明前面添加 `type` 修饰符。这样做表示它们仅用于类型系统中。还可以将包的默认导入标记为 `type`。

在以下代码片段中，当 *index.ts* 转译为输出的 *index.js* 时，只有 `value` 的导入和导出被保留：

```
// index.ts
import { type TypeOne, value } from "my-example-types";
import type { TypeTwo } from "my-example-types";
import type DefaultType from "my-example-types";

export { type TypeOne, value };
export type { DefaultType, TypeTwo };
```

```
// index.js
import { value } from "my-example-types";

export { value };
```

一些 TypeScript 开发者甚至更喜欢选择使用仅类型导入，以明确哪些导入仅用作类型。如果将导入标记为仅类型，则尝试将其用作运行时值将触发 TypeScript 错误。

[ClassOne](https://learningtypescript.com/syntax-extensions)被正常导入并可在运行时使用，但是`ClassTwo`不能，因为它作为类型导入：

```
import { ClassOne, type ClassTwo } from "my-example-types";

new ClassOne(); // Ok

new ClassTwo();
//  ~~~~~~~~
// Error: 'ClassTwo' cannot be used as a value
// because it was imported using 'import type'.
```

而不是向生成的 JavaScript 添加复杂性，仅限于类型的导入和导出能够明确告知 TypeScript 之外的转译器在何时可以移除代码片段。因此，大多数 TypeScript 开发者不会像对待本章所覆盖的先前语法扩展一样，对它们产生厌恶情绪。

# 总结

在本章中，你使用了 TypeScript 中包含的一些 JavaScript 语法扩展：

+   在类构造函数中声明类参数属性

+   使用装饰器增强类及其字段

+   用枚举表示值的组合

+   使用命名空间在文件间或类型定义中创建分组

+   仅限类型的导入和导出

###### 提示

在你完成阅读本章之后，练习你所学到的内容[*https://learningtypescript.com/syntax-extensions*](https://learningtypescript.com/syntax-extensions)。

> 在 TypeScript 中，你如何称呼支持遗留 JavaScript 扩展的成本？
> 
> “罪恶税”
