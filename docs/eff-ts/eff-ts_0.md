# 前言

2016 年春天，我访问了我在 Google 旧同事 Evan Martin 在旧金山办公室，并问他最近对什么感到兴奋。多年来，我问过他同样的问题很多次，因为答案多种多样且不可预测，但总是很有趣：C++构建工具、Linux 音频驱动程序、在线填字游戏、emacs 插件。这次，Evan 对 TypeScript 和 Visual Studio Code 感到兴奋。

我很惊讶！我以前听说过 TypeScript，但我只知道它是由微软创建的，而且我错误地认为它与.NET 有关。作为一名终身 Linux 用户，我无法相信 Evan 竟然加入了微软团队。

然后 Evan 向我展示了 vscode 和 TypeScript playground，我立即被它们征服了。一切都如此迅速，代码智能使得建立类型系统的心理模型变得轻而易举。多年来，我在 Closure Compiler 的 JSDoc 注释中编写类型注解，而现在感觉就像是真正有效的带有类型的 JavaScript。而且微软还基于 Chromium 构建了一个跨平台文本编辑器？也许这是一个值得学习的语言和工具链。

我最近加入了 Sidewalk Labs，并开始编写我们的第一个 JavaScript 代码。代码库仍然很小，Evan 和我能够在接下来的几天内将其全部转换为 TypeScript。

从那以后，我一直着迷。TypeScript 不仅仅是一个类型系统，它还带来了一整套快速易用的语言服务。累积效应是，TypeScript 不仅使 JavaScript 开发更加安全，而且更加有趣！

# 本书适合谁阅读

*Effective* 系列书籍旨在成为其领域的“标准第二本书”。如果你之前有一些实际使用 JavaScript 和 TypeScript 的经验，你将能够从 *Effective TypeScript* 中获得最大的收益。我这本书的目标不是教你 TypeScript 或 JavaScript，而是帮助你从初级或中级用户进阶到专家级别。本书中的内容通过帮助你建立 TypeScript 及其生态系统如何工作的心理模型，使你意识到应避免的陷阱和问题，并指导你以最有效的方式利用 TypeScript 的多种能力来实现这一目标。而参考书将解释一种语言允许你以五种方式执行 X，*Effective* 书将告诉你应该选择其中哪一种以及为什么选择它。

过去几年中，TypeScript 已经快速发展，但我希望它已经足够稳定，以至于本书内容将在未来数年内仍然有效。本书主要关注语言本身，而不涉及任何框架或构建工具。你不会在本书中找到如何在 TypeScript 中使用 React 或 Angular 的示例，也不会找到如何配置 TypeScript 与 webpack、Babel 或 Rollup 配合使用的内容。本书的建议应该适用于所有 TypeScript 用户。

# 为什么我写这本书

当我刚开始在谷歌工作时，我得到了第三版《Effective C++》的副本。这本书不像我读过的任何其他编程书籍。它没有试图对初学者友好，也没有试图成为语言的完全指南。它不是告诉你 C++ 的不同特性是什么，而是告诉你如何使用它们以及不应该如何使用它们。它通过数十个短小的具体条目来做到这一点，每个条目都有具体的例子作为动机。

在日常使用语言时阅读所有这些示例的效果是明显的。我以前使用过 C++，但这是我第一次感到对它感到舒适，并知道如何思考它所提供的选择。后来的几年里，我读了《Effective Java》和《Effective JavaScript》时也有类似的经历。

如果你已经熟悉在几种不同的编程语言中工作，那么直接深入到一种新语言的奇怪角落可能是挑战你思维模式并了解其不同之处的有效方法。从撰写本书中，我学到了大量有关 TypeScript 的知识。希望你阅读本书时也能有同样的体验！

# 本书的组织方式

本书是一系列“条目”，每篇都是一篇短小的技术文章，向您提供有关 TypeScript 某些方面的具体建议。这些条目按主题分组成章节，但请随意跳跃阅读您最感兴趣的部分。

