# 第六章：对象

对象是 JavaScript 中最基本的数据类型，您在本章之前的章节中已经多次看到它们。因为对象对于 JavaScript 语言非常重要，所以您需要详细了解它们的工作原理，而本章提供了这些细节。它从对象的正式概述开始，然后深入到关于创建对象和查询、设置、删除、测试和枚举对象属性的实用部分。这些以属性为重点的部分之后是关于如何扩展、序列化和定义对象重要方法的部分。最后，本章以关于 ES6 和更高版本语言中新对象字面量语法的长篇部分结束。

# 6.1 对象简介

对象是一个复合值：它聚合了多个值（原始值或其他对象），并允许您通过名称存储和检索这些值。对象是一个无序的*属性*集合，每个属性都有一个名称和一个值。属性名称通常是字符串（尽管，正如我们将在§6.10.3 中看到的，属性名称也可以是符号），因此我们可以说对象将字符串映射到值。这种字符串到值的映射有各种名称——您可能已经熟悉了以“哈希”、“哈希表”、“字典”或“关联数组”命名的基本数据结构。然而，对象不仅仅是一个简单的字符串到值的映射。除了维护自己的一组属性外，JavaScript 对象还继承另一个对象的属性，称为其“原型”。对象的方法通常是继承的属性，这种“原型继承”是 JavaScript 的一个关键特性。

JavaScript 对象是动态的——属性通常可以添加和删除——但它们可以用来模拟静态类型语言的静态对象和“结构”。它们也可以被用来（通过忽略字符串到值映射的值部分）表示字符串集合。

任何在 JavaScript 中不是字符串、数字、符号、`true`、`false`、`null` 或 `undefined` 的值都是对象。即使字符串、数字和布尔值不是对象，它们也可以像不可变对象一样行事。

从§3.8 中回想起，对象是*可变*的，通过引用而不是值来操作。如果变量 `x` 引用一个对象，并且执行代码 `let y = x;`，那么变量 `y` 持有对同一对象的引用，而不是该对象的副本。通过变量 `y` 对对象进行的任何修改也会通过变量 `x` 可见。

对象最常见的操作是创建它们并设置、查询、删除、测试和枚举它们的属性。这些基本操作在本章的开头部分进行了描述。之后的部分涵盖了更高级的主题。

*属性*具有名称和值。属性名称可以是任何字符串，包括空字符串（或任何符号），但没有对象可以具有两个具有相同名称的属性。该值可以是任何 JavaScript 值，或者它可以是一个 getter 或 setter 函数（或两者）。我们将在§6.10.6 中学习有关 getter 和 setter 函数的内容。

有时重要的是能够区分直接在对象上定义的属性和从原型对象继承的属性。JavaScript 使用术语*自有属性*来指代非继承的属性。

除了名称和值之外，每个属性还有三个*属性属性*：

+   *writable* 属性指定属性的值是否可以被设置。

+   *enumerable* 属性指定属性名称是否由 `for/in` 循环返回。

+   *configurable* 属性指定属性是否可以被删除以及其属性是否可以被更改。

JavaScript 的许多内置对象具有只读、不可枚举或不可配置的属性。但是，默认情况下，您创建的对象的所有属性都是可写的、可枚举的和可配置的。§14.1 解释了指定对象的非默认属性属性值的技术。

# 6.2 创建对象

使用对象字面量、`new`关键字和`Object.create()`函数可以创建对象。下面的小节描述了每种技术。

## 6.2.1 对象字面量

创建对象的最简单方法是在 JavaScript 代码中包含一个对象字面量。在其最简单的形式中，*对象字面量*是一个逗号分隔的冒号分隔的名称:值对列表，包含在花括号中。属性名是 JavaScript 标识符或字符串字面量（允许空字符串）。属性值是任何 JavaScript 表达式；表达式的值（可以是原始值或对象值）成为属性的值。以下是一些示例：

```js
let empty = {};                          // An object with no properties
let point = { x: 0, y: 0 };              // Two numeric properties
let p2 = { x: point.x, y: point.y+1 };   // More complex values
let book = {
    "main title": "JavaScript",          // These property names include spaces,
    "sub-title": "The Definitive Guide", // and hyphens, so use string literals.
    for: "all audiences",                // for is reserved, but no quotes.
    author: {                            // The value of this property is
        firstname: "David",              // itself an object.
        surname: "Flanagan"
    }
};
```

在对象字面量中最后一个属性后面加上逗号是合法的，一些编程风格鼓励使用这些尾随逗号，这样如果以后在对象字面量的末尾添加新属性，就不太可能导致语法错误。

对象字面量是一个表达式，每次评估时都会创建和初始化一个新的独立对象。每个属性的值在每次评估字面量时都会被评估。这意味着如果对象字面量出现在循环体内或重复调用的函数中，一个对象字面量可以创建许多新对象，并且这些对象的属性值可能彼此不同。

这里显示的对象字面量使用自 JavaScript 最早版本以来就合法的简单语法。语言的最新版本引入了许多新的对象字面量特性，这些特性在§6.10 中有介绍。

## 6.2.2 使用 new 创建对象

`new`运算符创建并初始化一个新对象。`new`关键字必须跟随一个函数调用。以这种方式使用的函数称为*构造函数*，用于初始化新创建的对象。JavaScript 包括其内置类型的构造函数。例如：

```js
let o = new Object();  // Create an empty object: same as {}.
let a = new Array();   // Create an empty array: same as [].
let d = new Date();    // Create a Date object representing the current time
let r = new Map();     // Create a Map object for key/value mapping
```

除了这些内置构造函数，通常会定义自己的构造函数来初始化新创建的对象。这在第九章中有介绍。

## 6.2.3 原型

在我们讨论第三种对象创建技术之前，我们必须停顿一下来解释原型。几乎每个 JavaScript 对象都有一个与之关联的第二个 JavaScript 对象。这第二个对象称为*原型*，第一个对象从原型继承属性。

