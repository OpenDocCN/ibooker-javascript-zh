# 第三章：数字

在日常编程中，很少有比数字更重要的要素。许多现代语言都有一组不同的数值数据类型，用于不同的场景，如整数、小数、浮点值等。但是当涉及到数字时，JavaScript 作为一种松散类型的脚本语言，揭示了其匆忙、稍微临时的创建。

直到最近，JavaScript 只有一个称为 `Number` 的全能数值数据类型。如今，它有两种：您几乎所有时候使用的标准 `Number`，以及只在需要处理巨大整数时考虑的非常专业化的 `BigInt`。在本章中，您将同时使用这两种类型，以及 `Math` 对象的实用方法。

# 生成随机数

## 问题

你想生成一个落在一定范围内的随机整数（例如，从 1 到 6）。

## 解决方案

您可以使用 `Math.random()` 方法生成介于 0 和 1 之间的浮点值。通常，您会缩放这个小数值并四舍五入，这样您就会得到一个特定范围内的整数。假设您的范围从某个最小数 `min` 到最大数 `max`，这是您需要的语句：

```
randomNumber = Math.floor(Math.random() * (max - min + 1) ) + min;
```

例如，如果您想在 1 到 6 之间选择一个随机数，代码如下：

```
const randomNumber = Math.floor(Math.random()*6) + 1;
```

现在 `randomNumber` 的可能值是 1、2、3、4、5 或 6。

## 讨论

`Math` 对象中充满了您随时可以调用的静态实用方法。本示例使用 `Math.random()` 获取一个随机小数，然后使用 `Math.floor()` 截断小数部分，得到一个整数。

要理解这是如何工作的，让我们考虑一个示例运行。首先，`Math.random()` 选择一个值介于 0 和 1 之间，如 *0.374324823*：

```
const randomNumber = Math.floor(0.374324823*6) + 1;
```

该数字乘以您范围内的值的数量（在本例中为 6），变为 *2.245948938*：

```
const randomNumber = Math.floor(2.245948938) + 1;
```

然后 `Math.floor()` 函数将其截断为 *2*：

```
const randomNumber = 2 + 1;
```

最后，将范围的起始数字相加，得到最终结果为 *3*。重复这个计算，你会得到一个不同的数字，但它将始终是我们设置的范围从 1 到 6 的整数。

## 参见

`Math.floor()` 方法只是一个四舍五入数字的方法之一。更多信息请参见“四舍五入到特定小数位”。

重要的是要理解 `Math.random()` 生成的数字是*伪随机*的，这意味着它们可以被猜测或逆向工程。它们对于密码学、彩票或复杂建模来说不够随机。有关差异的更多信息，请参见“生成密码安全的随机数”。如果您需要一种生成可重复序列的伪随机数的方法，请参考“额外：构建可重复的伪随机数生成器”。

# 生成密码安全的随机数

## 问题

你想创建一个不容易被逆向工程（猜测）的随机数。

## 解决方案

使用`window.crypto`属性来获取`Crypto`对象的一个实例。使用`Crypto.getRandomValues()`方法来生成比`Math.random()`产生的具有更多*熵*的随机值。（换句话说，它们很不可能被重复或预测到—详细内容请参见讨论部分。）

`Crypto.getRandomValues()`方法的工作方式与`Math.random()`不同。它不是给您一个在 0 到 1 之间的浮点数，而是用随机整数填充一个数组。您可以选择这些整数是 8 位、16 位还是 32 位，以及它们是有符号还是无符号的。（有符号数据类型可以是负数或正数，而无符号数仅为正数。）

有一个被接受的解决方案来将`getRandomValues()`的输出转换为 0 到 1 之间的分数值。诀窍是将随机值除以数据类型可以包含的最大可能数字：

```
const randomBuffer = new Uint32Array(1);
window.crypto.getRandomValues(randomBuffer);
const randomFraction = randomBuffer[0] / (0xffffffff + 1);
```

