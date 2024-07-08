# 第一章：我们的开发环境

约翰·伍登（John Wooden），UCLA 男子篮球队的已故教练，是有史以来最成功的教练之一，在 12 年内赢得了 10 个国家冠军。他的队伍包括顶级新秀，包括名人堂球员如 Lew Alcindor（Kareem Abdul-Jabbar）和 Bill Walton。在第一天的训练中，伍登会坐下他的每一位新秀队员，这些在美国高中时期曾经是最优秀的球员，并教他们正确穿袜子的方法。当被问及此事时，[伍登表示](https://oreil.ly/lnZkf)，“正是一些小细节造就了重大成就。”

厨师们使用术语*mise en place*，意思是“一切就位”，来描述在烹饪前准备菜单所需的工具和食材的做法。这种准备使得厨房的厨师们能够在繁忙时段成功地准备餐食，因为已经考虑到了小细节。就像伍德恩教练的球员和厨师为晚餐高峰期做准备一样，值得花时间来设置我们的开发环境。

一个有用的开发环境并不需要昂贵的软件或顶级硬件。事实上，我鼓励您从简单开始，使用开源软件，并随着您的需求增加工具。尽管跑步者喜欢特定品牌的运动鞋，木匠可能总是会拿起她喜欢的锤子，但建立这些偏好需要时间和经验。尝试不同的工具，观察他人，随着时间的推移，您将创建最适合自己的环境。

在本章中，我们将安装一个文本编辑器、Node.js、Git、MongoDB 和几个有用的 JavaScript 包，以及找到我们的终端应用程序。您可能已经拥有一个对您而言效果很好的开发环境；然而，我们还将安装本书中将使用的几个必需工具。如果您像我一样通常跳过使用说明书，我仍然鼓励您通读本指南。

如果您在任何时候遇到困难，请联系 JavaScript Everywhere 社区，通过我们的 Spectrum 频道[*spectrum.chat/jseverywhere*](https://spectrum.chat/jseverywhere)。

# 您的文本编辑器

文本编辑器就像衣服一样。我们都需要它们，但我们的偏好可能大相径庭。有些人喜欢简单且结构良好的。有些人喜欢花哨的佩斯利图案。没有错误的选择，您应该使用让自己最舒适的工具。

如果您还没有最喜欢的文本编辑器，我强烈推荐[Visual Studio Code (VSCode)](https://code.visualstudio.com)。它是一个开源编辑器，适用于 Mac、Windows 和 Linux。此外，它提供了内置功能来简化开发，并可以通过社区扩展进行轻松修改。它甚至是使用 JavaScript 构建的！

# 终端

如果您使用 VSCode，它带有集成终端。对于大多数开发任务，这可能就足够了。就我个人而言，我发现使用专用终端客户端更可取，因为我发现在我的机器上更容易管理多个选项卡并使用更多的专用窗口空间。我建议尝试两种方法，找到最适合您的方法。

## 使用专用终端应用程序

所有操作系统都带有一个内置的终端应用程序，这是一个很好的开始。在 macOS 上，它被称为 Terminal。在 Windows 操作系统上，从 Windows 7 开始，该程序是 PowerShell。Linux 发行版的终端名称可能有所不同，但通常包括“Terminal”。

## 使用 VSCode

要访问 VSCode 中的终端，请单击 Terminal → New Terminal。这将为您呈现一个终端窗口。提示将出现在与当前项目相同的目录中。

## 导航文件系统

一旦找到您的终端，您将需要关键的能力来导航文件系统。您可以使用`cd`命令来做到这一点，它代表“更改目录”。

# 命令行提示

终端指令通常在行首包含`$`或`>`。这些用于指示提示，不应复制。在本书中，我将用美元符号（`$`）表示终端提示。在输入指令到您的终端应用程序时，不要输入`$`。

当您打开终端应用程序时，您将看到一个光标提示，您可以在其中输入命令。默认情况下，您位于计算机的主目录中。如果还没有这样做，我建议在主目录中创建一个*项目*文件夹作为子目录。这个文件夹可以存放所有您的开发项目。您可以创建一个*项目*目录并像这样导航到该文件夹：

```
# first type cd, this will ensure you are in your root directory
$ cd
# next, if you don't already have a Projects directory, you can create one
# this will create Projects as a subfolder in your system's root directory
$ mkdir Projects
# finally you can cd into the Projects directory
$ cd Projects
```

在未来，您可以按照以下方式导航到您的*项目*目录：

```
$ cd # ensure you are in the root directory
$ cd Projects
```

现在假设您在*项目*目录中有一个名为*jseverywhere*的文件夹。您可以从*项目*目录中键入`cd jseverywhere`来导航到该文件夹。要向后导航到一个目录（在这种情况下是*项目*），您将键入`cd ..`（`cd`命令后跟两个句点）。

总的来说，这看起来可能是这样的：

```
> $ cd # ensure you are in your root directory
> $ cd Projects # navigate from root dir to Projects dir
/Projects > $ cd jseverywhere # navigate from Projects dir to jsevewehre dir
/Projects/jseverwhere > $ cd .. # navigate back from jseverwhere to Projects
/Projects > $ # Prompt is currently in the Projects dir
```

如果这对您来说是新的，请花一些时间浏览您的文件，直到您感到舒适。我发现文件系统问题是初学者开发者常见的绊脚石。掌握这一点将为您建立工作流程提供坚实的基础。

# 命令行工具和 Homebrew（仅限 Mac）

一些命令行实用程序只有在安装了 Xcode 后才能供 macOS 用户使用。您可以通过在终端中安装`xcode-select`来绕过这个问题，而不必安装 Xcode。要这样做，请运行以下命令并按照安装提示操作：

```
$ xcode-select --install
```

Homebrew 是 macOS 的包管理器。它使得安装开发依赖项（如编程语言和数据库）变得像在命令行提示符下运行一样简单。如果您使用 Mac，它将大大简化您的开发环境。要安装 Homebrew，请访问[*brew.sh*](https://brew.sh)，复制并粘贴安装命令，或者在一行中输入以下内容：

```
$ /usr/bin/ruby -e "$(curl -fsSL
https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

# Node.js 和 NPM

[Node.js](https://nodejs.org)是“建立在 Chrome 的 V8 JavaScript 引擎上的 JavaScript 运行时”。从实际角度来看，这意味着 Node 是一个平台，允许开发人员在非浏览器环境下编写 JavaScript 代码。Node.js 自带 NPM 作为默认的包管理器。NPM 使您能够在项目中安装数千个库和 JavaScript 工具。

# 管理 Node.js 版本

如果您计划管理大量的 Node 项目，您可能会发现需要在计算机上管理多个 Node 版本。如果是这种情况，我建议使用[Node Version Manager (NVM)](https://oreil.ly/fzBpO)来安装 Node。NVM 是一个脚本，使您能够管理多个活动的 Node 版本。对于 Windows 用户，我建议使用[nvm-windows](https://oreil.ly/qJeej)。我不会涵盖 Node 版本管理的内容，但这是一个有用的工具。如果这是您第一次使用 Node，请按照系统的以下说明操作。

## 在 macOS 上安装 Node.js 和 NPM

macOS 用户可以使用 Homebrew 安装 Node.js 和 NPM。要安装 Node.js，请在终端中输入以下命令：

```
$ brew update
$ brew install node
```

安装完成 Node 后，请打开您的终端应用程序以验证其是否正常工作。

```
$ node --version
## Expected output v12.14.1, your version number may differ
$ npm --version
## Expected output 6.13.7, your version number may differ
```

如果在输入这些命令后看到版本号码，恭喜你——你已成功在 macOS 上安装了 Node 和 NPM！

## 在 Windows 上安装 Node.js 和 NPM

对于 Windows 用户，安装 Node.js 最简单的方法是访问[*nodejs.org*](https://nodejs.org)，并下载适用于您操作系统的安装程序。

首先，请访问[*nodejs.org*](https://nodejs.org)，安装 LTS 版本（本文撰写时为 12.14.1），按照您操作系统的安装步骤进行操作。安装完成 Node 后，打开您的终端应用程序以验证其是否正常工作。

```
$ node --version
## Expected output v12.14.1, your version number may differ
$ npm --version
## Expected output 6.13.7, your version number may differ
```

# 什么是 LTS？

LTS 代表“长期支持”，这意味着 Node.js 基金会承诺为该主要版本号（在本例中为 12.x）提供支持和安全更新。标准支持窗口在该版本初始发布后持续三年。在 Node.js 中，偶数发布版本是 LTS 版本。我建议在应用程序开发中使用偶数发布版本。

如果在输入这些命令后看到版本号码，恭喜你——你已成功在 Windows 上安装了 Node 和 NPM！

# MongoDB

MongoDB 是我们在开发 API 时将使用的数据库。Mongo 是使用 Node.js 的热门选择，因为它将我们的数据视为 JSON（JavaScript 对象表示）文档。这意味着 JavaScript 开发人员可以从一开始就轻松使用它。

# 官方 MongoDB 安装文档

MongoDB 文档提供了跨操作系统安装 MongoDB Community Edition 的定期更新指南。如果在安装过程中遇到问题，建议查阅文档[*docs.mongodb.com/manual/administration/install-community*](https://docs.mongodb.com/manual/administration/install-community)。

## 安装和运行 MongoDB for macOS

要在 macOS 上安装 MongoDB，首先使用 Homebrew 安装：

```
$ brew update
$ brew tap mongodb/brew
$ brew install mongodb-community@4.2
```

要启动 MongoDB，我们可以将其作为 macOS 服务运行：

```
$ brew services start mongodb-community
```

这将启动 MongoDB 服务并将其作为后台进程运行。请注意，每次重新启动计算机并计划使用 Mongo 进行开发时，您可能需要再次运行此命令以重新启动 MongoDB 服务。要验证 MongoDB 是否已安装并运行，请在终端中键入**`ps -ef | grep mongod`**。这将列出当前正在运行的 Mongo 进程。

## 安装和运行 MongoDB for Windows

要在 Windows 上安装 MongoDB，请首先从[MongoDB 下载中心](https://oreil.ly/XNQj6)下载安装程序。一旦文件下载完成，按照安装向导运行安装程序。我建议选择完整设置类型，并将其配置为服务。所有其他值可以保持默认设置。

安装完成后，我们可能需要创建一个目录，Mongo 将在其中写入我们的数据。在您的终端中运行以下命令：

```
$ cd C:\
$ md "\data\db"
```

要验证 MongoDB 是否已安装并启动 Mongo 服务：

1.  定位 Windows 服务控制台。

1.  查找 MongoDB 服务。

1.  右键单击 MongoDB 服务。

1.  点击启动。

请注意，每次重新启动计算机并计划使用 Mongo 进行开发时，您可能需要重新启动 MongoDB 服务。

# Git

Git 是最流行的版本控制软件，允许您执行诸如复制代码存储库、将代码与其他人合并以及创建不相互影响的自己代码分支等操作。Git 将有助于“克隆”本书示例代码存储库，这意味着它将允许您直接复制一个示例代码文件夹。根据您的操作系统，Git 可能已经安装。在终端窗口中键入以下内容：

```
$ git --version
```

如果返回了数字，则表示您已准备就绪！如果没有，请访问[*git-scm.com*](https://git-scm.com)安装 Git，或者在 macOS 上使用 Homebrew。完成安装步骤后，再次在终端中键入**`git --version`**以验证是否已成功。

# Expo

Expo 是一个工具链，简化了使用 React Native 在 iOS 和 Android 项目中启动和开发的过程。我们需要安装 Expo 命令行工具，并可选地（但建议）安装 iOS 或 Android 上的 Expo 应用程序。我们将在本书的移动应用程序部分详细介绍这一点，但如果您有兴趣提前开始，请访问[*expo.io*](https://expo.io)了解更多信息。要安装命令行工具，请在终端中输入以下内容：

```
npm install -g expo-cli
```

使用`-g`全局标志将使`expo-cli`工具在您机器上的 Node.js 安装中全局可用。

要安装 Expo 移动应用程序，请访问您设备上的 Apple App Store 或 Google Play Store。

# Prettier

Prettier 是一个代码格式化工具，支持多种语言，包括 JavaScript、HTML、CSS、GraphQL 和 Markdown。它使得遵循基本的格式化规则变得很容易，这意味着当您运行 Prettier 命令时，您的代码会自动按照一套标准的最佳实践格式化。更好的是，您可以配置您的编辑器，在每次保存文件时自动执行此操作。这意味着您将永远不会再遇到项目中存在不一致空格和混合引号等问题。

我建议在您的机器上全局安装 Prettier 并为您的编辑器配置插件。要全局安装 Prettier，请转到命令行并键入：

```
npm install -g prettier
```

安装了 Prettier 后，请访问[*Prettier.io*](https://prettier.io)为您的文本编辑器找到插件。安装了编辑器插件后，我建议在编辑器的设置文件中添加以下设置：

```
"editor.formatOnSave": true,
"prettier.requireConfig": true
```

当项目中存在*.prettierrc*配置文件时，这些设置将在保存文件时自动格式化文件。*.prettierrc*文件指定了 Prettier 要遵循的选项。现在，每当该文件存在时，您的编辑器都会自动重新格式化代码以符合项目的约定。本书中的每个项目都将包括一个*.prettierrc*文件。

# ESLint

ESLint 是用于 JavaScript 的代码检查工具。与 Prettier 等格式化工具不同，代码检查工具还会检查代码质量规则，如未使用的变量、无限循环和 return 后面不可达的代码。与 Prettier 一样，我建议为您喜欢的文本编辑器安装 ESLint 插件。这将在您编写代码时实时警告您的错误。您可以在[ESLint 网站](https://oreil.ly/H3Zao)找到一系列编辑器插件。

类似于 Prettier，项目可以在*.eslintrc*文件中指定要遵循的 ESLint 规则。这为项目维护者提供了对其代码偏好的细粒度控制，并自动执行编码标准的手段。本书中的每个项目都将包括一组有用但宽容的 ESLint 规则，旨在帮助您避免常见的陷阱。

# 美观化

这是可选的，但我发现当我发现我的设置在美学上更加令人愉悦时，我更喜欢编程。我控制不住；我有艺术学位。花些时间测试不同的颜色主题和字体。就我个人而言，我已经喜欢上了[德拉库拉主题](https://draculatheme.com)，这是几乎每个文本编辑器和终端都可以使用的颜色主题，以及 Adobe 的[源代码 Pro 字体](https://oreil.ly/PktVn)。

# 结论

在本章中，我们在计算机上建立了一个工作灵活的 JavaScript 开发环境。编程的一大乐趣之一就是个性化你的环境。我鼓励你尝试使用不同的主题、颜色和工具，让这个环境更符合你的喜好。在本书的下一节中，我们将通过开发 API 应用程序来利用这个环境。
