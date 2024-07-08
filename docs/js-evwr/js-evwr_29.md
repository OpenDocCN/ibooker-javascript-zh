# 附录 B. 在本地运行 Web 应用

如果你选择跟随本书的 Electron 部分，但不涉及 web 开发章节，你仍然需要在本地运行一个 web 应用的副本。

第一步是确保你在本地运行了 API 的副本。如果你还没有，请参考附录 Appendix A 了解如何在本地运行 API。

当你的 API 运行起来后，你可以克隆 web 应用的一个副本。为了将代码克隆到你的本地机器上，打开终端，导航到你项目存放的目录，并 **`git clone`** 项目仓库：

```
$ cd Projects
# if keeping your projects in a notedly folder, cd into the notedly directory
$ cd notedly
$ git clone git@github.com:javascripteverywhere/web.git
$ cd web
```

接下来，你需要通过复制 *.sample.env* 文件并填写新创建的 *.env* 文件来更新你的环境变量信息。

在你的终端中运行：

```
$ cp .env.example .env
```

现在，在你的文本编辑器中，更新 *.env* 文件的数值，确保它与你本地运行的 API 的 URL 匹配。如果所有数值都保持默认，你不需要做任何更改。

```
API_URI=http://localhost:4000/api
```

最后，你可以运行最终的 web 代码示例。在你的终端应用中运行：

```
$ npm run final
```

按照这些指示操作后，你应该在你的系统上本地运行一个 Notedly web 应用的副本。
