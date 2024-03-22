# 第二章：词法结构

编程语言的词法结构是指定如何在该语言中编写程序的基本规则集。它是语言的最低级语法：它指定变量名的外观，注释的分隔符字符，以及如何将一个程序语句与下一个分隔开，例如。本短章记录了 JavaScript 的词法结构。它涵盖了：

+   区分大小写、空格和换行

+   注释

+   文字

+   标识符和保留字

+   Unicode

+   可选分号

# 2.1 JavaScript 程序的文本

JavaScript 是区分大小写的语言。这意味着语言关键字、变量、函数名和其他*标识符*必须始终以一致的字母大小写输入。例如，`while`关键字必须输入为`while`，而不是“While”或“WHILE”。同样，`online`、`Online`、`OnLine`和`ONLINE`是四个不同的变量名。

JavaScript 会忽略程序中标记之间出现的空格。在大多数情况下，JavaScript 也会忽略换行（但请参见§2.6 中的一个例外）。由于您可以在程序中自由使用空格和换行，因此可以以整洁一致的方式格式化和缩进程序，使代码易于阅读和理解。

除了常规空格字符（`\u0020`）外，JavaScript 还识别制表符、各种 ASCII 控制字符和各种 Unicode 空格字符作为空白。JavaScript 将换行符、回车符和回车符/换行符序列识别为行终止符。

# 2.2 注释

JavaScript 支持两种注释风格。任何位于`//`和行尾之间的文本都被视为注释，JavaScript 会忽略它。位于`/*`和`*/`之间的文本也被视为注释；这些注释可以跨越多行，但不能嵌套。以下代码行都是合法的 JavaScript 注释：

```js
// This is a single-line comment.

/* This is also a comment */  // and here is another comment.

/*
 * This is a multi-line comment. The extra * characters at the start of
 * each line are not a required part of the syntax; they just look cool!
 */
```

# 2.3 文字

*文字* 是直接出现在程序中的数据值。以下都是文字：

```js
12               // The number twelve
1.2              // The number one point two
"hello world"    // A string of text
'Hi'             // Another string
true             // A Boolean value
false            // The other Boolean value
null             // Absence of an object
```

数字和字符串文字的完整详细信息请参见第三章。

# 2.4 标识符和保留字

*标识符* 就是一个名字。在 JavaScript 中，标识符用于命名常量、变量、属性、函数和类，并为 JavaScript 代码中某些循环提供标签。JavaScript 标识符必须以字母、下划线（`_`）或美元符号（`$`）开头。后续字符可以是字母、数字、下划线或美元符号。（不允许数字作为第一个字符，以便 JavaScript 可以轻松区分标识符和数字。）以下都是合法的标识符：

```js
i
my_variable_name
v13
_dummy
$str
```

与任何语言一样，JavaScript 为语言本身保留了某些标识符。这些“保留字”不能用作常规标识符。它们在下一节中列出。

## 2.4.1 保留字

以下单词是 JavaScript 语言的一部分。其中许多（如`if`、`while`和`for`）是必须避免用作常量、变量、函数或类名称的保留关键字（尽管它们都可以用作对象内的属性名称）。其他一些单词（如`from`、`of`、`get`和`set`）在有限的上下文中使用时没有语法歧义，作为标识符是完全合法的。还有其他关键字（如`let`）为了保持与旧程序的向后兼容性而不能完全保留，因此有复杂的规则规定何时可以将其用作标识符，何时不行。（例如，如果在类外部用`var`声明，`let`可以用作变量名，但如果在类内部或用`const`声明，则不行。）最简单的方法是避免将这些单词用作标识符，除了`from`、`set`和`target`，它们是安全的并且已经被广泛使用。

```js
as      const      export     get         null     target   void
async   continue   extends    if          of       this     while
await   debugger   false      import      return   throw    with
break   default    finally    in          set      true     yield
case    delete     for        instanceof  static   try
catch   do         from       let         super    typeof
class   else       function   new         switch   var
```

JavaScript 还保留或限制了某些关键字的使用，这些关键字目前尚未被语言使用，但可能在未来版本中使用：

```js
enum  implements  interface  package  private  protected  public
```

由于历史原因，在某些情况下不允许将`arguments`和`eval`用作标识符，并且最好完全避免使用它们。

# 2.5 Unicode

JavaScript 程序使用 Unicode 字符集编写，您可以在字符串和注释中使用任何 Unicode 字符。为了便于移植和编辑，通常在标识符中仅使用 ASCII 字母和数字。但这只是一种编程约定，语言允许在标识符中使用 Unicode 字母、数字和表意文字（但不允许使用表情符号）。这意味着程序员可以使用数学符号和非英语语言中的单词作为常量和变量：

```js
const π = 3.14;
const sí = true;
```

## 2.5.1 Unicode 转义序列

一些计算机硬件和软件无法显示、输入或正确处理完整的 Unicode 字符集。为了支持使用较旧技术的程序员和系统，JavaScript 定义了转义序列，允许我们仅使用 ASCII 字符编写 Unicode 字符。这些 Unicode 转义以字符`\u`开头，后面要么跟着恰好四个十六进制数字（使用大写或小写字母 A-F），要么是由一个到六个十六进制数字括在花括号内。这些 Unicode 转义可能出现在 JavaScript 字符串文字、正则表达式文字和标识符中（但不出现在语言关键字中）。例如，字符“é”的 Unicode 转义是`\u00E9`；以下是三种包含此字符的变量名的不同写法：

```js
let café = 1; // Define a variable using a Unicode character
caf\u00e9     // => 1; access the variable using an escape sequence
caf\u{E9}     // => 1; another form of the same escape sequence
```

