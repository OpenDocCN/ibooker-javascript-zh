# 第五章：语句

第四章将表达式描述为 JavaScript 短语。按照这个类比，*语句*是 JavaScript 句子或命令。就像英语句子用句号终止并用句号分隔开一样，JavaScript 语句用分号终止（§2.6）。表达式*被评估*以产生一个值，但语句*被执行*以使某事发生。

使某事发生的一种方法是评估具有副作用的表达式。具有副作用的表达式，如赋值和函数调用，可以独立作为语句存在，当以这种方式使用时被称为*表达式语句*。另一类语句是*声明语句*，它声明新变量并定义新函数。

JavaScript 程序只不过是一系列要执行的语句。默认情况下，JavaScript 解释器按照它们编写的顺序一个接一个地执行这些语句。改变这种默认执行顺序的另一种方法是使用 JavaScript 中的一些语句或*控制结构*：

条件语句

诸如`if`和`switch`这样的语句根据表达式的值使 JavaScript 解释器执行或跳过其他语句

循环

诸如`while`和`for`这样重复执行其他语句的语句

跳转

诸如`break`、`return`和`throw`这样的语句会导致解释器跳转到程序的另一个部分

接下来的章节描述了 JavaScript 中的各种语句并解释了它们的语法。表 5-1 在本章末尾总结了语法。JavaScript 程序只不过是一系列语句，用分号分隔开，因此一旦熟悉了 JavaScript 的语句，就可以开始编写 JavaScript 程序。

# 5.1 表达式语句

JavaScript 中最简单的语句是具有副作用的表达式。这种语句在第四章中有所展示。赋值语句是表达式语句的一个主要类别。例如：

```js
greeting = "Hello " + name;
i *= 3;
```

递增和递减运算符`++`和`--`与赋值语句相关。它们具有改变变量值的副作用，就像执行了一个赋值一样：

```js
counter++;
```

`delete` 运算符的重要副作用是删除对象属性。因此，它几乎总是作为语句使用，而不是作为更大表达式的一部分：

```js
delete o.x;
```

函数调用是另一种重要的表达式语句。例如：

```js
console.log(debugMessage);
displaySpinner(); // A hypothetical function to display a spinner in a web app.
```

这些函数调用是表达式，但它们具有影响主机环境或程序状态的副作用，并且在这里被用作语句。如果一个函数没有任何副作用，那么调用它就没有意义，除非它是更大表达式或赋值语句的一部分。例如，你不会仅仅计算余弦值然后丢弃结果：

```js
Math.cos(x);
```

但你可能会计算值并将其赋给一个变量以备将来使用：

```js
cx = Math.cos(x);
```

请注意，这些示例中的每行代码都以分号结束。

# 5.2 复合语句和空语句

就像逗号运算符（§4.13.7）将多个表达式组合成一个单一表达式一样，*语句块*将多个语句组合成一个*复合语句*。语句块只是一系列语句被花括号包围起来。因此，以下行作为单个语句，并可以在 JavaScript 需要单个语句的任何地方使用：

```js
{
    x = Math.PI;
    cx = Math.cos(x);
    console.log("cos(π) = " + cx);
}
```

关于这个语句块有几点需要注意。首先，它*不*以分号结束。块内的原始语句以分号结束，但块本身不以分号结束。其次，块内的行相对于包围它们的花括号缩进。这是可选的，但它使代码更易于阅读和理解。

就像表达式经常包含子表达式一样，许多 JavaScript 语句包含子语句。形式上，JavaScript 语法通常允许单个子语句。例如，`while`循环语法包括一个作为循环体的单个语句。使用语句块，您可以在这个单个允许的子语句中放置任意数量的语句。

复合语句允许您在 JavaScript 语法期望单个语句的地方使用多个语句。*空语句*则相反：它允许您在期望一个语句的地方不包含任何语句。空语句如下所示：

```js
;
```

当执行空语句时，JavaScript 解释器不会采取任何操作。空语句偶尔在您想要创建一个空循环体的循环时很有用。考虑以下`for`循环（`for`循环将在§5.4.3 中介绍）：

```js
// Initialize an array a
for(let i = 0; i < a.length; a[i++] = 0) ;
```

在这个循环中，所有工作都由表达式`a[i++] = 0`完成，不需要循环体。然而，JavaScript 语法要求循环体作为一个语句，因此使用了一个空语句——只是一个裸分号。

请注意，在`for`循环、`while`循环或`if`语句的右括号后意外包含分号可能导致难以检测的令人沮丧的错误。例如，以下代码可能不会按照作者的意图执行：

```js
if ((a === 0) || (b === 0));  // Oops! This line does nothing...
    o = null;                 // and this line is always executed.
```

当您有意使用空语句时，最好以一种清晰表明您是有意这样做的方式对代码进行注释。例如：

```js
for(let i = 0; i < a.length; a[i++] = 0) /* empty */ ;
```

# 5.3 条件语句

条件语句根据指定表达式的值执行或跳过其他语句。这些语句是您代码的决策点，有时也被称为“分支”。如果想象一个 JavaScript 解释器沿着代码路径执行，条件语句是代码分支成两个或多个路径的地方，解释器必须选择要遵循的路径。

以下小节解释了 JavaScript 的基本条件语句`if/else`，并介绍了更复杂的多路分支语句`switch`。

## 5.3.1 if

`if`语句是允许 JavaScript 做出决策的基本控制语句，更准确地说，是有条件地执行语句。该语句有两种形式。第一种是：

```js
if (*`expression`*)
    *`statement`*
```

在这种形式中，*expression*被评估。如果结果值为真值，将执行*statement*。如果*expression*为假值，则不执行*statement*。（有关真值和假值的定义，请参见§3.4。）例如：

```js
if (username == null)       // If username is null or undefined,
    username = "John Doe";  // define it
```

或者类似地：

```js
// If username is null, undefined, false, 0, "", or NaN, give it a new value
if (!username) username = "John Doe";
```

请注意，围绕*expression*的括号是`if`语句语法的必需部分。

JavaScript 语法要求在`if`关键字和括号表达式之后有一个语句，但您可以使用语句块将多个语句组合成一个。因此，`if`语句也可能如下所示：

```js
if (!address) {
    address = "";
    message = "Please specify a mailing address.";
}
```

第二种形式的`if`语句引入了一个`else`子句，当*expression*为`false`时执行。其语法如下：

```js
if (*`expression`*)
    *`statement1`*
else
    *`statement2`*
```

该语句形式在*expression*为真值时执行`statement1`，在*expression*为假值时执行`statement2`。例如：

```js
if (n === 1)
    console.log("You have 1 new message.");
else
    console.log(`You have ${n} new messages.`);
```

当您有嵌套的带有`else`子句的`if`语句时，需要谨慎确保`else`子句与适当的`if`语句配对。考虑以下行：

```js
i = j = 1;
k = 2;
if (i === j)
    if (j === k)
        console.log("i equals k");
else
    console.log("i doesn't equal j");    // WRONG!!
```

