# 前言

当我坐下来着手编写*JavaScript Cookbook*的最新版本时，我仔细考虑了“烹饪书”这个比喻。什么才是一本伟大的食谱书？在我家餐厅的书架上翻阅烹饪书时，我注意到我的最爱不仅有美味的食谱，而且充满了主观的、值得的建议。一本烹饪书很少会试图教会你*每一种*烩牛肉的食谱；它更多地教你作者发现对他们最有效的技术和食谱，通常还会加入一些建议。我们编制这本 JavaScript 配方集就是以这个概念为基础的。本书中的建议来自三位经验丰富的专家，但最终是*我们*独特经验的结晶。其他任何一组开发者可能会出版一本类似但又不同的书。

JavaScript 已经发展成为一个令人惊叹且功能强大的多用途编程语言。有了这个收藏，你将能够解决遇到的各种问题，甚至可能开始开发自己的配方。

# 读者对象

为了包含当今 JavaScript 使用中反映的许多主题和话题，我们必须从一个前提开始：这不是一本面向编程新手的书籍。有许多适合那些想要学习使用 JavaScript 编程的好书和教程，我们觉得可以针对*实践开发者*，即那些希望用 JavaScript 解决特定问题和挑战的人。

如果你已经玩了几个月的 JavaScript，或许尝试过一点 Node 或 Web 开发，那么你应该对本书的内容感到舒适。此外，如果你是主要使用其他编程语言的开发者，但偶尔需要使用 JavaScript，这本书也会是一个有用的指南。最后，如果你是一个工作中的 JavaScript 开发者，有时会陷入语言的一些特异性之中，这本书应该是一个有用的资源。

# 书籍组织

本书有两类读者。第一类是那些从头到尾阅读它，途中获取适用知识碎片的人。第二类是那些按需浏览，寻找解决特定挑战或问题类别的人。我们尝试以这样的方式组织本书，使其对两类读者都有用，将其分为三个部分：

+   第 I 部分，*JavaScript 语言*，涵盖了 JavaScript 作为编程语言的配方。

+   第 II 部分，*JavaScript 在浏览器中*，涵盖了 JavaScript 在其自然栖息地中的应用：浏览器。

+   第 III 部分，*Node.js*，特别从 Node.js 的视角看待 JavaScript。

每一章的书籍都被细分为多个“配方”。一个配方由几个部分组成：

问题

这定义了一种常见的开发场景，其中可能使用 JavaScript。

解决方案

一个解决问题的解决方案，包括代码示例和简要描述。

讨论

对代码示例和技术进行深入讨论。

另外，配方可能包含在“另请参阅”部分中进一步阅读的推荐或在“额外”部分中的额外技术。

# 本书中使用的约定

本书使用以下排版约定：

斜体

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

**粗体**

表示应选择或单击的 UI 项，如菜单项和按钮。

`常量宽度`

表示广义上的计算机代码，包括命令、数组、元素、语句、选项、开关、变量、属性、键、函数、类型、类、命名空间、方法、模块、属性、参数、值、对象、事件、事件处理程序、XML 标签、HTML 标签、宏、文件内容以及命令的输出。

`**常量宽度粗体**`

显示用户应该按照字面意义输入的命令或其他文本。

*`常量宽度斜体`*

展示应由用户提供值或由上下文确定的值替换的文本。

###### 注意

此元素表示一般注释。

###### 提示

此元素表示提示或建议。

###### 警告

此元素表示警告或注意事项。

本书提及网站和页面，帮助您找到可能有用的在线信息。通常会同时提到地址（URL）和名称（或标题或适当的标题）。有些地址相对复杂。您可以使用您喜欢的搜索引擎通过名称搜索页面，从而更轻松地找到这些页面。如果不能通过地址找到页面，这也可能有帮助；URL 可能已更改，但名称仍然有效。

# 使用代码示例

可供下载的补充材料（代码示例、练习等）位于[*https://github.com/javascripteverywhere/cookbook*](https://github.com/javascripteverywhere/cookbook)。

本书旨在帮助您完成工作。一般而言，如果本书提供了示例代码，您可以在您的程序和文档中使用它。除非您复制了本书的大部分代码，否则无需联系我们以获取许可。例如，编写使用本书多个代码片段的程序不需要许可。销售或分发 O'Reilly 书籍中的示例代码需要许可。引用本书并引用示例代码回答问题不需要许可。将本书的大量示例代码整合到产品文档中需要许可。

我们感激但不要求署名。署名通常包括标题、作者、出版社和 ISBN。例如：*JavaScript Cookbook*，第三版，作者 Adam D. Scott、Matthew MacDonald 和 Shelley Powers。版权所有 2021 年 Adam D. Scott 和 Matthew MacDonald，978-1-492-05575-4。

如果您认为您对代码示例的使用超出了公平使用范围或此处授权，请随时通过 permissions@oreilly.com 联系我们。

# O’Reilly Online Learning

###### 注意

[*O’Reilly Media*](http://oreilly.com) 已经提供技术和商业培训、知识和见解，帮助公司取得成功超过 40 年。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专长。O’Reilly 的在线学习平台为您提供按需访问的实时培训课程、深度学习路径、交互式编码环境以及来自 O’Reilly 和其他 200 多个出版商的大量文本和视频。欲了解更多信息，请访问 [*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送至出版社：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（在美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为本书设立了一个网页，列出勘误表、示例和任何额外信息。您可以访问 [*https://oreil.ly/js-cookbook-3e*](https://oreil.ly/js-cookbook-3e)。

发送电子邮件至 *bookquestions@oreilly.com* 对本书进行评论或提出技术问题。

有关我们的书籍和课程的新闻和信息，请访问 [*http://oreilly.com*](http://oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

在 YouTube 观看我们：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)

# 致谢

这是 *JavaScript Cookbook* 的第三版。前两版由 Shelley Powers 编写。本版由 Adam Scott 和 Matthew MacDonald 撰写和更新。Adam 和 Matthew 感谢他们的编辑 Angela Rufino 和 Jennifer Pollock，在项目的所有成长阶段中引导本书；以及他们的顶尖技术审阅者 Sarah Wachs、Schalk Neethling 和 Elisabeth Robson，他们提供了许多深刻的见解和有益的建议。Adam 还感谢 John Paxton 在本版初稿期间的支持和交流。

Shelley 感谢她的编辑 Simon St. Laurent 和 Brian McDonald，以及她的技术审阅者 Dr. Axel Rauschmayer 和 Semmy Purewal。

我们所有人都感谢 O’Reilly 生产工作人员的持续帮助和支持。