现在您可以像处理从`Math.random()`返回的数字一样处理`randomFraction`。例如，您可以将其转换为特定范围内的随机整数，如“生成随机数”中所述：

```
// Use the random fraction to make a random integer from 1-6
const randomNumber = Math.floor(randomFraction*6) + 1;
console.log(randomNumber);
```

如果您在 Node.js 运行时环境中运行代码，则无法访问`window`对象。但是，您可以使用以下代码访问非常类似的 Web Crypto API 实现：

```
const crypto = require('crypto').webcrypto;
```

## 讨论

此示例中有很多需要解开的内容。首先，即使您不深入研究这段代码的*工作方式*，您也需要了解关于`Crypto.getRandomValues()`实现的一些重要细节：

+   从技术上讲，`Crypto`创建的是由数学公式生成的伪随机数，就像`Math.random()`提供的那样。不同之处在于，这些数字被认为是*密码学强*的，因为随机数生成器的种子是用真正随机的值生成的。这种权衡的好处是`getRandomValues()`的性能类似于`Math.random()`。（它很快。）

+   由于`Crypto`对象的种子是由实现（对于网页代码来说，这意味着浏览器制造商）决定的，因此无法知道它是如何生成的，而这又依赖于操作系统的功能。通常情况下，种子是通过键盘定时、鼠标移动和硬件读数的最近记录细节的组合来创建的。

+   无论您的随机数有多好，如果您的 JavaScript 代码在浏览器中运行，它都会受到大量攻击的威胁。毕竟，没有什么可以阻止恶意方看到您的代码并创建一个绕过所有随机数生成的修改副本。如果您的代码在服务器上运行，情况就不同了。

现在让我们更仔细地看一下`getRandomValues()`的工作原理。在调用`getRandomValues()`之前，您必须创建一个*类型化数组*，它是一个类似数组的对象，只能保存特定数据类型的值。（我们说它是*类数组*，因为它的行为像数组，但它不是`Array`类型的实例。）JavaScript 提供了几种强类型化的数组对象供您使用，比如`Uint32Array`（用于存储无符号 32 位整数的数组），`Uint16Array`，`Uint8Array`，以及有符号的对应物`Int32Array`，`Int16Array`，`Int8Array`。您可以根据需要创建这个数组的大小，`getRandomValues()`将填充整个缓冲区。

在此示例中，我们只为`Uint32Array`中的一个值留出空间：

```
const randomBuffer = new Uint32Array(1);
window.crypto.getRandomValues(randomBuffer);
```

最后一步是将这个随机值除以可能的最大无符号 32 位整数，即 4,294,967,295。这个数字在其十六进制表示中更为整洁，为`0xffffffff`：

```
const randomFraction = randomBuffer[0] / (0xffffffff + 1);
```

正如这段代码所示，您还需要将最大值加 1。这是因为随机值理论上可能恰好落在最大整数值上。如果确实如此，`randomFraction`将变为 1，这与`Math.random()`和大多数其他随机数生成器不同。（微小的与规范不符的变化可能导致错误的假设，进而导致后续错误。）

# 将数字四舍五入到特定小数位

## 问题

您想要将数字四舍五入到特定精度（例如，将 124.793 四舍五入到 124.80 或 120）。

## 解决方案

您可以使用`Math.round()`方法将数字四舍五入到最接近的整数：

```
const fractionalNumber = 19.48938;
const roundedNumber = Math.round(fractionalNumber);

// Now roundedNumber is 19
```

奇怪的是，`round()`方法没有接受可以指定保留小数位数的参数。如果您想要不同精度的结果，您需要自行将数值乘以适当的 10 的幂，四舍五入后再除以相同的 10 的幂。以下是此操作的一般公式：

```
const numberToRound = fractionalNumber * (10**numberOfDecimalPlaces);
let roundedNumber = Math.round(numberToRound);
roundedNumber = roundedNumber / (10**numberOfDecimalPlaces);
```