所有通过对象字面量创建的对象都有相同的原型对象，在 JavaScript 代码中我们可以将这个原型对象称为`Object.prototype`。使用`new`关键字和构造函数调用创建的对象使用构造函数的`prototype`属性的值作为它们的原型。因此，通过`new Object()`创建的对象继承自`Object.prototype`，就像通过`{}`创建的对象一样。类似地，通过`new Array()`创建的对象使用`Array.prototype`作为它们的原型，通过`new Date()`创建的对象使用`Date.prototype`作为它们的原型。初学 JavaScript 时可能会感到困惑。记住：几乎所有对象都有一个*原型*，但只有相对较少的对象有一个`prototype`属性。具有`prototype`属性的这些对象为所有其他对象定义了*原型*。

`Object.prototype`是少数没有原型的对象之一：它不继承任何属性。其他原型对象是具有原型的普通对象。大多数内置构造函数（以及大多数用户定义的构造函数）具有从`Object.prototype`继承的原型。例如，`Date.prototype`从`Object.prototype`继承属性，因此通过`new Date()`创建的 Date 对象从`Date.prototype`和`Object.prototype`继承属性。这个链接的原型对象系列被称为*原型链*。

如何工作属性继承的解释在§6.3.2 中。第九章更详细地解释了原型和构造函数之间的关系：它展示了如何通过编写构造函数并将其`prototype`属性设置为由该构造函数创建的“实例”使用的原型对象来定义新的对象“类”。我们将学习如何在§14.3 中查询（甚至更改）对象的原型。

## 6.2.4 Object.create()

`Object.create()`创建一个新对象，使用其第一个参数作为该对象的原型：

```js
let o1 = Object.create({x: 1, y: 2});     // o1 inherits properties x and y.
o1.x + o1.y                               // => 3
```

您可以传递`null`来创建一个没有原型的新对象，但如果这样做，新创建的对象将不会继承任何东西，甚至不会继承像`toString()`这样的基本方法（这意味着它也无法与`+`运算符一起使用）：

```js
let o2 = Object.create(null);             // o2 inherits no props or methods.
```

如果要创建一个普通的空对象（类似于`{}`或`new Object()`返回的对象），请传递`Object.prototype`：

```js
let o3 = Object.create(Object.prototype); // o3 is like {} or new Object().
```

使用具有任意原型的新对象的能力是强大的，我们将在本章的许多地方使用`Object.create()`。（`Object.create()`还接受一个可选的第二个参数，描述新对象的属性。这个第二个参数是一个高级功能，涵盖在§14.1 中。）

使用`Object.create()`的一个用途是当您想要防止通过您无法控制的库函数意外（但非恶意）修改对象时。您可以传递一个从中继承的对象而不是直接将对象传递给函数。如果函数读取该对象的属性，它将看到继承的值。但是，如果它设置属性，这些写入将不会影响原始对象。

```js
let o = { x: "don't change this value" };
library.function(Object.create(o));  // Guard against accidental modifications
```

要理解为什么这样做有效，您需要了解在 JavaScript 中如何查询和设置属性。这些是下一节的主题。

# 6.3 查询和设置属性

要获取属性的值，请使用§4.4 中描述的点号（`.`）或方括号（`[]`）运算符。左侧应该是一个值为对象的表达式。如果使用点运算符，则右侧必须是一个简单的标识符，用于命名属性。如果使用方括号，则括号内的值必须是一个求值为包含所需属性名称的字符串的表达式：

```js
let author = book.author;       // Get the "author" property of the book.
let name = author.surname;      // Get the "surname" property of the author.
let title = book["main title"]; // Get the "main title" property of the book.
```

要创建或设置属性，请像查询属性一样使用点号或方括号，但将它们放在赋值表达式的左侧：

```js
book.edition = 7;                   // Create an "edition" property of book.
book["main title"] = "ECMAScript";  // Change the "main title" property.
```

在使用方括号表示法时，我们已经说过方括号内的表达式必须求值为字符串。更精确的说法是，表达式必须求值为字符串或可以转换为字符串或符号的值（§6.10.3）。例如，在第七章中，我们将看到在方括号内使用数字是常见的。

## 6.3.1 对象作为关联数组

如前一节所述，以下两个 JavaScript 表达式具有相同的值：

```js
object.property
object["property"]
```

第一种语法，使用点和标识符，类似于在 C 或 Java 中访问结构体或对象的静态字段的语法。第二种语法，使用方括号和字符串，看起来像数组访问，但是是通过字符串而不是数字索引的数组。这种类型的数组被称为*关联数组*（或哈希或映射或字典）。JavaScript 对象就是关联数组，本节解释了为什么这很重要。

在 C、C++、Java 等强类型语言中，一个对象只能拥有固定数量的属性，并且这些属性的名称必须事先定义。由于 JavaScript 是一种弱类型语言，这个规则不适用：程序可以在任何对象中创建任意数量的属性。然而，当你使用`.`运算符访问对象的属性时，属性的名称必须表示为标识符。标识符必须直接输入到你的 JavaScript 程序中；它们不是一种数据类型，因此不能被程序操作。

另一方面，当你使用`[]`数组表示法访问对象的属性时，属性的名称表示为字符串。字符串是 JavaScript 数据类型，因此它们可以在程序运行时被操作和创建。因此，例如，你可以在 JavaScript 中编写以下代码：

```js
let addr = "";
for(let i = 0; i < 4; i++) {
    addr += customer[`address${i}`] + "\n";
}
```

这段代码读取并连接`customer`对象的`address0`、`address1`、`address2`和`address3`属性。

这个简短的示例展示了使用数组表示法访问对象属性时的灵活性。这段代码可以使用点表示法重写，但有些情况下只有数组表示法才能胜任。例如，假设你正在编写一个程序，该程序使用网络资源计算用户股票市场投资的当前价值。该程序允许用户输入他们拥有的每支股票的名称以及每支股票的股数。你可以使用一个名为`portfolio`的对象来保存这些信息。对象的每个属性都代表一支股票。属性的名称是股票的名称，属性值是该股票的股数。因此，例如，如果用户持有 IBM 的 50 股，`portfolio.ibm`属性的值为`50`。

这个程序的一部分可能是一个用于向投资组合添加新股票的函数：

```js
function addstock(portfolio, stockname, shares) {
    portfolio[stockname] = shares;
}
```

