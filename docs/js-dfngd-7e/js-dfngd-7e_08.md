# 第八章：函数

本章涵盖了 JavaScript 函数。函数是 JavaScript 程序的基本构建块，也是几乎所有编程语言中的常见特性。您可能已经熟悉了类似于*子程序*或*过程*的函数概念。

*函数*是一段 JavaScript 代码块，定义一次但可以执行或*调用*任意次数。JavaScript 函数是*参数化*的：函数定义可能包括一个标识符列表，称为*参数*，它们在函数体内作为局部变量。函数调用为函数的参数提供值，或*参数*，函数通常使用它们的参数值来计算*返回值*，该返回值成为函数调用表达式的值。除了参数之外，每次调用还有另一个值—*调用上下文*—它是`this`关键字的值。

如果函数分配给对象的属性，则称为该对象的*方法*。当在对象上调用函数时，该对象是函数的调用上下文或`this`值。用于初始化新创建对象的函数称为*构造函数*。构造函数在§6.2 中有描述，并将在第九章中再次介绍。

在 JavaScript 中，函数是对象，可以被程序操作。JavaScript 可以将函数分配给变量并将它们传递给其他函数，例如。由于函数是对象，您可以在它们上设置属性，甚至在它们上调用方法。

JavaScript 函数定义可以嵌套在其他函数中，并且可以访问在定义它们的作用域中的任何变量。这意味着 JavaScript 函数是*闭包*，并且它们可以实现重要且强大的编程技术。

# 8.1 定义函数

定义 JavaScript 函数最直接的方法是使用`function`关键字，可以用作声明或表达式。ES6 定义了一种重要的新定义函数的方式，即“箭头函数”没有`function`关键字：箭头函数具有特别简洁的语法，并且在将一个函数作为另一个函数的参数传递时非常有用。接下来的小节将介绍这三种定义函数的方式。请注意，涉及函数参数的函数定义语法的一些细节将推迟到§8.3 中。

在对象字面量和类定义中，有一种方便的简写语法用于定义方法。这种简写语法在§6.10.5 中介绍过，相当于使用函数定义表达式并将其分配给对象属性，使用基本的`name:value`对象字面量语法。在另一种特殊情况下，您可以在对象字面量中使用关键字`get`和`set`来定义特殊的属性获取器和设置器方法。这种函数定义语法在§6.10.6 中介绍过。

请注意，函数也可以使用`Function()`构造函数来定义，这是§8.7.7 的主题。此外，JavaScript 定义了一些特殊类型的函数。`function*`定义生成器函数（参见第十二章），而`async function`定义异步函数（参见第十三章）。

## 8.1.1 函数声明

函数声明由`function`关键字后跟这些组件组成：

+   用于命名函数的标识符。名称是函数声明的必需部分：它用作变量的名称，并且新定义的函数对象分配给该变量。

+   一对括号围绕着一个逗号分隔的零个或多个标识符列表。这些标识符是函数的参数名称，并且在函数体内部起到类似局部变量的作用。

+   一对大括号内包含零个或多个 JavaScript 语句。这些语句是函数的主体：每当调用函数时，它们都会被执行。

这里是一些示例函数声明：

```js
// Print the name and value of each property of o.  Return undefined.
function printprops(o) {
    for(let p in o) {
        console.log(`${p}: ${o[p]}\n`);
    }
}

// Compute the distance between Cartesian points (x1,y1) and (x2,y2).
function distance(x1, y1, x2, y2) {
    let dx = x2 - x1;
    let dy = y2 - y1;
    return Math.sqrt(dx*dx + dy*dy);
}

// A recursive function (one that calls itself) that computes factorials
// Recall that x! is the product of x and all positive integers less than it.
function factorial(x) {
    if (x <= 1) return 1;
    return x * factorial(x-1);
}
```

关于函数声明的重要事项之一是，函数的名称成为一个变量，其值为函数本身。函数声明语句被“提升”到封闭脚本、函数或块的顶部，以便以这种方式定义的函数可以从定义之前的代码中调用。另一种说法是，在 JavaScript 代码块中声明的所有函数将在该块中定义，并且它们将在 JavaScript 解释器开始执行该块中的任何代码之前定义。

我们描述的 `distance()` 和 `factorial()` 函数旨在计算一个值，并使用 `return` 将该值返回给调用者。`return` 语句导致函数停止执行并将其表达式的值（如果有）返回给调用者。如果 `return` 语句没有关联的表达式，则函数的返回值为 `undefined`。

`printprops()` 函数有所不同：它的作用是输出对象属性的名称和值。不需要返回值，并且函数不包括 `return` 语句。调用 `printprops()` 函数的值始终为 `undefined`。如果函数不包含 `return` 语句，它只是执行函数体中的每个语句，直到达到结尾，并将 `undefined` 值返回给调用者。

在 ES6 之前，只允许在 JavaScript 文件的顶层或另一个函数内部定义函数声明。虽然一些实现弯曲了规则，但在循环、条件语句或其他块的主体内定义函数实际上是不合法的。然而，在 ES6 的严格模式下，允许在块内部声明函数。在块内定义的函数仅存在于该块内部，并且在块外部不可见。

## 8.1.2 函数表达式

函数表达式看起来很像函数声明，但它们出现在更大表达式或语句的上下文中，名称是可选的。这里是一些示例函数表达式：

```js
// This function expression defines a function that squares its argument.
// Note that we assign it to a variable
const square = function(x) { return x*x; };

// Function expressions can include names, which is useful for recursion.
const f = function fact(x) { if (x <= 1) return 1; else return x*fact(x-1); };

// Function expressions can also be used as arguments to other functions:
[3,2,1].sort(function(a,b) { return a-b; });

// Function expressions are sometimes defined and immediately invoked:
let tensquared = (function(x) {return x*x;}(10));
```

请注意，对于定义为表达式的函数，函数名称是可选的，我们展示的大多数前面的函数表达式都省略了它。函数声明实际上 *声明* 了一个变量，并将函数对象分配给它。另一方面，函数表达式不声明变量：如果您需要多次引用它，您需要将新定义的函数对象分配给常量或变量。对于函数表达式，最好使用 `const`，这样您不会意外地通过分配新值来覆盖函数。

对于需要引用自身的函数（如阶乘函数），允许为函数指定名称。如果函数表达式包含名称，则该函数的本地函数作用域将包括将该名称绑定到函数对象。实际上，函数名称成为函数内部的局部变量。大多数作为表达式定义的函数不需要名称，这使得它们的定义更加紧凑（尽管不像下面描述的箭头函数那样紧凑）。

使用函数声明定义函数`f()`与在创建后将函数分配给变量`f`之间有一个重要的区别。当使用声明形式时，函数对象在包含它们的代码开始运行之前就已经创建，并且定义被提升，以便您可以从出现在定义语句上方的代码中调用这些函数。然而，对于作为表达式定义的函数来说，情况并非如此：这些函数直到定义它们的表达式实际被评估之后才存在。此外，为了调用一个函数，您必须能够引用它，而在将函数定义为表达式之前，您不能引用一个函数，因此使用表达式定义的函数在定义之前不能被调用。

## 8.1.3 箭头函数

在 ES6 中，你可以使用一种特别简洁的语法来定义函数，称为“箭头函数”。这种语法类似于数学表示法，并使用`=>`“箭头”来分隔函数参数和函数主体。不使用`function`关键字，而且，由于箭头函数是表达式而不是语句，因此也不需要函数名称。箭头函数的一般形式是用括号括起来的逗号分隔的参数列表，后跟`=>`箭头，再后跟用花括号括起来的函数主体：

```js
const sum = (x, y) => { return x + y; };
```

但是箭头函数支持更紧凑的语法。如果函数的主体是一个单独的`return`语句，您可以省略`return`关键字、与之配套的分号和花括号，并将函数主体写成要返回其值的表达式：

```js
const sum = (x, y) => x + y;
```

此外，如果箭头函数只有一个参数，您可以省略参数列表周围的括号：

```js
const polynomial = x => x*x + 2*x + 3;
```

请注意，一个没有任何参数的箭头函数必须用一个空的括号对写成：

```js
const constantFunc = () => 42;
```

请注意，在编写箭头函数时，不要在函数参数和`=>`箭头之间加入新行。否则，您可能会得到一行像`const polynomial = x`这样的行，这是一个语法上有效的赋值语句。

此外，如果箭头函数的主体是一个单独的`return`语句，但要返回的表达式是一个对象字面量，则必须将对象字面量放在括号内，以避免在函数主体的花括号和对象字面量的花括号之间产生语法歧义：

```js
const f = x => { return { value: x }; };  // Good: f() returns an object
const g = x => ({ value: x });            // Good: g() returns an object
const h = x => { value: x };              // Bad: h() returns nothing
const i = x => { v: x, w: x };            // Bad: Syntax Error
```

在此代码的第三行中，函数`h()`确实是模棱两可的：您打算作为对象字面量的代码可以被解析为标记语句，因此创建了一个返回`undefined`的函数。然而，在第四行，更复杂的对象字面量不是一个有效的语句，这种非法代码会导致语法错误。

