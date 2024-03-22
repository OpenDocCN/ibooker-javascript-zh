# 第十四章：元编程

本章介绍了一些高级 JavaScript 功能，这些功能在日常编程中并不常用，但对于编写可重用库的程序员可能很有价值，并且对于任何想要深入了解 JavaScript 对象行为细节的人也很有趣。

这里描述的许多功能可以宽泛地描述为“元编程”：如果常规编程是编写代码来操作数据，那么元编程就是编写代码来操作其他代码。在像 JavaScript 这样的动态语言中，编程和元编程之间的界限模糊——甚至简单地使用`for/in`循环迭代对象的属性的能力对更习惯于更静态语言的程序员来说可能被认为是“元编程”。

本章涵盖的元编程主题包括：

+   §14.1 控制对象属性的可枚举性、可删除性和可配置性

+   §14.2 控制对象的可扩展性，并创建“封闭”和“冻结”对象

+   §14.3 查询和设置对象的原型

+   §14.4 使用众所周知的符号微调类型的行为

+   §14.5 使用模板标签函数创建 DSL（领域特定语言）

+   §14.6 使用`reflect`方法探查对象

+   §14.7 使用代理控制对象行为

# 14.1 属性特性

JavaScript 对象的属性当然有名称和值，但每个属性还有三个关联属性，指定该属性的行为方式以及您可以对其执行的操作：

+   *可写* 属性指定属性的值是否可以更改。

+   *可枚举* 属性指定属性是否由`for/in`循环和`Object.keys()`方法枚举。

+   *可配置* 属性指定属性是否可以被删除，以及属性的属性是否可以更改。

在对象字面量中定义的属性或通过普通赋值给对象的属性是可写的、可枚举的和可配置的。但是，JavaScript 标准库中定义的许多属性并非如此。

本节解释了查询和设置属性特性的 API。这个 API 对于库作者尤为重要，因为：

+   它允许他们向原型对象添加方法并使它们不可枚举，就像内置方法一样。

+   它允许它们“锁定”它们的对象，定义不能被更改或删除的属性。

请回顾§6.10.6，在那里提到，“数据属性”具有值，“访问器属性”则具有 getter 和/或 setter 方法。对于本节的目的，我们将考虑访问器属性的 getter 和 setter 方法为属性特性。按照这种逻辑，我们甚至会说数据属性的值也是一个属性。因此，我们可以说属性有一个名称和四个属性。数据属性的四个属性是*值*、*可写*、*可枚举*和*可配置*。访问器属性没有*值*属性或*可写*属性：它们的可写性取决于是否存在 setter。因此，访问器属性的四个属性是*获取*、*设置*、*可枚举*和*可配置*。

JavaScript 用于查询和设置属性的方法使用一个称为*属性描述符*的对象来表示四个属性的集合。属性描述符对象具有与其描述的属性相同名称的属性。因此，数据属性的属性描述符对象具有名为`value`、`writable`、`enumerable`和`configurable`的属性。访问器属性的描述符具有`get`和`set`属性，而不是`value`和`writable`。`writable`、`enumerable`和`configurable`属性是布尔值，`get`和`set`属性是函数值。

要获取指定对象的命名属性的属性描述符，请调用`Object.getOwnPropertyDescriptor()`：

```js
// Returns {value: 1, writable:true, enumerable:true, configurable:true}
Object.getOwnPropertyDescriptor({x: 1}, "x");

// Here is an object with a read-only accessor property
const random = {
    get octet() { return Math.floor(Math.random()*256); },
};

// Returns { get: /*func*/, set:undefined, enumerable:true, configurable:true}
Object.getOwnPropertyDescriptor(random, "octet");

// Returns undefined for inherited properties and properties that don't exist.
Object.getOwnPropertyDescriptor({}, "x")        // => undefined; no such prop
Object.getOwnPropertyDescriptor({}, "toString") // => undefined; inherited
```

如其名称所示，`Object.getOwnPropertyDescriptor()`仅适用于自有属性。要查询继承属性的属性，必须显式遍历原型链。（参见§14.3 中的`Object.getPrototypeOf()`）；另请参阅§14.6 中的类似`Reflect.getOwnPropertyDescriptor()`函数。

要设置属性的属性或使用指定属性创建新属性，请调用`Object.defineProperty()`，传递要修改的对象、要创建或更改的属性的名称和属性描述符对象：

```js
let o = {};  // Start with no properties at all
// Add a non-enumerable data property x with value 1.
Object.defineProperty(o, "x", {
    value: 1,
    writable: true,
    enumerable: false,
    configurable: true
});

// Check that the property is there but is non-enumerable
o.x            // => 1
Object.keys(o) // => []

// Now modify the property x so that it is read-only
Object.defineProperty(o, "x", { writable: false });

// Try to change the value of the property
o.x = 2;      // Fails silently or throws TypeError in strict mode
o.x           // => 1

// The property is still configurable, so we can change its value like this:
Object.defineProperty(o, "x", { value: 2 });
o.x           // => 2

// Now change x from a data property to an accessor property
Object.defineProperty(o, "x", { get: function() { return 0; } });
o.x           // => 0
```

你传递给`Object.defineProperty()`的属性描述符不必包含所有四个属性。如果你正在创建一个新属性，那么被省略的属性被视为`false`或`undefined`。如果你正在修改一个现有属性，那么你省略的属性将保持不变。请注意，此方法会更改现有的自有属性或创建新的自有属性，但不会更改继承的属性。另请参阅§14.6 中的非常相似的`Reflect.defineProperty()`函数。

如果要一次创建或修改多个属性，请使用`Object.defineProperties()`。第一个参数是要修改的对象。第二个参数是将要创建或修改的属性的名称映射到这些属性的属性描述符的对象。例如：

```js
let p = Object.defineProperties({}, {
    x: { value: 1, writable: true, enumerable: true, configurable: true },
    y: { value: 1, writable: true, enumerable: true, configurable: true },
    r: {
        get() { return Math.sqrt(this.x*this.x + this.y*this.y); },
        enumerable: true,
        configurable: true
    }
});
p.r  // => Math.SQRT2
```

这段代码从一个空对象开始，然后向其添加两个数据属性和一个只读访问器属性。它依赖于`Object.defineProperties()`返回修改后的对象（`Object.defineProperty()`也是如此）。

`Object.create()` 方法是在§6.2 中引入的。我们在那里学到，该方法的第一个参数是新创建对象的原型对象。该方法还接受第二个可选参数，与`Object.defineProperties()`的第二个参数相同。如果你向`Object.create()`传递一组属性描述符，那么它们将用于向新创建的对象添加属性。

如果尝试创建或修改属性不被允许，`Object.defineProperty()`和`Object.defineProperties()`会抛出 TypeError。如果你尝试向不可扩展的对象添加新属性，就会发生这种情况（参见§14.2）。这些方法可能抛出 TypeError 的其他原因与属性本身有关。*可写*属性控制对*值*属性的更改尝试。*可配置*属性控制对其他属性的更改尝试（并指定属性是否可以被删除）。然而，规则并不完全直观。例如，如果属性是可配置的，那么即使该属性是不可写的，也可以更改该属性的值。此外，即使属性是不可配置的，也可以将属性从可写更改为不可写。以下是完整的规则。调用`Object.defineProperty()`或`Object.defineProperties()`尝试违反这些规则会抛出 TypeError：

