# 前言

本书涵盖了 JavaScript 语言以及 Web 浏览器和 Node 实现的 JavaScript API。我为一些具有先前编程经验的读者编写了这本书，他们想要学习 JavaScript，也为已经使用 JavaScript 的程序员编写了这本书，但希望将他们的理解提升到一个新的水平，并真正掌握这门语言。我写这本书的目标是全面和权威地记录 JavaScript 语言，并深入介绍 JavaScript 程序可用的最重要的客户端和服务器端 API。因此，这是一本长篇详细的书。然而，我希望它会奖励仔细学习，并且您花在阅读上的时间将很容易以更高的编程生产力形式收回。

本书的早期版本包括了一个全面的参考部分。我不再认为在印刷形式中包含这些材料是有意义的，因为在网上很容易找到最新的参考材料。如果您需要查找与核心或客户端 JavaScript 相关的任何内容，我建议您访问[MDN 网站](https://developer.mozilla.org)。对于服务器端 Node API，我建议您直接访问源并查阅[Node.js 参考文档](https://nodejs.org/api)。

# 本书中使用的约定

我在本书中使用以下排版约定：

*斜体*

用于强调和指示术语的首次使用。*斜体*也用于电子邮件地址，URL 和文件名。

`固定宽度`

用于所有 JavaScript 代码和 CSS 和 HTML 列表，通常用于编程时需要字面输入的任何内容。

*`固定宽度斜体`*

有时用于解释 JavaScript 语法。

**`固定宽度粗体`**

显示用户应该按照字面意义输入的命令或其他文本

###### 注意

此元素表示一般说明。

###### 重要

此元素表示警告或注意事项。

# 示例代码

本书的补充材料（代码示例，练习等）可在以下网址下载：

+   [*https://oreil.ly/javascript_defgd7*](https://oreil.ly/javascript_defgd7)

本书旨在帮助您完成工作。一般情况下，如果本书提供示例代码，您可以在您的程序和文档中使用它。除非您复制了代码的大部分内容，否则无需联系我们请求许可。例如，编写一个使用本书多个代码块的程序不需要许可。销售或分发 O'Reilly 图书中的示例需要许可。引用本书并引用示例代码回答问题不需要许可。将本书中大量示例代码合并到产品文档中需要许可。

我们感谢，但通常不要求署名。署名通常包括标题，作者，出版商和 ISBN。例如：“*JavaScript: The Definitive Guide*，第七版，作者 David Flanagan（O'Reilly）。版权所有 2020 年 David Flanagan，978-1-491-95202-3。”

如果您认为您使用的代码示例超出了合理使用范围或上述许可，请随时通过*permissions@oreilly.com*与我们联系。


# 致谢

许多人在创作本书时提供了帮助。我要感谢我的编辑 Angela Rufino，她让我保持在正确的轨道上，对我错过的截止日期的耐心。也感谢我的技术审阅者：Brian Sletten，Elisabeth Robson，Ethan Flanagan，Maximiliano Firtman，Sarah Wachs 和 Schalk Neethling。他们的评论和建议使这本书变得更好。

O’Reilly 的制作团队一如既往地出色：Kristen Brown 管理了制作过程，Deborah Baker 担任制作编辑，Rebecca Demarest 绘制了图表，Judy McConville 创建了索引。

本书的编辑、审阅者和贡献者包括：Andrew Schulman，Angelo Sirigos，Aristotle Pagaltzis，Brendan Eich，Christian Heilmann，Dan Shafer，Dave C. Mitchell，Deb Cameron，Douglas Crockford，Dr. Tankred Hirschmann，Dylan Schiemann，Frank Willison，Geoff Stearns，Herman Venter，Jay Hodges，Jeff Yates，Joseph Kesselman，Ken Cooper，Larry Sullivan，Lynn Rollins，Neil Berkman，Mike Loukides，Nick Thompson，Norris Boyd，Paula Ferguson，Peter-Paul Koch，Philippe Le Hegaret，Raffaele Cecco，Richard Yaker，Sanders Kleinfeld，Scott Furman，Scott Isaacs，Shon Katzenberger，Terry Allen，Todd Ditchendorf，Vidur Apparao，Waldemar Horwat 和 Zachary Kessin。

撰写第七版使我在许多深夜远离了家人。我爱他们，感谢他们忍受我的缺席。

David Flanagan，2020 年 3 月
