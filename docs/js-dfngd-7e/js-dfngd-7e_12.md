# 第十二章：迭代器和生成器

可迭代对象及其相关的迭代器是 ES6 的一个特性，在本书中我们已经多次见到。数组（包括 TypedArrays）、字符串以及 Set 和 Map 对象都是可迭代的。这意味着这些数据结构的内容可以被迭代——使用`for/of`循环遍历，就像我们在§5.4.4 中看到的那样：

```js
let sum = 0;
for(let i of [1,2,3]) { // Loop once for each of these values
    sum += i;
}
sum   // => 6
```

迭代器也可以与`...`运算符一起使用，将可迭代对象展开或“扩展”到数组初始化程序或函数调用中，就像我们在§7.1.2 中看到的那样：

```js
let chars = [..."abcd"]; // chars == ["a", "b", "c", "d"]
let data = [1, 2, 3, 4, 5];
Math.max(...data)        // => 5
```

迭代器可以与解构赋值一起使用：

```js
let purpleHaze = Uint8Array.of(255, 0, 255, 128);
let [r, g, b, a] = purpleHaze; // a == 128
```

当你迭代 Map 对象时，返回的值是`[key, value]`对，这与`for/of`循环中的解构赋值很好地配合使用：

```js
let m = new Map([["one", 1], ["two", 2]]);
for(let [k,v] of m) console.log(k, v); // Logs 'one 1' and 'two 2'
```

如果你只想迭代键或值而不是键值对，可以使用`keys()`和`values()`方法：

```js
[...m]            // => [["one", 1], ["two", 2]]: default iteration
[...m.entries()]  // => [["one", 1], ["two", 2]]: entries() method is the same
[...m.keys()]     // => ["one", "two"]: keys() method iterates just map keys
[...m.values()]   // => [1, 2]: values() method iterates just map values
```

最后，一些常用于 Array 对象的内置函数和构造函数实际上（在 ES6 及更高版本中）被编写为接受任意迭代器。`Set()`构造函数就是这样一个 API：

```js
// Strings are iterable, so the two sets are the same:
new Set("abc") // => new Set(["a", "b", "c"])
```

本章解释了迭代器的工作原理，并演示了如何创建自己的可迭代数据结构。在解释基本迭代器之后，本章涵盖了生成器，这是 ES6 的一个强大新功能，主要用作一种特别简单的创建迭代器的方法。

# 12.1 迭代器的工作原理

`for/of`循环和展开运算符与可迭代对象无缝配合，但值得理解实际上是如何使迭代工作的。在理解 JavaScript 中的迭代过程时，有三种不同的类型需要理解。首先是*可迭代*对象：这些是可以被迭代的类型，如 Array、Set 和 Map。其次，是执行迭代的*迭代器*对象本身。第三，是保存迭代每一步结果的*迭代结果*对象。

*可迭代*对象是任何具有特殊迭代器方法的对象，该方法返回一个迭代器对象。*迭代器*是任何具有返回迭代结果对象的`next()`方法的对象。而*迭代结果*对象是具有名为`value`和`done`的属性的对象。要迭代可迭代对象，首先调用其迭代器方法以获取一个迭代器对象。然后，重复调用迭代器对象的`next()`方法，直到返回的值的`done`属性设置为`true`为止。关于这一点的棘手之处在于，可迭代对象的迭代器方法没有传统的名称，而是使用符号`Symbol.iterator`作为其名称。因此，对可迭代对象`iterable`进行简单的`for/of`循环也可以以较困难的方式编写，如下所示：

```js
let iterable = [99];
let iterator = iterable[Symbol.iterator]();
for(let result = iterator.next(); !result.done; result = iterator.next()) {
    console.log(result.value)  // result.value == 99
}
```

内置可迭代数据类型的迭代器对象本身也是可迭代的。（也就是说，它有一个名为`Symbol.iterator`的方法，该方法返回自身。）这在以下代码中偶尔会有用，当你想要遍历“部分使用过”的迭代器时：

```js
let list = [1,2,3,4,5];
let iter = list[Symbol.iterator]();
let head = iter.next().value;  // head == 1
let tail = [...iter];          // tail == [2,3,4,5]
```

# 12.2 实现可迭代对象