箭头函数简洁的语法使它们在需要将一个函数传递给另一个函数时非常理想，这在像`map()`、`filter()`和`reduce()`这样的数组方法中是常见的做法（参见§7.8.1）：

```js
// Make a copy of an array with null elements removed.
let filtered = [1,null,2,3].filter(x => x !== null); // filtered == [1,2,3]
// Square some numbers:
let squares = [1,2,3,4].map(x => x*x);               // squares == [1,4,9,16]
```

箭头函数与其他方式定义的函数在一个关键方面有所不同：它们继承自定义它们的环境中的`this`关键字的值，而不是像其他方式定义的函数那样定义自己的调用上下文。这是箭头函数的一个重要且非常有用的特性，我们将在本章后面再次回到这个问题。箭头函数还与其他函数不同之处在于它们没有`prototype`属性，这意味着它们不能用作新类的构造函数（参见§9.2）。

## 8.1.4 嵌套函数

在 JavaScript 中，函数可以嵌套在其他函数中。例如：

```js
function hypotenuse(a, b) {
    function square(x) { return x*x; }
    return Math.sqrt(square(a) + square(b));
}
```

嵌套函数的有趣之处在于它们的变量作用域规则：它们可以访问嵌套在其中的函数（或函数）的参数和变量。例如，在这里显示的代码中，内部函数 `square()` 可以读取和写入外部函数 `hypotenuse()` 定义的参数 `a` 和 `b`。嵌套函数的这些作用域规则非常重要，我们将在 §8.6 中再次考虑它们。

# 8.2 调用函数

JavaScript 函数体组成的代码在定义函数时不会执行，而是在调用函数时执行。JavaScript 函数可以通过五种方式调用：

+   作为函数

+   作为方法

+   作为构造函数

+   通过它们的 `call()` 和 `apply()` 方法间接调用

+   隐式地，通过 JavaScript 语言特性，看起来不像正常函数调用

## 8.2.1 函数调用

函数可以作为函数或方法通过调用表达式调用（§4.5）。调用表达式由一个求值为函数对象的函数表达式、一个开括号、一个逗号分隔的零个或多个参数表达式和一个闭括号组成。如果函数表达式是一个属性访问表达式——如果函数是对象的属性或数组的元素——那么它就是一个方法调用表达式。这种情况将在下面的示例中解释。以下代码包含了许多常规函数调用表达式：

```js
printprops({x: 1});
let total = distance(0,0,2,1) + distance(2,1,3,5);
let probability = factorial(5)/factorial(13);
```

在调用中，每个参数表达式（括号之间的表达式）都会被求值，得到的值作为函数的参数。这些值被分配给函数定义中命名的参数。在函数体中，对参数的引用会求值为相应的参数值。

对于常规函数调用，函数的返回值成为调用表达式的值。如果函数返回是因为解释器到达末尾，返回值是 `undefined`。如果函数返回是因为解释器执行了 `return` 语句，则返回值是跟在 `return` 后面的表达式的值，如果 `return` 语句没有值，则返回值是 `undefined`。

在非严格模式下进行函数调用时，调用上下文（`this` 值）是全局对象。然而，在严格模式下，调用上下文是 `undefined`。请注意，使用箭头语法定义的函数行为不同：它们始终继承在定义它们的地方生效的 `this` 值。

为了作为函数调用而编写的函数（而不是作为方法调用），通常根本不使用 `this` 关键字。然而，可以使用该关键字来确定是否启用了严格模式：

```js
// Define and invoke a function to determine if we're in strict mode.
const strict = (function() { return !this; }());
```

## 8.2.2 方法调用

*方法* 只不过是存储在对象属性中的 JavaScript 函数。如果有一个函数 `f` 和一个对象 `o`，你可以用以下代码定义 `o` 的名为 `m` 的方法：

```js
o.m = f;
```

定义了对象 `o` 的方法 `m()` 后，可以像这样调用它：

```js
o.m();
```

或者，如果 `m()` 预期有两个参数，你可以这样调用它：

```js
o.m(x, y);
```

此示例中的代码是一个调用表达式：它包括一个函数表达式 `o.m` 和两个参数表达式 `x` 和 `y`。函数表达式本身是一个属性访问表达式，这意味着该函数被作为方法而不是作为常规函数调用。

方法调用的参数和返回值的处理方式与常规函数调用完全相同。然而，方法调用与函数调用有一个重要的区别：调用上下文。属性访问表达式由两部分组成：一个对象（在本例中是 `o`）和一个属性名（`m`）。在这样的方法调用表达式中，对象 `o` 成为调用上下文，函数体可以通过关键字 `this` 引用该对象。以下是一个具体示例：

```js
let calculator = { // An object literal
    operand1: 1,
    operand2: 1,
    add() {        // We're using method shorthand syntax for this function
        // Note the use of the this keyword to refer to the containing object.
        this.result = this.operand1 + this.operand2;
    }
};
calculator.add();  // A method invocation to compute 1+1.
calculator.result  // => 2
```

大多数方法调用使用点表示法进行属性访问，但使用方括号的属性访问表达式也会导致方法调用。例如，以下两者都是方法调用：

```js
o"m";   // Another way to write o.m(x,y).
a0        // Also a method invocation (assuming a[0] is a function).
```

方法调用也可能涉及更复杂的属性访问表达式：

```js
customer.surname.toUpperCase(); // Invoke method on customer.surname
f().m();                        // Invoke method m() on return value of f()
```

方法和`this`关键字是面向对象编程范式的核心。任何用作方法的函数实际上都会传递一个隐式参数——通过它被调用的对象。通常，方法在该对象上执行某种操作，而方法调用语法是一种优雅地表达函数正在操作对象的方式。比较以下两行：

```js
rect.setSize(width, height);
setRectSize(rect, width, height);
```

在这两行代码中调用的假设函数可能对（假设的）对象`rect`执行完全相同的操作，但第一行中的方法调用语法更清楚地表明了对象`rect`是操作的主要焦点。

请注意`this`是一个关键字，不是变量或属性名。JavaScript 语法不允许您为`this`赋值。

`this`关键字的作用域不同于变量，除了箭头函数外，嵌套函数不会继承包含函数的`this`值。如果嵌套函数被作为方法调用，其`this`值将是调用它的对象。如果嵌套函数（不是箭头函数）被作为函数调用，那么其`this`值将是全局对象（非严格模式）或`undefined`（严格模式）。假设在方法内部定义的嵌套函数并作为函数调用时可以使用`this`获取方法的调用上下文是一个常见的错误。以下代码演示了这个问题：

```js
let o = {                 // An object o.
    m: function() {       // Method m of the object.
        let self = this;  // Save the "this" value in a variable.
        this === o        // => true: "this" is the object o.
        f();              // Now call the helper function f().

        function f() {    // A nested function f
            this === o    // => false: "this" is global or undefined
            self === o    // => true: self is the outer "this" value.
        }
    }
};
o.m();                    // Invoke the method m on the object o.
```

在嵌套函数`f()`内部，`this`关键字不等于对象`o`。这被广泛认为是 JavaScript 语言的一个缺陷，因此重要的是要意识到这一点。上面的代码演示了一个常见的解决方法。在方法`m`内部，我们将`this`值分配给变量`self`，在嵌套函数`f`内部，我们可以使用`self`而不是`this`来引用包含的对象。

在 ES6 及更高版本中，另一个解决此问题的方法是将嵌套函数`f`转换为箭头函数，这样将正确继承`this`值。

```js
const f = () => {
    this === o  // true, since arrow functions inherit this
};
```

将函数定义为表达式而不是语句的方式不会被提升，因此为了使这段代码正常工作，函数`f`的定义需要移动到方法`m`内部，以便在调用之前出现。

另一个解决方法是调用嵌套函数的`bind()`方法来定义一个新函数，该函数将隐式在指定对象上调用：

```js
const f = (function() {
    this === o  // true, since we bound this function to the outer this
}).bind(this);
```

我们将在§8.7.5 中更详细地讨论`bind()`。

## 8.2.3 构造函数调用

如果函数或方法调用之前带有关键字`new`，那么这是一个构造函数调用。（构造函数调用在§4.6 和§6.2.2 中介绍过，并且构造函数将在第九章中更详细地讨论。）构造函数调用在处理参数、调用上下文和返回值方面与常规函数和方法调用不同。

如果构造函数调用包括括号中的参数列表，则这些参数表达式将被计算并传递给函数，方式与函数和方法调用相同。虽然不常见，但您可以在构造函数调用中省略一对空括号。例如，以下两行是等价的：

```js
o = new Object();
o = new Object;
```

构造函数调用创建一个新的空对象，该对象继承自构造函数的`prototype`属性指定的对象。构造函数旨在初始化对象，这个新创建的对象被用作调用上下文，因此构造函数可以使用`this`关键字引用它。请注意，即使构造函数调用看起来像方法调用，新对象也被用作调用上下文。也就是说，在表达式`new o.m()`中，`o`不被用作调用上下文。

