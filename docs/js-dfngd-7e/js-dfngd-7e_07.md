# 第七章：数组

本章介绍了数组，这是 JavaScript 和大多数其他编程语言中的一种基本数据类型。*数组*是一个有序的值集合。每个值称为一个*元素*，每个元素在数组中有一个数值位置，称为其*索引*。JavaScript 数组是*无类型*的：数组元素可以是任何类型，同一数组的不同元素可以是不同类型。数组元素甚至可以是对象或其他数组，这使您可以创建复杂的数据结构，例如对象数组和数组数组。JavaScript 数组是*基于零*的，并使用 32 位索引：第一个元素的索引为 0，最大可能的索引为 4294967294（2³²−2），最大数组大小为 4,294,967,295 个元素。JavaScript 数组是*动态*的：它们根据需要增长或缩小，并且在创建数组时无需声明固定大小，也无需在大小更改时重新分配。JavaScript 数组可能是*稀疏*的：元素不必具有连续的索引，可能存在间隙。每个 JavaScript 数组都有一个`length`属性。对于非稀疏数组，此属性指定数组中的元素数量。对于稀疏数组，`length`大于任何元素的最高索引。

JavaScript 数组是 JavaScript 对象的一种特殊形式，数组索引实际上只是整数属性名。我们将在本章的其他地方更详细地讨论数组的特殊性。实现通常会优化数组，使得对数值索引的数组元素的访问通常比对常规对象属性的访问要快得多。

数组从`Array.prototype`继承属性，该属性定义了一组丰富的数组操作方法，涵盖在§7.8 中。这些方法大多是*通用*的，这意味着它们不仅适用于真实数组，还适用于任何“类似数组的对象”。我们将在§7.9 中讨论类似数组的对象。最后，JavaScript 字符串的行为类似于字符数组，我们将在§7.10 中讨论这一点。

ES6 引入了一组被统称为“类型化数组”的新数组类。与常规的 JavaScript 数组不同，类型化数组具有固定的长度和固定的数值元素类型。它们提供高性能和对二进制数据的字节级访问，并在§11.2 中有所涉及。

# 7.1 创建数组

有几种创建数组的方法。接下来的小节将解释如何使用以下方式创建数组：

+   数组字面量

+   可迭代对象上的`...`展开运算符

+   `Array()`构造函数

+   `Array.of()`和`Array.from()`工厂方法

## 7.1.1 数组字面量

创造数组最简单的方法是使用数组字面量，它只是方括号内以逗号分隔的数组元素列表。例如：

```js
let empty = [];                 // An array with no elements
let primes = [2, 3, 5, 7, 11];  // An array with 5 numeric elements
let misc = [ 1.1, true, "a", ]; // 3 elements of various types + trailing comma
```

数组字面量中的值不必是常量；它们可以是任意表达式：

```js
let base = 1024;
let table = [base, base+1, base+2, base+3];
```

数组字面量可以包含对象字面量或其他数组字面量：

```js
let b = [[1, {x: 1, y: 2}], [2, {x: 3, y: 4}]];
```

如果数组字面量中包含多个连续的逗号，且之间没有值，那么该数组是稀疏的（参见§7.3）。省略值的数组元素并不存在，但如果查询它们，则看起来像是`undefined`：

```js
let count = [1,,3]; // Elements at indexes 0 and 2\. No element at index 1
let undefs = [,,];  // An array with no elements but a length of 2
```

数组字面量语法允许有可选的尾随逗号，因此`[,,]`的长度为 2，而不是 3。

## 7.1.2 展开运算符

在 ES6 及更高版本中，您可以使用“展开运算符”`...`将一个数组的元素包含在一个数组字面量中：

```js
let a = [1, 2, 3];
let b = [0, ...a, 4];  // b == [0, 1, 2, 3, 4]
```

这三个点“展开”数组`a`，使得它的元素成为正在创建的数组字面量中的元素。就好像`...a`被数组`a`的元素替换，字面上列为封闭数组字面量的一部分。 （请注意，尽管我们称这三个点为展开运算符，但这不是一个真正的运算符，因为它只能在数组字面量中使用，并且正如我们将在本书后面看到的，函数调用。）

展开运算符是创建（浅层）数组副本的便捷方式：

```js
let original = [1,2,3];
let copy = [...original];
copy[0] = 0;  // Modifying the copy does not change the original
original[0]   // => 1
```

展开运算符适用于任何可迭代对象。(*可迭代*对象是`for/of`循环迭代的对象；我们首次在§5.4.4 中看到它们，并且我们将在第十二章中看到更多关于它们的内容。) 字符串是可迭代的，因此您可以使用展开运算符将任何字符串转换为由单个字符字符串组成的数组：

```js
let digits = [..."0123456789ABCDEF"];
digits // => ["0","1","2","3","4","5","6","7","8","9","A","B","C","D","E","F"]
```

集合对象（§11.1.1）是可迭代的，因此从数组中删除重复元素的简单方法是将数组转换为集合，然后立即使用展开运算符将集合转换回数组：

```js
let letters = [..."hello world"];
[...new Set(letters)]  // => ["h","e","l","o"," ","w","r","d"]
```

## 7.1.3 Array() 构造函数

另一种创建数组的方法是使用`Array()`构造函数。您可以以三种不同的方式调用此构造函数： 

+   不带参数调用它：

    ```js
    let a = new Array();
    ```

    此方法创建一个没有元素的空数组，等同于数组字面量`[]`。

+   使用单个数字参数调用它，指定长度：

    ```js
    let a = new Array(10);
    ```

    这种技术创建具有指定长度的数组。当您事先知道将需要多少元素时，可以使用`Array()`构造函数的这种形式来预先分配数组。请注意，数组中不存储任何值，并且数组索引属性“0”、“1”等甚至未为数组定义。

+   明确指定两个或更多数组元素或单个非数值元素：

    ```js
    let a = new Array(5, 4, 3, 2, 1, "testing, testing");
    ```

    在这种形式中，构造函数参数成为新数组的元素。几乎总是比使用`Array()`构造函数更简单的是使用数组字面量。

## 7.1.4 Array.of()

当使用一个数值参数调用`Array()`构造函数时，它将该参数用作数组长度。但是，当使用多个数值参数调用时，它将这些参数视为要创建的数组的元素。这意味着`Array()`构造函数不能用于创建具有单个数值元素的数组。

在 ES6 中，`Array.of()`函数解决了这个问题：它是一个工厂方法，使用其参数值（无论有多少个）作为数组元素创建并返回一个新数组：