在这个例子中，内部的`if`语句形成了外部`if`语句语法允许的单个语句。不幸的是，不清楚（除了缩进给出的提示外）`else`与哪个`if`配对。而且在这个例子中，缩进是错误的，因为 JavaScript 解释器实际上将前一个例子解释为：

```js
if (i === j) {
    if (j === k)
        console.log("i equals k");
    else
        console.log("i doesn't equal j");    // OOPS!
}
```

JavaScript（与大多数编程语言一样）的规则是，默认情况下`else`子句是最近的`if`语句的一部分。为了使这个例子不那么模棱两可，更容易阅读、理解、维护和调试，您应该使用花括号：

```js
if (i === j) {
    if (j === k) {
        console.log("i equals k");
    }
} else {  // What a difference the location of a curly brace makes!
    console.log("i doesn't equal j");
}
```

许多程序员习惯将 `if` 和 `else` 语句的主体（以及其他复合语句，如 `while` 循环）放在花括号中，即使主体只包含一个语句。始终如此可以防止刚才显示的问题，我建议你采用这种做法。在这本印刷书中，我非常重视保持示例代码的垂直紧凑性，并且并不总是遵循自己在这个问题上的建议。

## 5.3.2 else if

`if/else` 语句评估一个表达式并根据结果执行两个代码块中的一个。但是当你需要执行多个代码块中的一个时怎么办？一种方法是使用 `else if` 语句。`else if` 实际上不是一个 JavaScript 语句，而只是一个经常使用的编程习惯，当使用重复的 `if/else` 语句时会出现：

```js
if (n === 1) {
    // Execute code block #1
} else if (n === 2) {
    // Execute code block #2
} else if (n === 3) {
    // Execute code block #3
} else {
    // If all else fails, execute block #4
}
```

这段代码没有什么特别之处。它只是一系列 `if` 语句，每个后续的 `if` 都是前一个语句的 `else` 子句的一部分。使用 `else if` 习惯比在其语法上等效的完全嵌套形式中编写这些语句更可取，也更易读：

```js
if (n === 1) {
    // Execute code block #1
}
else {
    if (n === 2) {
        // Execute code block #2
    }
    else {
        if (n === 3) {
            // Execute code block #3
        }
        else {
            // If all else fails, execute block #4
        }
    }
}
```

## 5.3.3 switch

`if` 语句会导致程序执行流程的分支，你可以使用 `else if` 习惯来执行多路分支。然而，当所有分支都依赖于相同表达式的值时，这并不是最佳解决方案。在这种情况下，多次在多个 `if` 语句中评估该表达式是浪费的。

`switch` 语句正好处理这种情况。`switch` 关键字后跟着括号中的表达式和花括号中的代码块：

```js
switch(*`expression`*) {
    *`statements`*
}
```

然而，`switch` 语句的完整语法比这更复杂。代码块中的各个位置都用 `case` 关键字标记，后跟一个表达式和一个冒号。当 `switch` 执行时，它计算*表达式*的值，然后寻找一个 `case` 标签，其表达式的值与之相同（相同性由 `===` 运算符确定）。如果找到一个匹配值的 `case`，它会从标记为 `case` 的语句开始执行代码块。如果找不到具有匹配值的 `case`，它会寻找一个标记为 `default:` 的语句。如果没有 `default:` 标签，`switch` 语句会跳过整个代码块。

`switch` 是一个很难解释的语句；通过一个例子，它的操作会变得更加清晰。下面的 `switch` 语句等同于前一节中展示的重复的 `if/else` 语句：

```js
switch(n) {
case 1:                        // Start here if n === 1
    // Execute code block #1.
    break;                     // Stop here
case 2:                        // Start here if n === 2
    // Execute code block #2.
    break;                     // Stop here
case 3:                        // Start here if n === 3
    // Execute code block #3.
    break;                     // Stop here
default:                       // If all else fails...
    // Execute code block #4.
    break;                     // Stop here
}
```

注意这段代码中每个 `case` 结尾使用的 `break` 关键字。`break` 语句会在本章后面描述，它会导致解释器跳出（或“中断”）`switch` 语句并继续执行后面的语句。`switch` 语句中的 `case` 子句只指定所需代码的*起始点*；它们不指定任何结束点。在没有 `break` 语句的情况下，`switch` 语句会从与其*表达式*值匹配的 `case` 标签开始执行其代码块，并继续执行语句直到达到代码块的末尾。在极少数情况下，编写“穿透”从一个 `case` 标签到下一个的代码是有用的，但 99% 的情况下，你应该小心地用 `break` 语句结束每个 `case`。（然而，在函数内部使用 `switch` 时，你可以使用 `return` 语句代替 `break` 语句。两者都用于终止 `switch` 语句并防止执行穿透到下一个 `case`。）

这里是 `switch` 语句的一个更加现实的例子；它根据值的类型将值转换为字符串：

```js
function convert(x) {
    switch(typeof x) {
    case "number":            // Convert the number to a hexadecimal integer
        return x.toString(16);
    case "string":            // Return the string enclosed in quotes
        return '"' + x + '"';
    default:                  // Convert any other type in the usual way
        return String(x);
    }
}
```

请注意，在前两个示例中，`case`关键字分别后跟数字和字符串字面量。这是`switch`语句在实践中最常用的方式，但请注意，ECMAScript 标准允许每个`case`后跟任意表达式。

`switch`语句首先评估跟在`switch`关键字后面的表达式，然后按照它们出现的顺序评估`case`表达式，直到找到匹配的值。匹配的情况是使用`===`身份运算符确定的，而不是`==`相等运算符，因此表达式必须在没有任何类型转换的情况下匹配。

因为并非每次执行`switch`语句时都会评估所有`case`表达式，所以应避免使用包含函数调用或赋值等副作用的`case`表达式。最安全的做法是将`case`表达式限制为常量表达式。

如前所述，如果没有`case`表达式与`switch`表达式匹配，`switch`语句将从标记为`default:`的语句处开始执行其主体。如果没有`default:`标签，则`switch`语句将完全跳过其主体。请注意，在所示示例中，`default:`标签出现在`switch`主体的末尾，跟在所有`case`标签后面。这是一个逻辑和常见的位置，但实际上它可以出现在语句主体的任何位置。

# 5.4 循环

要理解条件语句，我们可以想象 JavaScript 解释器通过源代码的分支路径。*循环语句*是将该路径弯回自身以重复代码部分的语句。JavaScript 有五个循环语句：`while`、`do/while`、`for`、`for/of`（及其`for/await`变体）和`for/in`。以下各小节依次解释每个循环语句。循环的一个常见用途是遍历数组元素。§7.6 详细讨论了这种循环，并涵盖了 Array 类定义的特殊循环方法。

## 5.4.1 while

就像`if`语句是 JavaScript 的基本条件语句一样，`while`语句是 JavaScript 的基本循环语句。它的语法如下：

```js
while (*`expression`*)
    *`statement`*
```

