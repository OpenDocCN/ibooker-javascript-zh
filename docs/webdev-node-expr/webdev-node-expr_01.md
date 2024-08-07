# 第一章：介绍 Express

# JavaScript 革命

在我介绍本书的主题之前，重要的是提供一些背景和历史背景，这意味着谈论 JavaScript 和 Node。JavaScript 的时代确实已经来临。从它作为客户端脚本语言的谦逊开始，它不仅在客户端完全普及，而且它作为服务器端语言的使用也终于起飞，多亏了 Node。

一种完全基于 JavaScript 的技术栈的承诺是明确的：不再需要切换上下文！你再也不必从 JavaScript 切换到 PHP、C#、Ruby 或 Python（或任何其他服务器端语言）。此外，它还赋予了前端工程师向服务器端编程迈进的能力。这并不是说服务器端编程只与语言有关；仍然有很多需要学习的地方。不过，至少使用 JavaScript 时，语言本身不会成为障碍。

这本书是为了所有看到 JavaScript 技术栈的潜力的人而写的。也许你是一名前端工程师，希望将你的经验扩展到后端开发。也许你像我一样是一名有经验的后端开发者，正在将 JavaScript 作为一种可行的替代方案来看待传统的服务器端语言。

如果你像我一样是一名软件工程师这么长时间了，你会看到许多语言、框架和 API 进入流行。有些大行其道，有些则逐渐被淘汰。你可能为自己能够快速学习新语言和新系统而感到自豪。你遇到的每种新语言都会感觉更加熟悉：你从大学学习的语言中认出一些内容，在几年前的工作中学到了另一些内容。有这样的视角感觉很好，当然，但也很疲惫。有时候你只想*做点事情*，而不必学习全新的技术或者重新磨练数月甚至数年未用的技能。

JavaScript 乍看起来可能不太可能成为冠军。我能理解，相信我。如果你在 2007 年告诉我，我不仅会认为 JavaScript 是我首选的语言，还会写一本关于它的书，我会告诉你你疯了。我对 JavaScript 有所有通常的偏见：我认为它是一个“玩具”语言，是给业余者和浅尝辄止者搞砸和滥用的东西。公平地说，JavaScript 确实降低了业余者的门槛，而且存在许多问题的 JavaScript，这并没有帮助改善语言的声誉。换句话说，可以说“恨玩家，不恨游戏”。

令人遗憾的是，人们对 JavaScript 持有这种偏见；这阻止了人们发现这门语言有多么强大、灵活和优雅。即使我们现在所知的这门语言自 1996 年以来已经存在了（尽管它的许多更吸引人的特性是在 2005 年添加的）。

通过阅读这本书，你可能摆脱了那种偏见：要么是因为像我一样，你已经克服了它，要么是因为你一开始就没有那种偏见。无论哪种情况，你都很幸运，我期待着向你介绍 Express，这是一项由一种令人愉快而惊喜的语言实现的技术。

2009 年，当人们开始意识到 JavaScript 作为浏览器脚本语言的强大和表现力时，Ryan Dahl 意识到 JavaScript 作为服务端语言的潜力，于是 Node.js 诞生了。这是互联网技术蓬勃发展的时期。Ruby（以及 Ruby on Rails）从学术计算机科学中吸取了一些很棒的思想，结合了一些自己的新思想，并向世界展示了一种更快构建网站和 Web 应用程序的方式。微软为了在互联网时代取得关键成就，不仅从 Ruby 和 JavaScript 中吸取了教训，还从 Java 的错误中汲取了教训，并且大量借鉴了学术界的成果。

如今，Web 开发人员有自由使用最新的 JavaScript 语言特性的权利，而不用担心排斥使用旧版浏览器的用户，这得益于 Babel 等转换技术。Webpack 已经成为管理 Web 应用程序依赖关系并确保性能的通用解决方案，而 React、Angular 和 Vue 等框架正在改变人们对 Web 开发的看法，将声明式的文档对象模型（DOM）操作库（如 jQuery）淘汰到昨天的新闻。

现在参与互联网技术是一件激动人心的事情。到处都有惊人的新思想（或者是惊人的老思想重焕生机）。创新和激情的精神现在比许多年前都要强烈。 

# 介绍 Express

Express 网站将 Express 描述为“一个最小化且灵活的 Node.js Web 应用程序框架，为 Web 和移动应用程序提供了丰富的功能集。” 然而，这究竟意味着什么呢？让我们详细了解一下这个描述：

最小化