```js
Array.of()        // => []; returns empty array with no arguments
Array.of(10)      // => [10]; can create arrays with a single numeric argument
Array.of(1,2,3)   // => [1, 2, 3]
```

## 7.1.5 Array.from()

`Array.from`是 ES6 中引入的另一个数组工厂方法。它期望一个可迭代或类似数组的对象作为其第一个参数，并返回一个包含该对象元素的新数组。对于可迭代参数，`Array.from(iterable)`的工作方式类似于展开运算符`[...iterable]`。这也是制作数组副本的简单方法：

```js
let copy = Array.from(original);
```

`Array.from()`也很重要，因为它定义了一种使类似数组对象的真数组副本的方法。类似数组的对象是具有数值长度属性并且具有存储值的属性的非数组对象，这些属性的名称恰好是整数。在使用客户端 JavaScript 时，某些 Web 浏览器方法的返回值是类似数组的，如果您首先将它们转换为真数组，那么使用它们可能会更容易：

```js
let truearray = Array.from(arraylike);
```

`Array.from()`还接受一个可选的第二个参数。如果将一个函数作为第二个参数传递，那么在构建新数组时，源对象的每个元素都将传递给您指定的函数，并且函数的返回值将存储在数组中，而不是原始值。（这非常类似于稍后将在本章介绍的数组`map()`方法，但在构建数组时执行映射比构建数组然后将其映射到另一个新数组更有效。）

# 7.2 读取和写入数组元素

使用`[]`运算符访问数组元素。方括号左侧应该是数组的引用。方括号内应该是一个非负整数值的任意表达式。你可以使用这种语法来读取和写入数组元素的值。因此，以下都是合法的 JavaScript 语句：

```js
let a = ["world"];     // Start with a one-element array
let value = a[0];      // Read element 0
a[1] = 3.14;           // Write element 1
let i = 2;
a[i] = 3;              // Write element 2
a[i + 1] = "hello";    // Write element 3
a[a[i]] = a[0];        // Read elements 0 and 2, write element 3
```

数组的特殊之处在于，当你使用非负整数且小于 2³²–1 的属性名时，数组会自动为你维护`length`属性的值。例如，在前面的例子中，我们创建了一个只有一个元素的数组`a`。然后我们在索引 1、2 和 3 处分配了值。随着我们的操作，数组的`length`属性也发生了变化，因此：

```js
a.length       // => 4
```

请记住，数组是一种特殊类型的对象。用于访问数组元素的方括号与用于访问对象属性的方括号工作方式相同。JavaScript 将你指定的数值数组索引转换为字符串——索引`1`变为字符串`"1"`——然后将该字符串用作属性名。将索引从数字转换为字符串没有什么特殊之处：你也可以对常规对象这样做：

```js
let o = {};    // Create a plain object
o[1] = "one";  // Index it with an integer
o["1"]         // => "one"; numeric and string property names are the same
```

清楚地区分*数组索引*和*对象属性名*是有帮助的。所有索引都是属性名，但只有介于 0 和 2³²–2 之间的整数属性名才是索引。所有数组都是对象，你可以在它们上面创建任何名称的属性。然而，如果你使用的是数组索引的属性，数组会根据需要更新它们的`length`属性。

请注意，你可以使用负数或非整数的数字对数组进行索引。当你这样做时，数字会转换为字符串，并且该字符串将用作属性名。由于名称不是非负整数，因此它被视为常规对象属性，而不是数组索引。此外，如果你使用恰好是非负整数的字符串对数组进行索引，它将表现为数组索引，而不是对象属性。如果你使用与整数相同的浮点数，情况也是如此：

```js
a[-1.23] = true;  // This creates a property named "-1.23"
a["1000"] = 0;    // This the 1001st element of the array
a[1.000] = 1;     // Array index 1\. Same as a[1] = 1;
```

数组索引只是对象属性名的一种特殊类型，这意味着 JavaScript 数组没有“越界”错误的概念。当你尝试查询任何对象的不存在属性时，你不会收到错误；你只会得到`undefined`。对于数组和对象来说，这一点同样适用：

```js
let a = [true, false]; // This array has elements at indexes 0 and 1
a[2]                   // => undefined; no element at this index.
a[-1]                  // => undefined; no property with this name.
```

# 7.3 稀疏数组

*稀疏*数组是指元素的索引不是从 0 开始的连续索引。通常，数组的`length`属性指定数组中元素的数量。如果数组是稀疏的，`length`属性的值将大于元素的数量。可以使用`Array()`构造函数创建稀疏数组，或者简单地通过分配给大于当前数组`length`的数组索引来创建稀疏数组。

```js
let a = new Array(5); // No elements, but a.length is 5.
a = [];               // Create an array with no elements and length = 0.
a[1000] = 0;          // Assignment adds one element but sets length to 1001.
```

我们稍后会看到，你也可以使用`delete`运算符使数组变得稀疏。

具有足够稀疏性的数组通常以比密集数组更慢、更节省内存的方式实现，查找这种数组中的元素将花费与常规对象属性查找相同的时间。

注意，当你在数组字面量中省略一个值（使用重复逗号，如`[1,,3]`），结果得到的数组是稀疏的，省略的元素简单地不存在：

```js
let a1 = [,];           // This array has no elements and length 1
let a2 = [undefined];   // This array has one undefined element
0 in a1                 // => false: a1 has no element with index 0
0 in a2                 // => true: a2 has the undefined value at index 0
```

理解稀疏数组是理解 JavaScript 数组真正本质的重要部分。然而，在实践中，你将使用的大多数 JavaScript 数组都不会是稀疏的。而且，如果你确实需要使用稀疏数组，你的代码可能会像对待具有`undefined`元素的非稀疏数组一样对待它。

# 7.4 数组长度

每个数组都有一个`length`属性，正是这个属性使数组与常规 JavaScript 对象不同。对于密集数组（即非稀疏数组），`length`属性指定数组中元素的数量。其值比数组中最高索引多一：

```js
[].length             // => 0: the array has no elements
["a","b","c"].length  // => 3: highest index is 2, length is 3
```

当数组是稀疏的时，`length`属性大于元素数量，我们只能说`length`保证大于数组中每个元素的索引。换句话说，数组（稀疏或非稀疏）永远不会有索引大于或等于其`length`的元素。为了保持这个不变量，数组有两个特殊行为。我们上面描述的第一个：如果您为索引`i`大于或等于数组当前`length`的数组元素分配一个值，`length`属性的值将设置为`i+1`。

