# 第六章：函数

函数是你用来组装程序的离散、可重用代码例程的构建块。但在 JavaScript 中，这只是故事的一部分。

JavaScript 函数也是真正的*对象*—`Function` 类型的实例。它们可以被赋值给变量并在代码中传递。它们可以在表达式中声明，不需要函数名，并且可以使用简化的*箭头语法*。你甚至可以将一个函数包装在另一个函数中，以创建一个包含函数状态的私有包（称为*闭包*）。

函数也是 JavaScript 面向对象支持的核心。这是因为自定义类实际上只是一种特殊类型的构造函数（正如你将在第八章中看到的）。迟早，JavaScript 中的一切都会回到函数。

# 将函数作为参数传递给另一个函数

## 问题

你正在调用一个期望你提供自己函数的函数。最佳的传递方式是什么？

## 解决方案

JavaScript 中的许多函数接受，甚至要求，作为参数传递的函数。一些操作要求回调函数在任务完成时触发。其他需要使用你的函数完成更广泛的任务。例如，`Array` 对象的许多方法要求你提供一个用于排序、转换、组合或选择数据的函数。然后数组多次使用你的函数，直到处理完每个元素。

在提供函数作为参数时，有几种不同的方法可以使用。以下是三种常见模式：

+   提供对已在代码中其他位置声明的函数的引用。如果你想在应用程序的其他部分使用该函数，或者该函数特别长或复杂，这种方法是有意义的。

+   在*函数表达式*中声明函数，然后将其作为参数传递。这种方法适用于简单的任务，并且如果你不打算在其他地方使用该函数。

+   在需要时内联声明函数—当你将其作为参数传递给另一个函数时。这类似于第二种方法，但使你的代码更加紧凑。它最适合非常简短、直接的函数（尤其是一行代码）。

让我们从一个简单的页面开始，页面上有这个按钮：

```
<button id="runTest">Run Test</button>
```

我们如下附加事件处理程序：

```
// Attach button event handler.
document.getElementById('runTest').addEventListener("click", buttonClicked);
```

现在考虑内置的`setTimeout()`函数，它安排一个函数在一定延迟后运行（你提供函数）。这是将函数传递的第一种方法，使用一个名为`showMessage()`的单独函数：

```
// Runs when a button is clicked
function buttonClicked() {
  // Trigger the function after 2000 milliseconds (2 seconds)
  setTimeout(showMessage, 2000);
}

// Runs when setTimeout() triggers it
function showMessage() {
  alert('You clicked the button 2 seconds ago');
}
```

###### 注意

当你通过名称传递函数引用时，请确保不要添加一组空括号。这个例子将`showMessage`传递给`setTimeout()`函数。如果你意外地写成`showMessage()`，JavaScript 将立即*运行*`showMessage()`函数，并将其返回值传递给`setTimeout()`，而不是传递函数引用。

这是第二种方法，它在需要的地方使用函数表达式声明函数：

```
function buttonClicked() {
  // Declare a function expression to use with setTimeout()
  const timeoutCallback = function showMessage() {
    alert('You clicked the button 2 seconds ago');
  }

  // Trigger the function after 2000 milliseconds (2 seconds)
  setTimeout(timeoutCallback, 2000);
}
```

在这种情况下，`showMessage()`的作用域仅限于`buttonClicked()`函数。它无法从代码中的其他函数中调用。选择性地，你可以省略函数名（`showMessage`），使其成为一个*匿名函数*。无论哪种方式，`timeoutCallback`的工作方式都是相同的，但是函数名在调试时非常有用，因为在堆栈跟踪中会显示它如果发生错误。

这是第三种方法，它在调用`setTimeout()`时内联声明函数：

```
function buttonClicked() {
  // Trigger the function after 2000 milliseconds (2 seconds)
  setTimeout(function showMessage() {
    alert('You clicked the button 2 seconds ago');
  }, 2000);
}
```

现在`showMessage()`函数在一个语句中声明并传递给`setTimeout()`。任何代码的其他部分都无法与`showMessage()`交互，即使在`buttonClicked()`函数内部也是如此。选择性地，你可以省略名称`showMessage()`，使其成为一个匿名函数：

```
  setTimeout(function() {
    alert('You clicked the button 2 seconds ago');
  }, 2000);
```

你可以进一步简化这种方法，使用箭头语法，如示例中的“使用箭头函数”所示。但是对于长或复杂的代码例程，使用函数名是一个很好的实践。这是因为如果函数内部发生错误，你将在堆栈跟踪中看到函数名。

###### 注意

当你使用匿名函数时，注意你组织的风格约定。一个常见的模式是在同一行上放置`function()`声明和开放的`{`括号。然后，在下面放置匿名函数的所有代码，缩进一个额外的级别。最后，将闭合的`}`括号放在单独的一行上，紧接着函数调用的其余参数。

## 讨论

这三种方法展示了逐渐缩小范围的示例，从最可访问的函数（第一个示例中）到最不可访问的函数（最后一个示例中）。作为一个通用的规则，尽可能使用最窄的范围是最好的。这减少了代码中的歧义（使其更容易理解后续开发者），并减少了意外副作用的可能性。但是，这是一个权衡。随着函数变得更长和更复杂，内联声明变得不太可读。如果你想单独使用函数或对其运行单元测试，你需要将其拆分为一个单独的函数。

