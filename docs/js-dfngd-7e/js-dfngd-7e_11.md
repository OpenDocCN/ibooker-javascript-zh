# 第十一章：JavaScript 标准库

一些数据类型，如数字和字符串（第三章）、对象（第六章）和数组（第七章）对于 JavaScript 来说是如此基础，以至于我们可以将它们视为语言本身的一部分。本章涵盖了其他重要但不太基础的 API，可以被视为 JavaScript 的“标准库”：这些是内置于 JavaScript 中的有用类和函数，可供所有 Web 浏览器和 Node 中的 JavaScript 程序使用。¹

本章的各节相互独立，您可以按任意顺序阅读它们。它们涵盖了：

+   Set 和 Map 类，用于表示值的集合和从一个值集合到另一个值集合的映射。

+   类型化数组（TypedArrays）等类似数组的对象，表示二进制数据的数组，以及用于从非数组二进制数据中提取值的相关类。

+   正则表达式和 RegExp 类，定义文本模式，对文本处理很有用。本节还详细介绍了正则表达式语法。

+   Date 类用于表示和操作日期和时间。

+   Error 类及其各种子类的实例，当 JavaScript 程序发生错误时抛出。

+   JSON 对象，其方法支持对由对象、数组、字符串、数字和布尔值组成的 JavaScript 数据结构进行序列化和反序列化。

+   Intl 对象及其定义的类，可帮助您本地化 JavaScript 程序。

+   Console 对象，其方法以特别有用的方式输出字符串，用于调试程序和记录程序的行为。

+   URL 类简化了解析和操作 URL 的任务。本节还涵盖了用于对 URL 及其组件进行编码和解码的全局函数。

+   `setTimeout()`及相关函数用于指定在经过指定时间间隔后执行的代码。

本章中的一些部分，尤其是关于类型化数组和正则表达式的部分，由于您需要理解的重要背景信息较多，因此相当长。然而，其他许多部分很短：它们只是介绍一个新的 API 并展示其使用示例。

# 11.1 集合和映射

JavaScript 的 Object 类型是一种多功能的数据结构，可以用来将字符串（对象的属性名称）映射到任意值。当被映射的值是像`true`这样固定的值时，那么对象实际上就是一组字符串。

在 JavaScript 编程中，对象实际上经常被用作映射和集合，但由于限制为字符串并且对象通常继承具有诸如“toString”之类名称的属性，这使得使用起来有些复杂，通常这些属性并不打算成为映射或集合的一部分。

出于这个原因，ES6 引入了真正的 Set 和 Map 类，我们将在接下来的子章节中介绍。

## 11.1.1 Set 类

集合是一组值，类似于数组。但与数组不同，集合没有顺序或索引，并且不允许重复：一个值要么是集合的成员，要么不是成员；无法询问一个值在集合中出现多少次。

使用`Set()`构造函数创建一个 Set 对象：

```js
let s = new Set();       // A new, empty set
let t = new Set([1, s]); // A new set with two members
```

`Set()`构造函数的参数不一定是数组：任何可迭代对象（包括其他 Set 对象）都是允许的：

```js
let t = new Set(s);                  // A new set that copies the elements of s.
let unique = new Set("Mississippi"); // 4 elements: "M", "i", "s", and "p"
```

集合的`size`属性类似于数组的`length`属性：它告诉你集合包含多少个值：

```js
unique.size        // => 4
```

创建集合时无需初始化。您可以随时使用`add()`、`delete()`和`clear()`添加和删除元素。请记住，集合不能包含重复项，因此向集合添加已包含的值不会产生任何效果：

```js
let s = new Set();  // Start empty
s.size              // => 0
s.add(1);           // Add a number
s.size              // => 1; now the set has one member
s.add(1);           // Add the same number again
s.size              // => 1; the size does not change
s.add(true);        // Add another value; note that it is fine to mix types
s.size              // => 2
s.add([1,2,3]);     // Add an array value
s.size              // => 3; the array was added, not its elements
s.delete(1)         // => true: successfully deleted element 1
s.size              // => 2: the size is back down to 2
s.delete("test")    // => false: "test" was not a member, deletion failed
s.delete(true)      // => true: delete succeeded
s.delete([1,2,3])   // => false: the array in the set is different
s.size              // => 1: there is still that one array in the set
s.clear();          // Remove everything from the set
s.size              // => 0
```

关于这段代码有几个重要的要点需要注意：

+   `add()`方法接受一个参数；如果传递一个数组，它会将数组本身添加到集合中，而不是单独的数组元素。但是，`add()`始终返回调用它的集合，因此如果要向集合添加多个值，可以使用链式方法调用，如`s.add('a').add('b').add('c');`。

+   `delete()`方法也仅一次删除单个集合元素。但是，与`add()`不同，`delete()`返回一个布尔值。如果您指定的值实际上是集合的成员，则`delete()`会将其删除并返回`true`。否则，它不执行任何操作并返回`false`。

+   最后，非常重要的是要理解集合成员是基于严格的相等性检查的，就像`===`运算符执行的那样。集合可以包含数字`1`和字符串`"1"`，因为它认为它们是不同的值。当值为对象（或数组或函数）时，它们也被视为使用`===`进行比较。这就是为什么我们无法从此代码中的集合中删除数组元素的原因。我们向集合添加了一个数组，然后尝试通过向`delete()`方法传递一个*不同*的数组（尽管具有相同元素）来删除该数组。为了使其工作，我们必须传递对完全相同的数组的引用。

###### 注意

Python 程序员请注意：这是 JavaScript 和 Python 集合之间的一个重要区别。Python 集合比较成员的相等性，而不是身份，但这样做的代价是 Python 集合只允许不可变成员，如元组，并且不允许将列表和字典添加到集合中。

在实践中，我们与集合最重要的事情不是向其中添加和删除元素，而是检查指定的值是否是集合的成员。我们使用`has()`方法来实现这一点：

```js
let oneDigitPrimes = new Set([2,3,5,7]);
oneDigitPrimes.has(2)    // => true: 2 is a one-digit prime number
oneDigitPrimes.has(3)    // => true: so is 3
oneDigitPrimes.has(4)    // => false: 4 is not a prime
oneDigitPrimes.has("5")  // => false: "5" is not even a number
```

关于集合最重要的一点是它们被优化用于成员测试，无论集合有多少成员，`has()`方法都会非常快。数组的`includes()`方法也执行成员测试，但所需时间与数组的大小成正比，使用数组作为集合可能比使用真正的 Set 对象慢得多。

Set 类是可迭代的，这意味着您可以使用`for/of`循环枚举集合的所有元素：

```js
let sum = 0;
for(let p of oneDigitPrimes) { // Loop through the one-digit primes
    sum += p;                  // and add them up
}
sum                            // => 17: 2 + 3 + 5 + 7
```

因为 Set 对象是可迭代的，您可以使用`...`扩展运算符将它们转换为数组和参数列表：

```js
[...oneDigitPrimes]         // => [2,3,5,7]: the set converted to an Array
Math.max(...oneDigitPrimes) // => 7: set elements passed as function arguments
```

集合经常被描述为“无序集合”。然而，对于 JavaScript Set 类来说，这并不完全正确。JavaScript 集合是无索引的：您无法像数组那样请求集合的第一个或第三个元素。但是 JavaScript Set 类始终记住元素插入的顺序，并且在迭代集合时始终使用此顺序：插入的第一个元素将是首个迭代的元素（假设您尚未首先删除它），最近插入的元素将是最后一个迭代的元素。²

除了可迭代外，Set 类还实现了类似于数组同名方法的`forEach()`方法：

```js
let product = 1;
oneDigitPrimes.forEach(n => { product *= n; });
product     // => 210: 2 * 3 * 5 * 7
```

数组的`forEach()`将数组索引作为第二个参数传递给您指定的函数。集合没有索引，因此 Set 类的此方法简单地将元素值作为第一个和第二个参数传递。

## 11.1.2 Map 类

Map 对象表示一组称为*键*的值，其中每个键都有另一个与之关联（或“映射到”）的值。在某种意义上，映射类似于数组，但是不同于使用一组顺序整数作为键，映射允许我们使用任意值作为“索引”。与数组一样，映射很快：查找与键关联的值将很快（尽管不像索引数组那样快），无论映射有多大。

使用`Map()`构造函数创建一个新的映射：

```js
let m = new Map();  // Create a new, empty map
let n = new Map([   // A new map initialized with string keys mapped to numbers
    ["one", 1],
    ["two", 2]
]);
```

`Map()` 构造函数的可选参数应该是一个可迭代对象，产生两个元素 `[key, value]` 数组。在实践中，这意味着如果你想在创建 map 时初始化它，你通常会将所需的键和关联值写成数组的数组。但你也可以使用 `Map()` 构造函数复制其他 map，或从现有对象复制属性名和值：

```js
let copy = new Map(n); // A new map with the same keys and values as map n
let o = { x: 1, y: 2}; // An object with two properties
let p = new Map(Object.entries(o)); // Same as new map([["x", 1], ["y", 2]])
```

一旦你创建了一个 Map 对象，你可以使用 `get()` 查询与给定键关联的值，并可以使用 `set()` 添加新的键/值对。但请记住，map 是一组键，每个键都有一个关联的值。这与一组键/值对并不完全相同。如果你使用一个已经存在于 map 中的键调用 `set()`，你将改变与该键关联的值，而不是添加一个新的键/值映射。除了 `get()` 和 `set()`，Map 类还定义了类似 Set 方法的方法：使用 `has()` 检查 map 是否包含指定的键；使用 `delete()` 从 map 中删除一个键（及其关联的值）；使用 `clear()` 从 map 中删除所有键/值对；使用 `size` 属性查找 map 包含多少个键。

```js
let m = new Map();   // Start with an empty map
m.size               // => 0: empty maps have no keys
m.set("one", 1);     // Map the key "one" to the value 1
m.set("two", 2);     // And the key "two" to the value 2.
m.size               // => 2: the map now has two keys
m.get("two")         // => 2: return the value associated with key "two"
m.get("three")       // => undefined: this key is not in the set
m.set("one", true);  // Change the value associated with an existing key
m.size               // => 2: the size doesn't change
m.has("one")         // => true: the map has a key "one"
m.has(true)          // => false: the map does not have a key true
m.delete("one")      // => true: the key existed and deletion succeeded
m.size               // => 1
m.delete("three")    // => false: failed to delete a nonexistent key
m.clear();           // Remove all keys and values from the map
```

像 Set 的 `add()` 方法一样，Map 的 `set()` 方法可以链接，这允许初始化 map 而不使用数组的数组：

```js
let m = new Map().set("one", 1).set("two", 2).set("three", 3);
m.size        // => 3
m.get("two")  // => 2
```

与 Set 一样，任何 JavaScript 值都可以用作 Map 中的键或值。这包括 `null`、`undefined` 和 `NaN`，以及对象和数组等引用类型。与 Set 类一样，Map 通过标识比较键，而不是通过相等性比较，因此如果你使用对象或数组作为键，它将被认为与每个其他对象和数组都不同，即使它们具有完全相同的属性或元素：

```js
let m = new Map();   // Start with an empty map.
m.set({}, 1);        // Map one empty object to the number 1.
m.set({}, 2);        // Map a different empty object to the number 2.
m.size               // => 2: there are two keys in this map
m.get({})            // => undefined: but this empty object is not a key
m.set(m, undefined); // Map the map itself to the value undefined.
m.has(m)             // => true: m is a key in itself
m.get(m)             // => undefined: same value we'd get if m wasn't a key
```

Map 对象是可迭代的，每个迭代的值都是一个包含两个元素的数组，第一个元素是键，第二个元素是与该键关联的值。如果你使用展开运算符与 Map 对象一起使用，你将得到一个类似于我们传递给 `Map()` 构造函数的数组的数组。在使用 `for/of` 循环迭代 map 时，惯用的做法是使用解构赋值将键和值分配给单独的变量：

```js
let m = new Map([["x", 1], ["y", 2]]);
[...m]    // => [["x", 1], ["y", 2]]

for(let [key, value] of m) {
    // On the first iteration, key will be "x" and value will be 1
    // On the second iteration, key will be "y" and value will be 2
}
```

像 Set 类一样，Map 类按插入顺序进行迭代。迭代的第一个键/值对将是最近添加到 map 中的键/值对，而迭代的最后一个键/值对将是最近添加的键/值对。

如果你想仅迭代 map 的键或仅迭代关联的值，请使用 `keys()` 和 `values()` 方法：这些方法返回可迭代对象，按插入顺序迭代键和值。（`entries()` 方法返回一个可迭代对象，按键/值对迭代，但这与直接迭代 map 完全相同。）

```js
[...m.keys()]     // => ["x", "y"]: just the keys
[...m.values()]   // => [1, 2]: just the values
[...m.entries()]  // => [["x", 1], ["y", 2]]: same as [...m]
```

Map 对象也可以使用首次由 Array 类实现的 `forEach()` 方法进行迭代。

```js
m.forEach((value, key) => {  // note value, key NOT key, value
    // On the first invocation, value will be 1 and key will be "x"
    // On the second invocation, value will be 2 and key will be "y"
});
```

可能会觉得上面的代码中值参数在键参数之前有些奇怪，因为在 `for/of` 迭代中，键首先出现。正如本节开头所述，你可以将 map 视为一个广义的数组，其中整数数组索引被任意键值替换。数组的 `forEach()` 方法首先传递数组元素，然后传递数组索引，因此，类比地，map 的 `forEach()` 方法首先传递 map 值，然后传递 map 键。

## 11.1.3 WeakMap 和 WeakSet

WeakMap 类是 Map 类的变体（但不是实际的子类），不会阻止其键值被垃圾回收。垃圾回收是 JavaScript 解释器回收不再“可达”的对象内存的过程，这些对象不能被程序使用。常规映射保持对其键值的“强”引用，它们通过映射保持可达性，即使所有对它们的其他引用都消失了。相比之下，WeakMap 对其键值保持“弱”引用，因此它们不可通过 WeakMap 访问，它们在映射中的存在不会阻止其内存被回收。

`WeakMap()`构造函数与`Map()`构造函数完全相同，但 WeakMap 和 Map 之间存在一些重要的区别：

+   WeakMap 的键必须是对象或数组；原始值不受垃圾回收的影响，不能用作键。

+   WeakMap 只实现了`get()`、`set()`、`has()`和`delete()`方法。特别是，WeakMap 不可迭代，并且不定义`keys()`、`values()`或`forEach()`。如果 WeakMap 是可迭代的，那么它的键将是可达的，它就不会是弱引用的。

+   同样，WeakMap 也不实现`size`属性，因为 WeakMap 的大小随时可能会随着对象被垃圾回收而改变。

WeakMap 的预期用途是允许您将值与对象关联而不会导致内存泄漏。例如，假设您正在编写一个函数，该函数接受一个对象参数并需要对该对象执行一些耗时的计算。为了效率，您希望缓存计算后的值以供以后重用。如果使用 Map 对象来实现缓存，将阻止任何对象被回收，但使用 WeakMap，您可以避免这个问题。（您通常可以使用私有 Symbol 属性直接在对象上缓存计算后的值来实现类似的结果。参见§6.10.3。）