每个条目的标题都传达了关键的要点。当你使用 TypeScript 时，这些是你应该记住的东西，因此浏览目录以记住它们是值得的。例如，如果你正在撰写文档，并且有一种不应该写类型信息的感觉，那么你会知道去阅读 第 30 条：不要在文档中重复类型信息。

每一项的文本都以标题的建议为动机，并通过具体例子和技术论证加以支持。这本书几乎每个观点都通过示例代码进行了演示。我倾向于通过查看示例和略读散文来阅读技术书籍，我想你也会做类似的事情。希望你能阅读散文和解释！但如果你只是略读示例，主要观点仍然应该能够传达。

阅读完条目后，你应该理解为什么它会帮助你更有效地使用 TypeScript。你还会知道足够的信息来判断它是否适用于你的情况。《Effective C++》的作者斯科特·迈尔斯给出了一个令人难忘的例子。他遇到了一个团队的工程师，他们编写的软件在导弹上运行。他们知道可以忽略他关于防止资源泄漏的建议，因为他们的程序在导弹击中目标并且硬件爆炸时总会终止。我不知道有没有带有 JavaScript 运行时的导弹，但詹姆斯·韦伯空间望远镜有一个，所以永远不知道！

最后，每个条目都以“Things to Remember”结束。这些是总结条目的几个要点。如果你在浏览中，可以阅读这些以了解条目的内容和是否希望进一步阅读。你仍然应该阅读条目！但总结在紧急情况下也足够了。

# TypeScript 代码示例中的约定

所有代码示例都是 TypeScript，除非从上下文中明确它们是 JSON、GraphQL 或其他语言。使用 TypeScript 的体验主要涉及与编辑器的交互，在印刷中会带来一些挑战。我采用了一些约定来解决这些问题。

大多数编辑器使用波浪线下划线显示错误。要查看完整的错误消息，你可以悬停在下划线文本上。为了指示代码示例中的错误，我在发生错误的地方的注释行中放置波浪线：

```
let str = 'not a number';
let num: number = str;
 // ~~~ Type 'string' is not assignable to type 'number'
```

我偶尔会编辑错误消息以提高清晰度和简洁性，但我从不删除错误。如果你将代码示例复制/粘贴到编辑器中，你应该得到精确的错误指示，没有多余的也没有少了的。

为了引起注意，我使用了 `// OK`：

```
let str = 'not a number';
let num: number = str as any;  // OK
```

你应该能够在编辑器中悬停在符号上查看 TypeScript 认为它的类型。为了在文本中表示这一点，我使用以“type is”开头的注释：

```
let v = {str: 'hello', num: 42};  // Type is { str: string; num: number; }
```

类型用于行中的第一个符号（在本例中是`v`）或函数调用的结果：

```
'four score'.split(' ');  // Type is string[]
```

这与你的编辑器中看到的字符一致。对于函数调用，你可能需要赋值给临时变量以查看类型。

我偶尔会引入无操作语句来指示代码中特定行的变量类型：

```
function foo(x: string|string[]) {
  if (Array.isArray(x)) {
    x;  // Type is string[]
  } else {
    x;  // Type is string
  }
}
```

`x;`行仅用于展示条件分支中的类型。你不需要（也不应该）在自己的代码中包含这样的语句。

除非另有说明或上下文明确，代码示例都应使用`--strict`标志进行检查。所有示例都经过 TypeScript 3.7.0-beta 的验证。

# 本书中使用的排版约定

以下是本书中使用的排版约定：

*Italic*

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`Constant width`

用于程序清单，以及段落中引用程序元素，如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

**`Constant width bold`**

显示用户应该按原样键入的命令或其他文本。

*`Constant width italic`*

显示应由用户提供值或由上下文确定值替换的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般注释。

###### 警告

此元素表示警告或注意事项。

# 使用代码示例