构造函数通常不使用`return`关键字。它们通常在初始化新对象后隐式返回，当它们到达函数体的末尾时。在这种情况下，新对象是构造函数调用表达式的值。然而，如果构造函数显式使用`return`语句返回一个对象，则该对象成为调用表达式的值。如果构造函数使用没有值的`return`，或者返回一个原始值，那么返回值将被忽略，新对象将作为调用的值。

## 8.2.4 间接调用

JavaScript 函数是对象，和所有 JavaScript 对象一样，它们有方法。其中两个方法，`call()`和`apply()`，间接调用函数。这两种方法允许您明确指定调用的`this`值，这意味着您可以将任何函数作为任何对象的方法调用，即使它实际上不是该对象的方法。这两种方法还允许您指定调用的参数。`call()`方法使用其自己的参数列表作为函数的参数，而`apply()`方法期望使用作为参数的值数组。`call()`和`apply()`方法在§8.7.4 中有详细描述。

## 8.2.5 隐式函数调用

有各种 JavaScript 语言特性看起来不像函数调用，但会导致函数被调用。在编写可能被隐式调用的函数时要特别小心，因为这些函数中的错误、副作用和性能问题比普通函数更难诊断和修复，因为从简单检查代码时可能不明显它们何时被调用。

可能导致隐式函数调用的语言特性包括：

+   如果对象定义了 getter 或 setter，则查询或设置其属性的值可能会调用这些方法。更多信息请参见§6.10.6。

+   当对象在字符串上下文中使用（例如与字符串连接时），会调用其`toString()`方法。类似地，当对象在数值上下文中使用时，会调用其`valueOf()`方法。详细信息请参见§3.9.3。

+   当您遍历可迭代对象的元素时，会发生许多方法调用。第十二章解释了迭代器在函数调用级别上的工作原理，并演示了如何编写这些方法，以便您可以定义自己的可迭代类型。

+   标记模板字面量是一个伪装成函数调用的函数。§14.5 演示了如何编写可与模板字面量字符串一起使用的函数。

+   代理对象（在§14.7 中描述）的行为完全由函数控制。对这些对象的几乎任何操作都会导致函数被调用。

# 8.3 函数参数和参数

JavaScript 函数定义不指定函数参数的预期类型，函数调用也不对传递的参数值进行任何类型检查。事实上，JavaScript 函数调用甚至不检查传递的参数数量。接下来的小节描述了当函数被调用时传入的参数少于声明的参数数量或多于声明的参数数量时会发生什么。它们还演示了如何显式测试函数参数的类型，如果需要确保函数不会被不适当的参数调用。

## 8.3.1 可选参数和默认值

当函数被调用时传入的参数少于声明的参数数量时，额外的参数将被设置为它们的默认值，通常是`undefined`。编写一些参数是可选的函数通常很有用。以下是一个例子：

```js
// Append the names of the enumerable properties of object o to the
// array a, and return a.  If a is omitted, create and return a new array.
function getPropertyNames(o, a) {
    if (a === undefined) a = [];  // If undefined, use a new array
    for(let property in o) a.push(property);
    return a;
}

// getPropertyNames() can be invoked with one or two arguments:
let o = {x: 1}, p = {y: 2, z: 3};  // Two objects for testing
let a = getPropertyNames(o); // a == ["x"]; get o's properties in a new array
getPropertyNames(p, a);      // a == ["x","y","z"]; add p's properties to it
```

在这个函数的第一行使用`if`语句的地方，你可以以这种成语化的方式使用`||`运算符：

```js
a = a || [];
```

回想一下§4.10.2 中提到的`||`运算符，如果第一个参数为真，则返回第一个参数，否则返回第二个参数。在这种情况下，如果将任何对象作为第二个参数传递，函数将使用该对象。但如果省略第二个参数（或传递`null`或另一个假值），则将使用一个新创建的空数组。

注意，在设计具有可选参数的函数时，应确保将可选参数放在参数列表的末尾，以便可以省略它们。调用函数的程序员不能省略第一个参数并传递第二个参数：他们必须明确地将`undefined`作为第一个参数传递。

在 ES6 及更高版本中，你可以直接在函数参数列表中为每个参数定义默认值。只需在参数名称后面加上等号和默认值，当没有为该参数提供参数时将使用默认值：

```js
// Append the names of the enumerable properties of object o to the
// array a, and return a.  If a is omitted, create and return a new array.
function getPropertyNames(o, a = []) {
    for(let property in o) a.push(property);
    return a;
}
```

参数默认表达式在调用函数时进行求值，而不是在定义函数时进行求值，因此每次调用`getPropertyNames()`函数时，都会创建一个新的空数组并传递。² 如果参数默认值是常量（或类似`[]`和`{}`的文字表达式），那么函数的推理可能是最简单的。但这不是必需的：你可以使用变量或函数调用来计算参数的默认值。一个有趣的情况是，对于具有多个参数的函数，可以使用前一个参数的值来定义其后参数的默认值：

```js
// This function returns an object representing a rectangle's dimensions.
// If only width is supplied, make it twice as high as it is wide.
const rectangle = (width, height=width*2) => ({width, height});
rectangle(1)  // => { width: 1, height: 2 }
```

这段代码演示了参数默认值如何与箭头函数一起使用。对于方法简写函数和所有其他形式的函数定义也是如此。

## 8.3.2 Rest 参数和可变长度参数列表

参数默认值使我们能够编写可以用比参数更少的参数调用的函数。*Rest 参数*使相反的情况成为可能：它们允许我们编写可以用任意多个参数调用的函数。以下是一个期望一个或多个数字参数并返回最大值的示例函数：

```js
function max(first=-Infinity, ...rest) {
    let maxValue = first; // Start by assuming the first arg is biggest
    // Then loop through the rest of the arguments, looking for bigger
    for(let n of rest) {
        if (n > maxValue) {
            maxValue = n;
        }
    }
    // Return the biggest
    return maxValue;
}

max(1, 10, 100, 2, 3, 1000, 4, 5, 6)  // => 1000
```

rest 参数由三个点前置，并且必须是函数声明中的最后一个参数。当你使用 rest 参数调用函数时，你传递的参数首先被分配给非 rest 参数，然后任何剩余的参数（即“剩余”的参数）都存储在一个数组中，该数组成为 rest 参数的值。这一点很重要：在函数体内，rest 参数的值始终是一个数组。数组可能为空，但 rest 参数永远不会是`undefined`。（由此可知，为 rest 参数定义参数默认值从未有用过，也不合法。）

像前面的例子那样可以接受任意数量参数的函数称为*可变参数函数*、*可变参数函数*或*vararg 函数*。本书使用最口语化的术语*varargs*，这个术语可以追溯到 C 编程语言的早期。

不要混淆函数定义中定义 rest 参数的 `...` 与 §8.3.4 中描述的展开运算符的 `...`，后者可用于函数调用中。

## 8.3.3 Arguments 对象

Rest 参数是在 ES6 中引入 JavaScript 的。在该语言版本之前，可变参数函数是使用 Arguments 对象编写的：在任何函数体内，标识符 `arguments` 指的是该调用的 Arguments 对象。Arguments 对象是一个类似数组的对象（参见 §7.9），允许按数字而不是名称检索传递给函数的参数值。以下是之前的 `max()` 函数，重写以使用 Arguments 对象而不是 rest 参数：

```js
function max(x) {
    let maxValue = -Infinity;
    // Loop through the arguments, looking for, and remembering, the biggest.
    for(let i = 0; i < arguments.length; i++) {
        if (arguments[i] > maxValue) maxValue = arguments[i];
    }
    // Return the biggest
    return maxValue;
}

max(1, 10, 100, 2, 3, 1000, 4, 5, 6)  // => 1000
```

Arguments 对象可以追溯到 JavaScript 最早的日子，并且携带一些奇怪的历史包袱，使其在严格模式之外尤其难以优化和难以使用。你可能仍然会遇到使用 Arguments 对象的代码，但是在编写任何新代码时应避免使用它。在重构旧代码时，如果遇到使用 `arguments` 的函数，通常可以用 `...args` rest 参数来替换它。Arguments 对象的不幸遗产之一是，在严格模式下，`arguments` 被视为保留字，你不能使用该名称声明函数参数或局部变量。

## 8.3.4 函数调用的展开运算符

展开运算符 `...` 用于在期望单个值的上下文中解包或“展开”数组（或任何其他可迭代对象，如字符串）的元素。我们在 §7.1.2 中看到展开运算符与数组文字一起使用。该运算符可以以相同的方式在函数调用中使用：

```js
let numbers = [5, 2, 10, -1, 9, 100, 1];
Math.min(...numbers)  // => -1
```

请注意，`...` 不是真正的运算符，因为它不能被评估为产生一个值。相反，它是一种特殊的 JavaScript 语法，可用于数组文字和函数调用中。

当我们在函数定义中而不是函数调用中使用相同的 `...` 语法时，它的效果与展开运算符相反。正如我们在 §8.3.2 中看到的，使用 `...` 在函数定义中将多个函数参数收集到一个数组中。Rest 参数和展开运算符通常一起使用，如下面的函数，该函数接受一个函数参数，并返回一个用于测试的函数的版本：

