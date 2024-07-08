# 附录 C. 安装 Minikube 和 Kubectl

Kubernetes 是一个完整的容器编排平台，允许工程师在多台机器上运行一组容器。Minikube 是一个简化版本，使得在单台机器上本地运行变得更加简单。官方 Kubernetes 文档维护了一个[安装工具](https://kubernetes.io/docs/tasks/tools/)页面，提供安装说明，但安装说明已在此重复供参考。

# Linux：Debian 软件包和预编译二进制文件

Minikube 可以通过安装 Debian 软件包获取（也提供了 RPM 软件包）。另一方面，Kubectl 可以通过下载二进制文件并将其放入 *$PATH* 中进行安装。运行以下命令在基于 Debian 的（包括 Ubuntu）机器上安装 Minikube 和 Kubectl：

```
$ curl -LO https://storage.googleapis.com/minikube/releases\
/latest/minikube_1.9.1-0_amd64.deb
$ sudo dpkg -i minikube_1.9.1-0_amd64.deb
$ curl -LO https://storage.googleapis.com/kubernetes-release\
/release/v1.18.2/bin/linux/amd64/kubectl
$ chmod +x ./kubectl
$ sudo mv kubectl /usr/bin/
```

# macOS：通过 Homebrew 安装

Docker Desktop 已经集成了 Kubernetes！但默认情况下它是禁用的。要启用它，点击菜单栏中的 Docker 图标，然后选择 Preferences 选项。在弹出的屏幕中，点击 Kubernetes 选项卡。最后，选中 Enable Kubernetes 复选框，然后点击 Apply & Restart。可能需要几分钟，但 UI 应该更新并显示 Kubernetes 已在运行。

接下来，您需要安装 Minikube，这是一个工具，简化了在本地计算机上运行 Kubernetes 的一些操作。运行以下命令来完成安装：

```
$ brew install minikube
```
