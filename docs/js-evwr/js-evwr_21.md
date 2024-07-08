# 第二十章：电子部署

我第一次教编程课时，想出了一个聪明的主意，通过一个文本冒险游戏介绍课程内容。学生们会进入实验室，坐在桌子前，按照一系列我认为很搞笑的提示和说明进行操作。这受到了不同的反响，不是因为笑话（也许也有笑话的原因），而是因为学生们以前没有以这种方式与“程序”进行互动。学生们习惯于图形用户界面（GUI），通过文本提示与程序进行互动对他们来说感觉*不对*。

现在，为了运行我们的应用程序，我们需要在终端应用程序中键入一个提示符来启动 Electron 进程。在本章中，我们将看看如何将我们的应用程序打包以进行分发。为了实现这一目标，我们将使用流行的[Electron Builder](https://www.electron.build)库，该库将帮助我们将应用程序打包并分发给用户。

# Electron Builder

Electron Builder 是一个旨在简化 Electron 和[Proton Native](https://proton-native.js.org) 应用程序打包和分发的库。虽然有其他打包解决方案，但 Electron Builder 简化了与应用程序分发相关的许多痛点，包括：

+   代码签名

+   多平台分发目标

+   自动更新

+   分发

它在灵活性和功能之间提供了很好的平衡。此外，虽然我们不会使用它们，但还有几个 Electron Builder 的样板文件适用于[Webpack](https://oreil.ly/faYta)、[React](https://oreil.ly/qli_e)、[Vue](https://oreil.ly/9QY2W)和[纯 JavaScript](https://oreil.ly/uJo7e)。

# Electron Builder 与 Electron Forge 的比较

[Electron Forge](https://www.electronforge.io) 是另一个流行的库，提供许多与 Electron Builder 类似的功能。Electron Forge 的一个主要优点是它基于官方 Electron 库，而 Electron Builder 是一个独立的构建工具。这意味着用户可以从 Electron 生态系统的增长中受益。缺点是 Electron Forge 基于更加严格的应用程序设置。对于本书的目的，Electron Builder 提供了功能和学习机会的良好平衡，但我鼓励你也仔细研究 Electron Forge。

## 配置 Electron Builder

所有 Electron Builder 的配置都将在我们应用程序的 *package.json* 文件中进行。在该文件中，我们可以看到 `electron-builder` 已列为开发依赖项。在 *package.json* 文件中，我们可以包含一个名为 `"build"` 的键，其中将包含所有给 Electron Builder 的打包应用程序的指令。首先，我们将包括两个字段：

`appId`

这是我们应用程序的唯一标识符。macOS 称之为[`CFBundle`​`Identifier`](https://oreil.ly/OOg1O)，Windows 称之为[`AppUser`​`ModelID`](https://oreil.ly/mr9si)。标准是使用反向 DNS 格式。例如，如果我们经营一个域名为*jseverywhere.io*的公司，并构建一个名为 Notedly 的应用程序，则该 ID 将是`io.jseverywhere.notedly`。

`productName`

这是我们产品名称的人类可读版本，因为`package.json`的`name`字段需要连字符或单词名称。

所有在一起，我们的初始构建配置将如下所示：

```
"build": {
  "appId": "io.jseverywhere.notedly",
  "productName": "Notedly"
},
```

Electron Builder 为我们提供了许多配置选项，我们将在本章中探讨其中的几个。完整列表，请访问[Electron Builder 文档](https://oreil.ly/ESAx-)。

# 为我们当前的平台构建

在我们完成最小配置后，我们可以创建我们的第一个应用程序构建。默认情况下，Electron Builder 将生成适合我们开发的系统的构建。例如，我现在正在 MacBook 上写作，我的构建将默认为 macOS。

首先，在我们的*package.json*文件中添加两个脚本，这些脚本将负责应用程序的构建。首先，`pack`脚本将生成一个包目录，而不会完全打包应用程序。这对于测试目的非常有用。其次，`dist`脚本将以可分发格式打包应用程序，例如 macOS DMG、Windows 安装程序或 DEB 包。

```
"scripts": {
  // add the pack and dist scripts to the existing npm scripts list
  "pack": "electron-builder --dir",
  "dist": "electron-builder"
}
```

有了这些改变，您可以在终端应用程序中运行`npm run dist`，这将在项目的*dist/*目录中打包应用程序。导航到*dist/*目录，您可以看到 Electron Builder 已经将应用程序打包为适合您操作系统的分发版本。

# 应用程序图标

你可能已经注意到的一件事是，我们的应用程序正在使用默认的 Electron 应用程序图标。这在本地开发中是可以的，但对于生产应用程序，我们希望使用自己的品牌。在我们项目的*/resources*文件夹中，我包含了一些适用于 macOS 和 Windows 的应用程序图标。为了从 PNG 文件生成这些图标，我使用了[iConvert Icons 应用程序](https://iconverticons.com)，它适用于 macOS 和 Windows。

在我们的*/resources*文件夹中，您将看到以下文件：

+   *icon.icns*，macOS 应用程序图标

+   *icon.ico*，Windows 应用程序图标

+   一个包含一系列不同大小的*.png*文件的*icons*目录，供 Linux 使用

可选地，我们还可以通过添加具有*background.png*和*background@2x.png*名称的图标来包含 macOS DMG 的背景图像，适用于视网膜屏幕。

现在，在我们的*package.json*文件中，我们更新`build`对象以指定构建资源目录的名称：

```
"build": {
  "appId": "io.jseverywhere.notedly",
  "productName": "Notedly",
  "directories": {
    "buildResources": "resources"
  }
},
```

现在，当我们构建应用程序时，Electron Builder 将使用我们自定义的应用程序图标打包它（见图 20-1）。

![macOS dock 图标的屏幕截图](img/jsev_2001.png)

###### 图 20-1\. 我们在 macOS dock 中的自定义应用程序图标

# 面向多个平台构建

目前，我们只为与我们开发平台匹配的操作系统构建我们的应用程序。作为平台的一个巨大优势之一，Electron 允许我们使用相同的代码来针对多个平台进行目标设置，通过更新我们的 `dist` 脚本来实现这一点。为了实现这一点，Electron Builder 使用了自由开源的 [`electron-build-service`](https://oreil.ly/IEIfW)。我们将使用此服务的公共实例，但组织可以自行托管它，以寻求额外的安全性和隐私。

在我们的 `package.json` 中更新 `dist` 脚本为：

```
"dist": "electron-builder -mwl"
```

这将导致一个面向 macOS、Windows 和 Linux 的构建。从这里，我们可以通过将其作为 GitHub 的发布来分发我们的应用程序，或者通过任何可以分发文件的地方，如 Amazon S3 或我们的 Web 服务器。

# 代码签名

macOS 和 Windows 都包括 *代码签名* 的概念。代码签名有助于提升应用程序的安全性和用户的信任度，因为它有助于表明应用程序的可信度。我不会详细介绍代码签名过程，因为它是特定于操作系统的，并且对开发者来说是有成本的。Electron Builder 文档提供了一篇关于各种平台代码签名的[全面文章](https://oreil.ly/g6wEz)。此外，[Electron 文档](https://oreil.ly/Yb4JF)提供了多个资源和链接。如果您正在构建生产应用程序，我鼓励您进一步研究 macOS 和 Windows 的代码签名选项。

# 结论

我们仅仅涉及了部署 Electron 应用程序的冰山一角。在本章中，我们使用 Electron Builder 来构建我们的应用程序。然后，我们可以轻松地通过任何 Web 主机上传和分发它们。一旦我们超越了这些需求，我们可以使用 Electron Builder 将构建集成到持续交付流水线中；自动将发布推送到 GitHub、S3 或其他分发平台；并将自动更新集成到应用程序中。如果您有兴趣进一步探索 Electron 开发和应用分发的主题，这些都是绝佳的下一步。
