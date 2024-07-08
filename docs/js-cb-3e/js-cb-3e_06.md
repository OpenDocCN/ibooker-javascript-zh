# 第五章：数组

自 JavaScript 问世以来，数组一直作为一个独立的数据类型存在。但多年来，我们与数组的交互方式发生了很大变化。

在过去，操作数组涉及大量的循环和迭代逻辑，以及一小组功能有限的方法。如今，`Array`对象提供了更多的功能，包括强调*功能性*方法的方法。使用这些方法，您可以过滤、排序、复制和转换数据，而无需逐个遍历数组元素。

在本章中，您将看到如何使用这些功能性方法，并了解何时可能需要回避它们。重点是使用今天可用的最现代实践来解决问题。

###### 注意

如果您在浏览器的开发者控制台中尝试这些示例，请注意*惰性评估*可能会误导您。例如，考虑以下情况：如果您使用`console.log()`输出一个数组，然后对其进行排序，并再次记录它。您期望看到两个不同排序的数组信息。但实际上，您将看到最终排序的数组两次。这是因为大多数浏览器直到您打开控制台并点击扩展数组时才会检查数组中的项目。避免此问题的一种方法是遍历数组并分别记录每个项目。有关更多信息，请参阅[“为什么 Chrome 的开发者控制台有时会撒谎”](https://oreil.ly/VDHtm)。

# 检查对象是否为数组

## 问题

在执行数组操作之前，您需要验证您的对象确实是一个数组。

## 解决方案

使用静态的`Array.isArray()`方法：

```
const browserNames = ['Firefox', 'Edge', 'Chrome', 'IE', 'Safari'];

if (Array.isArray(browserNames)) {
  // We end up here, because browserNames is a valid array.
}
```

## 讨论

`Array.isArray()`方法是一个明显的选择。问题出现在开发者们试图使用旧的`instanceOf`运算符时。由于历史原因，`instanceOf`运算符在处理数组时有一些奇怪的边缘情况（例如，当您测试在另一个执行上下文中创建的数组时，比如不同的窗口时，它会返回`false`）。`isArray()`方法被添加来填补这个空白。

重要的是要理解，`isArray()`专门检查`Array`对象的实例。如果你在其他类型的集合（如`Map`或`Set`）上调用它，它会返回`false`。即使这些集合具有类似数组的语义，甚至它们的名称中有*array*，比如`TypedArray`（用于二进制数据缓冲区的底层包装器）。

# 遍历数组中的所有元素

## 问题

您希望使用最佳方法来循环遍历数组中的每个元素，按顺序进行。

## 解决方案

传统方法是使用`for`…`of`循环，它会自动获取每个项目：

```
const animals = ['elephant', 'tiger', 'lion', 'zebra', 'cat', 'dog', 'rabbit'];

for (const animal of animals) {
  console.log(animal);
}
```

在现代 JavaScript 中，越来越普遍地倾向于在处理数组代码时采用*函数式*方法。你可以使用`Array.forEach()`方法以函数式的方式迭代你的数组。你提供一个函数，该函数会为数组中的每个元素调用一次，并传递三个潜在有用的参数（元素本身、元素的索引和原始数组）。以下是一个示例：

```
const animals = ['elephant', 'tiger', 'lion', 'zebra', 'cat', 'dog', 'rabbit'];

animals.forEach(function(animal, index, array) {
  console.log(animal);
});
```

使用箭头语法，可以进一步简化此过程（“使用箭头函数”）：

```
animals.forEach(animal => console.log(animal));
```

## 讨论

在像 JavaScript 这样的长寿语言中，通常有许多方法可以完成相同的事情。`for`…`of`循环为遍历数组提供了简单直接的语法。它不允许修改正在遍历的数组中的元素，这是一种安全而明智的方法。

然而，有些情况下你可能需要使用一些不同的方法。其中最灵活的选择之一是使用带有计数器的基本`for`循环：

```
const animals = ['elephant', 'tiger', 'lion', 'zebra', 'cat', 'dog', 'rabbit'];

for (let i = 0; i < animals.length; ++i) {
  console.log(animals[i]);
}
```

这种方法可以使易于出错的一错百错错误未被检测到，这在现代编程中仍然是一个令人惊讶的常见错误来源。然而，在某些情况下，例如同时遍历多个数组时（参见“检查两个数组是否相等”），你需要使用`for`循环。

你还可以通过将函数传递给`Array.forEach()`方法来迭代数组。然后，该函数将为每个元素调用一次。你的函数可以接收三个参数：当前数组元素、当前数组索引以及对原始数组的引用。通常，你只需要元素本身。（你可以使用索引来更改原始数组中的元素，但这被认为是不好的做法。）

如果你想使用函数式方法来更改或检查数组，请考虑使用更具体、有针对性的方法。表 5-1 列出了最有用的方法。

表 5-1\. 函数式数组处理方法

| 任务 | 数组方法 | 所涵盖 |
| --- | --- | --- |
| 更改每个数组元素 | `map()` | “转换数组的每个元素” |
| 查看所有元素是否满足特定条件 | `every()` | “验证数组内容” |
| 查看至少一个元素是否满足特定条件 | `some()` | “验证数组内容” |
| 查找满足条件的数组元素 | `filter()` | “提取满足特定条件的数组项” |
| 重新排序数组 | `sort()` | “按属性值对对象数组进行排序” |
| 在单个计算中使用数组的所有值 | `reduce()` | “将数组的值组合为单个计算” |

现代编程实践更倾向于使用*函数式方法*处理数组，而不是*迭代方法*。函数式方法的优势在于，你的代码可以更简洁，通常更易读，也更少出错。大多数情况下，函数式方法还会为你的数组强制*不可变性*。它通过创建一个包含你想要的变化的数组的新副本来实现这一点，而不是直接修改原始数组对象。这种方法还会减少某些类型的错误发生的可能性。

###### 注意

一般来说，将函数式数组方法作为*首选方法*。如果它们使你的任务变得更加困难（例如需要写多个数组或执行多个数组操作），切换到迭代方法。如果你正在编写性能密集型代码（例如操作极大数组的例程），考虑使用迭代方法，因为它通常执行效果更好。但是不要忘记首先对两种方法进行分析，看看差异是否真的显著。

# 检查两个数组是否相等

## 问题

你希望有一个简单的方法来测试两个数组是否等价（具有完全相同的内容）。

## 解决方案

最直接的方法实际上是老式的方法：使用基本的`for`循环和计数器，同时遍历两个数组，并比较每个元素。当然，在开始循环之前有几个检查要做，比如验证每个对象是否为数组，不为 null 等等。以下是将所有这些标准打包到一个有用函数中的代码片段：

```
function areArraysEqual(arrayA, arrayB) {
  if (!Array.isArray(arrayA) || !Array.isArray(arrayB)) {
    // These objects are null, undeclared, or non-array objects
    return false;
  }
  else if (arrayA === arrayB) {
    // Shortcut: they're two references pointing to the same array
    return true;
  }
  else if (arrayA.length !== arrayB.length) {
    // They can't match if they have a different item count
    return false;
  }
  else {
    // Time to look closer at each item
    for (let i = 0; i < arrayA.length; ++i) {
      // We require items to have the same content and be the same type,
      // but you could use loosely typed equality depending on your task
      if (arrayA[i] !== arrayB[i]) return false;
    }
    return true;
  }
}
```

现在你可以像这样检查两个数组是否相同：

```
const fruitNamesA = ['apple', 'kumquat', 'grapefruit', 'kiwi'];
const fruitNamesB = ['apple', 'kumquat', 'grapefruit', 'kiwi'];
const fruitNamesC = ['avocado', 'squash', 'red pepper', 'cucumber'];

console.log(areArraysEqual(fruitNamesA, fruitNamesB));  // true
console.log(areArraysEqual(fruitNamesA, fruitNamesC));  // false
```

在这个版本的`areArraysEqual()`中，将顺序不同但包含相同项的数组视为不匹配。你可以使用`Array.sort()`方法轻松地对字符串或数字数组进行排序。然而，把这段代码放在`areArrayEquals()`方法中可能并不合适，因为它可能不适用于你想要使用的数据类型，或者在比较大的数组时可能速度太慢。相反，在测试相等之前先对数组进行排序：

```
const fruitNamesA = ['apple', 'kumquat', 'grapefruit', 'kiwi'];
const fruitNamesB = ['kumquat', 'kiwi', 'grapefruit', 'apple'];

console.log(areArraysEqual(fruitNamesA.sort(), fruitNamesB.sort()));  // true
```

## 讨论

在编程中，常常由你自己决定什么是相等的。在这个例子中，`areArraysEqual()`执行的是*浅比较*。如果两个数组具有相同的基本类型或相同的对象引用，并且它们的元素顺序相同，它们就匹配。但是，如果开始比较更复杂的*对象*，就会出现歧义。

例如，考虑这两个包含单个相同`Date`对象的数组的比较：

```
const datesA = [new Date(2021,1,1)];
const datesB = [new Date(2021,1,1)];

console.log(areArraysEqual(datesA, datesB));  // false
```

这些数组不匹配，因为尽管底层的日期内容相同，但`Date` *实例*是不同的。（或者换句话说，有两个单独的`Date`对象，它们恰好保存相同的信息。）

当然，你可以轻松比较两个`Date`对象的内容（只需调用`getTime()`将它们转换为毫秒时间表示，如“比较日期和测试日期是否相等”中所解释的）。但是如果你想在数组比较中这样做，你需要编写一个不同的函数。在你的函数中，你可以使用`instanceOf`来识别`Date`对象，然后在它们上调用`getTime()`：

```
function areArraysEqual(arrayA, arrayB) {
  if (!Array.isArray(arrayA) || !Array.isArray(arrayB)) {
    return false;
  }
  else if (arrayA === arrayB) {
    return true;
  }
  else if (arrayA.length !== arrayB.length) {
    return false;
  }
  else {
    for (let i = 0; i < arrayA.length; ++i) {
      // Check for equal dates
      if (arrayA[i] instanceOf Date && arrayB[i] instanceOf Date) {
        if (arrayA[i].getTime() !== arrayB[i].getTime()) return false;
      }
      else {
        // Use the normal strict equality check
        if (arrayA[i] !== arrayB[i]) return false;
      }
    }
    return true;
  }
}
```

此示例中展示的问题适用于保存任何类型 JavaScript 对象的数组。甚至适用于保存嵌套数组的情况（因为每个`Array`都是一个对象）。然而，由于不同的对象需要不同的等式测试，因此你的解决方案可能会有所不同。

最后值得注意的是，许多流行的 JavaScript 库都有它们自己的通用解决方案来进行深度数组比较，这可能适合也可能不适合你的数据。如果你已经在使用像 Lodash 或 *Underscore.js* 这样的库，请调查它们的`isEqual()`方法。

# 将数组拆分为单独变量

## 问题

你需要将数组元素的值分配给多个变量，但是你希望有一种方便的方法，不需要分别为每个变量分配值。

## 解决方案

使用数组的*解构语法*一次性赋值多个变量。你可以编写一个表达式，在左侧声明多个变量，并从数组中获取值（在右侧）。下面是一个示例：

```
const stateValues = [459, 144, 96, 34, 0, 14];
const [arizona, missouri, idaho, nebraska, texas, minnesota] = stateValues;
console.log(missouri);   // 144
```

当你使用数组解构时，值是按位置复制的。在这个例子中，这意味着`arizona`获取数组中的第一个值，`missouri`获取第二个值，依此类推。如果你有多个变量而数组元素不足，额外的变量将获得`undefined`的值。

## 讨论

当你使用数组解构时，你不需要复制数组中的每个值。通过在没有变量名的情况下添加额外的逗号，你可以跳过不想要的值：

```
const stateValues = [459, 144, 96, 34, 0, 14];

// Just get three values from the array
const [arizona, , , nebraska, texas] = stateValues;
console.log(nebraska);   // 34
```

你还可以使用*剩余操作符*将所有剩余的值（那些你没有显式分配给变量的值）装入一个名为`others`的新数组中。下面是一个示例，将最后三个数组元素复制到数组`others`中：

```
const stateValues = [459, 144, 96, 34, 0, 14];
const [arizona, missouri, idaho, ...others] = stateValues;
console.log(others);   // 34, 0, 14
```

###### 注意

JavaScript 的剩余操作符看起来就像扩展操作符（它是变量前的三个点）。它们在你的代码中甚至“感觉”相似，尽管它们实际上扮演互补的角色。剩余操作符可以收集额外的值并将它们压缩成单个数组。扩展操作符则将数组（或其他类型的可迭代对象）*展开*为单独的值。

到目前为止，你已经看到了变量声明和赋值在一条语句中的情况，但是你可以像创建普通变量一样将它们分开。只需确保保留方括号，因为它们表示你正在使用数组解构：

```
let arizona, missouri, idaho, nebraska, texas, minnesota;
[arizona, missouri, idaho, nebraska, texas, minnesota] = stateValues;
```

## 参见

如果你想要一种将数组转换为值列表*而不*将这些值分配给变量的方法，请查看描述在“将数组传递给期望值列表的函数”中的扩展操作符。

# 将数组传递给期望值列表的函数

## 问题

你的数组有一个值列表，你想将其传递给一个函数。但是函数期望的是一组参数值，而不是一个数组对象。

## 解决方案

使用扩展运算符来展开你的数组。这里有一个使用`Math.max()`方法的示例：

```
const numbers = [2, 42, 5, 304, 1, 13];

// This syntax is not allowed. The result is NaN.
const maximumFail = Math.max(numbers);

// But this works, thanks to the spread operator. (The answer is 304.)
const maximum = Math.max(...numbers);
```

## 讨论

扩展运算符将一个数组展开为元素列表。从技术上讲，它适用于任何可迭代对象，包括其他类型的集合。你将在本章的几个示例中看到它的应用。

扩展运算符不需要为函数提供所有参数，甚至最终参数。像这样使用是完全有效的：

```
const numbers = [2, 42, 5, 304, 1, 13];

// Call max() on the array values, along with three more arguments.
const maximum = Math.max(24, ...numbers, 96, 7);
```

如果你的参数顺序有任何意义，你可能不希望使用这种方法。很容易最终得到一个比预期稍大或稍小的数组，这会导致你的其他参数被移到新位置并改变其意义。

## 参见

“合并两个数组”展示了如何使用扩展运算符合并不同的数组的示例。“删除或替换数组元素”展示了在移除项目时如何使用扩展运算符。“克隆数组”展示了如何使用扩展运算符复制数组。

# 克隆数组

## 问题

你想复制一个现有的数组。

## 解决方案

使用扩展运算符将你的数组展开为项目，并将其馈送到一个新数组中：

```
const numbers = [2, 42, 5, 304, 1, 13];
const numbersCopy = [...numbers];
```

同样好的方法是使用`Array.slice()`方法，不带参数，告诉它取整个数组的一个切片：

```
const numbers = [2, 42, 5, 304, 1, 13];
const numbersCopy = numbers.slice();
```

这两种方法都比手动遍历数组元素并逐个构建新数组更可取。

## 讨论

创建数组副本很重要，因为它允许你执行*非破坏性更改*。例如，你可以保持原始数组不变，同时对新副本进行更改。这样，你可以减少意外副作用的风险（例如，如果代码的其他部分仍在使用原始数组）。

如同所有引用对象一样，数组不能通过赋值来复制。例如，下面的代码结束时，两个变量指向同一个内存中的`Array`对象：

```
const numbers = [2, 42, 5, 304, 1, 13];
const numbersCopy = numbers;
```

要正确复制一个数组，你需要复制它的所有元素。最简单的方法是使用扩展运算符，尽管`Array.slice()`方法同样有效。

这里展示的两种方法都创建了*浅拷贝*。如果你的数组由基本类型（数字、字符串或布尔值）组成，复制后的数组完全匹配。但是如果你的数组包含对象，这些技术会复制*引用*，而不是整个对象。因此，你的新数组将有指向相同对象的引用。改变复制数组中的一个对象也会影响原始数组：

```
const objectsOriginal = [{name: 'Sadie', age: 12}, {name: 'Patrick', age: 18}];
const objectsCopy = [...objectsOriginal];

// Change one of the people objects in objectsCopy
objectsCopy[0].age = 14;

// Investigate the same object in objectsOriginal
console.log(objectsOriginal[0].age);  // 14
```

这可能是一个问题，也可能不是，这取决于你计划如何使用你的数组。如果你想要多个可单独操作的对象副本，有几种可能的解决方案可以使用：

+   使用`for`循环遍历数组，显式创建所需的新对象，然后将它们添加到新数组中。

+   使用`Array.map()`函数。这对于简单对象很有效，但并不完全深度复制所有内容。（例如，如果你有对象引用*其他*对象，只有对象的第一层真正被复制。）

+   使用来自其他 JavaScript 库的辅助函数，如 Lodash 中的`cloneDeep()`或 Ramda 中的`clone()`。

这里有一个示例演示了`Array.map()`的使用。它通过首先使用展开操作符（…`element`）将数组元素扩展为其属性，然后使用这些属性创建一个新对象（`{`…`element}`），并将其分配给新数组：

```
const objectsOriginal = [{name: 'Sadie', age: 12}, {name: 'Patrick', age: 18}];

// Create a new array with copied objects
const objectsCopy = objectsOriginal.map( element => ({...element}) );

// Change one of the people objects in objectsCopy
objectsCopy[0].age = 14;

// Investigate the same object in objectsOriginal
console.log(objectsOriginal[0].age);  // 12
```

要深入了解`map()`方法，请参阅“转换数组的每个元素”中的完整解释。

###### 注意

展开操作符（`...`）有双重作用。在原始解决方案中，你看到展开操作符如何将数组展开为单独的元素。在`Array.map()`示例中，展开操作符将一个*对象*展开为单独的属性。有关展开操作符在对象上的工作原理的更多信息，请参阅“合并两个对象的属性”。

## 参见

如果你只想复制*某些*数组项，请参阅“按位置复制数组的部分项”。要了解有关不同方式的深层复制对象的更多信息，请参阅“深层复制对象”。

# 合并两个数组

## 问题

想要将两个完整的数组合并成一个新数组。

## 解决方案

有两种常用的方法可以组合两个数组。传统的方法（可能也是性能最佳的选项）是使用`Array.concat()`方法。在第一个数组上调用`concat()`，将第二个数组作为参数传入。结果是一个包含两者所有元素的第三个数组：

```
const evens = [2, 4, 6, 8];
const odds = [1, 3, 5, 7, 9];

const evensAndOdds = evens.concat(odds);
// now evensAddOdds contains [2, 4, 6, 8, 1, 3, 5, 7, 9]
```

结果数组的项首先是第一个数组的项（在此示例中是偶数），然后是第二个数组的项（奇数）。当然，你可以在`concat()`之后调用`Array.sort()`方法（“按属性值对对象数组进行排序”）。

另一种方法是使用展开操作符（引入自“将数组传递给期望值列表的函数”）：

```
const evens = [2, 4, 6, 8];
const odds = [1, 3, 5, 7, 9];

const evensAndOdds = [...evens, ...odds];
```

这种方法的优点在于代码（可以说）更直观和更易读。展开操作符也是一个很棒的工具，如果你想一次性组合多个数组，或者将数组与字面值组合：

```
const evens = [2, 4, 6, 8];
const odds = [1, 3, 5, 7, 9];

const evensAndOdds = [...evens, 10, 12, ...odds, 11];
```

性能测试表明，在当前实现中，使用 `concat()` 更快地合并大数组。但在大多数情况下，这种性能差异不会显著（甚至不会明显）。

## 讨论

在您使用任一种合并数组的技术后，您会得到三个数组：原始两个数组和新合并的结果数组。如果您的数组包含基本值（数字、字符串、布尔值），这些值在新数组中会被复制。但如果您的数组包含对象，则会复制对象的*引用*。例如，如果您合并两个包含 `Date` 对象的数组，则不会创建新的 `Date` 对象。相反，新合并的数组会得到指向*相同* `Date` 对象的引用。如果您在合并后的数组中更改了 `Date` 对象，您将在原始数组中看到相同的修改：

```
const dates2020 = [new Date(2020,1,10), new Date(2020,2,10)];
const dates2021 = [new Date(2021,1,10), new Date(2021,2,10)];

const datesCombined = [...dates2020, ...dates2021];

// Change a date in the new array
datesCombined[0].setYear(2022);

// The same object is in the first array
console.log(dates2020[0]);   // 2022/02/10
```

欲了解浅复制和深复制之间的区别，请参阅 “深复制对象”。

## 另请参阅

当您合并数组时，您无法控制元素的组合方式。如果您想要复制数组的一部分，或者将一个数组放在另一个数组的*中间*，请参阅 “按位置复制数组的一部分” 中的 `slice()` 方法。

# 按位置复制数组的一部分

## 问题

您想要复制数组的一部分，并保持原始数组不变。

## 解决方案

使用 `Array.slice()` 方法，它可以*浅复制*现有数组的一部分，并将其作为新数组返回：

```
const animals = ['elephant', 'tiger', 'lion', 'zebra', 'cat', 'dog',
 'rabbit', 'goose'];

// Get the chunk from index 4 to index 7.
const domestic = animals.slice(4, 7);

console.log(domestic); // ['cat', 'dog', 'rabbit']
```

## 讨论

`slice()` 方法接受两个参数，表示起始和结束位置。您可以省略第二个参数以从起始索引到数组末尾。对数组调用 `slice(0)` 将复制整个数组。

例如，以下代码使用 `slice` 获取第一个数组的两个子部分，并使用它们构建一个新数组：

```
const animals = ['elephant', 'tiger', 'lion', 'zebra', 'cat', 'dog',
 'rabbit', 'goose'];

const firstHalf = animals.slice(0, 3);
const secondHalf = animals.slice(4, 7);

// Put two new animals in the middle
const extraAnimals = [...firstHalf, 'emu', 'platypus', ...secondHalf];
```

这可能看起来像是一个随意的示例，因为索引数字是硬编码的。但是您可以将其与数组搜索和 `findIndex()` 方法结合使用（参见 “在数组中搜索精确匹配项”）来确定您应该分割数组的位置。

###### 注意

`slice()` 方法很容易与 `splice()` 方法混淆，后者用于替换或删除数组的部分内容。与 `slice()` 不同，`splice()` 方法会对原始数组进行影响的就地修改。在现代实践中，最好将对象锁定，尽可能保持其不可变性（因此使用 `const`），并通过创建带有更改的新副本来实现。因此，除非您有强烈的理由要使用 `splice()`（例如，在您的使用情况下性能存在显著差异），否则请坚持使用 `slice()`。

## 另请参阅

“删除或替换数组元素” 展示了如何使用 `slice()` 删除数组的部分内容。

# 提取符合特定条件的数组项

## 问题

你想要找出数组中所有满足某个条件的项，并将它们复制到一个新数组中。

## 解决方案

使用 `Array.filter()` 方法在每个项上运行一个测试：

```
function startsWithE(animal) {
  return animal[0].toLowerCase() === 'e';
}

const animals = ['elephant', 'tiger', 'emu', 'zebra', 'cat', 'dog',
 'eel', 'rabbit', 'goose', 'earwig'];
const animalsE = animals.filter(startsWithE);
console.log(animalsE);   // ["elephant", "emu", "eel", "earwig"]
```

这个例子有意冗长，以便你能看到解决方案的不同部分。*过滤函数* 对数组中的每个项进行调用。在这种情况下，意味着 `startsWithE()` 会被调用 10 次，并传递不同的字符串。如果过滤函数返回 `true`，那么该项将被添加到新数组中。

这是使用箭头函数压缩后的相同示例。现在过滤逻辑在代码中的同一位置定义，你可以直接使用它：

```
const animals = ['elephant', 'tiger', 'emu', 'zebra', 'cat', 'dog',
 'eel', 'rabbit', 'goose', 'earwig'];
const animalsE = animals.filter(animal => animal[0].toLowerCase() === 'e');
```

## 讨论

在这个例子中，过滤函数检查每个项是否以字母 *e* 开头。但是你也可以轻松地获取落在某个范围内的数字，或者具有特定属性值的对象。

`filter()` 方法是一组现代数组方法中的一员，它取代了老式的迭代代码，采用了功能化方法。你可以使用 `for` 循环遍历数组，测试每个项，并使用 `Array.push()` 将匹配项插入新数组中。但是，如果能用 `filter()` 方法完成同样的任务，通常可以得到更紧凑的代码和更简便的测试。

## 参见

本章中的几个示例介绍了类似的功能化数组处理方法。特别是，“数组每个元素的转换”（#mapping_array）展示了如何转换数组中的所有元素，“将数组值组合成单一计算结果”（#reducing_array）展示了如何将数组中所有值组合成一个结果。

# 清空数组

## 问题

你需要从数组中删除所有元素，以释放内存或使数组可以重用。

## 解决方案

将数组的 `length` 属性设置为 0：

```
const numbers = [2, 42, 5, 304, 1, 13];
numbers.length = 0;
```

## 讨论

给自己创建一个新数组的最简单方法之一是简单地分配一个新的空数组，就像这样：

```
myArray = [];
```

然而，这种方法有一些限制。首先，因为它创建了一个全新的数组对象，所以如果你使用 `const` 关键字定义了数组，它就不起作用。这是一个小细节，但是现代实践倾向于使用 `const` 而不是 `let`，以减少代码中的错误可能性。其次，这种赋值并没有真正销毁数组。如果你有另一个变量指向你的数组，它将继续存在并驻留在内存中。

另一种解决方案是反复调用`Array.pop()`方法。每次调用`pop()`时，都会从数组中移除最后一项，因此你可以通过一个循环调用`pop()`直到数组为空来清空数组。然而，`length`设置技巧具有完全相同的效果，并且只需要一个语句。开发人员有时会忽视这种技术，因为他们期望`length`是只读属性（在许多其他语言中确实如此）。但是在 JavaScript 数组上设置`length`允许你缩小其大小并丢弃剩余的项。

还有其他有趣的方法可以使用`length`属性。例如，你可以通过减少`length`来仅截取数组的一部分，但不全为 0。或者，你可以通过增加`length`向数组末尾添加空白项：

```
const numbers = [2, 42, 5, 304, 1, 13];
numbers.length = 3;

console.log(numbers);  // [2, 42, 5]

numbers.length = 5;
console.log(numbers);  // [2, 42, 5, undefined, undefined]
```

# 移除重复值

## 问题

你希望确保数组中的每个值都是唯一的，通过删除重复项。

## 解决方案

创建一个新的`Set`对象并用你的数组填充它。`Set`对象会自动丢弃重复项。然后，将`Set`对象转换回数组：

```
const numbersWithDuplicates = [2, 42, 5, 42, 304, 1, 13, 2, 13];

// Create a Set with unique values (the duplicate 42, 2, and 13 are discarded)
const uniqueNumbersSet = new Set(numbersWithDuplicates);

// Turn the Set back into an array (now with 6 items)
const uniqueNumbersArray = Array.from(uniqueNumbersSet);
```

一旦理解了这个想法，你可以用展开运算符将其压缩为一个单一的语句：

```
const numbersWithDuplicates = [2, 42, 5, 42, 304, 1, 13, 2, 13];

const uniqueNumbers = [...new Set(numbersWithDuplicates)];
```

## 讨论

`Set`对象是一种特殊类型的集合，它忽略重复值。它还可以作为一种快速高效地从数组中删除重复项的方法。这种技术（切换到`Set`，然后再转回数组）比遍历数组并使用`findIndex()`查找重复项要高效得多。

在搜索重复项时，`Set`使用类似于严格相等比较`===`的测试，这意味着 3 和`'3'`不被视为重复项。`Set`实现的一个特殊行为是，它将重复的`NaN`值视为重复项，即使`NaN === NaN`通常计算结果为`false`。

## 参见

本示例使用了在“将数组传递给期望值列表的函数”中描述的展开运算符。关于`Set`对象的更多信息，请参见“创建一个无重复值的集合”。

# 展平二维数组

## 问题

你想要将二维数组展平，使其成为一维列表。

## 解决方案

使用`Array.flat()`方法：

```
const fruitArray = [];

// Add three elements to fruitArray
// Each element is an array of strings
fruitArray[0] = ['strawberry', 'blueberry', 'raspberry'];
fruitArray[1] = ['lime', 'lemon', 'orange', 'grapefruit'];
fruitArray[2] = ['tangerine', 'apricot', 'peach', 'plum'];

const fruitList = fruitArray.flat();
// Now fruitList has 11 elements, and each one is a string
```

## 讨论

考虑一个二维数组，例如：

```
const fruitArray = [];
fruitArray[0] = ['strawberry', 'blueberry', 'raspberry'];
fruitArray[1] = ['lime', 'lemon', 'orange', 'grapefruit'];
fruitArray[2] = ['tangerine', 'apricot', 'peach', 'plum'];
```

`fruitArray`中的每个元素都包含*另一个*数组。例如，`fruitArray[0]`有三个字符串，代表不同的浆果。`fruitArray[1]`有柑橘类水果，`fruitArray[2]`有核果类水果。

你可以利用`concat()`方法转换`fruitArray`。从第一个嵌套数组开始，调用`concat()`，然后传递其他嵌套数组，就像这样：

```
const fruitList =
 fruitArray[0].concat(fruitArray[1],fruitArray[2],fruitArray[3]);
```

如果数组有多个成员，这种方法很繁琐且容易出错。或者，你可以使用循环或递归，但这些方法同样很繁琐。`flat()`方法实现了相同的逻辑，并为你拼接了每一行。

`flat()` 方法接受一个可选的 `depth` 参数，默认值为 1。你可以增加这个数字以深度展平嵌套更深的数组。例如，假设你有一个包含嵌套数组的数组，而这些数组又包含*另一层*嵌套数组。在这种情况下，`depth` 为 2 将连接两个层，将所有内容放入单个列表中：

```
// An array with several levels of nested arrays inside
const threeDimensionalNumbers = [1, [2, [3, 4, 5], 6], 7];

// The default flattening
const flat2D = threeDimensionalNumbers.flat(1);
// now flat2D = [1, 2, [3, 4, 5], 6, 7]

// Flatten two levels
const flat1D = threeDimensionalNumbers.flat(2);
// now flat1D = [1, 2, 3, 4, 5, 6, 7]

// Flatten all levels, no matter how many there are
const flattest = threeDimensionalNumbers.flat(Infinity);
```

`depth` 参数设置所需的最大展开级别。增加 `depth` 超出数组的实际维度不会有风险。

# 搜索数组以查找完全匹配项

## 问题

你想要在数组中搜索特定值。你可能想知道数组是否包含匹配项，或者匹配发生的位置。

## 解决方案

使用以下数组搜索方法之一：`indexOf()`、`lastIndexOf()` 或 `includes()`：

```
const animals = ['dog', 'cat', 'seal', 'elephant', 'walrus', 'lion'];
console.log(animals.indexOf('elephant'));    // 3
console.log(animals.lastIndexOf('walrus'));  // 4
console.log(animals.includes('dog'));        // true
```

此技术仅适用于基本值（通常是数字、字符串和布尔值）。如果要搜索对象，则需要改用 `Array.find()` 方法（“搜索数组以满足特定条件的项”）。

## 讨论

`indexOf()` 和 `lastIndexOf()` 都接受一个搜索值，然后将其与数组中的每个元素进行比较。如果找到该值，则返回数组元素的索引位置。如果未找到该值，则返回*-1*。

`indexOf()` 方法从最低索引开始搜索并返回找到的第一个匹配项（换句话说，从数组的开始向前搜索）。`lastIndexOf()` 方法则相反，从数组的末尾开始搜索。如果相同项在数组中出现多次，则会出现差异：

```
const animals = ['dog', 'cat', 'seal', 'walrus', 'lion', 'cat'];

console.log(animals.indexOf('cat'));      // 1
console.log(animals.lastIndexOf('cat'));  // 5
```

`indexOf()` 和 `lastIndexOf()` 都接受一个可选的起始索引参数。它设置搜索将从该位置开始：

```
const animals = ['dog', 'cat', 'seal', 'walrus', 'lion', 'cat'];

console.log(animals.indexOf('cat', 2));      // 5
console.log(animals.lastIndexOf('cat', 4));  // 1
```

你可能会想到可以使用循环通过 `indexOf()` 逐步增加索引，直到找到所有匹配项。但在编写这种样板代码之前，请考虑使用 `filter()` 方法，该方法可以根据指定的条件快速轻松地创建包含所有匹配项的数组（参见“按特定条件提取数组项”）。

最后，重要的是要理解 `indexOf()`、`lastIndexOf()` 和 `includes()` 都使用 `===` 运算符来测试匹配。这意味着不会执行类型转换（因此 `3` 不等于 `'3'`）。此外，如果数组包含对象，则比较的是引用而不是内容。如果需要更改相等性的含义或者想使用不同的搜索测试，请改用 `findIndex()` 方法（参见“搜索数组以满足特定条件的项”）。

## 参见

对于可定制的搜索，请参阅 “搜索数组以满足特定条件的项” 中的 `find()` 和 `findIndex()` 方法。

# 在数组中搜索符合特定标准的项目

## 问题

你想在数组中搜索符合某些标准的项。例如，也许你在寻找具有特定属性的对象。

## 解决方案

使用一种函数式数组搜索方法：`find()`或`findIndex()`。无论哪种方式，你都需要提供一个测试每个项目的函数，直到找到匹配项。

这里有一个例子，找出第一个大于 10 的数字：

```
const nums = [2, 4, 19, 15, 183, 6, 7, 1, 1];

// Find the first value over 10.
const bigNum = nums.find(element => element > 10);

console.log(bigNum);  // 19 (the first match)
```

如果你更希望知道匹配元素的位置，而不是找到它，你可以使用类似的`findIndex()`方法：

```
const nums = [2, 4, 19, 15, 183, 6, 7, 1, 1];

const bigNumIndex = nums.findIndex(element => element > 100);

console.log(bigNumIndex);  // 4 (the index of the first match)
```

如果没有找到匹配项，`find()`返回`undefined`，而`findIndex()`返回-1。

## 讨论

当使用`find()`和`findIndex()`时，你需要提供一个回调函数，该函数最多接收三个参数（迭代中的当前数组元素、其索引和数组本身）。箭头语法提供了一种更简洁的方法，允许你在使用时直接定义回调函数。

当你需要编写更复杂的条件时，`find()`和`findIndex()`方法真正发挥了作用。考虑以下代码，它找到特定年份中的第一个日期：

```
// Remember, the Date constructor takes a zero-based month number, so a
// month value of 10 corresponds to the eleventh month, November
const dates = [new Date(2021, 10, 20), new Date(2020, 3, 12),
 new Date(2020, 5, 23), new Date(2022, 3, 18)];

// Find the first date in 2020
const matchingDate = dates.find(date => date.getFullYear() === 2020);

console.log(matchingDate);  // 'Sun Apr 12 2020 ...'
```

使用`indexOf()`方法是不可能的，因为它涉及检查数组项的*属性*。（事实上，标准的`indexOf()`方法甚至不能测试`Date`对象是否相等，因为它只检查对象引用是否匹配。）

## 另见

如果你想编写一个查找函数并用它获取多个结果，你可能需要使用在“按特定标准提取数组项”中描述的`filter()`函数。有关箭头函数语法的更多信息，请参见“使用箭头函数”。

# 移除或替换数组元素

## 问题

你想在数组中查找给定值的出现次数，并且要么移除该元素，要么替换它。

## 解决方案

首先，使用`indexOf()`找到你想要移除的项的位置。然后，你可以使用两种方法中的任何一种。

对于小任务，最干净的解决方案是构建一个新的数组，*围绕*你不想要的项。你使用`slice()`和扩展运算符构建新数组：

```
const animals = ['dog', 'cat', 'seal', 'walrus', 'lion', 'cat'];

// Find where the 'walrus' item is
const walrusIndex = animals.indexOf('walrus');

// Join the portion before 'walrus' to the portion after 'walrus'
const animalsSliced =
 [...animals.slice(0, walrusIndex), ...animals.slice(walrusIndex+1)];

// now animalsSliced has ['dog', 'cat', 'seal', 'lion', 'cat']
```

## 讨论

另一种方法是进行就地数组编辑，而不是创建一个更改的副本。这对于大型数组可能会表现更好。然而，你允许的可变性越多，你的代码就会变得越复杂，这可能会使将来更难管理和调试。

为了进行就地编辑，你使用同名但非常不同的`splice()`方法。它允许你从任何位置开始移除你想要的任意数量的项：

```
const animals = ['dog', 'cat', 'seal', 'walrus', 'lion', 'cat'];

// Find where the 'walrus' item is
const walrusIndex = animals.indexOf('walrus');

// Starting at walrusIndex, remove 1 element
animals.splice(walrusIndex, 1);

// now animals = ['dog', 'cat', 'seal', 'lion', 'cat']
```

`splice()`方法的第一个参数是切片开始的位置。这是你需要提供的唯一参数。如果省略其他参数，从索引到末尾的所有数组元素都会被移除：

```
const animals = ['cat', 'walrus', 'lion', 'cat'];

// Start at 'lion', and remove the rest of the elements
animals.splice(2);
// now animals = ['cat', 'walrus']
```

可选的第二个参数是要删除的元素数。第三个参数是要在相同位置*插入*的可选替换元素集。

```
const animals = ['cat', 'walrus', 'lion', 'cat'];

// Remove one element and add two new elements
animals.splice(2, 1, 'zebra', 'elephant');
// now animals = ['cat', 'walrus', 'zebra', 'elephant', 'cat']
```

您可以在循环中使用 `indexOf()` 来查找并删除一系列匹配元素。但是，如果这是您的目标，通常使用 `filter()` 方法会更简洁，它允许您定义一个选择要保留的项目的函数（参见 “按特定条件提取数组项”）。

# 根据属性值对对象数组进行排序

## 问题

您希望根据其属性对包含对象的数组进行排序。

## 解决方案

`Array.sort()` 方法重新排序数组。例如，它可以将数字数组从最小到最大排列，或者将字符串数组按字母顺序排列。但是你不必局限于数组的标准排序系统。相反，你可以将一个比较函数传递给 `sort()` 方法，数组将使用它来排序其项。

比较函数获取两个项（对应于两个不同的数组元素），比较它们，并返回一个指示结果的数字。如果值应被视为相等，则返回 *0*，如果第一个值小于第二个值，则返回 *–1*，如果第一个值大于第二个值，则返回 *1*。

下面是对带有人员信息的对象数组进行排序的简单实现：

```
const people  = [
 { firstName: 'Joe', lastName: 'Khan', age: 21 },
 { firstName: 'Dorian', lastName: 'Khan', age: 15 },
 { firstName: 'Tammy', lastName: 'Smith', age: 41 },
 { firstName: 'Noor', lastName: 'Biles', age: 33 },
 { firstName: 'Sumatva', lastName: 'Chen', age: 19 }
];

// Sort the people from youngest to oldest
people.sort( function(a, b) {
  if (a.age < b.age) {
    return -1;
  } else if (a.age > b.age) {
    return 1;
  } else {
    return 0;
  }
});
console.log(people);
// Now the order is Dorian, Sumatva, Joe, Noor, Tammy
```

这里可以有几个快捷方式。从技术上讲，您可以返回任何负数而不是 -1，并返回任何正数而不是 1。这允许您编写一个更短的比较函数：

```
people.sort(function(a, b) {
  // Subtract the ages to sort from youngest to oldest
  return a.age - b.age;
});
```

结合紧凑的箭头语法，它变得更短：

```
people.sort((a,b) => a.age - b.age);
```

有时，在进行排序时，您可以利用现有的比较方法。例如，如果您希望此示例按姓氏排序，无需重新发明轮子。相反，充分利用 `String.localeCompare()` 方法，如下所示：

```
people.sort((a,b) => a.lastName.localeCompare(b.lastName));
console.log(people);
// Now the order is Noor, Sumatva, Joe, Dorian, Tammy
```

## 讨论

`sort()` 方法会*直接修改*您的数组。这与您将使用的大多数其他数组方法不同，后者返回更改后的副本，但保留原始数组不变。如果这不是您想要的行为，您可以在排序之前克隆数组，如 “克隆数组” 中详细说明的那样。

# 转换数组的每个元素

## 问题

您希望使用相同的转换将数组中的每个元素转换，并使用更改后的值构建一个新数组。

## 解决方案

使用 `Array.map()` 方法，并提供执行更改的函数。`map()` 方法遍历整个数组，将您的函数应用于每个元素，并使用返回值构建一个新数组。

这里有一个示例，使用这种方法将十进制数字数组转换为具有其十六进制等效项的新数组（使用 “将十进制转换为十六进制值” 中描述的转换技术）：

```
const decArray = [23, 255, 122, 5, 16, 99];

// Use the toString() method to conver to base-16 values
const hexArray = decArray.map( element => element.toString(16) );

console.log(hexArray);  // ['17', 'ff', '7a', '5', '10', '63']
```

## 讨论

通常，`map()`函数只关注数组元素。然而，你的回调函数可以接受两个额外参数：索引和原始数组。使用这些细节，从技术上讲，可以使用`map()`来更改你的*原始*数组。这被认为是一种反模式。换句话说，如果你不打算使用`map()`返回的新数组，就不应该使用`map()`方法。考虑使用`forEach()`方法代替（“遍历数组中的所有元素”），或者只是按程序顺序迭代你的数组。

# 将数组的值合并为单个计算

## 问题

你想要在某种聚合计算中使用数组中的所有值，比如计算总和或平均值。

## 解决方案

你可以在循环中迭代数组。但为了更简洁的解决方案，可以使用带有回调函数的`Array.reduce()`方法。你的函数（称为*reducer 函数*）会对数组中的每个元素进行调用。你使用*累加器*构建某种运行总和，这是`reduce()`方法在过程结束前维护的值。

例如，想象一下，你想要计算一个数字数组的总和。每次调用你的 reducer 函数时，它会在累加器中获取当前的运行总和。然后将当前元素的值加上并返回新的总和：

```
const reducerFunction = function (accumulator, element) {
  // Add the current value to the running total in the accumulator.
  const newTotal = accumulator + element;
  return newTotal;
}
```

当 reducer 为*下一个*项目调用时，这个新的总和就会成为累加器。

现在你可以使用这个函数来对数组求和：

```
const numbers = [23, 255, 122, 5, 16, 99];

// The second argument (0) sets the starting value of the accumulator.
// If you don't set a starting value, the accumulator is automatically set
// to the first element.
const total = numbers.reduce(reducerFunction, 0);
console.log(total);  // 520
```

当 reducer 函数在最后一个项目上调用时，它进行最终计算。该返回值成为从`reduce()`返回的结果。

一旦你熟悉了`reduce()`的工作方式，你可以使用内联函数和箭头语法使你的代码更短更简洁。以下是一个演示，使用`reduce()`来计算平方值的总和、平均值和最大值：

```
const numbers = [23, 255, 122, 5, 16, 99];

// The reducer function adds to the accumulator
const totalSquares = numbers.reduce( (acc, val) => acc + val**2, 0);
// totalSquares = 90520

// The reducer function adds to the accumulator
const average = numbers.reduce( (acc, val) => acc + val, 0) / numbers.length;
// average = 86.66...

// The reducer function returns the higher value (accumulator or current value)
const max = numbers.reduce( (acc, val) => acc > val ? acc: val);
// max = 255
```

## 讨论

使用`reduce()`方法可能比其他函数式数组处理方法更复杂，比如`map()`（“转换数组的每个元素”）、`filter()`（“提取符合特定条件的数组项”）或`sort()`（“按属性值对对象数组进行排序”）。不同之处在于，你需要仔细考虑每个函数调用后需要存储的数据。记住，你可以使用累加器来存储一个具有多个属性的自定义对象，从而跟踪所需的所有信息。你还可以向 reducer 函数添加两个可选参数：`index`（元素的当前索引号）和`array`（正在被 reduce 的整个数组）。但要小心。过于热衷于使用`reduce()`的代码可能会很快变得难以理解。

## 参见

还有另一种方法可以从数字数组中获取最大值。您可以使用`Math.max()`方法与展开运算符结合使用，将数组转换为参数列表（参见“将数组传递给期望值列表的函数”）。

# 验证数组内容

## 问题

您希望确保数组内容满足特定的条件。

## 解决方案

使用`Array.every()`方法来检查每个元素是否通过了给定的测试。例如，以下代码使用正则表达式来确保数组中每个元素都由字母组成：

```
// The testing function
function containsLettersOnly(element) {
  const textExp = /^[a-zA-Z]+$/;
  return textExp.test(element);
}

// Test an array
const mysteryItems = ['**', 123, 'aaa', 'abc', '-', 46, 'AAA'];
let result = mysteryItems.every(containsLettersOnly);
console.log(result);  // false

// Test another array
const mysteryItems2 = ['elephant', 'lion', 'cat', 'dog'];
result = mysteryItems2.every(containsLettersOnly);
console.log(result);  // true
```

或者，使用`Array.some()`方法来确保至少有一个元素通过了测试。例如，以下代码检查数组中至少有一个元素是字母字符串：

```
const mysteryItems = new Array('**', 123, 'aaa', 'abc', '-', 46, 'AAA');

// testing function
function testValue (element) {
   const textExp = /^[a-zA-Z]+$/;
   return textExp.test(element);
}

// run test
const result = mysteryItems.some(testValue);
console.log(result);  // true
```

## 讨论

不同于许多其他使用回调函数的数组方法，`every()`和`some()`方法不会处理所有数组元素。相反，它们只会处理足够多的数组元素以满足其功能需求。

解决方案展示了相同的回调函数可以同时用于`every()`和`some()`方法。区别在于，使用`every()`时，一旦函数返回`false`值，处理就结束，并且方法返回`false`。`some()`方法会继续对每个数组元素进行测试，直到回调函数返回`true`。此时，不再验证其他元素，并且方法返回`true`。但是，如果回调函数对所有元素进行了测试，并且没有为任何元素返回`true`，`some()`将返回`false`。

## 参见

要查看此示例中用于字符串匹配模式的正则表达式语法，请参阅“使用正则表达式替换字符串中的模式”。

# 创建一个非重复值的集合

## 问题

您希望创建一个类似数组的对象，它从不包含超过一次的相同值。

## 解决方案

创建一个`Set`对象。它会静默地忽略尝试多次添加相同项的操作，而不会生成错误。

`Set`不是数组，但像数组一样，它是一个可迭代的元素集合。您可以使用`add()`方法逐个添加元素到`Set`中，或者您可以将数组传递给`Set`构造函数一次性添加多个项：

```
// Start with six elements
const animals = new Set(['elephant', 'tiger', 'lion', 'zebra', 'cat', 'dog']);

// Add two more
animals.add('rabbit');
animals.add('goose');

// Nothing happens, because this item is already in the Set
animals.add('tiger');

// Iterate over the Set, just as you would with an array
for (const animal of animals) {
    console.log(animal);
}
```

## 讨论

`Set`对象不是数组。与`Array`类不同，后者提供了三十多个有用的方法，`Set`类提供的功能要少得多。您可以使用`add()`插入项目，`delete()`删除项目，`has()`检查项目是否在`Set`中，以及`clear()`一次性删除所有项目。没有用于排序、过滤、转换或复制的方法。

然而，如果需要像处理数组一样处理您的`Set`对象，通过将`Set`传递给静态的`Array.from()`方法进行转换非常容易：

```
// Convert an array to a Set
const animalSet = new Set(['elephant', 'tiger', 'zebra', 'cat', 'dog']);

// Convert a Set to an array
const animalArray = Array.from(animalSet);
```

实际上，你可以随意将`Set`转换为`Array`对象，反复进行操作，除了可能的性能损失外（如果列表项非常长）并不会有其他成本。

###### 注意

要计算`Set`或`Map`集合中的项数，您使用`size`属性。这与数组不同，数组具有`length`属性。

# 创建带有键索引项的集合

## 问题

您想创建一个集合，其中每个项都带有唯一的字符串键。

## 解决方案

使用`Map`对象。每个对象都以唯一键索引（通常是字符串，但不一定）。要添加项目，可以调用`set()`方法。当需要检索特定项目时，可以通过键直接获取所需的项目：

```
const products = new Map();

// Add three items
products.set('RU007', {name: 'Rain Racer 2000', price: 1499.99});
products.set('STKY1', {name: 'Edible Tape', price: 3.99});
products.set('P38', {name: 'Escape Vehicle (Air)', price: 2999.00});

// Check for two items using the item code
console.log(products.has('RU007'));  // true
console.log(products.has('RU494'));  // false

// Retrieve an item
const product = products.get('P38');
if (typeof product !== 'undefined') {
  console.log(product.price);  // 2999
}

// Remove the Edible Tape item
products.delete('STKY1');

console.log(products.size);  // 2
```

## 讨论

当向`Map`对象添加项时，必须始终使用`set()`方法。不要陷入这个陷阱：

```
const products = new Map();

// Don't do this!
products['RU007'] = {name: 'Rain Racer 2000', price: 1499.99};
```

尽管一开始似乎能行得通（并且它使用了许多其他编程语言中用于名称-值集合的相同语法），但实际上是绕过了`Map`集合，并在`Map`对象上设置了一个名为`RU007`的普通属性。如果你用`for`...`of`循环迭代`Map`时，这些属性不会出现，并且它们对于`has()`或`get()`方法也是不可见的。

`Map`对象有一小组用于管理其内容的方法：`set()`、`get()`、`has()`和`delete()`。如果您想利用`Array`对象中的功能，可以使用静态的`Array.from()`方法轻松将`Map`转换为数组：

```
const productArray = Array.from(products);

console.log(productArray[0]);
 // ['RU007', {name: 'Rain Racer 2000', price: 1499.99}]
```

您可能期望此示例中的`productArray`将保存一组产品对象，但这并不完全正确。相反，`productsArray`中的每个元素都是一个*单独*的数组，其中第一个元素是键（如`*RUU07*`），第二个元素是值（产品对象）。

在某些情况下，当您将`Map`转换为数组时，可能不需要保留键名。也许键名不重要，或者被元素的属性重复了。在这种情况下，您可以选择转换您的集合，将键值丢弃，同时将数据从`Map`中复制出来。这是它的工作原理：

```
const productArray = Array.from(products, ([name, value]) => value);

console.log(productArray[0]);
 // {name: 'Rain Racer 2000', price: 1499.99}
```
