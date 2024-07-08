# 序言

在 *NodeSchool San Francisco* 和 *Ann Arbor PHP MySQL* 团体之间，我花了几年时间教授他人如何编程。到目前为止，我已经与数百名学生合作，通常从安装所需软件并配置它的单调过程开始。之后，通过一点代码和大量解释，我们达到了学生程序运行的部分，一切都变得“顺畅”了。我总能感觉到它的发生：学生微笑着讨论他们新获得的技能可能带来的各种可能性，就像是视频游戏中的一个能力提升。

我的目标是通过本书为您重新创造那种令人兴奋的感觉。在这些页面中，您将找到许多实际示例，您可以在开发机器上运行各种后端服务，并使用示例 Node.js 应用程序代码与其进行交互。随之而来的是大量解释和小的离题讨论，以满足那些好奇心强的人。

完成本书后，您将安装并运行许多不同的服务，并且针对每个服务，您都将编写 Node.js 应用程序代码与其交互。本书更加强调这些交互，而不是审查 Node.js 应用程序代码本身。

JavaScript 是一种强大的语言，能够开发前端和后端应用程序。这使得我们很容易完全专注于学习语言本身，而避开周边的技术。本书的主旨是，我们 JavaScript 工程师通过与许多人认为只有使用传统企业平台如 Java 或 .NET 的工程师熟悉的技术进行第一手体验，会受益匪浅。

# 目标受众

本书不会教您如何使用 Node.js，并且为了从中获得最大收益，您应该已经编写过几个 Node.js 应用程序，并且对 JavaScript 有一个具体的理解。尽管如此，本书确实涵盖了一些关于 Node.js 和 JavaScript 的高级和不太知名的概念，例如 “JavaScript 的单线程特性” 和 “Node.js 事件循环”。您还应该熟悉 HTTP 的基础知识，至少使用过一种数据库来持久化状态，并且了解在运行中的 Node.js 进程中维护状态的易用性和危险性。

或许您已经在一家拥有运行后端服务基础架构的公司工作，并且渴望了解它的工作原理，以及您的 Node.js 应用程序可以从中受益。或者您有一个 Node.js 应用程序作为副业项目，厌倦了它的崩溃。您甚至可能是一家年轻创业公司的 CTO，决心满足不断增长的用户需求。如果您的情况与其中任何一种相似，那么这本书适合您。

# 目标

Node.js 通常用于构建前端 Web 应用程序。 本书不涵盖与前端开发或浏览器相关的任何主题。 已经有许多书籍涵盖了这样的内容。 相反，本书的目标是让您将后端 Node.js 服务与支持现代分布式系统的各种服务集成起来。

读完本书后，您将了解在生产环境中运行 Node.js 服务所需的许多技术。 例如，如何部署和扩展应用程序，如何使其冗余并对故障具有韧性，如何可靠地与其他分布式进程通信以及如何观察应用程序的健康状况。

仅凭阅读本书，您不会成为这些系统的专家。 例如，不涉及调整和分片以及将可扩展的 ELK 服务部署到生产所需的操作工作。 但是，您将了解如何运行本地 ELK 实例，将其日志发送到 Node.js 服务中，并创建用于可视化服务健康状况的仪表板（在“使用 ELK 进行日志记录”中介绍）。

本书当然不涵盖您所在公司使用的所有技术。 虽然第七章讨论了 Kubernetes，这是一个用于编排应用程序代码部署的技术，但您的雇主可能使用不同的解决方案，如 Apache Mesos。 或者您可能依赖云环境中的 Kubernetes 版本，其中底层实现对您隐藏。 无论如何，通过了解分布式后端服务堆栈中不同层的工具，您将更容易理解可能遇到的其他技术堆栈。

# 本书中使用的约定

本书使用以下排版约定：

*斜体*

指示新术语、URL、电子邮件地址、文件名和文件扩展名。

`Constant width`

用于程序清单，以及在段落中引用程序元素，例如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

**`Constant width bold`**

显示用户应按字面输入的命令或其他文本。

*`Constant width italic`*

显示应由用户提供的值或由上下文确定的值替换的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般注释。