这是 Express 中最吸引人的方面之一。许多时候，框架开发者会忘记通常情况下“少即是多”的原则。Express 的理念是在你的大脑和服务器之间提供*最小*的层次。这并不意味着它不强大，或者它没有足够的有用功能。它的意思是它少给你增添麻烦，让你完全表达自己的想法，同时提供有用的东西。Express 提供了一个最小的框架，你可以根据需要添加 Express 功能的不同部分，替换不符合你需求的部分。这是一种清新的气息。这么多框架给你提供*一切*，使你在甚至写一行代码之前就面临庞大、神秘和复杂的项目。通常，第一项任务是浪费时间剔除不需要的功能，或者替换不符合要求的功能。Express 采取了相反的方式，允许你在需要时添加你需要的东西。

灵活

说到底，Express 所做的事情非常简单：它接受来自客户端的 HTTP 请求（可以是浏览器、移动设备、另一个服务器、桌面应用程序…… 任何能够使用 HTTP 接口的东西），然后返回一个 HTTP 响应。这一基本模式描述了几乎与互联网有关的所有事物，使得 Express 在应用中非常灵活。

Web 应用框架

也许更准确的描述应该是“Web 应用框架的服务器端组件”。今天，当你想到“Web 应用框架”，你通常会考虑到像 React、Angular 或 Vue 这样的单页应用框架。然而，除了少数独立应用程序外，大多数 Web 应用需要共享数据，并与其他服务集成。它们通常通过 Web API 来实现，这可以被认为是 Web 应用框架的服务器端组件。请注意，仍然有可能（有时也是可取的）只使用服务器端渲染构建整个应用程序，这种情况下，Express 很可能构成整个 Web 应用框架！

除了 Express 自己描述的功能，我还想补充两点：

快速

随着 Express 成为 Node.js 开发的首选 Web 框架，吸引了许多运行高性能、高流量网站的大公司的注意。这给 Express 团队带来了对性能的关注，使得 Express 现在为高流量网站提供了领先的性能。

无固执己见

JavaScript 生态系统的一个显著特点是其规模和多样性。虽然 Express 经常成为 Node.js Web 开发的核心，但在一个 Express 应用程序中，涉及到成百上千（如果不是成千上万）的社区包。Express 团队意识到了这种生态系统的多样性，并通过提供极其灵活的中间件系统来响应，使得在创建应用程序时能够轻松使用自己选择的组件。在 Express 的发展过程中，你可以看到它放弃了一些“内置”组件，转而采用可配置的中间件。

我提到 Express 是一个 Web 应用程序框架的“服务器端部分”…因此我们可能应该考虑一下服务器端应用程序和客户端应用程序之间的关系。

# 服务器端和客户端应用程序

*服务器端应用程序* 是指在服务器上渲染应用程序的页面（以 HTML、CSS、图像和其他多媒体资产以及 JavaScript 的形式）并发送给客户端的应用程序。相比之下，*客户端应用程序* 大多数情况下从一次性发送的初始应用程序捆绑包中渲染其自己的用户界面。也就是说，一旦浏览器接收到最初（通常非常简化的）HTML，它就使用 JavaScript 动态修改 DOM，不需要依赖服务器来显示新页面（尽管原始数据通常仍然来自服务器）。

在 1999 年之前，服务器端应用程序是标准。事实上，*Web 应用程序* 这个术语就是在那一年正式引入的。我认为大约在 1999 年到 2012 年之间的时期是 Web 2.0 时代，这段时间内开发了最终成为客户端应用程序的技术和技巧。到了 2012 年，随着智能手机的广泛普及，向网络发送尽可能少的信息成为常规做法，这种做法有利于客户端应用程序的发展。

服务器端应用程序通常被称为*服务器端渲染*（SSR），客户端应用程序通常称为*单页应用程序*（SPA）。客户端应用程序在诸如 React、Angular 和 Vue 等框架中得到了完全实现。我一直觉得“单页”有点不恰当，因为从用户的角度来看，确实可以有很多页面。唯一的区别在于页面是从服务器发送还是在客户端动态渲染。

实际上，在服务器端应用程序和客户端应用程序之间有许多模糊的界限。许多客户端应用程序有两到三个 HTML 捆绑包可以发送给客户端（例如，公共界面和已登录界面，或者常规界面和管理员界面）。此外，SPA 通常与 SSR 结合使用，以提高第一次加载性能并帮助搜索引擎优化（SEO）。

总的来说，如果服务器发送少量 HTML 文件（通常是一到三个），并且用户通过动态 DOM 操作体验丰富的多视图体验，我们认为这是客户端渲染。不同视图的数据（通常以 JSON 形式）和多媒体资产通常仍然来自网络。

当然，Express 并不太在乎你是在制作服务器端应用程序还是客户端应用程序；它愿意在任何角色中发挥作用。对于 Express 来说，无论您是提供一个 HTML 包还是一百个 HTML 包都没有区别。