数组为了保持长度不变的第二个特殊行为是，如果您将`length`属性设置为小于当前值的非负整数`n`，则任何索引大于或等于`n`的数组元素将从数组中删除：

```js
a = [1,2,3,4,5];     // Start with a 5-element array.
a.length = 3;        // a is now [1,2,3].
a.length = 0;        // Delete all elements.  a is [].
a.length = 5;        // Length is 5, but no elements, like new Array(5)
```

您还可以将数组的`length`属性设置为大于当前值的值。这样做实际上并不向数组添加任何新元素；它只是在数组末尾创建了一个稀疏区域。

# 7.5 添加和删除数组元素

我们已经看到向数组添加元素的最简单方法：只需为新索引分配值：

```js
let a = [];      // Start with an empty array.
a[0] = "zero";   // And add elements to it.
a[1] = "one";
```

您还可以使用`push()`方法将一个或多个值添加到数组的末尾：

```js
let a = [];           // Start with an empty array
a.push("zero");       // Add a value at the end.  a = ["zero"]
a.push("one", "two"); // Add two more values.  a = ["zero", "one", "two"]
```

将值推送到数组`a`上与将值分配给`a[a.length]`相同。您可以使用`unshift()`方法（在§7.8 中描述）在数组的开头插入一个值，将现有数组元素移动到更高的索引。`pop()`方法是`push()`的相反操作：它删除数组的最后一个元素并返回它，将数组的长度减少 1。类似地，`shift()`方法删除并返回数组的第一个元素，将长度减 1 并将所有元素向下移动到比当前索引低一个索引。有关这些方法的更多信息，请参阅§7.8。

您可以使用`delete`运算符删除数组元素，就像您可以删除对象属性一样：

```js
let a = [1,2,3];
delete a[2];   // a now has no element at index 2
2 in a         // => false: no array index 2 is defined
a.length       // => 3: delete does not affect array length
```

删除数组元素与将`undefined`分配给该元素类似（但略有不同）。请注意，使用`delete`删除数组元素不会改变`length`属性，并且不会将具有更高索引的元素向下移动以填补被删除属性留下的空白。如果从数组中删除一个元素，数组将变得稀疏。

正如我们上面看到的，您也可以通过将`length`属性设置为新的所需长度来从数组末尾删除元素。

最后，`splice()`是用于插入、删除或替换数组元素的通用方法。它改变`length`属性并根据需要将数组元素移动到更高或更低的索引。有关详细信息，请参阅§7.8。

# 7.6 遍历数组

从 ES6 开始，遍历数组（或任何可迭代对象）的最简单方法是使用`for/of`循环，这在§5.4.4 中有详细介绍：

```js
let letters = [..."Hello world"];  // An array of letters
let string = "";
for(let letter of letters) {
    string += letter;
}
string  // => "Hello world"; we reassembled the original text
```

`for/of`循环使用的内置数组迭代器按升序返回数组的元素。对于稀疏数组，它没有特殊行为，只是对于不存在的数组元素返回`undefined`。

如果您想要使用`for/of`循环遍历数组并需要知道每个数组元素的索引，请使用数组的`entries()`方法，以及解构赋值，如下所示：

```js
let everyother = "";
for(let [index, letter] of letters.entries()) {
    if (index % 2 === 0) everyother += letter;  // letters at even indexes
}
everyother  // => "Hlowrd"
```

另一种遍历数组的好方法是使用`forEach()`。这不是`for`循环的新形式，而是一种提供数组迭代功能的数组方法。您将一个函数传递给数组的`forEach()`方法，`forEach()`在数组的每个元素上调用您的函数一次：

```js
let uppercase = "";
letters.forEach(letter => {  // Note arrow function syntax here
    uppercase += letter.toUpperCase();
});
uppercase  // => "HELLO WORLD"
```

正如你所期望的那样，`forEach()`按顺序迭代数组，并将数组索引作为第二个参数传递给你的函数，这有时很有用。与`for/of`循环不同，`forEach()`知道稀疏数组，并且不会为不存在的元素调用你的函数。

§7.8.1 详细介绍了`forEach()`方法。该部分还涵盖了类似`map()`和`filter()`的相关方法，执行特定类型的数组迭代。

您还可以使用传统的`for`循环遍历数组的元素（§5.4.3）：

```js
let vowels = "";
for(let i = 0; i < letters.length; i++) { // For each index in the array
    let letter = letters[i];              // Get the element at that index
    if (/[aeiou]/.test(letter)) {         // Use a regular expression test
        vowels += letter;                 // If it is a vowel, remember it
    }
}
vowels  // => "eoo"
```

在嵌套循环或其他性能关键的情况下，有时会看到基本的数组迭代循环被写成只查找一次数组长度而不是在每次迭代中查找。以下两种`for`循环形式都是惯用的，尽管不是特别常见，并且在现代 JavaScript 解释器中，它们是否会对性能产生影响并不清楚：

```js
// Save the array length into a local variable
for(let i = 0, len = letters.length; i < len; i++) {
    // loop body remains the same
}

// Iterate backwards from the end of the array to the start
for(let i = letters.length-1; i >= 0; i--) {
    // loop body remains the same
}
```

这些示例假设数组是密集的，并且所有元素都包含有效数据。如果不是这种情况，您应该在使用数组元素之前对其进行测试。如果要跳过未定义和不存在的元素，您可以这样写：

```js
for(let i = 0; i < a.length; i++) {
    if (a[i] === undefined) continue; // Skip undefined + nonexistent elements
    // loop body here
}
```

# 7.7 多维数组

JavaScript 不支持真正的多维数组，但可以用数组的数组来近似实现。要访问数组中的值，只需简单地两次使用`[]`运算符。例如，假设变量`matrix`是一个包含数字数组的数组。`matrix[x]`中的每个元素都是一个数字数组。要访问这个数组中的特定数字，你可以写成`matrix[x][y]`。以下是一个使用二维数组作为乘法表的具体示例：

```js
// Create a multidimensional array
let table = new Array(10);               // 10 rows of the table
for(let i = 0; i < table.length; i++) {
    table[i] = new Array(10);            // Each row has 10 columns
}

// Initialize the array
for(let row = 0; row < table.length; row++) {
    for(let col = 0; col < table[row].length; col++) {
        table[row][col] = row*col;
    }
}

// Use the multidimensional array to compute 5*7
table[5][7]  // => 35
```

# 7.8 数组方法