如果你对函数如何*使用*函数引用有任何疑问，这里有一个简单的示例，使用一个名为`callYouBack()`的自定义函数，它接受一个函数参数然后调用它。在`callYouBack()`函数内部，你像调用普通函数一样对待函数引用，通过名称调用它并提供它需要的任何参数：

```
function buttonClicked() {
  // Create a function that will handle the callback
  function logTime(time) {
    console.log('Logging at: ' + time.toLocaleTimeString());
  }

  console.log('About to call callYouBack()');
  callYouBack(logTime);
  console.log('All finished');
}

function callYouBack(callbackFunction) {
  console.log('Starting callYouBack()');

  // Call the provided function and supply an argument
  callbackFunction(new Date());

  console.log('Ending callYouBack()');
}
```

如果你运行这段代码并点击按钮，它会产生如下输出：

```
About to call callYouBack()
Starting callYouBack()
Logging at: 2:20:59 PM
Ending callYouBack()
All finished
```

## 参见

参见“使用箭头函数”，这种语法允许您简化匿名函数的声明，特别适用于返回值的单行函数。参见表 5-1 中接受函数参数的最重要的`Array`方法。

# 使用箭头函数

## 问题

想要使用 JavaScript 的箭头语法以最简洁的方式声明内联函数。

## 解决方案

近年来，JavaScript 已经转向强调函数式编程模式——数组处理和异步 Promise 是其中两个显著例子。为了帮助，他们添加了一种新的、简化的函数语法来编写内联函数，称为*箭头语法*。

这是使用`Array.map()`方法对数组内容进行转换的示例，使用了不带箭头语法的命名函数。初始数组是一组数字，转换后的数组是每个数字的平方：

```
const numbers = [1,2,3,4,5,6,7,8,9,10];

function squareNumber(number) {
  return number**2;
}
const squares = numbers.map(squareNumber);

console.log(squares);
// Displays [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

这是相同的示例，但使用箭头语法内联声明了`squareNumber()`函数：

```
const numbers = [1,2,3,4,5,6,7,8,9,10];
const squares = numbers.map( number => number**2 );

console.log(squares);
```

## 讨论

此示例使用最简洁的箭头语法形式。这适用于单参数、单语句函数。其他函数可能无法使用箭头语法的所有简化。为了理解其中的原因，这里逐步解释如何将命名函数转换为使用箭头语法的函数表达式：

1.  首先放置参数列表，然后是`=>`符号。如果没有参数，在`=>`符号之前使用空的括号。

```
(number) =>
```

1.  如果只有一个参数（如此示例），可以删除参数列表周围的括号。

```
number =>
```

1.  将函数的大括号和函数体放在箭头的另一侧。

```
number => {
  return number**2;
}
```

1.  如果只有一个语句，可以删除大括号和`return`关键字。但如果有多个语句，则必须保留大括号和`return`关键字。

```
number => number**2;
```

记住，箭头函数用于声明内联函数，因此你总是会将它传递给一个参数或将其分配给一个表达式中的变量：

```
const myFunc = number => number**2;