尽管单页应用（SPAs）已经明确地“赢得了”主导的 Web 应用架构地位，本书始于与服务器端应用一致的例子。它们仍然相关，并且服务一个 HTML 包或多个之间的概念差异很小。在第十六章中有一个 SPA 示例。

# Express 的简史

Express 的创造者 TJ Holowaychuk 将 Express 描述为受 Sinatra 启发的 Web 框架，Sinatra 是基于 Ruby 的 Web 框架。Express 借鉴了建立在 Ruby 上的框架并不奇怪：Ruby 衍生出了大量优秀的 Web 开发方法，旨在使 Web 开发更快、更高效和更易于维护。

尽管 Express 受到了 Sinatra 的启发，但它也与 Connect 深度交织，Connect 是 Node 的一个“插件”库。Connect 创造了“中间件”这个术语，用来描述可以处理 Web 请求的可插拔 Node 模块。在 2014 年的 4.0 版本中，Express 移除了对 Connect 的依赖，但仍然保留了其中间件的概念。

###### 注意

Express 在 2.x 和 3.0 之间经历了相当大的重写，然后在 3.x 和 4.0 之间又重写了一次。本书侧重于 4.0 版本。

# Node：一种新型 Web 服务器

从某种意义上说，Node 与其他流行的 Web 服务器（如 Microsoft 的 Internet Information Services（IIS）或 Apache）有很多共同点。不过更有趣的是它们的区别，所以让我们从这里开始。

与 Express 类似，Node 对于 Web 服务器的处理非常简化。与花费多年来掌握的 IIS 或 Apache 不同，Node 的设置和配置都很容易。这并不是说在生产环境中调优 Node 服务器以获得最佳性能是一件微不足道的事情；只是配置选项更简单，更直接。

Node 和更传统的 Web 服务器之间的另一个重要区别是 Node 是单线程的。乍一看，这似乎是向后退。事实证明，这是一个天才的举措。单线程极大地简化了编写 Web 应用程序的业务，并且如果你需要多线程应用程序的性能，你可以简单地启动更多 Node 实例，从而有效地获得多线程的性能优势。敏锐的读者可能会觉得这听起来像是虚张声势。毕竟，通过服务器并行性（与应用程序并行性相对），多线程并不是简单地将复杂性移动，而是重新分配了复杂性。也许是这样，但根据我的经验，它已经将复杂性移动到了恰到好处的位置。此外，随着云计算和将服务器视为通用商品的日益流行，这种方法变得更加合理。IIS 和 Apache 确实很强大，它们被设计用来从今天强大的硬件中挤取最后一滴性能。然而，这是有代价的：它们需要相当多的专业知识来设置和调优以达到这种性能。

在应用程序编写方式上，Node 应用与 PHP 或 Ruby 应用更为相似，而不是.NET 或 Java 应用。虽然 Node 使用的 JavaScript 引擎（Google 的 V8）将 JavaScript 编译为本机机器码（类似于 C 或 C++），但它是透明的，所以从用户的角度来看，它表现得像一个纯解释语言。没有单独的编译步骤可以减少维护和部署的麻烦：你只需要更新一个 JavaScript 文件，你的更改就会自动生效。

另一个 Node 应用的引人注目的好处是 Node 非常独立于平台。它并非第一个或唯一的平台无关服务器技术，但平台独立实际上更像是一个连续的谱而不是一个二进制命题。例如，你可以通过 Mono 在 Linux 服务器上运行.NET 应用，但由于文档不完善和系统不兼容性，这是一项艰难的任务。同样地，你可以在 Windows 服务器上运行 PHP 应用，但通常不像在 Linux 机器上那样容易设置。另一方面，Node 在所有主要操作系统（Windows、macOS 和 Linux）上都很容易设置，并支持简单的协作。在网站设计团队中，PC 和 Mac 的混合使用非常普遍。某些平台，比如.NET，给前端开发人员和设计师带来了挑战，而他们通常使用 Mac，这对协作和效率产生了巨大影响。能够在几分钟甚至几秒钟内在任何操作系统上启动一个功能齐全的服务器的想法实现了梦想。

# Node 生态系统

当然，Node 位于整个堆栈的核心。它是使 JavaScript 在服务器上运行的软件，与浏览器解耦，从而允许使用 JavaScript 编写的框架（如 Express）。另一个重要组件是数据库，我们将在第十三章中更深入地讨论它。除了最简单的 Web 应用程序之外，几乎所有的 Web 应用程序都需要数据库，而有些数据库比其他数据库更适合 Node 生态系统。