由于用户在运行时输入股票名称，所以你无法提前知道属性名称。因为在编写程序时你无法知道属性名称，所以无法使用`.`运算符访问`portfolio`对象的属性。然而，你可以使用`[]`运算符，因为它使用字符串值（动态的，可以在运行时更改）而不是标识符（静态的，必须在程序中硬编码）来命名属性。

在第五章中，我们介绍了`for/in`循环（我们很快会再次看到它，在§6.6 中）。当你考虑它与关联数组一起使用时，这个 JavaScript 语句的强大之处就显而易见了。下面是计算投资组合总价值时如何使用它的示例：

```js
function computeValue(portfolio) {
    let total = 0.0;
    for(let stock in portfolio) {       // For each stock in the portfolio:
        let shares = portfolio[stock];  // get the number of shares
        let price = getQuote(stock);    // look up share price
        total += shares * price;        // add stock value to total value
    }
    return total;                       // Return total value.
}
```

JavaScript 对象通常被用作关联数组，如下所示，了解这是如何工作的很重要。然而，在 ES6 及以后的版本中，描述在§11.1.2 中的 Map 类通常比使用普通对象更好。

## 6.3.2 继承

JavaScript 对象有一组“自有属性”，它们还从它们的原型对象继承了一组属性。要理解这一点，我们必须更详细地考虑属性访问。本节中的示例使用`Object.create()`函数创建具有指定原型的对象。然而，我们将在第九章中看到，每次使用`new`创建类的实例时，都会创建一个从原型对象继承属性的对象。

假设您查询对象`o`中的属性`x`。如果`o`没有具有该名称的自有属性，则将查询`o`的原型对象¹的属性`x`。如果原型对象没有具有该名称的自有属性，但具有自己的原型，则将在原型的原型上执行查询。这将继续，直到找到属性`x`或直到搜索具有`null`原型的对象。正如您所看到的，对象的`prototype`属性创建了一个链或链接列表，从中继承属性：

```js
let o = {};               // o inherits object methods from Object.prototype
o.x = 1;                  // and it now has an own property x.
let p = Object.create(o); // p inherits properties from o and Object.prototype
p.y = 2;                  // and has an own property y.
let q = Object.create(p); // q inherits properties from p, o, and...
q.z = 3;                  // ...Object.prototype and has an own property z.
let f = q.toString();     // toString is inherited from Object.prototype
q.x + q.y                 // => 3; x and y are inherited from o and p
```

现在假设您对对象`o`的属性`x`进行赋值。如果`o`已经具有自己的（非继承的）名为`x`的属性，则赋值将简单地更改此现有属性的值。否则，赋值将在对象`o`上创建一个名为`x`的新属性。如果`o`先前继承了属性`x`，那么新创建的同名自有属性将隐藏该继承的属性。

属性赋值仅检查原型链以确定是否允许赋值。例如，如果`o`继承了一个名为`x`的只读属性，则不允许赋值。（有关何时可以设置属性的详细信息，请参见§6.3.3。）然而，如果允许赋值，它总是在原始对象中创建或设置属性，而不会修改原型链中的对象。查询属性时发生继承，但在设置属性时不会发生继承是 JavaScript 的一个关键特性，因为它允许我们有选择地覆盖继承的属性：

```js
let unitcircle = { r: 1 };         // An object to inherit from
let c = Object.create(unitcircle); // c inherits the property r
c.x = 1; c.y = 1;                  // c defines two properties of its own
c.r = 2;                           // c overrides its inherited property
unitcircle.r                       // => 1: the prototype is not affected
```

有一个例外情况，即属性赋值要么失败，要么在原始对象中创建或设置属性。如果`o`继承了属性`x`，并且该属性是一个具有 setter 方法的访问器属性（参见§6.10.6），那么将调用该 setter 方法，而不是在`o`中创建新属性`x`。然而，请注意，setter 方法是在对象`o`上调用的，而不是在定义属性的原型对象上调用的，因此如果 setter 方法定义了任何属性，它将在`o`上进行，而且它将再次不修改原型链。

## 6.3.3 属性访问错误

属性访问表达式并不总是返回或设置一个值。本节解释了在查询或设置属性时可能出现的问题。

查询不存在的属性并不是错误的。如果在`o`的自有属性或继承属性中找不到属性`x`，则属性访问表达式`o.x`将求值为`undefined`。请记住，我们的书对象具有“子标题”属性，但没有“subtitle”属性：

```js
book.subtitle    // => undefined: property doesn't exist
```

然而，尝试查询不存在的对象的属性是错误的。`null`和`undefined`值没有属性，查询这些值的属性是错误的。继续前面的例子：

```js
let len = book.subtitle.length; // !TypeError: undefined doesn't have length
```

如果`.`的左侧是`null`或`undefined`，则属性访问表达式将失败。因此，在编写诸如`book.author.surname`的表达式时，如果不确定`book`和`book.author`是否已定义，应谨慎。以下是防止此类问题的两种方法：

```js
// A verbose and explicit technique
let surname = undefined;
if (book) {
    if (book.author) {
        surname = book.author.surname;
    }
}

// A concise and idiomatic alternative to get surname or null or undefined
surname = book && book.author && book.author.surname;
```

要理解为什么这种成语表达式可以防止 TypeError 异常，您可能需要回顾一下`&&`运算符的短路行为，详情请参见§4.10.1。

如§4.4.1 中所述，ES2020 支持使用`?.`进行条件属性访问，这使我们可以将先前的赋值表达式重写为：

```js
let surname = book?.author?.surname;
```

尝试在 `null` 或 `undefined` 上设置属性也会导致 TypeError。在其他值上尝试设置属性也不总是成功：某些属性是只读的，无法设置，某些对象不允许添加新属性。在严格模式下（§5.6.3），每当尝试设置属性失败时都会抛出 TypeError。在非严格模式下，这些失败通常是静默的。

指定属性赋值何时成功何时失败的规则是直观的，但难以简洁表达。在以下情况下，尝试设置对象 `o` 的属性 `p` 失败：

+   `o` 有一个自己的只读属性 `p`：无法设置只读属性。

+   `o` 具有一个继承的只读属性 `p`：无法通过具有相同名称的自有属性隐藏继承的只读属性。

