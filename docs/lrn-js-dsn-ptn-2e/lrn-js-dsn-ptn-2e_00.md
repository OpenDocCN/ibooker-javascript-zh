# 前言

自从我 10 多年前编写第一版《学习 JavaScript 设计模式》以来，JavaScript 世界已经发生了翻天覆地的变化。那时，我正在处理大规模 Web 应用程序，并发现 JavaScript 代码的缺乏结构和组织使得维护和扩展这些应用程序变得困难。

快进到今天，Web 开发的景观发生了巨大变化。JavaScript 已成为世界上最流行的编程语言之一，并被用于从简单脚本到复杂 Web 应用程序的各种用途。JavaScript 语言已经演变，包括了模块、promises 和`async`/`await`，这极大地影响了我们如何设计应用程序架构。开发者编写组件的方式，如 React 中的方式，也显著影响了他们对可维护性的思考。这促使我们需要考虑到这些新变化的现代模式。

随着现代库和框架如 React、Vue 和 Angular 的兴起，开发者现在构建的应用比以往任何时候都更加复杂。我意识到需要更新《学习 JavaScript 设计模式》的版本，以反映 JavaScript 和 Web 应用程序开发中的变化。

在《学习 JavaScript 设计模式》的第二版中，我旨在帮助开发者将现代设计模式应用到他们的 JavaScript 代码和 React 应用程序中。本书涵盖了超过 20 种对构建可维护和可扩展应用程序至关重要的设计模式。本书不仅仅是关于设计模式，还涵盖了渲染和性能模式，这些对现代 Web 应用程序的成功至关重要。

本书的第一版侧重于经典设计模式，如模块模式、观察者模式和中介者模式。这些模式今天仍然重要和相关，但 Web 开发世界在过去的十年中发生了重大变化，出现了新的模式。本新版涵盖了这些新模式，如 promises、`async`/`await`以及模块模式的新变体。我们还涵盖了诸如 MVC、MVP 和 MVVM 等架构模式，并讨论现代框架与这些架构模式的关系。

如今的开发者接触到许多特定于库或框架的设计模式。React 成熟的生态系统及其利用新的 JavaScript 原语提供了一个优秀的平台，来讨论在框架或库上下文中的最佳实践和模式。除了经典设计模式外，本书还涵盖了现代 React 模式，如 Hooks、Higher-Order Components 和 Render Props。这些模式是特定于 React 的，对使用这一流行框架构建现代 Web 应用程序至关重要。

这本书不仅仅是关于设计模式，它还涵盖了最佳实践。我们涉及诸如代码组织、性能和渲染等主题，这些对于构建高质量的 Web 应用程序至关重要。你将学习动态导入、代码分割、服务器端渲染、水合以及 Islands 架构，这些都是构建快速响应的 Web 应用程序所必需的。

通过本书的学习，你将深入了解设计模式及如何将其应用到你的 JavaScript 代码和 React 应用程序中。你还将了解到哪些模式适用于现代 Web，哪些不适用。本书不仅仅是模式的参考，也是构建高质量 Web 应用程序的指南。你将学习如何结构化你的代码以获得最大的可维护性和可扩展性，以及如何优化性能。

# 书的结构

本书共分为 15 章，旨在从现代视角引导你学习 JavaScript 设计模式，融入更新的语言特性和 React 特定模式。每章都建立在前一章的基础上，帮助你逐步扩展知识并有效应用。

+   第一章，“设计模式简介”：熟悉设计模式的历史及其在编程世界中的重要性。

+   第二章，“‘模式’性测试、原型模式和三原则”：理解评估和完善设计模式的过程。

+   第三章，“模式的结构和编写”：学习良好编写模式的结构以及如何创建它们。

+   第四章，“反模式”：了解什么是反模式以及如何在你的代码中避免它们。

+   第五章，“现代 JavaScript 语法和特性”：探索最新的 JavaScript 语言特性及其对设计模式的影响。

+   第六章，“设计模式的分类”：深入探讨设计模式的不同分类：创建型、结构型和行为型。

+   第七章，“JavaScript 设计模式”：学习 JavaScript 中超过 20 种经典设计模式及其现代适应。

+   第八章，“JavaScript MV*模式”：了解像 MVC、MVP 和 MVVM 等架构模式及其在现代 Web 开发中的重要性。

+   第九章，“异步编程模式”：了解 JavaScript 中异步编程的强大之处及处理它的各种模式。

+   第十章，“模块化 JavaScript 设计模式”：发现组织和模块化你的 JavaScript 代码的模式。

