# 第一章：引言

所以，你决定买一本关于 TypeScript 的书。为什么呢？

或许是因为你受够了那些奇怪的`cannot read property blah of` `undefined` JavaScript 错误。或者你听说过 TypeScript 能帮助你的代码更好地扩展，并想看看到底有什么了不起的地方。或者你是 C#开发者，一直在考虑尝试一下这整个 JavaScript 的事情。或者你是函数式编程者，决定是时候把自己的技能提升到更高的水平。或者你的老板对你的代码在生产环境中引起问题感到非常厌烦，所以他们把这本书作为圣诞礼物送给了你（如果我说得不对，请告诉我）。

无论你的理由是什么，你听到的都是真的。TypeScript 是下一代 Web 应用程序、移动应用程序、NodeJS 项目和物联网（IoT）设备的核心语言。它通过检查常见的错误使你的程序更加安全，为你自己和未来的工程师提供文档，使重构变得轻松，并使得，像，你一半的单元测试变得不必要（“什么是单元测试？”）。TypeScript 将使你作为程序员的生产力翻倍，并让你和对面街上那个可爱的咖啡师有个约会。

但在你匆忙穿过马路之前，让我们详细分析一下，首先从这里开始：当我说“更安全”时，我到底是什么意思？当然，我说的是*类型安全*。

这里有一些无效的例子：

+   将一个数字和一个列表相乘

+   当实际需要一个对象列表时，却用一个字符串列表调用一个函数

+   当在一个对象上调用一个方法，而该方法实际上并不存在于该对象上时

+   导入最近移动的模块

有些编程语言试图充分利用这些错误。它们试图弄清楚当你做一些无效操作时，你实际上想做什么，因为嘿，你尽力而为，对吧？拿 JavaScript 来说：

```
3 + []            // Evaluates to the string "3"

let obj = {}
obj.foo           // Evaluates to undefined

function a(b) {
  return b/2
}
a("z")            // Evaluates to NaN
```

注意，JavaScript 不会在尝试显然无效的操作时抛出异常，而是尽力做到最好，尽量避免异常。JavaScript 这样做是有帮助的吗？当然。它能让你快速捕捉到 bug 吗？可能不会。

现在想象一下，如果 JavaScript 抛出更多的异常，而不是悄悄地做到最好。我们可能会得到这样的反馈：

```
3 + []            // Error: Did you really mean to add a number and an array?

let obj = {}
obj.foo           // Error: You forgot to define the property "foo" on obj.

function a(b) {
  return b/2
}
a("z")            // Error: The function "a" expects a number,
                  // but you gave it a string.
```

不要误会我的意思：为我们修复错误是一个很棒的编程语言功能（如果它可以用于更多的东西就好了！）。但对于 JavaScript 来说，这个特性会导致你在编写代码时犯了一个错误，而*发现*这个错误的时间有所延迟。通常情况下，你第一次听说自己的错误是从别人那里听到的。

所以这里有一个问题：JavaScript 究竟在什么时候告诉你你犯了一个错误呢？

Right：当你实际*运行*你的程序时。你的程序可能会在你在浏览器中测试它时运行，或者当用户访问你的网站时运行，或者当你运行单元测试时运行。如果你很有纪律性地编写大量的单元测试和端到端测试，在推送代码之前进行冒烟测试，并在向用户发布之前在内部进行测试，你有希望在用户之前发现你的错误。但如果你没有呢？

这就是 TypeScript 的作用。比 TypeScript 给出有用的错误消息更酷的是*它给你的时间*：TypeScript 在你输入时会在你的文本编辑器中给出错误消息。这意味着你不必依赖单元测试、冒烟测试或同事来捕捉这些问题：TypeScript 会在你编写程序时捕捉并警告你。让我们看看 TypeScript 对我们之前例子的看法：

```
3 + []            // Error TS2365: Operator '+' cannot be applied to types '3'
                  // and 'never[]'.

let obj = {}
obj.foo           // Error TS2339: Property 'foo' does not exist on type '{}'.

function a(b: number) {
  return b / 2
}
a("z")            // Error TS2345: Argument of type '"z"' is not assignable to
                  // parameter of type 'number'.
```

除了消除整类与类型相关的错误之外，这实际上会改变你编写代码的方式。你会发现自己在填写值级别之前，在类型级别上勾勒出程序；^(2)在设计程序时，你会考虑边缘情况，而不是事后的想法；而且你会设计更简单、更快速、更易于理解和维护的程序。

你准备好开始这段旅程了吗？让我们开始吧！

^(1)根据你使用的静态类型语言，“无效”可能意味着一系列不同的情况，从运行时会崩溃的程序到明显荒谬但不会崩溃的事物。

^(2)如果你不确定这里的“类型级别”是什么意思，别担心。我们将在后面的章节中深入讨论它。