```js
// This function takes a function and returns a wrapped version
function timed(f) {
    return function(...args) {  // Collect args into a rest parameter array
        console.log(`Entering function ${f.name}`);
        let startTime = Date.now();
        try {
            // Pass all of our arguments to the wrapped function
            return f(...args);  // Spread the args back out again
        }
        finally {
            // Before we return the wrapped return value, print elapsed time.
            console.log(`Exiting ${f.name} after ${Date.now()-startTime}ms`);
        }
    };
}

// Compute the sum of the numbers between 1 and n by brute force
function benchmark(n) {
    let sum = 0;
    for(let i = 1; i <= n; i++) sum += i;
    return sum;
}

// Now invoke the timed version of that test function
timed(benchmark)(1000000) // => 500000500000; this is the sum of the numbers
```

## 8.3.5 将函数参数解构为参数

当你使用一系列参数值调用函数时，这些值最终被分配给函数定义中声明的参数。函数调用的初始阶段很像变量赋值。因此，我们可以使用解构赋值技术（参见 §3.10.3）与函数一起使用，这并不奇怪。

如果你定义一个带有方括号内参数名称的函数，那么你告诉函数期望传递一个数组值以用于每对方括号。在调用过程中，数组参数将被解包到各个命名参数中。举个例子，假设我们将 2D 向量表示为包含两个数字的数组，其中第一个元素是 X 坐标，第二个元素是 Y 坐标。使用这种简单的数据结构，我们可以编写以下函数来添加两个向量：

```js
function vectorAdd(v1, v2) {
    return [v1[0] + v2[0], v1[1] + v2[1]];
}
vectorAdd([1,2], [3,4])  // => [4,6]
```

如果我们将两个向量参数解构为更清晰命名的参数，代码将更容易理解：

```js
function vectorAdd([x1,y1], [x2,y2]) { // Unpack 2 arguments into 4 parameters
    return [x1 + x2, y1 + y2];
}
vectorAdd([1,2], [3,4])  // => [4,6]
```

同样，如果你正在定义一个期望对象参数的函数，你可以解构该对象的参数。再次使用矢量示例，假设我们将矢量表示为具有`x`和`y`参数的对象：

```js
// Multiply the vector {x,y} by a scalar value
function vectorMultiply({x, y}, scalar) {
    return { x: x*scalar, y: y*scalar };
}
vectorMultiply({x: 1, y: 2}, 2)  // => {x: 2, y: 4}
```

将单个对象参数解构为两个参数的示例相当清晰，因为我们使用的参数名称与传入对象的属性名称匹配。当你需要将具有一个名称的属性解构为具有不同名称的参数时，语法会更冗长且更令人困惑。这里是基于对象的矢量的矢量加法示例的实现：

```js
function vectorAdd(
    {x: x1, y: y1}, // Unpack 1st object into x1 and y1 params
    {x: x2, y: y2}  // Unpack 2nd object into x2 and y2 params
)
{
    return { x: x1 + x2, y: y1 + y2 };
}
vectorAdd({x: 1, y: 2}, {x: 3, y: 4})  // => {x: 4, y: 6}
```

关于解构语法如`{x:x1, y:y1}`，让人难以记住哪些是属性名称，哪些是参数名称。要记住解构赋值和解构函数调用的规则是，被声明的变量或参数放在你期望值在对象字面量中的位置。因此，属性名称始终在冒号的左侧，参数（或变量）名称在右侧。

你可以使用解构参数定义参数默认值。这里是适用于 2D 或 3D 矢量的矢量乘法：

```js
// Multiply the vector {x,y} or {x,y,z} by a scalar value
function vectorMultiply({x, y, z=0}, scalar) {
    return { x: x*scalar, y: y*scalar, z: z*scalar };
}
vectorMultiply({x: 1, y: 2}, 2)  // => {x: 2, y: 4, z: 0}
```

一些语言（如 Python）允许函数的调用者以`name=value`形式指定参数调用函数，当存在许多可选参数或参数列表足够长以至于难以记住正确顺序时，这是很方便的。JavaScript 不直接允许这样做，但你可以通过将对象参数解构为函数参数来近似实现。考虑一个函数，它从一个数组中复制指定数量的元素到另一个数组中，并为每个数组指定可选的起始偏移量。由于有五个可能的参数，其中一些具有默认值，并且调用者很难记住传递参数的顺序，我们可以像这样定义和调用`arraycopy()`函数：

```js
function arraycopy({from, to=from, n=from.length, fromIndex=0, toIndex=0}) {
    let valuesToCopy = from.slice(fromIndex, fromIndex + n);
    to.splice(toIndex, 0, ...valuesToCopy);
    return to;
}
let a = [1,2,3,4,5], b = [9,8,7,6,5];
arraycopy({from: a, n: 3, to: b, toIndex: 4}) // => [9,8,7,6,1,2,3,5]
```

当你解构一个数组时，你可以为被解构的数组中的额外值定义一个剩余参数。方括号内的剩余参数与函数的真正剩余参数完全不同：

```js
// This function expects an array argument. The first two elements of that
// array are unpacked into the x and y parameters. Any remaining elements
// are stored in the coords array. And any arguments after the first array
// are packed into the rest array.
function f([x, y, ...coords], ...rest) {
    return [x+y, ...rest, ...coords];  // Note: spread operator here
}
f([1, 2, 3, 4], 5, 6)   // => [3, 5, 6, 3, 4]
```

在 ES2018 中，当你解构一个对象时，也可以使用剩余参数。该剩余参数的值将是一个对象，其中包含未被解构的任何属性。对象剩余参数通常与对象展开运算符一起使用，这也是 ES2018 的一个新功能：

```js
// Multiply the vector {x,y} or {x,y,z} by a scalar value, retain other props
function vectorMultiply({x, y, z=0, ...props}, scalar) {
    return { x: x*scalar, y: y*scalar, z: z*scalar, ...props };
}
vectorMultiply({x: 1, y: 2, w: -1}, 2)  // => {x: 2, y: 4, z: 0, w: -1}
```

最后，请记住，除了解构参数对象和数组外，你还可以解构对象数组、具有数组属性的对象以及具有对象属性的对象，实际上可以解构到任何深度。考虑表示圆的图形代码，其中圆被表示为具有`x`、`y`、`半径`和`颜色`属性的对象，其中`颜色`属性是红色、绿色和蓝色颜色分量的数组。你可以定义一个函数，该函数期望传递一个圆对象，但将该圆对象解构为六个单独的参数：

```js
function drawCircle({x, y, radius, color: [r, g, b]}) {
    // Not yet implemented
}
```

如果函数参数解构比这更复杂，我发现代码变得更难阅读，而不是更简单。有时，明确地访问对象属性和数组索引会更清晰。

## 8.3.6 参数类型

JavaScript 方法参数没有声明类型，并且不对传递给函数的值执行类型检查。通过为函数参数选择描述性名称并在每个函数的注释中仔细记录它们，可以帮助使代码自我描述。（或者，参见§17.8 中允许你在常规 JavaScript 之上添加类型检查的语言扩展。）

如 §3.9 中所述，JavaScript 根据需要执行自由的类型转换。因此，如果您编写一个期望字符串参数的函数，然后使用其他类型的值调用该函数，那么当函数尝试将其用作字符串时，您传递的值将被简单地转换为字符串。所有原始类型都可以转换为字符串，所有对象都有 `toString()` 方法（不一定是有用的），因此在这种情况下不会发生错误。

然而，这并不总是正确的。再次考虑之前显示的 `arraycopy()` 方法。它期望一个或两个数组参数，并且如果这些参数的类型错误，则会失败。除非您正在编写一个只会从代码附近的部分调用的私有函数，否则值得添加代码来检查参数的类型。当传递错误的值时，最好让函数立即和可预测地失败，而不是开始执行然后在稍后失败并显示可能不清晰的错误消息。这里有一个执行类型检查的示例函数：

```js
// Return the sum of the elements an iterable object a.
// The elements of a must all be numbers.
function sum(a) {
    let total = 0;
    for(let element of a) { // Throws TypeError if a is not iterable
        if (typeof element !== "number") {
            throw new TypeError("sum(): elements must be numbers");
        }
        total += element;
    }
    return total;
}
sum([1,2,3])    // => 6
sum(1, 2, 3);   // !TypeError: 1 is not iterable
sum([1,2,"3"]); // !TypeError: element 2 is not a number
```

# 8.4 函数作为值

函数最重要的特点是它们可以被定义和调用。函数的定义和调用是 JavaScript 和大多数其他编程语言的语法特性。然而，在 JavaScript 中，函数不仅仅是语法，还是值，这意味着它们可以被分配给变量，存储在对象的属性或数组的元素中，作为函数的参数传递等。³

要理解函数如何既可以是 JavaScript 数据又可以是 JavaScript 语法，请考虑这个函数定义：

```js
function square(x) { return x*x; }
```

这个定义创建了一个新的函数对象并将其分配给变量 `square`。函数的名称实际上并不重要；它只是一个指向函数对象的变量的名称。该函数可以分配给另一个变量，仍然可以正常工作：

