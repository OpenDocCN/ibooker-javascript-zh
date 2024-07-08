# 前言

我对 TypeScript 的旅程并不是一帆风顺或快速的。我在学校主要写 Java，然后是 C++，像许多在静态类型语言上成长的新开发人员一样，我看不起 JavaScript，认为它只是人们随意扔在网站上的懒散脚本语言。

我在这门语言上的第一个重要项目是一个纯 HTML5/CSS/JavaScript 实现的愚蠢的 *超级马里奥兄弟* 游戏的重新制作，就像许多第一个项目一样，一开始完全混乱。在项目开始时，我本能地不喜欢 JavaScript 的奇怪灵活性和缺乏防护。直到项目接近尾声，我才真正开始尊重 JavaScript 的特性和怪癖：语言的灵活性，能够混合和匹配小函数，以及在用户加载页面后几秒钟内*即刻生效*。

在我完成第一个项目时，我已经深深爱上了 JavaScript。

静态分析（分析你的代码而无需运行它的工具），比如 TypeScript，起初让我感到不适。*JavaScript 如此轻松流畅*，我想，*为什么要用严格的结构和类型束缚自己？*我们是不是在回到我曾经离开的 Java 和 C++ 的世界？

回顾我的旧项目时，我花了整整十分钟才勉强读懂我那些古怪混乱的 JavaScript 代码，理解没有静态分析时事情会变得多么混乱。整理那些代码的过程向我展示了所有可能从结构化中受益的地方。从那时起，我便着迷于尽可能向我的项目中添加静态分析。

距我第一次尝试 TypeScript 已经快十年了，我仍然像以前一样享受它。这门语言仍在不断发展，引入新特性，如今比以往任何时候都更有用，为 JavaScript 提供了*安全性*和*结构*。

我希望通过阅读 *学习 TypeScript*，你能像我一样学会欣赏 TypeScript：它不仅仅是找出错误和拼写错误的手段，当然也不是 JavaScript 代码模式的重大变化，而是带有类型的 JavaScript：一个声明我们的 JavaScript 应该如何工作的美丽系统，并帮助我们坚持下去。

# 适合读者

如果你理解如何编写 JavaScript 代码，可以在终端运行基本命令，并有兴趣学习 TypeScript，那么这本书适合你。

也许你听说过 TypeScript 能帮助你用更少的错误写更多的 JavaScript（*属实！*），或者为其他人阅读文档良好地记录你的代码（*同样属实！*）。也许你看到 TypeScript 在许多职位招聘中出现，或者在你即将开始的新角色中。

无论你的理由是什么，只要你已掌握 JavaScript 的基础知识——变量、函数、闭包/作用域和类——这本书将带你从零开始掌握 TypeScript 的基础知识和最重要的特性。通过本书，你将理解：

+   TypeScript 在“纯”JavaScript 之上为何有用的历史和背景

+   类型系统如何对代码建模

+   类型检查器如何分析代码

+   如何使用仅用于开发的类型注解来指导类型系统

+   TypeScript 如何与 IDE（集成开发环境）协作，提供代码探索和重构工具

你将能够：

+   阐述 TypeScript 的好处和其类型系统的一般特性。

+   在代码中添加有用的类型注解。

+   使用 TypeScript 的内置推断和新语法来表示中等复杂类型。

+   使用 TypeScript 协助本地开发进行代码重构。

# 我为什么写这本书

TypeScript 是一个在工业界和开源社区都极为流行的语言：

+   GitHub 的 2021 年和 2020 年 Octoverse 报告显示，它是该平台第四受欢迎的语言，比 2019 年和 2018 年的第七位以及 2017 年的第十位有所上升。

+   StackOverflow 2021 年的开发者调查显示，它是世界上第三受欢迎的语言（72.73% 的用户）。

+   2020 年 JS 调查显示，TypeScript 作为构建工具和 JavaScript 变体，始终保持高度满意度和使用率。

对于前端开发人员来说，TypeScript 在所有主要的 UI 库和框架中都得到了良好支持，包括强烈推荐使用 TypeScript 的 Angular，以及 Gatsby、Next.js、React、Svelte 和 Vue。对于后端开发人员来说，TypeScript 生成的 JavaScript 可以原生运行在 Node.js 中；Deno，Node.js 的创造者创建的类似运行时环境，强调直接支持 TypeScript 文件。

然而，尽管有如此多受欢迎的项目支持，当我第一次学习这门语言时，我对在线缺乏良好的入门内容感到相当失望。许多在线文档源并没有很好地解释“类型系统”是什么以及如何使用它。它们通常假设读者在 JavaScript 和强类型语言方面有大量的先验知识，或者只是提供了粗略的代码示例。

在几年前，没有看到 O'Reilly 出版的一本有趣动物封面介绍 TypeScript 的书让我感到失望。虽然现在包括 O'Reilly 在内的其他出版商出版了许多关于 TypeScript 的书籍，但在这本书之前，我找不到一本专注于这门语言基础的书籍，解释它为何以其特定方式工作以及其核心特性如何协同工作。这本书从语言基础开始解释，逐步添加功能。我很高兴能够为那些对 TypeScript 原理不熟悉的读者清晰、全面地介绍 TypeScript 语言基础。

# 导读本书

*学习 TypeScript* 有两个目的：

+   您可以通过一次阅读了解 TypeScript 的整体情况。

+   稍后，您可以将其作为实际入门 TypeScript 语言的参考书。

这本书分为三个主要部分，从概念到实际运用逐步展开：

+   第一部分，“概念”：JavaScript 的发展历程，TypeScript 在其中的贡献以及 TypeScript 创建的*类型系统*的基础。