+   `o` 没有自己的属性 `p`；`o` 没有继承具有 setter 方法的属性 `p`，且 `o` 的 *可扩展* 属性（见 §14.2）为 `false`。由于 `o` 中 `p` 不存在，并且没有 setter 方法可调用，因此必须将 `p` 添加到 `o` 中。但如果 `o` 不可扩展，则无法在其上定义新属性。

# 6.4 删除属性

`delete` 运算符（§4.13.4）从对象中删除属性。其单个操作数应为属性访问表达式。令人惊讶的是，`delete` 不是作用于属性的值，而是作用于属性本身：

```js
delete book.author;          // The book object now has no author property.
delete book["main title"];   // Now it doesn't have "main title", either.
```

`delete` 运算符仅删除自有属性，而不删除继承的属性。（要删除继承的属性，必须从定义该属性的原型对象中删除它。这会影响从该原型继承的每个对象。）

`delete` 表达式在删除成功删除或删除无效（例如删除不存在的属性）时求值为 `true`。当与非属性访问表达式一起使用时，`delete` 也会求值为 `true`（毫无意义地）：

```js
let o = {x: 1};    // o has own property x and inherits property toString
delete o.x         // => true: deletes property x
delete o.x         // => true: does nothing (x doesn't exist) but true anyway
delete o.toString  // => true: does nothing (toString isn't an own property)
delete 1           // => true: nonsense, but true anyway
```

`delete` 不会删除具有 *可配置* 属性为 `false` 的属性。某些内置对象的属性是不可配置的，变量声明和函数声明创建的全局对象的属性也是如此。在严格模式下，尝试删除不可配置属性会导致 TypeError。在非严格模式下，此情况下 `delete` 简单地求值为 `false`：

```js
// In strict mode, all these deletions throw TypeError instead of returning false
delete Object.prototype // => false: property is non-configurable
var x = 1;              // Declare a global variable
delete globalThis.x     // => false: can't delete this property
function f() {}         // Declare a global function
delete globalThis.f     // => false: can't delete this property either
```

在非严格模式下删除全局对象的可配置属性时，可以省略对全局对象的引用，只需跟随 `delete` 运算符后面的属性名：

```js
globalThis.x = 1;       // Create a configurable global property (no let or var)
delete x                // => true: this property can be deleted
```

然而，在严格模式下，如果其操作数是像 `x` 这样的未限定标识符，`delete` 会引发 SyntaxError，并且您必须明确指定属性访问：

```js
delete x;               // SyntaxError in strict mode
delete globalThis.x;    // This works
```

# 6.5 测试属性

JavaScript 对象可以被视为属性集合，通常有必要能够测试是否属于该集合——检查对象是否具有给定名称的属性。您可以使用 `in` 运算符、`hasOwnProperty()` 和 `propertyIsEnumerable()` 方法，或者简单地查询属性来实现此目的。这里显示的示例都使用字符串作为属性名称，但它们也适用于符号（§6.10.3）。

`in` 运算符在其左侧期望一个属性名，在其右侧期望一个对象。如果对象具有该名称的自有属性或继承属性，则返回 `true`：

```js
let o = { x: 1 };
"x" in o         // => true: o has an own property "x"
"y" in o         // => false: o doesn't have a property "y"
"toString" in o  // => true: o inherits a toString property
```

对象的 `hasOwnProperty()` 方法测试该对象是否具有给定名称的自有属性。对于继承属性，它返回 `false`：

```js
let o = { x: 1 };
o.hasOwnProperty("x")        // => true: o has an own property x
o.hasOwnProperty("y")        // => false: o doesn't have a property y
o.hasOwnProperty("toString") // => false: toString is an inherited property
```

`propertyIsEnumerable()` 优化了 `hasOwnProperty()` 测试。只有在命名属性是自有属性且其*可枚举*属性为 `true` 时才返回 `true`。某些内置属性是不可枚举的。通过正常的 JavaScript 代码创建的属性是可枚举的，除非你使用了 §14.1 中展示的技术之一使它们变为不可枚举。

```js
let o = { x: 1 };
o.propertyIsEnumerable("x")  // => true: o has an own enumerable property x
o.propertyIsEnumerable("toString")  // => false: not an own property
Object.prototype.propertyIsEnumerable("toString") // => false: not enumerable
```

不必使用 `in` 运算符，通常只需查询属性并使用 `!==` 来确保它不是未定义的：

```js
let o = { x: 1 };
o.x !== undefined        // => true: o has a property x
o.y !== undefined        // => false: o doesn't have a property y
o.toString !== undefined // => true: o inherits a toString property
```

`in` 运算符可以做到这里展示的简单属性访问技术无法做到的一件事。`in` 可以区分不存在的属性和已设置为 `undefined` 的属性。考虑以下代码：

```js
let o = { x: undefined };  // Property is explicitly set to undefined
o.x !== undefined          // => false: property exists but is undefined
o.y !== undefined          // => false: property doesn't even exist
"x" in o                   // => true: the property exists
"y" in o                   // => false: the property doesn't exist
delete o.x;                // Delete the property x
"x" in o                   // => false: it doesn't exist anymore
```

# 6.6 枚举属性

有时我们不想测试单个属性的存在，而是想遍历或获取对象的所有属性列表。有几种不同的方法可以做到这一点。

`for/in` 循环在 §5.4.5 中有介绍。它会为指定对象的每个可枚举属性（自有或继承的）执行一次循环体，将属性的名称赋给循环变量。对象继承的内置方法是不可枚举的，但你的代码添加到对象的属性默认是可枚举的。例如：

```js
let o = {x: 1, y: 2, z: 3};          // Three enumerable own properties
o.propertyIsEnumerable("toString")   // => false: not enumerable
for(let p in o) {                    // Loop through the properties
    console.log(p);                  // Prints x, y, and z, but not toString
}
```

为了防止使用 `for/in` 枚举继承属性，你可以在循环体内添加一个显式检查：

```js
for(let p in o) {
    if (!o.hasOwnProperty(p)) continue;       // Skip inherited properties
}

for(let p in o) {
    if (typeof o[p] === "function") continue; // Skip all methods
}
```