```js
let s = square;  // Now s refers to the same function that square does
square(4)        // => 16
s(4)             // => 16
```

函数也可以被分配给对象属性而不是变量。正如我们之前讨论过的，当我们这样做时，我们将这些函数称为“方法”：

```js
let o = {square: function(x) { return x*x; }}; // An object literal
let y = o.square(16);                          // y == 256
```

函数甚至不需要名称，比如当它们被分配给数组元素时：

```js
let a = [x => x*x, 20]; // An array literal
a0              // => 400
```

最后一个示例的语法看起来很奇怪，但仍然是一个合法的函数调用表达式！

作为将函数视为值的有用性的一个例子，考虑 `Array.sort()` 方法。该方法对数组的元素进行排序。由于有许多可能的排序顺序（数字顺序、字母顺序、日期顺序、升序、降序等），`sort()` 方法可以选择接受一个函数作为参数，告诉它如何执行排序。这个函数的工作很简单：对于传递给它的任何两个值，它返回一个指定哪个元素在排序后的数组中首先出现的值。这个函数参数使 `Array.sort()` 变得非常通用和无限灵活；它可以将任何类型的数据按照任何可想象的顺序进行排序。示例在 §7.8.6 中展示。

示例 8-1 展示了当函数被用作值时可以做的事情。这个例子可能有点棘手，但注释解释了发生了什么。

##### 示例 8-1。将函数用作数据

```js
// We define some simple functions here
function add(x,y) { return x + y; }
function subtract(x,y) { return x - y; }
function multiply(x,y) { return x * y; }
function divide(x,y) { return x / y; }

// Here's a function that takes one of the preceding functions
// as an argument and invokes it on two operands
function operate(operator, operand1, operand2) {
    return operator(operand1, operand2);
}

// We could invoke this function like this to compute the value (2+3) + (4*5):
let i = operate(add, operate(add, 2, 3), operate(multiply, 4, 5));

// For the sake of the example, we implement the simple functions again,
// this time within an object literal;
const operators = {
    add:      (x,y) => x+y,
    subtract: (x,y) => x-y,
    multiply: (x,y) => x*y,
    divide:   (x,y) => x/y,
    pow:      Math.pow  // This works for predefined functions too
};

// This function takes the name of an operator, looks up that operator
// in the object, and then invokes it on the supplied operands. Note
// the syntax used to invoke the operator function.
function operate2(operation, operand1, operand2) {
    if (typeof operators[operation] === "function") {
        return operatorsoperation;
    }
    else throw "unknown operator";
}

operate2("add", "hello", operate2("add", " ", "world")) // => "hello world"
operate2("pow", 10, 2)  // => 100
```

## 8.4.1 定义自己的函数属性

在 JavaScript 中，函数不是原始值，而是一种特殊的对象，这意味着函数可以有属性。当一个函数需要一个“静态”变量，其值在调用之间保持不变时，通常方便使用函数本身的属性。例如，假设你想编写一个函数，每次调用时都返回一个唯一的整数。该函数可能两次返回相同的值。为了管理这个问题，函数需要跟踪它已经返回的值，并且这个信息必须在函数调用之间保持不变。你可以将这个信息存储在一个全局变量中，但这是不必要的，因为这个信息只被函数本身使用。最好将信息存储在 Function 对象的属性中。下面是一个示例，每次调用时都返回一个唯一的整数： 

```js
// Initialize the counter property of the function object.
// Function declarations are hoisted so we really can
// do this assignment before the function declaration.
uniqueInteger.counter = 0;

// This function returns a different integer each time it is called.
// It uses a property of itself to remember the next value to be returned.
function uniqueInteger() {
    return uniqueInteger.counter++;  // Return and increment counter property
}
uniqueInteger()  // => 0
uniqueInteger()  // => 1
```

举个例子，考虑下面的`factorial()`函数，它利用自身的属性（将自身视为数组）来缓存先前计算的结果：

```js
// Compute factorials and cache results as properties of the function itself.
function factorial(n) {
    if (Number.isInteger(n) && n > 0) {           // Positive integers only
        if (!(n in factorial)) {                  // If no cached result
            factorial[n] = n * factorial(n-1);    // Compute and cache it
        }
        return factorial[n];                      // Return the cached result
    } else {
        return NaN;                               // If input was bad
    }
}
factorial[1] = 1;  // Initialize the cache to hold this base case.
factorial(6)  // => 720
factorial[5]  // => 120; the call above caches this value
```

# 8.5 函数作为命名空间

在函数内声明的变量在函数外部是不可见的。因此，有时候定义一个函数仅仅作为一个临时的命名空间是很有用的，你可以在其中定义变量而不会使全局命名空间混乱。

例如，假设你有一段 JavaScript 代码块，你想在许多不同的 JavaScript 程序中使用（或者对于客户端 JavaScript，在许多不同的网页上使用）。假设这段代码，像大多数代码一样，定义变量来存储计算的中间结果。问题在于，由于这段代码将在许多不同的程序中使用，你不知道它创建的变量是否会与使用它的程序创建的变量发生冲突。解决方案是将代码块放入一个函数中，然后调用该函数。这样，原本将是全局的变量变为函数的局部变量：

```js
function chunkNamespace() {
    // Chunk of code goes here
    // Any variables defined in the chunk are local to this function
    // instead of cluttering up the global namespace.
}
chunkNamespace();  // But don't forget to invoke the function!
```

这段代码只定义了一个全局变量：函数名`chunkNamespace`。如果即使定义一个属性也太多了，你可以在单个表达式中定义并调用一个匿名函数：

```js
(function() {  // chunkNamespace() function rewritten as an unnamed expression.
    // Chunk of code goes here
}());          // End the function literal and invoke it now.
```

定义和调用一个函数的单个表达式的技术经常被使用，已经成为惯用语，并被称为“立即调用函数表达式”。请注意前面代码示例中括号的使用。在`function`之前的开括号是必需的，因为没有它，JavaScript 解释器会尝试将`function`关键字解析为函数声明语句。有了括号，解释器正确地将其识别为函数定义表达式。前导括号还有助于人类读者识别何时定义一个函数以立即调用，而不是为以后使用而定义。

当我们在命名空间函数内部定义一个或多个函数，并使用该命名空间内的变量，然后将它们作为命名空间函数的返回值传递出去时，函数作为命名空间的用法变得非常有用。这样的函数被称为*闭包*，它们是下一节的主题。

# 8.6 闭包

像大多数现代编程语言一样，JavaScript 使用*词法作用域*。这意味着函数在定义时使用的变量作用域，而不是在调用时使用的变量作用域。为了实现词法作用域，JavaScript 函数对象的内部状态必须包括函数的代码以及函数定义所在的作用域的引用。在计算机科学文献中，函数对象和作用域（一组变量绑定）的组合，用于解析函数变量的作用域，被称为*闭包*。

从技术上讲，所有的 JavaScript 函数都是闭包，但由于大多数函数是从定义它们的同一作用域中调用的，通常并不重要闭包是否涉及其中。当闭包从与其定义所在不同的作用域中调用时，闭包就变得有趣起来。这种情况最常见于从定义它的函数中返回嵌套函数对象时。有许多强大的编程技术涉及到这种嵌套函数闭包，它们在 JavaScript 编程中的使用变得相对常见。当你第一次遇到闭包时，它们可能看起来令人困惑，但重要的是你要足够了解它们以便舒适地使用它们。

理解闭包的第一步是复习嵌套函数的词法作用域规则。考虑以下代码：

```js
let scope = "global scope";          // A global variable
function checkscope() {
    let scope = "local scope";       // A local variable
    function f() { return scope; }   // Return the value in scope here
    return f();
}
checkscope()                         // => "local scope"
```

`checkscope()`函数声明了一个局部变量，然后定义并调用一个返回该变量值的函数。你应该清楚为什么调用`checkscope()`会返回“local scope”。现在，让我们稍微改变一下代码。你能告诉这段代码会返回什么吗？

```js
let scope = "global scope";          // A global variable
function checkscope() {
    let scope = "local scope";       // A local variable
    function f() { return scope; }   // Return the value in scope here
    return f;
}
let s = checkscope()();              // What does this return?
```

在这段代码中，一对括号已经从`checkscope()`内部移到了外部。现在，`checkscope()`不再调用嵌套函数并返回其结果，而是直接返回嵌套函数对象本身。当我们在定义它的函数之外调用该嵌套函数（在代码的最后一行中的第二对括号中）时会发生什么？

记住词法作用域的基本规则：JavaScript 函数是在定义它们的作用域中执行的。嵌套函数`f()`是在一个作用域中定义的，该作用域中变量`scope`绑定到值“local scope”。当执行`f`时，这个绑定仍然有效，无论从哪里执行。因此，前面代码示例的最后一行返回“local scope”，而不是“global scope”。这就是闭包的令人惊讶和强大的本质：它们捕获了它们所定义的外部函数的局部变量（和参数）绑定。