前面的部分重点介绍了处理数组的基本 JavaScript 语法。然而，一般来说，Array 类定义的方法是最强大的。接下来的部分记录了这些方法。在阅读这些方法时，请记住其中一些方法会修改调用它们的数组，而另一些方法则会保持数组不变。其中一些方法会返回一个数组：有时这是一个新数组，原始数组保持不变。其他时候，一个方法会就地修改数组，并同时返回修改后的数组的引用。

接下来的各小节涵盖了一组相关的数组方法：

+   迭代方法循环遍历数组的元素，通常在每个元素上调用您指定的函数。

+   栈和队列方法向数组的开头和结尾添加和移除数组元素。

+   子数组方法用于提取、删除、插入、填充和复制较大数组的连续区域。

+   搜索和排序方法用于在数组中定位元素并对数组元素进行排序。

以下小节还涵盖了 Array 类的静态方法以及一些用于连接数组和将数组转换为字符串的杂项方法。

## 7.8.1 数组迭代方法

本节描述的方法通过将数组元素按顺序传递给您提供的函数来迭代数组，并提供了方便的方法来迭代、映射、过滤、测试和减少数组。

然而，在详细解释这些方法之前，值得对它们做一些概括。首先，所有这些方法都接受一个函数作为它们的第一个参数，并为数组的每个元素（或某些元素）调用该函数。如果数组是稀疏的，您传递的函数不会为不存在的元素调用。在大多数情况下，您提供的函数会被调用三个参数：数组元素的值、数组元素的索引和数组本身。通常，您只需要第一个参数值，可以忽略第二和第三个值。

在下面的小节中描述的大多数迭代器方法都接受一个可选的第二个参数。如果指定了，函数将被调用，就好像它是第二个参数的方法一样。也就是说，您传递的第二个参数将成为您作为第一个参数传递的函数内部的 `this` 关键字的值。您传递的函数的返回值通常很重要，但不同的方法以不同的方式处理返回值。这里描述的方法都不会修改调用它们的数组（尽管您传递的函数可以修改数组，当然）。

每个这些函数都是以一个函数作为其第一个参数调用的，很常见的是在方法调用表达式中定义该函数内联，而不是使用在其他地方定义的现有函数。箭头函数语法（参见§8.1.3）与这些方法特别配合，我们将在接下来的示例中使用它。

### forEach()

`forEach()` 方法遍历数组，为每个元素调用您指定的函数。正如我们所描述的，您将函数作为第一个参数传递给 `forEach()`。然后，`forEach()` 使用三个参数调用您的函数：数组元素的值，数组元素的索引和数组本身。如果您只关心数组元素的值，您可以编写一个只有一个参数的函数——额外的参数将被忽略：

```js
let data = [1,2,3,4,5], sum = 0;
// Compute the sum of the elements of the array
data.forEach(value => { sum += value; });          // sum == 15

// Now increment each array element
data.forEach(function(v, i, a) { a[i] = v + 1; }); // data == [2,3,4,5,6]
```

请注意，`forEach()` 不提供在所有元素被传递给函数之前终止迭代的方法。也就是说，您无法像在常规 `for` 循环中使用 `break` 语句那样使用。

### map()

`map()` 方法将调用它的数组的每个元素传递给您指定的函数，并返回一个包含您函数返回的值的数组。例如：

```js
let a = [1, 2, 3];
a.map(x => x*x)   // => [1, 4, 9]: the function takes input x and returns x*x
```

传递给 `map()` 的函数的调用方式与传递给 `forEach()` 的函数相同。然而，对于 `map()` 方法，您传递的函数应该返回一个值。请注意，`map()` 返回一个新数组：它不会修改调用它的数组。如果该数组是稀疏的，您的函数将不会为缺失的元素调用，但返回的数组将与原始数组一样稀疏：它将具有相同的长度和相同的缺失元素。

### filter()

`filter()` 方法返回一个包含调用它的数组的元素子集的数组。传递给它的函数应该是谓词：一个返回 `true` 或 `false` 的函数。谓词的调用方式与 `forEach()` 和 `map()` 相同。如果返回值为 `true`，或者可以转换为 `true` 的值，则传递给谓词的元素是子集的成员，并将添加到将成为返回值的数组中。示例：

```js
let a = [5, 4, 3, 2, 1];
a.filter(x => x < 3)         // => [2, 1]; values less than 3
a.filter((x,i) => i%2 === 0) // => [5, 3, 1]; every other value
```

请注意，`filter()` 跳过稀疏数组中的缺失元素，并且其返回值始终是密集的。要填补稀疏数组中的空白，您可以这样做：

```js
let dense = sparse.filter(() => true);
```

要填补空白并删除未定义和空元素，您可以使用 `filter`，如下所示：

```js
a = a.filter(x => x !== undefined && x !== null);
```

### find() 和 findIndex()

`find()` 和 `findIndex()` 方法类似于 `filter()`，它们遍历数组，寻找使谓词函数返回真值的元素。然而，这两种方法在谓词第一次找到元素时停止遍历。当这种情况发生时，`find()` 返回匹配的元素，而 `findIndex()` 返回匹配元素的索引。如果找不到匹配的元素，`find()` 返回 `undefined`，而 `findIndex()` 返回 `-1`：

```js
let a = [1,2,3,4,5];
a.findIndex(x => x === 3)  // => 2; the value 3 appears at index 2
a.findIndex(x => x < 0)    // => -1; no negative numbers in the array
a.find(x => x % 5 === 0)   // => 5: this is a multiple of 5
a.find(x => x % 7 === 0)   // => undefined: no multiples of 7 in the array
```

### every() 和 some()

`every()` 和 `some()` 方法是数组谓词：它们将您指定的谓词函数应用于数组的元素，然后返回 `true` 或 `false`。

`every()` 方法类似于数学中的“对于所有”量词 ∀：仅当它的谓词函数对数组中的所有元素返回 `true` 时，它才返回 `true`：

```js
let a = [1,2,3,4,5];
a.every(x => x < 10)      // => true: all values are < 10.
a.every(x => x % 2 === 0) // => false: not all values are even.
```

`some()`方法类似于数学中的“存在”量词∃：如果数组中存在至少一个使谓词返回`true`的元素，则返回`true`，如果谓词对数组的所有元素返回`false`，则返回`false`：

```js
let a = [1,2,3,4,5];
a.some(x => x%2===0)  // => true; a has some even numbers.
a.some(isNaN)         // => false; a has no non-numbers.
```

请注意，`every()`和`some()`都会在他们知道要返回的值时停止迭代数组元素。`some()`在您的谓词第一次返回`true`时返回`true`，只有在您的谓词始终返回`false`时才会遍历整个数组。`every()`则相反：当您的谓词第一次返回`false`时返回`false`，只有在您的谓词始终返回`true`时才会迭代所有元素。还要注意，按照数学约定，当在空数组上调用`every()`时，`every()`返回`true`，而在空数组上调用`some`时，`some`返回`false`。