+   第二部分，“特性”：展开类型系统如何与您在编写 TypeScript 代码时使用的 JavaScript 的主要部分互动。

+   第三部分，“用法”：现在您已经了解了构成 TypeScript 语言的特性，如何在实际情况中使用它们来提高您的代码阅读和编辑体验。

在末尾添加了一个第四部分，“额外积分”章节，涵盖了较少使用但偶尔仍然有用的 TypeScript 特性。您不需要深入了解它们来自认为自己是 TypeScript 开发者。但它们都是有用的概念，可能会在您实际项目中使用 TypeScript 时遇到。一旦您完成了对前三部分的理解，我强烈建议您学习额外积分部分。

每章以俳句开头，以双关语结尾，以进入其内容的精神。整个 Web 开发社区以及其中的 TypeScript 社区以其欢乐和对新手的欢迎而闻名。我尽量使本书对像我这样不喜欢冗长、枯燥文风的学习者来说愉快阅读。

## 示例和项目

不同于许多其他介绍 TypeScript 的资源，本书有意侧重于通过独立示例展示新信息，而不深入介绍中等或大型项目。我更喜欢这种教学方法，因为它首先突出 TypeScript 语言。TypeScript 在许多框架和平台上都很有用——其中许多框架和平台经常进行 API 更新——我不想在本书中保留任何特定于框架或平台的内容。

话虽如此，在学习编程语言时，立即在介绍后练习概念非常有用。我强烈建议每章结束后休息一下，复习该章节的内容。每章都建议访问其在[*https://learningtypescript.com*](https://learningtypescript.com)上的部分，并完成列出的示例和项目。

# 本书使用的约定

本书使用以下排版约定：

*斜体*

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`固定宽度`

用于程序清单，以及段落内引用程序元素，例如变量或函数名称、数据类型、语句和关键字。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般注意事项。

###### 警告

此元素指示警告或注意事项。

# 使用代码示例

附加材料（代码示例、练习等）可在[*https://learningtypescript.com*](https://learningtypescript.com)下载。

如果您对使用代码示例有技术问题或问题，请发送电子邮件至*bookquestions@oreilly.com*。

本书旨在帮助您完成工作任务。通常情况下，如果本书提供示例代码，您可以在您的程序和文档中使用它。除非您复制了代码的大部分内容，否则无需征得我们的许可。例如，编写一个使用本书多个代码片段的程序不需要许可。销售或分发 O’Reilly 图书中的示例需要许可。引用本书回答问题并引用示例代码不需要许可。将本书大量示例代码整合到您产品的文档中需要许可。

我们感谢，但通常不要求署名。署名通常包括标题、作者、出版商和 ISBN。例如：“*学习 TypeScript* by Josh Goldberg (O’Reilly)。2022 年版权 Josh Goldberg, 978-1-098-11033-8。”

如果您认为您对代码示例的使用超出了合理使用或上述许可，请随时通过*permissions@oreilly.com*与我们联系。

# O’Reilly 在线学习

###### 注意

40 多年来，[*O’Reilly Media*](http://oreilly.com)一直致力于为公司提供技术和业务培训、知识和见解，帮助其成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专长。O’Reilly 的在线学习平台为您提供按需访问的实时培训课程、深度学习路径、交互式编码环境以及来自 O’Reilly 和 200 多家其他出版商的广泛的文本和视频集合。更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请发送有关本书的评论和问题至出版商：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（在美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为本书设有网页，列出勘误、示例和任何额外信息。您可以访问[*https://oreil.ly/learning-typescript*](https://oreil.ly/learning-typescript)。

通过电子邮件*bookquestions@oreilly.com*发表评论或提出有关本书的技术问题。

有关我们的图书和课程的新闻和信息，请访问[*https://oreilly.com*](https://oreilly.com)。

在 LinkedIn 上找到我们：[*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media)。

在 Twitter 上关注我们：[*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia)。

在 YouTube 上关注我们：[*https://www.youtube.com/oreillymedia*](https://www.youtube.com/oreillymedia)。

# 致谢

这本书是团队的努力成果，我要衷心感谢所有让这成为可能的人。首先是我的超人编辑总监 Rita Fernando，在整个创作过程中展现的不可思议的耐心和卓越指导。额外向 O'Reilly 团队的其他成员致敬：Kristen Brown，Suzanne Huston，Clare Jensen，Carol Keller，Elizabeth Kelly，Cheryl Lenser，Elizabeth Oliver 和 Amanda Quinn。你们都太棒了！

特别感谢技术审阅者，他们提供了始终如一的高水平教学见解和 TypeScript 专业知识：Mike Boyle，Ryan Cavanaugh，Sara Gallagher，Michael Hoffman，Adam Reineke 和 Dan Vanderkam。没有你们，这本书将不会如此。希望我成功捕捉到你们所有伟大建议的意图！

特别感谢给予书籍详尽评论以帮助我提升技术准确性和写作质量的各位同行和赞誉者：Robert Blake，Andrew Branch，James Henry，Adam Kaczmarek，Loren Sands-Ramshaw，Nik Stern 和 Lenz Weber-Tronic。每一个建议都非常重要！

最后，我要感谢多年来家人们的爱和支持。感谢我的父母，Frances 和 Mark，以及我的兄弟 Danny — 感谢你们让我有时间玩乐高、读书和玩游戏。感谢我的配偶 Mariah Goldberg，在我长时间编辑和写作的时候给予的耐心，还有我们的猫 Luci，Tiny 和 Jerry，因为它们卓越的毛发和陪伴。
