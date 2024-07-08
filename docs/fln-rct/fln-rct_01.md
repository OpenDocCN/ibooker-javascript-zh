# 前言

本书不适合想要学习如何使用 React 的人。如果你对 React 不熟悉并且正在寻找教程，一个很好的起点是[*react.dev*](https://react.dev/)上的 React 文档。相反，本书适合那些好奇的人：那些对如何使用 React 不太感兴趣，但对*React 如何工作*更感兴趣的人。

在我们在一起的时间里，我们将通过一些 React 概念的探索，并理解它们的基础机制，探索所有这些如何组合在一起，使我们能更有效地使用 React 创建应用程序。在我们追求理解基础机制的过程中，我们将开发必要的思维模型，以便以高度的准确性推理 React 及其生态系统。

本书假定我们对以下声明有了一个满意的理解：浏览器渲染网页。网页是由 CSS 样式化并通过 JavaScript 交互的 HTML 文档。它还假设我们对如何使用 React 有一些了解，并且我们在过去的时间里构建过一两个 React 应用程序。理想情况下，我们的一些 React 应用程序已经投入使用。

我们将从 React 的简介开始，并回顾其历史，回溯到 2013 年它首次作为开源软件发布的时候。从那里开始，我们将探索 React 的核心概念，包括组件模型、虚拟 DOM 和调和过程。我们将深入探讨 JSX 的编译器理论，讨论 fiber，以及深入理解其并发编程模型。这样我们将获得强大的收获，帮助我们更流畅地记忆应该被记忆的内容，并通过像`React.memo`和`useTransition`这样的强大基元推迟渲染工作。

在本书的后半部分，我们将探讨 React 框架：它们解决的问题以及它们解决问题的机制。我们将通过编写我们自己的框架来做到这一点，该框架解决了几乎所有 Web 应用程序中的三个突出问题：服务器渲染、路由和数据获取。

一旦我们自己解决了这些问题，理解框架如何解决它们就变得更加可接近。我们还将深入探讨 React Server Components（RSCs）和服务器动作，理解下一代工具如捆绑器和同构路由的作用。

最后，我们将放大镜头，远离 React，探索诸如 Vue、Solid、Angular、Qwik 等的替代方案。我们将探索信号和细粒度反应性，与 React 的较粗略的响应性模型进行对比。我们还将探讨 React 对信号的响应：Forget 工具链，以及与信号比较时的表现。

这里有很多内容要涉及，所以让我们不再浪费时间。让我们开始吧！

# 本书使用的约定

本书中使用了以下排版约定：

*斜体*

指示新术语、URL、电子邮件地址、文件名和文件扩展名。

`等宽字体`

用于程序清单，以及段落中用来指代程序元素，如变量或函数名、数据库、数据类型、环境变量、语句和关键字。

**`Constant width bold`** 和浅灰色文本

用于印刷版第十章中，用以突出代码块中的差异。

###### 注意

此元素表示一般说明。

# O’Reilly 在线学习

###### 注意

40 多年来，[*O’Reilly Media*](https://oreilly.com) 提供技术和商业培训、知识和见解，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专业知识。O’Reilly 的在线学习平台让您随时访问现场培训课程、深入学习路径、交互式编码环境，以及来自 O’Reilly 和其他 200 多家出版商的大量文本和视频。更多信息，请访问[*https://oreilly.com*](https://oreilly.com)。

# 如何联系我们

有关此书的评论和问题，请联系出版商：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-889-8969（美国或加拿大）

+   707-827-7019（国际或本地）

+   707-829-0104（传真）

+   *support@oreilly.com*

+   [*https://www.oreilly.com/about/contact.html*](https://www.oreilly.com/about/contact.html)

我们有这本书的网页，上面列出了勘误、示例和任何其他信息。您可以访问[*https://oreil.ly/fluent-react*](https://oreil.ly/fluent-react)。

有关我们的书籍和课程的新闻和信息，请访问[*https://oreilly.com*](https://oreilly.com)。

在 LinkedIn 上找到我们：[*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media)

在 Twitter 上关注我们：[*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia)

在 YouTube 上关注我们：[*https://youtube.com/oreillymedia*](https://youtube.com/oreillymedia)

# 致谢

这是我写过的第一本书，我非常感激我并不孤单。您即将阅读的内容是许多杰出人士共同努力的成果。在这里，我们感谢这些人为这段文字作出的贡献。

请不要忽视这些人，因为这些人值得您的关注和感激。让我们从直接帮助我完成这本书的人开始：

+   第一位永远是我的妻子，Lea。我花了很多时间写这本书，常常以牺牲共处和家庭时间为代价。由于我对主题的热爱和希望与大家分享，这本书的编写稍微影响了假期和其他与妻子共度时间的机会。她一直给予我支持和鼓励，我为此心怀感激。

+   Shira Evans，我在这本书的开发编辑。来自 O'Reilly 的 Shira 一直是一位非常愉快的合作伙伴，始终给予支持、鼓励和理解，即使我们因为 React 的新事物不断涌现（如 Forget 和服务器动作）而遇到了一些延迟。Shira 耐心地帮助我们度过了这一切，我对她心怀感激。

+   我亲爱的朋友和兄弟 Kent C. Dodds (*@kentcdodds*) 在这本书以外继续的指导，以及他为这本书写的前言。多年来，Kent 一直是我的亲密朋友和导师，我对他的持续支持和鼓励感激不尽。

+   评审人员。感谢那些与我合作的评审人员在这本书中对细节的出色关注和关心：

    +   Adam Rackis (*@adamrackis*)

    +   Daniel Afonso (*@danieljcafonso*)

    +   Fabien Bernard (*@fabien0102*)

    +   Kent C. Dodds (*@kentcdodds*)

    +   Mark Erikson (*@acemarke*)

    +   Lenz Weber-Tronic (*@phry*)

    +   Rick Hanlon II (*@rickhanlonii*)

    +   Sergeii Kirianov (*@SergiiKirianov*)

    +   Matheus Albuquerque (*@ythecombinator*)

+   Meta 的 React 团队，感谢他们在 React 上持续的工作，不断推动 React 的可能性边界，并通过他们的才华、创造力和工程能力使其使用起来愉悦。特别感谢 Dan Abramov (*@dan_abramov*)，他花时间解释了 React 服务器组件架构中捆绑器的角色，并为 第九章 的大部分内容做出了贡献。

最后，我要感谢你，读者，对这本书的兴趣。我希望你发现它和我写作时一样有益。
