# 前言

# 本书适合谁

这本书是为那些想使用 JavaScript、Node 和 Express 创建 Web 应用程序（传统网站、使用 React、Angular 或 Vue 的单页面应用程序、REST API 或介于两者之间的任何应用程序）的程序员而写的。Node 开发的一个令人兴奋的方面是，它吸引了一大批新的程序员群体。JavaScript 的易用性和灵活性吸引了来自世界各地的自学程序员。在计算机科学史上，编程从未如此易于接触。在线学习编程（以及在遇到困难时获得帮助）的资源数量和质量真是令人惊讶和鼓舞。所以对于那些新手（可能是自学的）程序员，我诚挚地欢迎您。

当然，还有像我这样在编程领域摸爬滚打多年的程序员。像我这样的程序员，从汇编语言和 BASIC 开始，经历了 Pascal、C++、Perl、Java、PHP、Ruby、C、C#和 JavaScript。在大学期间，我接触到了更多的专业语言，如 ML、LISP 和 PROLOG。这些语言中的许多都让我怀念至深，但在这些语言中，我看到的未来最具潜力的是 JavaScript。因此，我也为像我这样有丰富经验并且对特定技术有更深层见解的程序员写下了这本书。

不需要 Node 的经验，但应具有一定的 JavaScript 经验。如果您是新手程序员，我推荐 [Codecademy](http://bit.ly/2KfDqkQ)。如果您是中级或有经验的程序员，我推荐我的书，*[学习 JavaScript，第三版](http://shop.oreilly.com/product/0636920035534.do)*（O’Reilly）。本书的示例可以在任何支持 Node 的系统上使用（包括 Windows、macOS 和 Linux 等）。这些示例面向命令行（终端）用户，因此您应对系统的终端有一定的熟悉。

最重要的是，这本书是为那些充满激情的程序员而写的。他们对互联网的未来充满期待，并希望成为其中的一部分。他们对学习新事物、新技术和看待 Web 开发的新方式充满激情。如果您，亲爱的读者，还没有激情，我希望在您读完本书之前，您会有所激发……

# 第二版说明

写这本书的第一版是一件愉快的事情，我至今对我能够在其中提出的实用建议和我的读者们的热烈反响感到满意。第一版正值 Express 4.0 从测试版发布，尽管 Express 仍然是 4.x 版本，但与 Express 配套的中间件和工具已经发生了*巨大*变化。此外，JavaScript 本身也在发展，甚至 Web 应用程序的设计方式也发生了地质学上的变化（从纯服务端渲染转向单页应用程序[SPA]）。尽管第一版中的许多原则仍然有用和有效，但具体的技术和工具几乎完全不同。第二版已经迫在眉睫。由于 SPA 的盛行，本书的第二版也将更多地强调 Express 作为 API 和静态资产服务器，并包括一个 SPA 示例。

# 本书的组织方式

第一章和第二章将向你介绍 Node 和 Express 以及本书中将使用的一些工具。在第三章和第四章中，你开始使用 Express 并构建一个示例网站的框架，该示例网站将贯穿本书的其余部分作为运行示例。

第五章讨论了测试和 QA，第六章介绍了 Node 的一些重要构造以及它们如何被 Express 扩展和使用。第七章讲解了模板（使用 Handlebars），为使用 Express 构建有用网站打下了基础。第八章和第九章涵盖了 Cookies、Sessions 和表单处理，完善了你构建基本功能网站所需的知识。

第十章深入探讨了中间件，这是 Express 的核心概念。第十一章解释了如何使用中间件从服务器发送电子邮件，并讨论了与电子邮件相关的安全性和布局问题。

第十二章提前介绍了生产方面的考虑。尽管在本书的这个阶段，你可能还没有构建生产就绪网站所需的所有信息，但现在考虑生产问题可以帮助你避免未来的重大问题。

第十三章讨论持久性问题，重点关注 MongoDB（领先的文档数据库之一）和 PostgreSQL（流行的开源关系数据库管理系统）。

第十四章深入讲解 Express 路由的细节（URL 如何映射到内容），而第十五章则专注于使用 Express 编写 API。第十七章详细介绍了提供静态内容的细节，着重于最大化性能。

第十八章讨论了安全性：如何将身份验证和授权集成到您的应用程序中（重点是使用第三方身份验证提供者），以及如何通过 HTTPS 运行您的站点。

第十九章解释了如何与第三方服务集成。使用的示例包括 Twitter、Google Maps 和美国国家气象局。

第十六章利用我们对 Express 的学习，将运行示例重构为 SPA，Express 作为后端服务器提供我们在第十五章中创建的 API。

第二十章和第二十一章为您准备了大日子：您的网站上线。它们涵盖了调试，使您可以在上线前排除任何缺陷，以及上线的过程。第二十二章讨论了下一个重要（但常被忽视）阶段：维护。

本书以第二十三章结束，指向额外的资源，以便进一步学习有关 Node 和 Express 的知识，以及在哪里获取帮助。

# 示例网站

从第三章开始，本书将始终使用一个运行示例：Meadowlark Travel 网站。我在从里斯本旅行后不久写了第一版，当时我心里想着旅行，所以我选择的示例网站是我家乡俄勒冈州的一个虚构旅行公司（西部草地鹨是俄勒冈州的州鸟）。Meadowlark Travel 允许旅行者与当地的“业余导游”联系，并与提供自行车和滑板车租赁以及当地旅行的公司合作，专注于生态旅游。

就像任何教学示例一样，Meadowlark Travel 网站是虚构的，但它是一个涵盖真实世界网站面临的许多挑战的示例：第三方组件集成、地理位置、电子商务、性能和安全性。

由于本书的重点是后端基础设施，示例网站不会完整；它仅仅作为一个真实世界网站的虚构示例，以便为示例提供深度和背景。假设你正在开发自己的网站，你可以将 Meadowlark Travel 示例作为其模板。

# 本书使用的约定

本书使用以下印刷约定：

*斜体*

表示新术语、网址、电子邮件地址、文件名和文件扩展名。

`等宽字体`

用于程序列表，以及在段落内引用程序元素，如变量或函数名、数据库、数据类型、环境变量、语句和关键字。

**`等宽粗体`**

显示应由用户按字面意思输入的命令或其他文本。

*`等宽斜体`*

显示应由用户提供值或由上下文确定的值替换的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般注意事项。

###### 警告

此元素表示警告或注意事项。

# 使用代码示例

可下载附加材料（代码示例、练习等）[*https://github.com/EthanRBrown/web-development-with-node-and-express-2e*](https://github.com/EthanRBrown/web-development-with-node-and-express-2e)。

此书旨在帮助您完成工作。一般情况下，如果本书提供示例代码，则可以在您的程序和文档中使用它。除非您正在复制代码的大部分内容，否则不需要联系我们以获取权限。例如，编写一个使用本书多个代码片段的程序不需要许可。出售或分发包含 O’Reilly 书籍示例的 CD-ROM 需要许可。通过引用本书并引用示例代码来回答问题不需要许可。将本书的大量示例代码合并到您产品的文档中需要许可。

我们感激但不要求署名。署名通常包括标题、作者、出版商和 ISBN。例如：“*使用 Node 和 Express 进行 Web 开发，第二版* 由伊桑·布朗（O’Reilly）著作权 2019 年伊桑·布朗，978-1-492-05351-4。”

如果您认为使用代码示例超出了公平使用或此处授予权限，请随时联系我们*permissions@oreilly.com*。

# O’Reilly 在线学习

###### 注意

几乎 40 年来，[*O’Reilly Media*](http://oreilly.com) 提供技术和商业培训、知识和洞察，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章、会议和我们的在线学习平台分享他们的知识和专业知识。O’Reilly 的在线学习平台为您提供按需访问的现场培训课程、深入学习路径、交互式编码环境以及来自 O’Reilly 和 200 多家其他出版商的广泛文本和视频收藏。有关更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请联系出版商以解决有关此书的评论和问题：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为本书设立了一个网页，列出勘误、示例和任何额外信息。您可以访问此页面：[*https://oreil.ly/web_dev_node_express_2e*](https://oreil.ly/web_dev_node_express_2e)。

如需对本书进行评论或提出技术问题，请发送电子邮件至 *bookquestions@oreilly.com*。

欲了解更多关于我们的书籍、课程、会议和新闻的信息，请访问我们的网站：[*http://www.oreilly.com*](http://www.oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

在 YouTube 上观看我们：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)

# 致谢

生活中有许多人对这本书的实现起到了重要作用；如果没有那些触及我生命并塑造我今日的人们的影响，这本书是不可能完成的。

首先，我要感谢 Pop Art 的所有人：在 Pop Art 的时光不仅让我对工程学有了新的热情，而且我从每个人那里学到了很多，没有他们的支持，这本书也不会存在。感谢 Steve Rosenbaum 创建了一个激励人心的工作场所，感谢 Del Olds 让我加入团队，让我感到受欢迎，并且是一个光荣的领导者。感谢 Paul Inman 对工程学持续的支持和鼓舞人心的态度，以及 Tony Alferez 的热情支持，帮助我腾出时间来写作，而不影响 Pop Art。最后，感谢所有优秀的工程师们，你们让我保持警觉：John Skelton、Dylan Hallstrom、Greg Yung、Quinn Michaels、CJ Stritzel、Colwyn Fritze-Moor、Diana Holland、Sam Wilskey、Cory Buckley 和 Damion Moyer。

我对目前在价值管理策略公司的团队深表感激。我从 Robert Stewart 和 Greg Brink 那里学到了很多关于软件业务的知识，从 Ashley Carson 那里学到了团队沟通、凝聚力和效率的重要性（感谢你的坚定支持，Scratch Chromatic）。Terry Hays、Cheryl Kramer 和 Eric Trimble，感谢你们的辛勤工作和支持！还要感谢 Damon Yeutter、Tyler Brenton 和 Brad Wells 在需求分析和项目管理中的关键工作。最重要的是，感谢那些在 VMS 与我一同不懈努力的才华横溢且敬业的开发者们：Adam Smith、Shane Ryan、Jeremy Loss、Dan Mace、Michael Meow、Julianne Soifer、Matt Nakatani 和 Jake Feldmann。

感谢所有在 School of Rock 的乐队成员！这是一段多么疯狂的旅程，也是一个充满乐趣的创造性出口。特别感谢那些分享他们对音乐的激情和知识的导师们：Josh Thomas、Amanda Sloane、Dave Coniglio、Dan Lee、Derek Blackstone 和 Cory West。谢谢你们给我成为摇滚明星的机会！

Zach Mason，感谢你给了我启发。这本书也许不像*奥德赛的失落书籍*那样，但这是*我的*，如果没有你的榜样，我可能不会如此大胆。

Elizabeth 和 Ezra，谢谢你们给我的礼物。我会永远爱你们两个。

我要感谢我的家人。我再也找不到比他们给我的更好、更有爱心的教育了，我也看到他们出色的家庭教育在我姐姐身上得到了体现。

特别感谢 Simon St. Laurent 给了我这个机会，以及 Angela Rufino（第二版）和 Brian Anderson（第一版）坚定而鼓舞人心的编辑工作。感谢 O’Reilly 的每一位员工对工作的奉献和热情。特别感谢 Alejandra Olvera-Novack、Chetan Karande、Brian Sletten、Tamas Piros、Jennifer Pierce、Mike Wilson、Ray Villalobos 和 Eric Elliot 对技术审查的彻底和建设性意见。

Katy Roberts 和 Hanna Nelson 在我“越过墙头”提案中提供了宝贵的反馈和建议，这使得这本书得以问世。非常感谢你们！感谢 Chris Cowell-Shah 在 QA 章节上的优秀反馈。

最后，感谢我亲爱的朋友们，没有你们，我肯定会变得疯狂：Byron Clayton、Mark Booth、Katy Roberts 和 Kimberly Christensen。我爱你们所有人。
