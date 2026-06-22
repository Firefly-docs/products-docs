## 编译环境搭建

本章介绍 Linux SDK 的编译环境搭建

<font color=red>

**注意：**

**（1）推荐在 X86_64 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。**

**（2）使用普通用户进行解压和编译，不要使用 root 用户权限进行解压和编译。不要解压到虚拟机的共享文件夹。**</font>


### 获取 SDK

首先准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以`~/proj`为例

<font color=red>**不要在虚拟机共享文件夹以及非英文目录存放、解压SDK，避免产生不必要的错误**</font>

获取 SDK 需要先安装：
```
sudo apt update
sudo apt install -y repo git python
```

* 方法一（推荐）

**由于 Firefly_Linux_SDK 源码包比较大，部分用户电脑不支持4G以上文件或单个文件网络传输较慢, 所以我们采用分卷压缩的方法来打包 SDK。请联系商务 sales@t-firefly.com 获取下载地址。**

下载完成后，提取 SDK：

```bash
# 给脚本添加执行权限
cd /path/to/rk3562_linux_release
chmod +x ./sdk_tools.sh

# 创建 SDK 目录
mkdir -p ~/proj/rk3562_sdk

# 校验并解压
./sdk_tools.sh --unpack -C ~/proj/rk3562_sdk

# 还原工作目录
./sdk_tools.sh --sync -C ~/proj/rk3562_sdk
```

* 方法二

通过 repo 拉取代码，此方法对网络要求较高，有条件可以使用

可选择获取完整 SDK 或者 BSP：

```bash
# 创建 SDK 目录
mkdir ~/proj/rk3562_sdk
cd ~/proj/rk3562_sdk

# 请联系商务 sales@t-firefly.com 获取 repo 链接。
```

### 同步代码

执行如下命令同步代码：

```bash
# 进入 SDK 根目录
cd ~/proj/rk3562_sdk

# 同步
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

后续可以使用以下命令更新 SDK：

```bash
.repo/repo/repo sync -c --no-tags
```

因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行。

### 目录结构

```bash
.
├── app
├── buildroot                                               # buildroot 仓库
├── build.sh -> device/rockchip/common/scripts/build.sh     # 编译脚本
├── device/rockchip                                         # 管理编译与配置的仓库
├── docs                                                    # 文档
├── external                                                # 各种组件
├── kernel                                                  # 内核仓库
├── Makefile -> device/rockchip/common/Makefile
├── output                                                  # 编译输出目录
├── prebuilt_rootfs                                         # 预编译的 rootfs 存放目录
├── prebuilts/gcc/linux-x86/                                # 交叉编译工具链
├── README.md -> device/rockchip/common/README.md
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
├── rockdev -> output/firmware                              # 存放编译出的各种分区镜像
├── tools                                                   # 各种工具，如烧录工具等
└── u-boot                                                  # uboot 仓库
```

### 安装依赖

* 方法一：

在 PC 中自行安装环境：
```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake \
unzip device-tree-compiler python-pip ncurses-dev python-pyelftools
```
* 方法二：使用 Docker

使用 dockerfile 创建容器，在容器中进行编译，完美解决编译环境问题，并且与主机环境隔离，互不影响。

首先在主机中安装 docker，请参考：[安装教程](https://wiki.t-firefly.com/zh_CN/Firefly-Linux-Guide/first_use.html#an-zhuang-docker)

创建一个目录作为 docker 工作目录，例如`~/docker/`，在其中创建文件`dockerfile`，内容如下：
```dockerfile
FROM ubuntu:18.04
MAINTAINER firefly "service@t-firefly.com"

ENV DEBIAN_FRONTEND=noninteractive

RUN cp -a /etc/apt/sources.list /etc/apt/sources.list.bak
RUN sed -i 's@http://.*ubuntu.com@http://repo.huaweicloud.com@g' /etc/apt/sources.list

RUN apt update

RUN apt install -y build-essential crossbuild-essential-arm64 \
	bash-completion vim sudo locales time rsync bc python

RUN apt install -y repo git ssh libssl-dev liblz4-tool lib32stdc++6 \
	expect patchelf chrpath gawk texinfo diffstat binfmt-support \
	qemu-user-static live-build bison flex fakeroot cmake \
	unzip device-tree-compiler python-pip ncurses-dev python-pyelftools \
	subversion asciidoc w3m dblatex graphviz python-matplotlib cpio \
	libparse-yapp-perl default-jre patchutils swig expect-dev u-boot-tools

RUN apt update && apt install -y -f

# language support
RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8

# switch to a no-root user
RUN useradd -c 'firefly user' -m -d /home/firefly -s /bin/bash firefly
RUN sed -i -e '/\%sudo/ c \%sudo ALL=(ALL) NOPASSWD: ALL' /etc/sudoers
RUN usermod -a -G sudo firefly

USER firefly
WORKDIR /home/firefly
```
创建镜像
```bash
cd ~/docker
docker build -t sdkcompiler .
# sdkcompiler 是镜像名称，可随意更改，注意命令最后有一个‘.’
# 此过程需要一段时间，请耐心等待
```
镜像创建完毕后，创建容器并启动
```bash
# 此处将主机内 SDK 所在文件夹挂载到容器内，这样容器内就能访问主机中的 SDK 了
# source= 填 SDK 所在目录；target= 填容器内的一个目录，必须是空目录
# ubuntu18 是容器名，firefly 是容器 hostname，均可随意更改
# sdkcompiler 是上一步的镜像名
docker run --privileged --mount type=bind,source=/home/fierfly/proj,target=/home/firefly/proj --name="ubuntu18" -h firefly -it sdkcompiler
```
现在就可以在容器中进行 SDK 的编译了。

退出容器、重启容器的方法：
```bash
# 在容器内输入 exit 即可退出

# 查看所有容器（包括已退出的）
docker ps -a

# 重启一个退出的容器并连接
docker start ubuntu18 # 容器名
docker attach ubuntu18
```