在§8.4.1 中，我们定义了一个`uniqueInteger()`函数，该函数使用函数本身的属性来跟踪下一个要返回的值。这种方法的一个缺点是，有错误或恶意代码可能会重置计数器或将其设置为非整数，导致`uniqueInteger()`函数违反其“unique”或“integer”部分的约定。闭包捕获了单个函数调用的局部变量，并可以将这些变量用作私有状态。下面是我们如何使用立即调用函数表达式来重新编写`uniqueInteger()`，以定义一个命名空间和使用该命名空间来保持其状态私有的闭包：

```js
let uniqueInteger = (function() {  // Define and invoke
    let counter = 0;               // Private state of function below
    return function() { return counter++; };
}());
uniqueInteger()  // => 0
uniqueInteger()  // => 1
```

要理解这段代码，你必须仔细阅读它。乍一看，代码的第一行看起来像是将一个函数赋给变量`uniqueInteger`。实际上，代码正在定义并调用一个函数（第一行的开括号提示了这一点），因此将函数的返回值赋给了`uniqueInteger`。现在，如果我们研究函数体，我们会发现它的返回值是另一个函数。正是这个嵌套函数对象被赋给了`uniqueInteger`。嵌套函数可以访问其作用域中的变量，并且可以使用外部函数中定义的`counter`变量。一旦外部函数返回，其他代码就无法看到`counter`变量：内部函数对其具有独占访问权限。

像`counter`这样的私有变量不一定是单个闭包的专有：完全可以在同一个外部函数中定义两个或更多个嵌套函数并共享相同的作用域。考虑以下代码：

```js
function counter() {
    let n = 0;
    return {
        count: function() { return n++; },
        reset: function() { n = 0; }
    };
}

let c = counter(), d = counter();   // Create two counters
c.count()                           // => 0
d.count()                           // => 0: they count independently
c.reset();                          // reset() and count() methods share state
c.count()                           // => 0: because we reset c
d.count()                           // => 1: d was not reset
```

`counter()`函数返回一个“计数器”对象。这个对象有两个方法：`count()`返回下一个整数，`reset()`重置内部状态。首先要理解的是，这两个方法共享对私有变量`n`的访问。其次要理解的是，每次调用`counter()`都会创建一个新的作用域——独立于先前调用使用的作用域，并在该作用域内创建一个新的私有变量。因此，如果您两次调用`counter()`，您将得到两个具有不同私有变量的计数器对象。在一个计数器对象上调用`count()`或`reset()`对另一个没有影响。

值得注意的是，您可以将闭包技术与属性的 getter 和 setter 结合使用。下面这个`counter()`函数的版本是§6.10.6 中出现的代码的变体，但它使用闭包来实现私有状态，而不是依赖于常规对象属性：

```js
function counter(n) {  // Function argument n is the private variable
    return {
        // Property getter method returns and increments private counter var.
        get count() { return n++; },
        // Property setter doesn't allow the value of n to decrease
        set count(m) {
            if (m > n) n = m;
            else throw Error("count can only be set to a larger value");
        }
    };
}

let c = counter(1000);
c.count            // => 1000
c.count            // => 1001
c.count = 2000;
c.count            // => 2000
c.count = 2000;    // !Error: count can only be set to a larger value
```

注意，这个`counter()`函数的版本并没有声明一个局部变量，而是只是使用其参数`n`来保存属性访问方法共享的私有状态。这允许`counter()`的调用者指定私有变量的初始值。

示例 8-2 是通过我们一直在演示的闭包技术对共享私有状态进行泛化的一个例子。这个示例定义了一个`addPrivateProperty()`函数，该函数定义了一个私有变量和两个嵌套函数来获取和设置该变量的值。它将这些嵌套函数作为您指定对象的方法添加。

##### 示例 8-2\. 使用闭包的私有属性访问方法

```js
// This function adds property accessor methods for a property with
// the specified name to the object o. The methods are named get<name>
// and set<name>. If a predicate function is supplied, the setter
// method uses it to test its argument for validity before storing it.
// If the predicate returns false, the setter method throws an exception.
//
// The unusual thing about this function is that the property value
// that is manipulated by the getter and setter methods is not stored in
// the object o. Instead, the value is stored only in a local variable
// in this function. The getter and setter methods are also defined
// locally to this function and therefore have access to this local variable.
// This means that the value is private to the two accessor methods, and it
// cannot be set or modified except through the setter method.
function addPrivateProperty(o, name, predicate) {
    let value;  // This is the property value

    // The getter method simply returns the value.
    o[`get${name}`] = function() { return value; };

    // The setter method stores the value or throws an exception if
    // the predicate rejects the value.
    o[`set${name}`] = function(v) {
        if (predicate && !predicate(v)) {
            throw new TypeError(`set${name}: invalid value ${v}`);
        } else {
            value = v;
        }
    };
}

// The following code demonstrates the addPrivateProperty() method.
let o = {};  // Here is an empty object

// Add property accessor methods getName and setName()
// Ensure that only string values are allowed
addPrivateProperty(o, "Name", x => typeof x === "string");

o.setName("Frank");       // Set the property value
o.getName()               // => "Frank"
o.setName(0);             // !TypeError: try to set a value of the wrong type
```

现在我们已经看到了许多例子，其中两个闭包在同一个作用域中定义并共享对相同私有变量或变量的访问。这是一个重要的技术，但同样重要的是要认识到闭包无意中共享对不应共享的变量的访问。考虑以下代码：

```js
// This function returns a function that always returns v
function constfunc(v) { return () => v; }

// Create an array of constant functions:
let funcs = [];
for(var i = 0; i < 10; i++) funcs[i] = constfunc(i);

// The function at array element 5 returns the value 5.
funcs[5]()    // => 5
```

在处理像这样使用循环创建多个闭包的代码时，一个常见的错误是尝试将循环移到定义闭包的函数内部。例如，考虑以下代码：

```js
// Return an array of functions that return the values 0-9
function constfuncs() {
    let funcs = [];
    for(var i = 0; i < 10; i++) {
        funcs[i] = () => i;
    }
    return funcs;
}

let funcs = constfuncs();
funcs[5]()    // => 10; Why doesn't this return 5?
```

这段代码创建了 10 个闭包并将它们存储在一个数组中。这些闭包都在同一个函数调用中定义，因此它们共享对变量`i`的访问。当`constfuncs()`返回时，变量`i`的值为 10，所有 10 个闭包都共享这个值。因此，返回的函数数组中的所有函数都返回相同的值，这并不是我们想要的。重要的是要记住，与闭包相关联的作用域是“活动的”。嵌套函数不会创建作用域的私有副本，也不会对变量绑定进行静态快照。从根本上说，这里的问题是使用`var`声明的变量在整个函数中都被定义。我们的`for`循环使用`var i`声明循环变量，因此变量`i`在整个函数中被定义，而不是更窄地限制在循环体内。这段代码展示了 ES5 及之前版本中常见的一类错误，但 ES6 引入的块作用域变量解决了这个问题。如果我们只是用`let`或`const`替换`var`，问题就消失了。因为`let`和`const`是块作用域的，循环的每次迭代都定义了一个独立于所有其他迭代的作用域，并且每个作用域都有自己独立的`i`绑定。

写闭包时要记住的另一件事是，`this`是 JavaScript 关键字，而不是变量。正如前面讨论的，箭头函数继承了包含它们的函数的`this`值，但使用`function`关键字定义的函数不会。因此，如果您编写一个需要使用其包含函数的`this`值的闭包，您应该在返回之前使用箭头函数或调用`bind()`，或将外部`this`值分配给闭包将继承的变量：

```js
const self = this;  // Make the this value available to nested functions
```

# 8.7 函数属性、方法和构造函数

我们已经看到函数在 JavaScript 程序中是值。当应用于函数时，`typeof`运算符返回字符串“function”，但函数实际上是 JavaScript 对象的一种特殊类型。由于函数是对象，它们可以像任何其他对象一样具有属性和方法。甚至有一个`Function()`构造函数来创建新的函数对象。接下来的小节记录了`length`、`name`和`prototype`属性；`call()`、`apply()`、`bind()`和`toString()`方法；以及`Function()`构造函数。

## 8.7.1 length 属性

函数的只读`length`属性指定函数的*arity*——它在参数列表中声明的参数数量，通常是函数期望的参数数量。如果函数有一个剩余参数，那么这个参数不会计入`length`属性的目的。

## 8.7.2 名称属性

函数的只读`name`属性指定函数在定义时使用的名称，如果它是用名称定义的，或者在创建时未命名的函数表达式被分配给的变量或属性的名称。当编写调试或错误消息时，此属性非常有用。

## 8.7.3 prototype 属性

所有函数，除了箭头函数，都有一个`prototype`属性，指向一个称为*原型对象*的对象。每个函数都有一个不同的原型对象。当一个函数被用作构造函数时，新创建的对象会从原型对象继承属性。原型和`prototype`属性在§6.2.3 中讨论过，并将在第九章中再次涉及。

## 8.7.4 call()和 apply()方法

