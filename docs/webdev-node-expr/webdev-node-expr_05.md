# 第五章：质量保证

*质量保证*是一个容易让开发人员心生畏惧的短语——这是不幸的。毕竟，您不是想要制造高质量的软件吗？当然是。因此，关键不在于最终目标，而是于政策的处理。我发现在 Web 开发中存在两种常见情况：

大型或财力雄厚的组织

通常会有一个质量保证部门，不幸的是，质量保证和开发之间会出现对抗性关系。这是可能发生的最糟糕的事情。虽然两个部门都在为同一个目标而战，但质量保证通常将成功定义为发现更多的缺陷，而开发则将成功定义为生成更少的缺陷，这就成为冲突和竞争的基础。

小型组织和预算有限的组织

通常情况下，没有质量保证部门；开发人员预计要同时兼顾建立质量保证和开发软件的双重角色。这并不是想象或利益冲突的荒谬伸展。然而，质量保证是与开发非常不同的学科，吸引不同的个性和才能。这不是一个不可能的情况，当然也有开发人员具备质量保证思维方式，但是在截止日期逼近时，通常是质量保证受到短缺待遇，对项目的影响不利。

对于大多数真实世界的努力来说，需要多种技能，并且越来越难成为所有这些技能的专家。然而，对于您不直接负责的领域有一定的能力将使您对团队更有价值，并使团队运作更有效。开发人员获取质量保证技能是一个很好的例子：这两个领域紧密相连，跨学科的理解极为重要。

还有一个常见的做法是将传统上由质量保证完成的活动转移到开发部门，使开发人员负责质量保证。在这种范式中，专门从事质量保证的软件工程师几乎像开发人员的顾问，帮助他们将质量保证融入其开发工作流程中。无论质量保证角色是分开还是整合的，理解质量保证对开发人员都是有益的。

本书不是针对质量保证专业人士的，而是面向开发人员的。因此，我的目标不是让您成为质量保证专家，而是为您提供一些在该领域获得经验的机会。如果您的组织有专门的质量保证人员，那么您将更容易与他们沟通和合作。如果没有，这将为您建立项目全面质量保证计划提供一个起点。

在本章中，您将学到以下内容：

+   质量基础和有效习惯

+   测试类型（单元测试和集成测试）

+   如何使用 Jest 编写单元测试

+   如何使用 Puppeteer 编写集成测试

+   如何配置 ESLint 以帮助预防常见错误

+   连续集成是什么以及如何开始学习它

# QA 计划

开发基本上是一个创造性的过程：设想某事然后将其变成现实。相比之下，QA 更多地生活在验证和秩序的领域。因此，QA 的一个重要部分只是*知道需要做什么*并*确保它被完成*。因此，QA 是一种非常适合使用清单、流程和文档的学科。我甚至可以说，QA 的主要活动不是软件本身的测试，而是*创建全面且可重复的 QA 计划*。

我建议为每个项目创建一个 QA 计划，无论其大小如何（是的，即使是您的周末“娱乐”项目也是如此！）。QA 计划不必很大或很复杂；您可以将其放在文本文件、文字处理文档或 wiki 中。QA 计划的目标是记录您将采取的所有步骤，以确保您的产品按预期运行。

无论采取什么形式，QA 计划都是一个活的文档。您将根据以下内容更新它：

+   新功能

+   现有功能的变化

+   已删除的功能

+   测试技术或技术的变化

+   QA 计划未发现的缺陷

最后一点特别值得一提。无论您的质量保证（QA）多么健全，缺陷都会发生。当它们发生时，您应该问自己：“我们如何才能预防这种情况？”当您回答了这个问题，就可以相应地修改您的 QA 计划，以防止将来发生这种类型的缺陷。

到目前为止，您可能已经感受到 QA 所需的不小的努力，并且您可能合理地想知道您想要投入多少努力。

# QA：是否值得？

QA 有时可能会非常昂贵——*非常*昂贵。那么这是否值得呢？这是一个复杂的公式，涉及复杂的输入。大多数组织采用某种“投资回报”模型。如果您花钱，您必须期望能够获得至少同等数量的回报（最好是更多）。然而，与 QA 相关的关系可能会变得混淆。例如，一个经过充分建立和良好评价的产品，可能比一个新的和未知的项目能够更长时间地容忍质量问题。显然，没有人*想*制造低质量的产品，但技术上的压力很大。时间至关重要，有时候，与其在几个月后推出完美的产品，不如在市场上推出一个不完美的产品更好。

在 Web 开发中，质量可以分解为四个维度：