WeakSet 实现了一组对象，不会阻止这些对象被垃圾回收。`WeakSet()`构造函数的工作方式类似于`Set()`构造函数，但 WeakSet 对象与 Set 对象的区别与 WeakMap 对象与 Map 对象的区别相同：

+   WeakSet 不允许原始值作为成员。

+   WeakSet 只实现了`add()`、`has()`和`delete()`方法，并且不可迭代。

+   WeakSet 没有`size`属性。

WeakSet 并不经常使用：它的用例类似于 WeakMap。如果你想标记（或“品牌化”）一个对象具有某些特殊属性或类型，例如，你可以将其添加到 WeakSet 中。然后，在其他地方，当你想检查该属性或类型时，可以测试该 WeakSet 的成员资格。使用常规集合会阻止所有标记对象被垃圾回收，但使用 WeakSet 时不必担心这个问题。

# 11.2 类型化数组和二进制数据

常规 JavaScript 数组可以具有任何类型的元素，并且可以动态增长或缩小。JavaScript 实现执行许多优化，使得 JavaScript 数组的典型用法非常快速。然而，它们与低级语言（如 C 和 Java）的数组类型仍然有很大不同。*类型化数组*是 ES6 中的新功能，³它们更接近这些语言的低级数组。类型化数组在技术上不是数组（`Array.isArray()`对它们返回`false`），但它们实现了§7.8 中描述的所有数组方法以及一些自己的方法。然而，它们与常规数组在一些非常重要的方面有所不同：

+   类型化数组的元素都是数字。然而，与常规 JavaScript 数字不同，类型化数组允许您指定要存储在数组中的数字的类型（有符号和无符号整数和 IEEE-754 浮点数）和大小（8 位到 64 位）。

+   创建类型化数组时必须指定其长度，并且该长度永远不会改变。

+   类型化数组的元素在创建数组时始终初始化为 0。

## 11.2.1 类型化数组类型

JavaScript 没有定义 TypedArray 类。相反，有 11 种类型化数组，每种具有不同的元素类型和构造函数：

| 构造函数 | 数值类型 |
| --- | --- |
| `Int8Array()` | 有符号字节 |
| `Uint8Array()` | 无符号字节 |
| `Uint8ClampedArray()` | 无溢出的无符号字节 |
| `Int16Array()` | 有符号 16 位短整数 |
| `Uint16Array()` | 无符号 16 位短整数 |
| `Int32Array()` | 有符号 32 位整数 |
| `Uint32Array()` | 无符号 32 位整数 |
| `BigInt64Array()` | 有符号 64 位 BigInt 值（ES2020） |
| `BigUint64Array()` | 无符号 64 位 BigInt 值（ES2020） |
| `Float32Array()` | 32 位浮点值 |
| `Float64Array()` | 64 位浮点值：普通的 JavaScript 数字 |

名称以 `Int` 开头的类型保存有符号整数，占用 1、2 或 4 字节（8、16 或 32 位）。名称以 `Uint` 开头的类型保存相同长度的无符号整数。名称为 “BigInt” 和 “BigUint” 的类型保存 64 位整数，以 BigInt 值的形式表示在 JavaScript 中（参见 §3.2.5）。以 `Float` 开头的类型保存浮点数。`Float64Array` 的元素与普通的 JavaScript 数字相同类型。`Float32Array` 的元素精度较低，范围较小，但只需一半的内存。 （在 C 和 Java 中，此类型称为 `float`。）

`Uint8ClampedArray` 是 `Uint8Array` 的特殊变体。这两种类型都保存无符号字节，可以表示 0 到 255 之间的数字。对于 `Uint8Array`，如果将大于 255 或小于零的值存储到数组元素中，它会“环绕”，并且会得到其他值。这是计算机内存在低级别上的工作原理，因此速度非常快。`Uint8ClampedArray` 进行了一些额外的类型检查，以便如果存储大于 255 或小于 0 的值，则会“夹紧”到 255 或 0，而不会环绕。 （这种夹紧行为是 HTML `<canvas>` 元素的低级 API 用于操作像素颜色所必需的。）

每个类型化数组构造函数都有一个 `BYTES_PER_ELEMENT` 属性，其值为 1、2、4 或 8，取决于类型。

## 11.2.2 创建类型化数组

创建类型化数组的最简单方法是调用适当的构造函数，并提供一个数字参数，指定数组中要包含的元素数量：

```js
let bytes = new Uint8Array(1024);    // 1024 bytes
let matrix = new Float64Array(9);    // A 3x3 matrix
let point = new Int16Array(3);       // A point in 3D space
let rgba = new Uint8ClampedArray(4); // A 4-byte RGBA pixel value
let sudoku = new Int8Array(81);      // A 9x9 sudoku board
```

通过这种方式创建类型化数组时，数组元素都保证初始化为 `0`、`0n` 或 `0.0`。但是，如果您知道要在类型化数组中使用的值，也可以在创建数组时指定这些值。每个类型化数组构造函数都有静态的 `from()` 和 `of()` 工厂方法，类似于 `Array.from()` 和 `Array.of()`：

```js
let white = Uint8ClampedArray.of(255, 255, 255, 0);  // RGBA opaque white
```

请记住，`Array.from()` 工厂方法的第一个参数应为类似数组或可迭代对象。对于类型化数组变体也是如此，只是可迭代或类似数组的对象还必须具有数值元素。例如，字符串是可迭代的，但将它们传递给类型化数组的 `from()` 工厂方法是没有意义的。

如果只使用 `from()` 的单参数版本，可以省略 `.from` 并直接将可迭代或类似数组对象传递给构造函数，其行为完全相同。请注意，构造函数和 `from()` 工厂方法都允许您复制现有的类型化数组，同时可能更改类型：

```js
let ints = Uint32Array.from(white);  // The same 4 numbers, but as ints
```

当从现有数组、可迭代对象或类似数组对象创建新的类型化数组时，值可能会被截断以符合数组的类型约束。当发生这种情况时，不会有警告或错误：

```js
// Floats truncated to ints, longer ints truncated to 8 bits
Uint8Array.of(1.23, 2.99, 45000) // => new Uint8Array([1, 2, 200])
```

最后，还有一种使用 ArrayBuffer 类型创建 typed arrays 的方法。ArrayBuffer 是一个对一块内存的不透明引用。你可以用构造函数创建一个，只需传入你想要分配的内存字节数：

```js
let buffer = new ArrayBuffer(1024*1024);
buffer.byteLength   // => 1024*1024; one megabyte of memory
```

ArrayBuffer 类不允许你读取或写入你分配的任何字节。但你可以创建使用 buffer 内存的 typed arrays，并且允许你读取和写入该内存。为此，调用 typed array 构造函数，第一个参数是一个 ArrayBuffer，第二个参数是数组缓冲区内的字节偏移量，第三个参数是数组长度（以元素而不是字节计算）。第二和第三个参数是可选的。如果两者都省略，则数组将使用数组缓冲区中的所有内存。如果只省略长度参数，则你的数组将使用从起始位置到数组结束的所有可用内存。关于这种形式的 typed array 构造函数还有一件事要记住：数组必须是内存对齐的，所以如果你指定了一个字节偏移量，该值应该是你的类型大小的倍数。例如，`Int32Array()` 构造函数需要四的倍数，而 `Float64Array()` 需要八的倍数。

给定之前创建的 ArrayBuffer，你可以创建这样的 typed arrays：

```js
let asbytes = new Uint8Array(buffer);          // Viewed as bytes
let asints = new Int32Array(buffer);           // Viewed as 32-bit signed ints
let lastK = new Uint8Array(buffer, 1023*1024); // Last kilobyte as bytes
let ints2 = new Int32Array(buffer, 1024, 256); // 2nd kilobyte as 256 integers
```

这四种 typed arrays 提供了对由 ArrayBuffer 表示的内存的四种不同视图。重要的是要理解，所有 typed arrays 都有一个底层的 ArrayBuffer，即使你没有明确指定一个。如果你调用一个 typed array 构造函数而没有传递一个 buffer 对象，一个适当大小的 buffer 将会被自动创建。正如后面所描述的，任何 typed array 的 `buffer` 属性都指向它的底层 ArrayBuffer 对象。直接使用 ArrayBuffer 对象的原因是有时你可能想要有一个单一 buffer 的多个 typed array 视图。

## 11.2.3 使用 Typed Arrays

一旦你创建了一个 typed array，你可以用常规的方括号表示法读取和写入它的元素，就像你对待任何其他类似数组的对象一样：

```js
// Return the largest prime smaller than n, using the sieve of Eratosthenes
function sieve(n) {
    let a = new Uint8Array(n+1);         // a[x] will be 1 if x is composite
    let max = Math.floor(Math.sqrt(n));  // Don't do factors higher than this
    let p = 2;                           // 2 is the first prime
    while(p <= max) {                    // For primes less than max
        for(let i = 2*p; i <= n; i += p) // Mark multiples of p as composite
            a[i] = 1;
        while(a[++p]) /* empty */;       // The next unmarked index is prime
    }
    while(a[n]) n--;                     // Loop backward to find the last prime
    return n;                            // And return it
}
```

这里的函数计算比你指定的数字小的最大质数。代码与使用常规 JavaScript 数组完全相同，但在我的测试中使用 `Uint8Array()` 而不是 `Array()` 使代码运行速度超过四倍，并且使用的内存少了八倍。

Typed arrays 不是真正的数组，但它们重新实现了大多数数组方法，所以你可以几乎像使用常规数组一样使用它们：

```js
let ints = new Int16Array(10);       // 10 short integers
ints.fill(3).map(x=>x*x).join("")    // => "9999999999"
```

记住，typed arrays 有固定的长度，所以 `length` 属性是只读的，而改变数组长度的方法（如 `push()`、`pop()`、`unshift()`、`shift()` 和 `splice()`）对 typed arrays 没有实现。改变数组内容而不改变长度的方法（如 `sort()`、`reverse()` 和 `fill()`）是实现的。返回新数组的 `map()` 和 `slice()` 等方法返回与调用它们的 typed array 相同类型的 typed array。

## 11.2.4 Typed Array 方法和属性

除了标准数组方法外，typed arrays 也实现了一些自己的方法。`set()` 方法通过将常规或 typed array 的元素复制到 typed array 中一次设置多个元素：

```js
let bytes = new Uint8Array(1024);        // A 1K buffer
let pattern = new Uint8Array([0,1,2,3]); // An array of 4 bytes
bytes.set(pattern);      // Copy them to the start of another byte array
bytes.set(pattern, 4);   // Copy them again at a different offset
bytes.set([0,1,2,3], 8); // Or just copy values direct from a regular array
bytes.slice(0, 12)       // => new Uint8Array([0,1,2,3,0,1,2,3,0,1,2,3])
```

`set()` 方法以数组或 typed array 作为第一个参数，以元素偏移量作为可选的第二个参数，如果未指定则默认为 0。如果你从一个 typed array 复制值到另一个，这个操作可能会非常快。

Typed arrays 还有一个 `subarray` 方法，返回调用它的数组的一部分：

```js
let ints = new Int16Array([0,1,2,3,4,5,6,7,8,9]);       // 10 short integers
let last3 = ints.subarray(ints.length-3, ints.length);  // Last 3 of them
last3[0]       // => 7: this is the same as ints[7]
```

`subarray()`接受与`slice()`方法相同的参数，并且似乎工作方式相同。但有一个重要的区别。`slice()`返回一个新的、独立的类型化数组，其中包含指定的元素，不与原始数组共享内存。`subarray()`不复制任何内存；它只返回相同底层值的新视图：

```js
ints[9] = -1;  // Change a value in the original array and...
last3[2]       // => -1: it also changes in the subarray
```

`subarray()`方法返回现有数组的新视图，这让我们回到了 ArrayBuffers 的话题。每个类型化数组都有三个与底层缓冲区相关的属性：

```js
last3.buffer                 // The ArrayBuffer object for a typed array
last3.buffer === ints.buffer // => true: both are views of the same buffer
last3.byteOffset             // => 14: this view starts at byte 14 of the buffer
last3.byteLength             // => 6: this view is 6 bytes (3 16-bit ints) long
last3.buffer.byteLength      // => 20: but the underlying buffer has 20 bytes
```

`buffer`属性是数组的 ArrayBuffer。`byteOffset`是数组数据在底层缓冲区中的起始位置。`byteLength`是数组数据的字节长度。对于任何类型化数组`a`，这个不变式应该始终成立：

```js
a.length * a.BYTES_PER_ELEMENT === a.byteLength  // => true
```

ArrayBuffer 只是不透明的字节块。您可以使用类型化数组访问这些字节，但 ArrayBuffer 本身不是类型化数组。但要小心：您可以像在任何 JavaScript 对象上一样使用数字数组索引访问 ArrayBuffers。这样做并不会让您访问缓冲区中的字节，但可能会导致混乱的错误：

```js
let bytes = new Uint8Array(8);
bytes[0] = 1;           // Set the first byte to 1
bytes.buffer[0]         // => undefined: buffer doesn't have index 0
bytes.buffer[1] = 255;  // Try incorrectly to set a byte in the buffer
bytes.buffer[1]         // => 255: this just sets a regular JS property
bytes[1]                // => 0: the line above did not set the byte
```

我们之前看到，您可以使用`ArrayBuffer()`构造函数创建一个 ArrayBuffer，然后创建使用该缓冲区的类型化数组。另一种方法是创建一个初始类型化数组，然后使用该数组的缓冲区创建其他视图：

```js
let bytes = new Uint8Array(1024);            // 1024 bytes
let ints = new Uint32Array(bytes.buffer);    // or 256 integers
let floats = new Float64Array(bytes.buffer); // or 128 doubles
```

## 11.2.5 DataView 和字节顺序

类型化数组允许您以 8、16、32 或 64 位的块查看相同的字节序列。这暴露了“字节序”：字节被排列成更长字的顺序。为了效率，类型化数组使用底层硬件的本机字节顺序。在小端系统上，数字的字节从最不重要到最重要的顺序排列在 ArrayBuffer 中。在大端平台上，字节从最重要到最不重要的顺序排列。您可以使用以下代码确定底层平台的字节顺序：

```js
// If the integer 0x00000001 is arranged in memory as 01 00 00 00, then
// we're on a little-endian platform. On a big-endian platform, we'd get
// bytes 00 00 00 01 instead.
let littleEndian = new Int8Array(new Int32Array([1]).buffer)[0] === 1;
```

如今，最常见的 CPU 架构是小端。然而，许多网络协议和一些二进制文件格式要求大端字节顺序。如果您正在使用来自网络或文件的数据的类型化数组，您不能仅仅假设平台的字节顺序与数据的字节顺序相匹配。一般来说，在处理外部数据时，您可以使用 Int8Array 和 Uint8Array 将数据视为单个字节的数组，但不应使用其他具有多字节字长的类型化数组。相反，您可以使用 DataView 类，该类定义了用于从具有明确定义的字节顺序的 ArrayBuffer 中读取和写入值的方法：

```js
// Assume we have a typed array of bytes of binary data to process. First,
// we create a DataView object so we can flexibly read and write
// values from those bytes
let view = new DataView(bytes.buffer,
                        bytes.byteOffset,
                        bytes.byteLength);

let int = view.getInt32(0);     // Read big-endian signed int from byte 0
int = view.getInt32(4, false);  // Next int is also big-endian
int = view.getUint32(8, true);  // Next int is little-endian and unsigned
view.setUint32(8, int, false);  // Write it back in big-endian format
```