+   如果一个对象不可扩展，你可以编辑其现有的自有属性，但不能向其添加新属性。

+   如果一个属性不可配置，你就无法改变它的可配置或可枚举属性。

+   如果一个访问器属性不可配置，你就无法更改其 getter 或 setter 方法，也无法将其更改为数据属性。

+   如果一个数据属性不可配置，你就无法将其更改为访问器属性。

+   如果一个数据属性不可配置，你就无法将其*可写*属性从`false`更改为`true`，但你可以将其从`true`更改为`false`。

+   如果一个数据属性不可配置且不可写，你就无法改变它的值。但是，如果一个属性是可配置但不可写的，你可以改变它的值（因为这与使其可写，然后改变值，然后将其转换回不可写是一样的）。

§6.7 描述了`Object.assign()`函数，它将一个或多个源对象的属性值复制到目标对象中。`Object.assign()`只复制可枚举属性和属性值，而不是属性属性。这通常是我们想要的，但这意味着，例如，如果一个源对象具有一个访问器属性，那么复制到目标对象的是 getter 函数返回的值，而不是 getter 函数本身。示例 14-1 演示了如何使用`Object.getOwnPropertyDescriptor()`和`Object.defineProperty()`创建`Object.assign()`的变体，该变体复制整个属性描述符而不仅仅是复制属性值。

##### 示例 14-1\. 从一个对象复制属性及其属性到另一个对象

```js
/*
 * Define a new Object.assignDescriptors() function that works like
 * Object.assign() except that it copies property descriptors from
 * source objects into the target object instead of just copying
 * property values. This function copies all own properties, both
 * enumerable and non-enumerable. And because it copies descriptors,
 * it copies getter functions from source objects and overwrites setter
 * functions in the target object rather than invoking those getters and
 * setters.
 *
 * Object.assignDescriptors() propagates any TypeErrors thrown by
 * Object.defineProperty(). This can occur if the target object is sealed
 * or frozen or if any of the source properties try to change an existing
 * non-configurable property on the target object.
 *
 * Note that the assignDescriptors property is added to Object with
 * Object.defineProperty() so that the new function can be created as
 * a non-enumerable property like Object.assign().
 */
Object.defineProperty(Object, "assignDescriptors", {
    // Match the attributes of Object.assign()
    writable: true,
    enumerable: false,
    configurable: true,
    // The function that is the value of the assignDescriptors property.
    value: function(target, ...sources) {
        for(let source of sources) {
            for(let name of Object.getOwnPropertyNames(source)) {
                let desc = Object.getOwnPropertyDescriptor(source, name);
                Object.defineProperty(target, name, desc);
            }

            for(let symbol of Object.getOwnPropertySymbols(source)) {
                let desc = Object.getOwnPropertyDescriptor(source, symbol);
                Object.defineProperty(target, symbol, desc);
            }
        }
        return target;
    }
});

let o = {c: 1, get count() {return this.c++;}}; // Define object with getter
let p = Object.assign({}, o);                   // Copy the property values
let q = Object.assignDescriptors({}, o);        // Copy the property descriptors
p.count   // => 1: This is now just a data property so
p.count   // => 1: ...the counter does not increment.
q.count   // => 2: Incremented once when we copied it the first time,
q.count   // => 3: ...but we copied the getter method so it increments.
```

# 14.2 对象的可扩展性

对象的*可扩展*属性指定了是否可以向对象添加新属性。普通的 JavaScript 对象默认是可扩展的，但你可以通过本节描述的函数来改变这一点。

要确定一个对象是否可扩展，请将其传递给`Object.isExtensible()`。要使对象不可扩展，请将其传递给`Object.preventExtensions()`。一旦这样做，任何尝试向对象添加新属性的操作在严格模式下都会抛出 TypeError，在非严格模式下会静默失败而不会报错。此外，尝试更改不可扩展对象的原型（参见§14.3）将始终抛出 TypeError。

请注意，一旦将对象设置为不可扩展，就没有办法再使其可扩展。另外，请注意，调用`Object.preventExtensions()`只影响对象本身的可扩展性。如果向不可扩展对象的原型添加新属性，那么不可扩展对象将继承这些新属性。

有两个类似的函数，`Reflect.isExtensible()`和`Reflect.preventExtensions()`，在§14.6 中描述。

*可扩展*属性的目的是能够将对象“锁定”到已知状态，并防止外部篡改。对象的*可扩展*属性通常与属性的*可配置*和*可写*属性一起使用，JavaScript 定义了使设置这些属性变得容易的函数：

+   `Object.seal()`的作用类似于`Object.preventExtensions()`，但除了使对象不可扩展外，它还使该对象的所有自有属性不可配置。这意味着无法向对象添加新属性，也无法删除或配置现有属性。但是，可写的现有属性仍然可以设置。无法取消密封的对象。你可以使用`Object.isSealed()`来确定对象是否被密封。

+   `Object.freeze()` 更加严格地锁定对象。除了使对象不可扩展和其属性不可配置外，它还使对象的所有自有数据属性变为只读。（如果对象具有具有 setter 方法的访问器属性，则这些属性不受影响，仍然可以通过对属性赋值来调用。）使用 `Object.isFrozen()` 来确定对象是否被冻结。

需要理解的是 `Object.seal()` 和 `Object.freeze()` 只会影响它们所传递的对象：它们不会影响该对象的原型。如果你想完全锁定一个对象，可能需要同时封闭或冻结原型链中的对象。

`Object.preventExtensions()`, `Object.seal()`, 和 `Object.freeze()` 都会返回它们所传递的对象，这意味着你可以在嵌套函数调用中使用它们：

```js
// Create a sealed object with a frozen prototype and a non-enumerable property
let o = Object.seal(Object.create(Object.freeze({x: 1}),
                                  {y: {value: 2, writable: true}}));
```

如果你正在编写一个 JavaScript 库，将对象传递给库用户编写的回调函数，你可能会在这些对象上使用 `Object.freeze()` 来防止用户的代码修改它们。这样做很容易和方便，但也存在一些权衡：冻结的对象可能会干扰常见的 JavaScript 测试策略，例如。

# 14.3 原型属性

一个对象的 `prototype` 属性指定了它继承属性的对象。（查看 §6.2.3 和 §6.3.2 了解更多关于原型和属性继承的内容。）这是一个非常重要的属性，我们通常简单地说“`o` 的原型”而不是“`o` 的 `prototype` 属性”。还要记住，当 `prototype` 出现在代码字体中时，它指的是一个普通对象属性，而不是 `prototype` 属性：第九章 解释了构造函数的 `prototype` 属性指定了使用该构造函数创建的对象的 `prototype` 属性。

`prototype` 属性在对象创建时设置。通过对象字面量创建的对象使用 `Object.prototype` 作为它们的原型。通过 `new` 创建的对象使用它们构造函数的 `prototype` 属性的值作为它们的原型。通过 `Object.create()` 创建的对象使用该函数的第一个参数（可能为 `null`）作为它们的原型。

你可以通过将对象传递给 `Object.getPrototypeOf()` 来查询任何对象的原型：

```js
Object.getPrototypeOf({})      // => Object.prototype
Object.getPrototypeOf([])      // => Array.prototype
Object.getPrototypeOf(()=>{})  // => Function.prototype
```