覆盖率

覆盖率指的是您产品的市场渗透率：访问您网站或使用您服务的人数。覆盖率与盈利能力直接相关：访问网站的人越多，购买产品或服务的人也越多。从开发的角度来看，搜索引擎优化（SEO）将对覆盖率产生最大影响，这也是为什么我们将在我们的 QA 计划中包含 SEO。

功能性

一旦人们访问您的网站或使用您的服务，您网站功能的质量将对用户保留率产生重大影响；一个按照承诺运行的网站比那些没有按照承诺运行的网站更有可能促使用户再次访问。功能性为测试自动化提供了最大的机会。

可用性

当涉及功能正确性时，可用性评估人机交互（HCI）。根本问题是：“功能以对目标受众有用的方式提供了吗？”这通常转化为“使用起来容易吗？”尽管追求易用性往往会与灵活性或强大性相对立；对程序员来说容易的东西可能与非技术消费者认为容易的东西不同。换句话说，评估可用性时必须考虑目标受众。由于可用性评估的一个基本输入是用户，因此通常无法自动化。然而，用户测试应包括在您的 QA 计划中。

美学

美学是四个维度中最主观的，因此对开发的影响最小。虽然在涉及到您网站美学时，开发上的担忧很少，但您的 QA 计划应包括对网站美学的定期审查。向代表性样本观众展示您的网站，并了解它是否显得过时或未引发预期的反应。请记住，美学是时间敏感的（审美标准随时间变化）和特定于受众的（一个受众喜欢的东西可能完全无趣于另一个受众）。

虽然您的 QA 计划应涵盖所有四个维度，但功能性测试和 SEO 可以在开发过程中进行自动化测试，因此本章将重点放在这些方面。

# 逻辑与呈现

广义上说，在您的网站中，有两个“领域”：*逻辑*（通常称为*业务逻辑*，我因其对商业活动的偏见而避免使用此术语）和*呈现*。您可以将您网站的逻辑想象成一种纯粹的智力领域。例如，在我们的 Meadowlark Travel 场景中，可能存在一个规则，即客户在租用滑板车之前必须拥有有效的驾驶证。这是一个简单的数据规则：每次滑板车预订，用户都需要有效的驾驶证。而*呈现*则是分离的。也许它只是订单页面最后表单上的一个复选框，或者客户必须提供一个由 Meadowlark Travel 验证的有效驾驶证号码。这是一个重要的区别，因为在逻辑领域中应尽可能清晰简单，而呈现可以根据需要复杂或简单。呈现还受可用性和美学问题的影响，而业务领域则不受此影响。

在可能的情况下，您应该寻求清晰地区分您的逻辑和表现。有很多方法可以做到这一点，在本书中，我们将专注于将逻辑封装在 JavaScript 模块中。另一方面，展示将是 HTML、CSS、多媒体、JavaScript 以及像 React、Vue 或 Angular 这样的前端框架的组合。

# 测试类型

在本书中，我将考虑的测试类型分为两大类：单元测试和集成测试（我认为系统测试是集成测试的一种类型）。单元测试非常精细，测试单个组件以确保其正常工作，而集成测试测试多个组件之间甚至整个系统的交互。

一般来说，单元测试在逻辑测试中更加有用和适当。集成测试在两个领域都很有用。

# QA 技术概述

在本书中，我们将使用以下技术和软件来进行彻底的测试：

单元测试

单元测试覆盖应用程序中最小的功能单元，通常是单个函数。它们几乎总是由开发人员编写，而不是 QA（尽管 QA 应该有能力评估单元测试的质量和覆盖范围）。在本书中，我们将使用 Jest 进行单元测试。

单元测试

集成测试覆盖应用程序中更大的功能单元，通常涉及多个部分（函数、模块、子系统等）。由于我们正在构建 Web 应用程序，“终极”集成测试是在浏览器中呈现应用程序，操作该浏览器，并验证应用程序是否按预期行为。这些测试通常更复杂，设置和维护起来更加困难，由于本书的重点不是 QA，我们只有一个简单的示例，使用 Puppeteer 和 Jest。

Linting

Linting 并不是为了发现错误，而是*潜在*的错误。Linting 的一般概念是识别可能表示潜在错误的区域，或者脆弱的结构可能导致将来出现错误。我们将使用 ESLint 进行 Linting。

让我们从 Jest 开始，我们的测试框架（将运行单元测试和集成测试）。

# 安装和配置 Jest

