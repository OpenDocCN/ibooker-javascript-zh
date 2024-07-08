# 前言

在过去的十年中，Node.js 已经从新奇变成了新应用的事实标准平台。在此期间，我有机会帮助来自世界各地的成千上万的 Node.js 开发者定位自己，找到成功的道路。我见过 Node.js 用于各种用途。真的：有人甚至用 Node.js 构建了低级可引导操作系统。

在我在旧金山创建的 SFNode meetup 上，我们有一个超级演讲者，他比任何人都多发言。你猜对了：Thomas Hunter II，这本书的作者。虽然你可能可以用 Node.js 做任何事情，但有些实际的事情特别适合用 Node.js 来完成。在今天的云优先世界中，大多数系统都变成了分布式系统。在这本书和我在 SFNode 和全球各地看到 Thomas 进行的无数次演讲中，务实主义至上。这本书充满了经验测试过的、实用的指导，帮助你从今天的位置走向明天的目标。

JavaScript 语言使我们作为开发者能够以思想的速度创造。它需要很少的仪式感，我们编写的代码通常足够简单，手工编写比生成更为高效。JavaScript 的这种简单之美与 Node.js 完美契合。我们经常称之为 Node，它故意保持简约。Ryan Dahl，它的创造者，编写 Node 来构建比任何人习惯的应用服务器更简单和更快的应用。结果甚至超出了我们最疯狂的梦想。Node.js 的简便和简单性使你能够以前所未有的方式创建、验证和创新，这在 10 年前根本不可能实现。

在学习 Node.js 之前，我是一个全栈开发者，使用 JavaScript 构建交互式的基于 Web 的体验，使用 Java 提供 API 和后端服务。我会沉浸在 JavaScript 的创造性流程中，然后完全转向，将所有内容翻译成 Java 的对象模型。多么浪费时间啊！当我发现 Node.js 时，我终于能够在客户端和服务器上都有效率和有效地迭代。我真的放下一切，卖掉了房子，搬到旧金山与 Node.js 一起工作。

我用 Node.js 构建了数据聚合系统、社交媒体平台和视频聊天。然后我帮助 Netflix、PayPal、沃尔玛，甚至 NASA 学会如何有效地使用这个平台。JavaScript 的 API 很少是人们最大的挑战。最让人困惑的是异步编程模型。如果你不理解你正在使用的工具，你如何能够期望以最佳的结果使用这些工具呢？异步编程要求你像计算机系统一样思考，而不是按顺序执行的线性脚本。这种异步性是一个良好分布式系统的核心。

当 Thomas 要求我审阅这本书的目录以确保他已经涵盖了所有内容时，我注意到关于扩展性的部分以集群模块的概述开始。我立即将其标记为一个关注的重点。集群模块的创建是为了实现单实例并发，可以暴露到系统的单个端口上。我见过新手们使用这个功能并假设由于并发可能是可取的，集群模块就是他们需求的正确工具。在分布式系统中，实例级别的并发通常是浪费时间的。幸运的是，Thomas 和我意见一致，这导致我们在 SFNode 的顶级演讲者的精彩对话。

因此，当你作为一个 Node.js 开发者和分布式系统开发者建立你的能力时，请花时间了解你系统中的约束和机会。Node.js 拥有非常高效的 I/O 能力。我见过旧服务被移除并用 Node.js 实现后，下游系统变得不堪重负的情况。这些系统原本可以承受这些服务，添加一个简单的 Node.js 代理可以解决大多数问题，直到下游服务被更新或替换。

使用 Node 开发的便捷性可以让你尝试很多事情。不要害怕抛弃代码并重新开始。Node.js 的开发在迭代中茁壮成长。分布式系统让我们在服务级别隔离和封装逻辑，然后可以通过负载平衡来验证整个系统的性能。但不要仅仅听我的话。这本书的页面会向你展示如何做到这一点最有效。

在学习的过程中享受乐趣，并分享你所学到的东西。

Dan Shaw（[@dshaw](https://twitter.com/dshaw)）

创始人兼首席技术官，NodeSource

Node.js 公司

始终相信 Node.js