一个非常相似的函数 `Reflect.getPrototypeOf()` 在 §14.6 中描述。

要确定一个对象是否是另一个对象的原型（或是原型链的一部分），使用 `isPrototypeOf()` 方法：

```js
let p = {x: 1};                   // Define a prototype object.
let o = Object.create(p);         // Create an object with that prototype.
p.isPrototypeOf(o)                // => true: o inherits from p
Object.prototype.isPrototypeOf(p) // => true: p inherits from Object.prototype
Object.prototype.isPrototypeOf(o) // => true: o does too
```

注意，`isPrototypeOf()` 执行类似于 `instanceof` 运算符的功能（参见 §4.9.4）。

对象的 `prototype` 属性在对象创建时设置并通常保持不变。但是，你可以使用 `Object.setPrototypeOf()` 改变对象的原型：

```js
let o = {x: 1};
let p = {y: 2};
Object.setPrototypeOf(o, p); // Set the prototype of o to p
o.y      // => 2: o now inherits the property y
let a = [1, 2, 3];
Object.setPrototypeOf(a, p); // Set the prototype of array a to p
a.join   // => undefined: a no longer has a join() method
```

通常不需要使用 `Object.setPrototypeOf()`。JavaScript 实现可能会基于对象原型是固定且不变的假设进行激进的优化。这意味着如果你调用 `Object.setPrototypeOf()`，使用修改后的对象的任何代码可能比通常运行得慢得多。

一个类似的函数 `Reflect.setPrototypeOf()` 在 §14.6 中描述。

一些早期的浏览器实现暴露了对象的`prototype`属性，通过`__proto__`属性（以两个下划线开头和结尾）。尽管这种做法早已被弃用，但网络上存在大量依赖`__proto__`的现有代码，因此 ECMAScript 标准要求所有在 Web 浏览器中运行的 JavaScript 实现都必须支持它（Node 也支持，尽管标准不要求 Node 支持）。在现代 JavaScript 中，`__proto__`是可读写的，你可以（尽管不应该）将其用作`Object.getPrototypeOf()`和`Object.setPrototypeOf()`的替代方法。然而，`__proto__`的一个有趣用途是定义对象字面量的原型：

```js
let p = {z: 3};
let o = {
    x: 1,
    y: 2,
    __proto__: p
};
o.z  // => 3: o inherits from p
```

# 14.4 众所周知的符号

Symbol 类型是在 ES6 中添加到 JavaScript 中的，这样做的一个主要原因是可以安全地向语言添加扩展，而不会破坏已部署在 Web 上的代码的兼容性。我们在第十二章中看到了一个例子，我们学到可以通过实现一个方法，其“名称”是符号`Symbol.iterator`，使一个类可迭代。

`Symbol.iterator`是“众所周知的符号”中最为人熟知的例子。这些是一组作为`Symbol()`工厂函数属性存储的符号值，用于允许 JavaScript 代码控制对象和类的某些低级行为。接下来的小节描述了每个这些众所周知的符号，并解释了它们的用途。

## 14.4.1 Symbol.iterator 和 Symbol.asyncIterator

`Symbol.iterator` 和 `Symbol.asyncIterator` 符号允许对象或类使自己可迭代或异步可迭代。它们在第十二章和§13.4.2 中有详细介绍，这里仅为完整性而提及。

## 14.4.2 Symbol.hasInstance

当描述`instanceof`运算符时，在§4.9.4 中我们说右侧必须是一个构造函数，并且表达式`o instanceof f`通过查找`o`的原型链中的值`f.prototype`来进行评估。这仍然是正确的，但在 ES6 及更高版本中，`Symbol.hasInstance`提供了一种替代方法。在 ES6 中，如果`instanceof`的右侧是具有`[Symbol.hasInstance]`方法的任何对象，则该方法将以其参数作为左侧值调用，并且方法的返回值，转换为布尔值，成为`instanceof`运算符的值。当然，如果右侧的值没有`[Symbol.hasInstance]`方法但是一个函数，则`instanceof`运算符会按照其普通方式行为。

`Symbol.hasInstance`意味着我们可以使用`instanceof`运算符来进行通用类型检查，只需定义适当的伪类型对象即可。例如：

```js
// Define an object as a "type" we can use with instanceof
let uint8 = {
    Symbol.hasInstance {
        return Number.isInteger(x) && x >= 0 && x <= 255;
    }
};
128 instanceof uint8     // => true
256 instanceof uint8     // => false: too big
Math.PI instanceof uint8 // => false: not an integer
```

请注意，这个例子很巧妙但令人困惑，因为它使用了一个非类对象，而通常期望的是一个类。对于你的代码读者来说，编写一个`isUint8()`函数而不依赖于`Symbol.hasInstance`行为会更容易理解。

## 14.4.3 Symbol.toStringTag

如果调用基本 JavaScript 对象的`toString()`方法，你会得到字符串“[object Object]”：

```js
{}.toString()  // => "[object Object]"
```

如果将相同的`Object.prototype.toString()`函数作为内置类型的实例方法调用，你会得到一些有趣的结果：

```js
Object.prototype.toString.call([])     // => "[object Array]"
Object.prototype.toString.call(/./)    // => "[object RegExp]"
Object.prototype.toString.call(()=>{}) // => "[object Function]"
Object.prototype.toString.call("")     // => "[object String]"
Object.prototype.toString.call(0)      // => "[object Number]"
Object.prototype.toString.call(false)  // => "[object Boolean]"
```

使用`Object.prototype.toString().call()`技术可以获取任何 JavaScript 值的“类属性”，其中包含了否则无法获取的类型信息。下面的`classof()`函数可能比`typeof`运算符更有用，因为它可以区分不同类型的对象：

```js
function classof(o) {
    return Object.prototype.toString.call(o).slice(8,-1);
}

classof(null)       // => "Null"
classof(undefined)  // => "Undefined"
classof(1)          // => "Number"
classof(10n**100n)  // => "BigInt"
classof("")         // => "String"
classof(false)      // => "Boolean"
classof(Symbol())   // => "Symbol"
classof({})         // => "Object"
classof([])         // => "Array"
classof(/./)        // => "RegExp"
classof(()=>{})     // => "Function"
classof(new Map())  // => "Map"
classof(new Set())  // => "Set"
classof(new Date()) // => "Date"
```

在 ES6 之前，`Object.prototype.toString()`方法的这种特殊行为仅适用于内置类型的实例，如果你在自己定义的类的实例上调用`classof()`函数，它将简单地返回“Object”。然而，在 ES6 中，`Object.prototype.toString()`会在其参数上查找一个名为`Symbol.toStringTag`的符号名称属性，如果存在这样的属性，它将在输出中使用属性值。这意味着如果你定义了自己的类，你可以轻松地使其与`classof()`等函数一起工作：

```js
class Range {
    get [Symbol.toStringTag]() { return "Range"; }
    // the rest of this class is omitted here
}
let r = new Range(1, 10);
Object.prototype.toString.call(r)   // => "[object Range]"
classof(r)                          // => "Range"
```

## 14.4.4 Symbol.species

在 ES6 之前，JavaScript 没有提供任何真正的方法来创建 Array 等内置类的强大子类。然而，在 ES6 中，你可以简单地使用`class`和`extends`关键字来扩展任何内置类。§9.5.2 演示了这个简单的 Array 子类：