### reduce()和 reduceRight()

`reduce()`和`reduceRight()`方法使用您指定的函数组合数组的元素，以产生单个值。这是函数式编程中的常见操作，也称为“注入”和“折叠”。示例有助于说明它是如何工作的：

```js
let a = [1,2,3,4,5];
a.reduce((x,y) => x+y, 0)          // => 15; the sum of the values
a.reduce((x,y) => x*y, 1)          // => 120; the product of the values
a.reduce((x,y) => (x > y) ? x : y) // => 5; the largest of the values
```

`reduce()`接受两个参数。第一个是执行减少操作的函数。这个减少函数的任务是以某种方式将两个值组合或减少为单个值，并返回该减少值。在我们这里展示的示例中，这些函数通过相加、相乘和选择最大值来组合两个值。第二个（可选）参数是传递给函数的初始值。

使用`reduce()`的函数与`forEach()`和`map()`中使用的函数不同。熟悉的值、索引和数组值作为第二、第三和第四个参数传递。第一个参数是到目前为止减少的累积结果。在第一次调用函数时，这个第一个参数是您作为`reduce()`的第二个参数传递的初始值。在后续调用中，它是前一个函数调用返回的值。在第一个示例中，减少函数首先使用参数 0 和 1 进行调用。它将它们相加并返回 1。然后再次使用参数 1 和 2 调用它并返回 3。接下来，它计算 3+3=6，然后 6+4=10，最后 10+5=15。这个最终值 15 成为`reduce()`的返回值。

您可能已经注意到此示例中对`reduce()`的第三次调用只有一个参数：没有指定初始值。当您像这样调用`reduce()`而没有初始值时，它将使用数组的第一个元素作为初始值。这意味着减少函数的第一次调用将具有数组的第一个和第二个元素作为其第一个和第二个参数。在求和和乘积示例中，我们可以省略初始值参数。

在空数组上调用`reduce()`且没有初始值参数会导致 TypeError。如果您只使用一个值调用它——要么是一个具有一个元素且没有初始值的数组，要么是一个空数组和一个初始值——它将简单地返回那个值，而不会调用减少函数。

`reduceRight()`的工作方式与`reduce()`完全相同，只是它从最高索引到最低索引（从右到左）处理数组，而不是从最低到最高。如果减少操作具有从右到左的结合性，您可能希望这样做，例如：

```js
// Compute 2^(3⁴).  Exponentiation has right-to-left precedence
let a = [2, 3, 4];
a.reduceRight((acc,val) => Math.pow(val,acc)) // => 2.4178516392292583e+24
```

请注意，`reduce()`和`reduceRight()`都不接受一个可选参数，该参数指定要调用减少函数的`this`值。可选的初始值参数代替了它。如果您需要将您的减少函数作为特定对象的方法调用，请参阅`Function.bind()`方法（§8.7.5）。

到目前为止所展示的示例都是为了简单起见而是数值的，但`reduce()`和`reduceRight()`并不仅仅用于数学计算。任何能将两个值（如两个对象）合并为相同类型值的函数都可以用作缩减函数。另一方面，使用数组缩减表达的算法可能很快变得复杂且难以理解，你可能会发现，如果使用常规的循环结构来处理数组，那么阅读、编写和推理代码会更容易。

## 7.8.2 使用 flat()和flatMap()展平数组

在 ES2019 中，`flat()`方法创建并返回一个新数组，其中包含调用它的数组的相同元素，除了那些本身是数组的元素被“展平”到返回的数组中。例如：

```js
[1, [2, 3]].flat()    // => [1, 2, 3]
[1, [2, [3]]].flat()  // => [1, 2, [3]]
```

当不带参数调用时，`flat()`会展平一层嵌套。原始数组中本身是数组的元素会被展平，但*那些*数组的元素不会被展平。如果你想展平更多层次，请向`flat()`传递一个数字：

```js
let a = [1, [2, [3, [4]]]];
a.flat(1)   // => [1, 2, [3, [4]]]
a.flat(2)   // => [1, 2, 3, [4]]
a.flat(3)   // => [1, 2, 3, 4]
a.flat(4)   // => [1, 2, 3, 4]
```

`flatMap()`方法的工作方式与`map()`方法相同（参见`map()`），只是返回的数组会自动展平，就像传递给`flat()`一样。也就是说，调用`a.flatMap(f)`与（但更有效率）`a.map(f).flat()`相同：

```js
let phrases = ["hello world", "the definitive guide"];
let words = phrases.flatMap(phrase => phrase.split(" "));
words // => ["hello", "world", "the", "definitive", "guide"];
```

你可以将`flatMap()`视为`map()`的一般化，允许输入数组的每个元素映射到输出数组的任意数量的元素。特别是，`flatMap()`允许你将输入元素映射到一个空数组，这在输出数组中展平为无内容：

```js
// Map non-negative numbers to their square roots
[-2, -1, 1, 2].flatMap(x => x < 0 ? [] : Math.sqrt(x)) // => [1, 2**0.5]
```

## 7.8.3 使用 concat()添加数组

`concat()`方法创建并返回一个新数组，其中包含调用`concat()`的原始数组的元素，后跟`concat()`的每个参数。如果其中任何参数本身是一个数组，则连接的是数组元素，而不是数组本身。但请注意，`concat()`不会递归展平数组的数组。`concat()`不会修改调用它的数组：

```js
let a = [1,2,3];
a.concat(4, 5)          // => [1,2,3,4,5]
a.concat([4,5],[6,7])   // => [1,2,3,4,5,6,7]; arrays are flattened
a.concat(4, [5,[6,7]])  // => [1,2,3,4,5,[6,7]]; but not nested arrays
a                       // => [1,2,3]; the original array is unmodified
```

注意`concat()`会在调用时创建原始数组的新副本。在许多情况下，这是正确的做法，但这是一个昂贵的操作。如果你发现自己写的代码像`a = a.concat(x)`，那么你应该考虑使用`push()`或`splice()`来就地修改数组，而不是创建一个新数组。

## 7.8.4 使用 push()、pop()、shift()和 unshift()实现栈和队列