在 ES6 中，可迭代对象非常有用，因此当它们表示可以被迭代的内容时，你应该考虑使自己的数据类型可迭代。在第 9-2 和第 9-3 示例中展示的 Range 类是可迭代的。这些类使用生成器函数使自己可迭代。我们稍后会介绍生成器，但首先，我们将再次实现 Range 类，使其可迭代而不依赖于生成器。

要使类可迭代，必须实现一个方法，其名称为符号`Symbol.iterator`。该方法必须返回具有`next()`方法的迭代器对象。而`next()`方法必须返回具有`value`属性和/或布尔`done`属性的迭代结果对象。示例 12-1 实现了一个可迭代的 Range 类，并演示了如何创建可迭代、迭代器和迭代结果对象。

##### 示例 12-1\. 一个可迭代的数字范围类

```js
/*
 * A Range object represents a range of numbers {x: from <= x <= to}
 * Range defines a has() method for testing whether a given number is a member
 * of the range. Range is iterable and iterates all integers within the range.
 */
class Range {
    constructor (from, to) {
        this.from = from;
        this.to = to;
    }

    // Make a Range act like a Set of numbers
    has(x) { return typeof x === "number" && this.from <= x && x <= this.to; }

    // Return string representation of the range using set notation
    toString() { return `{ x | ${this.from} ≤ x ≤ ${this.to} }`; }

    // Make a Range iterable by returning an iterator object.
    // Note that the name of this method is a special symbol, not a string.
    [Symbol.iterator]() {
        // Each iterator instance must iterate the range independently of
        // others. So we need a state variable to track our location in the
        // iteration. We start at the first integer >= from.
        let next = Math.ceil(this.from);  // This is the next value we return
        let last = this.to;               // We won't return anything > this
        return {                          // This is the iterator object
            // This next() method is what makes this an iterator object.
            // It must return an iterator result object.
            next() {
                return (next <= last)   // If we haven't returned last value yet
                    ? { value: next++ } // return next value and increment it
                    : { done: true };   // otherwise indicate that we're done.
            },

            // As a convenience, we make the iterator itself iterable.
            [Symbol.iterator]() { return this; }
        };
    }
}

for(let x of new Range(1,10)) console.log(x); // Logs numbers 1 to 10
[...new Range(-2,2)]                          // => [-2, -1, 0, 1, 2]
```

除了使您的类可迭代之外，定义返回可迭代值的函数也非常有用。考虑这些基于迭代的替代方案，用于 JavaScript 数组的`map()`和`filter()`方法：

```js
// Return an iterable object that iterates the result of applying f()
// to each value from the source iterable
function map(iterable, f) {
    let iterator = iterable[Symbol.iterator]();
    return {     // This object is both iterator and iterable
        [Symbol.iterator]() { return this; },
        next() {
            let v = iterator.next();
            if (v.done) {
                return v;
            } else {
                return { value: f(v.value) };
            }
        }
    };
}

// Map a range of integers to their squares and convert to an array
[...map(new Range(1,4), x => x*x)]  // => [1, 4, 9, 16]

// Return an iterable object that filters the specified iterable,
// iterating only those elements for which the predicate returns true
function filter(iterable, predicate) {
    let iterator = iterable[Symbol.iterator]();
    return { // This object is both iterator and iterable
        [Symbol.iterator]() { return this; },
        next() {
            for(;;) {
                let v = iterator.next();
                if (v.done || predicate(v.value)) {
                    return v;
                }
            }
        }
    };
}

// Filter a range so we're left with only even numbers
[...filter(new Range(1,10), x => x % 2 === 0)]  // => [2,4,6,8,10]
```

可迭代对象和迭代器的一个关键特性是它们本质上是惰性的：当需要计算下一个值时，该计算可以推迟到实际需要该值时。例如，假设您有一个非常长的文本字符串，您希望将其标记为以空格分隔的单词。您可以简单地使用字符串的`split()`方法，但如果这样做，那么必须在使用第一个单词之前处理整个字符串。并且您最终会为返回的数组及其中的所有字符串分配大量内存。以下是一个函数，允许您惰性迭代字符串的单词，而无需一次性将它们全部保存在内存中（在 ES2020 中，使用返回迭代器的`matchAll()`方法更容易实现此函数，该方法在 §11.3.2 中描述）：