作为使用 `for/in` 循环的替代方案，通常更容易获得对象的属性名称数组，然后使用 `for/of` 循环遍历该数组。有四个函数可以用来获取属性名称数组：

+   `Object.keys()` 返回一个对象的可枚举自有属性名称的数组。它不包括不可枚举属性、继承属性或名称为 Symbol 的属性（参见 §6.10.3）。

+   `Object.getOwnPropertyNames()` 的工作方式类似于 `Object.keys()`，但会返回一个非枚举自有属性名称的数组，只要它们的名称是字符串。

+   `Object.getOwnPropertySymbols()` 返回那些名称为 Symbol 的自有属性，无论它们是否可枚举。

+   `Reflect.ownKeys()` 返回所有自有属性名称，包括可枚举和不可枚举的，以及字符串和 Symbol。 (参见 §14.6.)

在 §6.7 中有关于使用 `Object.keys()` 与 `for/of` 循环的示例。

## 6.6.1 属性枚举顺序

ES6 正式定义了对象自有属性枚举的顺序。`Object.keys()`、`Object.getOwnPropertyNames()`、`Object.getOwnPropertySymbols()`、`Reflect.ownKeys()` 和相关方法如 `JSON.stringify()` 都按照以下顺序列出属性，受其自身关于是否列出非枚举属性或属性名称为字符串或 Symbol 的额外约束：

+   名称为非负整数的字符串属性首先按数字顺序从小到大列出。这个规则意味着数组和类数组对象的属性将按顺序枚举。

+   列出所有看起来像数组索引的属性后，所有剩余的具有字符串名称的属性也会被列出（包括看起来像负数或浮点数的属性）。这些属性按照它们添加到对象的顺序列出。对于对象字面量中定义的属性，这个顺序与它们在字面量中出现的顺序相同。

+   最后，那些名称为 Symbol 对象的属性按照它们添加到对象的顺序列出。

`for/in` 循环的枚举顺序并没有像这些枚举函数那样严格规定，但通常的实现会按照刚才描述的顺序枚举自有属性，然后沿着原型链向上遍历，对每个原型对象按照相同的顺序枚举属性。然而，请注意，如果同名属性已经被枚举过，或者即使同名的不可枚举属性已经被考虑过，该属性将不会被枚举。

# 6.7 扩展对象

JavaScript 程序中的一个常见操作是需要将一个对象的属性复制到另一个对象中。可以使用以下代码轻松实现这一操作：

```js
let target = {x: 1}, source = {y: 2, z: 3};
for(let key of Object.keys(source)) {
    target[key] = source[key];
}
target  // => {x: 1, y: 2, z: 3}
```

但由于这是一个常见的操作，各种 JavaScript 框架已经定义了实用函数，通常命名为 `extend()`，来执行这种复制操作。最后，在 ES6 中，这种能力以 `Object.assign()` 的形式进入了核心 JavaScript 语言。

`Object.assign()` 期望两个或更多对象作为其参数。它修改并返回第一个参数，即目标对象，但不会改变第二个或任何后续参数，即源对象。对于每个源对象，它将该对象的可枚举自有属性（包括那些名称为 Symbols 的属性）复制到目标对象中。它按照参数列表顺序处理源对象，因此第一个源对象中的属性将覆盖目标对象中同名的属性，第二个源对象中的属性（如果有的话）将覆盖第一个源对象中同名的属性。

`Object.assign()` 使用普通的属性获取和设置操作来复制属性，因此如果源对象具有 getter 方法或目标对象具有 setter 方法，则它们将在复制过程中被调用，但它们本身不会被复制。

将一个对象的属性分配到另一个对象中的一个原因是，当你有一个对象定义了许多属性的默认值，并且希望将这些默认属性复制到另一个对象中，如果该对象中不存在同名属性。简单地使用 `Object.assign()` 不会达到你想要的效果：

```js
Object.assign(o, defaults);  // overwrites everything in o with defaults
```

相反，您可以创建一个新对象，将默认值复制到其中，然后用 `o` 中的属性覆盖这些默认值：

```js
o = Object.assign({}, defaults, o);
```

我们将在 §6.10.4 中看到，您还可以使用 `...` 展开运算符来表达这种对象复制和覆盖操作，就像这样：

```js
o = {...defaults, ...o};
```

我们也可以通过编写一个只在属性缺失时才复制属性的版本的 `Object.assign()` 来避免额外的对象创建和复制开销：

```js
// Like Object.assign() but doesn't override existing properties
// (and also doesn't handle Symbol properties)
function merge(target, ...sources) {
    for(let source of sources) {
        for(let key of Object.keys(source)) {
            if (!(key in target)) { // This is different than Object.assign()
                target[key] = source[key];
            }
        }
    }
    return target;
}
Object.assign({x: 1}, {x: 2, y: 2}, {y: 3, z: 4})  // => {x: 2, y: 3, z: 4}
merge({x: 1}, {x: 2, y: 2}, {y: 3, z: 4})          // => {x: 1, y: 2, z: 4}
```

编写其他类似这个 `merge()` 函数的属性操作实用程序是很简单的。例如，`restrict()` 函数可以删除对象的属性，如果这些属性在另一个模板对象中不存在。或者 `subtract()` 函数可以从另一个对象中删除所有属性。

# 6.8 序列化对象

对象*序列化*是将对象状态转换为一个字符串的过程，以便以后可以恢复该对象。函数 `JSON.stringify()` 和 `JSON.parse()` 可以序列化和恢复 JavaScript 对象。这些函数使用 JSON 数据交换格式。JSON 代表“JavaScript 对象表示法”，其语法与 JavaScript 对象和数组文字非常相似：

```js
let o = {x: 1, y: {z: [false, null, ""]}}; // Define a test object
let s = JSON.stringify(o);   // s == '{"x":1,"y":{"z":[false,null,""]}}'
let p = JSON.parse(s);       // p == {x: 1, y: {z: [false, null, ""]}}
```

