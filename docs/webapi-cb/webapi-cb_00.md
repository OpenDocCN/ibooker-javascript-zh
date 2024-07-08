# 序言

自 1995 年末引入以来，JavaScript 已经走过了漫长的道路。在早期，内置于 Web 浏览器中的核心 API 有限。更先进的功能通常需要第三方 JavaScript 库，或者在某些情况下甚至需要浏览器插件。

Web API 是浏览器公开的一系列全局对象和函数。您的 JavaScript 代码可以使用这些对象与文档对象模型（DOM）交互，执行网络通信，与本机设备功能集成等等。

# 现代浏览器的威力

现代 Web API 对 Web 平台有两个重要优势：

不再需要插件

在过去，这些功能大部分只能供原生应用程序或笨重的浏览器插件使用。（还记得 ActiveX 和 Flash 吗？）

更少的第三方依赖

现代浏览器提供了大量以前需要第三方 JavaScript 库才能实现的功能。通常不再需要流行的库，如 jQuery、Lodash 和 Moment。

# 第三方库的缺点

第三方库可以帮助旧版浏览器或较新功能，但它们也有一些成本：

需要下载更多的代码

使用库会增加浏览器加载的 JavaScript 量。无论是与您的应用捆绑在一起还是从内容交付网络（CDN）单独加载，浏览器仍然需要下载它。这可能导致加载时间更长，移动设备的电池使用量更高。

增加的风险

即使是流行的开源库，也可能被放弃。当发现漏洞或安全问题时，并不能保证会有更新。通常情况下，浏览器由大公司支持（主要浏览器来自谷歌、Mozilla、苹果和微软），更可能修复这些问题。

这并不是说第三方库*不好*。它们也有许多好处，特别是在需要支持旧版浏览器时。就像软件开发中的其他一切一样，库的使用是一个平衡的过程。

# 本书适合谁

本书适合有一定 JavaScript 经验的软件开发者，希望在 Web 平台上获得最大收益。

它假设您对 JavaScript 语言本身有良好的了解：语法、语言特性和标准库函数。您还应了解用于构建交互式、基于浏览器的 JavaScript 应用程序的 DOM API 的工作原理。

本书中包含大量的配方，适合各种技能和经验水平的开发者。

# 本书内容概述

每章包含一系列*配方*——用于完成特定任务的代码示例。每个配方分为三个部分：

问题

描述配方解决的问题。

解决方案

包含实现配方解决方案的代码和解释。

讨论

关于主题的更深入讨论。本节可能包含额外的代码示例，并与其他技术进行比较。