要执行`while`语句，解释器首先评估*expression*。如果表达式的值为假值，则解释器跳过作为循环体的*statement*并继续执行程序中的下一条语句。另一方面，如果*expression*为真值，则解释器执行*statement*并重复，跳回循环的顶部并再次评估*expression*。另一种说法是，解释器在*expression*为真值时重复执行*statement*。请注意，您可以使用`while(true)`语法创建一个无限循环。

通常，您不希望 JavaScript 一遍又一遍地执行完全相同的操作。在几乎每个循环中，一个或多个变量会随着循环的每次*迭代*而改变。由于变量会改变，执行*statement*的操作可能每次循环时都不同。此外，如果涉及到*expression*中的变化变量，那么表达式的值可能每次循环时都不同。这很重要；否则，一开始为真值的表达式永远不会改变，循环永远不会结束！以下是一个打印从 0 到 9 的数字的`while`循环示例：

```js
let count = 0;
while(count < 10) {
    console.log(count);
    count++;
}
```

正如你所看到的，变量`count`从 0 开始，并且在循环体运行每次后递增。一旦循环执行了 10 次，表达式变为`false`（即变量`count`不再小于 10），`while`语句结束，解释器可以继续执行程序中的下一条语句。许多循环都有像`count`这样的计数变量。变量名`i`、`j`和`k`通常用作循环计数器，但如果使用更具描述性的名称可以使代码更易于理解。

## 5.4.2 do/while

`do/while`循环类似于`while`循环，不同之处在于循环表达式在循环底部测试而不是在顶部测试。这意味着循环体始终至少执行一次。语法是：

```js
do
    *`statement`*
while (*`expression`*);
```

`do/while`循环比其`while`表亲更少使用——实际上，很少有确定要执行至少一次循环的情况。以下是`do/while`循环的示例：

```js
function printArray(a) {
    let len = a.length, i = 0;
    if (len === 0) {
        console.log("Empty Array");
    } else {
        do {
            console.log(a[i]);
        } while(++i < len);
    }
}
```

`do/while`循环和普通的`while`循环之间有一些语法上的差异。首先，`do`循环需要`do`关键字（标记循环开始）和`while`关键字（标记结束并引入循环条件）。此外，`do`循环必须始终以分号结尾。如果循环体用大括号括起来，则`while`循环不需要分号。

## 5.4.3 for

`for`语句提供了一个循环结构，通常比`while`语句更方便。`for`语句简化了遵循常见模式的循环。大多数循环都有某种计数变量。该变量在循环开始之前初始化，并在每次循环迭代之前进行测试。最后，在循环体结束之前，计数变量会递增或以其他方式更新，然后再次测试该变量。在这种循环中，初始化、测试和更新是循环变量的三个关键操作。`for`语句将这三个操作编码为表达式，并将这些表达式作为循环语法的显式部分：

```js
for(*`initialize`* ; *`test`* ; *`increment`*)
    *`statement`*
```

*initialize*、*test*和*increment*是三个（用分号分隔的）表达式，负责初始化、测试和递增循环变量。将它们都放在循环的第一行中可以轻松理解`for`循环正在做什么，并防止遗漏初始化或递增循环变量等错误。

解释`for`循环如何工作的最简单方法是展示等效的`while`循环：²

```js
*`initialize`*;
while(*`test`*) {
    *`statement`*
    *`increment`*;
}
```

换句话说，*initialize*表达式在循环开始之前只计算一次。为了有用，此表达式必须具有副作用（通常是赋值）。JavaScript 还允许*initialize*是一个变量声明语句，这样您可以同时声明和初始化循环计数器。*test*表达式在每次迭代之前进行评估，并控制循环体是否执行。如果*test*评估为真值，则执行循环体的*statement*。最后，评估*increment*表达式。同样，这必须是具有副作用的表达式才能有效。通常，它是一个赋值表达式，或者使用`++`或`--`运算符。

我们可以使用以下`for`循环打印从 0 到 9 的数字。将其与前一节中显示的等效`while`循环进行对比：

```js
for(let count = 0; count < 10; count++) {
    console.log(count);
}
```

当然，循环可能比这个简单示例复杂得多，有时多个变量在循环的每次迭代中都会发生变化。这种情况是 JavaScript 中唯一常用逗号运算符的地方；它提供了一种将多个初始化和递增表达式组合成适合在`for`循环中使用的单个表达式的方法：

```js
let i, j, sum = 0;
for(i = 0, j = 10 ; i < 10 ; i++, j--) {
    sum += i * j;
}
```

到目前为止，我们所有的循环示例中，循环变量都是数字。这是很常见的，但并非必须的。以下代码使用`for`循环遍历一个链表数据结构并返回列表中的最后一个对象（即，第一个没有`next`属性的对象）：

```js
function tail(o) {                          // Return the tail of linked list o
    for(; o.next; o = o.next) /* empty */ ; // Traverse while o.next is truthy
    return o;
}
```

注意，这段代码没有*初始化*表达式。`for`循环中的三个表达式中的任何一个都可以省略，但两个分号是必需的。如果省略*测试*表达式，则循环将永远重复，`for(;;)`就像`while(true)`一样是写无限循环的另一种方式。

## 5.4.4 `for/of`

ES6 定义了一种新的循环语句：`for/of`。这种新类型的循环使用`for`关键字，但是与常规的`for`循环完全不同。（它也与我们将在§5.4.5 中描述的旧的`for/in`循环完全不同。）

`for/of`循环适用于*可迭代*对象。我们将在第十二章中详细解释对象何时被视为可迭代，但在本章中，只需知道数组、字符串、集合和映射是可迭代的：它们代表一个序列或一组元素，您可以使用`for/of`循环进行循环或迭代。

例如，这里是我们如何使用`for/of`循环遍历一个数字数组的元素并计算它们的总和：

```js
let data = [1, 2, 3, 4, 5, 6, 7, 8, 9], sum = 0;
for(let element of data) {
    sum += element;
}
sum       // => 45
```

表面上，语法看起来像是常规的`for`循环：`for`关键字后面跟着包含有关循环应该执行的详细信息的括号。在这种情况下，括号包含一个变量声明（或者对于已经声明的变量，只是变量的名称），后面跟着`of`关键字和一个求值为可迭代对象的表达式，就像这种情况下的`data`数组一样。与所有循环一样，`for/of`循环的主体跟在括号后面，通常在花括号内。

在刚才显示的代码中，循环体会针对`data`数组的每个元素运行一次。在执行循环体之前，数组的下一个元素会被分配给元素变量。数组元素按顺序从第一个到最后一个进行迭代。

数组是“实时”迭代的——在迭代过程中进行的更改可能会影响迭代的结果。如果我们在循环体内添加`data.push(sum);`这行代码，那么我们将创建一个无限循环，因为迭代永远无法到达数组的最后一个元素。

### 使用对象进行`for/of`循环

对象默认情况下不可迭代。尝试在常规对象上使用`for/of`会在运行时引发 TypeError：

```js
let o = { x: 1, y: 2, z: 3 };
for(let element of o) { // Throws TypeError because o is not iterable
    console.log(element);
}
```

