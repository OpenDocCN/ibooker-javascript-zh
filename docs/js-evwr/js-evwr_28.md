# 附录 A. 在本地运行 API

如果您选择参与书中的 UI 部分，但不参与 API 开发章节，仍然需要在本地运行 API 的副本。

第一步是确保您已经在系统上安装并运行了 MongoDB，如第一章所述。数据库运行起来后，您可以克隆 API 的副本并复制最终的代码。要将代码克隆到本地机器上，请打开终端，导航到您保存项目的目录，并使用`git clone`克隆项目仓库。如果尚未这样做，创建一个*notedly*目录以便更好地组织项目代码也可能很有帮助：

```
$ cd Projects
# only run the following mkdir command if you do not yet have a notedly directory
$ mkdir notedly
$ cd notedly
$ git clone git@github.com:javascripteverywhere/api.git
$ cd api
```

最后，您需要通过复制*.sample.env*文件并填写新创建的*.env*文件中的信息来更新您的环境变量。

在您的终端中运行：

```
$ cp .env.example .env
```

现在，在您的文本编辑器中，更新*.env*文件的值：

```
## Database
DB_HOST=mongodb://localhost:27017/notedly
TEST_DB=mongodb://localhost:27017/notedly-test

## Authentication
JWT_SECRET=YOUR_PASSWORD
```

最后，您可以启动 API。在您的终端中运行：

```
$ npm start
```

完成这些指令后，您应该在系统上有一个本地运行的 Notedly API 的副本。