毫不奇怪，针对所有主要关系数据库（如 MySQL、MariaDB、PostgreSQL、Oracle、SQL Server）都提供了数据库接口；忽视这些已经建立起来的巨头是愚蠢的。然而，Node 开发的出现使数据库存储采用了一种新的方法：所谓的 NoSQL 数据库。将某物定义为其“不是”并不总是有帮助，因此我们会补充说，这些 NoSQL 数据库更恰当地称为“文档数据库”或“键/值对数据库”。它们提供了一种概念上更简单的数据存储方法。虽然有很多种，但 MongoDB 是其中的佼佼者，也是本书中我们将使用的 NoSQL 数据库。

由于构建一个功能性网站依赖于多种技术组件，因此衍生出了一些缩写词来描述网站所依赖的“技术堆栈”。例如，Linux、Apache、MySQL 和 PHP 的组合被称为*LAMP*堆栈。MongoDB 工程师瓦列里·卡尔波夫创造了缩写*MEAN*：Mongo、Express、Angular 和 Node。虽然这确实引人注目，但它有其局限性：数据库和应用程序框架的选择如此之多，以至于“MEAN”无法捕捉到生态系统的多样性（它还忽略了我认为很重要的一个组件：渲染引擎）。

创造一个包容性缩写词是一个有趣的练习。当然，不可或缺的组件是 Node。虽然还有其他服务器端 JavaScript 容器，但 Node 正逐渐成为主导。Express 也不是唯一的 Web 应用程序框架，尽管它在与 Node 的竞争中占据了重要地位。通常对 Web 应用程序开发至关重要的另外两个组件是数据库服务器和渲染引擎（如 Handlebars 这样的模板引擎或者像 React 这样的 SPA 框架）。对于这最后两个组件来说，没有明确的佼佼者，这就是我认为局限性是一种错误。

将所有这些技术联系在一起的是 JavaScript，为了包容性，我将使用“JavaScript 堆栈”这个术语。对于本书的目的而言，这意味着 Node、Express 和 MongoDB（在第十三章中还有一个关于关系数据库的示例）。

# 许可证

在开发 Node 应用程序时，你可能会发现自己需要比以往更多地关注许可证问题（我确实有这种感觉）。Node 生态系统的一大优点是提供给你的大量包。然而，每个包都有其自己的许可证，更糟糕的是，每个包可能依赖于其他包，这意味着理解你编写的应用程序各部分的许可证可能会有些棘手。

不过，还有一些好消息。Node 包中最流行的许可证之一是 MIT 许可证，非常宽松，允许你几乎可以做*任何*你想做的事情，包括在闭源软件中使用该包。但是，你不应该假设你使用的每个包都是 MIT 许可的。

###### 提示

在 npm 中有几个可用的包会尝试查找你项目中每个依赖项的许可证。在 npm 中搜索 `nlf` 或 `license-report`。

虽然 MIT 是你最常遇到的许可证，但你可能也会看到以下许可证：

GNU 通用公共许可证（GPL）

GPL 是一种流行的开源许可证，精心设计用于保持软件的自由。这意味着如果你在项目中使用 GPL 许可的代码，你的项目*也*必须使用 GPL 许可。自然地，这意味着你的项目不能是闭源的。

Apache 2.0

此许可证与 MIT 类似，允许你为你的项目选择不同的许可证，包括闭源许可证。然而，你必须包含使用 Apache 2.0 许可证组件的通知。

伯克利软件分发许可证（BSD）

类似于 Apache，该许可证允许你在项目中使用任何你希望的许可证，只要包含 BSD 许可证组件的通知。

###### 注意

有时软件会以*双重许可*（以两种不同的许可证许可）发布。这样做的一个常见原因是允许软件在 GPL 项目和更宽松许可的项目中使用。（对于一个组件要用于 GPL 软件，该组件必须是 GPL 许可的。）这是我在自己的项目中经常采用的许可方案：使用 GPL 和 MIT 双重许可。

最后，如果你发现自己正在编写自己的包，你应该成为一个好公民，并为你的包选择一个许可证，并正确地记录它。对于开发者来说，没有什么比使用某人的包并不得不深入源代码查找许可证或者更糟糕的是发现它根本没有许可证更令人沮丧的了。

# 结论

希望本章节能让你更加深入地了解 Express 是什么以及它如何融入更大的 Node 和 JavaScript 生态系统中，同时也更加清晰地了解服务端和客户端 Web 应用程序之间的关系。

如果你仍然对 Express 究竟是什么感到困惑，不用担心：有时候，开始使用它以理解它的本质要容易得多，这本书将帮助你开始使用 Express 构建 Web 应用程序。然而，在我们开始使用 Express 之前，我们将在下一章节中对 Node 进行介绍，这是理解 Express 如何工作的重要背景信息。

^(1) 经常被称为*即时*（JIT）编译。