如果要遍历对象的属性，可以使用`for/in`循环（在§5.4.5 中介绍），或者使用`for/of`与`Object.keys()`方法：

```js
let o = { x: 1, y: 2, z: 3 };
let keys = "";
for(let k of Object.keys(o)) {
    keys += k;
}
keys  // => "xyz"
```

这是因为`Object.keys()`返回一个对象的属性名称数组，数组可以使用`for/of`进行迭代。还要注意，与上面的数组示例不同，对象的键的这种迭代不是实时的——在循环体中对对象`o`进行的更改不会影响迭代。如果您不关心对象的键，也可以像这样迭代它们对应的值：

```js
let sum = 0;
for(let v of Object.values(o)) {
    sum += v;
}
sum // => 6
```

如果您对对象属性的键和值都感兴趣，可以使用`for/of`与`Object.entries()`和解构赋值：

```js
let pairs = "";
for(let [k, v] of Object.entries(o)) {
    pairs += k + v;
}
pairs  // => "x1y2z3"
```

`Object.entries()`返回一个数组，其中每个内部数组表示对象的一个属性的键/值对。在这个代码示例中，我们使用解构赋值来将这些内部数组解包成两个单独的变量。

### 使用字符串进行`for/of`循环

在 ES6 中，字符串是逐个字符可迭代的：

```js
let frequency = {};
for(let letter of "mississippi") {
    if (frequency[letter]) {
        frequency[letter]++;
    } else {
        frequency[letter] = 1;
    }
}
frequency   // => {m: 1, i: 4, s: 4, p: 2}
```

请注意，字符串是按 Unicode 代码点迭代的，而不是按 UTF-16 字符。字符串“I ❤ ![](img/cat.png)”的`.length`为 5（因为两个表情符号字符分别需要两个 UTF-16 字符来表示）。但如果您使用`for/of`迭代该字符串，循环体将运行三次，分别为每个代码点“I”、“❤”和“![](img/cat.png)”。

### 使用 Set 和 Map 进行 `for/of`

内置的 ES6 Set 和 Map 类是可迭代的。当您使用 `for/of` 迭代 Set 时，循环体会为集合的每个元素运行一次。您可以使用以下代码打印文本字符串中的唯一单词：

```js
let text = "Na na na na na na na na Batman!";
let wordSet = new Set(text.split(" "));
let unique = [];
for(let word of wordSet) {
    unique.push(word);
}
unique // => ["Na", "na", "Batman!"]
```

Map 是一个有趣的情况，因为 Map 对象的迭代器不会迭代 Map 键或 Map 值，而是键/值对。在每次迭代中，迭代器返回一个数组，其第一个元素是键，第二个元素是相应的值。给定一个 Map `m`，您可以像这样迭代并解构其键/值对：

```js
let m = new Map([[1, "one"]]);
for(let [key, value] of m) {
    key    // => 1
    value  // => "one"
}
```

### 使用 `for/await` 进行异步迭代

ES2018 引入了一种新类型的迭代器，称为*异步迭代器*，以及与之配套的 `for/of` 循环的变体，称为 `for/await` 循环，可与异步迭代器一起使用。

您需要阅读第十二章和第十三章才能理解 `for/await` 循环，但以下是代码示例：

```js
// Read chunks from an asynchronously iterable stream and print them out
async function printStream(stream) {
    for await (let chunk of stream) {
        console.log(chunk);
    }
}
```

## 5.4.5 for/in

`for/in` 循环看起来很像 `for/of` 循环，只是将 `of` 关键字更改为 `in`。在 `of` 之后，`for/of` 循环需要一个可迭代对象，而 `for/in` 循环在 `in` 之后可以使用任何对象。`for/of` 循环是 ES6 中的新功能，但 `for/in` 从 JavaScript 最初就存在（这就是为什么它具有更自然的语法）。

`for/in` 语句循环遍历指定对象的属性名称。语法如下：

```js
for (*`variable`* in *`object`*)
    *`statement`*
```

*variable* 通常命名一个变量，但它也可以是一个变量声明或任何适合作为赋值表达式左侧的内容。*object* 是一个求值为对象的表达式。通常情况下，*statement* 是作为循环主体的语句或语句块。

您可能会像这样使用 `for/in` 循环：

```js
for(let p in o) {      // Assign property names of o to variable p
    console.log(o[p]); // Print the value of each property
}
```

要执行 `for/in` 语句，JavaScript 解释器首先评估 *object* 表达式。如果它评估为 `null` 或 `undefined`，解释器将跳过循环并继续执行下一条语句。解释器现在会为对象的每个可枚举属性执行循环体。然而，在每次迭代之前，解释器会评估 *variable* 表达式并将属性的名称（一个字符串值）赋给它。

请注意，在 `for/in` 循环中的 *variable* 可以是任意表达式，只要它评估为适合赋值左侧的内容。这个表达式在每次循环时都会被评估，这意味着它可能每次评估的结果都不同。例如，您可以使用以下代码将所有对象属性的名称复制到数组中：

```js
let o = { x: 1, y: 2, z: 3 };
let a = [], i = 0;
for(a[i++] in o) /* empty */;
```

JavaScript 数组只是一种特殊类型的对象，数组索引是可以用 `for/in` 循环枚举的对象属性。例如，以下代码后面加上这行代码，将枚举数组索引 0、1 和 2：

```js
for(let i in a) console.log(i);
```

我发现在我的代码中常见的错误来源是意外使用数组时使用 `for/in` 而不是 `for/of`。在处理数组时，您几乎总是希望使用 `for/of` 而不是 `for/in`。

`for/in` 循环实际上并不枚举对象的所有属性。它不会枚举名称为符号的属性。对于名称为字符串的属性，它只循环遍历*可枚举*属性（参见§14.1）。核心 JavaScript 定义的各种内置方法都不可枚举。例如，所有对象都有一个 `toString()` 方法，但 `for/in` 循环不会枚举这个 `toString` 属性。除了内置方法，许多内置对象的其他属性也是不可枚举的。默认情况下，您代码定义的所有属性和方法都是可枚举的（您可以使用§14.1 中解释的技术使它们变为不可枚举）。

可枚举的继承属性（参见§6.3.2）也会被`for/in`循环枚举。这意味着如果您使用`for/in`循环，并且还使用定义了所有对象都继承的属性的代码，那么您的循环可能不会按您的预期方式运行。因此，许多程序员更喜欢使用`Object.keys()`的`for/of`循环而不是`for/in`循环。

如果`for/in`循环的主体删除尚未枚举的属性，则该属性将不会被枚举。如果循环的主体在对象上定义了新属性，则这些属性可能会被枚举，也可能不会被枚举。有关`for/in`枚举对象属性的顺序的更多信息，请参见§6.6.1。

# 5.5 跳转

另一类 JavaScript 语句是*跳转语句*。顾名思义，这些语句会导致 JavaScript 解释器跳转到源代码中的新位置。`break`语句使解释器跳转到循环或其他语句的末尾。`continue`使解释器跳过循环体的其余部分，并跳回到循环的顶部开始新的迭代。JavaScript 允许对语句进行命名，或*标记*，`break`和`continue`可以标识目标循环或其他语句标签。