在决定在本书中使用哪个测试框架方面，我有些挣扎。Jest 最初是作为测试 React 应用程序的框架而诞生的（现在仍然是这样做的明显选择），但 Jest 并不专门针对 React，它是一个优秀的通用测试框架。当然，Jest 并不是唯一的选择：[Mocha](https://mochajs.org)、[Jasmine](https://jasmine.github.io)、[Ava](https://github.com/avajs/ava)和[Tape](https://github.com/substack/tape)也是优秀的选择。

最后，我选择了 Jest，因为我觉得它提供了最好的整体体验（这一观点得到了 Jest 在[2018 JavaScript 现状](http://bit.ly/33ErHUE)调查中的优秀评分支持）。也就是说，这里提到的测试框架有很多相似之处，因此您应该能够将学到的知识应用到您喜欢的测试框架中。

要安装 Jest，请从您的项目根目录运行以下命令：

```
npm install --save-dev jest
```

（请注意，我们在这里使用了`--save-dev`；这告诉 npm 这是一个开发依赖项，并且不需要它来使应用程序本身正常运行；它将在*package.json*文件的`devDependencies`部分而不是`dependencies`部分中列出。）

在我们继续之前，我们需要一种方法来运行 Jest（它将运行项目中的任何测试）。通常的做法是在*package.json*中添加一个脚本。编辑*package.json*（在伴随的仓库中的*ch05/package.json*），并修改`scripts`属性（如果不存在则添加）：

```
  "scripts": {
    "test": "jest"
  },
```

现在，您只需键入以下内容即可运行项目中的所有测试：

```
npm test
```

如果您现在尝试，可能会收到一个错误，提示没有配置任何测试……因为我们还没有添加任何测试。所以让我们编写一些单元测试！

###### 注意

通常情况下，如果您在*package.json*文件中添加了一个脚本，您可以通过`npm run`来运行它。例如，如果您添加了一个名为`foo`的脚本，您可以键入`npm run foo`来运行它。然而，`test`脚本非常常见，因此 npm 知道如果您简单地键入`npm test`，它就会运行它。

# 单元测试

现在我们将把注意力转向单元测试。由于单元测试的重点是隔离单个函数或组件，因此我们首先需要学习模拟，这是实现隔离的重要技术之一。

## 模拟

您经常面临的挑战之一是如何编写“可测试”的代码。一般来说，试图做太多或假设很多依赖关系的代码比专注于少量或没有依赖关系的代码更难测试。

每当您有一个依赖项时，您就需要对其进行*mock*（模拟）以进行有效的测试。例如，我们的主要依赖项是 Express，它已经经过了彻底的测试，因此我们不需要也不想测试 Express 本身，只需要测试*我们如何使用它*。我们能够确定我们是否正确使用 Express 的唯一方法就是模拟 Express 本身。

我们目前拥有的路由（主页、关于页面、404 页面和 500 页面）在测试时相当困难，因为它们假设对 Express 有三个依赖：它们假设我们有一个 Express 应用程序（所以我们可以有`app.get`），以及请求和响应对象。幸运的是，很容易消除对 Express 应用程序本身的依赖性（请求和响应对象则更难……稍后详细讨论）。幸运的是，我们并没有从响应对象中使用太多功能（我们仅使用`render`方法），因此很容易对其进行 mock，我们很快就会看到。

## 为了增强可测试性重构应用程序

实际上，在我们的应用程序中，我们并没有太多的代码需要测试。到目前为止，我们只添加了少数路由处理程序和 `getFortune` 函数。

为了使我们的应用程序更易于测试，我们将*提取*实际的路由处理程序到它们自己的库中。创建一个文件 *lib/handlers.js*（在配套仓库中是 *ch05/lib/handlers.js*）：

```
const fortune = require('./fortune')

exports.home = (req, res) => res.render('home')

exports.about = (req, res) =>
  res.render('about', { fortune: fortune.getFortune() })

exports.notFound = (req, res) => res.render('404')

exports.serverError = (err, req, res, next) => res.render('500')
```

现在，我们可以重写我们的 *meadowloark.js* 应用程序文件来使用这些处理程序（在配套仓库中是 *ch05/meadowlark.js*）：

```
// typically at the top of the file
const handlers = require('./lib/handlers')

app.get('/', handlers.home)

app.get('/about', handlers.about)

// custom 404 page
app.use(handlers.notFound)

// custom 500 page
app.use(handlers.serverError)
```

现在测试这些处理程序变得更容易了：它们只是接受请求和响应对象的函数，我们需要验证我们是否正确地使用了这些对象。

## 编写我们的第一个测试

有多种方法可以让 Jest 找到测试。最常见的两种方法是将测试放在名为 *__test__* 的子目录中（在 *test* 前后加上两个下划线），以及将文件命名为 *.test.js* 扩展名。我个人喜欢结合这两种技术，因为它们各自在我的脑海中都有用途。将测试放在 *__test__* 目录中可以防止我的测试混杂在源代码目录中（否则，你的源目录中将会看到一份 *foo.test.js* 对应每个 *foo.js* 文件），而使用 *.test.js* 扩展名则意味着，如果我在编辑器中查看一堆选项卡，我一眼就能看出哪些是测试，哪些是源代码。

所以让我们创建一个名为 *lib/__tests__/handlers.test.js*（在配套仓库中是 *ch05/lib/__tests__/handlers.test.js*）的文件：

```
const handlers = require('../handlers')

test('home page renders', () => {
  const req = {}
  const res = { render: jest.fn() }
  handlers.home(req, res)
  expect(res.render.mock.calls[0][0]).toBe('home')
})
```

如果你对测试还不太熟悉，这可能看起来有点奇怪，所以让我们一步步来分析。

首先，我们导入要测试的代码（在本例中是路由处理程序）。然后，每个测试都有一个描述；我们试图描述正在测试的内容。在这种情况下，我们要确保首页得到渲染。

要调用我们的渲染器，我们需要请求和响应对象。如果我们要模拟整个请求和响应对象，可能需要写上整整一周的代码，但幸运的是，我们实际上并不需要它们太多内容。我们知道，在这种情况下，我们根本不需要请求对象中的任何内容（所以我们只是使用一个空对象），而我们从响应对象中唯一需要的是一个渲染方法。注意我们如何构造渲染函数：我们只需调用一个名为 *jest.fn()* 的 Jest 方法。这将创建一个通用的模拟函数，用于跟踪它的调用方式。

最后，我们来到测试的重要部分：断言。我们已经费了很大的劲来调用我们正在测试的代码，但是我们如何断言它是否按照预期工作？在这种情况下，代码应该调用响应对象的`render`方法，并传递字符串`home`。Jest 的模拟函数会跟踪它被调用的所有次数，所以我们只需验证它被调用了一次（如果调用了两次可能会有问题），这就是第一个`expect`所做的事情，并且它被调用时`home`作为第一个参数传入（第一个数组索引指定调用，第二个数组索引指定参数）。

###### Tip

当您每次对代码进行更改时，不断重新运行测试可能会变得乏味。幸运的是，大多数测试框架都有一个“观察”模式，它会持续监视您的代码和测试的更改并自动重新运行它们。要在观察模式下运行您的测试，请键入`npm test -- --watch`（额外的双破折号是必需的，让 npm 知道将`--watch`参数传递给 Jest）。

请继续修改您的`home`处理程序，以渲染除主页视图之外的其他内容；您会注意到您的测试现在失败了，并且您捕捉到了一个错误！

现在，我们可以为其他路由添加测试：

```
test('about page renders with fortune', () => {
  const req = {}
  const res = { render: jest.fn() }
  handlers.about(req, res)
  expect(res.render.mock.calls.length).toBe(1)
  expect(res.render.mock.calls[0][0]).toBe('about')
  expect(res.render.mock.calls[0][1])
    .toEqual(expect.objectContaining({
      fortune: expect.stringMatching(/\W/),
    }))
})

test('404 handler renders', () => {
  const req = {}
  const res = { render: jest.fn() }
  handlers.notFound(req, res)
  expect(res.render.mock.calls.length).toBe(1)
  expect(res.render.mock.calls[0][0]).toBe('404')
})

test('500 handler renders', () => {
  const err = new Error('some error')
  const req = {}
  const res = { render: jest.fn() }
  const next = jest.fn()
  handlers.serverError(err, req, res, next)
  expect(res.render.mock.calls.length).toBe(1)
  expect(res.render.mock.calls[0][0]).toBe('500')
})
```

注意“about”和服务器错误测试中的一些额外功能。 “about”渲染函数被调用时会带有一个幸运符，因此我们添加了一个期望，即它将获得一个包含至少一个字符的字符串作为幸运符。本书的范围不包括描述通过 Jest 及其`expect`方法可用的所有功能，但您可以在[Jest 主页](https://jestjs.io)找到详尽的文档。请注意，服务器错误处理程序需要四个参数，而不是两个，因此我们必须提供额外的模拟。

## 测试维护

您可能意识到测试并不是一劳永逸的事务。例如，如果出于合理的原因重命名我们的“home”视图，我们的测试会失败，然后我们不得不修复代码之外还要修复测试。

因此，团队花了很多精力来设定关于应该进行测试以及测试应该有多具体的现实期望。例如，我们不必检查“about”处理程序是否被带有幸运符调用过...这样做将节省我们免于不得不修复测试的麻烦，如果我们放弃了该功能。

此外，我不能为您提供有关测试代码深入程度的建议。我预计您对测试航空电子设备或医疗设备的代码的标准会与测试营销网站背后的代码有很大不同。

我可以为您提供一种回答“我的代码有多少被测试覆盖？”这个问题的方法，答案叫做*代码覆盖率*，我们接下来会讨论它。

## 代码覆盖率

代码覆盖率提供了对您的代码有多少被测试覆盖的量化答案，但像编程中的大多数主题一样，没有简单的答案。

Jest 提供了一些有用的自动化代码覆盖分析。要查看你的代码被测试了多少，运行以下命令：

```
npm test -- --coverage
```

如果你一直在跟进，你应该看到 *lib* 文件夹中一堆令人放心的绿色“100%”覆盖率数字。Jest 将报告语句（Stmts）、分支、函数（Funcs）和行的覆盖率百分比。

语句是指 JavaScript 语句，例如每个表达式、控制流语句等。请注意，你可以拥有 100% 的行覆盖率，但不一定有 100% 的语句覆盖率，因为你可以在 JavaScript 中将多个语句放在一行上。分支覆盖率涉及控制流语句，例如 `if-else`。如果你有一个 `if-else` 语句，而你的测试仅执行了 `if` 部分，那么这个语句的分支覆盖率为 50%。

你可能注意到 *meadowlark.js* 并不具备 100% 的覆盖率。这并不一定是问题；如果你看一下我们重构后的 *meadowlark.js* 文件，你会发现现在大部分内容只是配置……我们只是把各种东西粘合在一起。我们正在用相关的中间件配置 Express 并启动服务器。不仅这段代码很难进行有意义的测试，而且合理的论点是你不应该这样做，因为它只是组装经过充分测试的代码。

你甚至可以提出一个论点，迄今为止我们编写的测试并不特别有用；它们也只是验证我们是否正确配置了 Express。

再次地，我没有简单的答案。在一天结束时，你正在构建的应用类型、你的经验水平以及团队的规模和配置将对你有多深入地进行测试产生很大影响。我鼓励你在测试方面保守一点，比不足要多一些，但随着经验的增加，你会找到“恰到好处”的甜蜜点。

# 集成测试

在我们的应用中目前没有什么有趣的东西可以测试；我们只有几个页面，没有互动。因此，在编写集成测试之前，让我们添加一些可以测试的功能。为了保持简单，我们让这个功能是一个链接，可以让你从主页跳转到关于页面。事实上，用户看起来这似乎很简单，但这是一个真正的集成测试，因为它不仅测试了两个 Express 路由处理程序，还测试了 HTML 和 DOM 交互（用户点击链接和结果页面导航）。让我们在 *views/home.handlebars* 中添加一个链接：

```
<p>Questions?  Checkout out our
<a href="/about" data-test-id="about">About Us</a> page!</p>
```

你可能会想到`data-test-id`属性。为了进行测试，我们需要一种方法来识别链接，以便可以（虚拟）点击它。我们可以使用 CSS 类来实现这一点，但我更喜欢将类保留用于样式化，而使用数据属性进行自动化。我们还可以搜索*关于我们*的文本，但这将是一个脆弱且昂贵的 DOM 搜索。我们还可以根据`href`参数进行查询，这也是有道理的（但这样做会使得这个测试很难失败，这是我们出于教育目的希望的）。

我们可以启动我们的应用程序，并用我们笨拙的人类手来验证功能是否按预期工作，然后再进入更自动化的内容。

在我们开始安装 Puppeteer 并编写集成测试之前，我们需要修改我们的应用程序，使其可以作为模块被引用（目前它只能直接运行）。在 Node 中做到这一点的方法有点不透明：在*meadowlark.js*的底部，用以下内容替换对`app.listen`的调用：

```
if(require.main === module) {
  app.listen(port, () => {
    console.log( `Express started on http://localhost:${port}` +
      '; press Ctrl-C to terminate.' )
  })
} else {
  module.exports = app
}
```

我将跳过这个技术解释，因为它相当冗长，但如果你感兴趣，仔细阅读[Node 的模块文档](http://bit.ly/32BDO3H)将会让你明白。重要的是要知道，如果你直接用 node 运行一个 JavaScript 文件，`require.main`将等于全局的`module`；否则，它是从另一个模块中导入的。

现在我们已经搞定了，可以安装 Puppeteer 了。Puppeteer 本质上是一个可控的、无头版本的 Chrome 浏览器。（*无头*意味着浏览器可以在不渲染 UI 的情况下运行。）要安装 Puppeteer：

```
npm install --save-dev puppeteer
```

我们还将安装一个小工具来找到一个空闲端口，这样我们的应用程序就不会因为无法在请求的端口上启动而产生大量的测试错误：

```
npm install --save-dev portfinder
```

现在我们可以编写一个执行以下操作的集成：

1.  在一个未占用的端口上启动我们的应用程序服务器

1.  启动一个无头 Chrome 浏览器并打开一个页面

1.  导航到我们应用程序的主页

1.  查找带有`data-test-id="about"`的链接并点击它

1.  等待导航发生

1.  验证我们是否在*/about*页面上

创建一个名为*integration-tests*（如果你愿意，也可以起其他名字）的目录，并在该目录中创建一个文件*basic-navigation.test.js*（在伴随的仓库中是*ch05/integration-tests/basic-navigation.test.js*）：

```
const portfinder = require('portfinder')
const puppeteer = require('puppeteer')