DataView 为每个 10 个类型化数组类定义了 10 个`get`方法（不包括 Uint8ClampedArray）。它们的名称类似于`getInt16()`、`getUint32()`、`getBigInt64()`和`getFloat64()`。第一个参数是 ArrayBuffer 中数值开始的字节偏移量。除了`getInt8()`和`getUint8()`之外，所有这些获取方法都接受一个可选的布尔值作为第二个参数。如果省略第二个参数或为`false`，则使用大端字节顺序。如果第二个参数为`true`，则使用小端顺序。

DataView 还定义了 10 个相应的 Set 方法，用于将值写入底层 ArrayBuffer。第一个参数是值开始的偏移量。第二个参数是要写入的值。除了`setInt8()`和`setUint8()`之外，每个方法都接受一个可选的第三个参数。如果省略参数或为`false`，则以大端格式写入值，最重要的字节在前。如果参数为`true`，则以小端格式写入值，最不重要的字节在前。

类型化数组和 DataView 类为您提供了处理二进制数据所需的所有工具，并使您能够编写执行诸如解压缩 ZIP 文件或从 JPEG 文件中提取元数据等操作的 JavaScript 程序。

# 11.3 使用正则表达式进行模式匹配

*正则表达式*是描述文本模式的对象。JavaScript RegExp 类表示正则表达式，String 和 RegExp 都定义了使用正则表达式执行强大的模式匹配和搜索替换功能的方法。然而，为了有效地使用 RegExp API，您还必须学习如何使用正则表达式语法描述文本模式，这本质上是一种自己的迷你编程语言。幸运的是，JavaScript 正则表达式语法与许多其他编程语言使用的语法非常相似，因此您可能已经熟悉它。 （如果您不熟悉，学习 JavaScript 正则表达式所投入的努力可能也对您在其他编程环境中有所帮助。）

接下来的小节首先描述了正则表达式语法，然后，在解释如何编写正则表达式之后，它们解释了如何使用它们与 String 和 RegExp 类的方法。

## 11.3.1 定义正则表达式

在 JavaScript 中，正则表达式由 RegExp 对象表示。当然，RegExp 对象可以使用`RegExp()`构造函数创建，但更常见的是使用特殊的字面量语法创建。正如字符串字面量是在引号内指定的字符一样，正则表达式字面量是在一对斜杠（`/`）字符内指定的字符。因此，您的 JavaScript 代码可能包含如下行：

```js
let pattern = /s$/;
```

此行创建一个新的 RegExp 对象，并将其赋给变量`pattern`。这个特定的 RegExp 对象匹配任何以字母“s”结尾的字符串。这个正则表达式也可以用`RegExp()`构造函数定义，就像这样：

```js
let pattern = new RegExp("s$");
```

正则表达式模式规范由一系列字符组成。大多数字符，包括所有字母数字字符，只是描述要匹配的字符。因此，正则表达式`/java/`匹配包含子字符串“java”的任何字符串。正则表达式中的其他字符不是字面匹配的，而是具有特殊意义。例如，正则表达式`/s$/`包含两个字符。第一个“s”是字面匹配的。第二个“$”是一个特殊的元字符，匹配字符串的结尾。因此，这个正则表达式匹配任何以字母“s”作为最后一个字符的字符串。

正如我们将看到的，正则表达式也可以有一个或多个标志字符，影响它们的工作方式。标志是在 RegExp 文本的第二个斜杠字符后指定的，或者作为`RegExp()`构造函数的第二个字符串参数。例如，如果我们想匹配以“s”或“S”结尾的字符串，我们可以在正则表达式中使用`i`标志，表示我们要进行不区分大小写的匹配：

```js
let pattern = /s$/i;
```

以下各节描述了 JavaScript 正则表达式中使用的各种字符和元字符。

### 字面字符