const squaredNumber = myFunc(10);
// squaredNumber = 100
```

现在让我们看看如何转换这个稍微复杂的函数：

```
function raiseToPower(number, power) {
  return number**power;
}
```

你可以执行步骤 1、3 和 4，但步骤 2 不适用（因为此函数有两个参数）：

```
const myFunc = (number, power) => number**power;
```

或者，考虑这个更详细的字符串处理函数：

```
function applyTitleCase(inputString) {
  // Split the string into an array of words
  const wordArray = inputString.split(' ');

  // Create a new array that will hold the processed words
  const processedWordArray = [];

  for (const word of wordArray) {
    // Capitalize the first letter of this word
    processedWordArray.push(word[0].toUpperCase() + word.slice(1));
  }

  // Join the words back into a single string
  return processedWordArray.join(' ');
}
```

这里，步骤 1、2 和 3 适用，但步骤 4 不适用。你必须保留大括号和`return`语句不变。

```
const myFunc = inputString => {
  // Split the string into an array of words
  const wordArray = inputString.split(' ');

  // Create a new array that will hold the processed words
  const processedWordArray = [];

  for (const word of wordArray) {
    // Capitalize the first letter of this word
    processedWordArray.push(word[0].toUpperCase() + word.slice(1));
  }

  // Join the words back into a single string
  return processedWordArray.join(' ');
}
```

现在传统方法和箭头语法之间的差异变得更小。只有开头的函数声明发生了变化，总体代码节省量极少。

###### 注意

在这里，围绕箭头语法的决策变得更加模糊。通常可以将几个语句的函数压缩为单个表达式。在字符串处理示例中，您可以使用方法链（如“替换字符串的所有出现”）和`Array.map()`函数（“转换数组的每个元素”）而不是`for`循环。如果应用得当，这些更改可以将`applyTitleCase()`缩短为一个长语句。然后，您可以使用所有箭头语法的快捷方式。然而，在这种情况下，更简洁的代码目标不值得以清晰度为代价。作为一般的经验法则，箭头语法仅在帮助您编写更可读的代码时才有益。

箭头函数有一种不同的绑定`this`关键字的方式。在声明的函数中，`this`映射到调用函数的对象，可以是当前窗口、按钮等。在箭头函数中，`this`简单地指向定义箭头函数的代码。换句话说，无论在何处创建箭头函数时`this`是什么，函数运行时`this`保持不变。这种行为简化了许多问题，但代价是箭头语法不适合对象方法和构造函数，因为箭头函数不会绑定到它们被调用的对象上。即使使用`Function.bind()`也不会改变这种行为。

还有一些较小的限制。箭头函数不能与`yield`一起用于生成器函数，并且不支持`arguments`对象。

## 参见

第五章有许多使用箭头语法将短函数传递给数组处理方法的示例。例如，Recipes，，和。

# 提供默认参数值

## 问题

您希望为参数指定默认值，如果调用函数时未传入参数，则将使用该默认值。

## 解决方案

当您声明一个函数时，可以直接为参数分配默认值。以下是一个为第三个参数`thirdNum`设置默认值的示例：

```
function addNumbers(firstNum, secondNum, thirdNum=0) {
  return firstNum+secondNum+thirdNum;
}
```

现在可以调用此函数而不指定所有三个参数：

```
console.log(addNumbers(42, 6, 10));  // displays 58
console.log(addNumbers(42, 6));      // displays 48
```

## 讨论

默认参数是一个相对较新的发明。然而，JavaScript 从未强制函数调用者为函数提供所有参数。在遥远的过去，函数可以简单地检查参数是否为`undefined`（通过使用`typeof`运算符测试，如“检查对象是否为某一类型”中所述）。

您可以为尽可能多的参数设置默认值。作为良好风格的一部分，应首先放置您需要的参数，然后是具有默认值的参数。换句话说，一旦添加了默认参数，后面的所有参数*都应该*变得可选并具有默认值。这种约定不是必需的，但可以使代码更清晰。

当调用具有多个默认参数的函数时，可以选择提供哪些值。考虑以下示例：

```
function addNumbers(firstNum=10, secondNum=20, thirdNum=30, multiplier=1) {
  return multiplier*(firstNum+secondNum+thirdNum);
}
```

如果要指定`firstNum`、`secondNum`和`multiplier`，但省略`thirdNum`参数，则需要使用`undefined`作为占位符。这使您可以按正确的顺序传递所有参数：

```
const sum = addNumbers(42, 10, undefined, 1);
// sum = 82
```

但是`null`不能作为占位符使用。在这个例子中，它被简单地转换为数字 0，改变了结果：

```
const sum = addNumbers(42, 10, null, 1);
// sum = 52
```

许多其他语言有更好的默认参数快捷方式（例如使用逗号指示顺序而无需提供占位符值，或通过名称设置参数值）。JavaScript 没有这样的功能，尽管您可以使用对象文字语法模拟命名参数（“使用命名函数参数”）。

# 创建一个接受无限参数的函数

## 问题

您希望创建一个函数，该函数接受调用者想要提供的任意数量的参数，而无需创建数组。

## 解决方案

在声明函数时使用*rest 参数*。rest 参数在其名称之前用三个点定义：

```
function sumRounds(...numbers) {
  let sum = 0;
  for(let i = 0; i < numbers.length; i+=1)  {
    sum += Math.round(numbers[i]);
  }
  return sum;
}

console.log(sumRounds(2.3, 4, 5, 16, 18.1));  // 45
```

## 讨论

rest 参数不需要是唯一的参数，但必须是最后一个参数。它收集传递给函数的所有额外参数，并将它们添加到一个新数组中。

在过去，JavaScript 开发者使用`arguments`对象来实现类似的功能。`arguments`对象在每个函数中都可用（技术上来说，它是`Function.arguments`属性），它提供了类似数组的访问所有参数的方式。然而，`arguments`不是真正的数组，开发者经常使用样板代码将其转换为数组。您可能仍然会在实际应用中看到这种方法，但现在使用 rest 参数可以避免这种麻烦。

###### 注意

rest 参数看起来与展开运算符相似（“将数组分解为单独变量”），但两者扮演互补角色。展开运算符*展开*一个数组或对象的属性为单独的值，而 rest 运算符则收集单独的值并将它们插入到一个单一的数组对象中。

## 参见

如果您有一个值数组，想要将其传递给一个函数，但该函数期望一个 rest 参数，您可以使用展开运算符进行转换（参见“将数组分解为单独变量”）。

本例中使用循环来处理值数组，但您可以使用`Array.reduce()`函数更清晰地实现相同的结果，如“将数组的值组合成单一计算”中演示的那样。

# 使用命名函数参数

## 问题

您希望有一种更简单的方法来选择发送到函数的可选参数。

## 解决方案

将所有可选参数捆绑到单个对象字面量中（“使用对象字面量捆绑数据”）。调用者可以决定在创建对象字面量时包含哪些可选参数。这是使用此模式的函数调用示例：

```
someFunction(arg1, arg2, {optionalArg1: val1, optionalArg2: val2});
```

在您的函数中，您可以使用 *解构赋值* 快速地将值从对象字面量中复制到单独的变量中。这是一个接受三个参数的函数的示例。前两个参数（`newerDate` 和 `olderDate`）是必需的，但第三个参数是一个对象字面量，可以包含三个可选值（`discardTime`、`discardYears` 和 `precision`）：

```
function dateDifferenceInSeconds(
 newerDate, olderDate, {discardTime, discardYears, precision} = {}) {
  if (discardTime) {
    newerDate = newerDate.setHours(0,0,0,0);
    olderDate = newerDate.setHours(0,0,0,0);
  }
  if (discardYears) {
    newerDate.setYear(0);
    olderDate.setYear(0);
  }

  const differenceInSeconds = (newerDate.getTime() - olderDate.getTime())/1000;
  return differenceInSeconds.toFixed(precision);
}
```

可以带有或不带有对象字面量调用 `dateDifferenceInSeconds()`：

```
// Compare the current date to an older date
const newDate = new Date();
const oldDate = new Date(2010, 1, 10);

