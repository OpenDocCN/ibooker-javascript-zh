# 附录 A. 安装 HAProxy

HAProxy 是一个反向代理，用于在将请求传递到应用程序代码之前拦截请求。在本书中，它用于卸载一些否则不应由 Node.js 进程处理的任务。

如果你使用 Linux，你有几个选项。第一种选择是尝试使用你发行版的软件安装程序直接安装`haproxy`。这可能像**`sudo apt install haproxy`**这样简单。但是，这可能会安装一个版本过旧的 HAProxy。如果你的发行版提供的 HAProxy 版本早于 *v2*，你可以在安装后运行**`haproxy -v`**来检查，那么你需要以另一种方式安装它。

# Linux：从源代码构建

这种方法将从[*http://haproxy.org*](http://haproxy.org)网站下载官方源代码包。然后，解压内容，编译应用程序并执行安装。此方法还将安装`man`页，提供有用的文档。运行以下命令下载并编译 HAProxy：

```
$ sudo apt install libssl-dev # Debian / Ubuntu
$ curl -O http://www.haproxy.org/download/2.1/src/haproxy-2.1.8.tar.gz
$ tar -xf haproxy-2.1.8.tar.gz
$ cd haproxy-2.1.8
$ make -j4 TARGET=linux-glibc USE_ZLIB=yes USE_OPENSSL=yes
$ sudo make install
```

如果在编译过程中出现错误，则可能需要使用你发行版的包管理器安装缺少的软件包。

# Linux：安装预编译的二进制文件

然而，如果你希望避免编译软件的过程，可以选择下载预编译的二进制文件。我找不到官方的预编译版本，因此这里是我本地编译并上传到我的 Web 服务器的版本。运行以下命令下载、解压和安装预编译的二进制文件：

```
$ curl -O https://thomashunter.name/pkg/haproxy-2.1.8-linux.tar.gz
$ tar -xf haproxy-2.1.8-linux.tar.gz
$ ./haproxy -v # test
$ sudo mv ./haproxy /usr/bin/haproxy
$ sudo chown root:root /usr/bin/haproxy
```

# macOS：通过 Homebrew 安装

如果你使用 macOS，我强烈建议安装[Homebrew](https://brew.sh)，如果你还没有安装的话。Homebrew 通常提供最新版本的软件，并包含现代版本的 HAProxy。使用 Homebrew，你可以通过运行以下命令安装 HAProxy：

```
$ brew install haproxy@2.1.8
$ haproxy -v # test
```