```js
// A trivial Array subclass that adds getters for the first and last elements.
class EZArray extends Array {
    get first() { return this[0]; }
    get last() { return this[this.length-1]; }
}

let e = new EZArray(1,2,3);
let f = e.map(x => x * x);
e.last  // => 3: the last element of EZArray e
f.last  // => 9: f is also an EZArray with a last property
```

Array 定义了`concat()`、`filter()`、`map()`、`slice()`和`splice()`等方法，它们返回数组。当我们创建一个像 EZArray 这样继承这些方法的数组子类时，继承的方法应该返回 Array 的实例还是 EZArray 的实例？对于任一选择都有充分的理由，但 ES6 规范表示（默认情况下）这五个返回数组的方法将返回子类的实例。

它是如何工作的：

+   在 ES6 及以后的版本中，`Array()`构造函数有一个名为`Symbol.species`的符号属性。（请注意，此 Symbol 用作构造函数的属性名称。这里描述的大多数其他知名 Symbols 用作原型对象的方法名称。）

+   当我们使用`extends`创建一个子类时，结果子类构造函数会继承自超类构造函数的属性。（这是正常继承的一种，其中子类的实例继承自超类的方法。）这意味着每个 Array 的子类构造函数也会继承一个名为`Symbol.species`的继承属性。（或者子类可以定义自己的具有此名称的属性，如果需要的话。）

+   在 ES6 及以后的版本中，像`map()`和`slice()`这样创建并返回新数组的方法稍作调整。它们（实际上）调用`new this.constructor[Symbol.species]()`来创建新数组。

现在这里有趣的部分。假设`Array[Symbol.species]`只是一个常规的数据属性，像这样定义：

```js
Array[Symbol.species] = Array;
```

在这种情况下，子类构造函数将以`Array()`构造函数作为其“species”继承，并在数组子类上调用`map()`将返回超类的实例而不是子类的实例。然而，ES6 实际上并不是这样行为的。原因是`Array[Symbol.species]`是一个只读的访问器属性，其 getter 函数简单地返回`this`。子类构造函数继承了这个 getter 函数，这意味着默认情况下，每个子类构造函数都是其自己的“species”。

然而，有时这种默认行为并不是你想要的。如果你希望 EZArray 的返回数组方法返回常规的 Array 对象，你只需要将`EZArray[Symbol.species]`设置为`Array`。但由于继承的属性是只读访问器，你不能简单地使用赋值运算符来设置它。不过，你可以使用`defineProperty()`：

```js
EZArray[Symbol.species] = Array; // Attempt to set a read-only property fails

// Instead we can use defineProperty():
Object.defineProperty(EZArray, Symbol.species, {value: Array});
```

最简单的选择可能是在创建子类时明确定义自己的`Symbol.species`getter：

```js
class EZArray extends Array {
    static get [Symbol.species]() { return Array; }
    get first() { return this[0]; }
    get last() { return this[this.length-1]; }
}

let e = new EZArray(1,2,3);
let f = e.map(x => x - 1);
e.last  // => 3
f.last  // => undefined: f is a regular array with no last getter
```

创建有用的 Array 子类是引入`Symbol.species`的主要用例，但这并不是这个众所周知的 Symbol 被使用的唯一场合。Typed array 类使用 Symbol 的方式与 Array 类相同。类似地，ArrayBuffer 的`slice()`方法查看`this.constructor`的`Symbol.species`属性，而不是简单地创建一个新的 ArrayBuffer。而像`then()`这样返回新 Promise 对象的 Promise 方法也通过这种 species 协议创建这些对象。最后，如果您发现自己正在对 Map（例如）进行子类化并定义返回新 Map 对象的方法，您可能希望为了您的子类的好处自己使用`Symbol.species`。

## 14.4.5 `Symbol.isConcatSpreadable`

Array 方法`concat()`是前一节描述的使用`Symbol.species`确定返回的数组要使用的构造函数之一。但`concat()`还使用`Symbol.isConcatSpreadable`。回顾§7.8.3，数组的`concat()`方法将其`this`值和其数组参数与非数组参数区别对待：非数组参数简单地附加到新数组，但`this`数组和任何数组参数被展平或“展开”，以便将数组的元素连接起来而不是数组参数本身。

在 ES6 之前，`concat()`只是使用`Array.isArray()`来确定是否将一个值视为数组。在 ES6 中，算法略有改变：如果传递给`concat()`的参数（或`this`值）是一个对象，并且具有符号名称`Symbol.isConcatSpreadable`的属性，则该属性的布尔值用于确定是否应“展开”参数。如果不存在这样的属性，则像语言的早期版本一样使用`Array.isArray()`。

有两种情况下您可能想使用这个 Symbol：

+   如果您创建一个类似数组的对象（参见§7.9），并希望在传递给`concat()`时表现得像真正的数组，您可以简单地向您的对象添加这个符号属性：

    ```js
    let arraylike = {
        length: 1,
        0: 1,
        [Symbol.isConcatSpreadable]: true
    };
    [].concat(arraylike)  // => [1]: (would be [[1]] if not spread)
    ```

+   默认情况下，数组子类是可展开的，因此，如果您正在定义一个数组子类，而不希望在使用`concat()`时表现得像数组，那么您可以在子类中添加一个类似这样的 getter：

    ```js
    class NonSpreadableArray extends Array {
        get [Symbol.isConcatSpreadable]() { return false; }
    }
    let a = new NonSpreadableArray(1,2,3);
    [].concat(a).length // => 1; (would be 3 elements long if a was spread)
    ```

## 14.4.6 模式匹配符号

§11.3.2 记载了使用 RegExp 参数执行模式匹配操作的 String 方法。在 ES6 及更高版本中，这些方法已被泛化，可以与 RegExp 对象或任何定义了通过具有符号名称的属性进行模式匹配行为的对象一起使用。对于每个字符串方法`match()`、`matchAll()`、`search()`、`replace()`和`split()`，都有一个对应的众所周知的 Symbol：`Symbol.match`、`Symbol.search`等等。

正则表达式是描述文本模式的一种通用且非常强大的方式，但它们可能会很复杂，不太适合模糊匹配。通过泛化的字符串方法，您可以使用众所周知的 Symbol 方法定义自己的模式类，以提供自定义匹配。例如，您可以使用 Intl.Collator（参见§11.7.3）执行字符串比较，以在匹配时忽略重音。或者您可以基于 *Soundex* 算法定义一个模式类，根据其近似音调匹配单词或宽松匹配给定的 Levenshtein 距离内的字符串。

通常，当您在模式对象上调用这五个 String 方法之一时：

```js
string.method(pattern, arg)
```

那个调用会变成在您的模式对象上调用一个以符号命名的方法：

```js
patternsymbol
```

举个例子，考虑下一个示例中的模式匹配类，它使用简单的`*`和`?`通配符实现模式匹配，你可能从文件系统中熟悉这些通配符。这种模式匹配风格可以追溯到 Unix 操作系统的早期，这些模式通常被称为*globs*：