###### 警告

此元素表示警告或注意事项。

# 使用代码示例

补充材料（代码示例、练习等）可在[*https://github.com/tlhunter/distributed-node*](https://github.com/tlhunter/distributed-node)下载。

如果您有技术问题或在使用代码示例时遇到问题，请发送电子邮件至*bookquestions@oreilly.com*。

本书旨在帮助您完成工作。一般来说，如果本书提供示例代码，您可以在您的程序和文档中使用它。除非您复制了代码的大部分，否则无需联系我们请求许可。例如，编写一个使用本书中几个代码块的程序不需要许可。出售或分发 O’Reilly 图书中的示例需要许可。通过引用本书回答问题并引用示例代码不需要许可。将本书中大量示例代码合并到产品文档中需要许可。

我们感谢，但通常不要求归属。归属通常包括标题、作者、出版商和 ISBN。例如：“*使用 Node.js 进行分布式系统*，作者 Thomas Hunter II（O’Reilly）。版权所有 2020 年 Thomas Hunter II，978-1-492-07729-9。”

如果您觉得您对代码示例的使用超出了合理使用范围或上述许可，请随时通过*permissions@oreilly.com*与我们联系。

# O’Reilly 在线学习

###### 注意

超过 40 年来，[*O’Reilly Media*](http://oreilly.com)提供技术和商业培训、知识和见解，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专长。O’Reilly 的在线学习平台为您提供按需访问实时培训课程、深入学习路径、交互式编码环境以及来自 O’Reilly 和其他 200 多家出版商的大量文本和视频。有关更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送至出版商：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为这本书创建了一个网页，列出勘误、示例和任何额外信息。您可以访问[*https://oreil.ly/dist-nodejs*](https://oreil.ly/dist-nodejs)查看此页面。

发送电子邮件至*bookquestions@oreilly.com*评论或询问有关本书的技术问题。

有关我们的图书和课程的新闻和信息，请访问[*http://oreilly.com*](http://oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

在 YouTube 上观看我们：[*http://youtube.com/oreillymedia*](http://youtube.com/oreillymedia)

# 致谢

本书得以完成，要感谢以下人员提供的详细技术审查：

Fernando Larrañaga（[@xabadu](https://twitter.com/xabadu)）

Fernando 是一位工程师、开源贡献者，多年来在南美和美国领导 JavaScript 和 Node.js 社区。他目前是 Square 公司的高级软件工程师，在 Twilio 和 Groupon 等其他主要科技公司任职过。他已经有七年多的时间开发企业级 Node.js 应用和扩展支持数百万用户的 Web 应用。

Bryan English ([@bengl](https://twitter.com/bengl))

Bryan 是一位开源 JavaScript 和 Rust 程序员和爱好者，参与过大型企业系统、仪器化和应用安全的工作。目前是 Datadog 公司的高级开源软件工程师。他从 Node.js 初期就开始专业和个人项目中使用 Node.js，并且是 Node.js 核心合作者，在多个工作组中以多种方式贡献了 Node.js。

Julián Duque ([@julian_duque](https://twitter.com/julian_duque))

Julián Duque 是社区领袖、公众演讲家、JavaScript/Node.js 传道者，以及官方 Node.js 合作者（名誉退休）。目前在 Salesforce Heroku 担任高级开发者倡导者，同时组织 JSConf 和 NodeConf Colombia，也在帮助组织 JSConf México 和 MedellinJS，哥伦比亚最大的 JavaScript 用户组，拥有 5000 多名注册会员。他对教育充满热情，通过不同的社区工作坊、专业培训和在线平台如 Platzi 教授软件开发基础、JavaScript 和 Node.js。

我也要特别感谢那些给予我指导和反馈的人：Dan Shaw ([@dshaw](https://twitter.com/dshaw)), Brad Vogel ([@BradVogel](https://twitter.com/BradVogel)), Matteo Collina ([@matteocollina](https://twitter.com/matteocollina)), Matt Ranney ([@mranney](https://twitter.com/mranney)), 和 Rich Trott ([@trott](https://twitter.com/trott))。