`return`语句使解释器从函数调用跳回到调用它的代码，并提供调用的值。`throw`语句是一种临时从生成器函数返回的方式。`throw`语句引发异常，并设计用于与`try/catch/finally`语句一起工作，后者建立了一个异常处理代码块。这是一种复杂的跳转语句：当抛出异常时，解释器会跳转到最近的封闭异常处理程序，该处理程序可能在同一函数中或在调用函数的调用堆栈中。

关于这些跳转语句的详细信息在接下来的章节中。

## 5.5.1 标记语句

任何语句都可以通过在其前面加上标识符和冒号来*标记*：

```js
*`identifier`*: *`statement`*
```

通过给语句加上标签，您为其赋予一个名称，以便在程序的其他地方引用它。您可以为任何语句加上标签，尽管只有为具有主体的语句加上标签才有用，例如循环和条件语句。通过给循环命名，您可以在循环体内使用`break`和`continue`语句来退出循环或直接跳转到循环的顶部开始下一次迭代。`break`和`continue`是唯一使用语句标签的 JavaScript 语句；它们在以下子节中介绍。这里是一个带有标签的`while`循环和使用标签的`continue`语句的示例。

```js
mainloop: while(token !== null) {
    // Code omitted...
    continue mainloop;  // Jump to the next iteration of the named loop
    // More code omitted...
}
```

用于标记语句的*标识符*可以是任何合法的 JavaScript 标识符，不能是保留字。标签的命名空间与变量和函数的命名空间不同，因此您可以将相同的标识符用作语句标签和变量或函数名称。语句标签仅在其适用的语句内部定义（当然也包括其子语句）。语句不能具有包含它的语句相同的标签，但是只要一个语句不嵌套在另一个语句内，两个语句可以具有相同的标签。标记的语句本身也可以被标记。实际上，这意味着任何语句可以具有多个标签。

## 5.5.2 break

单独使用的`break`语句会导致最内层的循环或`switch`语句立即退出。其语法很简单：

```js
break;
```

因为它导致循环或`switch`退出，所以这种形式的`break`语句只有在出现在这些语句内部时才合法。

您已经看到了`switch`语句中`break`语句的示例。在循环中，当不再需要完成循环时，通常会提前退出。当循环具有复杂的终止条件时，通常更容易使用`break`语句实现其中一些条件，而不是尝试在单个循环表达式中表达所有条件。以下代码搜索数组元素以找到特定值。当它在数组中找到所需的内容时，循环以正常方式终止；如果在数组中找到所需的内容，则使用`break`语句终止：

```js
for(let i = 0; i < a.length; i++) {
    if (a[i] === target) break;
}
```

JavaScript 还允许在`break`关键字后面跟着一个语句标签（只是标识符，没有冒号）：

```js
break *`labelname`*;
```

当`break`与标签一起使用时，它会跳转到具有指定标签的结束语句，或终止该结束语句。如果没有具有指定标签的结束语句，则以这种形式使用`break`语句是语法错误。使用这种形式的`break`语句时，命名的语句不必是循环或`switch`：`break`可以“跳出”任何包含语句。这个语句甚至可以是一个仅用于使用标签命名块的大括号组成的语句块。

在`break`关键字和*labelname*之间不允许换行。这是由于 JavaScript 自动插入省略的分号：如果在`break`关键字和后面的标签之间放置换行符，JavaScript 会认为您想使用简单的、无标签的语句形式，并将换行符视为分号。（参见§2.6。）

当您想要跳出不是最近的循环或`switch`的语句时，您需要带标签的`break`语句。以下代码演示了：

```js
let matrix = getData();  // Get a 2D array of numbers from somewhere
// Now sum all the numbers in the matrix.
let sum = 0, success = false;
// Start with a labeled statement that we can break out of if errors occur
computeSum: if (matrix) {
    for(let x = 0; x < matrix.length; x++) {
        let row = matrix[x];
        if (!row) break computeSum;
        for(let y = 0; y < row.length; y++) {
            let cell = row[y];
            if (isNaN(cell)) break computeSum;
            sum += cell;
        }
    }
    success = true;
}
// The break statements jump here. If we arrive here with success == false
// then there was something wrong with the matrix we were given.
// Otherwise, sum contains the sum of all cells of the matrix.
```

最后，请注意，`break`语句，无论是否带有标签，都不能跨越函数边界转移控制。例如，您不能给函数定义语句加上标签，然后在函数内部使用该标签。

## 5.5.3 continue

`continue`语句类似于`break`语句。但是，`continue`不是退出循环，而是在下一次迭代时重新开始循环。`continue`语句的语法与`break`语句一样简单：

```js
continue;
```

`continue`语句也可以与标签一起使用：

```js
continue *`labelname`*;
```

`continue`语句，无论是带标签还是不带标签，只能在循环体内使用。在其他任何地方使用它都会导致语法错误。

当执行`continue`语句时，将终止当前循环的迭代，并开始下一次迭代。对于不同类型的循环，这意味着不同的事情：

+   在`while`循环中，循环开始时测试循环开头的指定*表达式*，如果为`true`，则从顶部执行循环体。

+   在`do/while`循环中，执行跳转到循环底部，然后再次测试循环条件，然后重新开始循环。

+   在`for`循环中，将评估*增量*表达式，并再次测试*测试*表达式以确定是否应进行另一次迭代。

+   在`for/of`或`for/in`循环中，循环将重新开始，下一个迭代值或下一个属性名将被赋给指定的变量。

请注意`while`和`for`循环中`continue`语句的行为差异：`while`循环直接返回到其条件，但`for`循环首先评估其*增量*表达式，然后返回到其条件。之前，我们考虑了`for`循环的行为，以等效的`while`循环来描述。然而，由于`continue`语句对这两种循环的行为不同，因此仅使用`while`循环无法完全模拟`for`循环。

以下示例显示了在发生错误时使用未标记的`continue`语句跳过当前迭代的其余部分的情况：

```js
for(let i = 0; i < data.length; i++) {
    if (!data[i]) continue;  // Can't proceed with undefined data
    total += data[i];
}
```

与`break`语句类似，`continue`语句可以在嵌套循环中的标记形式中使用，当要重新启动的循环不是直接包围的循环时。同样，与`break`语句一样，`continue`语句和其*labelname*之间不允许换行。

## 5.5.4 return

请记住函数调用是表达式，所有表达式都有值。函数内部的`return`语句指定了该函数调用的值。下面是`return`语句的语法：

```js
return *`expression`*;
```

`return`语句只能出现在函数体内部。在其他任何地方出现都会导致语法错误。当执行`return`语句时，包含它的函数将*expression*的值返回给调用者。例如：

```js
function square(x) { return x*x; } // A function that has a return statement
square(2)                          // => 4
```