所有字母字符和数字在正则表达式中都以字面意义匹配自身。JavaScript 正则表达式语法还支持以反斜杠（`\`）开头的转义序列表示某些非字母字符。例如，序列`\n`在字符串中匹配一个字面换行符。表 11-1 列出了这些字符。

表 11-1\. 正则表达式字面字符

| 字符 | 匹配 |
| --- | --- |
| 字母数字字符 | 本身 |
| `\0` | NUL 字符（`\u0000`） |
| `\t` | 制表符（`\u0009`） |
| `\n` | 换行符（`\u000A`） |
| `\v` | 垂直制表符（`\u000B`） |
| `\f` | 换页符（`\u000C`） |
| `\r` | 回车符（`\u000D`） |
| `\x`*nn* | 十六进制数字 *nn* 指定的拉丁字符；例如，`\x0A` 等同于 `\n`。 |
| `\u`*xxxx* | 十六进制数字 *xxxx* 指定的 Unicode 字符；例如，`\u0009` 等同于 `\t`。 |
| `\u{`*n*`}` | 由代码点 *n* 指定的 Unicode 字符，其中 *n* 是 0 到 10FFFF 之间的一到六个十六进制数字。请注意，此语法仅在使用 `u` 标志的正则表达式中受支持。 |
| `\c`*X* | 控制字符 `^`*X*；例如，`\cJ` 等同于换行符 `\n`。 |

许多标点符号在正则表达式中具有特殊含义。它们是：

```js
^ $ . * + ? = ! : | \ / ( ) [ ] { }
```

这些字符的含义将在接下来的章节中讨论。其中一些字符仅在正则表达式的某些上下文中具有特殊含义，在其他上下文中被视为文字。然而，作为一般规则，如果要在正则表达式中字面包含任何这些标点符号，必须在其前面加上 `\`。其他标点符号，如引号和 `@`，没有特殊含义，只是在正则表达式中字面匹配自身。

如果你记不清哪些标点符号需要用反斜杠转义，你可以安全地在任何标点符号前面放置一个反斜杠。另一方面，请注意，许多字母和数字在前面加上反斜杠时具有特殊含义，因此任何你想字面匹配的字母或数字不应该用反斜杠转义。要在正则表达式中字面包含反斜杠字符，当然必须用反斜杠转义它。例如，以下正则表达式匹配包含反斜杠的任意字符串：`/\\/`。（如果你使用 `RegExp()` 构造函数，请记住你的正则表达式中的任何反斜杠都需要加倍，因为字符串也使用反斜杠作为转义字符。）

### 字符类

通过将单个文字字符组合到方括号中，可以形成*字符类*。字符类匹配其中包含的任意一个字符。因此，正则表达式 `/[abc]/` 匹配字母 a、b 或 c 中的任意一个。也可以定义否定字符类；这些匹配除方括号中包含的字符之外的任意字符。否定字符类通过在左方括号内的第一个字符处放置插入符号（`^`）来指定。正则表达式 `/[^abc]/` 匹配除 a、b 或 c 之外的任意一个字符。字符类可以使用连字符指示字符范围。要匹配拉丁字母表中的任意一个小写字母，请使用 `/[a-z]/`，要匹配拉丁字母表中的任意字母或数字，请使用 `/[a-zA-Z0-9]/`。（如果要在字符类中包含实际连字符，只需将其放在右方括号之前。）

由于某些字符类通常被使用，JavaScript 正则表达式语法包括特殊字符和转义序列来表示这些常见类。例如，`\s` 匹配空格字符、制表符和任何其他 Unicode 空白字符；`\S` 匹配任何*非* Unicode 空白字符。表 11-2 列出了这些字符并总结了字符类语法。（请注意，其中几个字符类转义序列仅匹配 ASCII 字符，并未扩展为适用于 Unicode 字符。但是，你可以显式定义自己的 Unicode 字符类；例如，`/[\u0400-\u04FF]/` 匹配任意一个西里尔字母字符。）

表格 11-2\. 正则表达式字符类

| 字符 | 匹配 |
| --- | --- |
| `[...]` | 方括号内的任意一个字符。 |
| `[^...]` | 方括号内的任意一个字符。 |
| `.` | 除换行符或其他 Unicode 行终止符之外的任何字符。或者，如果 RegExp 使用 `s` 标志，则句点匹配任何字符，包括行终止符。 |
| `\w` | 任何 ASCII 单词字符。等同于 `[a-zA-Z0-9_]`。 |
| `\W` | 任何不是 ASCII 单词字符的字符。等同于 `[^a-zA-Z0-9_]`。 |
| `\s` | 任何 Unicode 空白字符。 |
| `\S` | 任何不是 Unicode 空白字符的字符。 |
| `\d` | 任何 ASCII 数字。等同于 `[0-9]`。 |
| `\D` | 任何 ASCII 数字之外的字符。等同于 `[⁰-9]`。 |
| `[\b]` | 一个字面退格（特殊情况）。 |

请注意，特殊字符类转义可以在方括号内使用。`\s` 匹配任何空白字符，`\d` 匹配任何数字，因此 `/[\s\d]/` 匹配任何一个空白字符或数字。请注意有一个特殊情况。正如稍后将看到的，`\b` 转义具有特殊含义。但是，在字符类中使用时，它表示退格字符。因此，要在正则表达式中字面表示退格字符，请使用具有一个元素的字符类：`/[\b]/`。

### 重复

到目前为止，您学到的正则表达式语法可以将两位数描述为 `/\d\d/`，将四位数描述为 `/\d\d\d\d/`。但是，您没有任何方法来描述，例如，可以具有任意数量的数字或三个字母后跟一个可选数字的字符串。这些更复杂的模式使用指定正则表达式元素可以重复多少次的正则表达式语法。

指定重复的字符始终跟随其应用的模式。由于某些类型的重复非常常见，因此有特殊字符来表示这些情况。例如，`+` 匹配前一个模式的一个或多个出现。

表 11-3 总结了重复语法。

表 11-3\. 正则表达式重复字符

| 字符 | 含义 |
| --- | --- |
| `{`*n*`,`*m*`}` | 匹配前一个项目至少 *n* 次但不超过 *m* 次。 |
| `{`*n*`,}` | 匹配前一个项目 *n* 次或更多次。 |
| `{`*n*`}` | 匹配前一个项目的 *n* 次出现。 |
| `?` | 匹配前一个项目的零次或一次出现。也就是说，前一个项目是可选的。等同于 `{0,1}`。 |
| `+` | 匹配前一个项目的一个或多个出现。等同于 `{1,}`。 |
| `*` | 匹配前一个项目的零次或多次。等同于 `{0,}`。 |

以下行显示了一些示例：

```js
let r = /\d{2,4}/; // Match between two and four digits
r = /\w{3}\d?/;    // Match exactly three word characters and an optional digit
r = /\s+java\s+/;  // Match "java" with one or more spaces before and after
r = /[^(]*/;       // Match zero or more characters that are not open parens
```

请注意，在所有这些示例中，重复说明符应用于它们之前的单个字符或字符类。如果要匹配更复杂表达式的重复，您需要使用括号定义一个组，这将在以下部分中解释。

使用 `*` 和 `?` 重复字符时要小心。由于这些字符可能匹配前面的内容的零次，它们允许匹配空内容。例如，正则表达式 `/a*/` 实际上匹配字符串“bbbb”，因为该字符串不包含字母 a 的任何出现！

### 非贪婪重复

表 11-3 中列出的重复字符尽可能多次匹配，同时仍允许正则表达式的任何后续部分匹配。我们说这种重复是“贪婪的”。还可以指定以非贪婪方式进行重复。只需在重复字符后面跟一个问号：`??`、`+?`、`*?`，甚至 `{1,5}?`。例如，正则表达式 `/a+/` 匹配一个或多个字母 a 的出现。当应用于字符串“aaa”时，它匹配所有三个字母。但是 `/a+?/` 匹配一个或多个字母 a 的出现，尽可能少地匹配字符。当应用于相同字符串时，此模式仅匹配第一个字母 a。

使用非贪婪重复可能不总是产生您期望的结果。考虑模式`/a+b/`，它匹配一个或多个 a，后跟字母 b。当应用于字符串“aaab”时，它匹配整个字符串。现在让我们使用非贪婪版本：`/a+?b/`。这应该匹配由尽可能少的 a 前导的字母 b。当应用于相同字符串“aaab”时，您可能希望它仅匹配一个 a 和最后一个字母 b。但实际上，此模式与贪婪版本的模式一样匹配整个字符串。这是因为正则表达式模式匹配是通过找到字符串中可能发生匹配的第一个位置来完成的。由于从字符串的第一个字符开始就可能发生匹配，因此从后续字符开始的较短匹配甚至不会被考虑。

### 备选项、分组和引用

正则表达式语法包括用于指定备选项、分组子表达式和引用先前子表达式的特殊字符。`|`字符分隔备选项。例如，`/ab|cd|ef/`匹配字符串“ab”或字符串“cd”或字符串“ef”。而`/\d{3}|[a-z]{4}/`匹配三个数字或四个小写字母中的任何一个。

请注意，备选项从左到右考虑，直到找到匹配项。如果左侧备选项匹配，则右侧备选项将被忽略，即使它可能产生“更好”的匹配。因此，当将模式`/a|ab/`应用于字符串“ab”时，它仅匹配第一个字母。

括号在正则表达式中有几个目的。一个目的是将单独的项目分组为单个子表达式，以便可以通过`|`、`*`、`+`、`?`等将项目视为单个单元。例如，`/java(script)?/`匹配“java”后跟可选的“script”。而`/(ab|cd)+|ef/`匹配字符串“ef”或一个或多个重复的字符串“ab”或“cd”中的任何一个。

正则表达式中括号的另一个目的是在完整模式内定义子模式。当正则表达式成功匹配目标字符串时，可以提取匹配任何特定括号子模式的目标字符串部分。（您将在本节后面看到如何获取这些匹配的子字符串。）例如，假设您正在寻找一个或多个小写字母后跟一个或多个数字。您可能会使用模式`/[a-z]+\d+/`。但是假设您只关心每个匹配末尾的数字。如果将模式的这部分放在括号中`(/[a-z]+(\d+)/)`，您可以提取任何找到的匹配中的数字，如后面所述。

括号子表达式的一个相关用途是允许您在同一正则表达式中稍后引用子表达式。这是通过在`\`字符后跟一个或多个数字来完成的。这些数字指的是正则表达式中括号子表达式的位置。例如，`\1`引用第一个子表达式，`\3`引用第三个。请注意，由于子表达式可以嵌套在其他子表达式中，因此计算的是左括号的位置。例如，在以下正则表达式中，嵌套的子表达式`([Ss]cript)`被称为`\2`：

```js
/([Jj]ava([Ss]cript)?)\sis\s(fun\w*)/
```

对正则表达式的先前子表达式的引用*不*是指该子表达式的模式，而是指匹配该模式的文本。因此，引用可用于强制要求字符串的不同部分包含完全相同的字符。例如，以下正则表达式匹配单引号或双引号内的零个或多个字符。但是，它不要求开头和结尾引号匹配（即，都是单引号或双引号）：

```js
/['"][^'"]*['"]/
```

要求引号匹配，请使用引用：

```js
/(['"])[^'"]*\1/
```

`\1`匹配第一个括号子表达式匹配的内容。在此示例中，它强制约束闭合引号与开放引号匹配。此正则表达式不允许单引号在双引号字符串内部，反之亦然。（在字符类内部使用引用是不合法的，因此不能写成：`/(['"])[^\1]*\1/`。）

当我们稍后讨论 RegExp API 时，您会看到对括号子表达式的引用是正则表达式搜索和替换操作的一个强大功能。

也可以在正则表达式中分组项目而不创建对这些项目的编号引用。不要简单地在`(`和`)`内部分组项目，而是从`(?:`开始组，以`)`结束。考虑以下模式：

```js
/([Jj]ava(?:[Ss]cript)?)\sis\s(fun\w*)/
```

在此示例中，子表达式`(?:[Ss]cript)`仅用于分组，因此`?`重复字符可以应用于该组。这些修改后的括号不生成引用，因此在此正则表达式中，`\2`指的是由`(fun\w*)`匹配的文本。

表格 11-4 总结了正则表达式的交替、分组和引用操作符。

表格 11-4\. 正则表达式的交替、分组和引用字符

| 字符 | 含义 |
| --- | --- |
| `&#124;` | 交替：匹配左侧子表达式或右侧子表达式。 |
| `(...)` | 分组：将项目分组为一个单元，可以与`*`、`+`、`?`、`&#124;`等一起使用。还要记住匹配此组的字符，以便后续引用。 |
| `(?:...)` | 仅分组：将项目分组为一个单元，但不记住匹配此组的字符。 |
| `\`*n* | 匹配在第一次匹配组号 *n* 时匹配的相同字符。组是括号内的子表达式（可能是嵌套的）。组号是通过从左到右计算左括号来分配的。使用`(?:`形成的组不编号。 |

### 指定匹配位置

正如前面所述，正则表达式的许多元素匹配字符串中的单个字符。例如，`\s`匹配单个空白字符。其他正则表达式元素匹配字符之间的位置而不是实际字符。例如，`\b`匹配 ASCII 单词边界——`\w`（ASCII 单词字符）和`\W`（非单词字符）之间的边界，或者 ASCII 单词字符和字符串的开头或结尾之间的边界。[⁴] 元素如`\b`不指定要在匹配的字符串中使用的任何字符；但它们指定的是合法的匹配位置。有时这些元素被称为*正则表达式锚点*，因为它们将模式锚定到搜索字符串中的特定位置。最常用的锚定元素是`^`，将模式绑定到字符串的开头，以及`$`，将模式锚定到字符串的结尾。

例如，要匹配单独一行的单词“JavaScript”，可以使用正则表达式`/^JavaScript$/`。如果要搜索“Java”作为单独的单词（而不是作为“JavaScript”中的前缀），可以尝试模式`/\sJava\s/`，这需要单词前后有空格。但是这种解决方案有两个问题。首先，它不匹配字符串的开头或结尾的“Java”，而只有在两侧有空格时才匹配。其次，当此模式找到匹配时，返回的匹配字符串具有前导和尾随空格，这不是所需的。因此，与其用`\s`匹配实际空格字符，不如用`\b`匹配（或锚定）单词边界。得到的表达式是`/\bJava\b/`。元素`\B`将匹配锚定到不是单词边界的位置。因此，模式`/\B[Ss]cript/`匹配“JavaScript”和“postscript”，但不匹配“script”或“Scripting”。

您还可以使用任意正则表达式作为锚定条件。如果在`(?=`和`)`字符之间包含一个表达式，那么这是一个前瞻断言，并且它指定封闭字符必须匹配，而不实际匹配它们。例如，要匹配一个常见编程语言的名称，但只有在后面跟着一个冒号时，您可以使用`/[Jj]ava([Ss]cript)?(?=\:)/`。这个模式匹配“JavaScript”中的单词“JavaScript: The Definitive Guide”，但不匹配“Java in a Nutshell”中的“Java”，因为它后面没有跟着冒号。

如果您使用`(?!`引入断言，那么这是一个负向前瞻断言，指定接下来的字符不得匹配。例如，`/Java(?!Script)([A-Z]\w*)/`匹配“Java”后跟一个大写字母和任意数量的其他 ASCII 单词字符，只要“Java”后面不跟着“Script”。它匹配“JavaBeans”但不匹配“Javanese”，它匹配“JavaScrip”但不匹配“JavaScript”或“JavaScripter”。表 11-5 总结了正则表达式锚点。

表 11-5\. 正则表达式锚点字符

| 字符 | 含义 |
| --- | --- |
| `^` | 匹配字符串的开头或者在使用`m`标志时，匹配行的开头。 |
| `$` | 匹配字符串的结尾，并且在使用`m`标志时，匹配行的结尾。 |
| `\b` | 匹配单词边界。也就是说，匹配`\w`字符和`\W`字符之间的位置，或者匹配`\w`字符和字符串的开头或结尾之间的位置。（但请注意，`[\b]`匹配退格键。） |
| `\B` | 匹配不是单词边界的位置。 |
| `(?=`*p*`)` | 正向前瞻断言。要求接下来的字符匹配模式*p*，但不包括这些字符在匹配中。 |
| `(?!`*p*`)` | 负向前瞻断言。要求接下来的字符不匹配模式*p*。 |

### 标志

每个正则表达式都可以有一个或多个与之关联的标志，以改变其匹配行为。JavaScript 定义了六个可能的标志，每个标志由一个字母表示。标志在正则表达式字面量的第二个`/`字符之后指定，或者作为传递给`RegExp()`构造函数的第二个参数的字符串。支持的标志及其含义如下：

`g`

`g`标志表示正则表达式是“全局”的，也就是说，我们打算在字符串中找到所有匹配项，而不仅仅是找到第一个匹配项。这个标志不会改变匹配模式的方式，但正如我们稍后将看到的，它确实以重要的方式改变了 String `match()`方法和 RegExp `exec()`方法的行为。

`i`

`i`标志指定匹配模式时应该忽略大小写。

`m`

`m`标志指定匹配应该在“多行”模式下进行。它表示正则表达式将与多行字符串一起使用，并且`^`和`$`锚点应该匹配字符串的开头和结尾，以及字符串中各行的开头和结尾。

`s`

与`m`标志类似，`s`标志在处理包含换行符的文本时也很有用。通常，正则表达式中的“.”匹配除行终止符之外的任何字符。但是，当使用`s`标志时，“.”将匹配任何字符，包括行终止符。`s`标志在 ES2018 中添加到 JavaScript 中，并且截至 2020 年初，在 Node、Chrome、Edge 和 Safari 中支持，但在 Firefox 中不支持。

`u`

`u`标志代表 Unicode，它使正则表达式匹配完整的 Unicode 代码点，而不是匹配 16 位值。这个标志是在 ES6 中引入的，你应该养成在所有正则表达式上使用它的习惯，除非你有某种理由不这样做。如果你不使用这个标志，那么你的正则表达式将无法很好地处理包含表情符号和其他需要超过 16 位的字符（包括许多中文字符）的文本。没有`u`标志，"."字符匹配任何 1 个 UTF-16 16 位值。然而，有了这个标志，"."匹配一个 Unicode 代码点，包括那些超过 16 位的代码点。在正则表达式上设置`u`标志还允许你使用新的`\u{...}`转义序列来表示 Unicode 字符，并且还启用了`\p{...}`表示 Unicode 字符类。

`y`

`y`标志表示正则表达式是“粘性”的，应该在字符串的开头或上一个匹配项后的第一个字符处匹配。当与旨在找到单个匹配项的正则表达式一起使用时，它有效地将该正则表达式视为以`^`开头以将其锚定到字符串开头。这个标志在重复使用用于在字符串中找到所有匹配项的正则表达式时更有用。在这种情况下，它导致 String `match()`方法和 RegExp `exec()`方法的特殊行为，以强制每个后续匹配项都锚定到上一个匹配项结束的字符串位置。

这些标志可以以任何组合和任何顺序指定。例如，如果你希望你的正则表达式能够识别 Unicode 以进行不区分大小写的匹配，并且打算在字符串中查找多个匹配项，你可以指定标志`uig`，`gui`或这三个字母的任何其他排列。

## 11.3.2 用于模式匹配的字符串方法

到目前为止，我们一直在描述用于定义正则表达式的语法，但没有解释这些正则表达式如何在 JavaScript 代码中实际使用。我们现在转而介绍使用 RegExp 对象的 API。本节首先解释了使用正则表达式执行模式匹配和搜索替换操作的字符串方法。接下来的部分将继续讨论使用 JavaScript 正则表达式进行模式匹配，讨论 RegExp 对象及其方法和属性。

### search()

字符串支持四种使用正则表达式的方法。最简单的是`search()`。这个方法接受一个正则表达式参数，并返回第一个匹配子字符串的起始字符位置，如果没有匹配则返回-1：

```js
"JavaScript".search(/script/ui)  // => 4
"Python".search(/script/ui)      // => -1
```

如果`search()`的参数不是正则表达式，则首先通过将其传递给`RegExp`构造函数将其转换为正则表达式。`search()`不支持全局搜索；它会忽略其正则表达式参数的`g`标志。

### replace()

`replace()`方法执行搜索替换操作。它将正则表达式作为第一个参数，替换字符串作为第二个参数。它在调用它的字符串中搜索与指定模式匹配的内容。如果正则表达式设置了`g`标志，`replace()`方法将在字符串中替换所有匹配项为替换字符串；否则，它只会替换找到的第一个匹配项。如果`replace()`的第一个参数是一个字符串而不是正则表达式，该方法会直接搜索该字符串而不是像`search()`那样将其转换为正则表达式。例如，你可以使用`replace()`如下提供文本字符串中“JavaScript”一词的统一大写格式：

```js
// No matter how it is capitalized, replace it with the correct capitalization
text.replace(/javascript/gi, "JavaScript");
```

然而，`replace()`比这更强大。回想一下，正则表达式的括号子表达式从左到右编号，并且正则表达式记住了每个子表达式匹配的文本。如果替换字符串中出现了`$`后跟一个数字，`replace()`将用指定子表达式匹配的文本替换这两个字符。这是一个非常有用的功能。例如，你可以使用它将字符串中的引号替换为其他字符：

```js
// A quote is a quotation mark, followed by any number of
// nonquotation mark characters (which we capture), followed
// by another quotation mark.
let quote = /"([^"]*)"/g;
// Replace the straight quotation marks with guillemets
// leaving the quoted text (stored in $1) unchanged.
'He said "stop"'.replace(quote, '«$1»')  // => 'He said «stop»'
```

如果你的正则表达式使用了命名捕获组，那么你可以通过名称而不是数字引用匹配的文本：

```js
let quote = /"(?<quotedText>[^"]*)"/g;
'He said "stop"'.replace(quote, '«$<quotedText>»')  // => 'He said «stop»'
```

不需要将替换字符串作为第二个参数传递给`replace()`，你也可以传递一个函数作为替换值的计算方法。替换函数会被调用并传入多个参数。首先是整个匹配的文本。接下来，如果正则表达式有捕获组，那么被这些组捕获的子字符串将作为参数传递。下一个参数是匹配被找到的字符串中的位置。之后，调用`replace()`的整个字符串也会被传递。最后，如果正则表达式包含任何命名捕获组，替换函数的最后一个参数是一个对象，其属性名与捕获组名匹配，值为匹配的文本。例如，这里是使用替换函数将字符串中的十进制整数转换为十六进制的代码：

```js
let s = "15 times 15 is 225";
s.replace(/\d+/gu, n => parseInt(n).toString(16))  // => "f times f is e1"
```

### match()

`match()`方法是 String 正则表达式方法中最通用的。它将正则表达式作为唯一参数（或通过将其传递给`RegExp()`构造函数将其参数转换为正则表达式）并返回一个包含匹配结果的数组，如果没有找到匹配则返回`null`。如果正则表达式设置了`g`标志，该方法将返回出现在字符串中的所有匹配项的数组。例如：

```js
"7 plus 8 equals 15".match(/\d+/g)  // => ["7", "8", "15"]
```

如果正则表达式没有设置`g`标志，`match()`不会进行全局搜索；它只是搜索第一个匹配项。在这种非全局情况下，`match()`仍然返回一个数组，但数组元素完全不同。没有`g`标志时，返回数组的第一个元素是匹配的字符串，任何剩余的元素是正则表达式中括号捕获组匹配的子字符串。因此，如果`match()`返回一个数组`a`，`a[0]`包含完整匹配，`a[1]`包含匹配第一个括号表达式的子字符串，依此类推。与`replace()`方法类比，`a[1]`与`$1`相同，`a[2]`与`$2`相同，依此类推。

例如，考虑使用以下代码解析 URL⁵：

```js
// A very simple URL parsing RegExp
let url = /(\w+):\/\/([\w.]+)\/(\S*)/;
let text = "Visit my blog at http://www.example.com/~david";
let match = text.match(url);
let fullurl, protocol, host, path;
if (match !== null) {
    fullurl = match[0];   // fullurl == "http://www.example.com/~david"
    protocol = match[1];  // protocol == "http"
    host = match[2];      // host == "www.example.com"
    path = match[3];      // path == "~david"
}
```

在这种非全局情况下，`match()`返回的数组除了编号数组元素外还有一些对象属性。`input`属性指的是调用`match()`的字符串。`index`属性是匹配开始的字符串位置。如果正则表达式包含命名捕获组，那么返回的数组还有一个`groups`属性，其值是一个对象。这个对象的属性与命名组的名称匹配，值为匹配的文本。例如，我们可以像这样重新编写之前的 URL 解析示例：

```js
let url = /(?<protocol>\w+):\/\/(?<host>[\w.]+)\/(?<path>\S*)/;
let text = "Visit my blog at http://www.example.com/~david";
let match = text.match(url);
match[0]               // => "http://www.example.com/~david"
match.input            // => text
match.index            // => 17
match.groups.protocol  // => "http"
match.groups.host      // => "www.example.com"
match.groups.path      // => "~david"
```

我们已经看到，`match()` 的行为在正则表达式是否设置了 `g` 标志时会有很大不同。当设置了 `y` 标志时，行为也会有重要但不那么显著的差异。请记住，`y` 标志通过限制匹配开始的位置使正则表达式“粘滞”。如果一个正则表达式同时设置了 `g` 和 `y` 标志，那么 `match()` 返回一个匹配字符串的数组，就像在设置了 `g` 而没有设置 `y` 时一样。但第一个匹配必须从字符串的开头开始，每个后续匹配必须从前一个匹配的字符紧随其后开始。

如果设置了 `y` 标志但没有设置 `g`，那么 `match()` 会尝试找到单个匹配，并且默认情况下，此匹配受限于字符串的开头。然而，您可以通过设置 RegExp 对象的 `lastIndex` 属性来更改此默认匹配开始位置，指定要匹配的索引位置。如果找到匹配，那么 `lastIndex` 将自动更新为匹配后的第一个字符，因此如果再次调用 `match()`，它将寻找下一个匹配。(`lastIndex` 可能看起来是一个奇怪的属性名称，它指定开始 *下一个* 匹配的位置。当我们讨论 RegExp `exec()` 方法时，我们将再次看到它，这个名称在那种情况下可能更有意义。)

```js
let vowel = /[aeiou]/y;  // Sticky vowel match
"test".match(vowel)      // => null: "test" does not begin with a vowel
vowel.lastIndex = 1;     // Specify a different match position
"test".match(vowel)[0]   // => "e": we found a vowel at position 1
vowel.lastIndex          // => 2: lastIndex was automatically updated
"test".match(vowel)      // => null: no vowel at position 2
vowel.lastIndex          // => 0: lastIndex gets reset after failed match
```

值得注意的是，将非全局正则表达式传递给字符串的 `match()` 方法与将字符串传递给正则表达式的 `exec()` 方法是相同的：返回的数组及其属性在这两种情况下都是相同的。

### matchAll()

`matchAll()` 方法在 ES2020 中定义，并且在 2020 年初已被现代 Web 浏览器和 Node 实现。`matchAll()` 期望一个设置了 `g` 标志的正则表达式。然而，与 `match()` 返回匹配子字符串的数组不同，它返回一个迭代器，该迭代器产生与使用非全局 RegExp 时 `match()` 返回的匹配对象相同的对象。这使得 `matchAll()` 成为遍历字符串中所有匹配的最简单和最通用的方法。

您可以使用 `matchAll()` 遍历文本字符串中的单词，如下所示：

```js
// One or more Unicode alphabetic characters between word boundaries
const words = /\b\p{Alphabetic}+\b/gu; // \p is not supported in Firefox yet
const text = "This is a naïve test of the matchAll() method.";
for(let word of text.matchAll(words)) {
    console.log(`Found '${word[0]}' at index ${word.index}.`);
}
```

您可以设置 RegExp 对象的 `lastIndex` 属性，告诉 `matchAll()` 在字符串中的哪个索引开始匹配。然而，与其他模式匹配方法不同，`matchAll()` 永远不会修改您调用它的 RegExp 的 `lastIndex` 属性，这使得它在您的代码中更不容易出错。

### split()

String 对象的正则表达式方法中的最后一个是 `split()`。这个方法将调用它的字符串分割成一个子字符串数组，使用参数作为分隔符。它可以像这样使用一个字符串参数：

```js
"123,456,789".split(",")           // => ["123", "456", "789"]
```

`split()` 方法也可以接受正则表达式作为参数，这样可以指定更通用的分隔符。在这里，我们使用一个包含任意数量空白的分隔符来调用它：

```js
"1, 2, 3,\n4, 5".split(/\s*,\s*/)  // => ["1", "2", "3", "4", "5"]
```

令人惊讶的是，如果你使用包含捕获组的正则表达式分隔符调用 `split()`，那么匹配捕获组的文本将包含在返回的数组中。例如：

```js
const htmlTag = /<([^>]+)>/;  // < followed by one or more non->, followed by >
"Testing<br/>1,2,3".split(htmlTag)  // => ["Testing", "br/", "1,2,3"]
```

## 11.3.3 RegExp 类

本节介绍了 `RegExp()` 构造函数、RegExp 实例的属性以及 RegExp 类定义的两个重要模式匹配方法。

`RegExp()` 构造函数接受一个或两个字符串参数，并创建一个新的 RegExp 对象。这个构造函数的第一个参数是一个包含正则表达式主体的字符串——在正则表达式字面量中出现在斜杠内的文本。请注意，字符串字面量和正则表达式都使用 `\` 字符作为转义序列，因此当您将正则表达式作为字符串字面量传递给 `RegExp()` 时，必须将每个 `\` 字符替换为 `\\`。`RegExp()` 的第二个参数是可选的。如果提供，它表示正则表达式的标志。它应该是 `g`、`i`、`m`、`s`、`u`、`y`，或这些字母的任意组合。

例如：

```js
// Find all five-digit numbers in a string. Note the double \\ in this case.
let zipcode = new RegExp("\\d{5}", "g");
```

`RegExp()` 构造函数在动态创建正则表达式时非常有用，因此无法使用正则表达式字面量语法表示。例如，要搜索用户输入的字符串，必须在运行时使用 `RegExp()` 创建正则表达式。

除了将字符串作为 `RegExp()` 的第一个参数传递之外，您还可以传递一个 RegExp 对象。这允许您复制正则表达式并更改其标志：

```js
let exactMatch = /JavaScript/;
let caseInsensitive = new RegExp(exactMatch, "i");
```

### RegExp 属性

RegExp 对象具有以下属性：

`source`

这是正则表达式的源文本的只读属性：在 RegExp 字面量中出现在斜杠之间的字符。

`flags`

这是一个只读属性，指定表示 RegExp 标志的字母集合的字符串。

`global`

一个只读的布尔属性，如果设置了 `g` 标志，则为 true。

`ignoreCase`

一个只读的布尔属性，如果设置了 `i` 标志，则为 true。

`multiline`

一个只读的布尔属性，如果设置了 `m` 标志，则为 true。

`dotAll`

一个只读的布尔属性，如果设置了 `s` 标志，则为 true。

`unicode`

一个只读的布尔属性，如果设置了 `u` 标志，则为 true。

`sticky`

一个只读的布尔属性，如果设置了 `y` 标志，则为 true。

`lastIndex`

这个属性是一个读/写整数。对于具有 `g` 或 `y` 标志的模式，它指定下一次搜索开始的字符位置。它由 `exec()` 和 `test()` 方法使用，这两个方法在下面的两个小节中描述。

### test()

RegExp 类的 `test()` 方法是使用正则表达式的最简单的方法。它接受一个字符串参数，并在字符串与模式匹配时返回 `true`，否则返回 `false`。

`test()` 的工作原理是简单地调用（更复杂的）下一节中描述的 `exec()` 方法，并在 `exec()` 返回非空值时返回 `true`。因此，如果您使用带有 `g` 或 `y` 标志的 RegExp 来使用 `test()`，那么它的行为取决于 RegExp 对象的 `lastIndex` 属性的值，这个值可能会意外更改。有关更多详细信息，请参阅“lastIndex 属性和 RegExp 重用”。

### exec()

RegExp `exec()` 方法是使用正则表达式的最通用和强大的方式。它接受一个字符串参数，并在该字符串中查找匹配项。如果找不到匹配项，则返回 `null`。但是，如果找到匹配项，则返回一个数组，就像对于非全局搜索的 `match()` 方法返回的数组一样。数组的第 0 个元素包含与正则表达式匹配的字符串，任何后续的数组元素包含与任何捕获组匹配的子字符串。返回的数组还具有命名属性：`index` 属性包含匹配发生的字符位置，`input` 属性指定被搜索的字符串，如果定义了 `groups` 属性，则指的是一个保存与任何命名捕获组匹配的子字符串的对象。

与 String 的 `match()` 方法不同，`exec()` 无论正则表达式是否有全局 `g` 标志，都返回相同类型的数组。回想一下，当传递一个全局正则表达式时，`match()` 返回一个匹配数组。相比之下，`exec()` 总是返回一个单一匹配，并提供关于该匹配的完整信息。当在具有全局 `g` 标志或粘性 `y` 标志的正则表达式上调用 `exec()` 时，它会查看 RegExp 对象的 `lastIndex` 属性，以确定从哪里开始查找匹配。如果设置了 `y` 标志，它还会限制匹配从该位置开始。对于新创建的 RegExp 对象，`lastIndex` 为 0，并且搜索从字符串的开头开始。但每次 `exec()` 成功找到一个匹配时，它会更新 `lastIndex` 属性为匹配文本后面的字符的索引。如果 `exec()` 未找到匹配，它会将 `lastIndex` 重置为 0。这种特殊行为允许你重复调用 `exec()` 以循环遍历字符串中的所有正则表达式匹配。例如，以下代码中的循环将运行两次：

```js
let pattern = /Java/g;
let text = "JavaScript > Java";
let match;
while((match = pattern.exec(text)) !== null) {
    console.log(`Matched ${match[0]} at ${match.index}`);
    console.log(`Next search begins at ${pattern.lastIndex}`);
}
```

# 11.4 日期和时间

Date 类是 JavaScript 用于处理日期和时间的 API。使用 `Date()` 构造函数创建一个 Date 对象。如果没有参数，它会返回一个代表当前日期和时间的 Date 对象：

```js
let now = new Date();     // The current time
```

如果你传递一个数字参数，`Date()` 构造函数会将该参数解释为自 1970 年起的毫秒数：

```js
let epoch = new Date(0);  // Midnight, January 1st, 1970, GMT
```

如果你指定两个或更多整数参数，它们会被解释为年、月、日、小时、分钟、秒和毫秒，使用你的本地时区，如下所示：

```js
let century = new Date(2100,         // Year 2100
                       0,            // January
                       1,            // 1st
                       2, 3, 4, 5);  // 02:03:04.005, local time
```

Date API 的一个怪癖是，一年中的第一个月是数字 0，但一个月中的第一天是数字 1。如果省略时间字段，`Date()` 构造函数会将它们全部默认为 0，将时间设置为午夜。

请注意，当使用多个数字调用 `Date()` 构造函数时，它会使用本地计算机设置的任何时区进行解释。如果你想在 UTC（协调世界时，又称 GMT）中指定日期和时间，那么你可以使用 `Date.UTC()`。这个静态方法接受与 `Date()` 构造函数相同的参数，在 UTC 中解释它们，并返回一个毫秒时间戳，你可以传递给 `Date()` 构造函数：

```js
// Midnight in England, January 1, 2100
let century = new Date(Date.UTC(2100, 0, 1));
```

如果你打印一个日期（例如使用 `console.log(century)`），默认情况下会以你的本地时区打印。如果你想在 UTC 中显示一个日期，你应该明确地将其转换为字符串，使用 `toUTCString()` 或 `toISOString()`。

最后，如果你将一个字符串传递给 `Date()` 构造函数，它将尝试将该字符串解析为日期和时间规范。构造函数可以解析由 `toString()`、`toUTCString()` 和 `toISOString()` 方法生成的格式指定的日期：

```js
let century = new Date("2100-01-01T00:00:00Z");  // An ISO format date
```

一旦你有了一个 Date 对象，各种获取和设置方法允许你查询和修改 Date 的年、月、日、小时、分钟、秒和毫秒字段。每个方法都有两种形式：一种使用本地时间进行获取或设置，另一种使用 UTC 时间进行获取或设置。例如，要获取或设置 Date 对象的年份，你可以使用 `getFullYear()`、`getUTCFullYear()`、`setFullYear()` 或 `setUTCFullYear()`：

```js
let d = new Date();                  // Start with the current date
d.setFullYear(d.getFullYear() + 1);  // Increment the year
```

要获取或设置 Date 的其他字段，将方法名称中的“FullYear”替换为“Month”、“Date”、“Hours”、“Minutes”、“Seconds”或“Milliseconds”。一些日期设置方法允许你一次设置多个字段。`setFullYear()` 和 `setUTCFullYear()` 还可选择设置月份和日期。而 `setHours()` 和 `setUTCHours()` 还允许你指定分钟、秒和毫秒字段，除了小时字段。

请注意，查询日期的方法是`getDate()`和`getUTCDate()`。更自然的函数`getDay()`和`getUTCDay()`返回星期几（星期日为 0，星期六为 6）。星期几是只读的，因此没有相应的`setDay()`方法。

## 11.4.1 时间戳

JavaScript 将日期内部表示为整数，指定自 1970 年 1 月 1 日午夜（或之前）以来的毫秒数。支持的整数最大为 8,640,000,000,000,000，因此 JavaScript 在 270,000 年后不会用尽毫秒。

对于任何日期对象，`getTime()`方法返回内部值，而`setTime()`方法设置它。因此，您可以像这样为日期添加 30 秒：

```js
d.setTime(d.getTime() + 30000);
```

这些毫秒值有时被称为*时间戳*，直接使用它们而不是 Date 对象有时很有用。静态的`Date.now()`方法返回当前时间作为时间戳，当您想要测量代码运行时间时很有帮助：

```js
let startTime = Date.now();
reticulateSplines(); // Do some time-consuming operation
let endTime = Date.now();
console.log(`Spline reticulation took ${endTime - startTime}ms.`);
```

## 11.4.2 日期算术

可以使用 JavaScript 的标准`<`、`<=`、`>`和`>=`比较运算符比较日期对象。您可以从一个日期对象中减去另一个日期对象以确定两个日期之间的毫秒数。（这是因为 Date 类定义了一个返回时间戳的`valueOf()`方法。）

如果要从日期中添加或减去指定数量的秒、分钟或小时，通常最简单的方法是修改时间戳，就像前面示例中添加 30 秒到日期一样。如果要添加天数，这种技术变得更加繁琐，对于月份和年份则根本不起作用，因为它们的天数不同。要进行涉及天数、月份和年份的日期算术，可以使用`setDate()`、`setMonth()`和`setYear()`。例如，以下是将三个月和两周添加到当前日期的代码：

```js
let d = new Date();
d.setMonth(d.getMonth() + 3, d.getDate() + 14);
```

即使溢出，日期设置方法也能正常工作。当我们向当前月份添加三个月时，可能得到大于 11 的值（代表 12 月）。`setMonth()`通过根据需要递增年份来处理这一点。同样，当我们将月份的日期设置为大于该月份天数的值时，月份会适当递增。

## 11.4.3 格式化和解析日期字符串

如果您使用 Date 类实际跟踪日期和时间（而不仅仅是测量时间间隔），那么您可能需要向代码的用户显示日期和时间。Date 类定义了许多不同的方法来将 Date 对象转换为字符串。以下是一些示例：

```js
let d = new Date(2020, 0, 1, 17, 10, 30); // 5:10:30pm on New Year's Day 2020
d.toString()  // => "Wed Jan 01 2020 17:10:30 GMT-0800 (Pacific Standard Time)"
d.toUTCString()         // => "Thu, 02 Jan 2020 01:10:30 GMT"
d.toLocaleDateString()  // => "1/1/2020": 'en-US' locale
d.toLocaleTimeString()  // => "5:10:30 PM": 'en-US' locale
d.toISOString()         // => "2020-01-02T01:10:30.000Z"
```

这是 Date 类的字符串格式化方法的完整列表：

`toString()`

此方法使用本地时区，但不以区域感知方式格式化日期和时间。

`toUTCString()`

此方法使用 UTC 时区，但不以区域感知方式格式化日期。

`toISOString()`

此方法以 ISO-8601 标准的标准年-月-日小时:分钟:秒.ms 格式打印日期和时间。字母“T”将输出的日期部分与时间部分分开。时间以 UTC 表示，并且最后一个字母“Z”表示这一点。

`toLocaleString()`

此方法使用本地时区和适合用户区域的格式。

`toDateString()`

此方法仅格式化日期部分并省略时间。它使用本地时区，不进行区域适当的格式化。

`toLocaleDateString()`

此方法仅格式化日期。它使用本地时区和适合区域的日期格式。

`toTimeString()`

此方法仅格式化时间并省略日期。它使用本地时区，但不以区域感知方式格式化时间。

`toLocaleTimeString()`

这种方法以区域感知方式格式化时间，并使用本地时区。

当将日期和时间格式化为向最终用户显示时，这些日期转换为字符串的方法都不是理想的。查看 §11.7.2 以获取更通用且区域感知的日期和时间格式化技术。

最后，除了这些将 Date 对象转换为字符串的方法之外，还有一个静态的 `Date.parse()` 方法，它以字符串作为参数，尝试将其解析为日期和时间，并返回表示该日期的时间戳。`Date.parse()` 能够解析 `Date()` 构造函数可以解析的相同字符串，并且保证能够解析 `toISOString()`、`toUTCString()` 和 `toString()` 的输出。

# 11.5 错误类

JavaScript 的 `throw` 和 `catch` 语句可以抛出和捕获任何 JavaScript 值，包括原始值。没有必须用于信号错误的异常类型。但是，JavaScript 确实定义了一个 Error 类，并且在使用 `throw` 信号错误时传统上使用 Error 的实例或子类。使用 Error 对象的一个很好的理由是，当您创建一个 Error 时，它会捕获 JavaScript 堆栈的状态，如果异常未被捕获，堆栈跟踪将显示在错误消息中，这将帮助您调试问题。（请注意，堆栈跟踪显示 Error 对象的创建位置，而不是 `throw` 语句抛出它的位置。如果您总是在使用 `throw new Error()` 抛出之前创建对象，这将不会引起任何混淆。）

Error 对象有两个属性：`message` 和 `name`，以及一个 `toString()` 方法。`message` 属性的值是您传递给 `Error()` 构造函数的值，必要时转换为字符串。对于使用 `Error()` 创建的错误对象，`name` 属性始终为“Error”。`toString()` 方法简单地返回 `name` 属性的值，后跟一个冒号和空格，以及 `message` 属性的值。

尽管它不是 ECMAScript 标准的一部分，但 Node 和所有现代浏览器也在 Error 对象上定义了一个 `stack` 属性。该属性的值是一个多行字符串，其中包含 JavaScript 调用堆栈在创建 Error 对象时的堆栈跟踪。当捕获到意外错误时，这可能是有用的信息进行记录。

除了 Error 类之外，JavaScript 还定义了一些子类，用于信号 ECMAScript 定义的特定类型的错误。这些子类包括 EvalError、RangeError、ReferenceError、SyntaxError、TypeError 和 URIError。如果看起来合适，您可以在自己的代码中使用这些错误类。与基本 Error 类一样，这些子类的每个都有一个接受单个消息参数的构造函数。并且每个这些子类的实例都有一个 `name` 属性，其值与构造函数名称相同。

您可以随意定义最能封装您自己程序的错误条件的 Error 子类。请注意，您不仅限于 `name` 和 `message` 属性。如果创建一个子类，您可以定义新属性以提供错误详细信息。例如，如果您正在编写解析器，可能会发现定义一个具有指定解析失败确切位置的 `line` 和 `column` 属性的 ParseError 类很有用。或者，如果您正在处理 HTTP 请求，可能希望定义一个具有保存失败请求的 HTTP 状态码（例如 404 或 500）的 `status` 属性的 HTTPError 类。

例如：

```js
class HTTPError extends Error {
    constructor(status, statusText, url) {
        super(`${status} ${statusText}: ${url}`);
        this.status = status;
        this.statusText = statusText;
        this.url = url;
    }

    get name() { return "HTTPError"; }
}

let error = new HTTPError(404, "Not Found", "http://example.com/");
error.status        // => 404
error.message       // => "404 Not Found: http://example.com/"
error.name          // => "HTTPError"
```

# 11.6 JSON 序列化和解析

当程序需要保存数据或需要将数据通过网络连接传输到另一个程序时，它必须将其内存中的数据结构转换为一串字节或字符，这些字节或字符可以被保存或传输，然后稍后被解析以恢复原始的内存中的数据结构。将数据结构转换为字节流或字符流的过程称为*序列化*（或*编组*甚至*腌制*）。

在 JavaScript 中序列化数据的最简单方法使用了一种称为 JSON 的序列化格式。这个首字母缩写代表“JavaScript 对象表示法”，正如名称所示，该格式使用 JavaScript 对象和数组文字语法将由对象和数组组成的数据结构转换为字符串。JSON 支持原始数字和字符串，以及值`true`、`false`和`null`，以及由这些原始值构建的数组和对象。JSON 不支持 Map、Set、RegExp、Date 或类型化数组等其他 JavaScript 类型。尽管如此，它已被证明是一种非常多才多艺的数据格式，即使在非基于 JavaScript 的程序中也被广泛使用。

JavaScript 支持使用两个函数`JSON.stringify()`和`JSON.parse()`进行 JSON 序列化和反序列化，这两个函数在§6.8 中简要介绍过。给定一个不包含任何非可序列化值（如 RegExp 对象或类型化数组）的对象或数组（任意深度嵌套），您可以通过将其传递给`JSON.stringify()`来简单地序列化对象。正如名称所示，此函数的返回值是一个字符串。并且给定`JSON.stringify()`返回的字符串，您可以通过将字符串传递给`JSON.parse()`来重新创建原始数据结构：

```js
let o = {s: "", n: 0, a: [true, false, null]};
let s = JSON.stringify(o);  // s == '{"s":"","n":0,"a":[true,false,null]}'
let copy = JSON.parse(s);   // copy == {s: "", n: 0, a: [true, false, null]}
```

如果我们忽略序列化数据保存到文件或通过网络发送的部分，我们可以将这对函数用作创建对象的深层副本的一种效率较低的方式：

```js
// Make a deep copy of any serializable object or array
function deepcopy(o) {
    return JSON.parse(JSON.stringify(o));
}
```

# JSON 是 JavaScript 的一个子集

当数据序列化为 JSON 格式时，结果是一个有效的 JavaScript 源代码，用于评估为原始数据结构的副本。如果您在 JSON 字符串前面加上`var data =`并将结果传递给`eval()`，您将获得将原始数据结构的副本分配给变量`data`的结果。但是，您绝对不应该这样做，因为这是一个巨大的安全漏洞——如果攻击者可以将任意 JavaScript 代码注入 JSON 文件中，他们可以使您的程序运行他们的代码。只需使用`JSON.parse()`来解码 JSON 格式化数据，这样更快速和安全。

JSON 有时被用作人类可读的配置文件格式。如果您发现自己手动编辑 JSON 文件，请注意 JSON 格式是 JavaScript 的一个非常严格的子集。不允许注释，属性名称必须用双引号括起来，即使 JavaScript 不需要这样做。

通常，您只向`JSON.stringify()`和`JSON.parse()`传递单个参数。这两个函数都接受一个可选的第二个参数，允许我们扩展 JSON 格式，接下来将对此进行描述。`JSON.stringify()`还接受一个可选的第三个参数，我们将首先讨论这个参数。如果您希望您的 JSON 格式化字符串可读性强（例如用作配置文件），那么应将`null`作为第二个参数传递，并将数字或字符串作为第三个参数传递。第三个参数告诉`JSON.stringify()`应该将数据格式化为多个缩进行。如果第三个参数是一个数字，则它将使用该数字作为每个缩进级别的空格数。如果第三个参数是一个空格字符串（例如`'\t'`），它将使用该字符串作为每个缩进级别。

```js
let o = {s: "test", n: 0};
JSON.stringify(o, null, 2)  // => '{\n  "s": "test",\n  "n": 0\n}'
```

`JSON.parse()`会忽略空格，因此向`JSON.stringify()`传递第三个参数对我们将字符串转换回数据结构的能力没有影响。

## 11.6.1 JSON 自定义

如果`JSON.stringify()`被要求序列化一个 JSON 格式不支持的值，它会查看该值是否有一个`toJSON()`方法，如果有，它会调用该方法，然后将返回值序列化以替换原始值。Date 对象实现了`toJSON()`：它返回与`toISOString()`方法相同的字符串。这意味着如果序列化包含 Date 的对象，日期将自动转换为字符串。当您解析序列化的字符串时，重新创建的数据结构将不会与您开始的完全相同，因为它将在原始对象有 Date 的地方有一个字符串。

如果需要重新创建 Date 对象（或以任何其他方式修改解析的对象），可以将“恢复器”函数作为第二个参数传递给`JSON.parse()`。如果指定了，这个“恢复器”函数将被用于从输入字符串解析的每个原始值（但不包含这些原始值的对象或数组）。该函数被调用时带有两个参数。第一个是属性名称—一个对象属性名称或转换为字符串的数组索引。第二个参数是该对象属性或数组元素的原始值。此外，该函数作为包含原始值的对象或数组的方法被调用，因此您可以使用`this`关键字引用该包含对象。

恢复函数的返回值将成为命名属性的新值。如果它返回其第二个参数，则属性将保持不变。如果返回`undefined`，则在`JSON.parse()`返回给用户之前，命名属性将从对象或数组中删除。

作为示例，这里是一个调用`JSON.parse()`的示例，使用恢复器函数来过滤一些属性并重新创建 Date 对象：

```js
let data = JSON.parse(text, function(key, value) {
    // Remove any values whose property name begins with an underscore
    if (key[0] === "_") return undefined;

    // If the value is a string in ISO 8601 date format convert it to a Date.
    if (typeof value === "string" &&
        /^\d\d\d\d-\d\d-\d\dT\d\d:\d\d:\d\d.\d\d\dZ$/.test(value)) {
        return new Date(value);
    }

    // Otherwise, return the value unchanged
    return value;
});
```

除了前面描述的`toJSON()`的使用，`JSON.stringify()`还允许通过将数组或函数作为可选的第二个参数来自定义其输出。

如果作为第二个参数传递的是字符串数组（或数字—它们会被转换为字符串），那么这些将被用作对象属性（或数组元素）的名称。任何名称不在数组中的属性都将被省略。此外，返回的字符串将按照它们在数组中出现的顺序包括属性（在编写测试时非常有用）。

如果传递一个函数，它是一个替换函数—实际上是您可以传递给`JSON.parse()`的可选恢复函数的反函数。如果指定了替换函数，那么替换函数将被用于要序列化的每个值。替换函数的第一个参数是该对象中值的对象属性名称或数组索引，第二个参数是值本身。替换函数作为包含要序列化值的对象或数组的方法被调用。替换函数的返回值将被序列化以替换原始值。如果替换函数返回`undefined`或根本没有返回任何内容，则该值（及其数组元素或对象属性）将被省略在序列化中。

```js
// Specify what fields to serialize, and what order to serialize them in
let text = JSON.stringify(address, ["city","state","country"]);

// Specify a replacer function that omits RegExp-value properties
let json = JSON.stringify(o, (k, v) => v instanceof RegExp ? undefined : v);
```

这里的两个`JSON.stringify()`调用以一种良性的方式使用第二个参数，产生的序列化输出可以在不需要特殊恢复函数的情况下反序列化。然而，一般来说，如果为类型定义了`toJSON()`方法，或者使用一个实际上用可序列化值替换不可序列化值的替换函数，那么通常需要使用自定义恢复函数与`JSON.parse()`一起来获取原始数据结构。如果这样做，你应该明白你正在定义一种自定义数据格式，并牺牲了与大量 JSON 兼容工具和语言的可移植性和兼容性。

# 11.7 国际化 API

JavaScript 国际化 API 由三个类 Intl.NumberFormat、Intl.DateTimeFormat 和 Intl.Collator 组成，允许我们以区域设置适当的方式格式化数字（包括货币金额和百分比）、日期和时间，并以区域设置适当的方式比较字符串。这些类不是 ECMAScript 标准的一部分，但作为[ECMA402 标准](https://tc39.es/ecma402/)的一部分定义，并得到 Web 浏览器的良好支持。Intl API 也受 Node 支持，但在撰写本文时，预构建的 Node 二进制文件不包含所需的本地化数据，以使它们能够与除美国英语以外的区域设置一起使用。因此，为了在 Node 中使用这些类，您可能需要下载一个单独的数据包或使用自定义构建的 Node。

国际化中最重要的部分之一是显示已翻译为用户语言的文本。有各种方法可以实现这一点，但这些方法都不在此处描述的 Intl API 的范围内。

## 11.7.1 格式化数字

世界各地的用户期望以不同的方式格式化数字。小数点可以是句点或逗号。千位分隔符可以是逗号或句点，并且并非在所有地方每三位数字都使用。一些货币被分成百分之一，一些被分成千分之一，一些没有细分。最后，尽管所谓的“阿拉伯数字”0 到 9 在许多语言中使用，但这并非普遍，一些国家的用户期望看到使用其自己脚本中的数字编写的数字。

Intl.NumberFormat 类定义了一个`format()`方法，考虑到所有这些格式化可能性。构造函数接受两个参数。第一个参数指定应为其格式化数字的区域设置，第二个是一个对象，指定有关如何格式化数字的更多详细信息。如果省略或`undefined`第一个参数，则将使用系统区域设置（我们假设为用户首选区域设置）。如果第一个参数是字符串，则指定所需的区域设置，例如`"en-US"`（美国使用的英语）、`"fr"`（法语）或`"zh-Hans-CN"`（中国使用简体汉字书写系统）。第一个参数也可以是区域设置字符串数组，在这种情况下，Intl.NumberFormat 将选择最具体且受支持的区域设置。

如果指定了`Intl.NumberFormat()`构造函数的第二个参数，则应该是一个定义一个或多个以下属性的对象：

`style`

指定所需的数字格式化类型。默认值为`"decimal"`。指定`"percent"`将数字格式化为百分比，或指定`"currency"`将数字格式化为货币金额。

`currency`

如果样式为`"currency"`，则需要此属性来指定所需货币的三个字母 ISO 货币代码（例如`"USD"`表示美元或`"GBP"`表示英镑）。

`currencyDisplay`

如果样式为`"currency"`，则此属性指定货币的显示方式。默认值`"symbol"`使用货币符号（如果货币有符号）。值`"code"`使用三个字母 ISO 代码，值`"name"`以长形式拼写货币名称。

`useGrouping`

将此属性设置为`false`，如果您不希望数字具有千位分隔符（或其相应的区域设置等价物）。

`minimumIntegerDigits`

用于显示数字整数部分的最小位数。如果数字的位数少于此值，则将在左侧用零填充。默认值为 1，但可以使用高达 21 的值。

`minimumFractionDigits`，`maximumFractionDigits`

这两个属性控制数字的小数部分的格式。如果一个数字的小数位数少于最小值，它将在右侧用零填充。如果小数位数超过最大值，那么小数部分将被四舍五入。这两个属性的合法值介于 0 和 20 之间。默认最小值为 0，最大值为 3，除了在格式化货币金额时，小数部分的长度会根据指定的货币而变化。

`minimumSignificantDigits`，`maximumSignificantDigits`

这些属性控制在格式化数字时使用的有效数字位数，使其适用于格式化科学数据等情况。如果指定了这些属性，它们将覆盖先前列出的整数和小数位数属性。合法值介于 1 和 21 之间。

一旦您使用所需的区域设置和选项创建了一个 Intl.NumberFormat 对象，您可以通过将数字传递给其`format()`方法来使用它，该方法将返回一个适当格式化的字符串。例如：

```js
let euros = Intl.NumberFormat("es", {style: "currency", currency: "EUR"});
euros.format(10)    // => "10,00 €": ten euros, Spanish formatting

let pounds = Intl.NumberFormat("en", {style: "currency", currency: "GBP"});
pounds.format(1000) // => "£1,000.00": One thousand pounds, English formatting
```

`Intl.NumberFormat`（以及其他 Intl 类）的一个有用功能是它的`format()`方法绑定到它所属的 NumberFormat 对象。因此，您可以将`format()`方法分配给一个变量，并像独立函数一样使用它，而不是定义一个引用格式化对象的变量，然后在该变量上调用`format()`方法，就像这个例子中一样：

```js
let data = [0.05, .75, 1];
let formatData = Intl.NumberFormat(undefined, {
    style: "percent",
    minimumFractionDigits: 1,
    maximumFractionDigits: 1
}).format;

data.map(formatData)   // => ["5.0%", "75.0%", "100.0%"]: in en-US locale
```

一些语言，比如阿拉伯语，使用自己的脚本来表示十进制数字：

```js
let arabic = Intl.NumberFormat("ar", {useGrouping: false}).format;
arabic(1234567890)   // => "١٢٣٤٥٦٧٨٩٠"
```

其他语言，比如印地语，使用自己的数字字符集，但默认情况下倾向于使用 ASCII 数字 0-9。如果要覆盖用于数字的默认字符集，请在区域设置中添加`-u-nu-`，然后跟上简写的字符集名称。例如，您可以这样格式化数字，使用印度风格的分组和天城数字：

```js
let hindi = Intl.NumberFormat("hi-IN-u-nu-deva").format;
hindi(1234567890)    // => "१,२३,४५,६७,८९०"
```

在区域设置中的`-u-`指定接下来是一个 Unicode 扩展。`nu`是编号系统的扩展名称，`deva`是 Devanagari 的缩写。Intl API 标准为许多其他编号系统定义了名称，主要用于南亚和东南亚的印度语言。

## 11.7.2 格式化日期和时间

Intl.DateTimeFormat 类与 Intl.NumberFormat 类非常相似。`Intl.DateTimeFormat()`构造函数接受与`Intl.NumberFormat()`相同的两个参数：区域设置或区域设置数组以及格式选项对象。使用 Intl.DateTimeFormat 实例的方法是调用其`format()`方法，将 Date 对象转换为字符串。

如§11.4 中所述，Date 类定义了简单的`toLocaleDateString()`和`toLocaleTimeString()`方法，为用户的区域设置生成适当的输出。但是这些方法不会让您控制显示的日期和时间字段。也许您想省略年份，但在日期格式中添加一个工作日。您希望月份是以数字形式表示还是以名称拼写出来？Intl.DateTimeFormat 类根据传递给构造函数的第二个参数中的选项对象中的属性提供对输出的细粒度控制。但是，请注意，Intl.DateTimeFormat 不能总是精确显示您要求的内容。如果指定了格式化小时和秒的选项但省略了分钟，您会发现格式化程序仍然会显示分钟。这个想法是您使用选项对象指定要向用户呈现的日期和时间字段以及您希望如何格式化这些字段（例如按名称或按数字），然后格式化程序将查找最接近您要求的内容的适合区域设置的格式。

可用的选项如下。只为您希望出现在格式化输出中的日期和时间字段指定属性。

`年`

使用`"numeric"`表示完整的四位数年份，或使用`"2-digit"`表示两位数缩写。

`月`

使用`"numeric"`表示可能的短数字，如“1”，或`"2-digit"`表示始终有两位数字的数字表示，如“01”。使用`"long"`表示全名，如“January”，`"short"`表示缩写，如“Jan”，`"narrow"`表示高度缩写，如“J”，不保证唯一。

`day`

使用`"numeric"`表示一位或两位数字，或`"2-digit"`表示月份的两位数字。

`weekday`

使用`"long"`表示全名，如“Monday”，`"short"`表示缩写，如“Mon”，`"narrow"`表示高度缩写，如“M”，不保证唯一。

`era`

此属性指定日期是否应以时代（如 CE 或 BCE）格式化。如果您正在格式化很久以前的日期或使用日本日历，则可能很有用。合法值为`"long"`、`"short"`和`"narrow"`。

`hour`、`minute`、`second`

这些属性指定您希望如何显示时间。使用`"numeric"`表示一位或两位数字字段，或`"2-digit"`强制将单个数字左侧填充为 0。

`timeZone`

此属性指定应为其格式化日期的所需时区。如果省略，将使用本地时区。实现始终识别“UTC”，并且还可以识别互联网分配的数字管理局（IANA）时区名称，例如“America/Los_Angeles”。

`timeZoneName`

此属性指定应如何在格式化的日期或时间中显示时区。使用`"long"`表示完全拼写的时区名称，`"short"`表示缩写或数字时区。

`hour12`

这个布尔属性指定是否使用 12 小时制。默认是与地区相关的，但你可以用这个属性来覆盖它。

`hourCycle`

此属性允许您指定午夜是写作 0 小时、12 小时还是 24 小时。默认是与地区相关的，但您可以用此属性覆盖默认值。请注意，`hour12`优先于此属性。使用值`"h11"`指定午夜为 0，午夜前一小时为 11pm。使用`"h12"`指定午夜为 12。使用`"h23"`指定午夜为 0，午夜前一小时为 23。使用`"h24"`指定午夜为 24。

以下是一些示例：

```js
let d = new Date("2020-01-02T13:14:15Z");  // January 2nd, 2020, 13:14:15 UTC

// With no options, we get a basic numeric date format
Intl.DateTimeFormat("en-US").format(d) // => "1/2/2020"
Intl.DateTimeFormat("fr-FR").format(d) // => "02/01/2020"

// Spelled out weekday and month
let opts = { weekday: "long", month: "long", year: "numeric", day: "numeric" };
Intl.DateTimeFormat("en-US", opts).format(d) // => "Thursday, January 2, 2020"
Intl.DateTimeFormat("es-ES", opts).format(d) // => "jueves, 2 de enero de 2020"

// The time in New York, for a French-speaking Canadian
opts = { hour: "numeric", minute: "2-digit", timeZone: "America/New_York" };
Intl.DateTimeFormat("fr-CA", opts).format(d) // => "8 h 14"
```

Intl.DateTimeFormat 可以使用除基于基督教时代的默认儒略历之外的其他日历显示日期。尽管一些地区可能默认使用非基督教日历，但您始终可以通过在地区后添加`-u-ca-`并在其后跟日历名称来明确指定要使用的日历。可能的日历名称包括“buddhist”、“chinese”、“coptic”、“ethiopic”、“gregory”、“hebrew”、“indian”、“islamic”、“iso8601”、“japanese”和“persian”。继续前面的示例，我们可以确定各种非基督教历法中的年份：

```js
let opts = { year: "numeric", era: "short" };
Intl.DateTimeFormat("en", opts).format(d)                // => "2020 AD"
Intl.DateTimeFormat("en-u-ca-iso8601", opts).format(d)   // => "2020 AD"
Intl.DateTimeFormat("en-u-ca-hebrew", opts).format(d)    // => "5780 AM"
Intl.DateTimeFormat("en-u-ca-buddhist", opts).format(d)  // => "2563 BE"
Intl.DateTimeFormat("en-u-ca-islamic", opts).format(d)   // => "1441 AH"
Intl.DateTimeFormat("en-u-ca-persian", opts).format(d)   // => "1398 AP"
Intl.DateTimeFormat("en-u-ca-indian", opts).format(d)    // => "1941 Saka"
Intl.DateTimeFormat("en-u-ca-chinese", opts).format(d)   // => "36 78"
Intl.DateTimeFormat("en-u-ca-japanese", opts).format(d)  // => "2 Reiwa"
```

## 11.7.3 比较字符串

将字符串按字母顺序排序（或对于非字母脚本的更一般“排序顺序”）的问题比英语使用者通常意识到的更具挑战性。英语使用相对较小的字母表，没有重音字母，并且我们有字符编码（ASCII，已合并到 Unicode 中）的好处，其数值完全匹配我们的标准字符串排序顺序。在其他语言中情况并不那么简单。例如，西班牙语将ñ视为一个独立的字母，位于 n 之后和 o 之前。立陶宛语将 Y 排在 J 之前，威尔士语将 CH 和 DD 等二合字母视为单个字母，CH 排在 C 之后，DD 排在 D 之后。

如果要按用户自然顺序显示字符串，仅使用数组字符串的`sort()`方法是不够的。但是，如果创建 Intl.Collator 对象，可以将该对象的`compare()`方法传递给`sort()`方法，以执行符合区域设置的字符串排序。Intl.Collator 对象可以配置为使`compare()`方法执行不区分大小写的比较，甚至只考虑基本字母并忽略重音和其他变音符号的比较。

与`Intl.NumberFormat()`和`Intl.DateTimeFormat()`一样，`Intl.Collator()`构造函数接受两个参数。第一个指定区域设置或区域设置数组，第二个是一个可选对象，其属性精确指定要执行的字符串比较类型。支持的属性如下：

`usage`

此属性指定如何使用排序器对象。默认值为`"sort"`，但也可以指定`"search"`。想法是，在对字符串进行排序时，通常希望排序器尽可能区分多个字符串以产生可靠的排序。但是，在比较两个字符串时，某些区域设置可能希望进行较不严格的比较，例如忽略重音。

`sensitivity`

此属性指定比较字符串时，排序器是否对大小写和重音敏感。值为`"base"`会忽略大小写和重音，只考虑每个字符的基本字母。（但请注意，某些语言认为某些带重音的字符是不同的基本字母。）`"accent"`考虑重音但忽略大小写。`"case"`考虑大小写但忽略重音。`"variant"`执行严格的比较，考虑大小写和重音。当`usage`为`"sort"`时，此属性的默认值为`"variant"`。如果`usage`为`"search"`，则默认灵敏度取决于区域设置。

`ignorePunctuation`

将此属性设置为`true`以在比较字符串时忽略空格和标点符号。将此属性设置为`true`后，例如，字符串“any one”和“anyone”将被视为相等。

`numeric`

如果要比较的字符串是整数或包含整数，并且希望它们按数字顺序而不是按字母顺序排序，请将此属性设置为`true`。设置此选项后，例如，字符串“Version 9”将在“Version 10”之前排序。

`caseFirst`

此属性指定哪种大小写应该优先。如果指定为`"upper"`，则“A”将在“a”之前排序。如果指定为`"lower"`，则“a”将在“A”之前排序。无论哪种情况，请注意相同字母的大写和小写变体将按顺序排列在一起，这与 Unicode 词典排序（数组`sort()`方法的默认行为）不同，在该排序中，所有 ASCII 大写字母都排在所有 ASCII 小写字母之前。此属性的默认值取决于区域设置，并且实现可能会忽略此属性并不允许您覆盖大小写排序顺序。

一旦为所需区域设置和选项创建了 Intl.Collator 对象，就可以使用其`compare()`方法比较两个字符串。此方法返回一个数字。如果返回值小于零，则第一个字符串在第二个字符串之前。如果大于零，则第一个字符串在第二个字符串之后。如果`compare()`返回零，则这两个字符串在此排序器的意义上相等。

此接受两个字符串并返回小于、等于或大于零的数字的`compare()`方法正是数组`sort()`方法期望的可选参数。此外，Intl.Collator 会自动将`compare()`方法绑定到其实例，因此可以直接将其传递给`sort()`，而无需编写包装函数并通过排序器对象调用它。以下是一些示例：

```js
// A basic comparator for sorting in the user's locale.
// Never sort human-readable strings without passing something like this:
const collator = new Intl.Collator().compare;
["a", "z", "A", "Z"].sort(collator)      // => ["a", "A", "z", "Z"]

// Filenames often include numbers, so we should sort those specially
const filenameOrder = new Intl.Collator(undefined, { numeric: true }).compare;
["page10", "page9"].sort(filenameOrder)  // => ["page9", "page10"]

// Find all strings that loosely match a target string
const fuzzyMatcher = new Intl.Collator(undefined, {
    sensitivity: "base",
    ignorePunctuation: true
}).compare;
let strings = ["food", "fool", "Føø Bar"];
strings.findIndex(s => fuzzyMatcher(s, "foobar") === 0)  // => 2
```

一些地区有多种可能的排序顺序。例如，在德国，电话簿使用的排序顺序比字典稍微更加语音化。在西班牙，在 1994 年之前，“ch” 和 “ll” 被视为单独的字母，因此该国现在有现代排序顺序和传统排序顺序。在中国，排序顺序可以基于字符编码、每个字符的基本部首和笔画，或者基于字符的拼音罗马化。这些排序变体不能通过 Intl.Collator 选项参数进行选择，但可以通过在区域设置字符串中添加 `-u-co-` 并添加所需变体的名称来选择。例如，在德国使用 `"de-DE-u-co-phonebk"` 进行电话簿排序，在台湾使用 `"zh-TW-u-co-pinyin"` 进行拼音排序。

```js
// Before 1994, CH and LL were treated as separate letters in Spain
const modernSpanish = Intl.Collator("es-ES").compare;
const traditionalSpanish = Intl.Collator("es-ES-u-co-trad").compare;
let palabras = ["luz", "llama", "como", "chico"];
palabras.sort(modernSpanish)      // => ["chico", "como", "llama", "luz"]
palabras.sort(traditionalSpanish) // => ["como", "chico", "luz", "llama"]
```

# 11.8 控制台 API

你在本书中看到了 `console.log()` 函数的使用：在网页浏览器中，它会在浏览器的开发者工具窗格的“控制台”选项卡中打印一个字符串，这在调试时非常有帮助。在 Node 中，`console.log()` 是一个通用输出函数，将其参数打印到进程的 stdout 流中，在终端窗口中通常会显示给用户作为程序输出。

控制台 API 除了 `console.log()` 外还定义了许多有用的函数。该 API 不是任何 ECMAScript 标准的一部分，但受到浏览器和 Node 的支持，并已经正式编写和标准化在 [*https://console.spec.whatwg.org*](https://console.spec.whatwg.org)。

控制台 API 定义了以下函数：

`console.log()`

这是控制台函数中最为人熟知的。它将其参数转换为字符串并将它们输出到控制台。它在参数之间包含空格，并在输出所有参数后开始新的一行。

`console.debug()`, `console.info()`, `console.warn()`, `console.error()`

这些函数几乎与 `console.log()` 完全相同。在 Node 中，`console.error()` 将其输出发送到 stderr 流而不是 stdout 流，但其他函数是 `console.log()` 的别名。在浏览器中，每个函数生成的输出消息可能会以指示其级别或严重性的图标为前缀，并且开发者控制台还可以允许开发者按级别过滤控制台消息。

`console.assert()`

如果第一个参数为真值（即如果断言通过），则此函数不执行任何操作。但如果第一个参数为 `false` 或其他假值，则剩余的参数将被打印，就像它们已经被传递给带有“Assertion failed”前缀的 `console.error()` 一样。请注意，与典型的 `assert()` 函数不同，当断言失败时，`console.assert()` 不会抛出异常。

`console.clear()`

此函数在可能的情况下清除控制台。这在浏览器和在 Node 将其输出显示到终端时有效。但是，如果 Node 的输出已被重定向到文件或管道，则调用此函数没有效果。

`console.table()`

这个函数是一个非常强大但鲜为人知的功能，用于生成表格输出，特别适用于需要总结数据的 Node 程序。`console.table()`尝试以表格形式显示其参数（尽管如果无法做到这一点，它会使用常规的`console.log()`格式）。当参数是一个相对较短的对象数组，并且数组中的所有对象具有相同（相对较小）的属性集时，这种方法效果最佳。在这种情况下，数组中的每个对象被格式化为表格的一行，每个属性是表格的一列。您还可以将属性名称数组作为可选的第二个参数传递，以指定所需的列集。如果传递的是对象而不是对象数组，则输出将是一个具有属性名称列和属性值列的表格。或者，如果这些属性值本身是对象，则它们的属性名称将成为表格中的列。

`console.trace()`

这个函数像`console.log()`一样记录其参数，并且在输出后跟随一个堆栈跟踪。在 Node 中，输出会发送到 stderr 而不是 stdout。

`console.count()`

这个函数接受一个字符串参数，并记录该字符串，然后记录调用该字符串的次数。在调试事件处理程序时，这可能很有用，例如，如果需要跟踪事件处理程序被触发的次数。

`console.countReset()`

这个函数接受一个字符串参数，并重置该字符串的计数器。

`console.group()`

这个函数将其参数打印到控制台，就像它们已被传递给`console.log()`一样，然后设置控制台的内部状态，以便所有后续的控制台消息（直到下一个`console.groupEnd()`调用）将相对于刚刚打印的消息进行缩进。这允许将一组相关消息视觉上分组并缩进。在 Web 浏览器中，开发者控制台通常允许将分组消息折叠和展开为一组。`console.group()`的参数通常用于为组提供解释性名称。

`console.groupCollapsed()`

这个函数与`console.group()`类似，但在 Web 浏览器中，默认情况下，该组将“折叠”，并且它包含的消息将被隐藏，除非用户点击以展开该组。在 Node 中，此函数是`console.group()`的同义词。

`console.groupEnd()`

这个函数不接受任何参数。它不产生自己的输出，但结束了由最近调用的`console.group()`或`console.groupCollapsed()`引起的缩进和分组。

`console.time()`

这个函数接受一个字符串参数，记录调用该字符串的时间，并不产生输出。

`console.timeLog()`

这个函数将一个字符串作为其第一个参数。如果该字符串之前已传递给`console.time()`，则打印该字符串，然后是自`console.time()`调用以来经过的时间。如果`console.timeLog()`有任何额外的参数，它们将被打印，就像它们已被传递给`console.log()`一样。

`console.timeEnd()`

这个函数接受一个字符串参数。如果之前已将该参数传递给`console.time()`，则打印该参数和经过的时间。在调用`console.timeEnd()`之后，再次调用`console.timeLog()`而不先调用`console.time()`是不合法的。

## 11.8.1 使用控制台进行格式化输出

类似`console.log()`打印其参数的控制台函数有一个鲜为人知的功能：如果第一个参数是包含`%s`、`%i`、`%d`、`%f`、`%o`、`%O`或`%c`的字符串，则此第一个参数将被视为格式字符串，⁶，并且后续参数的值将替换两个字符`%`序列的位置。

序列的含义如下：

`%s`

参数被转换为字符串。

`%i` 和 `%d`

参数被转换为数字，然后截断为整数。

`%f`

参数被转换为数字

`%o` 和 `%O`

参数被视为对象，并显示属性名称和值。（在 Web 浏览器中，此显示通常是交互式的，用户可以展开和折叠属性以探索嵌套的数据结构。）`%o` 和 `%O` 都显示对象的详细信息。大写变体使用一个依赖于实现的输出格式，被认为对软件开发人员最有用。

`%c`

在 Web 浏览器中，参数被解释为一串 CSS 样式，并用于为接下来的任何文本设置样式（直到下一个 `%c` 序列或字符串结束）。在 Node 中，`%c` 序列及其对应的参数会被简单地忽略。

请注意，通常不需要在控制台函数中使用格式字符串：通常只需将一个或多个值（包括对象）传递给函数，让实现以有用的方式显示它们即可。例如，请注意，如果将 Error 对象传递给 `console.log()`，它将自动打印出其堆栈跟踪。

# 11.9 URL API

由于 JavaScript 在 Web 浏览器和 Web 服务器中被广泛使用，JavaScript 代码通常需要操作 URL。URL 类解析 URL 并允许修改（例如添加搜索参数或更改路径）现有的 URL。它还正确处理了 URL 的各个组件的转义和解码这一复杂主题。

URL 类不是任何 ECMAScript 标准的一部分，但它在 Node 和除了 Internet Explorer 之外的所有互联网浏览器中都可以使用。它在 [*https://url.spec.whatwg.org*](https://url.spec.whatwg.org) 上标准化。

使用 `URL()` 构造函数创建一个 URL 对象，将绝对 URL 字符串作为参数传递。或者将相对 URL 作为第一个参数传递，将其相对的绝对 URL 作为第二个参数传递。一旦创建了 URL 对象，它的各种属性允许您查询 URL 的各个部分的未转义版本：

```js
let url = new URL("https://example.com:8000/path/name?q=term#fragment");
url.href        // => "https://example.com:8000/path/name?q=term#fragment"
url.origin      // => "https://example.com:8000"
url.protocol    // => "https:"
url.host        // => "example.com:8000"
url.hostname    // => "example.com"
url.port        // => "8000"
url.pathname    // => "/path/name"
url.search      // => "?q=term"
url.hash        // => "#fragment"
```

尽管不常用，URL 可以包含用户名或用户名和密码，URL 类也可以解析这些 URL 组件：

```js
let url = new URL("ftp://admin:1337!@ftp.example.com/");
url.href       // => "ftp://admin:1337!@ftp.example.com/"
url.origin     // => "ftp://ftp.example.com"
url.username   // => "admin"
url.password   // => "1337!"
```

这里的 `origin` 属性是 URL 协议和主机（包括指定的端口）的简单组合。因此，它是一个只读属性。但前面示例中演示的每个其他属性都是读/写的：您可以设置这些属性中的任何一个来设置 URL 的相应部分：

```js
let url = new URL("https://example.com");  // Start with our server
url.pathname = "api/search";               // Add a path to an API endpoint
url.search = "q=test";                     // Add a query parameter
url.toString()  // => "https://example.com/api/search?q=test"
```

URL 类的一个重要特性是在需要时正确添加标点符号并转义 URL 中的特殊字符：

```js
let url = new URL("https://example.com");
url.pathname = "path with spaces";
url.search = "q=foo#bar";
url.pathname  // => "/path%20with%20spaces"
url.search    // => "?q=foo%23bar"
url.href      // => "https://example.com/path%20with%20spaces?q=foo%23bar"
```

这些示例中的 `href` 属性是一个特殊的属性：读取 `href` 等同于调用 `toString()`：它将 URL 的所有部分重新组合成 URL 的规范字符串形式。将 `href` 设置为新字符串会重新运行 URL 解析器，就好像再次调用 `URL()` 构造函数一样。

在前面的示例中，我们一直使用 `search` 属性来引用 URL 的整个查询部分，该部分由问号到 URL 结尾的第一个井号字符组成。有时，将其视为单个 URL 属性就足够了。然而，HTTP 请求通常使用 `application/x-www-form-urlencoded` 格式将多个表单字段或多个 API 参数的值编码到 URL 的查询部分中。在此格式中，URL 的查询部分是一个问号，后面跟着一个或多个名称/值对，它们之间用和号分隔。同一个名称可以出现多次，导致具有多个值的命名搜索参数。

如果你想将这些名称/值对编码到 URL 的查询部分中，那么`searchParams`属性比`search`属性更有用。`search`属性是一个可读/写的字符串，允许你获取和设置 URL 的整个查询部分。`searchParams`属性是一个只读引用，指向一个 URLSearchParams 对象，该对象具有用于获取、设置、添加、删除和排序编码到 URL 查询部分的参数的 API：

```js
let url = new URL("https://example.com/search");
url.search                            // => "": no query yet
url.searchParams.append("q", "term"); // Add a search parameter
url.search                            // => "?q=term"
url.searchParams.set("q", "x");       // Change the value of this parameter
url.search                            // => "?q=x"
url.searchParams.get("q")             // => "x": query the parameter value
url.searchParams.has("q")             // => true: there is a q parameter
url.searchParams.has("p")             // => false: there is no p parameter
url.searchParams.append("opts", "1"); // Add another search parameter
url.search                            // => "?q=x&opts=1"
url.searchParams.append("opts", "&"); // Add another value for same name
url.search                            // => "?q=x&opts=1&opts=%26": note escape
url.searchParams.get("opts")          // => "1": the first value
url.searchParams.getAll("opts")       // => ["1", "&"]: all values
url.searchParams.sort();              // Put params in alphabetical order
url.search                            // => "?opts=1&opts=%26&q=x"
url.searchParams.set("opts", "y");    // Change the opts param
url.search                            // => "?opts=y&q=x"
// searchParams is iterable
[...url.searchParams]                 // => [["opts", "y"], ["q", "x"]]
url.searchParams.delete("opts");      // Delete the opts param
url.search                            // => "?q=x"
url.href                              // => "https://example.com/search?q=x"
```

`searchParams`属性的值是一个 URLSearchParams 对象。如果你想将 URL 参数编码到查询字符串中，可以创建一个 URLSearchParams 对象，追加参数，然后将其转换为字符串并设置在 URL 的`search`属性上：

```js
let url = new URL("http://example.com");
let params = new URLSearchParams();
params.append("q", "term");
params.append("opts", "exact");
params.toString()               // => "q=term&opts=exact"
url.search = params;
url.href                        // => "http://example.com/?q=term&opts=exact"
```

## 11.9.1 传统 URL 函数

在之前描述的 URL API 定义之前，已经有多次尝试在核心 JavaScript 语言中支持 URL 转义和解码。第一次尝试是全局定义的`escape()`和`unescape()`函数，现在已经被弃用，但仍然被广泛实现。不应该使用它们。

当`escape()`和`unescape()`被弃用时，ECMAScript 引入了两对替代的全局函数：

`encodeURI()`和`decodeURI()`

`encodeURI()`以字符串作为参数，返回一个新字符串，其中非 ASCII 字符和某些 ASCII 字符（如空格）被转义。`decodeURI()`则相反。需要转义的字符首先被转换为它们的 UTF-8 编码，然后该编码的每个字节被替换为一个`%`*xx*转义序列，其中*xx*是两个十六进制数字。因为`encodeURI()`旨在对整个 URL 进行编码，它不会转义 URL 分隔符字符，如`/`、`?`和`#`。但这意味着`encodeURI()`无法正确处理 URL 中包含这些字符的各个组件的 URL。

`encodeURIComponent()`和`decodeURIComponent()`

这一对函数的工作方式与`encodeURI()`和`decodeURI()`完全相同，只是它们旨在转义 URI 的各个组件，因此它们还会转义用于分隔这些组件的字符，如`/`、`?`和`#`。这些是传统 URL 函数中最有用的，但请注意，`encodeURIComponent()`会转义路径名中的`/`字符，这可能不是你想要的。它还会将查询参数中的空格转换为`%20`，尽管在 URL 的这部分中应该用`+`转义空格。

所有这些传统函数的根本问题在于，它们试图对 URL 的所有部分应用单一的编码方案，而事实上 URL 的不同部分使用不同的编码。如果你想要一个格式正确且编码正确的 URL，解决方案就是简单地使用 URL 类来进行所有的 URL 操作。

# 11.10 定时器

自 JavaScript 诞生以来，Web 浏览器就定义了两个函数——`setTimeout()`和`setInterval()`——允许程序要求浏览器在指定的时间过去后调用一个函数，或者在指定的时间间隔内重复调用函数。这些函数从未作为核心语言的一部分标准化，但它们在所有浏览器和 Node 中都有效，并且是 JavaScript 标准库的事实部分。

`setTimeout()`的第一个参数是一个函数，第二个参数是一个数字，指定在调用函数之前应该经过多少毫秒。在指定的时间过去后（如果系统繁忙可能会稍长一些），函数将被调用，不带任何参数。这里，例如，是三个`setTimeout()`调用，分别在一秒、两秒和三秒后打印控制台消息：

```js
setTimeout(() => { console.log("Ready..."); }, 1000);
setTimeout(() => { console.log("set..."); }, 2000);
setTimeout(() => { console.log("go!"); }, 3000);
```

请注意，`setTimeout()`在返回之前不会等待时间过去。这个示例中的三行代码几乎立即运行，但在经过 1,000 毫秒后才会发生任何事情。

如果省略`setTimeout()`的第二个参数，则默认为 0。然而，这并不意味着您指定的函数会立即被调用。相反，该函数被注册为“尽快”调用。如果浏览器忙于处理用户输入或其他事件，可能需要 10 毫秒或更长时间才能调用该函数。

`setTimeout()`注册一个函数，该函数将在一次调用后被调用。有时，该函数本身会调用`setTimeout()`以安排在将来的某个时间再次调用。然而，如果要重复调用一个函数，通常更简单的方法是使用`setInterval()`。`setInterval()`接受与`setTimeout()`相同的两个参数，但每当指定的毫秒数（大约）过去时，它会重复调用函数。

`setTimeout()`和`setInterval()`都会返回一个值。如果将此值保存在变量中，您随后可以使用它通过传递给`clearTimeout()`或`clearInterval()`来取消函数的执行。返回的值在 Web 浏览器中通常是一个数字，在 Node 中是一个对象。实际类型并不重要，您应该将其视为不透明值。您可以使用此值的唯一操作是将其传递给`clearTimeout()`以取消使用`setTimeout()`注册的函数的执行（假设尚未调用）或停止使用`setInterval()`注册的函数的重复执行。

这是一个示例，演示了如何使用`setTimeout()`、`setInterval()`和`clearInterval()`来显示一个简单的数字时钟与 Console API：

```js
// Once a second: clear the console and print the current time
let clock = setInterval(() => {
    console.clear();
    console.log(new Date().toLocaleTimeString());
}, 1000);

// After 10 seconds: stop the repeating code above.
setTimeout(() => { clearInterval(clock); }, 10000);
```

当我们讨论异步编程时，我们将再次看到`setTimeout()`和`setInterval()`，详见第十三章。

# 11.11 总结

学习一门编程语言不仅仅是掌握语法。同样重要的是研究标准库，以便熟悉语言附带的所有工具。本章记录了 JavaScript 的标准库，其中包括：

+   重要的数据结构，如 Set、Map 和类型化数组。

+   用于处理日期和 URL 的 Date 和 URL 类。

+   JavaScript 的正则表达式语法及其用于文本模式匹配的 RegExp 类。

+   JavaScript 的国际化库，用于格式化日期、时间和数字以及对字符串进行排序。

+   用于序列化和反序列化简单数据结构的`JSON`对象和用于记录消息的`console`对象。

¹ 这里记录的并非 JavaScript 语言规范定义的所有内容：这里记录的一些类和函数首先是在 Web 浏览器中实现的，然后被 Node 采用，使它们成为 JavaScript 标准库的事实成员。

² 这种可预测的迭代顺序是 JavaScript 集合中的另一件事，可能会让 Python 程序员感到惊讶。

³ 当 Web 浏览器添加对 WebGL 图形的支持时，类型化数组首次引入到客户端 JavaScript 中。ES6 中的新功能是它们已被提升为核心语言特性。

⁴ 除了在字符类（方括号）内部，`\b`匹配退格字符。

⁵ 使用正则表达式解析 URL 并不是一个好主意。请参见§11.9 以获取更健壮的 URL 解析器。

⁶ C 程序员将从`printf()`函数中认出许多这些字符序列。
