# 前言

JavaScript 框架在现代 Web 前端开发中扮演着重要角色。在开发 Web 项目时，公司会出于多种原因选择框架，包括最终产品的质量、开发成本、编码标准和开发便利性。因此，学习使用 JavaScript 框架（如 Vue）对于任何现代 Web 开发者（或前端开发者或全栈开发者）都是至关重要的。

本书适用于希望使用 Vue 库从头到尾开发 Web 应用程序的程序员，使用 JavaScript 和 TypeScript。它专注于 Vue 及其生态系统如何帮助您在最简单和最舒适的方向上构建可伸缩和交互式 Web 应用程序。在介绍基础知识的同时，我们还将深入 Vue Router 和 Pinia 用于状态管理、测试、动画、部署和服务器端渲染，确保您可以立即开始开发复杂的 Vue 项目。

如果您对 Vue 或虚拟 DOM 的概念不熟悉也没关系。本书不假设您具备 Vue 或任何类似框架的任何先验知识。我将从零开始介绍和引导您了解 Vue 的基础知识。在第二章中，我还将为您讲解 Vue 中的虚拟 DOM 概念和响应系统，作为本书其余部分的基础。

本书不要求您了解 TypeScript，但如果您熟悉 TypeScript 基础，则会更有准备。如果您事先掌握了 HTML、CSS 和 JavaScript 的基础知识，那么您也将更好地准备好阅读本书的内容。在深入任何网络（或前端）JavaScript 框架之前，这三者的扎实基础始终至关重要。

# 本书使用的约定

本书使用以下排版约定：

*斜体*

指示新术语、网址、电子邮件地址、文件名和文件扩展名。

`常宽`

用于程序清单，以及在段落内用于引用程序元素，如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

**`常宽粗体`**

显示用户应直接输入的命令或其他文本。

*`常宽斜体`*

显示应由用户提供的值或由上下文确定的值替换的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般性说明。

###### 警告

此元素指示警告或注意事项。

# 使用代码示例

补充材料（代码示例、练习等）可在[*https://github.com/mayashavin/learning-vue-app*](https://github.com/mayashavin/learning-vue-app)下载。

如果您有技术问题或在使用代码示例时遇到问题，请发送电子邮件至*bookquestions@oreilly.com*。

本书旨在帮助您完成工作。一般来说，如果本书提供示例代码，您可以在程序和文档中使用它。除非您复制了代码的大部分，否则无需获得我们的许可。例如，编写一个程序使用本书中几个代码块不需要许可。出售或分发奥莱利书籍中的示例需要许可。引用本书并引用示例代码回答问题不需要许可。将本书的大量示例代码整合到您产品的文档中需要许可。

我们感谢，但通常不要求署名。署名通常包括标题、作者、出版商和 ISBN。例如：“*Learning Vue* by Maya Shavin (O’Reilly). Copyright 2024 Maya Shavin, 978-1-492-09882-9.”

如果您认为您使用的代码示例超出了合理使用范围或上述许可，请随时与我们联系，邮件至 *permissions@oreilly.com*。

# 奥莱利在线学习

###### 注意

超过 40 年来，[*奥莱利媒体*](http://oreilly.com) 提供技术和商业培训，知识和洞察力，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专业知识。奥莱利的在线学习平台为您提供按需访问的实时培训课程、深入学习路径、交互式编码环境以及来自奥莱利和其他 200 多个出版商的广泛文本和视频资源。更多信息，请访问 [*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送至出版社：

+   奥莱利媒体，公司

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-889-8969（美国或加拿大）

+   707-829-7019（国际或本地）

+   707-829-0104（传真）

+   *support@oreilly.com*

+   [*https://www.oreilly.com/about/contact.html*](https://www.oreilly.com/about/contact.html)

我们为本书设有网页，列出勘误、示例和任何额外信息。您可以访问此网页 [*https://oreil.ly/learning-vue-1e*](https://oreil.ly/learning-vue-1e)。

欲了解有关我们的图书和课程的新闻和信息，请访问 [*https://oreilly.com*](https://oreilly.com)。

在 LinkedIn 上找到我们：[*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media)

在 Twitter 上关注我们：[*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia)

在 YouTube 上观看我们：[*https://youtube.com/oreillymedia*](https://youtube.com/oreillymedia)

# 致谢

在撰写这本书的旅程中，我的家庭正经历着动荡不安的时期，充满高低起伏。尽管享受每一个时刻，但写作这本书需要大量的时间、精力和奉献精神。没有家人的支持，特别是我的丈夫 Natan 的支持，我无法全身心投入其中。他对我的编程能力的鼓励和信任，在前端开发方面的幽默感，以及在我工作出差期间帮助照顾孩子、倾听我的日常怨言、帮助我平衡工作和个人生活的支持，都是无价的。没有 Natan，我今天不会有现在的成就。

就像优质的代码需要彻底审查一样，这本书的卓越之处很大程度上归功于 Jakub Andrzejewski、Chris Fritz、Lipi Patnaik、Edward Wong 和 Vishwesh Ravi Shrimali 的关键技术见解和鼓励。你们宝贵的反馈在提升我的专注力和提高这部作品的质量方面起到了关键作用。

我由衷感谢我的 O'Reilly 团队：Zan McQuade 和 Amanda Quinn，在*Learning Vue*的收购过程中给予我的指导；还有我出色的编辑 Michele Cronin。Michele，你在书稿最后阶段特别是在挑战性阶段期间提供的深刻反馈、专业精神和同理心是非凡的。Ashley Stussy 的制作编辑技能和 Beth Richards 的文稿编辑专业技能对提升我的手稿质量至关重要。没有你们的共同努力，这本书不会如期望般实现。

特别感谢 Vue 核心团队开发了如此优秀的框架和生态系统，以及 Vue 社区成员和朋友们的支持和启发。我从你们那里获得的知识和见解是无法估量的，且每天都在丰富着我。

最后，我对你们读者表达深深的感激之情。从众多资源中选择这本书，包括无数的视频和教程，展示了你们对我的工作的信任，我深表感激。我希望*Learning Vue*能成为你们在 Web、前端或全栈开发之旅中的宝贵工具。

由衷感谢你们。请记住，在 Web 开发的世界里，永远要“用 Vue 来反应”。