没有`return`语句时，函数调用会依次执行函数体中的每个语句，直到到达函数末尾然后返回给调用者。在这种情况下，调用表达式评估为`undefined`。`return`语句通常出现在函数中的最后一个语句，但不一定非得是最后一个：当执行`return`语句时，函数返回给调用者，即使函数体中还有其他语句。

`return`语句也可以在没有*expression*的情况下使用，使函数返回`undefined`给调用者。例如：

```js
function displayObject(o) {
    // Return immediately if the argument is null or undefined.
    if (!o) return;
    // Rest of function goes here...
}
```

由于 JavaScript 的自动分号插入（§2.6），你不能在`return`关键字和其后的表达式之间插入换行符。

## 5.5.5 yield

`yield`语句与`return`语句非常相似，但仅在 ES6 生成器函数（参见§12.3）中使用，用于生成值序列中的下一个值而不实际返回：

```js
// A generator function that yields a range of integers
function* range(from, to) {
    for(let i = from; i <= to; i++) {
        yield i;
    }
}
```

要理解`yield`，你必须理解迭代器和生成器，这将在第十二章中介绍。然而，为了完整起见，这里包括了`yield`。（严格来说，`yield`是一个运算符而不是语句，如§12.4.2 中所解释的。）

## 5.5.6 throw

*异常*是指示发生了某种异常情况或错误的信号。*抛出*异常是指示发生了这样的错误或异常情况。*捕获*异常是处理它 - 采取必要或适当的措施来从异常中恢复。在 JavaScript 中，每当发生运行时错误或程序明确使用`throw`语句抛出异常时，都会抛出异常。异常可以通过`try/catch/finally`语句捕获，下一节将对此进行描述。

`throw`语句的语法如下：

```js
throw *`expression`*;
```

*expression*可能会评估为任何类型的值。你可以抛出一个代表错误代码的数字，或者包含人类可读错误消息的字符串。当 JavaScript 解释器本身抛出错误时，会使用 Error 类及其子类，你也可以使用它们。一个 Error 对象有一个`name`属性指定错误类型，一个`message`属性保存传递给构造函数的字符串。下面是一个示例函数，当使用无效参数调用时会抛出一个 Error 对象：

```js
function factorial(x) {
    // If the input argument is invalid, throw an exception!
    if (x < 0) throw new Error("x must not be negative");
    // Otherwise, compute a value and return normally
    let f;
    for(f = 1; x > 1; f *= x, x--) /* empty */ ;
    return f;
}
factorial(4)   // => 24
```

当抛出异常时，JavaScript 解释器立即停止正常程序执行，并跳转到最近的异常处理程序。异常处理程序使用`try/catch/finally`语句的`catch`子句编写，下一节将对其进行描述。如果抛出异常的代码块没有关联的`catch`子句，解释器将检查下一个最高级别的封闭代码块，看看它是否有与之关联的异常处理程序。这将一直持续下去，直到找到处理程序。如果在一个不包含`try/catch/finally`语句来处理异常的函数中抛出异常，异常将传播到调用该函数的代码。通过这种方式，异常通过 JavaScript 方法的词法结构向上传播，并沿着调用堆栈向上传播。如果从未找到异常处理程序，异常将被视为错误并报告给用户。

## 5.5.7 try/catch/finally

`try/catch/finally`语句是 JavaScript 的异常处理机制。该语句的`try`子句简单地定义了要处理异常的代码块。`try`块后面是一个`catch`子句，当`try`块内部发生异常时，将调用一组语句。`catch`子句后面是一个`finally`块，其中包含清理代码，无论`try`块中发生了什么，都保证会执行。`catch`和`finally`块都是可选的，但`try`块必须至少伴随其中一个。`try`、`catch`和`finally`块都以大括号开始和结束。这些大括号是语法的必要部分，即使一个子句只包含一个语句也不能省略。

以下代码示例说明了`try/catch/finally`语句的语法和目的：

```js
try {
    // Normally, this code runs from the top of the block to the bottom
    // without problems. But it can sometimes throw an exception,
    // either directly, with a throw statement, or indirectly, by calling
    // a method that throws an exception.
}
catch(e) {
    // The statements in this block are executed if, and only if, the try
    // block throws an exception. These statements can use the local variable
    // e to refer to the Error object or other value that was thrown.
    // This block may handle the exception somehow, may ignore the
    // exception by doing nothing, or may rethrow the exception with throw.
}
finally {
    // This block contains statements that are always executed, regardless of
    // what happens in the try block. They are executed whether the try
    // block terminates:
    //   1) normally, after reaching the bottom of the block
    //   2) because of a break, continue, or return statement
    //   3) with an exception that is handled by a catch clause above
    //   4) with an uncaught exception that is still propagating
}
```

请注意，`catch`关键字通常后面跟着一个括号中的标识符。这个标识符类似于函数参数。当捕获到异常时，与异常相关联的值（例如一个 Error 对象）将被分配给这个参数。与`catch`子句关联的标识符具有块作用域——它只在`catch`块内定义。

这里是`try/catch`语句的一个实际例子。它使用了前一节中定义的`factorial()`方法以及客户端 JavaScript 方法`prompt()`和`alert()`来进行输入和输出：

```js
try {
    // Ask the user to enter a number
    let n = Number(prompt("Please enter a positive integer", ""));
    // Compute the factorial of the number, assuming the input is valid
    let f = factorial(n);
    // Display the result
    alert(n + "! = " + f);
}
catch(ex) {     // If the user's input was not valid, we end up here
    alert(ex);  // Tell the user what the error is
}
```

这个例子是一个没有`finally`子句的`try/catch`语句。虽然`finally`不像`catch`那样经常使用，但它也是有用的。然而，它的行为需要额外的解释。如果`try`块的任何部分被执行，`finally`子句将被执行。它通常用于在`try`子句中的代码执行完毕后进行清理。

在正常情况下，JavaScript 解释器执行完`try`块后，然后继续执行`finally`块，执行任何必要的清理工作。如果解释器因为`return`、`continue`或`break`语句而离开`try`块，那么在解释器跳转到新目的地之前，将执行`finally`块。

如果在`try`块中发生异常，并且有一个关联的`catch`块来处理异常，解释器首先执行`catch`块，然后执行`finally`块。如果没有本地`catch`块来处理异常，解释器首先执行`finally`块，然后跳转到最近的包含`catch`子句。

如果`finally`块本身导致使用`return`、`continue`、`break`或`throw`语句跳转，或通过调用抛出异常的方法，解释器会放弃任何待处理的跳转并执行新的跳转。例如，如果`finally`子句抛出异常，那个异常会替换正在被抛出的任何异常。如果`finally`子句发出`return`语句，方法会正常返回，即使已经抛出异常但尚未处理。

`try`和`finally`可以在没有`catch`子句的情况下一起使用。在这种情况下，`finally`块只是保证会被执行的清理代码，无论`try`块中发生了什么。请记住，我们无法完全用`while`循环模拟`for`循环，因为`continue`语句对这两种循环的行为是不同的。如果我们添加一个`try/finally`语句，我们可以编写一个像`for`循环一样工作并正确处理`continue`语句的`while`循环：

