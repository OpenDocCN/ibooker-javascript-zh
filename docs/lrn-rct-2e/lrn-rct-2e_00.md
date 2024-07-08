# 前言

本书适用于希望学习 React 库同时了解当前 JavaScript 语言新技术的开发者。现在是成为 JavaScript 开发者的激动人心时刻。生态系统正在涌现出许多新工具、语法和最佳实践，承诺解决我们的许多开发问题。我们的目标是通过本书组织这些技术，让你可以立即开始使用 React。我们将介绍状态管理、React Router、测试和服务器渲染，因此承诺不仅介绍基础知识然后让你自己摸索。

本书不假设读者对 React 有任何了解。我们将从零开始介绍 React 的所有基础知识。同样，我们不会假设你已经熟悉最新的 JavaScript 语法。这将在第二章中作为后续章节的基础介绍。

如果你对 HTML、CSS 和 JavaScript 感到熟悉，你将更好地准备好本书的内容。在深入学习 JavaScript 库之前，熟悉这三大要素通常是最佳选择。

在学习过程中，请查看[GitHub 仓库](http://github.com/moonhighway/learning-react)。所有示例都在那里，可以让你进行实际操作练习。

# 本书使用的约定

本书中使用以下排版约定：

*斜体*

表示新术语、网址、电子邮件地址、文件名和文件扩展名。

`等宽字体`

用于程序列表，以及在段落内引用程序元素，如变量或函数名、数据库、数据类型、环境变量、语句和关键字。

**`等宽字体粗体`**

显示用户应该按字面输入的命令或其他文本。

###### 提示

这个元素表示提示或建议。

###### 注释

这个元素表示一般注释。

###### 警告

这个元素表示警告或注意事项。

# 使用代码示例

可以下载补充材料（代码示例、练习等）[*https://github.com/moonhighway/learning-react*](https://github.com/moonhighway/learning-react)。

如果你有技术问题或在使用代码示例时遇到问题，请发送电子邮件至*bookquestions@oreilly.com*。

本书旨在帮助你完成工作任务。一般而言，如果本书提供了示例代码，你可以在你的程序和文档中使用它。除非你要复制本书的大部分代码，否则不需要联系我们寻求许可。例如，编写一个使用本书多个代码片段的程序不需要许可。销售或分发 O’Reilly 图书示例需要许可。通过引用本书回答问题并引用示例代码不需要许可。将本书大量示例代码整合到产品文档中需要许可。

我们感激，但通常不要求署名。署名通常包括标题、作者、出版商和 ISBN 号。例如：“*学习 React* by Alex Banks and Eve Porcello (O’Reilly). Copyright 2020 Alex Banks and Eve Porcello, 978-1-492-05172-5.”

如果您认为您对代码示例的使用超出了合理使用范围或上述许可，请随时通过*permissions@oreilly.com*联系我们。

# 奥莱利在线学习

###### 注意

超过 40 年来，[*奥莱利传媒*](http://oreilly.com) 一直为企业提供技术和商业培训、知识和洞察，帮助其成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专长。奥莱利的在线学习平台为您提供按需访问直播培训课程、深入学习路径、交互式编码环境以及来自奥莱利和其他 200 多个出版商的大量文本和视频。有关更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送至出版商：

+   奥莱利传媒有限公司

+   1005 Gravenstein Highway North

+   CA 95472 Sebastopol

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为这本书设有网页，列出勘误、示例和任何额外信息。您可以访问[*https://oreil.ly/learningReact_2e*](https://oreil.ly/learningReact_2e)。

电子邮件*bookquestions@oreilly.com*评论或询问有关本书的技术问题。

有关我们的书籍和课程的新闻和信息，请访问[*http://oreilly.com*](http://oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

在 YouTube 上观看我们：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)

# 致谢

如果没有一些老式的幸运，我们的 React 之旅不会开始。在我们在雅虎内部创建全栈 JavaScript 程序的培训材料时，我们使用了 YUI。然后在 2014 年 8 月，YUI 的开发结束了。我们不得不更改所有课程文件，但是改用什么？现在前端应该使用什么？答案是：React。我们并不是立即爱上 React；我们花了几个小时才被它迷住。看起来 React 可能会彻底改变一切。我们很早就加入并非常幸运。

我们感谢 Angela Rufino 和 Jennifer Pollock 在开发第二版过程中提供的所有支持。我们还要感谢 Ally MacDonald 在第一版中的所有编辑帮助。我们感激我们的技术审阅员 Scott Iwako, Adam Rackis, Brian Sletten, Max Firtman 和 Chetan Karande。

这本书也离不开 Sharon Adams 和 Marilyn Messineo 的支持。她们合谋购买了 Alex 的第一台计算机，一台 Tandy TRS 80 彩色计算机。此外，没有 Jim 和 Lorri Porcello 以及 Mike 和 Sharon Adams 的爱心支持和鼓励，这本书也无法问世。

我们还要感谢加利福尼亚州塔霍市的 Coffee Connexion 提供我们完成这本书所需的咖啡，以及其老板 Robin，他给了我们这样的建议：“写一本关于编程的书？听起来很无聊！”