```js
class Glob {
    constructor(glob) {
        this.glob = glob;

        // We implement glob matching using RegExp internally.
        // ? matches any one character except /, and * matches zero or more
        // of those characters. We use capturing groups around each.
        let regexpText = glob.replace("?", "([^/])").replace("*", "([^/]*)");

        // We use the u flag to get Unicode-aware matching.
        // Globs are intended to match entire strings, so we use the ^ and $
        // anchors and do not implement search() or matchAll() since they
        // are not useful with patterns like this.
        this.regexp = new RegExp(`^${regexpText}$`, "u");
    }

    toString() { return this.glob; }

    Symbol.search { return s.search(this.regexp); }
    Symbol.match  { return s.match(this.regexp); }
    Symbol.replace {
        return s.replace(this.regexp, replacement);
    }
}

let pattern = new Glob("docs/*.txt");
"docs/js.txt".search(pattern)   // => 0: matches at character 0
"docs/js.htm".search(pattern)   // => -1: does not match
let match = "docs/js.txt".match(pattern);
match[0]     // => "docs/js.txt"
match[1]     // => "js"
match.index  // => 0
"docs/js.txt".replace(pattern, "web/$1.htm")  // => "web/js.htm"
```

## 14.4.7 Symbol.toPrimitive

§3.9.3 解释了 JavaScript 有三种稍有不同的算法来将对象转换为原始值。粗略地说，对于期望或偏好字符串值的转换，JavaScript 首先调用对象的`toString()`方法，如果未定义或未返回原始值，则回退到`valueOf()`方法。对于偏好数值的转换，JavaScript 首先尝试`valueOf()`方法，如果未定义或未返回原始值，则回退到`toString()`。最后，在没有偏好的情况下，它让类决定如何进行转换。日期对象首先使用`toString()`进行转换，而所有其他类型首先尝试`valueOf()`。

在 ES6 中，著名的 Symbol `Symbol.toPrimitive` 允许你重写默认的对象到原始值的行为，并完全控制你自己类的实例将如何转换为原始值。为此，请定义一个具有这个符号名称的方法。该方法必须返回某种代表对象的原始值。你定义的方法将被调用一个字符串参数，告诉你 JavaScript 正在尝试对你的对象进行什么样的转换：

+   如果参数是`"string"`，这意味着 JavaScript 在一个期望或偏好（但不是必须）字符串的上下文中进行转换。例如，当你将对象插入模板文字中时会发生这种情况。

+   如果参数是`"number"`，这意味着 JavaScript 在一个期望或偏好（但不是必须）数字值的上下文中进行转换。当你使用对象与`<`或`>`运算符或使用`-`和`*`等算术运算符时会发生这种情况。

+   如果参数是`"default"`，这意味着 JavaScript 在一个数字或字符串值都可以使用的上下文中转换你的对象。这发生在`+`、`==`和`!=`运算符中。

许多类可以忽略参数，并在所有情况下简单地返回相同的原始值。如果你希望你的类的实例可以使用`<`和`>`进行比较和排序，那么这是定义`[Symbol.toPrimitive]`方法的一个很好的理由。

## 14.4.8 Symbol.unscopables

我们将在这里介绍的最后一个著名 Symbol 是一个晦涩的 Symbol，它是为了解决由于已弃用的`with`语句引起的兼容性问题而引入的。回想一下，`with`语句接受一个对象，并执行其语句体，就好像它在对象的属性是变量的作用域中执行一样。当向 Array 类添加新方法时，这导致了兼容性问题，并且破坏了一些现有代码。`Symbol.unscopables` 就是解决方案。在 ES6 及更高版本中，`with`语句已经稍作修改。当与对象`o`一起使用时，`with`语句计算`Object.keys(o[Symbol.unscopables]||{})`，并在创建模拟作用域以执行其语句体时忽略其名称在生成的数组中的属性。ES6 使用这个方法向`Array.prototype`添加新方法，而不会破坏网络上的现有代码。这意味着你可以通过评估来找到最新的 Array 方法列表：

```js
let newArrayMethods = Object.keys(Array.prototype[Symbol.unscopables]);
```

# 14.5 模板标签

反引号内的字符串称为“模板字面量”，在 §3.3.4 中有介绍。当一个值为函数的表达式后面跟着一个模板字面量时，它变成了一个函数调用，并且我们称之为“标记模板字面量”。为标记模板字面量定义一个新的标记函数可以被视为元编程，因为标记模板经常用于定义 DSL—领域特定语言—并且定义一个新的标记函数就像向 JavaScript 添加新的语法。标记模板字面量已经被许多前端 JavaScript 包采用。GraphQL 查询语言使用 `gql` 标签函数来允许查询嵌入到 JS 代码中。以及，Emotion 库使用`css`标记函数来使 CSS 样式嵌入到 JavaScript 中。本节演示了如何编写自己的类似这样的标记函数。

标记函数没有什么特别之处：它们是普通的 JavaScript 函数，不需要特殊的语法来定义它们。当一个函数表达式后面跟着一个模板字面量时，该函数被调用。第一个参数是一个字符串数组，然后是零个或多个额外参数，这些参数可以是任何类型的值。

参数的数量取决于插入到模板字面量中的值的数量。如果模板字面量只是一个没有插值的常量字符串，那么标记函数将被调用一个包含那个字符串的数组和没有额外参数的数组。如果模板字面量包含一个插入值，那么标记函数将被调用两个参数。第一个是包含两个字符串的数组，第二个是插入的值。初始数组中的字符串是插入值左侧的字符串和右侧的字符串，其中任何一个都可能是空字符串。如果模板字面量包含两个插入值，那么标记函数将被调用三个参数：一个包含三个字符串和两个插入值的数组。这三个字符串（任何一个或全部可能为空）是第一个值左侧的文本、两个值之间的文本和第二个值右侧的文本。在一般情况下，如果模板字面量有 `n` 个插入值，那么标记函数将被调用 `n+1` 个参数。第一个参数将是一个包含 `n+1` 个字符串的数组，其余参数是 `n` 个插入值，按照它们在模板字面量中出现的顺序。

模板字面量的值始终是一个字符串。但是标记模板字面量的值是标记函数返回的任何值。这可能是一个字符串，但是当标记函数用于实现 DSL 时，返回值通常是一个非字符串数据结构，它是字符串的解析表示。

作为一个返回字符串的模板标签函数的示例，考虑以下 `html` 模版，当你打算将值安全插入 HTML 字符串时比较实用。在使用它构建最终字符串之前，该标签在每个值上执行 HTML 转义：

```js

function html(strings, ...values) {
    // 将每个值转换为字符串并转义特殊的 HTML 字符
    let escaped = values.map(v => String(v)
                                .replace("&", "&amp;")
                                .replace("<", "&lt;")
                                .replace(">", "&gt;");
                                .replace('"', "&quot;")
                                .replace("'", "&#39;"));
    // 返回连接的字符串和转义的值
    let result = strings[0];
    for(let i = 0; i < escaped.length; i++) {
        result += escaped[i] + strings[i+1];
    }
    return result;
}
let operator = "<";
html`<b>x ${operator} y</b>`             // => "<b>x &lt; y</b>"
let kind = "game", name = "D&D";
html`<div class="${kind}">${name}</div>` // =>'<div class="game">D&amp;D</div>'
```

对于不返回字符串而是返回字符串的解析表示的标记函数的示例，回想一下 14.4.6 中定义的 Glob 模式类。由于`Glob()`构造函数采用单个字符串参数，因此我们可以定义一个标记函数来创建新的 Glob 对象:

```js

function glob(strings, ...values) {
    // 将字符串和值组装成一个字符串
    let s = strings[0];
    for(let i = 0; i < values.length; i++) {
        s += values[i] + strings[i+1];
    }
    // 返回该字符串的解析表示
    return new Glob(s);
}
let root = "/tmp";
let filePattern = glob`${root}/*.html`;  // 一个 RegExp 替代方案
"/tmp/test.html".match(filePattern)[1]   // => "test"

```

3.3.4 节中提到的特性之一是`String.raw`标签函数，以其“原始”形式返回一个字符串，而不解释任何反斜杠转义序列。这是使用我们尚未讨论过的标签函数调用的一个特性实现的。当调用标签函数时，我们已经看到它的第一个参数是一个字符串数组。但是这个数组还有一个名为 `raw` 的属性，该属性的值是另一个具有相同数量元素的字符串数组。参数数组包含已解释转义序列的字符串。原始数组包含未解释转义序列的字符串。如果要定义使用反斜杠的语法的语法的 DSL，这个晦涩的特性是重要的。例如，如果我们想要我们的 `glob` 标签函数支持 Windows 风格路径上的模式匹配（它使用反斜杠而不是正斜杠），并且我们不希望标签的用户必须双写每个反斜杠，我们可以重写该函数来使用`strings.raw[]`而不是`strings[]`。 当然，缺点可能是我们不能再在我们的 glob 字面值中使用转义，例如`\u`。

# 14.6 Reflect API

Reflect 对象不是一个类；像 Math 对象一样，其属性只是定义了一组相关函数。这些函数在 ES6 中添加，定义了一个 API，用于“反射”对象及其属性。这里几乎没有新功能：Reflect 对象定义了一组方便的函数，全部在一个命名空间中，模仿核心语言语法的行为并复制各种现有 Object 函数的功能。

尽管 Reflect 函数不提供任何新功能，但它们将功能组合在一个便利的 API 中。而且，重要的是，Reflect 函数集与我们将在§14.7 中学习的 Proxy 处理程序方法一一对应。

Reflect API 包括以下函数：

`Reflect.apply(f, o, args)`

此函数将函数`f`作为`o`的方法调用（如果`o`为`null`，则作为没有`this`值的函数调用），并将`args`数组中的值作为参数传递。它等效于`f.apply(o, args)`。

`Reflect.construct(c, args, newTarget)`

此函数调用构造函数`c`，就像使用`new`关键字一样，并将数组`args`的元素作为参数传递。如果指定了可选的`newTarget`参数，则它将用作构造函数调用中的`new.target`值。如果未指定，则`new.target`值将为`c`。

`Reflect.defineProperty(o, name, descriptor)`

此函数在对象`o`上定义属性，使用`name`（字符串或符号）作为属性的名称。描述符对象应该定义属性的值（或 getter 和/或 setter）和属性的属性。`Reflect.defineProperty()`与`Object.defineProperty()`非常相似，但成功时返回`true`，失败时返回`false`。（`Object.defineProperty()`成功时返回`o`，失败时抛出 TypeError。）

`Reflect.deleteProperty(o, name)`

此函数从对象`o`中删除具有指定字符串或符号名称的属性，如果成功（或不存在此属性），则返回`true`，如果无法删除属性，则返回`false`。调用此函数类似于编写`delete o[name]`。

`Reflect.get(o, name, receiver)`

此函数返回具有指定名称（字符串或符号）的对象`o`的属性的值。如果属性是具有 getter 的访问器方法，并且指定了可选的`receiver`参数，则 getter 函数将作为`receiver`的方法调用，而不是作为`o`的方法调用。调用此函数类似于评估`o[name]`。

`Reflect.getOwnPropertyDescriptor(o, name)`

此函数返回描述对象，描述对象描述了对象`o`的名为`name`的属性的属性，如果不存在此属性，则返回`undefined`。此函数与`Object.getOwnPropertyDescriptor()`几乎相同，只是 Reflect API 版本的函数要求第一个参数是对象，如果不是则抛出 TypeError。

`Reflect.getPrototypeOf(o)`

此函数返回对象`o`的原型，如果对象没有原型则返回`null`。如果`o`是原始值而不是对象，则抛出 TypeError。此函数与`Object.getPrototypeOf()`几乎相同，只是`Object.getPrototypeOf()`仅对`null`和`undefined`参数抛出 TypeError，并将其他原始值强制转换为其包装对象。

`Reflect.has(o, name)`

如果对象`o`具有指定的`name`属性（必须是字符串或符号），则此函数返回`true`。调用此函数类似于评估`name in o`。

`Reflect.isExtensible(o)`

此函数返回`true`如果对象`o`是可扩展的（§14.2），如果不可扩展则返回`false`。如果`o`不是对象，则抛出 TypeError。`Object.isExtensible()`类似，但当传递一个不是对象的参数时，它只返回`false`。

`Reflect.ownKeys(o)`

此函数返回对象`o`的属性名称的数组，如果`o`不是对象则抛出 TypeError。返回的数组中的名称将是字符串和/或符号。调用此函数类似于调用`Object.getOwnPropertyNames()`和`Object.getOwnPropertySymbols()`并组合它们的结果。

`Reflect.preventExtensions(o)`

此函数将对象`o`的*可扩展*属性（§14.2）设置为`false`，并返回`true`以指示成功。如果`o`不是对象，则抛出 TypeError。`Object.preventExtensions()`具有相同的效果，但返回`o`而不是`true`，并且不会为非对象参数抛出 TypeError。

`Reflect.set(o, name, value, receiver)`

此函数将对象`o`的指定`name`属性设置为指定的`value`。成功时返回`true`，失败时返回`false`（如果属性是只读的，则可能失败）。如果`o`不是对象，则抛出 TypeError。如果指定的属性是具有 setter 函数的访问器属性，并且传递了可选的`receiver`参数，则将调用 setter 作为`receiver`的方法，而不是作为`o`的方法。调用此函数通常与评估`o[name] = value`相同。

`Reflect.setPrototypeOf(o, p)`

此函数将对象`o`的原型设置为`p`，成功时返回`true`，失败时返回`false`（如果`o`不可扩展或操作会导致循环原型链）。如果`o`不是对象或`p`既不是对象也不是`null`，则抛出 TypeError。`Object.setPrototypeOf()`类似，但成功时返回`o`，失败时抛出 TypeError。请记住，调用这些函数之一可能会使您的代码变慢，因为它会破坏 JavaScript 解释器的优化。

# 14.7 代理对象

ES6 及更高版本中提供的 Proxy 类是 JavaScript 最强大的元编程功能。它允许我们编写改变 JavaScript 对象基本行为的代码。在§14.6 中描述的 Reflect API 是一组函数，它们直接访问 JavaScript 对象上的一组基本操作。Proxy 类的作用是允许我们自己实现这些基本操作并创建行为与普通对象不同的对象。

创建代理对象时，我们指定另外两个对象，目标对象和处理程序对象：

```js

let proxy = new Proxy(target, handlers);

```

生成的代理对象没有自己的状态或行为。每当对其执行操作（读取属性、写入属性、定义新属性、查找原型、将其作为函数调用），它都将这些操作分派到处理程序对象或目标对象。

代理对象支持的操作与 Reflect API 定义的操作相同。假设`p`是代理对象，您写`delete p.x`。`Reflect.deleteProperty()`函数的行为与`delete`运算符相同。当使用`delete`运算符删除代理对象的属性时，它会在处理程序对象上查找`deleteProperty()`方法。如果存在这样的方法，则调用它。如果处理程序对象上不存在这样的方法，则代理对象将在目标对象上执行属性删除。

对于所有基本操作，代理对象都是这样工作的：如果处理程序对象上存在适当的方法，则调用该方法执行操作。（方法名称和签名与§14.6 中涵盖的 Reflect 函数相同。）如果处理程序对象上不存在该方法，则代理对象将在目标对象上执行基本操作。这意味着代理对象可以从目标对象或处理程序对象获取其行为。如果处理程序对象为空，则代理基本上是目标对象周围的透明包装器：

```js

let t = { x: 1, y: 2 };
let p = new Proxy(t, {});
p.x          // => 1
delete p.y   // => true: 删除代理的属性 y
t.y          // => undefined: 这也会在目标中删除它
p.z = 3;     // 在代理上定义一个新属性
t.z          // => 3: 在目标上定义属性

```

这种透明包装代理本质上等同于底层目标对象，这意味着没有理由使用它而不是包装对象。然而，当创建为“可撤销代理”时，透明包装器可以很有用。你可以使用 `Proxy.revocable()` 工厂函数，而不是使用 `Proxy()` 构造函数来创建代理。这个函数返回一个对象，其中包括一个代理对象和一个 `revoke()` 函数。一旦调用 `revoke()` 函数，代理立即停止工作：

```js

function accessTheDatabase() { /* 实现省略 */ return 42; }
let {proxy, revoke} = Proxy.revocable(accessTheDatabase, {});
proxy()   // => 42: 代理提供对基础目标函数的访问
revoke(); // 但是访问可以随时关闭
proxy();  // !TypeError: 我们不能再调用这个函数

