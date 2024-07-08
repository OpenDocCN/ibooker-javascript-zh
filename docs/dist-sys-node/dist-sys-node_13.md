# 附录 B. 安装 Docker

Docker 是一个在特定机器上运行应用程序的工具。在本书中，它被用于运行各种数据库、第三方服务，甚至是您编写的应用程序。Docker 维护一个[安装 Docker 引擎](https://docs.docker.com/engine/install/)页面，但 macOS 和 Linux 的说明在此重复列出供您参考。

# macOS：安装 Docker Desktop for Mac

在 macOS 上安装 Docker 的主要方法是安装 Docker Desktop for Mac。这不仅会为您提供 Docker 守护程序和 CLI 工具，还会为您提供一个在菜单栏中运行的 GUI 工具。访问[Docker Desktop for Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac/)页面并下载稳定的磁盘映像，然后按照通常的 macOS 安装流程进行安装。

# Linux：方便的安装脚本

如果您使用基于 Ubuntu 的操作系统，您可以通过将 Docker 存储库添加到系统并使用软件包管理器进行安装。这将允许 Docker 通过正常的软件包升级操作保持更新。

Docker 提供了一个便捷的脚本，可以执行几项任务。首先，它会配置您的 Linux 发行版的软件包管理器以使用 Docker 存储库。该脚本支持多个发行版，如 Ubuntu 和 CentOS。它还将从 Docker 存储库安装必要的软件包到您的本地机器。将来进行软件包升级时，您的机器也将更新 Docker：

```
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

如果您希望在当前帐户中控制 Docker 而无需每次都提供`sudo`，请运行以下命令。第一个命令将把您的用户添加到`docker`组中，第二个命令将在您的终端会话中应用新的组（尽管您需要注销并重新登录以使全局更改生效）：

```
$ sudo usermod -aG docker $USER
$ su - $USER
```

您还需要安装`docker-compose`来运行本书中几个章节的示例。目前需要单独添加它，因为它不在 Docker 存储库中提供。运行以下命令下载预编译的二进制文件并使其可执行：

```
$ sudo curl -L "https://github.com/docker/compose/releases/download\
/1.26.2/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```