+   第十一章，“命名空间模式”：学习各种将你的 JavaScript 代码命名空间化的技术，以避免全局命名空间污染。

+   第十二章，“React.js 设计模式”：探索 React 特定的模式，包括高阶组件、渲染属性和 Hooks。

+   第十三章，“渲染模式”：理解不同的渲染技术，如客户端渲染、服务器端渲染、渐进式水合和 Islands 架构。

+   第十四章，“React.js 应用结构”：学习如何为你的 React 应用程序进行更好的组织、可维护性和可扩展性。

+   第十五章，“总结”：总结本书的关键收获和最终思考。

本书贯穿始终，提供了实际示例来说明所讨论的模式和概念。通过你的学习，你将对 JavaScript 设计模式有扎实的理解，并能编写优雅、可维护和可扩展的代码。

无论你是经验丰富的网页开发者还是初学者，本书都将为你提供构建现代、可维护和可扩展的网页应用所需的知识和工具。我希望这本书能成为你在继续发展技能和构建令人惊叹的网页应用过程中的宝贵资源。

# 本书使用的约定

本书中使用以下排版约定：

*Italic*

表示新术语、网址、电子邮件地址、文件名和文件扩展名。

`Constant width`

用于程序清单，以及在段落内指代变量或函数名、数据库、数据类型、环境变量、语句和关键字等程序元素。

*`Constant width italic`*

显示应该由用户提供值或上下文确定的值替换的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般注释。

# 使用代码示例

补充材料（代码示例、练习等）可在[*https://github.com/addyosmani/learning-jsdp*](https://github.com/addyosmani/learning-jsdp)下载。

如果你有技术问题或使用代码示例遇到问题，请发送电子邮件至*bookquestions@oreilly.com*。

本书旨在帮助您完成工作。一般情况下，如果本书附带示例代码，您可以在您的程序和文档中使用它。除非您重复使用代码的大部分，否则您无需联系我们以获得许可。例如，编写一个使用本书中几个代码片段的程序无需许可。出售或分发 O’Reilly 图书中的示例需要许可。引用本书回答问题并引用示例代码不需要许可。将本书中大量示例代码整合到产品文档中需要许可。

我们感谢您的认可，尽管通常不要求，但是通常包括标题、作者、出版商和 ISBN 的归属。例如：“*学习 JavaScript 设计模式*，第 2 版，作者 Addy Osmani（O’Reilly）。版权所有 2023 年 Adnan Osmani，978-1-098-13987-2。”

如果您认为您使用的代码示例超出了公平使用范围或上述许可，请随时通过*permissions@oreilly.com*与我们联系。

# O’Reilly 在线学习

###### 注意

超过 40 年来，[*O’Reilly Media*](https://oreilly.com) 一直致力于为公司提供技术和商业培训、知识和见解，帮助其成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享其知识和专长。O’Reilly 的在线学习平台为您提供按需访问的实时培训课程、深入学习路径、交互式编码环境，以及来自 O’Reilly 和 200 多个其他出版商的大量文本和视频。有关更多信息，请访问[*https://oreilly.com*](https://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送至出版社：

+   O’Reilly Media，Inc.

+   1005 Gravenstein Highway North

+   Sebastopol，CA 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为本书设有一个网页，列出勘误、示例和任何其他信息。您可以访问[*https://oreil.ly/js_design_patterns_2e*](https://oreil.ly/js_design_patterns_2e)查看此页面。

电子邮件*bookquestions@oreilly.com* 以评论或询问有关本书的技术问题。

有关我们的图书和课程的新闻和信息，请访问[*https://oreilly.com*](https://oreilly.com)。

在 LinkedIn 上关注我们：[*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media)

在 Twitter 上关注我们：[*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia)

在 YouTube 上观看我们：[*https://youtube.com/oreillymedia*](https://youtube.com/oreillymedia)

# 致谢

我要感谢第二版的出色审阅人员，包括 Stoyan Stefanov，Julian Setiawan，Viswesh Ravi Shrimali，Adam Scott 和 Lydia Hallie。

第一版的热情、才华横溢的技术审阅者包括 Nicholas Zakas、Andrée Hansson、Luke Smith、Eric Ferraiuolo、Peter Michaux 和 Alex Sexton。他们——以及社区的其他成员——帮助审阅和改进了这本书，他们为这个项目带来的知识和热情简直令人惊叹。

特别感谢 Leena Sohoni-Kasture 对第二版编辑工作的贡献和反馈。

最后，我要感谢我的美妙妻子 Elle，在我编写这本出版物的过程中给予我的所有支持。