早期版本的 JavaScript 仅支持四位数转义序列。带有花括号的版本是在 ES6 中引入的，以更好地支持需要超过 16 位的 Unicode 代码点，例如表情符号：

```js
console.log("\u{1F600}");  // Prints a smiley face emoji
```

Unicode 转义也可能出现在注释中，但由于注释被忽略，因此在该上下文中它们仅被视为 ASCII 字符，而不被解释为 Unicode。

## 2.5.2 Unicode 规范化

如果您在 JavaScript 程序中使用非 ASCII 字符，您必须意识到 Unicode 允许以多种方式对相同字符进行编码。例如，字符串“é”可以编码为单个 Unicode 字符`\u00E9`，也可以编码为常规 ASCII 的“e”后跟重音符组合标记`\u0301`。这两种编码在文本编辑器中显示时通常看起来完全相同，但它们具有不同的二进制编码，这意味着 JavaScript 认为它们是不同的，这可能导致非常令人困惑的程序：

```js
const café = 1;  // This constant is named "caf\u{e9}"
const café = 2;  // This constant is different: "cafe\u{301}"
café  // => 1: this constant has one value
café  // => 2: this indistinguishable constant has a different value
```

Unicode 标准定义了所有字符的首选编码，并指定了一种规范化过程，将文本转换为适合比较的规范形式。JavaScript 假定它正在解释的源代码已经被规范化，并且*不*会自行进行任何规范化。如果您计划在 JavaScript 程序中使用 Unicode 字符，您应确保您的编辑器或其他工具对源代码执行 Unicode 规范化，以防止您最终得到不同但在视觉上无法区分的标识符。

# 2.6 可选分号

像许多编程语言一样，JavaScript 使用分号（`;`）来分隔语句（参见第五章）。这对于使代码的含义清晰很重要：没有分隔符，一个语句的结尾可能看起来是下一个语句的开头，反之亦然。在 JavaScript 中，如果两个语句写在不同行上，通常可以省略这两个语句之间的分号。（如果程序的下一个标记是闭合大括号`}`，也可以省略分号。）许多 JavaScript 程序员（以及本书中的代码）使用分号明确标记语句的结尾，即使不需要也是如此。另一种风格是尽可能省略分号，只在需要时使用。无论你选择哪种风格，都应该了解 JavaScript 中可选分号的一些细节。

考虑以下代码。由于两个语句出现在不同行上，第一个分号可以省略：

```js
a = 3;
b = 4;
```

然而，按照以下方式书写，第一个分号是必需的：

```js
a = 3; b = 4;
```

请注意，JavaScript 并不会将每个换行符都视为分号：通常只有在无法解析代码而需要添加隐式分号时，才会将换行符视为分号。更正式地说（稍后描述的三个例外情况），如果下一个非空格字符无法被解释为当前语句的延续，JavaScript 将换行符视为分号。考虑以下代码：

```js
let a
a
=
3
console.log(a)
```

JavaScript 解释这段代码如下：

```js
let a; a = 3; console.log(a);
```

JavaScript 将第一个换行符视为分号，因为它无法解析不带分号的代码`let a a`。第二个`a`可以作为语句`a;`独立存在，但 JavaScript 不会将第二个换行符视为分号，因为它可以继续解析较长的语句`a = 3;`。

这些语句终止规则会导致一些令人惊讶的情况。这段代码看起来像是两个用换行符分隔的独立语句：

```js
let y = x + f
(a+b).toString()
```

但是代码的第二行括号可以被解释为从第一行调用`f`的函数调用，JavaScript 会这样解释代码：

```js
let y = x + f(a+b).toString();
```

很可能这并不是代码作者打算的解释。为了作为两个独立语句工作，这种情况下需要一个显式分号。

一般来说，如果语句以`(`、`[`、`/`、`+`或`-`开头，那么它可能被解释为前一个语句的延续。以`/`、`+`和`-`开头的语句在实践中相当罕见，但以`(`和`[`开头的语句在某些 JavaScript 编程风格中并不罕见。一些程序员喜欢在这类语句的开头放置一个防御性分号，以便即使修改了其前面的语句并删除了先前的分号，它仍将正确工作：

```js
let x = 0                         // Semicolon omitted here
;[x,x+1,x+2].forEach(console.log) // Defensive ; keeps this statement separate
```

有三个例外情况不符合 JavaScript 将换行符解释为分号的一般规则，即当它无法将第二行解析为第一行语句的延续时。第一个例外涉及`return`、`throw`、`yield`、`break`和`continue`语句（参见第五章）。这些语句通常是独立的，但有时会跟随标识符或表达式。如果这些单词之后（在任何其他标记之前）出现换行符，JavaScript 将始终将该换行符解释为分号。例如，如果你写：

```js
return
true;
```

JavaScript 假设你的意思是：

```js
return; true;
```

然而，你可能的意思是：

```js
return true;
```

这意味着你不能在`return`、`break`或`continue`与后面的表达式之间插入换行符。如果插入换行符，你的代码很可能会以难以调试的非明显方式失败。

第二个例外涉及`++`和`−−`运算符（§4.8）。这些运算符可以是前缀运算符，出现在表达式之前，也可以是后缀运算符，出现在表达式之后。如果要将这些运算符之一用作后缀运算符，它们必须出现在应用于的表达式的同一行上。第三个例外涉及使用简洁的“箭头”语法定义的函数：`=>`箭头本身必须出现在参数列表的同一行上。

# 2.7 总结

本章展示了 JavaScript 程序是如何在最低级别编写的。下一章将带我们迈向更高一级，并介绍作为 JavaScript 程序计算的基本单位的原始类型和值（数字、字符串等）。