```js
// Simulate for(*`initialize`* ; *`test`* ;*`increment`* ) body;
*`initialize`* ;
while( *`test`* ) {
    try { *`body`* ; }
    finally { *`increment`* ; }
}
```

但是请注意，包含`break`语句的*body*在`while`循环中的行为略有不同（导致在退出之前额外增加一次递增）与在`for`循环中的行为不同，因此即使有`finally`子句，也无法完全用`while`模拟`for`循环。

# 5.6 其他语句

本节描述了剩余的三个 JavaScript 语句——`with`、`debugger`和`"use strict"`。

## 5.6.1 with

`with`语句会将指定对象的属性作为作用域内的变量运行一段代码块。它的语法如下：

```js
with (*`object`*)
    *`statement`*
```

这个语句创建一个临时作用域，将*object*的属性作为变量，然后在该作用域内执行*statement*。

`with`语句在严格模式下是被禁止的（参见§5.6.3），在非严格模式下应被视为已弃用：尽量避免使用。使用`with`的 JavaScript 代码很难优化，并且可能比不使用`with`语句编写的等效代码运行得慢得多。

`with`语句的常见用法是使得在深度嵌套的对象层次结构中更容易工作。例如，在客户端 JavaScript 中，你可能需要输入这样的表达式来访问 HTML 表单的元素：

```js
document.forms[0].address.value
```

如果你需要多次编写这样的表达式，你可以使用`with`语句将表单对象的属性视为变量处理：

```js
with(document.forms[0]) {
    // Access form elements directly here. For example:
    name.value = "";
    address.value = "";
    email.value = "";
}
```

这样可以减少你需要输入的内容：你不再需要在每个表单属性名称前加上`document.forms[0]`。当然，避免使用`with`语句并像这样编写前面的代码同样简单：

```js
let f = document.forms[0];
f.name.value = "";
f.address.value = "";
f.email.value = "";
```

请注意，如果在`with`语句的主体中使用`const`、`let`或`var`声明变量或常量，它会创建一个普通变量，而不会在指定对象中定义一个新属性。

## 5.6.2 debugger

`debugger`语句通常不会执行任何操作。然而，如果一个调试器程序可用且正在运行，那么实现可能（但不是必须）执行某种调试操作。实际上，这个语句就像一个断点：JavaScript 代码的执行会停止，你可以使用调试器打印变量的值，检查调用堆栈等。例如，假设你在函数`f()`中遇到异常，因为它被使用未定义的参数调用，而你无法弄清楚这个调用是从哪里来的。为了帮助你调试这个问题，你可以修改`f()`，使其如下所示开始：

```js
function f(o) {
  if (o === undefined) debugger;  // Temporary line for debugging purposes
  ...                             // The rest of the function goes here.
}
```

现在，当没有参数调用`f()`时，执行会停止，你可以使用调试器检查调用堆栈，并找出这个错误调用是从哪里来的。

请注意，仅仅拥有一个调试器是不够的：`debugger`语句不会为你启动调试器。然而，如果你正在使用一个网页浏览器并且打开了开发者工具控制台，这个语句会导致断点。

## 5.6.3 “use strict”

`"use strict"`是 ES5 中引入的*指令*。 指令不是语句（但足够接近，以至于在此处记录了`"use strict"`）。 `"use strict"`指令和常规语句之间有两个重要区别：

+   它不包括任何语言关键字：该指令只是一个表达式语句，由一个特殊的字符串文字（单引号或双引号）组成。

+   它只能出现在脚本的开头或函数体的开头，在任何真实语句出现之前。

`"use strict"`指令的目的是指示随后的代码（在脚本或函数中）是*严格代码*。 如果脚本有`"use strict"`指令，则脚本的顶级（非函数）代码是严格代码。 如果函数体在严格代码中定义或具有`"use strict"`指令，则函数体是严格代码。 如果从严格代码调用`eval()`方法，则传递给`eval()`的代码是严格代码，或者如果代码字符串包含`"use strict"`指令。 除了明确声明为严格的代码外，`class`体（第九章）中的任何代码或 ES6 模块（§10.3）中的任何代码都自动成为严格代码。 这意味着如果所有 JavaScript 代码都编写为模块，则所有代码都自动成为严格代码，您将永远不需要使用显式的`"use strict"`指令。

严格模式下执行*严格模式*。 严格模式是语言的受限子集，修复了重要的语言缺陷，并提供了更强的错误检查和增强的安全性。 由于严格模式不是默认设置，仍然使用语言的不足遗留功能的旧 JavaScript 代码将继续正确运行。 严格模式和非严格模式之间的区别如下（前三个特别重要）：

+   在严格模式下，不允许使用`with`语句。

+   在严格模式下，所有变量必须声明：如果将值分配给未声明的变量、函数、函数参数、`catch`子句参数或全局对象的属性，则会抛出 ReferenceError。（在非严格模式下，这将通过向全局对象添加新属性来隐式声明全局变量。）

+   在严格模式下，作为函数调用的函数（而不是作为方法）的`this`值为`undefined`。（在非严格模式下，作为函数调用的函数始终将全局对象作为其`this`值传递。）此外，在严格模式下，当使用`call()`或`apply()`（§8.7.4）调用函数时，`this`值正好是传递给`call()`或`apply()`的第一个参数的值。（在非严格模式下，`null`和`undefined`值将替换为全局对象，非对象值将转换为对象。）

+   在严格模式下，对不可写属性的赋值和尝试在不可扩展对象上创建新属性会抛出 TypeError。（在非严格模式下，这些尝试会静默失败。）

+   在严格模式下，传递给`eval()`的代码不能在调用者的范围内声明变量或定义函数，就像在非严格模式下那样。 相反，变量和函数定义存在于为`eval()`创建的新作用域中。 当`eval()`返回时，此作用域将被丢弃。

+   在严格模式下，函数中的 Arguments 对象（§8.3.3）保存传递给函数的值的静态副本。 在非严格模式下，Arguments 对象具有“神奇”的行为，其中数组的元素和命名函数参数都指向相同的值。

+   在严格模式下，如果`delete`运算符后跟未经限定的标识符（如变量、函数或函数参数），则会抛出 SyntaxError。（在非严格模式下，这样的`delete`表达式不起作用并计算为`false`。）

+   在严格模式下，尝试删除不可配置属性会抛出 TypeError。 （在非严格模式下，尝试失败，`delete`表达式的值为`false`。）

+   在严格模式下，对象字面量定义具有相同名称的两个或更多属性是语法错误。（在非严格模式下，不会发生错误。）

+   在严格模式下，函数声明具有两个或更多具有相同名称的参数是语法错误。（在非严格模式下，不会发生错误。）

+   在严格模式下，不允许使用八进制整数字面量（以 0 开头且后面不跟 x）。（在非严格模式下，一些实现允许八进制字面量。）

+   在严格模式下，标识符`eval`和`arguments`被视为关键字，不允许更改它们的值。不能为这些标识符分配值，将它们声明为变量，将它们用作函数名称，将它们用作函数参数名称，或将它们用作`catch`块的标识符。