```js
function words(s) {
    var r = /\s+|$/g;                     // Match one or more spaces or end
    r.lastIndex = s.match(/[^ ]/).index;  // Start matching at first nonspace
    return {                              // Return an iterable iterator object
        [Symbol.iterator]() {             // This makes us iterable
            return this;
        },
        next() {                          // This makes us an iterator
            let start = r.lastIndex;      // Resume where the last match ended
            if (start < s.length) {       // If we're not done
                let match = r.exec(s);    // Match the next word boundary
                if (match) {              // If we found one, return the word
                    return { value: s.substring(start, match.index) };
                }
            }
            return { done: true };        // Otherwise, say that we're done
        }
    };
}

[...words(" abc def  ghi! ")] // => ["abc", "def", "ghi!"]
```

## 12.2.1 “关闭”迭代器：返回方法

想象一个（服务器端）JavaScript 变体的`words()`迭代器，它不是以源字符串作为参数，而是以文件流作为参数，打开文件，从中读取行，并迭代这些行中的单词。在大多数操作系统中，打开文件以从中读取的程序在完成读取后需要记住关闭这些文件，因此这个假设的迭代器将确保在`next()`方法返回其中的最后一个单词后关闭文件。

但迭代器并不总是运行到结束：`for/of`循环可能会被`break`、`return`或异常终止。同样，当迭代器与解构赋值一起使用时，`next()`方法只会被调用足够次数以获取每个指定变量的值。迭代器可能有更多值可以返回，但它们永远不会被请求。

如果我们假设的文件中的单词迭代器从未完全运行到结束，它仍然需要关闭打开的文件。因此，迭代器对象可能会实现一个`return()`方法，与`next()`方法一起使用。如果在`next()`返回具有`done`属性设置为`true`的迭代结果之前迭代停止（通常是因为您通过`break`语句提前离开了`for/of`循环），那么解释器将检查迭代器对象是否具有`return()`方法。如果存在此方法，解释器将以无参数调用它，使迭代器有机会关闭文件，释放内存，并在完成后进行清理。`return()`方法必须返回一个迭代结果对象。对象的属性将被忽略，但返回非对象值是错误的。

`for/of`循环和展开运算符是 JavaScript 的非常有用的特性，因此在创建 API 时，尽可能使用它们是一个好主意。但是，必须使用可迭代对象、其迭代器对象和迭代器的结果对象来处理过程有些复杂。幸运的是，生成器可以极大地简化自定义迭代器的创建，我们将在本章的其余部分中看到。

# 12.3 生成器

*生成器*是一种使用强大的新 ES6 语法定义的迭代器；当要迭代的值不是数据结构的元素，而是计算结果时，它特别有用。

要创建一个生成器，你必须首先定义一个*生成器函数*。生成器函数在语法上类似于普通的 JavaScript 函数，但是用关键字`function*`而不是`function`来定义。（从技术上讲，这不是一个新关键字，只是在关键字`function`之后和函数名之前加上一个`*`。）当你调用一个生成器函数时，它实际上不会执行函数体，而是返回一个生成器对象。这个生成器对象是一个迭代器。调用它的`next()`方法会导致生成器函数的主体从头开始运行（或者从当前位置开始），直到达到一个`yield`语句。`yield`在 ES6 中是新的，类似于`return`语句。`yield`语句的值成为迭代器上`next()`调用返回的值。通过示例可以更清楚地理解这一点：

```js
// A generator function that yields the set of one digit (base-10) primes.
function* oneDigitPrimes() { // Invoking this function does not run the code
    yield 2;                 // but just returns a generator object. Calling
    yield 3;                 // the next() method of that generator runs
    yield 5;                 // the code until a yield statement provides
    yield 7;                 // the return value for the next() method.
}

// When we invoke the generator function, we get a generator
let primes = oneDigitPrimes();

// A generator is an iterator object that iterates the yielded values
primes.next().value          // => 2
primes.next().value          // => 3
primes.next().value          // => 5
primes.next().value          // => 7
primes.next().done           // => true

// Generators have a Symbol.iterator method to make them iterable
primes[Symbol.iterator]()    // => primes

// We can use generators like other iterable types
[...oneDigitPrimes()]        // => [2,3,5,7]
let sum = 0;
for(let prime of oneDigitPrimes()) sum += prime;
sum                          // => 17
```

在这个例子中，我们使用了`function*`语句来定义一个生成器。然而，和普通函数一样，我们也可以以表达式形式定义生成器。再次强调，我们只需在`function`关键字后面加上一个星号：