const app = require('../meadowlark.js')

let server = null
let port = null

beforeEach(async () => {
  port = await portfinder.getPortPromise()
  server = app.listen(port)
})

afterEach(() => {
  server.close()
})

test('home page links to about page', async () => {
  const browser = await puppeteer.launch()
  const page = await browser.newPage()
  await page.goto(`http://localhost:${port}`)
  await Promise.all([
    page.waitForNavigation(),
    page.click('[data-test-id="about"]'),
  ])
  expect(page.url()).toBe(`http://localhost:${port}/about`)
  await browser.close()
})
```

我们正在使用 Jest 的`beforeEach`和`afterEach`钩子在每个测试之前启动服务器，并在每个测试之后关闭它（现在我们只有一个测试，所以当我们添加更多测试时，这将变得更有意义）。我们也可以使用`beforeAll`和`afterAll`，这样我们就不会为每个测试都启动和关闭服务器，这可能会加快测试速度，但代价是每个测试都不会有一个“干净”的环境。也就是说，如果你的某个测试对后续测试结果产生影响，你就引入了难以维护的依赖关系。

我们实际测试使用了 Puppeteer 的 API，它为我们提供了大量的 DOM 查询功能。请注意，这里几乎所有的操作都是异步的，我们大量使用 `await` 来使测试更易于阅读和编写（几乎所有 Puppeteer API 调用都返回一个 promise）。^(1) 我们将导航和点击包装在 `Promise.all` 调用中，以避免竞争条件，按照 Puppeteer 文档的建议。

Puppeteer API 中有比我在本书中能够涵盖的功能多得多。幸运的是，它有 [优秀的文档](http://bit.ly/2KctokI)。

测试是确保产品质量的重要后备工具，但它并不是你手头唯一的工具。代码检查帮助你在第一时间防止常见错误。

# 代码检查

一个好的代码检查器就像有第二双眼睛一样：它会发现那些会轻易被我们的大脑忽略的问题。最初的 JavaScript 代码检查器是道格拉斯·克罗克福德的 JSLint。2011 年，安东·科瓦留夫分叉了 JSLint，JSHint 诞生了。科瓦留夫发现 JSLint 变得过于主观，他想创建一个更可定制、由社区开发的 JavaScript 代码检查器。在 JSHint 之后是尼古拉斯·扎卡斯的 [ESLint](https://eslint.org)，它已成为最受欢迎的选择（在 [2017 JavaScript 现状调查](http://bit.ly/2Q7w32O) 中遥遥领先）。除了普及性之外，ESLint 显然是最积极维护的代码检查器，我更喜欢它灵活的配置而不是 JSHint，并且这也是我推荐的。

ESLint 可以基于每个项目安装或全局安装。为了避免无意中破坏东西，我尽量避免全局安装（例如，如果我全局安装 ESLint 并经常更新它，旧项目可能由于破坏性更改而无法成功 lint，现在我必须额外工作来更新我的项目）。

要在你的项目中安装 ESLint：

```
npm install --save-dev eslint
```

ESLint 需要一个配置文件来告诉它应用哪些规则。从零开始做这件事将是一项耗时的任务，所幸 ESLint 提供了一个实用工具来为你创建。从你的项目根目录运行以下命令：

```
./node_modules/.bin/eslint --init
```

###### 注意

如果我们全局安装了 ESLint，我们可以直接使用 `eslint --init`。笨拙的 `./node_modules/.bin` 路径是必须的，以便直接运行本地安装的工具。我们很快会看到，如果我们将工具添加到 *package.json* 文件的 `scripts` 部分，我们就不必这样做，这对于我们经常做的事情是推荐的。但是，创建 ESLint 配置是我们每个项目只需要做一次的事情。

ESLint 会询问你一些问题。对于大多数问题，选择默认值是安全的，但有几个值得注意：

你的项目使用哪种类型的模块？

因为我们使用的是 Node（而不是将在浏览器中运行的代码），你会想要选择“CommonJS (require/exports)”。“你的项目中可能也有客户端 JavaScript，这种情况下可能需要一个单独的 lint 配置。最简单的方法是在同一个项目中有两个分开的项目，但在同一个项目中有多个 ESLint 配置也是可能的。请参阅 [ESLint 文档](https://eslint.org/) 获取更多信息。

你的项目使用哪个框架？

除非你在那里看到 Express（在我写作时还没有），选择“None of these”。

你的代码在哪里运行？

选择 Node。

现在 ESLint 已经设置好了，我们需要一个便捷的方式来运行它。将以下内容添加到你的 *package.json* 的 `scripts` 部分：

```
  "lint": "eslint meadowlark.js lib"