代码示例和实时演示可在配套网站上找到，*[*https://WebAPIs.info*](https://WebAPIs.info)*。

# 其他资源

网络的特性使其随时都在变化。在线上有许多出色的资源可帮助澄清可能出现的任何问题。

## CanIUse.com

在撰写本书时，一些 API 仍在开发中或处于“实验”阶段。请注意查看使用这些 API 的食谱中的兼容性说明。对于大多数功能，您可以在 [*https://CanIUse.com*](https://CanIUse.com) 上查看最新的兼容性数据。您可以按功能名称搜索，并查看有关支持该 API 的浏览器版本以及特定浏览器版本的任何限制或注意事项的最新信息。

## MDN Web Docs

[MDN Web Docs](https://oreil.ly/rLxi7) 是所有网络相关内容的事实标准 API 文档。它详细涵盖了本书中所有的 API，以及诸如 CSS 和 HTML 等其他主题。文档中包含深入的文章、教程和 API 规范。

## 规范

如有疑问，特性或 API 的规范是权威资源。它们可能不是最令人兴奋的阅读材料，但在查找有关边缘情况或预期行为的详细信息时是一个很好的地方。

不同的 API 有不同的标准，但大多数可以在 [Web 超文本应用技术工作组 (WHATWG)](https://oreil.ly/PR0x7) 或 [万维网联盟 (W3C)](https://oreil.ly/dFokl) 找到。

ECMAScript 的标准（指定 JavaScript 语言中的特性）由 Ecma 国际技术委员会 39 维护和开发，更为人所知的是 [TC39](https://tc39.es)。

# 本书使用的约定

本书使用以下排版约定：

*Italic*

指示新术语、网址、电子邮件地址、文件名和文件扩展名。

`Constant width`

用于程序清单，以及在段落内引用程序元素，例如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

**`Constant width bold`**

显示用户应按字面意思键入的命令或其他文本。

*`Constant width italic`*

显示应由用户提供的值或由上下文确定的值替换的文本。

###### Tip

此元素表示提示或建议。

###### Note

此元素表示一般性说明。

###### Warning

此元素表示警告或注意事项。

# 使用代码示例

补充资料（代码示例、练习等）可在 [*https://github.com/joeattardi/web-api-cookbook*](https://github.com/joeattardi/web-api-cookbook) 下载。还请查看 [配套网站](https://WebAPIs.info)，在那里本书中许多代码示例和食谱都扩展为完整的、实时的工作示例。

如果您有技术问题或在使用代码示例时遇到问题，请发送电子邮件至*bookquestions@oreilly.com*。

这本书旨在帮助您完成工作。一般来说，如果书中提供了示例代码，您可以将其用于您的程序和文档。除非您复制了代码的重大部分，否则不需要联系我们。例如，编写一个使用这本书中的几个代码块的程序不需要许可。出售或分发 O’Reilly 书籍中的示例代码需要许可。回答一个问题时引用这本书并引用示例代码不需要许可。将大量示例代码从这本书中整合到您的产品文档中确实需要许可。

我们感激，但通常不需要归属。归属通常包括书名、作者、出版商和 ISBN。例如：“*Web API 食谱* 由 Joseph Attardi 著（O’Reilly 出版）。版权所有 2024 年 Joe Attardi, 978-1-098-15069-3。”

如果您认为您使用示例代码的情况超出了公平使用或上述许可，随时可以联系我们：*permissions@oreilly.com*。

# O’Reilly 在线学习

###### 注意

40 多年来，[*O’Reilly Media*](https://oreilly.com) 提供技术和商业培训、知识和洞察力，以帮助公司成功。

我们的独特网络的专家和创新者通过书籍、文章和我们的在线学习平台分享他们的知识和专业知识。O’Reilly 的在线学习平台为您提供对实时培训课程、深入学习路径、互动编码环境和 O’Reilly 及其他 200 多个出版商的文本和视频的大量访问权限。有关更多信息，请访问 [*https://oreilly.com*](https://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题通知给出版商：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-889-8969（美国或加拿大）

+   707-827-7019（国际或当地）

+   707-829-0104（传真）

+   *support@oreilly.com*

+   [*https://www.oreilly.com/about/contact.html*](https://www.oreilly.com/about/contact.html)

我们有一本专门的网页，用来列出校正、示例和任何其他信息。您可以在 [*https://oreil.ly/web-api-cookbook*](https://oreil.ly/web-api-cookbook) 上访问此页面。

有关我们的书籍和课程的新闻和信息，请访问 [*https://oreilly.com*](https://oreilly.com)。

找到我们在 LinkedIn 上：[*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media)

观看我们在 YouTube 上的节目：[*https://youtube.com/oreillymedia*](https://youtube.com/oreillymedia)

# 致谢

首先，我衷心感谢我的家庭和朋友们的支持，尤其是我的妻子 Liz 和儿子 Benjamin，感谢他们忍受我的不断敲击键盘。当我处于高峰状态时，我习惯性地打字非常快速且大声。

感谢阿曼达·奎恩（Amanda Quinn），高级内容采购编辑，让我成为奥莱利（O’Reilly）的作者。多年来我读了无数本奥莱利的书，从未想过有朝一日我也会写自己的一本书。同时也要感谢路易丝·科里根（Louise Corrigan）介绍我认识阿曼达，并启动了整个过程（她几年前与我合作出版了我的第一本书！）。

特别感谢维吉尼亚·威尔逊（Virginia Wilson），高级开发编辑，她在整个书写过程中的指导至关重要，并定期会面以确保一切顺利进行。

我还要感谢本书的出色技术审阅者：马丁·道登（Martine Dowden）、沙尔克·尼斯林（Schalk Neethling）、莎拉·舒克（Sarah Shook）和亚当·斯科特（Adam Scott）。在他们宝贵的反馈下，这本书变得更加出色。

最后，我要向设计和开发这些现代 Web API 的团队们致以崇高的敬意。没有他们，这本书将无法问世！
