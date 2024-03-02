# 你不懂 JS：ES6 与未来 第二章：语法（下）

## 数字字面量扩展

在 ES5 之前，数字字面量看起来就像下面的东西 —— 八进制形式没有被官方指定，唯一被允许的是各种浏览器已经实质上达成一致的一种扩展：

```js
var dec = 42,
    oct = 052,
    hex = 0x2a;
```

**注意：** 虽然你用不同的进制来指定一个数字，但是数字的数学值才是被存储的东西，而且默认的输出解释方式总是 10 进制的。前面代码段中的三个变量都在它们当中存储了值`42`。

为了进一步说明`052`是一种非标准形式扩展，考虑如下代码：

```js
Number( "42" );                // 42
Number( "052" );            // 52
Number( "0x2a" );            // 42
```

ES5 继续允许这种浏览器扩展的八进制形式（包括这样的不一致性），除了在 strict 模式下，八进制字面量（`052`）是不允许的。做出这种限制的主要原因是，许多开发者似乎习惯于下意识地为了将代码对齐而在十进制的数字前面前缀`0`，然后遭遇他们完全改变了数字的值的意外！

ES6 延续了除十进制数字之外的数字字面量可以被表示的遗留的改变/种类。现在有了一种官方的八进制形式，一种改进了的十六进制形式，和一种全新的二进制形式。由于 Web 兼容性的原因，在非 strict 模式下老式的八进制形式`052`将继续是合法的，但其实应当永远不再被使用了。

这些是新的 ES6 数字字面形式：

```js
var dec = 42,
    oct = 0o52,            // or `0O52` :(
    hex = 0x2a,            // or `0X2a` :/
    bin = 0b101010;        // or `0B101010` :/
```

唯一允许的小数形式是十进制的。八进制，十六进制，和二进制都是整数形式。

而且所有这些形式的字符串表达形式都是可以被强制转换/变换为它们的数字等价物的：

```js
Number( "42" );            // 42
Number( "0o52" );        // 42
Number( "0x2a" );        // 42
Number( "0b101010" );    // 42
```

虽然严格来说不是 ES6 新增的，但一个鲜为人知的事实是你其实可以做反方向的转换（好吧，某种意义上的）：

```js
var a = 42;

a.toString();            // "42" —— 也可使用`a.toString( 10 )`
a.toString( 8 );        // "52"
a.toString( 16 );        // "2a"
a.toString( 2 );        // "101010"
```

事实上，以这种方你可以用从`2`到`36`的任何进制表达一个数字，虽然你会使用标准进制 —— 2，8，10，和 16 ——之外的情况非常少见。

## Unicode

