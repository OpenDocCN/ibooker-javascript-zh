# 序言

又是一个美好的加利福尼亚温暖的夜晚。微弱的海风让你感到百分之百的“啊啊！”地方：洛杉矶；年份：2000 多年。我正准备将我的新网页应用程序 CSSsprites.com 通过 FTP 上传到服务器并发布到世界上。在最后几个晚上我花在应用程序上的时间里，我考虑了一个问题：为什么要花费 20% 的精力来完成应用程序的“核心”，然后花 80% 的精力与用户界面搏斗呢？如果我不需要一直使用 `getElementById()` 并担心应用程序的状态会有多好？（用户完成上传了吗？什么，出错了？这个对话框还在吗？）为什么 UI 开发如此耗时？各种浏览器又是怎么回事？慢慢地，“啊啊”变成了“啊啊哦！”

快进到 2015 年 3 月，在 Facebook 的 F8 大会上。我所在的团队准备宣布两个 Web 应用程序的完全重写：我们的第三方评论服务以及相应的管理工具。与我自己的 CSSsprites.com 应用程序相比，这些都是功能齐全的 Web 应用程序，具有更多的功能，更强大的能力和大量的流量。然而，开发过程非常愉快。团队中的新成员（甚至有些人对 JavaScript 和 CSS 都是新手）能够快速而轻松地加入，贡献自己的一点一滴，迅速提升速度。正如团队中的一名成员所说：“啊哈，现在我明白了为什么大家都这么喜欢它！”

路上发生了什么？React。

React 是用于构建用户界面的库——它帮助你一次性定义界面。然后，当应用程序的状态发生变化时，界面将重新构建以*反应*这些变化，而你无需做任何额外的工作。毕竟，你已经定义了界面。定义？更像是*声明*。你使用小巧而易管理的*组件*来构建一个功能强大的大型应用程序。不再需要在函数体中花费大量时间寻找 DOM 节点；你所需做的只是维护应用程序的状态（使用一个普通的 JavaScript 对象），剩下的事情就跟着进行了。

学习 React 是一笔划算的买卖——你学习一个库，然后用它创建以下所有内容：

+   Web 应用程序

+   本地 iOS 和 Android 应用程序

+   电视应用程序

+   本地桌面应用程序

你可以使用构建组件和用户界面的相同思想来创建具有本地性能和本地控件（*真正*的本地控件，而不是看起来像本地的复制品）的本地应用程序。这不是“一次编写，到处运行”（我们行业一直在失败），而是“学一次，到处使用”。

简而言之：学习 React，节省 80% 的时间，并专注于真正重要的事情（比如你的应用程序存在的真正原因）。

# 关于本书

本书侧重于从 Web 开发角度学习 React。在前三章中，你从一个空白的 HTML 文件开始，然后逐步构建起来。这样可以让你专注于学习 React，而不是学习新的语法或辅助工具。

第五章更多关注 JSX，这是一种通常与 React 一起使用的单独可选技术。

从这里，您将了解开发真实应用程序所需的内容，以及可以帮助您完成这一过程的附加工具。该书使用`create-react-app`快速起步，并将辅助技术的讨论保持在最低限度。其目标是专注于 React。

一个有争议的决定是除了*函数*组件外，还包括*类*组件。函数组件很可能是未来的趋势；然而，读者可能会遇到仅讨论类组件的现有代码和教程。了解两种语法方式可以提高读懂和理解现有代码的机会。

祝您在学习 React 的旅程中好运，愿其顺利而富有成效！

# 本书使用的约定

本书使用以下印刷约定：

*斜体*

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`常量宽度`

用于程序清单，以及在段落中用来指代程序元素，如变量或函数名、数据库、数据类型、环境变量、语句和关键字。

**`常量宽度粗体`**

显示用户应按字面意义键入的命令或其他文本。

###### 小贴士

该元素表示提示或建议。

###### 注意

该元素表示一般性注释。

# 使用代码示例

补充材料（例如代码示例、练习等）可在[*https://github.com/stoyan/reactbook2*](https://github.com/stoyan/reactbook2)下载。

如果您在使用代码示例时遇到技术问题或困难，请发送电子邮件至*bookquestions@oreilly.com*。

本书旨在帮助您完成工作。通常情况下，如果本书提供了示例代码，您可以在您的程序和文档中使用它。除非您复制了代码的大部分，否则您无需联系我们以获取许可。例如，编写一个使用本书多个代码片段的程序不需要许可。销售或分发 O’Reilly 图书示例需要许可。通过引用本书并引用示例代码回答问题不需要许可。将本书的大量示例代码整合到您产品的文档中需要许可。

我们感谢您的使用，但不需要署名。署名通常包括标题、作者、出版商和 ISBN 号。例如：“*React: Up & Running*, 第 2 版，作者 Stoyan Stefanov（O’Reilly）。版权所有 2022 Stoyan Stefanov，978-1-492-05146-6。”

如果您认为您使用的代码示例超出了合理使用范围或上述许可，请随时通过*permissions@oreilly.com*与我们联系。

# O’Reilly 在线学习

###### 注意

超过 40 年来，[*O’Reilly Media*](https://oreilly.com) 提供技术和商业培训，知识和见解，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专业知识。O’Reilly 的在线学习平台为您提供按需访问的实时培训课程、深入学习路径、交互式编码环境以及来自 O’Reilly 和其他 200 多家出版商的大量文本和视频。有关更多信息，请访问 [*https://oreilly.com*](https://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送给出版商：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为本书制作了一个网页，列出了勘误、示例和任何额外信息。您可以访问这个页面：[*https://oreil.ly/reactUR_2e*](https://oreil.ly/reactUR_2e)。

发送电子邮件至 *bookquestions@oreilly.com* 对本书提出评论或技术问题。

获取关于我们的书籍和课程的新闻和信息，请访问 [*https://oreilly.com*](https://oreilly.com)。

在 Facebook 上找到我们：[*https://facebook.com/oreilly*](https://facebook.com/oreilly)

在 Twitter 上关注我们：[*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia)

在 YouTube 上观看我们：[*https://www.youtube.com/oreillymedia*](https://www.youtube.com/oreillymedia)

# 致谢

我想感谢所有阅读过本书不同草稿并发送反馈和更正意见的人。

对于第一版：Andreea Manole、Iliyan Peychev、Kostadin Ilov、Mark Duppenthaler、Stephan Alber 和 Asen Bozhilov。对于第二版：Adam Rackis、Maximiliano Firtman、Chetan Karande、Kiril Christov 和 Scott Satoshi Iwako。

感谢所有在 Facebook 上工作（或与之合作）并在日常工作中回答我的问题的人，以及不断推出优秀工具、库、文章和使用模式的扩展 React 社区。

非常感谢 Jordan Walke。

感谢所有使本书成为可能的 O’Reilly 的工作人员：Angela Rufino、Jennifer Pollock、Meg Foley、Kim Cofer、Justin Billing、Nicole Shelby、Kristen Brown 和许多其他人。

感谢 Javor Vatchkov 设计了本书中开发的示例应用的用户界面（您可以在 [*whinepad.com*](https://www.whinepad.com) 上试用）。