```js
const seq = function*(from,to) {
    for(let i = from; i <= to; i++) yield i;
};
[...seq(3,5)]  // => [3, 4, 5]
```

在类和对象字面量中，我们可以使用简写符号来完全省略定义方法时的`function`关键字。在这种情况下定义生成器，我们只需在方法名之前使用一个星号，而不是使用`function`关键字：

```js
let o = {
    x: 1, y: 2, z: 3,
    // A generator that yields each of the keys of this object
    *g() {
        for(let key of Object.keys(this)) {
            yield key;
        }
    }
};
[...o.g()] // => ["x", "y", "z", "g"]
```

请注意，没有办法使用箭头函数语法编写生成器函数。

生成器通常使得定义可迭代类变得特别容易。我们可以用一个更简短的`*Symbol.iterator&rbrack;()`生成器函数来替换[示例 12-1 中展示的`[Symbol.iterator]()`方法，代码如下：

```js
*[Symbol.iterator]() {
    for(let x = Math.ceil(this.from); x <= this.to; x++) yield x;
}
```

查看第九章中的示例 9-3 以查看上下文中基于生成器的迭代器函数。

## 12.3.1 生成器示例

如果生成器实际上*生成*它们通过进行某种计算来产生的值，那么生成器就更有趣了。例如，这里是一个产生斐波那契数的生成器函数：

```js
function* fibonacciSequence() {
    let x = 0, y = 1;
    for(;;) {
        yield y;
        [x, y] = [y, x+y];  // Note: destructuring assignment
    }
}
```

注意，这里的`fibonacciSequence()`生成器函数有一个无限循环，并且永远产生值而不返回。如果这个生成器与`...`扩展运算符一起使用，它将循环直到内存耗尽并且程序崩溃。然而，经过谨慎处理，可以在`for/of`循环中使用它：

```js
// Return the nth Fibonacci number
function fibonacci(n) {
    for(let f of fibonacciSequence()) {
        if (n-- <= 0) return f;
    }
}
fibonacci(20)   // => 10946
```

这种无限生成器与这样的`take()`生成器结合使用更有用：

```js
// Yield the first n elements of the specified iterable object
function* take(n, iterable) {
    let it = iterable[Symbol.iterator](); // Get iterator for iterable object
    while(n-- > 0) {           // Loop n times:
        let next = it.next();  // Get the next item from the iterator.
        if (next.done) return; // If there are no more values, return early
        else yield next.value; // otherwise, yield the value
    }
}

// An array of the first 5 Fibonacci numbers
[...take(5, fibonacciSequence())]  // => [1, 1, 2, 3, 5]
```

这里是另一个有用的生成器函数，它交错多个可迭代对象的元素：

```js
// Given an array of iterables, yield their elements in interleaved order.
function* zip(...iterables) {
    // Get an iterator for each iterable
    let iterators = iterables.map(i => i[Symbol.iterator]());
    let index = 0;
    while(iterators.length > 0) {       // While there are still some iterators
        if (index >= iterators.length) {    // If we reached the last iterator
            index = 0;                      // go back to the first one.
        }
        let item = iterators[index].next(); // Get next item from next iterator.
        if (item.done) {                    // If that iterator is done
            iterators.splice(index, 1);     // then remove it from the array.
        }
        else {                              // Otherwise,
            yield item.value;               // yield the iterated value
            index++;                        // and move on to the next iterator.
        }
    }
}

// Interleave three iterable objects
[...zip(oneDigitPrimes(),"ab",[0])]     // => [2,"a",0,3,"b",5,7]
```

## 12.3.2 `yield*` 和递归生成器

除了在前面的示例中定义的`zip()`生成器之外，可能还有一个类似的生成器函数很有用，它按顺序而不是交错地产生多个可迭代对象的元素。我们可以这样编写这个生成器：

```js
function* sequence(...iterables) {
    for(let iterable of iterables) {
        for(let item of iterable) {
            yield item;
        }
    }
}

[...sequence("abc",oneDigitPrimes())]  // => ["a","b","c",2,3,5,7]
```

在生成器函数中产生其他可迭代对象的元素的过程在生成器函数中是很常见的，ES6 为此提供了特殊的语法。`yield*`关键字类似于`yield`，不同之处在于，它不是产生单个值，而是迭代一个可迭代对象并产生每个结果值。我们使用的`sequence()`生成器函数可以用`yield*`简化如下：

```js
function* sequence(...iterables) {
    for(let iterable of iterables) {
        yield* iterable;
    }
}

[...sequence("abc",oneDigitPrimes())]  // => ["a","b","c",2,3,5,7]
```

数组的`forEach()`方法通常是遍历数组元素的一种优雅方式，因此你可能会尝试像这样编写`sequence()`函数：

```js
function* sequence(...iterables) {
    iterables.forEach(iterable => yield* iterable );  // Error
}
```

然而，这是行不通的。`yield`和`yield*`只能在生成器函数内部使用，但是这段代码中的嵌套箭头函数是一个普通函数，而不是`function*`生成器函数，因此不允许使用`yield`。

`yield*`可以与任何类型的可迭代对象一起使用，包括使用生成器实现的可迭代对象。这意味着`yield*`允许我们定义递归生成器，你可以使用这个特性来允许对递归定义的树结构进行简单的非递归迭代，例如。

# 12.4 高级生成器功能

生成器函数最常见的用途是创建迭代器，但生成器的基本特性是允许我们暂停计算，产生中间结果，然后稍后恢复计算。这意味着生成器具有超出迭代器的功能，并且我们将在以下部分探讨这些功能。

## 12.4.1 生成器函数的返回值

到目前为止，我们看到的生成器函数没有`return`语句，或者如果有的话，它们被用来导致早期返回，而不是返回一个值。不过，与任何函数一样，生成器函数可以返回一个值。为了理解在这种情况下会发生什么，回想一下迭代的工作原理。`next()`函数的返回值是一个具有`value`属性和/或`done`属性的对象。对于典型的迭代器和生成器，如果`value`属性被定义，则`done`属性未定义或为`false`。如果`done`为`true`，则`value`为未定义。但是对于返回值的生成器，最后一次调用`next`会返回一个同时定义了`value`和`done`的对象。`value`属性保存生成器函数的返回值，`done`属性为`true`，表示没有更多的值可迭代。这个最终值被`for/of`循环和展开运算符忽略，但对于手动使用显式调用`next()`的代码是可用的：

```js
function *oneAndDone() {
    yield 1;
    return "done";
}

// The return value does not appear in normal iteration.
[...oneAndDone()]   // => [1]

// But it is available if you explicitly call next()
let generator = oneAndDone();
generator.next()           // => { value: 1, done: false}
generator.next()           // => { value: "done", done: true }
// If the generator is already done, the return value is not returned again
generator.next()           // => { value: undefined, done: true }
```

## 12.4.2 yield 表达式的值

在前面的讨论中，我们将`yield`视为接受值但没有自身值的语句。实际上，`yield`是一个表达式，它可以有一个值。

当调用生成器的`next()`方法时，生成器函数运行直到达到`yield`表达式。`yield`关键字后面的表达式被评估，该值成为`next()`调用的返回值。此时，生成器函数在评估`yield`表达式的过程中停止执行。下次调用生成器的`next()`方法时，传递给`next()`的参数成为暂停的`yield`表达式的值。因此，生成器通过`yield`向其调用者返回值，调用者通过`next()`向生成器传递值。生成器和调用者是两个独立的执行流，来回传递值（和控制）。以下代码示例：

```js
function* smallNumbers() {
    console.log("next() invoked the first time; argument discarded");
    let y1 = yield 1;    // y1 == "b"
    console.log("next() invoked a second time with argument", y1);
    let y2 = yield 2;    // y2 == "c"
    console.log("next() invoked a third time with argument", y2);
    let y3 = yield 3;    // y3 == "d"
    console.log("next() invoked a fourth time with argument", y3);
    return 4;
}

let g = smallNumbers();
console.log("generator created; no code runs yet");
let n1 = g.next("a");   // n1.value == 1
console.log("generator yielded", n1.value);
let n2 = g.next("b");   // n2.value == 2
console.log("generator yielded", n2.value);
let n3 = g.next("c");   // n3.value == 3
console.log("generator yielded", n3.value);
let n4 = g.next("d");   // n4 == { value: 4, done: true }
console.log("generator returned", n4.value);
```

当运行这段代码时，会产生以下输出，展示了两个代码块之间的来回交互：

```js
generator created; no code runs yet
next() invoked the first time; argument discarded
generator yielded 1
next() invoked a second time with argument b
generator yielded 2
next() invoked a third time with argument c
generator yielded 3
next() invoked a fourth time with argument d
generator returned 4
```

注意这段代码中的不对称性。第一次调用`next()`启动了生成器，但传递给该调用的值对生成器不可访问。

## 12.4.3 生成器的 return()和 throw()方法

我们已经看到可以接收生成器函数产生的值。您可以通过在调用生成器的`next()`方法时传递这些值来向正在运行的生成器传递值。

除了使用`next()`向生成器提供输入外，还可以通过调用其`return()`和`throw()`方法来更改生成器内部的控制流。如其名称所示，调用这些方法会导致生成器返回一个值或抛出异常，就好像生成器中的下一条语句是`return`或`throw`一样。

在本章的前面提到，如果迭代器定义了一个`return()`方法并且迭代提前停止，那么解释器会自动调用`return()`方法，以便让迭代器有机会关闭文件或进行其他清理工作。对于生成器来说，你不能定义一个自定义的`return()`方法来处理清理工作，但你可以结构化生成器代码以使用`try/finally`语句，在生成器返回时确保必要的清理工作已完成（在`finally`块中）。通过强制生成器返回，生成器的内置`return()`方法确保在生成器不再使用时运行清理代码。

就像生成器的`next()`方法允许我们向正在运行的生成器传递任意值一样，生成器的`throw()`方法给了我们一种向生成器发送任意信号（以异常的形式）的方法。调用`throw()`方法总是在生成器内部引发异常。但如果生成器函数编写了适当的异常处理代码，异常不必是致命的，而可以是改变生成器行为的手段。例如，想象一个计数器生成器，产生一个不断增加的整数序列。这可以被编写成使用`throw()`发送的异常将计数器重置为零。

当生成器使用`yield*`从其他可迭代对象中产生值时，那么对生成器的`next()`方法的调用会导致对可迭代对象的`next()`方法的调用。`return()`和`throw()`方法也是如此。如果生成器在可迭代对象上使用`yield*`，那么在生成器上调用`return()`或`throw()`会导致依次调用迭代器的`return()`或`throw()`方法。所有迭代器*必须*有一个`next()`方法。需要在不完整迭代后进行清理的迭代器*应该*定义一个`return()`方法。任何迭代器*可以*定义一个`throw()`方法，尽管我不知道任何实际原因这样做。

## 12.4.4 关于生成器的最后说明

生成器是一种非常强大的通用控制结构。它们使我们能够使用`yield`暂停计算，并在任意后续时间点以任意输入值重新启动。可以使用生成器在单线程 JavaScript 代码中创建一种协作线程系统。也可以使用生成器掩盖程序中的异步部分，使你的代码看起来是顺序和同步的，尽管你的一些函数调用实际上是异步的并依赖于网络事件。

尝试用生成器做这些事情会导致代码难以理解或解释。然而，已经做到了，唯一真正实用的用例是管理异步代码。然而，JavaScript 现在有`async`和`await`关键字（见第十三章）用于这个目的，因此不再有任何滥用生成器的理由。

# 12.5 总结

在本章中，你学到了：

+   `for/of`循环和`...`扩展运算符适用于可迭代对象。

+   如果一个对象有一个名为`[Symbol.iterator]`的方法返回一个迭代器对象，那么它就是可迭代的。

+   迭代器对象有一个`next()`方法返回一个迭代结果对象。

+   迭代结果对象有一个`value`属性，保存下一个迭代的值（如果有的话）。如果迭代已完成，则结果对象必须将`done`属性设置为`true`。

+   你可以通过定义一个`[Symbol.iterator]()`方法返回一个具有`next()`方法返回迭代结果对象的对象来实现自己的可迭代对象。你也可以实现接受迭代器参数并返回迭代器值的函数。

+   生成器函数（使用`function*`而不是`function`定义的函数）是定义迭代器的另一种方式。

+   当调用生成器函数时，函数体不会立即运行；相反，返回值是一个可迭代的迭代器对象。每次调用迭代器的`next()`方法时，生成器函数的另一个块会运行。

+   生成器函数可以使用`yield`运算符指定迭代器返回的值。每次调用`next()`都会导致生成器函数运行到下一个`yield`表达式。该`yield`表达式的值然后成为迭代器返回的值。当没有更多的`yield`表达式时，生成器函数返回，迭代完成。