例如，如果您想要将数字四舍五入到两位小数，代码如下：

```
const fractionalNumber = 19.48938;
const numberToRound = fractionalNumber * (10**2);
let roundedNumber = Math.round(numberToRound);
roundedNumber = roundedNumber / (10**2);

// Now roundedNumber is 19.49
```

如果您想要将小数点左边的数字四舍五入（例如，四舍五入到最接近的十位、百位等），只需对`numberOfDecimalPlaces`使用负数即可。例如，-1 四舍五入到最接近的十位，-2 四舍五入到最接近的百位，依此类推。

## 讨论

`Math`对象有几个静态方法，用于将分数值转换为整数。`floor()`方法删除所有小数位，将一个数向下舍入到最接近的整数。`ceil()`方法则相反，总是将分数数值向上舍入到最接近的整数。`round()`方法将数值四舍五入到最接近的整数。

关于`round()`如何工作，有两点重要的内容您需要知道：

+   精确值 0.5 始终四舍五入为最接近的整数，即使它与下一个较低和较高整数的距离相等。在金融和科学中，通常使用不同的舍入技术来消除这种偏差（例如将某些 0.5 值向上舍入，将其他值向下舍入）。但是，如果你希望 JavaScript 中有这种行为，你需要自己实现或使用第三方库。

+   在对负数进行舍入时，JavaScript 会将–0.5 向零*舍入*。这意味着–4.5 会被舍入为–4，这与许多其他编程语言的舍入实现不同。

## 参见

舍入数字是将数值接近适当显示格式的一种方法。如果你正在使用舍入准备向用户显示的数字，你可能也对在“将数值转换为格式化字符串”中描述的`Number`格式化方法感兴趣。

# 在十进制值中保持精度

## 问题

JavaScript 中的所有数字都是浮点数值，某些操作可能会产生微小的四舍五入误差。在某些应用中（例如处理金额时），这些错误可能是不可接受的。

## 解决方案

浮点数的四舍五入误差是一个众所周知的现象，几乎存在于每一种编程语言中。要在 JavaScript 中看到它，请运行以下代码：

```
const sum = 0.1 + 0.2;
console.log(sum);      // displays 0.30000000000000004
```

你无法避免四舍五入误差，但可以将其最小化。如果你正在处理有两位小数精度的货币类型（如美元），考虑将所有值乘以 100 以避免处理小数。而不是编写如下代码：

```
const currentBalance = 5382.23;
const transactionAmount = 14.02;

const updatedBalance = currentBalance - transactionAmount;

// Now updatedBalance = 5368.209999999999
```

像这样使用货币变量：

```
const currentBalanceInCents = 538223;
const transactionAmountInCents = 1402;

const updatedBalanceInCents = currentBalanceInCents - transactionAmountInCents;

// Now updatedBalanceInCents = 536821
```

这解决了像加减几个分数值得到精确整数时的操作问题。但是，当你需要计算税款或利息时会发生什么？在这些情况下，无论如何都会得到分数值，并且在交易后立即对数值进行四舍五入，这就是企业和银行的做法：

```
const costInCents = 4899;

// Calculate 11% tax, and round the result to the nearest cent
const costWithTax = Math.round(costInCents*1.11);
```

## 讨论

浮点数的四舍五入问题源于某些十进制值无法在二进制表示中存储而不四舍五入的事实。与十进制数系统相同的问题（例如尝试写出 1/3 的结果）。浮点数的区别在于效果是*反直觉*的。我们不期望在加 0.1 和 0.2 时会有问题，因为在十进制表示法中这两个分数都可以被精确表示。