`call()`和`apply()`允许您间接调用（§8.2.4）一个函数，就好像它是另一个对象的方法一样。`call()`和`apply()`的第一个参数是要调用函数的对象；这个参数是调用上下文，并在函数体内成为`this`关键字的值。要将函数`f()`作为对象`o`的方法调用（不传递参数），可以使用`call()`或`apply()`：

```js
f.call(o);
f.apply(o);
```

这两行代码中的任何一行与以下代码类似（假设`o`尚未具有名为`m`的属性）：

```js
o.m = f;     // Make f a temporary method of o.
o.m();       // Invoke it, passing no arguments.
delete o.m;  // Remove the temporary method.
```

请记住，箭头函数继承了定义它们的上下文的`this`值。这不能通过`call()`和`apply()`方法覆盖。如果在箭头函数上调用这些方法之一，第一个参数实际上会被忽略。

在第一个调用上下文参数之后的任何`call()`参数都是传递给被调用函数的值（对于箭头函数，这些参数不会被忽略）。例如，要向函数`f()`传递两个数字，并将其作为对象`o`的方法调用，可以使用以下代码：

```js
f.call(o, 1, 2);
```

`apply()`方法类似于`call()`方法，只是要传递给函数的参数被指定为一个数组：

```js
f.apply(o, [1,2]);
```

如果一个函数被定义为接受任意数量的参数，`apply()` 方法允许你在任意长度的数组内容上调用该函数。在 ES6 及更高版本中，我们可以直接使用扩展运算符，但你可能会看到使用 `apply()` 而不是扩展运算符的 ES5 代码。例如，要在不使用扩展运算符的情况下找到数组中的最大数，你可以使用 `apply()` 方法将数组的元素传递给 `Math.max()` 函数：

```js
let biggest = Math.max.apply(Math, arrayOfNumbers);
```

下面定义的 `trace()` 函数类似于 §8.3.4 中定义的 `timed()` 函数，但它适用于方法而不是函数。它使用 `apply()` 方法而不是扩展运算符，通过这样做，它能够以与包装方法相同的参数和 `this` 值调用被包装的方法：

```js
// Replace the method named m of the object o with a version that logs
// messages before and after invoking the original method.
function trace(o, m) {
    let original = o[m];         // Remember original method in the closure.
    o[m] = function(...args) {   // Now define the new method.
        console.log(new Date(), "Entering:", m);      // Log message.
        let result = original.apply(this, args);      // Invoke original.
        console.log(new Date(), "Exiting:", m);       // Log message.
        return result;                                // Return result.
    };
}
```

## 8.7.5 `bind()` 方法

`bind()` 的主要目的是将函数*绑定*到对象。当你在函数 `f` 上调用 `bind()` 方法并传递一个对象 `o` 时，该方法会返回一个新函数。调用新函数（作为函数）会将原始函数 `f` 作为 `o` 的方法调用。传递给新函数的任何参数都会传递给原始函数。例如：

```js
function f(y) { return this.x + y; } // This function needs to be bound
let o = { x: 1 };                    // An object we'll bind to
let g = f.bind(o);                   // Calling g(x) invokes f() on o
g(2)                                 // => 3
let p = { x: 10, g };                // Invoke g() as a method of this object
p.g(2)                               // => 3: g is still bound to o, not p.
```

箭头函数从定义它们的环境继承它们的 `this` 值，并且该值不能被 `bind()` 覆盖，因此如果前面代码中的函数 `f()` 被定义为箭头函数，绑定将不起作用。然而，调用 `bind()` 最常见的用例是使非箭头函数的行为类似箭头函数，因此在实践中，对绑定箭头函数的限制并不是问题。

`bind()` 方法不仅仅是将函数绑定到对象，它还可以执行部分应用：在第一个参数之后传递给 `bind()` 的任何参数都与 `this` 值一起绑定。`bind()` 的这种部分应用特性适用于箭头函数。部分应用是函数式编程中的常见技术，有时被称为*柯里化*。以下是 `bind()` 方法用于部分应用的一些示例：

```js
let sum = (x,y) => x + y;      // Return the sum of 2 args
let succ = sum.bind(null, 1);  // Bind the first argument to 1
succ(2)  // => 3: x is bound to 1, and we pass 2 for the y argument

function f(y,z) { return this.x + y + z; }
let g = f.bind({x: 1}, 2);     // Bind this and y
g(3)     // => 6: this.x is bound to 1, y is bound to 2 and z is 3
```

由 `bind()` 返回的函数的 `name` 属性是调用 `bind()` 的函数的名称属性，前缀为“bound”。

## 8.7.6 `toString()` 方法

像所有 JavaScript 对象一样，函数有一个 `toString()` 方法。ECMAScript 规范要求该方法返回一个遵循函数声明语法的字符串。实际上，大多数（但不是所有）实现这个 `toString()` 方法的实现会返回函数的完整源代码。内置函数通常返回一个包含类似“[native code]”的字符串作为函数体的字符串。

## 8.7.7 `Function()` 构造函数

因为函数是对象，所以有一个 `Function()` 构造函数可用于创建新函数：

```js
const f = new Function("x", "y", "return x*y;");
```

这行代码创建了一个新函数，它与使用熟悉语法定义的函数更或多少等效：

```js
const f = function(x, y) { return x*y; };
```

`Function()` 构造函数期望任意数量的字符串参数。最后一个参数是函数体的文本；它可以包含任意 JavaScript 语句，用分号分隔。构造函数的所有其他参数都是指定函数参数名称的字符串。如果你定义一个不带参数的函数，你只需将一个字符串（函数体）传递给构造函数。

注意 `Function()` 构造函数没有传递任何指定创建的函数名称的参数。与函数字面量一样，`Function()` 构造函数创建匿名函数。

有几点很重要需要了解关于 `Function()` 构造函数：

+   `Function()` 构造函数允许在运行时动态创建和编译 JavaScript 函数。

+   `Function()`构造函数解析函数体并在每次调用时创建一个新的函数对象。如果构造函数的调用出现在循环中或在频繁调用的函数内部，这个过程可能效率低下。相比之下，在循环中出现的嵌套函数和函数表达式在遇到时不会重新编译。

+   关于`Function()`构造函数的最后一个非常重要的观点是，它创建的函数不使用词法作用域；相反，它们总是被编译为顶级函数，如下面的代码所示：

    ```js
    let scope = "global";
    function constructFunction() {
        let scope = "local";
        return new Function("return scope");  // Doesn't capture local scope!
    }
    // This line returns "global" because the function returned by the
    // Function() constructor does not use the local scope.
    constructFunction()()  // => "global"
    ```

`Function()`构造函数最好被视为`eval()`的全局作用域版本（参见§4.12.2），它在自己的私有作用域中定义新的变量和函数。你可能永远不需要在你的代码中使用这个构造函数。

# 8.8 函数式编程

JavaScript 不像 Lisp 或 Haskell 那样是一种函数式编程语言，但 JavaScript 可以将函数作为对象进行操作的事实意味着我们可以在 JavaScript 中使用函数式编程技术。数组方法如`map()`和`reduce()`特别适合函数式编程风格。接下来的部分演示了 JavaScript 中函数式编程的技术。它们旨在探索 JavaScript 函数的强大功能，而不是规范良好的编程风格。

## 8.8.1 使用函数处理数组

假设我们有一个数字数组，我们想要计算这些值的均值和标准差。我们可以像这样以非函数式的方式进行：

```js
let data = [1,1,3,5,5];  // This is our array of numbers

// The mean is the sum of the elements divided by the number of elements
let total = 0;
for(let i = 0; i < data.length; i++) total += data[i];
let mean = total/data.length;  // mean == 3; The mean of our data is 3

// To compute the standard deviation, we first sum the squares of
// the deviation of each element from the mean.
total = 0;
for(let i = 0; i < data.length; i++) {
    let deviation = data[i] - mean;
    total += deviation * deviation;
}
let stddev = Math.sqrt(total/(data.length-1));  // stddev == 2
```

我们可以使用数组方法`map()`和`reduce()`以简洁的函数式风格执行相同的计算，如下所示（参见§7.8.1 回顾这些方法）：

```js
// First, define two simple functions
const sum = (x,y) => x+y;
const square = x => x*x;

// Then use those functions with Array methods to compute mean and stddev
let data = [1,1,3,5,5];
let mean = data.reduce(sum)/data.length;  // mean == 3
let deviations = data.map(x => x-mean);
let stddev = Math.sqrt(deviations.map(square).reduce(sum)/(data.length-1));
stddev  // => 2
```

这个新版本的代码看起来与第一个版本非常不同，但仍然在对象上调用方法，因此仍然保留了一些面向对象的约定。让我们编写`map()`和`reduce()`方法的函数式版本：

```js
const map = function(a, ...args) { return a.map(...args); };
const reduce = function(a, ...args) { return a.reduce(...args); };
```

有了这些定义的`map()`和`reduce()`函数，我们现在计算均值和标准差的代码如下：

```js
const sum = (x,y) => x+y;
const square = x => x*x;

let data = [1,1,3,5,5];
let mean = reduce(data, sum)/data.length;
let deviations = map(data, x => x-mean);
let stddev = Math.sqrt(reduce(map(deviations, square), sum)/(data.length-1));
stddev  // => 2
```