`push()`和`pop()`方法允许你像处理栈一样处理数组。`push()`方法将一个或多个新元素附加到数组的末尾，并返回数组的新长度。与`concat()`不同，`push()`不会展平数组参数。`pop()`方法则相反：它删除数组的最后一个元素，减少数组长度，并返回它删除的值。请注意，这两种方法都会就地修改数组。`push()`和`pop()`的组合允许你使用 JavaScript 数组来实现先进后出的栈。例如：

```js
let stack = [];       // stack == []
stack.push(1,2);      // stack == [1,2];
stack.pop();          // stack == [1]; returns 2
stack.push(3);        // stack == [1,3]
stack.pop();          // stack == [1]; returns 3
stack.push([4,5]);    // stack == [1,[4,5]]
stack.pop()           // stack == [1]; returns [4,5]
stack.pop();          // stack == []; returns 1
```

`push()`方法不会展平你传递给它的数组，但如果你想将一个数组的所有元素推到另一个数组中，你可以使用展开运算符（§8.3.4）来显式展平它：

```js
a.push(...values);
```

`unshift()`和`shift()`方法的行为与`push()`和`pop()`类似，只是它们是从数组的开头而不是末尾插入和删除元素。`unshift()`在数组开头添加一个或多个元素，将现有数组元素向较高的索引移动以腾出空间，并返回数组的新长度。`shift()`移除并返回数组的第一个元素，将所有后续元素向下移动一个位置以占据数组开头的新空间。您可以使用`unshift()`和`shift()`来实现堆栈，但与使用`push()`和`pop()`相比效率较低，因为每次在数组开头添加或删除元素时都需要将数组元素向上或向下移动。不过，您可以通过使用`push()`在数组末尾添加元素并使用`shift()`从数组开头删除元素来实现队列数据结构：

```js
let q = [];            // q == []
q.push(1,2);           // q == [1,2]
q.shift();             // q == [2]; returns 1
q.push(3)              // q == [2, 3]
q.shift()              // q == [3]; returns 2
q.shift()              // q == []; returns 3
```

`unshift()`的一个值得注意的特点是，当向`unshift()`传递多个参数时，它们会一次性插入，这意味着它们以与逐个插入时不同的顺序出现在数组中：

```js
let a = [];            // a == []
a.unshift(1)           // a == [1]
a.unshift(2)           // a == [2, 1]
a = [];                // a == []
a.unshift(1,2)         // a == [1, 2]
```

## 7.8.5 使用 slice()、splice()、fill()和 copyWithin()创建子数组

数组定义了一些在连续区域、子数组或数组的“切片”上工作的方法。以下部分描述了用于提取、替换、填充和复制切片的方法。

### slice()

`slice()`方法返回指定数组的*切片*或子数组。它的两个参数指定要返回的切片的起始和结束。返回的数组包含由第一个参数指定的元素和直到第二个参数指定的元素之前的所有后续元素（不包括该元素）。如果只指定一个参数，则返回的数组包含从起始位置到数组末尾的所有元素。如果任一参数为负数，则它指定相对于数组长度的数组元素。例如，参数-1 指定数组中的最后一个元素，参数-2 指定该元素之前的元素。请注意，`slice()`不会修改调用它的数组。以下是一些示例：

```js
let a = [1,2,3,4,5];
a.slice(0,3);    // Returns [1,2,3]
a.slice(3);      // Returns [4,5]
a.slice(1,-1);   // Returns [2,3,4]
a.slice(-3,-2);  // Returns [3]
```

### splice()

`splice()`是一个通用的方法，用于向数组中插入或删除元素。与`slice()`和`concat()`不同，`splice()`会修改调用它的数组。请注意，`splice()`和`slice()`的名称非常相似，但执行的操作有很大不同。

`splice()`可以从数组中删除元素、向数组中插入新元素，或同时执行这两个操作。数组中插入或删除点之后的元素的索引会根据需要增加或减少，以使它们与数组的其余部分保持连续。`splice()`的第一个参数指定插入和/或删除开始的数组位置。第二个参数指定应从数组中删除的元素数量。（请注意，这是这两种方法之间的另一个区别。`slice()`的第二个参数是结束位置。`splice()`的第二个参数是长度。）如果省略了第二个参数，则从起始元素到数组末尾的所有数组元素都将被删除。`splice()`返回一个包含已删除元素的数组，如果没有删除元素，则返回一个空数组。例如：

```js
let a = [1,2,3,4,5,6,7,8];
a.splice(4)    // => [5,6,7,8]; a is now [1,2,3,4]
a.splice(1,2)  // => [2,3]; a is now [1,4]
a.splice(1,1)  // => [4]; a is now [1]
```

`splice()`的前两个参数指定要删除的数组元素。这些参数后面可以跟任意数量的额外参数，这些参数指定要插入到数组中的元素，从第一个参数指定的位置开始。例如：

```js
let a = [1,2,3,4,5];
a.splice(2,0,"a","b")  // => []; a is now [1,2,"a","b",3,4,5]
a.splice(2,2,[1,2],3)  // => ["a","b"]; a is now [1,2,[1,2],3,3,4,5]
```

请注意，与`concat()`不同，`splice()`插入的是数组本身，而不是这些数组的元素。

### 填充()

`fill()`方法将数组或数组的一个片段的元素设置为指定的值。它会改变调用它的数组，并返回修改后的数组：

```js
let a = new Array(5);   // Start with no elements and length 5
a.fill(0)               // => [0,0,0,0,0]; fill the array with zeros
a.fill(9, 1)            // => [0,9,9,9,9]; fill with 9 starting at index 1
a.fill(8, 2, -1)        // => [0,9,8,8,9]; fill with 8 at indexes 2, 3
```

`fill()`的第一个参数是要设置数组元素的值。可选的第二个参数指定开始索引。如果省略，填充将从索引 0 开始。可选的第三个参数指定结束索引——将填充到该索引之前的数组元素。如果省略此参数，则数组将从开始索引填充到结束。您可以通过传递负数来指定相对于数组末尾的索引，就像对`slice()`一样。

### copyWithin()

`copyWithin()`将数组的一个片段复制到数组内的新位置。它会就地修改数组并返回修改后的数组，但不会改变数组的长度。第一个参数指定要复制第一个元素的目标索引。第二个参数指定要复制的第一个元素的索引。如果省略第二个参数，则使用 0。第三个参数指定要复制的元素片段的结束。如果省略，将使用数组的长度。从开始索引到结束索引之前的元素将被复制。您可以通过传递负数来指定相对于数组末尾的索引，就像对`slice()`一样：

```js
let a = [1,2,3,4,5];
a.copyWithin(1)       // => [1,1,2,3,4]: copy array elements up one
a.copyWithin(2, 3, 5) // => [1,1,3,4,4]: copy last 2 elements to index 2
a.copyWithin(0, -2)   // => [4,4,3,4,4]: negative offsets work, too
```