// Call the function without an object literal
let difference = dateDifferenceInSeconds(newDate, oldDate);
console.log(difference);   // Shows something like 354378086

// Call the function with an object literal, and specify two properties
difference = dateDifferenceInSeconds(
 newDate, oldDate, {discardYears:true, precision:2});
console.log(difference);   // Shows something like 7226485.90
```

## 讨论

JavaScript 中的一个常见模式是使用对象字面量传递可选值。这样可以只设置需要的属性，而不必担心顺序。

```
// This works
dateDifferenceInSeconds(newDate, oldDate, {precision:2});

// This also works
dateDifferenceInSeconds(newDate, oldDate, {discardYears:true, precision:2});

// This works too
dateDifferenceInSeconds(newDate, oldDate, {precision:2, discardYears:true});
```

在函数中，您可以像这样单独从对象字面量中检索属性：

```
function dateDifferenceInSeconds(newerDate, olderDate, options) {
  const precision = options.precision;
```

但是本节中的解决方案使用了一个更好的快捷方式。它使用解构将对象字面量解包到命名变量中，这将对象的属性映射到单独的命名变量中。您可以在语句中使用解构赋值：

```
function dateDifferenceInSeconds(newerDate, olderDate, options) {
  const {discardTime, discardYears, precision} = options;
```

或者直接在函数声明中：

```
function dateDifferenceInSeconds(
 newerDate, olderDate, {discardTime, discardYears, precision})
```

将一个空对象字面量设置为默认值是一个很好的做法（“提供默认参数值”）。如果调用者没有提供对象字面量，则使用这个空对象：

```
function dateDifferenceInSeconds(
 newerDate, olderDate, {discardTime, discardYears, precision} `=` `{``}`)
```

调用者可以决定是否设置一些、全部或不设置对象字面量中的属性。未设置的任何值将计算为特殊值 `undefined`，您可以在代码中测试这些值。以下是一个不太优化的示例：

```
  if (discardTime != undefined || discardTime === true) {
```

通常，您不需要显式检查 `undefined` 值。例如，`undefined` 在条件逻辑中求值为 `false`。`dateDifferenceInSeconds()` 函数在评估 `discardYears` 和 `discardTime` 属性时使用这种行为，这使我们可以缩短代码：

```
  if (discardTime) {
```

`precision` 属性有一个类似的快捷方式。安全地调用 `Number.toPrecision(undefined)` 是可以的，因为这与不带参数调用 `toPrecision()` 是一样的。无论哪种方式，数字都会四舍五入到最接近的整数。

对象字面量模式的唯一缺点是无法防止属性命名错误，例如：

```
// We want discardYears, but we accidentally set discardYear
dateDifferenceInSeconds(newDate, oldDate, {discardYear:true});
```

## 另请参阅

“使用对象字面量捆绑数据” 介绍了对象字面量。“将数组分解为单独变量” 展示了数组解构语法，它类似于本节中使用的对象解构语法，只是作用于数组而不是对象（并使用方括号而不是花括号）。

# 使用闭包存储其状态的函数创建

## 问题

您想创建一个可以记住数据但又无需使用全局变量并且不需要在每个函数调用中重复发送相同数据的函数。

## 解决方案

将需要保留其状态的函数包装在*另一个*函数中。外部函数返回内部函数，遵循以下结构：

```
function outerFunction() {

  function innerFunction() {
    ...
  }

  return innerFunction;
}
```

这两个函数都可以接受参数。但这里有一个诀窍。外部函数的参数只要您引用内部函数就会一直存在。您可以随意调用内部函数，外部函数中的数据将持久存在。（在概念上，外部函数就像是一个对象创建方法，而内部函数就像是具有状态的对象。）

这里有一个完整的示例：

```
function greetingMaker(greeting) {
  function addName(name) {
    return `${greeting} ${name}`;
  }
  return addName;
}

// Use the outer function to create two copies of the inner function,
// each with a different value for greeting
const daytimeGreeting = greetingMaker('Good Day to you');
const nightGreeting = greetingMaker('Good Evening');

console.log(daytimeGreeting('Peter'));   // Shows 'Good Day to you Peter'
console.log(nightGreeting('Sally'));     // Shows 'Good Evening Sally'
```

## 讨论

通常，您会发现需要一种方法来存储跨多个函数调用使用的数据。您可以使用全局变量，但这是最后的手段。全局变量会导致命名冲突，使代码复杂化，并经常导致不同函数之间隐藏的相互依赖关系，限制代码的重用，并为隐藏的编码错误提供掩盖。

您可以要求函数调用者维护此信息，并在每个函数调用时发送它，但这样做可能会很尴尬。本示例展示了另一种解决方案——创建一个保持状态的函数包，称为*闭包*。

在此解决方案中，外部函数`greetingMaker()`接受一个参数，即特定的问候语。它还返回一个内部函数`addName()`，该函数本身接受人名。闭包包含`addName()`函数及其周围的上下文，其中包括传递给`greetingMaker()`函数的参数。为了演示这一事实，创建了两个`addName()`的副本，存在于两个不同的上下文中。一个存在于将白天消息传递给`greetingMaker()`的闭包中，另一个存在于将夜间消息传递给`greetingMaker()`的闭包中。无论如何，当调用`addName()`函数时，它都使用当前上下文构造其消息。

值得注意的是，状态不仅限于参数值。任何在外部函数中的变量只要函数引用存在就会保持活动状态。以下是一个示例，其中使用简单的计数器变量来跟踪调用了多少次函数：

```
function createCounter() {
  // This variable persists as long as the createCounter function reference
  let count = 0;

  function counter() {
    count += 1;
    console.log(count);
  }
  return counter;
}

const counterFunction = createCounter();
counterFunction();  // displays 1
counterFunction();  // displays 2
counterFunction();  // displays 3
```

## 参见

要查看使用闭包存储状态的另一个函数示例，请参阅“额外：构建可重复使用的伪随机数生成器”。

不是偶然闭包和包装函数似乎在模仿面向对象编程。过去，JavaScript 开发人员使用函数来模仿自定义类（参见“使用构造函数模式创建自定义类”），而 JavaScript 的 `class` 关键字扩展了这种方法（参见“创建可重用类”）。

# 创建一个生成器函数，可以产生多个值

## 问题

你想创建一个*生成器*，一个能按需提供多个值的函数。每当生成器返回一个值时，它会暂停执行，直到调用者请求下一个值。

## 解决方案

要声明一个生成器函数，首先用`function*`替换`function`关键字：

```
function* generateValues() {
}
```

在生成器函数内部，每当你想要返回一个结果时使用`yield`关键字。记住，执行在你`yield`后停止（就像使用`return`关键字时一样）。然而，当调用者请求函数的下一个值时，执行会*恢复*。这个过程会一直持续，直到你的函数代码结束，或者你使用`return`关键字返回最终值。

这里是一个生成器的简单实现。（它可以工作，但并没有解决一个有用的问题。）这个函数生成三个值，然后返回一个值：

```
function* generateValues() {
  yield 895498;
  yield 'This is the second value';
  yield 5;
  return 'This is the end';
}
```

当你调用一个生成器函数时，你会得到一个`Generator`对象作为返回值。这发生在生成器函数代码开始运行之前立即发生。你可以使用`Generator`对象来运行函数并检索生成的值。你也可以用它来确定生成器函数何时完成。

每当你调用`Generator.next()`时，生成器函数会运行直到达到下一个`yield`（或最终的`return`）。`next()`方法返回一个带有两个值的对象。`value`属性包装了从生成器函数中产生的值或返回的值。`done`属性是一个布尔值，在生成器函数结束前保持为`false`。

```
const generator = generateValues();

// Start the generator (it runs from the beginning to the first yield)
console.log(generator.next().value);  // 895498

// Resume the generator (until the next yield)
console.log(generator.next().value);  // 'This is the second value'

// Get the final two values
console.log(generator.next().value);  // 5
console.log(generator.next().value);  // 'This is the end'
```

## 讨论

生成器允许你创建可以暂停和恢复的函数。最重要的是，JavaScript 会自动管理它们的状态，这意味着你不需要编写任何代码来在调用`next()`之间保存值。（这与构建自定义迭代器不同，例如。）

因为生成器具有延迟执行模型，所以它们非常适合耗时的数据创建或检索操作。例如，你可以使用生成器来计算复杂序列中的数字，或者从数据流中检索信息块。

通常情况下，你不会知道一个生成器会返回多少值。你可以编写一个`while`循环，检查`Generator.done`属性，并不断调用`next()`直到完成。但因为生成器对象是可迭代的，一个`for`…`of`循环效果更好：

```
// Get all the values from the generator
for (const value of generateValues()) {
  console.log(value);
}

// With spread syntax, you can dump everything into an array in one step
const values = [...generateValues()];
```

无论哪种方式，这种方法只能获取*yielded*的结果。如果你的生成器有最终的返回值，它将被忽略。

一些生成器函数设计为*无限*。只要你继续调用`next()`，它们就会继续生成值。如果你调用一个无限生成器，你不能把所有的值都放入数组中（程序会挂起）。相反，你可能会使用一个带有条件的`while`循环，当你得到所需的所有值时条件会变为`false`。

## 参见

“创建异步生成器函数”展示了如何创建异步运行的生成器。

## 额外信息：构建一个可重复的伪随机数生成器

虽然您已经剖析了生成器函数的基本语法，但尚未见过一个真正实用的示例。以下是一个演示无限生成器函数如何提供有用值序列的示例。

如“生成随机数”中所解释的，`Math.random()` 方法允许您生成伪随机数，但您无法控制*种子值*。（相反，`Math.random()` 使用不透明、非加密安全的方法来为其伪随机数生成器种子化，这可能因 JavaScript 实现而异。）这对大多数应用来说都是可以接受的。但在某些情况下，您需要一种方法来生成一个*可重复*的伪随机数序列。这些数字在分布上仍然需要是统计上随机的；唯一的区别在于，您需要能够要求您的伪随机数生成器给出同样的序列超过一次。需要重复的伪随机数的重要示例包括某些需要精确可再现的模拟或测试。

有几个第三方 JavaScript 库提供可种子化（因此可重复）的伪随机数生成器。您可以在[GitHub](https://github.com/bryc/code/blob/master/jshash/PRNGs.md)上找到一个长列表。其中一个最简单的是 Mulberry32。其 JavaScript 实现适合于单个密集的代码块：

```
function mulberry32(seed) {
  return function random() {
    let t = seed += 0x6D2B79F5;
    t = Math.imul(t ^ t >>> 15, t | 1);
    t ^= t + Math.imul(t ^ t >>> 7, t | 61);
    return ((t ^ t >>> 14) >>> 0) / 4294967296;
  }
}

// Choose a seed
const seed = 98345;

// Get a version of mulberry32() that uses this seed:
const randomFunction = mulberry32(seed);

// Generate some random numbers
console.log(randomFunction());  // 0.9057375795673579
console.log(randomFunction());  // 0.44091642647981644
console.log(randomFunction());  // 0.7662326360587031
```

`mulberry32()` 函数使用了在“创建使用闭包记住其状态的函数”中描述的闭包技术。它接受一个种子值，然后将其锁定在内部`random()`函数的上下文中。这意味着无论何时调用`random()`，原始种子值都将在外部函数中可用。这很重要，因为不同的种子意味着不同的随机变量序列。如果您使用相同的种子值调用`mulberry32()`，则保证从`random()`获得相同的伪随机数序列。

###### 注意

像大多数伪随机数生成器一样，Mulberry32 返回一个介于 0 和 1 之间的分数值。要将其转换为给定范围内的整数，请使用“生成随机数”中所示的技术。

自闭包从 JavaScript 语言的太古时代起就存在，但生成器却是一个较新的创新。您可以使用生成器函数重写此示例，更清晰地表达其目的：

```
function* mulberry32(seed) {
  let t = seed += 0x6D2B79F5;

  // Generate numbers indefinitely
  while(true) {
    t = Math.imul(t ^ t >>> 15, t | 1);
    t ^= t + Math.imul(t ^ t >>> 7, t | 61);
    yield ((t ^ t >>> 14) >>> 0) / 4294967296;
  }
}

// Use the same seed to get the same sequence.
const seed = 98345;

const generator = mulberry32(seed);
console.log(generator.next().value);  // 0.9057375795673579
console.log(generator.next().value);  // 0.7620641703251749
console.log(generator.next().value);  // 0.0211441791616380
```

因为`mulberry32()`函数声明为`function*`，所以立即清楚它将返回多个值。在内部，无限循环确保生成器始终准备好创建新的数字。每次通过循环后，`random()`都会产生一个新的随机值，然后暂停，直到使用`next()`请求新值。该解决方案的整体操作与其原始版本类似，但现在遵循一个熟悉的模式，这可能使其使用更容易发现。（但——像往常一样——这种重构的价值取决于您组织的约定、阅读您代码的人的期望以及您个人的品味。）

###### 注

在生成器中构建无限循环并没有危险，只要它们进行 yield 操作。通过 yield 操作暂停代码，确保不会阻塞 JavaScript 事件循环。与普通函数不同，没有期望生成器函数将运行到最终的闭括号。一旦`Generator`对象超出范围，该函数及其上下文将可供垃圾收集。

# 通过使用部分应用程序减少冗余

## 问题

您有一个接受多个参数的函数。您希望用一个或多个特定版本的新函数包装此函数，这些版本需要更少的参数。

## 解决方案

下面的`makestring()`函数接受三个参数（换句话说，它的*arity*为 3）：

```
function makeString(prefix, str, suffix) {
   return prefix + str + suffix;
}
```

但是，第一个和最后一个参数通常基于特定用例重复。您希望尽可能消除参数的重复。

您可以通过创建新的函数来解决这个问题，这些函数封装了先前创建的`makeString()`函数，但已知参数值被锁定：

```
function quoteString(str) {
   return makeString('"',str,'"');
}

function barString(str) {
   return makeString('-', str, '-');
}

function namedEntity(str) {
   return makeString('&#', str, ';');
}
```

现在只需一个参数即可调用这些新函数中的任何一个：

```
console.log(quoteString('apple')); // "apple"
console.log(barString('apple'));   // -apple-
console.log(namedEntity(169));     // "&#169; (the copyright symbol in HTML)
```

## 讨论

将一个函数包装在另一个函数中以锁定一个或多个参数值的技术称为*部分应用程序*（因为新函数*部分应用*到原始函数的参数值）。当然，这样做的权衡是，您创建的额外函数也可能会使您的代码变得混乱，因此不要构建您不打算使用和重用的包装器。

## 高级：部分函数工厂

您甚至可以通过创建一个能够部分化*任何*其他函数的函数进一步减少此方法的冗余。事实上，这种方法是一种相当常见的 JavaScript 设计模式。在过去，您需要依赖 JavaScript 的`arguments`对象和数组操作。在现代 JavaScript 中，剩余和展开运算符使这项工作变得更加简单。

在这里显示的实现中，部分化函数命名为`partial()`。它能够为任何函数减少任意数量的参数。

```
function partial(fn, ...argsToApply) {
  return function(...restArgsToApply) {
    return fn(...argsToApply, ...restArgsToApply);
  }
}
```

这个函数需要一点解包操作。但首先，看一个使用它的简单例子是有帮助的。在这里，`partial()`函数用于创建一个新的`cubeIt()`函数，它包装了更通用的`raiseToPower()`函数。换句话说，`cubeIt()`使用部分应用来锁定`raiseToPower()`的一个参数（指数，它设置为 3）。

```
// The function you want to partialize
function raiseToPower(exponent, number) {
  return number**exponent;
}

// Using partial(), make a customized function
const cubeIt = partial(raiseToPower, 3);

// Calculate the cube of 9 (9**3)
console.log(cubeIt(9));  // 729
```

现在当你调用`cubeIt(9)`时，调用被映射到`raiseToPower(3, 9)`。

那么它是如何工作的呢？`partial()`函数接受两个参数。第一个是你想要部分应用的函数（`fn`）。第二个是你想要锁定在原位的所有参数的列表（`argsToApply`），这些参数使用剩余操作符（`...`）捕获到一个数组中，如在“创建一个接受无限参数的函数”中所解释的那样。

```
function partial(fn, ...argsToApply) {
```

现在事情变得有趣了。`partial`函数返回一个嵌套的内部函数（一种在“创建一个存储其闭包状态的函数”中探讨的技术）。嵌套的内部函数接受所有未锁定在原位的参数。再次强调，这些参数使用剩余操作符（`...restToApply`）捕获到一个数组中：

```
  // This returns a new anonymous function
  return function(...restArgsToApply) {
```

现在，这个新创建的函数有三个关键部分的信息：底层函数（`fn`），锁定在原位的参数（`argsToApply`），以及每次调用函数时设置的参数（`restArgsToApply`）。

函数内部只有一行代码，但包含了很多信息。它使用展开操作符将两个数组展开为参数列表（这有些令人困惑，因为它看起来与剩余操作符完全相同）。换句话说，`argsToApply`变成了一个参数列表，后跟`restToApply`：

```
    // This calls the wrapped function
    return fn(...argsToApply, ...restArgsToApply);
```

###### 注意

函数式编程中的一个常见做法是编写*高阶函数*（操作其他函数的函数）。`partial()`函数是一个创建另一个函数包装器的高级函数。

这个`partial()`函数的实现有一个限制。因为它首先放置固定的参数，所以你无法锁定稍后的参数而不锁定所有先出现的参数。如果你想使用`partial()`为原始解决方案中的`makeString()`函数创建一个包装器，你需要首先重新排列它的参数：

```
function makeString(prefix, suffix, str) {
  return prefix + str + suffix;
}

const namedEntity = partial(makeString, "&#", ";");

console.log(namedEntity(169));
```

## 额外：使用 bind()部分提供参数

您还可以使用`Function.bind()`方法创建部分应用程序。`bind()`方法返回一个新函数，将`this`设置为提供的第一个参数。所有其他参数都被预置到新函数的参数列表中。

现在我们不必使用`partial()`来创建命名实体函数，而是可以使用`bind()`来提供相同的功能，将`undefined`作为第一个参数传递：

```
function makeString(prefix, suffix, str) {
  return prefix + str + suffix;
}

const named = makeString.bind(undefined, "&#", ";");

console.log(named(169)); // "&#169;"
```

现在你有两种好方法来创建使用不同参数的函数的多个版本。

# 修复此功能绑定问题

## 问题

您的函数试图使用关键字`this`，但未绑定到正确的对象。

## 解决方案

使用`Function.bind()`方法来更改函数的上下文和`this`引用的含义：

```
window.onload = function() {
  window.name = 'window';

  const newObject = {
    name: 'object',

    sayGreeting: function() {
      console.log(`Now this is easy, ${this.name}`);

      const nestedGreeting = function(greeting) {
        console.log(`${greeting} ${this.name}`);
        }.bind(this);

      nestedGreeting('hello');
    }
  };

  newObject.sayGreeting();
};
```

## 讨论

关键字`this`指的是函数的所有者或父级。在 JavaScript 中，与`this`相关的挑战是我们无法始终保证哪个父对象将应用于函数。

在解决方案中，对象有一个方法`sayGreeting()`，输出一条消息并将另一个嵌套函数映射到其属性`nestedGreeting`。如果您使用构造函数模式（“使用构造函数模式创建自定义类”）创建类似类的函数对象，您将看到这种方法。

没有`Function.bind()`方法，第一条消息将会说“现在这很容易，对象”，但第二条消息将会说“hello window”。第二条消息有不同名称的原因是因为函数的嵌套使内部函数与周围对象分离，所有*未作用域限定*的函数自动成为`window`对象的属性。

`bind()`方法通过将函数绑定到您选择的对象来解决这个问题。在示例中，`bind()`方法被调用在嵌套函数上，并给出对父对象的引用。现在，当`nestedGreeting()`内部代码使用`this`时，它指向您设置的父对象。

`bind()`方法特别适用于`setTimeout()`和`setInterval()`计时器函数。通常情况下，当这些函数触发您的回调时，`this`引用会丢失（变为`undefined`）。但是通过`bind()`，您可以确保回调函数保持您想要的引用。

示例 6-1 是一个网页，使用`setTimeout()`执行从 10 到 0 的倒计时操作。随着数字的倒数，它们被插入到网页中。此示例还使用构造函数模式进行对象创建（如“使用构造函数模式创建自定义类”中描述的）来创建类似类的`Counter`函数。

##### 示例 6-1\. 展示`bind()`的实用性

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Using Bind with Timers</title>
  </head>
  <body>
    <div id="counterDiv"></div>

    <script>
    // This is the constructor function for the Counter object.
    function Counter(from, to, divElement) {
      this.currentCount = from;
      this.finishCount = to;
      this.element = divElement;

      // The incrementCounter() method updates the page
      this.incrementCounter = function() {
        this.currentCount -= 1;
        this.element.textContent = this.currentCount;

        if (this.currentCount > this.finishCount) {
          // Schedule this function to run again after 1 second.
          setTimeout(this.incrementCounter.bind(this), 1000);
        }
      };

      this.startCounter = function() {
        this.incrementCounter();
      }
    }

    // Create the counter for this page.
    const counter = new Counter(10, 0, document.getElementById('counterDiv'));

    // When the page loads, start the counter.
    window.onload = function() {
      counter.startCounter();
    }
    </script>
  </body>
</html>
```

如果代码示例中的`setTimeout()`函数是以下内容：

```
setTimeout(this.incrementCounter, 1000);
```

它会丢失`this`，回调函数将无法访问像`currentCount`这样的变量，即使`incrementCounter()`方法是同一对象的一部分。

## 额外：self = this

使用`bind()`的一个较旧的替代方法，仍在使用中，是将`this`分配给外部函数中的一个变量，然后内部函数可以访问该变量。通常`this`被分配给一个名为`that`或`self`的变量：

```
window.onload = function() {
  window.name = 'window';

  const newObject = {
    name: 'object',

    sayGreeting: function() {
      const self = this;
      alert('Now this is easy, ' + this.name);
      nestedGreeting = function(greeting) {
        alert(greeting + ' ' + self.name);
      };

      nestedGreeting('hello');
    }
  };

  newObject.sayGreeting('hello');
};
```

没有这个赋值，第二条消息将再次引用“window”，而不是“object”。

# 实现递归算法

## 问题

要实现一个*调用自身*以完成任务的函数，这种技术称为递归。递归在处理分层数据结构（例如节点树或嵌套数组）、某些类型的算法（排序）和一些数学计算（斐波那契数列）时非常有用。

## 解决方案

递归在数学和计算机科学领域都是一个众所周知的概念。在数学中，递归的一个例子是*斐波那契数列*。斐波那契数是前两个斐波那契数的和：

```
f(n)= f(n-1) + f(n-2),
  for n= 2,3,4,...,n and
  f(0) = 0 and f(1) = 1
```

数学递归的另一个例子是*阶乘*，通常用感叹号表示（4!）。阶乘是从 1 到给定数字*n*的所有整数的乘积。如果*n*为 4，则阶乘（4!）为：

```
4! = 4 x 3 x 2 x 1 = 24
```

这些递归可以使用一系列循环和条件在 JavaScript 中编码，但也可以使用功能性递归来编码。这里是一个找到斐波那契数列第 n 个数的递归函数：

```
function fibonacci(n) {
  return n < 2 ? n : fibonacci(n - 1) + fibonacci(n - 2);
}
```

这里有一个解决阶乘的例子：

```
function factorial(n) {
  return n <= 1 ? 1 : n * factorial(n - 1);
}
```

## 讨论

区分递归函数的特征是*终止条件*（也称为*基本情况*）。递归函数不能随意地持续调用自身，因为这会导致无限循环（直到堆栈空间耗尽并导致程序失败）。相反，递归函数检查一个条件，然后决定调用自身（进入递归的下一层）或返回一个值（回到调用函数的一层）。当顶层函数返回一个值时，这个值就成为最终结果，递归操作完成。

在斐波那契的例子中，测试`n`是否小于 2。如果是，则返回它；否则再次调用斐波那契函数，分别传入(`n-1`)和(`n-2`)，并返回两者的和。

在阶乘的例子中，当首次调用函数时，传递的参数值与数字 1 进行比较。如果`n`小于或等于 1（在这个简单的实现中不支持负数），函数将终止并返回 1。然而，如果`n`大于 1，返回的是`n`乘以再次调用`factorial()`函数，这次传入的值是`n-1`。随着函数的每次迭代，`n`的值递减，直到达到终止条件。

当计算阶乘时，每个函数调用的中间值被推送到内存中的堆栈，并保留直到达到终止条件。然后，这些值从内存中弹出并返回，类似于以下状态：

| return 1; | // 0! |
| --- | --- |
| return 1; | // 1! |
| return 1 * 2; | // 2! |
| return 1 * 2 * 3; | // 3! |
| return 1 * 2 * 3 * 4; | // 4! |

大多数递归函数可以用通过某种循环线性执行相同功能的代码来替换。循环可能性能更好，尽管差异通常微不足道。递归的优势在于递归函数可以非常简洁和简洪。它们是否更清晰是一个有争议的问题。（它们显然*更短*，这使得它们更容易消化，但它们的自我引用性质可能使得它们的逻辑一开始就更难理解，特别是对于之前没有使用过递归函数的程序员来说。）

如果一个递归函数一遍又一遍地调用自身，最终会耗尽调用栈。这种情况会导致出现类似“栈空间不足”、“递归过深”或“超出最大调用栈大小”等错误。确切的错误消息和一次允许的开放函数调用数量取决于 JavaScript 引擎的实现。然而，这些错误消息通常表明一个结构不正确的递归函数未能评估其终止条件并在无限循环中调用自身。