+   在严格模式下，限制了检查调用堆栈的能力。在严格模式函数内，`arguments.caller`和`arguments.callee`都会抛出 TypeError。严格模式函数还具有`caller`和`arguments`属性，当读取时会抛出 TypeError。（一些实现在非严格函数上定义这些非标准属性。）

# 5.7 声明

关键字`const`、`let`、`var`、`function`、`class`、`import`和`export`在技术上不是语句，但它们看起来很像语句，因此本书非正式地将它们称为语句，因此它们在本章中值得一提。

这些关键字更准确地描述为*声明*而不是语句。我们在本章开头说过语句“让某事发生”。声明用于定义新值并为其赋予我们可以用来引用这些值的名称。它们本身并没有做太多事情，但通过为值提供名称，它们在重要意义上定义了程序中其他语句的含义。

当程序运行时，程序的表达式正在被评估，程序的语句正在被执行。程序中的声明不会像语句一样“运行”：相反，它们定义了程序本身的结构。可以粗略地将声明视为在代码开始运行之前处理的程序部分。

JavaScript 声明用于定义常量、变量、函数和类，并用于在模块之间导入和导出值。下一小节将给出所有这些声明的示例。它们在本书的其他地方都有更详细的介绍。

## 5.7.1 const、let 和 var

`const`、`let`和`var`声明在§3.10 中有介绍。在 ES6 及更高版本中，`const`声明常量，`let`声明变量。在 ES6 之前，`var`关键字是声明变量的唯一方式，没有办法声明常量。使用`var`声明的变量的作用域是包含函数而不是包含块。这可能导致错误，并且在现代 JavaScript 中，没有理由使用`var`而不是`let`。

```js
const TAU = 2*Math.PI;
let radius = 3;
var circumference = TAU * radius;
```

## 5.7.2 function

`function`声明用于定义函数，在第八章中有详细介绍。（我们还在§4.3 中看到`function`，那里它被用作函数表达式的一部分而不是函数声明。）函数声明如下所示：

```js
function area(radius) {
    return Math.PI * radius * radius;
}
```

函数声明创建一个函数对象并将其分配给指定的名称—在这个例子中是`area`。 在程序的其他地方，我们可以通过使用这个名称引用函数—并运行其中的代码。 JavaScript 代码块中的函数声明在代码运行之前被处理，并且函数名称在整个代码块中绑定到函数对象。 我们说函数声明被“提升”，因为它就好像它们都被移动到它们所在的作用域的顶部一样。 结果是调用函数的代码可以存在于程序中，在声明函数的代码之前。

§12.3 描述了一种特殊类型的函数，称为*生成器*。 生成器声明使用`function`关键字，但后面跟着一个星号。 §13.3 描述了异步函数，也是使用`function`关键字声明的，但前面加上`async`关键字。

## 5.7.3 类

在 ES6 及更高版本中，`class`声明创建一个新的类，并为其赋予一个我们可以用来引用它的名称。 类在第九章中有详细描述。 一个简单的类声明可能如下所示：

```js
class Circle {
    constructor(radius) { this.r = radius; }
    area() { return Math.PI * this.r * this.r; }
    circumference() { return 2 * Math.PI * this.r; }
}
```

与函数不同，类声明不会被提升，你不能在类声明之前的代码中使用以这种方式声明的类。

## 5.7.4 导入和导出

`import`和`export`声明一起使用，使得在 JavaScript 代码的一个模块中定义的值可以在另一个模块中使用。 模块是具有自己全局命名空间的 JavaScript 代码文件，完全独立于所有其他模块。 一个值（如函数或类）在一个模块中定义后，只有通过`export`导出并在另一个模块中使用`import`导入，才能在另一个模块中使用。 模块是第十章的主题，`import`和`export`在§10.3 中有详细介绍。

`import`指令用于从另一个 JavaScript 代码文件中导入一个或多个值，并在当前模块中为它们命名。 `import`指令有几种不同的形式。 以下是一些示例：

```js
import Circle from './geometry/circle.js';
import { PI, TAU } from './geometry/constants.js';
import { magnitude as hypotenuse } from './vectors/utils.js';
```

JavaScript 模块中的值是私有的，除非它们已经被明确导出，否则不能被导入到其他模块中。 `export`指令可以实现这一点：它声明当前模块中定义的一个或多个值被导出，因此可以被其他模块导入。 `export`指令比`import`指令有更多的变体。 这是其中之一：

```js
// geometry/constants.js
const PI = Math.PI;
const TAU = 2 * PI;
export { PI, TAU };
```

`export`关键字有时用作其他声明的修饰符，从而形成一种复合声明，同时定义一个常量、变量、函数或类并将其导出。 当一个模块只导出一个值时，通常使用特殊形式`export default`：

```js
export const TAU = 2 * Math.PI;
export function magnitude(x,y) { return Math.sqrt(x*x + y*y); }
export default class Circle { /* class definition omitted here */ }
```

# 5.8 JavaScript 语句总结

本章介绍了 JavaScript 语言的每个语句，总结在表 5-1 中。

表 5-1\. JavaScript 语句语法

| 语句 | 目的 |
| --- | --- |
| break | 退出最内层循环或`switch`或从命名封闭语句中退出 |
| case | 在`switch`语句中标记一个语句 |
| class | 声明一个类 |
| const | 声明和初始化一个或多个常量 |
| continue | 开始最内层循环或命名循环的下一次迭代 |
| debugger | 调试器断点 |
| default | 标记`switch`语句中的默认语句 |
| do/while | `while`循环的替代方案 |
| export | 声明可以被其他模块导入的值 |
| for | 一个易于使用的循环 |
| for/await | 异步迭代异步迭代器的值 |
| for/in | 枚举对象的属性名称 |
| for/of | 枚举可迭代对象（如数组）的值 |
| function | 声明一个函数 |
| if/else | 根据条件执行一个语句或另一个 |
| import | 声明在其他模块中定义的值的名称 |
| label | 为`break`和`continue`给语句命名 |
| let | 声明并初始化一个或多个块作用域变量（新语法） |
| return | 从函数中返回一个值 |
| switch | 多路分支到`case`或`default:`标签 |
| throw | 抛出异常 |
| try/catch/finally | 处理异常和代码清理 |
| “use strict” | 将严格模式限制应用于脚本或函数 |
| var | 声明并初始化一个或多个变量（旧语法） |
| while | 基本的循环结构 |
| with | 扩展作用域链（已弃用且在严格模式下禁止使用） |
| yield | 提供一个要迭代的值；仅在生成器函数中使用 |

¹ `case`表达式在运行时评估的事实使得 JavaScript 的`switch`语句与 C、C++和 Java 的`switch`语句有很大不同（且效率较低）。在那些语言中，`case`表达式必须是相同类型的编译时常量，并且`switch`语句通常可以编译为高效的*跳转表*。

² 当我们考虑在§5.5.3 中的`continue`语句时，我们会发现这个`while`循环并不是`for`循环的精确等价。