JSON 语法是 JavaScript 语法的*子集*，它不能表示所有 JavaScript 值。支持并可以序列化和还原的有对象、数组、字符串、有限数字、`true`、`false`和`null`。`NaN`、`Infinity`和`-Infinity`被序列化为`null`。Date 对象被序列化为 ISO 格式的日期字符串（参见`Date.toJSON()`函数），但`JSON.parse()`将它们保留为字符串形式，不会还原原始的 Date 对象。Function、RegExp 和 Error 对象以及`undefined`值不能被序列化或还原。`JSON.stringify()`只序列化对象的可枚举自有属性。如果属性值无法序列化，则该属性将简单地从字符串化输出中省略。`JSON.stringify()`和`JSON.parse()`都接受可选的第二个参数，用于通过指定要序列化的属性列表来自定义序列化和/或还原过程，例如，在序列化或字符串化过程中转换某些值。这些函数的完整文档在§11.6 中。

# 6.9 对象方法

正如前面讨论的，所有 JavaScript 对象（除了明确创建时没有原型的对象）都从`Object.prototype`继承属性。这些继承的属性主要是方法，因为它们是普遍可用的，所以它们对 JavaScript 程序员特别感兴趣。例如，我们已经看到了`hasOwnProperty()`和`propertyIsEnumerable()`方法。（我们也已经涵盖了`Object`构造函数上定义的许多静态函数，比如`Object.create()`和`Object.keys()`。）本节解释了一些定义在`Object.prototype`上的通用对象方法，但是这些方法旨在被其他更专门的实现所取代。在接下来的章节中，我们将展示在单个对象上定义这些方法的示例。在第九章中，您将学习如何为整个对象类更普遍地定义这些方法。

## 6.9.1 toString() 方法

`toString()` 方法不接受任何参数；它返回一个表示调用它的对象的值的字符串。JavaScript 在需要将对象转换为字符串时会调用这个方法。例如，当你使用`+`运算符将字符串与对象连接在一起，或者当你将对象传递给期望字符串的方法时，就会发生这种情况。

默认的`toString()`方法并不是很有信息量（尽管它对于确定对象的类很有用，正如我们将在§14.4.3 中看到的）。例如，以下代码行简单地评估为字符串“[object Object]”：

```js
let s = { x: 1, y: 1 }.toString();  // s == "[object Object]"
```

因为这个默认方法并不显示太多有用信息，许多类定义了它们自己的`toString()`版本。例如，当数组转换为字符串时，你会得到一个数组元素列表，它们各自被转换为字符串，当函数转换为字符串时，你会得到函数的源代码。你可以像这样定义自己的`toString()`方法：

```js
let point = {
    x: 1,
    y: 2,
    toString: function() { return `(${this.x}, ${this.y})`; }
};
String(point)    // => "(1, 2)": toString() is used for string conversions
```

## 6.9.2 toLocaleString() 方法

除了基本的`toString()`方法外，所有对象都有一个`toLocaleString()`方法。这个方法的目的是返回对象的本地化字符串表示。Object 定义的默认`toLocaleString()`方法不进行任何本地化：它只是调用`toString()`并返回该值。Date 和 Number 类定义了定制版本的`toLocaleString()`，试图根据本地惯例格式化数字、日期和时间。Array 定义了一个`toLocaleString()`方法，工作方式类似于`toString()`，只是通过调用它们的`toLocaleString()`方法而不是`toString()`方法来格式化数组元素。你可以像这样处理`point`对象：

```js
let point = {
    x: 1000,
    y: 2000,
    toString: function() { return `(${this.x}, ${this.y})`; },
    toLocaleString: function() {
        return `(${this.x.toLocaleString()}, ${this.y.toLocaleString()})`;
    }
};
point.toString()        // => "(1000, 2000)"
point.toLocaleString()  // => "(1,000, 2,000)": note thousands separators
```

在实现 `toLocaleString()` 方法时，§11.7 中记录的国际化类可能会很有用。

## 6.9.3 valueOf() 方法

`valueOf()` 方法类似于 `toString()` 方法，但当 JavaScript 需要将对象转换为除字符串以外的某种原始类型时（通常是数字），就会调用它。如果对象在需要原始值的上下文中使用，JavaScript 会自动调用这个方法。默认的 `valueOf()` 方法没有什么有趣的功能，但一些内置类定义了自己的 `valueOf()` 方法。Date 类定义了 `valueOf()` 方法来将日期转换为数字，这允许使用 `<` 和 `>` 来对日期对象进行比较。你可以通过定义一个 `valueOf()` 方法来实现类似的功能，返回从原点到点的距离：

```js
let point = {
    x: 3,
    y: 4,
    valueOf: function() { return Math.hypot(this.x, this.y); }
};
Number(point)  // => 5: valueOf() is used for conversions to numbers
point > 4      // => true
point > 5      // => false
point < 6      // => true
```

## 6.9.4 toJSON() 方法

`Object.prototype` 实际上并没有定义 `toJSON()` 方法，但 `JSON.stringify()` 方法（参见 §6.8）会在要序列化的任何对象上查找 `toJSON()` 方法。如果这个方法存在于要序列化的对象上，它就会被调用，返回值会被序列化，而不是原始对象。Date 类（§11.4）定义了一个 `toJSON()` 方法，返回日期的可序列化字符串表示。我们可以为我们的 Point 对象做同样的事情：

```js
let point = {
    x: 1,
    y: 2,
    toString: function() { return `(${this.x}, ${this.y})`; },
    toJSON: function() { return this.toString(); }
};
JSON.stringify([point])   // => '["(1, 2)"]'
```

# 6.10 扩展对象字面量语法

JavaScript 的最新版本在对象字面量的语法上以多种有用的方式进行了扩展。以下小节解释了这些扩展。

## 6.10.1 简写属性

假设你有存储在变量 `x` 和 `y` 中的值，并且想要创建一个具有名为 `x` 和 `y` 的属性的对象，其中包含这些值。使用基本对象字面量语法，你将重复每个标识符两次：

```js
let x = 1, y = 2;
let o = {
    x: x,
    y: y
};
```

在 ES6 及更高版本中，你可以省略冒号和一个标识符的副本，从而得到更简洁的代码：

```js
let x = 1, y = 2;
let o = { x, y };
o.x + o.y  // => 3
```

## 6.10.2 计算属性名

