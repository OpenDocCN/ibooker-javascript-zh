# 前言

我（Thomas）和 Bryan 第一次见面是在我去日本移动游戏开发公司 DeNA 旧金山分公司面试时。显然，大部分高层管理人员都要说不，但在那天晚上我们俩在一个 Node.js 聚会上一起玩了一会儿后，Bryan 去说服他们给我一个 offer。

在 DeNA 工作期间，Bryan 和我致力于编写可重用的 Node.js 模块，使游戏团队可以构建他们的游戏服务器，根据需要合并组件以满足他们的游戏需求。性能一直是我们始终在衡量的东西，并且在性能上指导游戏团队也是工作的一部分；我们的服务器一直在由传统依赖于 C++ 的行业开发者持续审查。

我们两个也在其他领域合作过。在一个名为 Intrinsic 的小型安全创业公司工作时，我们的另一个角色是专注于加固 Node.js 应用程序，达到了一个完全细化的水平，我怀疑世界上再也不会出现像它这样的产品。性能调优也是那款产品的一个重大问题，因为客户不希望降低吞吐量。我们花费了许多时间进行基准测试，仔细研究火焰图，并深入研究内部的 Node.js 代码。如果工作线程模块在我们的客户需求的所有 Node.js 版本中都可用，我毫不怀疑我们会将其纳入产品中。

我们也在非就业角色中合作过。NodeSchool SF 就是一个例子，我们两个都自愿教授他人如何使用 JavaScript 并创建 Node.js 程序。我们还在许多同样的会议和聚会上演讲过。

你们两位作者都对 JavaScript 和 Node.js 充满热情，并且乐于将它们教给他人，以消除误解。当我们意识到关于构建多线程 JavaScript 应用程序的文档极度匮乏时，我们知道我们必须采取行动。这本书源于我们不仅要教育他人 JavaScript 的能力，还要帮助证明像 Node.js 这样的平台在构建利用现有硬件的高性能服务方面与其他任何平台一样强大。

# 目标读者

本书的理想读者是一名已经写了几年 JavaScript 的工程师，可能没有写多线程应用程序的经验，甚至没有使用过传统多线程语言如 C++ 或 Java 的经验。我们确实包含了一些示例 C 应用程序代码，作为一种多线程的共同语言，但读者不必熟悉或理解这些代码。

如果您有使用这类语言的经验，那很好，本书将帮助您了解 JavaScript 相应功能的等价物。另一方面，如果您只用 JavaScript 编写代码，那么本书同样适合您。我们包括跨多个学习层次的信息；这包括低级 API 参考、高级模式以及足够填补任何空白的技术边角料。

# 目标

也许本书最激动人心的目标之一是向社区传达这样的知识：使用 JavaScript 可以构建多线程应用程序。传统上，JavaScript 代码受限于单个核心，事实上，有许多 Twitter 线程和论坛帖子描述了这种语言。通过《多线程 JavaScript》这样的标题，我们希望彻底消除 JavaScript 应用程序被限制在单个核心的观念。

更具体地说，本书的目标是教会读者有关编写多线程 JavaScript 应用程序的多个方面。在阅读完本书之后，您将了解到浏览器中提供的各种 Web Worker API、它们的优势和劣势，以及何时应该使用哪种 API。关于 Node.js，您将了解到 worker threads 模块以及其 API 与浏览器中 API 的比较。

本书着重介绍了构建多线程应用的两种方法：一种是使用消息传递，另一种是使用共享内存。通过阅读本书，您将了解到实现每种方法所使用的 API，以及何时应该选择其中一种方法或两种方法的结合，并且您甚至将亲自动手使用一些基于这些方法的高级模式。

# 本书中使用的惯例

本书使用以下排版惯例：

*斜体*

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`常量宽度`

用于程序列表，以及在段落中引用程序元素，如变量或函数名、数据库、数据类型、环境变量、语句和关键字。

**`常量宽度粗体`**

显示用户应该直接输入的命令或其他文本。

*`常量宽度斜体`*

显示应该由用户提供的值或根据上下文确定的值替换的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般注释。

###### 警告

此元素表示警告或注意。

# 使用代码示例

补充材料（代码示例、练习等）可从[*https://github.com/MultithreadedJSBook/code-samples*](https://github.com/MultithreadedJSBook/code-samples)下载。

如果您有技术问题或在使用代码示例时遇到问题，请发送电子邮件至*bookquestions@oreilly.com*。

本书的目的是帮助您完成工作。一般情况下，如果本书提供示例代码，您可以在您的程序和文档中使用它。除非您复制了代码的大部分，否则您无需联系我们以获得许可。例如，编写一个使用本书多个代码片段的程序不需要许可。销售或分发 O’Reilly 书籍的示例需要许可。引用本书回答问题并引用示例代码不需要许可。将本书的大量示例代码整合到您产品的文档中需要许可。

我们感谢但通常不需要署名。署名通常包括标题、作者、出版商和 ISBN。例如：“*Multithreaded JavaScript* by Thomas Hunter II and Bryan English（O’Reilly）。Copyright 2022 Thomas Hunter II and Bryan English, 978-1-098-10443-6。”

如果您觉得您对代码示例的使用超出了合理使用范围或上述许可，请随时通过*permissions@oreilly.com*与我们联系。

# O’Reilly 在线学习

###### 注意

40 多年来，[*O’Reilly Media*](http://oreilly.com)为企业提供技术和商业培训、知识和见解，帮助它们取得成功。

我们独特的专家和创新者网络通过书籍、文章以及我们的在线学习平台分享他们的知识和专长。O’Reilly 的在线学习平台为您提供按需访问实时培训课程、深度学习路径、交互式编码环境以及来自 O’Reilly 和其他 200 多家出版商的广泛文本和视频的机会。欲了解更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送至出版商：

+   O’Reilly Media, Inc.

+   Gravenstein Highway North 1005 号

+   加利福尼亚州塞巴斯托波尔 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为本书设立了一个网页，列出勘误、示例和任何额外信息。您可以访问[*https://oreil.ly/multithreaded-js*](https://oreil.ly/multithreaded-js)。

发送电子邮件至*bookquestions@oreilly.com*以就本书发表评论或提出技术问题。

欲了解有关我们的图书和课程的新闻和信息，请访问[*http://oreilly.com*](http://oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)。

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)。

在 YouTube 上观看我们：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)。

# 致谢

感谢以下人员提供的详细技术审查，使本书得以实现：

Anna Henningsen（[@addaleax](https://twitter.com/addaleax)）

Anna 目前是德国 MongoDB 开发者工具团队的一员，过去五年来一直是 Node.js 核心的最活跃贡献者之一，并在平台上实现了工作线程的重要参与。她对 Node.js 及其社区充满激情。

Shu-yu Guo ([@_shu](https://twitter.com/_shu))

Shu 致力于 JavaScript 的实现和标准化工作。他是 TC39 委员会代表之一，ECMAScript 规范的编辑之一，也是内存模型的作者。他目前在 Google 的 V8 引擎团队工作，负责领导 JavaScript 语言特性的实现和标准化工作。之前，他曾在 Mozilla 和 Bloomberg 工作过。

Fernando Larrañaga ([@xabadu](https://twitter.com/xabadu))

Fernando 是一名工程师和开源贡献者，多年来在南美和美国领导 JavaScript 和 Node.js 社区。他目前是 Square 的高级软件工程师，是 NodeSchool SF 的组织者，在 Twilio 和 Groupon 等其他主要科技公司有过任职，自 2014 年以来一直在开发企业级 Node.js 并扩展数百万用户使用的 Web 应用程序。