## 8.8.2 高阶函数

*高阶函数*是一个操作函数的函数，它接受一个或多个函数作为参数并返回一个新函数。这里有一个例子：

```js
// This higher-order function returns a new function that passes its
// arguments to f and returns the logical negation of f's return value;
function not(f) {
    return function(...args) {             // Return a new function
        let result = f.apply(this, args);  // that calls f
        return !result;                    // and negates its result.
    };
}

const even = x => x % 2 === 0; // A function to determine if a number is even
const odd = not(even);         // A new function that does the opposite
[1,1,3,5,5].every(odd)         // => true: every element of the array is odd
```

这个`not()`函数是一个高阶函数，因为它接受一个函数参数并返回一个新函数。再举一个例子，考虑接下来的`mapper()`函数。它接受一个函数参数并返回一个使用该函数将一个数组映射到另一个数组的新函数。这个函数使用了之前定义的`map()`函数，你需要理解这两个函数的不同之处很重要：

```js
// Return a function that expects an array argument and applies f to
// each element, returning the array of return values.
// Contrast this with the map() function from earlier.
function mapper(f) {
    return a => map(a, f);
}

const increment = x => x+1;
const incrementAll = mapper(increment);
incrementAll([1,2,3])  // => [2,3,4]
```

这里是另一个更一般的例子，它接受两个函数`f`和`g`，并返回一个计算`f(g())`的新函数：

```js
// Return a new function that computes f(g(...)).
// The returned function h passes all of its arguments to g, then passes
// the return value of g to f, then returns the return value of f.
// Both f and g are invoked with the same this value as h was invoked with.
function compose(f, g) {
    return function(...args) {
        // We use call for f because we're passing a single value and
        // apply for g because we're passing an array of values.
        return f.call(this, g.apply(this, args));
    };
}

const sum = (x,y) => x+y;
const square = x => x*x;
compose(square, sum)(2,3)  // => 25; the square of the sum
```

在接下来的部分中定义的`partial()`和`memoize()`函数是另外两个重要的高阶函数。

## 8.8.3 函数的部分应用

函数`f`的`bind()`方法（参见§8.7.5）返回一个在指定上下文中调用`f`并带有指定参数集的新函数。我们说它将函数绑定到一个对象并部分应用参数。`bind()`方法在左侧部分应用参数，也就是说，你传递给`bind()`的参数被放在传递给原始函数的参数列表的开头。但也可以在右侧部分应用参数：

```js
// The arguments to this function are passed on the left
function partialLeft(f, ...outerArgs) {
    return function(...innerArgs) { // Return this function
        let args = [...outerArgs, ...innerArgs]; // Build the argument list
        return f.apply(this, args);              // Then invoke f with it
    };
}

// The arguments to this function are passed on the right
function partialRight(f, ...outerArgs) {
    return function(...innerArgs) {  // Return this function
        let args = [...innerArgs, ...outerArgs]; // Build the argument list
        return f.apply(this, args);              // Then invoke f with it
    };
}

// The arguments to this function serve as a template. Undefined values
// in the argument list are filled in with values from the inner set.
function partial(f, ...outerArgs) {
    return function(...innerArgs) {
        let args = [...outerArgs]; // local copy of outer args template
        let innerIndex=0;          // which inner arg is next
        // Loop through the args, filling in undefined values from inner args
        for(let i = 0; i < args.length; i++) {
            if (args[i] === undefined) args[i] = innerArgs[innerIndex++];
        }
        // Now append any remaining inner arguments
        args.push(...innerArgs.slice(innerIndex));
        return f.apply(this, args);
    };
}

// Here is a function with three arguments
const f = function(x,y,z) { return x * (y - z); };
// Notice how these three partial applications differ
partialLeft(f, 2)(3,4)         // => -2: Bind first argument: 2 * (3 - 4)
partialRight(f, 2)(3,4)        // =>  6: Bind last argument: 3 * (4 - 2)
partial(f, undefined, 2)(3,4)  // => -6: Bind middle argument: 3 * (2 - 4)
```

这些部分应用函数使我们能够轻松地从已定义的函数中定义有趣的函数。以下是一些示例：

```js
const increment = partialLeft(sum, 1);
const cuberoot = partialRight(Math.pow, 1/3);
cuberoot(increment(26))  // => 3
```

当我们将部分应用与其他高阶函数结合时，部分应用变得更加有趣。例如，以下是使用组合和部分应用定义前面刚刚展示的`not()`函数的一种方法：

```js
const not = partialLeft(compose, x => !x);
const even = x => x % 2 === 0;
const odd = not(even);
const isNumber = not(isNaN);
odd(3) && isNumber(2)  // => true
```

我们还可以使用组合和部分应用来以极端函数式风格重新执行我们的均值和标准差计算：

```js
// sum() and square() functions are defined above. Here are some more:
const product = (x,y) => x*y;
const neg = partial(product, -1);
const sqrt = partial(Math.pow, undefined, .5);
const reciprocal = partial(Math.pow, undefined, neg(1));

// Now compute the mean and standard deviation.
let data = [1,1,3,5,5];   // Our data
let mean = product(reduce(data, sum), reciprocal(data.length));
let stddev = sqrt(product(reduce(map(data,
                                     compose(square,
                                             partial(sum, neg(mean)))),
                                 sum),
                          reciprocal(sum(data.length,neg(1)))));
[mean, stddev]  // => [3, 2]
```

请注意，这段用于计算均值和标准差的代码完全是函数调用；没有涉及运算符，并且括号的数量已经变得如此之多，以至于这段 JavaScript 代码开始看起来像 Lisp 代码。再次强调，这不是我推崇的 JavaScript 编程风格，但看到 JavaScript 代码可以有多函数式是一个有趣的练习。

## 8.8.4 Memoization

在§8.4.1 中，我们定义了一个阶乘函数，它缓存了先前计算的结果。在函数式编程中，这种缓存称为*memoization*。接下来的代码展示了一个高阶函数，`memoize()`，它接受一个函数作为参数，并返回该函数的一个记忆化版本：

```js
// Return a memoized version of f.
// It only works if arguments to f all have distinct string representations.
function memoize(f) {
    const cache = new Map();  // Value cache stored in the closure.

    return function(...args) {
        // Create a string version of the arguments to use as a cache key.
        let key = args.length + args.join("+");
        if (cache.has(key)) {
            return cache.get(key);
        } else {
            let result = f.apply(this, args);
            cache.set(key, result);
            return result;
        }
    };
}
```

`memoize()`函数创建一个新对象用作缓存，并将此对象分配给一个局部变量，以便它对（在返回的函数的闭包中）是私有的。返回的函数将其参数数组转换为字符串，并将该字符串用作缓存对象的属性名。如果缓存中存在值，则直接返回它。否则，调用指定的函数来计算这些参数的值，缓存该值，并返回它。以下是我们如何使用`memoize()`：

```js
// Return the Greatest Common Divisor of two integers using the Euclidian
// algorithm: http://en.wikipedia.org/wiki/Euclidean_algorithm
function gcd(a,b) {  // Type checking for a and b has been omitted
    if (a < b) {           // Ensure that a >= b when we start
        [a, b] = [b, a];   // Destructuring assignment to swap variables
    }
    while(b !== 0) {       // This is Euclid's algorithm for GCD
        [a, b] = [b, a%b];
    }
    return a;
}

const gcdmemo = memoize(gcd);
gcdmemo(85, 187)  // => 17

// Note that when we write a recursive function that we will be memoizing,
// we typically want to recurse to the memoized version, not the original.
const factorial = memoize(function(n) {
    return (n <= 1) ? 1 : n * factorial(n-1);
});
factorial(5)      // => 120: also caches values for 4, 3, 2 and 1.
```

# 8.9 总结

关于本章的一些关键要点如下：

+   您可以使用`function`关键字和 ES6 的`=>`箭头语法定义函数。

+   您可以调用函数，这些函数可以用作方法和构造函数。

+   一些 ES6 功能允许您为可选函数参数定义默认值，使用 rest 参数将多个参数收集到一个数组中，并将对象和数组参数解构为函数参数。

+   您可以使用`...`扩展运算符将数组或其他可迭代对象的元素作为参数传递给函数调用。

+   在封闭函数内部定义并返回的函数保留对其词法作用域的访问权限，因此可以读取和写入外部函数中定义的变量。以这种方式使用的函数称为*closures*，这是一种值得理解的技术。

+   函数是 JavaScript 可以操作的对象，这使得函数式编程成为可能。

¹ 这个术语是由 Martin Fowler 创造的。参见[*http://martinfowler.com/dslCatalog/methodChaining.html*](http://martinfowler.com/dslCatalog/methodChaining.html)。

² 如果你熟悉 Python，注意这与 Python 不同，其中每次调用都共享相同的默认值。

³ 这可能看起来不是特别有趣，除非您熟悉更静态的语言，在这些语言中，函数是程序的一部分，但不能被程序操纵。