```

注意，我们必须明确告诉 ESLint 我们要 lint 哪些文件和目录。这是一个建议将所有源代码集中在一个目录（通常是 *src*）下的理由。

现在准备好，运行以下命令：

```
npm run lint
```

你可能会看到很多看起来不太好看的错误—通常当你第一次运行 ESLint 时会发生这种情况。然而，如果你一直在进行 Jest 测试，会有一些与 Jest 相关的误报错误，看起来像这样：

```
   3:1   error  'test' is not defined    no-undef
   5:25  error  'jest' is not defined    no-undef
   7:3   error  'expect' is not defined  no-undef
   8:3   error  'expect' is not defined  no-undef
  11:1   error  'test' is not defined    no-undef
  13:25  error  'jest' is not defined    no-undef
  15:3   error  'expect' is not defined  no-undef
```

ESLint（非常合理地）不允许未识别的全局变量。Jest 注入了全局变量（特别是 `test`、`describe`、`jest` 和 `expect`）。幸运的是，这是一个容易解决的问题。在你的项目根目录下，打开 *.eslintrc.js* 文件（这是 ESLint 的配置文件）。在 `env` 部分，添加以下内容：

```
"jest": true,
```

现在如果你再次运行 `npm run lint`，你应该会看到更少的错误。

那么剩下的错误怎么办？这就是我能提供智慧但无具体指导的地方。总体来说，Lint 错误有三种原因：

+   这是一个合法的问题，你应该解决它。有时候问题可能并不明显，这时你可能需要参考 ESLint 文档中特定错误的部分。

+   这是一个你不同意的规则，你可以简单地禁用它。ESLint 中的许多规则都是主观的。稍后我会演示如何禁用一个规则。

+   你同意这个规则，但在特定情况下修复它是不可行或成本很高的。对于这些情况，你可以仅为文件中特定行禁用规则，我们也将看到一个示例。

如果你一直在跟进，你现在应该看到以下错误：

```
/Users/ethan/wdne2e-companion/ch05/meadowlark.js
  27:5  error  Unexpected console statement  no-console