有时候你需要创建一个具有特定属性的对象，但该属性的名称不是你可以在源代码中直接输入的编译时常量。相反，你需要的属性名称存储在一个变量中，或者是一个你调用的函数的返回值。你不能使用基本对象字面量来定义这种属性。相反，你必须先创建一个对象，然后作为额外步骤添加所需的属性：

```js
const PROPERTY_NAME = "p1";
function computePropertyName() { return "p" + 2; }

let o = {};
o[PROPERTY_NAME] = 1;
o[computePropertyName()] = 2;
```

使用 ES6 功能中称为*计算属性*的功能，可以更简单地设置一个对象，直接将前面代码中的方括号移到对象字面量中：

```js
const PROPERTY_NAME = "p1";
function computePropertyName() { return "p" + 2; }

let p = {
    [PROPERTY_NAME]: 1,
    [computePropertyName()]: 2
};

p.p1 + p.p2 // => 3
```

使用这种新的语法，方括号限定了任意的 JavaScript 表达式。该表达式被评估，结果值（如有必要，转换为字符串）被用作属性名。

一个情况下你可能想使用计算属性的地方是当你有一个 JavaScript 代码库，该库期望传递具有特定属性集的对象，并且这些属性的名称在该库中被定义为常量。如果你正在编写代码来创建将传递给该库的对象，你可以硬编码属性名称，但如果在任何地方输入属性名称错误，就会出现错误，如果库的新版本更改了所需的属性名称，就会出现版本不匹配的问题。相反，你可能会发现使用由库定义的属性名常量与计算属性语法使你的代码更加健壮。

## 6.10.3 符号作为属性名

计算属性语法还启用了另一个非常重要的对象字面量特性。在 ES6 及更高版本中，属性名称可以是字符串或符号。如果将符号分配给变量或常量，那么可以使用计算属性语法将该符号作为属性名：

```js
const extension = Symbol("my extension symbol");
let o = {
    [extension]: { /* extension data stored in this object */ }
};
o[extension].x = 0; // This won't conflict with other properties of o
```

如§3.6 中所解释的，符号是不透明的值。你不能对它们做任何操作，只能将它们用作属性名称。然而，每个符号都与其他任何符号都不同，这意味着符号非常适合创建唯一的属性名称。通过调用`Symbol()`工厂函数创建一个新符号。（符号是原始值，不是对象，因此`Symbol()`不是一个你使用`new`调用的构造函数。）`Symbol()`返回的值不等于任何其他符号或其他值。你可以向`Symbol()`传递一个字符串，当你的符号转换为字符串时，将使用该字符串。但这仅用于调试：使用相同字符串参数创建的两个符号仍然彼此不同。

符号的作用不是安全性，而是为 JavaScript 对象定义一个安全的扩展机制。如果你从你无法控制的第三方代码中获取一个对象，并且需要向该对象添加一些你自己的属性，但又希望确保你的属性不会与对象上可能已经存在的任何属性发生冲突，那么你可以安全地使用符号作为你的属性名称。如果你这样做，你还可以确信第三方代码不会意外地更改你的以符号命名的属性。（当然，该第三方代码可以使用`Object.getOwnPropertySymbols()`来发现你正在使用的符号，并可能更改或删除你的属性。这就是为什么符号不是一种安全机制。）

## 6.10.4 展开运算符

在 ES2018 及更高版本中，你可以使用“展开运算符”`...`将现有对象的属性复制到一个新对象中，写在对象字面量内部：

```js
let position = { x: 0, y: 0 };
let dimensions = { width: 100, height: 75 };
let rect = { ...position, ...dimensions };
rect.x + rect.y + rect.width + rect.height // => 175
```

在这段代码中，`position`和`dimensions`对象的属性被“展开”到`rect`对象字面量中，就好像它们被直接写在那些花括号内一样。请注意，这种`...`语法通常被称为展开运算符，但在任何情况下都不是真正的 JavaScript 运算符。相反，它是仅在对象字面量内部可用的特殊语法。 （在其他 JavaScript 上下文中，三个点用于其他目的，但对象字面量是唯一的上下文，其中这三个点会导致一个对象插入到另一个对象中。）

如果被展开的对象和被展开到的对象都有同名属性，则该属性的值将是最后一个出现的值：

```js
let o = { x: 1 };
let p = { x: 0, ...o };
p.x   // => 1: the value from object o overrides the initial value
let q = { ...o, x: 2 };
q.x   // => 2: the value 2 overrides the previous value from o.
```

还要注意，展开运算符只展开对象的自有属性，而不包括任何继承的属性：

```js
let o = Object.create({x: 1}); // o inherits the property x
let p = { ...o };
p.x                            // => undefined
```

最后，值得注意的是，尽管展开运算符在你的代码中只是三个小点，但它可能代表 JavaScript 解释器大量的工作。如果一个对象有*n*个属性，将这些属性展开到另一个对象中的过程可能是一个*O(n)*的操作。这意味着如果你发现自己在循环或递归函数中使用`...`来将数据累积到一个大对象中，你可能正在编写一个效率低下的*O(n²)*算法，随着*n*的增大，它的性能将不会很好。

## 6.10.5 简写方法

当一个函数被定义为对象的属性时，我们称该函数为*方法*（我们将在第八章和第九章中详细讨论方法）。在 ES6 之前，你可以使用函数定义表达式在对象字面量中定义一个方法，就像你定义对象的任何其他属性一样：

```js
let square = {
    area: function() { return this.side * this.side; },
    side: 10
};
square.area() // => 100
```

然而，在 ES6 中，对象字面量语法（以及我们将在第九章中看到的类定义语法）已经扩展，允许一种快捷方式，其中省略了`function`关键字和冒号，导致代码如下：

```js
let square = {
    area() { return this.side * this.side; },
    side: 10
};
square.area() // => 100
```

两种形式的代码是等价的：都向对象字面量添加了一个名为`area`的属性，并将该属性的值设置为指定的函数。简写语法使得`area()`是一个方法，而不是像`side`那样的数据属性。

当使用这种简写语法编写方法时，属性名称可以采用对象字面量中合法的任何形式：除了像上面的`area`名称一样的常规 JavaScript 标识符外，还可以使用字符串文字和计算属性名称，其中可以包括 Symbol 属性名称：