可以通过[*https://github.com/danvk/effective-typescript*](https://github.com/danvk/effective-typescript)下载补充材料（代码示例、练习等）。

如果您有技术问题或在使用代码示例时遇到问题，请发送电子邮件至*bookquestions@oreilly.com*.

这本书旨在帮助您完成工作。一般而言，如果本书提供了示例代码，您可以在您的程序和文档中使用它。除非您复制了代码的大部分，否则无需联系我们以获取许可。例如，编写一个使用本书几个代码块的程序不需要许可。销售或分发 O’Reilly 书籍的示例则需要许可。通过引用本书并引用示例代码回答问题不需要许可。将本书的大量示例代码合并到产品文档中确实需要许可。

我们感激，但通常不要求署名。署名通常包括标题、作者、出版商和 ISBN。例如：“*Effective TypeScript* by Dan Vanderkam (O’Reilly). Copyright 2020 Dan Vanderkam, 978-1-492-05374-3.”

如果您认为您对代码示例的使用超出了合理使用范围或上述许可，请随时通过*permissions@oreilly.com*联系我们。

# O’Reilly 在线学习

###### 注意

40 多年来，[*O’Reilly Media*](http://oreilly.com) 提供技术和业务培训、知识和见解，帮助企业成功。

我们独特的专家和创新者网络通过书籍、文章、会议以及我们的在线学习平台分享他们的知识和专长。O’Reilly 的在线学习平台为您提供按需访问的实时培训课程、深度学习路径、交互式编码环境，以及来自 O’Reilly 和其他 200 多个出版商的大量文本和视频内容。更多信息，请访问[*http://oreilly.com*](http://oreilly.com).

# 如何联系我们

请将有关本书的评论和问题发送给出版商：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938 (在美国或加拿大)

+   707-829-0515 (国际或本地)

+   707-829-0104 (传真)

您可以访问此书的网页，我们在那里列出勘误、示例和任何额外信息，网址为[*https://oreil.ly/Effective_TypeScript*](https://oreil.ly/Effective_TypeScript).

要对本书提出评论或提出技术问题，请发送电子邮件至*bookquestions@oreilly.com*.

有关我们的书籍、课程、会议和新闻的更多信息，请查看我们的网站[*http://www.oreilly.com*](http://www.oreilly.com).

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在 Twitter 关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

在 YouTube 上观看我们：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)

# 致谢

这本书能够问世，离不开许多帮助过我的人。感谢埃文·马丁（Evan Martin）介绍我认识 TypeScript 并教我如何思考它。感谢道威·奥辛加（Douwe Osinga）介绍我与 O'Reilly 的联系并对这个项目给予支持。感谢布雷特·斯拉特金（Brett Slatkin）在结构上的建议，并向我展示我认识的人也能写出一本*Effective*书籍。感谢斯科特·迈耶斯（Scott Meyers）提出这种格式，并感谢他的“Effective *Effective* Books”博客文章，为本书提供了重要指导。

感谢我的评审人员，里克·巴塔格林（Rick Battagline）、瑞安·卡瓦纳（Ryan Cavanaugh）、鲍里斯·切尔尼（Boris Cherny）、雅科夫·费恩（Yakov Fain）、杰西·哈莱特（Jesse Hallett）和杰森·基利安（Jason Killian）。感谢所有在 Sidewalk 和我一起多年学习 TypeScript 的同事们。感谢所有帮助这本书问世的 O’Reilly 同事们：安吉拉·鲁菲诺（Angela Rufino）、詹妮弗·波洛克（Jennifer Pollock）、德博拉·贝克（Deborah Baker）、尼克·亚当斯（Nick Adams）和贾斯敏·奎特因（Jasmine Kwityn）。感谢 TypeScript 纽约市的全体成员，杰森、奥尔塔和基里尔，以及所有演讲者。许多条款灵感来自 Meetup 上的演讲，如下所列：

+   第 3 条受到埃文·马丁（Evan Martin）的一篇博文的启发，我在初学 TypeScript 时觉得尤为启发。

+   第 7 条受安德斯（Anders）在 TSConf 2018 上关于结构类型和`keyof`关系的演讲以及杰西·哈莱特（Jesse Hallett）在 2019 年 4 月 TypeScript 纽约市 Meetup 上的演讲启发。

+   巴萨拉特（Basarat）的指南以及 Stack Overflow 上 DeeV 和 GPicazo 的有用回答对撰写第 9 条至关重要。

+   第 10 条建立在《Effective JavaScript》（Addison-Wesley）第 4 条中类似的建议之上。

+   我受到 2019 年 8 月 TypeScript 纽约市 Meetup 上围绕这个主题的大量混淆启发，因此写下了第 11 条。

+   第 13 条受 Stack Overflow 上关于`type` vs. `interface`的几个问题的极大帮助。杰西·哈莱特（Jesse Hallett）建议了关于可扩展性的表述方式。

+   雅各布·巴斯金（Jacob Baskin）在第 14 条的早期反馈和鼓励。

+   第 19 条受到提交到 r/typescript subreddit 的几个代码示例的启发。

+   第 26 条基于我在 Medium 上的写作以及我在 2018 年 10 月 TypeScript 纽约市 Meetup 上的演讲。

+   第 28 条基于 Haskell 中的常见建议（“让非法状态不可表示”）。法国航空 447 航班的故事灵感来自杰夫·怀斯（Jeff Wise）2011 年在《Popular Mechanics》发表的令人难以置信的文章。

+   第 29 条基于我在 Mapbox 类型声明中遇到的问题。杰森·基利安（Jason Killian）建议了标题的措辞。

+   关于命名的建议在第 36 条中很常见，但这个特定的表述灵感来自丹·诺斯（Dan North）在《97 Things Every Programmer Should Know》（O’Reilly）中的短文。

+   第 37 条目受到 Jason Killian 在 2017 年 9 月第一个 TypeScript NYC Meetup 上的演讲的启发。

+   第 41 条目基于 TypeScript 2.1 的发布说明。术语“evolving any”在 TypeScript 编译器之外并不广泛使用，但我觉得给这种不寻常的模式取个名字很有用。

+   第 42 条目受到 Jesse Hallett 博客文章的启发。第 43 条目得益于 Titian Cernicova Dragomir 在 TypeScript 问题#33128 中的反馈。

+   第 44 条目基于 York Yao 对`type-coverage`工具的工作。我想要类似的东西，它已经存在了！

+   第 46 条目基于我在 2017 年 12 月在 TypeScript NYC Meetup 上的演讲。

+   第 50 条目深受 David Sheldrick 在*Artsy*博客上关于条件类型的文章的启发，这些文章极大地解开了我对该主题的困惑。

+   第 51 条目受到 Steve Faulkner（即 southpolesteve）在 2019 年 2 月 Meetup 上的演讲的启发。

+   第 52 条目基于我在 Medium 上的写作和 typings-checker 工具的工作，后来被整合到了 dtslint 中。

+   第 53 条目受到 Kat Busch 在 Medium 上关于 TypeScript 各种枚举类型的帖子以及 Boris Cherny 在《Programming TypeScript》（O'Reilly）中关于该主题的写作的启发/强化。

+   第 54 条目受到我和同事对这个主题的困惑的启发。Anders 在 TypeScript PR #12253 中给出了最终的解释。

+   编写第 55 条目时，MDN 文档对我至关重要。

+   第 56 条目松散地基于《Effective JavaScript》（Addison-Wesley）的第 35 条目。

+   第八章基于我迁移老化的 dygraphs 库的经验。

我找到了许多导致本书的博客文章和演讲，都来源于出色的 r/typescript subreddit。我特别感谢在那里提供了代码示例的开发人员，这些示例对理解 TypeScript 初学者常见问题至关重要。感谢 Marius Schulz 提供的 TypeScript Weekly 通讯。虽然它偶尔才是每周的，但它始终是一个极好的资料来源，也是跟进 TypeScript 的好方法。感谢 Anders、Daniel、Ryan 以及 Microsoft 的整个 TypeScript 团队在演讲和所有问题反馈上的支持。我的大多数问题都是误解，但没有什么比提交一个 bug 然后立即看到 Anders Hejlsberg 本人修复它更令人满足！最后，感谢 Alex 在整个项目期间的支持和理解，包括我完成这个项目所需的所有工作假期、早晨、晚上和周末。