`copyWithin()`旨在作为一种高性能方法，特别适用于类型化数组（参见§11.2）。它模仿了 C 标准库中的`memmove()`函数。请注意，即使源区域和目标区域之间存在重叠，复制也会正确工作。

## 7.8.6 数组搜索和排序方法

数组实现了`indexOf()`、`lastIndexOf()`和`includes()`方法，这些方法与字符串的同名方法类似。还有`sort()`和`reverse()`方法用于重新排列数组的元素。这些方法将在接下来的小节中描述。

### indexOf()和 lastIndexOf()

`indexOf()`和`lastIndexOf()`搜索具有指定值的元素的数组，并返回找到的第一个这样的元素的索引，如果找不到则返回`-1`。`indexOf()`从开头到结尾搜索数组，`lastIndexOf()`从结尾到开头搜索：

```js
let a = [0,1,2,1,0];
a.indexOf(1)       // => 1: a[1] is 1
a.lastIndexOf(1)   // => 3: a[3] is 1
a.indexOf(3)       // => -1: no element has value 3
```

`indexOf()`和`lastIndexOf()`使用等价于`===`运算符的方式将它们的参数与数组元素进行比较。如果您的数组包含对象而不是原始值，这些方法将检查两个引用是否确实指向完全相同的对象。如果您想要实际查看对象的内容，请尝试使用带有自定义谓词函数的`find()`方法。

`indexOf()`和`lastIndexOf()`接受一个可选的第二个参数，该参数指定开始搜索的数组索引。如果省略此参数，`indexOf()`从开头开始，`lastIndexOf()`从末尾开始。第二个参数允许使用负值，并被视为从数组末尾的偏移量，就像`slice()`方法一样：例如，-1 表示数组的最后一个元素。

以下函数搜索数组中指定值的所有匹配索引，并返回一个*所有*匹配索引的数组。这演示了如何使用`indexOf()`的第二个参数来查找第一个之外的匹配项。

```js
// Find all occurrences of a value x in an array a and return an array
// of matching indexes
function findall(a, x) {
    let results = [],            // The array of indexes we'll return
        len = a.length,          // The length of the array to be searched
        pos = 0;                 // The position to search from
    while(pos < len) {           // While more elements to search...
        pos = a.indexOf(x, pos); // Search
        if (pos === -1) break;   // If nothing found, we're done.
        results.push(pos);       // Otherwise, store index in array
        pos = pos + 1;           // And start next search at next element
    }
    return results;              // Return array of indexes
}
```

请注意，字符串具有类似这些数组方法的`indexOf()`和`lastIndexOf()`方法，只是负的第二个参数被视为零。

### includes()

ES2016 的`includes()`方法接受一个参数，如果数组包含该值则返回`true`，否则返回`false`。它不会告诉您该值的索引，只会告诉您它是否存在。`includes()`方法实际上是用于数组的集合成员测试。但是请注意，数组不是集合的有效表示形式，如果您处理的元素超过几个，应该使用真正的 Set 对象（§11.1.1）。

`includes()`方法与`indexOf()`方法在一个重要方面略有不同。`indexOf()`使用与`===`运算符相同的算法进行相等性测试，该相等性算法认为非数字值与包括它本身在内的每个其他值都不同。`includes()`使用略有不同的相等性版本，它确实认为`NaN`等于它本身。这意味着`indexOf()`不会在数组中检测到`NaN`值，但`includes()`会：

```js
let a = [1,true,3,NaN];
a.includes(true)            // => true
a.includes(2)               // => false
a.includes(NaN)             // => true
a.indexOf(NaN)              // => -1; indexOf can't find NaN
```

### `sort()`

`sort()`对数组的元素进行原地排序并返回排序后的数组。当不带参数调用`sort()`时，它会按字母顺序对数组元素进行排序（如果需要，会临时将它们转换为字符串进行比较）：

```js
let a = ["banana", "cherry", "apple"];
a.sort(); // a == ["apple", "banana", "cherry"]
```

如果数组包含未定义的元素，则它们将被排序到数组的末尾。

要将数组按照字母顺序以外的某种顺序排序，您必须将比较函数作为参数传递给`sort()`。此函数决定哪个参数应该首先出现在排序后的数组中。如果第一个参数应该出现在第二个参数之前，则比较函数应返回小于零的数字。如果第一个参数应该在排序后的数组中出现在第二个参数之后，则函数应返回大于零的数字。如果两个值相等（即，如果它们的顺序无关紧要），则比较函数应返回 0。因此，例如，要将数组元素按照数字顺序而不是字母顺序排序，您可以这样做：

```js
let a = [33, 4, 1111, 222];
a.sort();               // a == [1111, 222, 33, 4]; alphabetical order
a.sort(function(a,b) {  // Pass a comparator function
    return a-b;         // Returns < 0, 0, or > 0, depending on order
});                     // a == [4, 33, 222, 1111]; numerical order
a.sort((a,b) => b-a);   // a == [1111, 222, 33, 4]; reverse numerical order
```

作为对数组项进行排序的另一个示例，您可以通过传递一个比较函数对字符串数组进行不区分大小写的字母排序，该函数在比较之前将其两个参数都转换为小写（使用`toLowerCase()`方法）：

```js
let a = ["ant", "Bug", "cat", "Dog"];
a.sort();    // a == ["Bug","Dog","ant","cat"]; case-sensitive sort
a.sort(function(s,t) {
    let a = s.toLowerCase();
    let b = t.toLowerCase();
    if (a < b) return -1;
    if (a > b) return 1;
    return 0;
});   // a == ["ant","Bug","cat","Dog"]; case-insensitive sort
```

### `reverse()`

`reverse()`方法颠倒数组的元素顺序并返回颠倒的数组。它在原地执行此操作；换句话说，它不会创建一个重新排列元素的新数组，而是在已经存在的数组中重新排列它们：

```js
let a = [1,2,3];
a.reverse();   // a == [3,2,1]
```

## 7.8.7 数组转换为字符串

Array 类定义了三种可以将数组转换为字符串的方法，通常在创建日志和错误消息时可能会使用。 （如果要以文本形式保存数组的内容以供以后重用，请使用`JSON.stringify()` [§6.8]来序列化数组，而不是使用这里描述的方法。）

`join()`方法将数组的所有元素转换为字符串并连接它们，返回生成的字符串。您可以指定一个可选的字符串，用于分隔生成的字符串中的元素。如果未指定分隔符字符串，则使用逗号：