```

注意，除了演示可撤销代理之外，上述代码还演示了代理可以与目标函数以及目标对象一起使用。但这里的主要观点是可撤销代理是一种代码隔离的基础，当你处理不受信任的第三方库时，可能会用到它们。例如，如果你必须将一个函数传递给一个你无法控制的库，你可以传递一个可撤销代理，然后在完成与库的交互后撤销代理。这可以防止库保留对你的函数的引用，并在意想不到的时候调用它。这种防御性编程在 JavaScript 程序中并不典型，但 Proxy 类至少使其成为可能。

如果我们将非空处理程序对象传递给 `Proxy()` 构造函数，那么我们不再定义一个透明的包装器对象，而是为我们的代理实现自定义行为。通过正确设置处理程序，底层目标对象基本上变得无关紧要。

例如，以下代码展示了如何实现一个看起来具有无限只读属性的对象，其中每个属性的值与属性的名称相同：

```js

// 我们使用代理创建一个对象, 看起来拥有每个
// 可能的属性, 每个属性的值都等于其名称
let identity = new Proxy({}, {
    // 每个属性都以其名称作为其值
    get(o, name, target) { return name; },
    // 每个属性名称都已定义
    has(o, name) { return true; },
    // 有太多属性要枚举, 所以我们只是抛出
    ownKeys(o) { throw new RangeError("属性数量无限"); },
    // 所有属性都存在且不可写, 不可配置或可枚举。
    getOwnPropertyDescriptor(o, name) {
        return {
            value: name,
            enumerable: false,
            writable: false,
            configurable: false
        };
    },
    // 所有属性都是只读的, 因此无法设置
    set(o, name, value, target) { return false; },
    // 所有属性都是不可配置的, 因此它们无法被删除
    deleteProperty(o, name) { return false; },
    // 所有属性都存在且不可配置, 因此我们无法定义更多
    defineProperty(o, name, desc) { return false; },
    // 实际上, 这意味着对象不可扩展
    isExtensible(o) { return false; },
    // 所有属性已在此对象上定义, 因此无法
    // 即使它有原型对象, 也不会继承任何东西。
    getPrototypeOf(o) { return null; },
    // 对象不可扩展, 因此我们无法更改原型
    setPrototypeOf(o, proto) { return false; },
});
identity.x                // => "x"
identity.toString         // => "toString"
identity[0]               // => "0"
identity.x = 1;           // 设置属性没有效果
identity.x                // => "x"
delete identity.x         // => false: 也无法删除属性
identity.x                // => "x"
Object.keys(identity);    // !RangeError: 无法列出所有键
for(let p of identity) ;  // !RangeError

```

代理对象可以从目标对象和处理程序对象派生其行为，到目前为止我们看到的示例都使用了一个对象或另一个对象。但通常更有用的是定义同时使用两个对象的代理。

例如，下面的代码使用 Proxy 创建了一个目标对象的只读包装器。当代码尝试从对象中读取值时，这些读取会正常转发到目标对象。但如果任何代码尝试修改对象或其属性，处理程序对象的方法会抛出 TypeError。这样的代理可能有助于编写测试：假设你编写了一个接受对象参数的函数，并希望确保你的函数不会尝试修改输入参数。如果你的测试传入一个只读包装器对象，那么任何写入操作都会抛出异常，导致测试失败：

```js

function readOnlyProxy(o) {
    function readonly() { throw new TypeError("只读"); }
    return new Proxy(o, {
        set: readonly,
        defineProperty: readonly,
        deleteProperty: readonly,
        setPrototypeOf: readonly,
    });
}
let o = { x: 1, y: 2 };    // 普通可写对象
let p = readOnlyProxy(o);  // 它的只读版本
p.x                        // => 1: 读取属性有效
p.x = 2;                   // !TypeError: 无法更改属性
delete p.y;                // !TypeError: 无法删除属性
p.z = 3;                   // !TypeError: 无法添加属性
p.__proto__ = {};          // !TypeError: 无法更改原型

```

写代理时的另一种技术是定义处理程序方法，拦截对象上的操作，但仍将操作委托给目标对象。Reflect API（§14.6）的函数具有与处理程序方法完全相同的签名，因此它们使得执行这种委托变得容易。

例如，这是一个代理，它将所有操作委托给目标对象，但使用处理程序方法记录操作：

```js

/*
 * 返回一个代理对象, 包装 o, 将所有操作委托给
 * 在记录每个操作后, 该对象的名称是一个字符串
 * 将出现在日志消息中以标识对象。如果 o 具有自有
 * 其值为对象或函数, 则如果您查询
 * 这些属性的值是对象或函数, 则返回代理而不是
 * 此代理的记录行为是“传染性的”。
 */
