# 第二章：“Pattern”-ity 测试、原型模式和三规则

从提出一个新模式的那一刻起，到其可能被广泛采纳，一个模式可能需要经历设计社区和软件开发者的多轮深入检查。本章讨论了一个新引入的“原型模式”通过“模式”-ity 测试的旅程，直到它最终被认可为一个模式，如果它符合*三规则*。

这一章和下一章探讨了组织、撰写、呈现和审查新兴设计模式的方法。如果您更愿意先学习已确立的设计模式，可以暂时跳过这两章。

# 什么是原型模式？

请记住，并非每个算法、最佳实践或解决方案都代表了可能被视为完整模式的内容。可能有一些关键要素缺失，而模式社区通常对声称是模式的东西持谨慎态度，除非经过他人的广泛和批判性评估和测试，即使某些东西*看起来*符合模式的标准，我们也不应将其视为模式。

再次回顾亚历山大的工作，他认为一个模式应该既是一个过程，也是一个“东西”。这个定义很模糊，因为他接着说，应该是过程创造了“东西”。这就是为什么模式通常专注于解决一个可视化可识别的结构问题；我们应该能够通过将模式放入实践中来视觉地描述（或绘制）代表结构的图像。

# “Pattern” 测试

当研究设计模式时，你可能经常会遇到术语“原型模式”。这是什么？嗯，一个尚未最终通过“模式”-ity 测试的模式通常被称为原型模式。原型模式可能源自于已经建立了值得与社区分享的特定解决方案的人的工作。然而，由于其相对年轻，社区尚未有机会适当地审核提出的解决方案。

或者，分享模式的个人可能没有时间或兴趣参与“模式”-ity 过程，可能会发布其原型模式的简短描述。这种类型的简短描述或片段被称为*patlets*。

全面记录合格模式所涉及的工作可能相当令人畏惧。回顾设计模式领域最早期的一些工作，如果一个模式能够做到以下几点，则可以认为它是“好”的：

解决特定问题

模式不仅仅是捕捉原则或策略。它们需要捕捉解决方案。这是一个好模式的最重要因素之一。

没有明显解决方案

我们可以发现，解决问题的技术通常试图从众所周知的第一原则中推导出来。最好的设计模式通常间接地为问题提供解决方案——这被认为是解决与设计相关的最具挑战性问题的必要方法。

描述了一个经过验证的概念

设计模式需要证明其按描述的方式运作，没有这种证明，设计就不能被认真考虑。如果一个模式在本质上是高度推测性的，只有勇敢的人才会尝试使用它。

描述了一种关系

在某些情况下，可能会出现一个模式描述了一种模块类型的情况。尽管实现看起来如何，但模式的官方描述必须描述更深层次的系统结构和机制，以解释它与代码的关系。

我们可能会原谅地认为，不符合指导方针的原型模式不值得学习；然而，这与事实相去甚远。许多原型模式实际上相当不错。我并不是说所有原型模式都值得一看，但在实践中确实有一些有用的原型模式可以帮助我们未来的项目。在考虑以上清单时，请慎重判断，你在选择过程中会没问题的。

# 三原则

模式有效的另一个要求是它展示了一些重复出现的现象。你通常可以在至少三个关键领域中确认这一点，称为三原则。为了使用这一原则展示重复性，必须展示以下内容：

适用性

这个模式被认为成功的原因是什么？

实用性

为什么这个模式被认为成功？

适用性

设计是否值得成为模式，因为它具有更广泛的适用性？如果是这样，这需要解释。在审查或定义模式时，牢记这些领域是至关重要的。

# 总结

本章展示了每个提出的原型模式并不总是被接受为模式。下一章分享了构建和记录模式的基本要素和最佳实践，以便社区能够轻松理解和消化它们。