/Users/ethan/wdne2e-companion/ch05/lib/handlers.js
  10:39  error  'next' is defined but never used  no-unused-vars
```

ESLint 抱怨控制台日志，因为这不一定是为你的应用提供输出的好方法；它可能会很嘈杂和不一致，并且根据运行方式，输出可能被忽略。然而，对于我们的用途，假设它并不影响我们，我们想要禁用这个规则。打开你的 *.eslintrc* 文件，找到 `rules` 部分（如果没有 `rules` 部分，请在导出对象的顶层创建一个），然后添加以下规则：

```
  "rules": {
    "no-console": "off",
  },
```

现在，如果我们再次运行`npm run lint`，就会看到这个错误不见了！接下来的问题有点棘手……

打开*lib/handlers.js*，考虑问题行：

```
exports.serverError = (err, req, res, next) => res.render('500')
```

ESLint 是正确的；我们将`next`作为参数传递，但没有做任何操作（我们也没有处理`err`和`req`，但由于 JavaScript 处理函数参数的方式，我们必须放置*某些*内容以便可以获取到`res`，我们确实在使用它）。

你可能会被诱惑只是删除`next`参数。“有什么害处？”你可能会想。确实，不会有运行时错误，并且你的代码检查工具会很高兴……但会造成一个难以察觉的伤害：你的自定义错误处理程序将停止工作！（如果你想自己看看，可以从一个路由抛出一个异常，然后尝试访问它，然后从`serverError`处理程序中删除`next`参数。）

Express 在这里做了一些微妙的事情：它使用您传递给它的实际参数数量来识别它应该是一个错误处理程序。如果没有那个`next`参数，无论您是否使用它，Express 都不再将其识别为错误处理程序。

###### 注意

Express 团队在错误处理程序上所做的事情无疑是“聪明”的，但聪明的代码往往会令人困惑、容易出错或难以理解。尽管我非常喜欢 Express，但这是我认为团队做错的选择之一：我认为它应该找到一种不那么特异的、更明确的方式来指定错误处理程序。

我们无法更改处理程序代码，我们需要我们的错误处理程序，但我们喜欢这个规则，不想禁用它。我们可以忍受这个错误，但错误会积累并成为一个不断的烦恼，最终会侵蚀拥有代码检查工具的初衷。幸运的是，我们可以通过禁用该规则来解决这个问题。在*lib/handlers.js*中编辑，并在你的错误处理程序周围添加以下内容：

```
// Express recognizes the error handler by way of its four
// arguments, so we have to disable ESLint's no-unused-vars rule
/* eslint-disable no-unused-vars */
exports.serverError = (err, req, res, next) => res.render('500')
/* eslint-enable no-unused-vars */
```

刚开始时代码检查可能有点令人沮丧——似乎它不停地让你出错。当然，你应该随意禁用那些不适合你的规则。随着你学会避免代码检查旨在捕捉的常见错误，你会发现它越来越不令人沮丧。

测试和代码检查无疑是有用的，但任何工具如果你从不使用它就毫无价值！也许你会觉得很疯狂，你会花费时间和精力编写单元测试和设置代码检查，但我看过这种情况发生，尤其是在压力之下。幸运的是，有一种方法可以确保这些有用的工具不被遗忘：持续集成。

# 持续集成

我给你留下另一个非常有用的 QA 概念：持续集成（CI）。如果你在团队中工作，这尤为重要，但即使你是独自工作，它也可以提供一些有益的纪律性。

基本上，持续集成会在每次向源代码库贡献代码时运行一些或所有的测试（你可以控制这适用于哪些分支）。如果所有测试都通过了，通常不会发生任何事情（根据你的持续集成配置，你可能会收到一封“干得好”的电子邮件）。

另一方面，如果测试失败，后果通常更为……公开。同样，这取决于你如何配置你的持续集成，但通常整个团队都会收到一封邮件，告诉你“破坏了构建”。如果你的集成主管真的很刻薄，有时你的老板也会在那个邮件列表中！我甚至知道一些团队，他们设置了灯光和警报器，当有人破坏了构建时，一个微型机器人泡沫导弹发射器会向违规开发者发射软弹！这是在提交代码之前运行你的 QA 工具链的一个强大激励。

本书的范围不包括安装和配置持续集成服务器，但涉及到 QA 的一章如果没有提到它，就不算完整。

目前，Node 项目中最流行的持续集成服务器是[Travis CI](https://travis-ci.org/)。Travis CI 是一种托管解决方案，这可能很吸引人（它可以帮助你避免设置自己的持续集成服务器）。如果你使用 GitHub，它提供了优秀的集成支持。[CircleCI](https://circleci.com) 是另一个选择。

如果你是独自工作在一个项目上，你可能不会从持续集成服务器中获得太多好处，但如果你在团队或开源项目中工作，我强烈建议你考虑为你的项目设置持续集成。

# 结论

本章涵盖了很多内容，但我认为这些是任何开发框架中的基本实际技能。JavaScript 生态系统非常庞大，如果你是新手，可能很难知道从哪里开始。我希望本章能指导你朝着正确的方向前进。

现在我们已经对这些工具有了一些经验，接下来我们将关注 Node 和 Express 对象的一些基本原理，这些对象包围着 Express 应用程序中发生的所有事情：请求和响应对象。

^(1) 如果你对 `await` 不熟悉，我推荐阅读[Tamas Piros 的这篇文章](http://bit.ly/2rEXU0d)。