```js
const METHOD_NAME = "m";
const symbol = Symbol();
let weirdMethods = {
    "method With Spaces"(x) { return x + 1; },
    METHOD_NAME { return x + 2; },
    symbol { return x + 3; }
};
weirdMethods"method With Spaces"  // => 2
weirdMethodsMETHOD_NAME           // => 3
weirdMethodssymbol                // => 4
```

使用符号作为方法名并不像看起来那么奇怪。为了使对象可迭代（以便与`for/of`循环一起使用），必须定义一个具有符号名称`Symbol.iterator`的方法，第十二章中有这样做的示例。

## 6.10.6 属性的 getter 和 setter

到目前为止，在本章中讨论的所有对象属性都是具有名称和普通值的*数据属性*。JavaScript 还支持*访问器属性*，它们没有单个值，而是具有一个或两个访问器方法：一个*getter*和/或一个*setter*。

当程序查询访问器属性的值时，JavaScript 会调用 getter 方法（不传递任何参数）。此方法的返回值成为属性访问表达式的值。当程序设置访问器属性的值时，JavaScript 会调用 setter 方法，传递赋值右侧的值。该方法负责在某种意义上“设置”属性值。setter 方法的返回值将被忽略。

如果一个属性同时具有 getter 和 setter 方法，则它是一个读/写属性。如果它只有 getter 方法，则它是一个只读属性。如果它只有 setter 方法，则它是一个只写属性（这是使用数据属性不可能实现的），并且尝试读取它的值总是评估为`undefined`。

访问器属性可以使用对象字面量语法的扩展来定义（与我们在这里看到的其他 ES6 扩展不同，getter 和 setter 是在 ES5 中引入的）：

```js
let o = {
    // An ordinary data property
    dataProp: value,

    // An accessor property defined as a pair of functions.
    get accessorProp() { return this.dataProp; },
    set accessorProp(value) { this.dataProp = value; }
};
```

访问器属性被定义为一个或两个方法，其名称与属性名称相同。它们看起来像使用 ES6 简写定义的普通方法，只是 getter 和 setter 定义前缀为`get`或`set`。（在 ES6 中，当定义 getter 和 setter 时，也可以使用计算属性名称。只需在`get`或`set`后用方括号中的表达式替换属性名称。）

上面定义的访问器方法只是获取和设置数据属性的值，并没有理由优先使用访问器属性而不是数据属性。但作为一个更有趣的例子，考虑以下表示 2D 笛卡尔点的对象。它具有普通数据属性来表示点的*x*和*y*坐标，并且具有访问器属性来给出点的等效极坐标：

```js
let p = {
    // x and y are regular read-write data properties.
    x: 1.0,
    y: 1.0,

    // r is a read-write accessor property with getter and setter.
    // Don't forget to put a comma after accessor methods.
    get r() { return Math.hypot(this.x, this.y); },
    set r(newvalue) {
        let oldvalue = Math.hypot(this.x, this.y);
        let ratio = newvalue/oldvalue;
        this.x *= ratio;
        this.y *= ratio;
    },

    // theta is a read-only accessor property with getter only.
    get theta() { return Math.atan2(this.y, this.x); }
};
p.r     // => Math.SQRT2
p.theta // => Math.PI / 4
```

注意在这个例子中，关键字`this`在 getter 和 setter 中的使用。JavaScript 将这些函数作为定义它们的对象的方法调用，这意味着在函数体内，`this`指的是点对象`p`。因此，`r`属性的 getter 方法可以将`x`和`y`属性称为`this.x`和`this.y`。更详细地讨论方法和`this`关键字在§8.2.2 中有介绍。

访问器属性是继承的，就像数据属性一样，因此可以将上面定义的对象`p`用作其他点的原型。您可以为新对象提供它们自己的`x`和`y`属性，并且它们将继承`r`和`theta`属性：

```js
let q = Object.create(p); // A new object that inherits getters and setters
q.x = 3; q.y = 4;         // Create q's own data properties
q.r                       // => 5: the inherited accessor properties work
q.theta                   // => Math.atan2(4, 3)
```

上面的代码使用访问器属性来定义一个 API，提供单组数据的两种表示（笛卡尔坐标和极坐标）。使用访问器属性的其他原因包括对属性写入进行检查和在每次属性读取时返回不同的值：

```js
// This object generates strictly increasing serial numbers
const serialnum = {
    // This data property holds the next serial number.
    // The _ in the property name hints that it is for internal use only.
    _n: 0,

    // Return the current value and increment it
    get next() { return this._n++; },

    // Set a new value of n, but only if it is larger than current
    set next(n) {
        if (n > this._n) this._n = n;
        else throw new Error("serial number can only be set to a larger value");
    }
};
serialnum.next = 10;    // Set the starting serial number
serialnum.next          // => 10
serialnum.next          // => 11: different value each time we get next
```

最后，这里是另一个示例，使用 getter 方法实现具有“神奇”行为的属性：

```js
// This object has accessor properties that return random numbers.
// The expression "random.octet", for example, yields a random number
// between 0 and 255 each time it is evaluated.
const random = {
    get octet() { return Math.floor(Math.random()*256); },
    get uint16() { return Math.floor(Math.random()*65536); },
    get int16() { return Math.floor(Math.random()*65536)-32768; }
};
```

# 6.11 总结

本章详细记录了 JavaScript 对象，涵盖的主题包括：

+   基本对象术语，包括诸如*可枚举*和*自有属性*等术语的含义。

+   对象字面量语法，包括 ES6 及以后版本中的许多新特性。

+   如何读取、写入、删除、枚举和检查对象的属性是否存在。

+   JavaScript 中基于原型的继承是如何工作的，以及如何使用`Object.create()`创建一个继承自另一个对象的对象。

+   如何使用`Object.assign()`将一个对象的属性复制到另一个对象中。

所有非原始值的 JavaScript 值都是对象。这包括数组和函数，它们是接下来两章的主题。

¹ 记住；几乎所有对象都有一个原型，但大多数对象没有名为`prototype`的属性。即使无法直接访问原型对象，JavaScript 继承仍然有效。但如果想学习如何做到这一点，请参见§14.3。