尽管其他编程语言也会经历同样的现象，但其中许多包含了专门的十进制或货币值数据类型。JavaScript 则不包括。然而，有一个提案用于[新的十进制类型](https://github.com/tc39/proposal-decimal)，可能会被纳入未来版本的 JavaScript 语言中。

## 参见

如果你进行大量的财务计算，可以通过使用像 [*bignumber.js*](https://github.com/MikeMcl/bignumber.js) 这样的第三方库简化你的生活，它提供了一个定制的数值数据类型，几乎与普通的 `Number` 类似，但保留了固定数量小数位的精确精度。

# 将字符串转换为数字

## 问题

你想要解析字符串中的数字并将其转换为数字数据类型。

## 解决方案

将数字转换为字符串是安全的，因为这个操作不会失败。反向任务——将字符串转换为数字，以便在计算中使用——则需要更谨慎。

使用 `Number()` 函数是最常见的方法：

```
const stringData = '42';
const numberData = Number(stringData);
```

`Number()` 函数不会接受带有货币符号和逗号分隔符的格式。它允许字符串开头和结尾的额外空格。`Number()` 函数还会将空字符串或仅包含空白字符的字符串转换为数字 0。这可能是一个合理的默认值（例如，如果你从文本框中获取用户输入），但并非总是合适的。为了避免这种情况，在调用 `Number()` 之前，请考虑检测是否为仅包含空白字符的字符串：

```
if (stringData.trim() === '') {
  // This is an all-whitespace or empty string
}
```

如果转换失败，`Number()` 函数会将值 `NaN`（即 *非数*）分配给你的变量。你可以在使用 `Number()` 后立即调用 `Number.isNaN()` 方法来测试此失败情况：

```
const numberData = Number(stringData);

if (Number.isNaN(numberData)) {
  // It's safe to process this data as a number
}
```

###### 注意

`isFinite()` 方法与 `isNaN()` 几乎相同，但避免了奇怪的边缘情况，例如 `1/0`，它返回一个值为 `infinity`。如果你在 `infinity` 上使用 `isNaN()` 方法，它会返回 `false`，这有些难以置信。

另一种方法是使用 `parseFloat()` 方法。它是一种稍微宽松的转换，可以容忍数字后的文本。然而，`parseFloat()` 对空字符串更为严格，它会拒绝这种情况。

```
console.log(Number('42'));               // 42
console.log(parseFloat('42'));           // 42

console.log(Number('12 goats'));         // NaN
console.log(parseFloat('12 goats'));     // 12

console.log(Number('goats 12'));         // NaN
console.log(parseFloat('goats 12'));     // NaN

console.log(Number('2001/01/01'));       // NaN
console.log(parseFloat('2001/01/01'));   // 2001

console.log(Number(' '));                // 0
console.log(parseFloat(' '));            // NaN
```

## 讨论

开发者使用一些与 `Number()` 函数功能上等效的转换技巧，例如将字符串乘以 1 (`numberInString*1`) 或使用一元操作符 (`+numberInString`)。但为了清晰起见，推荐使用 `Number()` 或 `parseFloat()`。

如果你有一个格式化的数字（例如 *2,300*），你需要做更多工作来进行转换。`Number()` 方法会返回 `NaN`，而 `parseFloat()` 会在逗号处停止，并将其视为 2。不幸的是，尽管 JavaScript 有一个 `Intl.NumberFormat` 对象，可以从数字创建格式化字符串（参见 “将数值转换为格式化字符串”），但它并未提供反向操作的解析功能。

您可以使用正则表达式来处理诸如从字符串中删除逗号的任务（参见“替换字符串的所有出现”）。但是自制解决方案可能存在风险，因为某些语言环境使用逗号分隔千位数，而其他语言环境使用逗号分隔小数。在这种情况下，像[Numeral](http://numeraljs.com)这样经过广泛使用和测试的 JavaScript 库更加合适。

# 将十进制数转换为十六进制值

## 问题

您有一个十进制值，需要找到其十六进制等效值。

## 解决方法

使用`Number.toString()`方法，使用指定转换为的基数参数*：

```
const num = 255;

// displays ff, which is hexadecimal equivalent for 255
console.log(num.toString(16));
```

## 讨论

默认情况下，JavaScript 中的数字是十进制的。但是，它们也可以转换为不同的*基数*，包括十六进制（16）和八进制（8）。十六进制数字以`0x`开头（零后跟小写 x）。八进制数字过去以零（0）开头，但现在应该以零开始，然后是拉丁字母*O*（大写或小写）：

```
const octalNumber = 0o255;  // equivalent to 173 decimal
const hexaNumber = 0xad;    // equivalent to 173 decimal
```

十进制数可以转换为 2 到 36 之间的其他基数：

```
const decNum = 55;
const octNum = decNum.toString(8);   // value of 67 octal
const hexNum = decNum.toString(16);  // value of 37 hexadecimal
const binNum = decNum.toString(2);   // value of 110111 binary
```

要完成八进制和十六进制的表示，您需要将`0o`连接到八进制，并将`0x`连接到十六进制值。但是请记住，一旦您将数字转换为字符串，无论其格式如何，都不要期望可以在任何数值计算中使用它。

尽管小数可以转换为任何基数（范围为 2 到 36 之间），但只有八进制、十六进制和十进制数字可以直接作为数字进行操作。

# 在度数和弧度之间进行转换

## 问题

您有一个以度数表示的角度。要在`Math`对象的三角函数中使用该值，您需要将度数转换为弧度。

## 解决方法

要将度数转换为弧度，请将度数值乘以`(Math.PI/180)`：

```
const radians = degrees * (Math.PI / 180);
```

因此，如果您有一个 90 度的角度，计算如下：

```
const radians = 90 * (Math.PI / 180);
console.log(radians);   // 1.5707963267948966
```

要将弧度转换为度数，请将弧度值乘以`(180/Math.PI)`：

```
const degrees = radians * (180 / Math.PI);
```

## 讨论

`Math`对象的所有三角函数方法（`sin()`、`cos()`、`tan()`、`asin()`、`acos()`、`atan()`和`atan2()`）接受弧度值，并返回弧度作为结果。然而，人们通常提供度数而不是弧度，因为度数是更熟悉的度量单位。

# 计算圆弧的长度

## 问题

给定圆的半径和弧度中的角度，找到弧的长度。

## 解决方法

使用`Math.PI`将度数转换为弧度，并在公式中使用结果以找到弧的长度：

```
// angle of arc is 120 degrees, radius of circle is 2
const radians = degrees * (Math.PI / 180);
const arclength = radians * radius; // value is 4.18879020478...
```

## 讨论

圆弧的长度通过将圆的半径乘以弧度的弧度角来确定。

如果角度以度数给出，则在乘以半径之前，您需要先将度数转换为弧度。这种计算经常用于 SVG 中绘制形状，如第十五章中所述。

# 使用 BigInt 处理非常大的数字

## 问题

如果需要处理非常大的整数（超过 2⁵³），而不丢失精度。

## 解决方案

使用 `BigInt` 数据类型，可以容纳任意大小的整数，仅受系统内存限制（或您使用的 JavaScript 引擎的 `BigInt` 实现）。

您可以通过两种方式创建 `BigInt`。可以使用 `BigInt()` 函数，如下所示：

```
// Create a BigInt and set it to 10
const bigInteger = BigInt(10);
```

或者您可以在数字末尾添加字母 *n*：

```
const bigInteger = 10n;
```

此示例显示了普通 `Number` 和 `BigInt` 在非常大数值上的差异：

```
// Ordinarily, large integers suffer from imprecision
const maxInt = Number.MAX_SAFE_INTEGER // Probably about 9007199254740991
console.log(maxInt + 1);  // 9007199254740992 (reasonable)
console.log(maxInt + 2);  // 9007199254740992 (not a typo, this seems wrong)
console.log(maxInt + 3);  // 9007199254740994 (sure)
console.log(maxInt + 4);  // 9007199254740996 (wait, what now?)

// BigInts behave more reliably
const bigInt = BigInt(maxInt);
console.log(bigInt + 1n);  // 9007199254740992 (as before)
console.log(bigInt + 2n);  // 9007199254740993 (this is better)
console.log(bigInt + 3n);  // 9007199254740994 (still good)
console.log(bigInt + 4n);  // 9007199254740995 (excellent!)
```

###### 注意

当将 `BigInt` 记录到开发者控制台时，它的值会附加 *n*（例如 *9007199254740992n*）。此约定使得 `BigInt` 值易于识别。但如果只想要 `BigInt` 的数值，只需首先将其转换为文本，使用 `BigInt.toString()`。

## 讨论

JavaScript 的原生 `Number` 类型符合 IEEE-754 规范，为 64 位双精度浮点数。该标准具有可接受的已知限制和不准确性。一个实际的限制是，超过 2⁵³ 后无法精确表示整数。超过此点后，之前仅限于小数点右侧的表示不准确性会跳到小数点左侧。换句话说，随着 JavaScript 引擎计数增加，不准确性的可能性增加。一旦超过 2⁵³，不准确性大于 1，并出现在整数计算中，而不仅仅是小数值计算中。

JavaScript 在 ECMAScript 2020 规范中引入了 `BigInt` 类型，部分解决了这个问题。`BigInt` 是一种任意大小的整数，允许您表示极大的数字。实际上，`BigInt` 的位宽没有上限。

几乎所有常规数字可用的运算符都可以用在 `BigInt` 上，包括加法（`+`）、减法（`-`）、乘法（`*`）、除法（`/`）和指数运算（`**`）。但是，`BigInt` 是整数类型，不存储小数值。执行除法操作时，`BigInt` 会静默丢弃小数部分：

```
const result = 10n / 6n;    // result is 1.
```

`BigInt` 和 `Number` 不可互换，也无法互操作。但可以使用 `Number()` 和 `BigInt()` 函数相互转换：

```
let bigInteger = 10n;
let integer = Number(bigInteger);  // Number is 10

integer = 20;
bigInteger = BigInt(integer);      // bigInteger is 20n
```

如果要使用期望 `Number` 的方法与 `BigInt` 一起使用，或者想要在与另一个 `BigInt` 进行计算时使用 `Number`，则需要进行转换。

如果尝试将包含小数值的 `Number` 转换为 `BigInt`，将收到 `RangeError`。可以通过先四舍五入来避免此问题：

```
const decimal = 10.8;
const bigInteger = BigInt(Math.round(decimal));    // bigInteger is 11n
```

记得保持操作与类型一致。有时候看似简单的操作可能会失败，因为你意外地将`BigInt`与普通数字结合在一起：

```
let x = 10n;
x = x * 2;    // throws a TypeError because x is a BigInt and 2 is a Number
x += 1;       // also throws a TypeError

x = x * 2n;   // x is now 20n, as expected
x += 1n;      // x is 21
```

使用标准比较运算符（`<`、`>`、`<=`、`>=`），可以将`BigInt`值与`Number`值进行比较。如果想要测试`BigInt`和数字是否相等，请使用宽松相等运算符（`==`和`!=`）。严格相等运算符（`===`）总是返回`false`，因为`BigInt`和`Number`是不同的数据类型。或者更好的方法是，将你的`Number`显式转换为`BigInt`，然后使用`===`进行比较。

关于`BigInt`还有一件事需要考虑：在发布时，它不能被序列化为 JSON。尝试对`BigInt`调用`JSON.stringify()`会导致语法错误。你有几个解决方案可供考虑。你可以通过适当的`toJSON()`方法对你的`BigInt`实现进行修补：

```
BigInt.prototype.toJSON = function() { return this.toString() }
```

你也可以使用像[granola](https://github.com/kanongil/granola)这样的库，它提供了对多种值（包括`BigInt`）进行 JSON 兼容字符串化的方法。