function loggingProxy(o, objname) {
    // 为我们的记录代理对象定义处理程序。
    // 每个处理程序记录一条消息, 然后委托给目标对象。
    const handlers = {
        // 这个处理程序是一个特例, 因为对于自有属性
        // 其值为对象或函数, 则返回代理而不是值本身。
        // 返回值本身。
        get(target, property, receiver){
            // 记录获取操作
            console.log(`Handler get(${objname}, ${property.toString()})`);
            // 使用 Reflect API 获取属性值
            let value = Reflect.get(target, property, receiver);
            // 如果属性是目标的自有属性且
            // 值是对象或函数, 则返回其代理。
            if(Reflect.ownKeys(target).includes(property)&&
                (typeof value === "object" || typeof value === "function")){
                return loggingProxy(value, `${objname}.${property.toString()}`);
            }
            // 否则返回未修改的值。
            返回值;
        },
        // 以下三种方法没有特殊之处：
        // 它们记录操作并委托给目标对象。
        // 它们是一个特例, 只是为了避免记录
        // 可能导致无限递归的接收器对象。
        set(target, prop, value, receiver){
            console.log(`Handler set(${objname}, ${prop.toString()}, ${value})`);
            return Reflect.set(target, prop, value, receiver);
        },
        apply(target, receiver, args){
            console.log(`Handler ${objname}(${args})`);
            return Reflect.apply(target, receiver, args);
        },
        构造(target, args, receiver){
            console.log(`Handler ${objname}(${args})`);
            return Reflect.construct(target, args, receiver);
        }
    };
    // 我们可以自动生成其余的处理程序。
    // 元编程 FTW！
    Reflect.ownKeys(Reflect).forEach(handlerName => {
        if(!(handlerName in handlers)){
            handlers[handlerName] = function(target, ...args){
                // 记录操作
                console.log(`Handler ${handlerName}(${objname}, ${args})`);
                // 委托操作
                return Reflect[handlerName](target, ...args);
            };
        }
    });
    // 返回一个对象的代理, 使用这些记录处理程序
    return new Proxy(o, handlers);

}

```

之前定义的 `loggingProxy()` 函数创建了记录其使用方式的代理。如果你试图了解一个未记录的函数如何使用你传递给它的对象，使用记录代理可以帮助。

考虑以下示例，这些示例对数组迭代产生了一些真正的见解：

```js

// 定义一个数据数组和一个具有函数属性的对象
let data = [10,20];
let methods = { square: x => x*x };
// 为数组和具有函数属性的对象创建记录代理
let proxyData = loggingProxy(data, "data");
let proxyMethods = loggingProxy(methods, "methods");
// 假设我们想要了解 Array.map()方法的工作原理
data.map(methods.square)        // => [100, 400]
// 首先, 让我们尝试使用一个记录代理数组
proxyData.map(methods.square)   // => [100, 400]
// 它产生以下输出：
// Handler get(data, map)
// Handler get(data, length)
// Handler get(data, constructor)
// Handler has(data, 0)
// Handler get(data, 0)
// Handler has(data, 1)
// Handler get(data, 1)
// 现在让我们尝试使用一个代理方法对象
data.map(proxyMethods.square)   // => [100, 400]
// 记录输出：
// Handler get(methods, square)
// Handler methods.square(10, 0, 10, 20)
// Handler methods.square(20, 1, 10, 20)
// 最后, 让我们使用一个记录代理来了解迭代协议
for(let x of proxyData) console.log("data", x);
// 记录输出：
// Handler get(data, Symbol(Symbol.iterator))
// Handler get(data, length)
// Handler get(data, 0)
// data 10
// Handler get(data, length)
// Handler get(data, 1)
// data 20
// Handler get(data, length)

```

从第一块日志输出中，我们了解到 `Array.map()` 方法在实际读取元素值之前明确检查每个数组元素的存在性（导致调用 `has()` 处理程序），然后读取元素值（触发 `get()` 处理程序）。这可能是为了区分不存在的数组元素和存在但为 undefined 的元素。

第二块日志输出可能会提醒我们，我们传递给 `Array.map()` 的函数会使用三个参数调用：元素的值、元素的索引和数组本身。（我们的日志输出中存在问题：`Array.toString()` 方法在其输出中不包含方括号，并且如果在参数列表中包含它们，日志消息将更清晰 `(10,0,[10,20])`。）

第三块日志输出向我们展示了 `for/of` 循环是通过查找具有符号名称 `[Symbol.iterator]` 的方法来工作的。它还演示了 Array 类对此迭代器方法的实现在每次迭代时都会检查数组长度，并且不假设数组长度在迭代过程中保持不变。

## 14.7.1 代理不变性

之前定义的 `readOnlyProxy()` 函数创建了实际上是冻结的代理对象：任何尝试更改属性值、属性属性或添加或删除属性的尝试都会引发异常。但只要目标对象未被冻结，我们会发现如果我们可以使用 `Reflect.isExtensible()` 和 `Reflect.getOwnPropertyDescriptor()` 查询代理，它会告诉我们应该能够设置、添加和删除属性。因此，`readOnlyProxy()` 创建了处于不一致状态的对象。我们可以通过添加 `isExtensible()` 和 `getOwnPropertyDescriptor()` 处理程序来解决这个问题，或者我们可以接受这种轻微的不一致性。

代理处理程序 API 允许我们定义具有主要不一致性的对象，但在这种情况下，代理类本身将阻止我们创建不良不一致的代理对象。在本节开始时，我们将代理描述为没有自己行为的对象，因为它们只是将所有操作转发到处理程序对象和目标对象。但这并不完全正确：在转发操作后，代理类会对结果执行一些检查，以确保不违反重要的 JavaScript 不变性。如果检测到违规，代理将抛出 TypeError，而不是让操作继续执行。

例如，如果为不可扩展对象创建代理，则如果 `isExtensible()` 处理程序返回 `true`，代理将抛出 TypeError：

```js

let target = Object.preventExtensions({});

let proxy = new Proxy(target, { isExtensible(){ return true; });

Reflect.isExtensible(proxy);  // !TypeError：不变违规

```

相关地，对于不可扩展目标的代理对象可能没有返回除目标的真实原型对象之外的 `getPrototypeOf()` 处理程序。此外，如果目标对象具有不可写、不可配置的属性，则代理类将在 `get()` 处理程序返回除实际值之外的任何内容时抛出 TypeError：

```js

let target = Object.freeze({x: 1});
let proxy = new Proxy(target, { get(){ return 99; });
proxy.x;         // !TypeError：get()返回的值与目标不匹配

```

代理强制执行许多附加不变性，几乎所有这些不变性都与不可扩展的目标对象和目标对象上的不可配置属性有关。

# 14.8 总结

在本章中，您已经学到了：

+   JavaScript 对象具有*可扩展*属性，对象属性具有*可写*、*可枚举*和*可配置*属性，以及值和 getter 和/或 setter 属性。您可以使用这些属性以各种方式“锁定”您的对象，包括创建“密封”和“冻结”对象。

+   JavaScript 定义了一些函数，允许您遍历对象的原型链，甚至更改对象的原型（尽管这样做可能会使您的代码变慢）。

+   `Symbol`对象的属性具有“众所周知的符号”值，您可以将其用作您定义的对象和类的属性或方法名称。这样做可以让您控制对象与 JavaScript 语言特性和核心库的交互方式。例如，众所周知的符号允许您使您的类可迭代，并控制将实例传递给`Object.prototype.toString()`时显示的字符串。在 ES6 之前，这种定制仅适用于内置到实现中的本机类。

+   标记模板字面量是一种函数调用语法，定义一个新的标签函数有点像向语言添加新的文字语法。定义一个解析其模板字符串参数的标签函数允许您在 JavaScript 代码中嵌入 DSL。标签函数还提供对原始、未转义的字符串文字的访问，其中反斜杠没有特殊含义。

+   代理类和相关的 Reflect API 允许对 JavaScript 对象的基本行为进行低级控制。代理对象可以用作可选撤销包装器，以改善代码封装，并且还可以用于实现非标准对象行为（例如早期 Web 浏览器定义的一些特殊情况 API）。

¹ V8 JavaScript 引擎中的一个错误意味着这段代码在 Node 13 中无法正常工作。
