# 前言

这本书包含了我们在多年构建 React 应用程序中发现有用的一系列代码。就像你在厨房中使用的食谱一样，我们设计它们作为你自己代码的起点或灵感。你应该根据自己的情况调整它们，并用更适合你需求的成分（比如示例服务器）替换掉它们。这些配方涵盖了从一般的 Web 开发提示到你可以概括为库的较大代码片段。

大部分的配方都是用 Create React App 构建的，因为这是现在大多数 React 项目的常见起点。转换每个配方以在 Preact 或 Gatsby 中使用应该很简单。

为了保持代码的简洁，我们通常使用钩子和函数而不是类组件。我们还使用 Prettier 工具在整个过程中应用标准的代码格式化。除了更窄的缩进和行长度外，我们使用了 Prettier 的默认选项，以便将代码整齐地打印到页面上。你应该调整代码格式以符合你的首选标准。

在创建这些配方时我们使用了许多库：

| Tool/library | 描述 | 版本 |
| --- | --- | --- |
| Apollo Client | GraphQL 客户端 | 3.3.19 |
| axios | HTTP 库 | 0.21.1 |
| chai | 单元测试支持库 | 4.3.0 |
| chromedriver | 浏览器自动化工具 | 88.0.0 |
| Create React App | 生成 React 应用的工具 | 4.0.3 |
| Cypress | 自动化测试系统 | 7.3.0 |
| Cypress Axe | 自动化无障碍测试 | 0.12.2 |
| Gatsby | 生成 React 应用的工具 | 3.4.1 |
| GraphQL | API 查询语言 | 15.5.0 |
| jsx-a11y | 用于无障碍的 ESLint 插件 | 6.4.1 |
| Material-UI | 组件库 | 4.11.4 |
| Node | JavaScript 运行时 | v12.20.0 |
| npm | Node 包管理器 | 6.14.8 |
| nvm | 运行多个 Node 环境的工具 | 0.33.2 |
| nwb | 生成 React 应用的工具 | 0.25.x |
| Next.js | 生成 React 应用的工具 | 10.2.0 |
| Preact | 轻量级类 React 框架 | 10.3.2 |
| Preact Custom Elements | 创建自定义元素的库 | 4.2.1 |
| preset-create-react-app | Storybook 插件 | 3.1.7 |
| Rails | Web 开发框架 | 6.0.3.7 |
| Razzle | 生成 React 应用的工具 | 4.0.4 |
| React | Web 框架 | 17.0.2 |
| React Media | 在 React 代码中使用媒体查询 | 1.10.0 |
| React Router (DOM) | 管理 React 路由的库 | 5.2.0 |
| React Testing Library | 用于 React 的单元测试库 | 11.1.0 |
| react-animations | React CSS 动画库 | 1.0.0 |
| React Focus Lock | 捕获键盘焦点的库 | 2.5.0 |
| react-md-editor | Markdown 编辑器 | 3.3.6 |
| React-Redux | Redux 的 React 支持库 | 7.2.2 |
| Redux | 状态管理库 | 4.0.5 |
| Redux-Persist | 存储 Redux 状态的库 | 6.0.0 |
| Ruby | Rails 使用的语言 | 2.7.0p0 |
| selenium-webdriver | 浏览器测试框架 | 4.0.0-beta.1 |
| Storybook | 组件库系统 | 6.2.9 |
| TweenOne | React 动画库 | 2.7.3 |
| Typescript | JavaScript 的类型安全扩展 | 4.1.2 |
| Webpacker | 用于向 Rails 应用程序添加 React 的工具 | 4.3.0 |
| Workbox | 用于创建服务工作者的库 | 5.1.3 |
| Yarn | 另一个 Node 包管理器 | 1.22.10 |

# 本书使用的约定

本书使用以下排版约定：

*Italic*

指示新术语、URL、电子邮件地址、文件名和文件扩展名。

`Constant width`

用于程序清单，以及在段落内引用程序元素，例如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

**`Constant width bold`**

显示用户应按原样输入的命令或其他文本。

*`Constant width italic`*

显示应由用户提供值或由上下文确定值的文本。

此元素表示提示或建议。

此元素表示一般注意事项。

此元素指示警告或注意事项。

# 使用代码示例

可以下载补充材料（代码示例、练习等）[*https://github.com/dogriffiths/ReactCookbook-source*](https://github.com/dogriffiths/ReactCookbook-source)。

如果您有技术问题或使用代码示例的问题，请发送电子邮件至*bookquestions@oreilly.com*。

本书旨在帮助您完成工作。一般情况下，如果本书提供示例代码，您可以在您的程序和文档中使用它。除非您复制了代码的大部分内容，否则无需征得我们的许可。例如，编写使用本书多个代码片段的程序不需要许可。销售或分发来自 O'Reilly 书籍的示例需要许可。引用本书并引用示例代码来回答问题不需要许可。将本书中大量示例代码整合到您产品的文档中需要许可。

我们感谢，但通常不要求署名。署名通常包括标题、作者、出版社和 ISBN。例如：“*React Cookbook* by David Griffiths and Dawn Griffiths (O’Reilly)。版权所有 2021 年 Dawn Griffiths 和 David Griffiths，978-1-492-08584-3。”

如果您认为使用的代码示例超出了公平使用范围或上述许可，请随时联系我们*permissions@oreilly.com*。

# O’Reilly 在线学习

40 多年来，[*O’Reilly Media*](http://oreilly.com) 提供技术和商业培训、知识和见解，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专业知识。O'Reilly 的在线学习平台为您提供按需访问的现场培训课程、深入学习路径、交互式编码环境以及来自 O'Reilly 和 200 多个其他出版商的大量文本和视频。有关更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将关于本书的评论和问题寄给出版商：

+   O'Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（在美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为本书制作了一个网页，列出勘误表、示例和任何其他信息。您可以访问[*https://oreil.ly/react-cb*](https://oreil.ly/react-cb)。

发送电子邮件至*bookquestions@oreilly.com*以对本书提出评论或技术问题。

关于我们的书籍和课程的新闻和信息，请访问[*http://oreilly.com*](http://oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

在 YouTube 上观看我们：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)

# 致谢

我们要感谢我们非常耐心的编辑 Corbin Collins 在过去一年里对写作过程中的帮助和建议。他的冷静和幽默在写作过程中起到了稳定作用。我们还要感谢 Amanda Quinn，O'Reilly Media 的高级内容获取编辑，委托出版这本书，以及 O'Reilly 的制作团队 Danny Elfanbaum，为实现实体和电子版本所做的贡献。

特别感谢 Sam Warner 和 Mark Hobson 对本书内容进行的非常严格的审查。

我们也感谢那些为支持 React 生态系统的许多开源库工作的开发人员。我们对他们所有人都心怀感激，特别是他们对错误报告或求助的迅速响应。

如果你觉得这些配方有用，主要是因为这些人的工作。如果你发现代码或文本中有错误，那完全是我们的责任。