我只能说这一节不是一个穷尽了“关于 Unicode 你想知道的一切”的资料。我想讲解的是，你需要知道在 ES6 中对 Unicode 改变了什么，但是我们不会比这深入太多。Mathias Bynens ([`twitter.com/mathias`](http://twitter.com/mathias)) 大量且出色地撰写/讲解了关于 JS 和 Unicode (参见 [`mathiasbynens.be/notes/javascript-unicode`](https://mathiasbynens.be/notes/javascript-unicode) 和 [`fluentconf.com/javascript-html-2015/public/content/2015/02/18-javascript-loves-unicode)。`](http://fluentconf.com/javascript-html-2015/public/content/2015/02/18-javascript-loves-unicode)%E3%80%82)

从`0x0000`到`0xFFFF`范围内的 Unicode 字符包含了所有的标准印刷字符（以各种语言），它们都是你可能看到过和互动过的。这组字符被称为 *基本多文种平面（Basic Multilingual Plane (BMP)）*。BMP 甚至包含像这个酷雪人一样的有趣字符: ☃ (U+2603)。

在这个 BMP 集合之外还有许多扩展的 Unicode 字符，它们的范围一直到`0x10FFFF`。这些符号经常被称为 *星形（astral）* 符号，这正是 BMP 之外的字符的 16 组 *平面* （也就是，分层/分组）的名称。星形符号的例子包括𝄞 （U+1D11E）和💩 （U+1F4A9）。

在 ES6 之前，JavaScript 字符串可以使用 Unicode 转义来指定 Unicode 字符，例如：

```js
var snowman = "\u2603";
console.log( snowman );            // "☃"
```

然而，`\uXXXX`Unicode 转义仅支持四个十六进制字符，所以用这种方式表示你只能表示 BMP 集合中的字符。要在 ES6 以前使用 Unicode 转义表示一个星形字符，你需要使用一个 *代理对（surrogate pair）* —— 基本上是两个经特殊计算的 Unicode 转义字符放在一起，被 JS 解释为一个单独星形字符：

```js
var gclef = "\uD834\uDD1E";
console.log( gclef );            // "𝄞"
```

在 ES6 中，我们现在有了一种 Unicode 转义的新形式（在字符串和正则表达式中），称为 Unicode *代码点转义*：

```js
var gclef = "\u{1D11E}";
console.log( gclef );            // "𝄞"
```

如你所见，它的区别是出现在转义序列中的`{ }`，它允许转义序列中包含任意数量的十六进制字符。因为你只需要六个就可以表示在 Unicode 中可能的最高代码点（也就是，0x10FFFF），所以这是足够的。

### Unicode 敏感的字符串操作

在默认情况下，JavaScript 字符串操作和方法对字符串值中的星形符号是不敏感的。所以，它们独立地处理每个 BMP 字符，即便是可以组成一个单独字符的两半代理。考虑如下代码：

```js
var snowman = "☃";
snowman.length;                    // 1

var gclef = "𝄞";
gclef.length;                    // 2
```

那么，我们如何才能正确地计算这样的字符串的长度呢？在这种场景下，下面的技巧可以工作：

```js
var gclef = "𝄞";

[...gclef].length;                // 1
Array.from( gclef ).length;        // 1
```

回想一下本章早先的“`for..of`循环”一节，ES6 字符串拥有内建的迭代器。这个迭代器恰好是 Unicode 敏感的，这意味着它将自动地把一个星形符号作为一个单独的值输出。我们在一个数组字面量上使用扩散操作符`...`，利用它创建了一个字符串符号的数组。然后我们只需检查这个结果数组的长度。ES6 的`Array.from(..)`基本上与`[...XYZ]`做的事情相同，不过我们将在第六章中讲解这个工具的细节。

**警告：** 应当注意的是，相对地讲，与理论上经过优化的原生工具/属性将做的事情比起来，仅仅为了得到一个字符串的长度就构建并耗尽一个迭代器在性能上的代价是高昂的。

不幸的是，完整的答案并不简单或直接。除了代理对（字符串迭代器可以搞定的），一些特殊的 Unicode 代码点有其他特殊的行为，解释起来非常困难。例如，有一组代码点可以修改前一个相邻的字符，称为 *组合变音符号（Combining Diacritical Marks）*

考虑这两个数组的输出：

```js
console.log( s1 );                // "é"
console.log( s2 );                // "é"
```

它们看起来一样，但它们不是！这是我们如何创建`s1`和`s2`的：

```js
var s1 = "\xE9",
    s2 = "e\u0301";
```

你可能猜到了，我们前面的`length`技巧对`s2`不管用：

```js
[...s1].length;                    // 1
[...s2].length;                    // 2
```

那么我们能做什么？在这种情况下，我们可以使用 ES6 的`String#normalize(..)`工具，在查询这个值的长度前对它实施一个 *Unicode 正规化操作*：

```js
var s1 = "\xE9",
    s2 = "e\u0301";

s1.normalize().length;            // 1
s2.normalize().length;            // 1

s1 === s2;                        // false
s1 === s2.normalize();            // true
```

实质上，`normalize(..)`接受一个`"e\u0301"`这样的序列，并把它正规化为`\xE9`。正规化甚至可以组合多个相邻的组合符号，如果存在适合他们组合的 Unicode 字符的话：

```js
var s1 = "o\u0302\u0300",
    s2 = s1.normalize(),
    s3 = "ồ";

s1.length;                        // 3
s2.length;                        // 1
s3.length;                        // 1

s2 === s3;                        // true
```

不幸的是，这里的正规化也不完美。如果你有多个组合符号在修改一个字符，你可能不会得到你所期望的长度计数，因为一个被独立定义的，可以表示所有这些符号组合的正规化字符可能不存在。例如：

```js
var s1 = "e\u0301\u0330";

console.log( s1 );                // "ḛ́"

s1.normalize().length;            // 2
```

你越深入这个兔子洞，你就越能理解要得到一个“长度”的精确定义是很困难的。我们在视觉上看到的作为一个单独字符绘制的东西 —— 更精确地说，它称为一个 *字形* —— 在程序处理的意义上不总是严格地关联到一个单独的“字符”上。

**提示：** 如果你就是想看看这个兔子洞有多深，看看“字形群集边界（Grapheme Cluster Boundaries）”算法([`www.Unicode.org/reports/tr29/#Grapheme_Cluster_Boundaries)。`](http://www.Unicode.org/reports/tr29/#Grapheme_Cluster_Boundaries)%E3%80%82)

### 字符定位

与长度的复杂性相似，“在位置 2 上的字符是什么？”，这么问的意思究竟是什么？前 ES6 的原生答案来自`charAt(..)`，它不会遵守一个星形字符的原子性，也不会考虑组合符号。

考虑如下代码：

```js
var s1 = "abc\u0301d",
    s2 = "ab\u0107d",
    s3 = "ab\u{1d49e}d";

console.log( s1 );                // "abćd"
console.log( s2 );                // "abćd"
console.log( s3 );                // "ab𝒞d"

s1.charAt( 2 );                    // "c"
s2.charAt( 2 );                    // "ć"
s3.charAt( 2 );                    // "" <-- 不可打印的代理字符
s3.charAt( 3 );                    // "" <-- 不可打印的代理字符
```

那么，ES6 会给我们 Unicode 敏感版本的`charAt(..)`吗？不幸的是，不。在本书写作时，在后 ES6 的考虑之中有一个这样的工具的提案。

但是使用我们在前一节探索的东西（当然也带着它的限制！），我们可以黑一个 ES6 的答案：

```js
var s1 = "abc\u0301d",
 s2 = "ab\u0107d",
 s3 = "ab\u{1d49e}d";

[...s1.normalize()][2];            // "ć"
[...s2.normalize()][2];            // "ć"
[...s3.normalize()][2];            // "𝒞"
```

**警告：** 提醒一个早先的警告：在每次你想得到一个单独的字符时构建并耗尽一个迭代器……在性能上不是很理想。对此，希望我们很快能在后 ES6 时代得到一个内建的，优化过的工具。

那么`charCodeAt(..)`工具的 Unicode 敏感版本呢？ES6 给了我们`codePointAt(..)`：

```js
var s1 = "abc\u0301d",
    s2 = "ab\u0107d",
    s3 = "ab\u{1d49e}d";

s1.normalize().codePointAt( 2 ).toString( 16 );
// "107"

s2.normalize().codePointAt( 2 ).toString( 16 );
// "107"

s3.normalize().codePointAt( 2 ).toString( 16 );
// "1d49e"
```

那么从另一个方向呢？`String.fromCharCode(..)`的 Unicode 敏感版本是 ES6 的`String.fromCodePoint(..)`：

```js
String.fromCodePoint( 0x107 );        // "ć"

String.fromCodePoint( 0x1d49e );    // "𝒞"
```

那么等一下，我们能组合`String.fromCodePoint(..)`与`codePointAt(..)`来得到一个刚才的 Unicode 敏感`charAt(..)`的更好版本吗？是的！

```js
var s1 = "abc\u0301d",
    s2 = "ab\u0107d",
    s3 = "ab\u{1d49e}d";

String.fromCodePoint( s1.normalize().codePointAt( 2 ) );
// "ć"

String.fromCodePoint( s2.normalize().codePointAt( 2 ) );
// "ć"

String.fromCodePoint( s3.normalize().codePointAt( 2 ) );
// "𝒞"
```

还有好几个字符串方法我们没有在这里讲解，包括`toUpperCase()`，`toLowerCase()`，`substring(..)`，`indexOf(..)`，`slice(..)`，以及其他十几个。它们中没有任何一个为了完全支持 Unicode 而被改变或增强过，所以在处理含有星形符号的字符串是，你应当非常小心 —— 可能干脆回避它们！

还有几个字符串方法为了它们的行为而使用正则表达式，比如`replace(..)`和`match(..)`。值得庆幸的是，ES6 为正则表达式带来了 Unicode 支持，正如我们在本章早前的“Unicode 标志”中讲解过的那样。

好了，就是这些！有了我们刚刚讲过的各种附加功能，JavaScript 的 Unicode 字符串支持要比前 ES6 时代好太多了（虽然还不完美）。

### Unicode 标识符名称

Unicode 还可以被用于标识符名称（变量，属性，等等）。在 ES6 之前，你可以通过 Unicode 转义这么做，比如：

```js
var \u03A9 = 42;

// 等同于：var Ω = 42;
```

在 ES6 中，你还可以使用前面讲过的代码点转义语法：

```js
var \u{2B400} = 42;

// 等同于：var 𫐀 = 42;
```

关于究竟哪些 Unicode 字符被允许使用，有一组复杂的规则。另外，有些字符只要不是标识符名称的第一个字符就允许使用。

**注意：** 关于所有这些细节，Mathias Bynens 写了一篇了不起的文章 ([`mathiasbynens.be/notes/javascript-identifiers-es6)。`](https://mathiasbynens.be/notes/javascript-identifiers-es6)%E3%80%82)

很少有理由，或者是为了学术上的目的，才会在标识符名称中使用这样不寻常的字符。你通常不会因为依靠这些深奥的功能编写代码而感到舒服。

## Symbol

在 ES6 中，长久以来首次，有一个新的基本类型被加入到了 JavaScript：`symbol`。但是，与其他的基本类型不同，symbol 没有字面形式。

这是你如何创建一个 symbol：

```js
var sym = Symbol( "some optional description" );

typeof sym;        // "symbol"
```

一些要注意的事情是：

*   你不能也不应该将`new`与`Symbol(..)`一起使用。它不是一个构造器，你也不是在产生一个对象。
*   被传入`Symbol(..)`的参数是可选的。如果传入的话，它应当是一个字符串，为 symbol 的目的给出一个友好的描述。
*   `typeof`的输出是一个新的值（`"symbol"`），这是识别一个 symbol 的主要方法。

如果描述被提供的话，它仅仅用于 symbol 的字符串化表示：

```js
sym.toString();        // "Symbol(some optional description)"
```

与基本字符串值如何不是`String`的实例的原理很相似，symbol 也不是`Symbol`的实例。如果，由于某些原因，你想要为一个 symbol 值构建一个封箱的包装器对像，你可以做如下的事情：

```js
sym instanceof Symbol;        // false

var symObj = Object( sym );
symObj instanceof Symbol;    // true

symObj.valueOf() === sym;    // true
```

**注意：** 在这个代码段中的`symObj`和`sym`是可以互换使用的；两种形式可以在 symbol 被用到的地方使用。没有太多的理由要使用封箱的包装对象形式（`symObj`），而不用基本类型形式（`sym`）。和其他基本类型的建议相似，使用`sym`而非`symObj`可能是最好的。

一个 symbol 本身的内部值 —— 称为它的`name` —— 被隐藏在代码之外而不能取得。你可以认为这个 symbol 的值是一个自动生成的，（在你的应用程序中）独一无二的字符串值。

但如果这个值是隐藏且不可取得的，那么拥有一个 symbol 还有什么意义？

一个 symbol 的主要意义是创建一个不会和其他任何值冲突的类字符串值。所以，举例来说，可以考虑将一个 symbol 用做表示一个事件的名称的值：

```js
const EVT_LOGIN = Symbol( "event.login" );
```

然后你可以在一个使用像`"event.login"`这样的一般字符串字面量的地方使用`EVT_LOGIN`：

```js
evthub.listen( EVT_LOGIN, function(data){
    // ..
} );
```

其中的好处是，`EVT_LOGIN`持有一个不能被其他任何值所（有意或无意地）重复的值，所以在哪个事件被分发或处理的问题上不可能存在任何含糊。

**注意：** 在前面的代码段的幕后，几乎可以肯定地认为`evthub`工具使用了`EVT_LOGIN`参数值的 symbol 值作为某个跟踪事件处理器的内部对象的属性/键。如果`evthub`需要将 symbol 值作为一个真实的字符串使用，那么它将需要使用`String(..)`或者`toString(..)`进行明确强制转换，因为 symbol 的隐含字符串强制转换是不允许的。

你可能会将一个 symbol 直接用做一个对象中的属性名/键，如此作为一个你想将之用于隐藏或元属性的特殊属性。重要的是，要知道虽然你试图这样对待它，但是它 *实际上* 并不是隐藏或不可接触的属性。

考虑这个实现了 *单例* 模式行为的模块 —— 也就是，它仅允许自己被创建一次：

```js
const INSTANCE = Symbol( "instance" );

function HappyFace() {
    if (HappyFace[INSTANCE]) return HappyFace[INSTANCE];

    function smile() { .. }

    return HappyFace[INSTANCE] = {
        smile: smile
    };
}

var me = HappyFace(),
    you = HappyFace();

me === you;            // true
```

这里的 symbol 值`INSTANCE`是一个被静态地存储在`HappyFace()`函数对象上的特殊的，几乎是隐藏的，类元属性。

替代性地，它本可以是一个像`__instance`这样的普通属性，而且其行为将会是一模一样的。symbol 的使用仅仅增强了程序元编程的风格，将这个`INSTANCE`属性与其他普通的属性间保持隔离。

### Symbol 注册表

在前面几个例子中使用 symbol 的一个微小的缺点是，变量`EVT_LOGIN`和`INSTANCE`不得不存储在外部作用域中（甚至也许是全局作用域），或者用某种方法存储在一个可用的公共位置，这样代码所有需要使用这些 symbol 的部分都可以访问它们。

为了辅助组织访问这些 symbol 的代码，你可以使用 *全局 symbol 注册表* 来创建 symbol。例如：

```js
const EVT_LOGIN = Symbol.for( "event.login" );

console.log( EVT_LOGIN );        // Symbol(event.login)
```

和：

```js
function HappyFace() {
    const INSTANCE = Symbol.for( "instance" );

    if (HappyFace[INSTANCE]) return HappyFace[INSTANCE];

    // ..

    return HappyFace[INSTANCE] = { .. };
}
```

`Symbol.for(..)`查询全局 symbol 注册表来查看一个 symbol 是否已经使用被提供的说明文本存储过了，如果有就返回它。如果没有，就创建一个并返回。换句话说，全局 symbol 注册表通过描述文本将 symbol 值看作它们本身的单例。

但这也意味着只要使用匹配的描述名，你的应用程序的任何部分都可以使用`Symbol.for(..)`从注册表中取得 symbol。

讽刺的是，基本上 symbol 的本意是在你的应用程序中取代 *魔法字符串* 的使用（被赋予了特殊意义的随意的字符串值）。但是你正是在全局 symbol 注册表中使用 *魔法* 描述字符串值来唯一识别/定位它们的！

为了避免意外的冲突，你可能想使你的 symbol 描述十分独特。这么做的一个简单的方法是在它们之中包含前缀/环境/名称空间的信息。

例如，考虑一个像下面这样的工具：

```js
function extractValues(str) {
    var key = Symbol.for( "extractValues.parse" ),
        re = extractValues[key] ||
            /[^=&]+?=([^&]+?)(?=&|$)/g,
        values = [], match;

    while (match = re.exec( str )) {
        values.push( match[1] );
    }

    return values;
}
```

我们使用魔法字符串值`"extractValues.parse"`，因为在注册表中的其他任何 symbol 都不太可能与这个描述相冲突。

如果这个工具的一个用户想要覆盖这个解析用的正则表达式，他们也可以使用 symbol 注册表：

```js
extractValues[Symbol.for( "extractValues.parse" )] =
    /..some pattern../g;

extractValues( "..some string.." );
```

除了 symbol 注册表在全局地存储这些值上提供的协助以外，我们在这里看到的一切其实都可以通过将魔法字符串`"extractValues.parse"`作为一个键，而不是一个 symbol，来做到。这其中在元编程的层次上的改进要多于在函数层次上的改进。

你可能偶然会使用一个已经被存储在注册表中的 symbol 值来查询它底层存储了什么描述文本（键）。例如，因为你无法传递 symbol 值本身，你可能需要通知你的应用程序的另一个部分如何在注册表中定位一个 symbol。

你可以使用`Symbol.keyFor(..)`取得一个被注册的 symbol 描述文本（键）：

```js
var s = Symbol.for( "something cool" );

var desc = Symbol.keyFor( s );
console.log( desc );            // "something cool"

// 再次从注册表取得 symbol
var s2 = Symbol.for( desc );

s2 === s;                        // true
```

### Symbols 作为对象属性

如果一个 symbol 被用作一个对象的属性/键，它会被以一种特殊的方式存储，以至这个属性不会出现在这个对象属性的普通枚举中：

```js
var o = {
    foo: 42,
    [ Symbol( "bar" ) ]: "hello world",
    baz: true
};

Object.getOwnPropertyNames( o );    // [ "foo","baz" ]
```

要取得对象的 symbol 属性：

```js
Object.getOwnPropertySymbols( o );    // [ Symbol(bar) ]
```

这表明一个属性 symbol 实际上不是隐藏的或不可访问的，因为你总是可以在`Object.getOwnPropertySymbols(..)`的列表中看到它。

#### 内建 Symbols

ES6 带来了好几种预定义的内建 symbol，它们暴露了在 JavaScript 对象值上的各种元行为。然而，正如人们所预料的那样，这些 symbol *没有* 没被注册到全局 symbol 注册表中。

取而代之的是，它们作为属性被存储到了`Symbol`函数对象中。例如，在本章早先的“`for..of`”一节中，我们介绍了值`Symbol.iterator`：

```js
var a = [1,2,3];

a[Symbol.iterator];            // native function
```

语言规范使用`@@`前缀注释指代内建的 symbol，最常见的几个是：`@@iterator`，`@@toStringTag`，`@@toPrimitive`。还定义了几个其他的 symbol，虽然他们可能不那么频繁地被使用。

**注意：** 关于这些内建 symbol 如何被用于元编程的详细信息，参见第七章的“通用 Symbol”。

## 复习

ES6 给 JavaScript 增加了一堆新的语法形式，有好多东西要学！

这些东西中的大多数都是为了缓解常见编程惯用法中的痛点而设计的，比如为函数参数设置默认值和将“剩余”的参数收集到一个数组中。解构是一个强大的工具，用来更简约地表达从数组或嵌套对象的赋值。

虽然像箭头函数`=>`这样的特性看起来也都是关于更简短更好看的语法，但是它们实际上拥有非常特殊的行为，你应当在恰当的情况下有意地使用它们。

扩展的 Unicode 支持，新的正则表达式技巧，和新的`symbol`基本类型充实了 ES6 语法的发展演变。