```js
let a = [1, 2, 3];
a.join()               // => "1,2,3"
a.join(" ")            // => "1 2 3"
a.join("")             // => "123"
let b = new Array(10); // An array of length 10 with no elements
b.join("-")            // => "---------": a string of 9 hyphens
```

`join()`方法是`String.split()`方法的反向操作，它通过将字符串分割成片段来创建数组。

数组，就像所有 JavaScript 对象一样，都有一个`toString()`方法。对于数组，此方法的工作方式与没有参数的`join()`方法相同：

```js
[1,2,3].toString()          // => "1,2,3"
["a", "b", "c"].toString()  // => "a,b,c"
[1, [2,"c"]].toString()     // => "1,2,c"
```

请注意，输出不包括方括号或任何其他类型的分隔符。

`toLocaleString()`是`toString()`的本地化版本。它通过调用元素的`toLocaleString()`方法将每个数组元素转换为字符串，然后使用特定于区域设置（和实现定义的）分隔符字符串连接生成的字符串。

## 7.8.8 静态数组函数

除了我们已经记录的数组方法之外，Array 类还定义了三个静态函数，您可以通过`Array`构造函数而不是在数组上调用这些函数。`Array.of()`和`Array.from()`是用于创建新数组的工厂方法。它们在§7.1.4 和§7.1.5 中有记录。

另一个静态数组函数是`Array.isArray()`，用于确定未知值是否为数组：

```js
Array.isArray([])     // => true
Array.isArray({})     // => false
```

# 7.9 类似数组对象

正如我们所见，JavaScript 数组具有其他对象没有的一些特殊功能：

+   当向列表添加新元素时，`length`属性会自动更新。

+   将`length`设置为较小的值会截断数组。

+   数组从`Array.prototype`继承了有用的方法。

+   对于数组，`Array.isArray()`返回`true`。

这些是使 JavaScript 数组与常规对象不同的特点。但它们并不是定义数组的基本特征。将任何具有数值`length`属性和相应非负整数属性的对象视为一种数组通常是完全合理的。

这些“类似数组”的对象实际上在实践中偶尔会出现，尽管你不能直接在它们上面调用数组方法或期望`length`属性有特殊行为，但你仍然可以使用与真实数组相同的代码迭代它们。事实证明，许多数组算法与类似数组对象一样有效，就像它们与真实数组一样有效一样。特别是如果你的算法将数组视为只读，或者至少保持数组长度不变时，这一点尤为真实。

以下代码将常规对象转换为类似数组对象，然后遍历生成的伪数组的“元素”：

```js
let a = {};  // Start with a regular empty object

// Add properties to make it "array-like"
let i = 0;
while(i < 10) {
    a[i] = i * i;
    i++;
}
a.length = i;

// Now iterate through it as if it were a real array
let total = 0;
for(let j = 0; j < a.length; j++) {
    total += a[j];
}
```

在客户端 JavaScript 中，许多用于处理 HTML 文档的方法（例如`document.querySelectorAll()`）返回类似数组的对象。以下是您可能用于测试类似数组对象的函数：

```js
// Determine if o is an array-like object.
// Strings and functions have numeric length properties, but are
// excluded by the typeof test. In client-side JavaScript, DOM text
// nodes have a numeric length property, and may need to be excluded
// with an additional o.nodeType !== 3 test.
function isArrayLike(o) {
    if (o &&                            // o is not null, undefined, etc.
        typeof o === "object" &&        // o is an object
        Number.isFinite(o.length) &&    // o.length is a finite number
        o.length >= 0 &&                // o.length is non-negative
        Number.isInteger(o.length) &&   // o.length is an integer
        o.length < 4294967295) {        // o.length < 2³² - 1
        return true;                    // Then o is array-like.
    } else {
        return false;                   // Otherwise it is not.
    }
}
```

我们将在后面的部分看到字符串的行为类似于数组。然而，对于类似数组对象的此类测试通常对字符串返回`false`——最好将其处理为字符串，而不是数组。

大多数 JavaScript 数组方法都故意定义为通用的，以便在应用于类似数组对象时与真实数组一样正确工作。由于类似数组对象不继承自`Array.prototype`，因此不能直接在它们上调用数组方法。但是，您可以间接使用`Function.call`方法调用它们（有关详细信息，请参见§8.7.4）：

```js
let a = {"0": "a", "1": "b", "2": "c", length: 3}; // An array-like object
Array.prototype.join.call(a, "+")                  // => "a+b+c"
Array.prototype.map.call(a, x => x.toUpperCase())  // => ["A","B","C"]
Array.prototype.slice.call(a, 0)   // => ["a","b","c"]: true array copy
Array.from(a)                      // => ["a","b","c"]: easier array copy
```

此代码倒数第二行在类似数组对象上调用 Array `slice()`方法，以将该对象的元素复制到真实数组对象中。这是一种成语技巧，存在于许多传统代码中，但现在使用`Array.from()`更容易实现。

# 7.10 字符串作为数组

JavaScript 字符串表现为 UTF-16 Unicode 字符的只读数组。您可以使用方括号而不是`charAt()`方法访问单个字符：

```js
let s = "test";
s.charAt(0)    // => "t"
s[1]           // => "e"
```

当然，对于字符串，`typeof`运算符仍然返回“string”，如果您将其传递给`Array.isArray()`方法，则返回`false`。

可索引字符串的主要好处仅仅是我们可以用方括号替换`charAt()`的调用，这样更简洁、可读，并且可能更高效。然而，字符串表现得像数组意味着我们可以将通用数组方法应用于它们。例如：

```js
Array.prototype.join.call("JavaScript", " ")  // => "J a v a S c r i p t"
```

请记住，字符串是不可变的值，因此当它们被视为数组时，它们是只读数组。像`push()`、`sort()`、`reverse()`和`splice()`这样的数组方法会就地修改数组，不适用于字符串。然而，尝试使用数组方法修改字符串不会导致错误：它只是悄无声息地失败。

# 7.11 总结

本章深入讨论了 JavaScript 数组，包括稀疏数组和类数组对象的奇特细节。从本章中可以得出的主要观点是：

+   数组字面量是用方括号括起来的逗号分隔的值列表编写的。

+   通过在方括号内指定所需的数组索引来访问单个数组元素。

+   ES6 中引入的`for/of`循环和`...`扩展运算符是迭代数组的特别有用的方式。

+   Array 类定义了一组丰富的方法来操作数组，你应该确保熟悉 Array API。
