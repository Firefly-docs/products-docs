# 编译 Linux4.19 固件

## 编译环境搭建

本章介绍 Linux SDK 的编译环境搭建

<font color=red>

**注意：**

**（1）推荐在 X86_64 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。**

**（2）使用普通用户进行编译，不要使用 root 用户权限进行编译。**</font>

### 获取 SDK

首先准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以`~/proj`为例

<font color=red>**不要在虚拟机共享文件夹以及非英文目录存放、解压SDK，避免产生不必要的错误**</font>

获取 SDK 需要先安装：
```
sudo apt update
sudo apt install -y repo git python
```

* 方法一（推荐）

**由于 Firefly_Linux_SDK 源码包比较大，部分用户电脑不支持4G以上文件或单个文件网络传输较慢, 所以我们采用分卷压缩的方法来打包 SDK。用户可以通过如下方式获取 Firefly_Linux_SDK 源码包：**[Firefly_Linux_SDK源码包]()

下载完成后先验证一下 MD5 码：

```bash
$ md5sum rk356x_linux_release_v1.3.0b_20221213_split_dir/*firefly_split*
409b81a9ed3bb9a7d6af91223836cad5  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file00
75cef82f2bf91052a7d3c6f0b8405a89  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file01
6f20f62e9652f8f999692587a2ac4b79  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file02
113acbbcd18d3abe0552ef296e983a3f  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file03
624c88a4da2eaa4a48f380783b126d00  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file04
1cf861afb0b36c9ebcf26a7d6effb260  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file05
3009e46fc14481e77fe7ec143e217de4  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file06
5e1cc90b99e34f20b75fb506d3e9bcd7  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file07
1a512fa7c9e2fd1a0781f8d40e228402  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file08
```

确认无误后，就可以解压：
```bash
# 解压
mkdir ~/proj/
cd ~/proj/
cat path/to/rk356x_linux_release_v1.3.0b_20221213_split_dir/*firefly_split* | tar -xzv

# 导出数据
.repo/repo/repo sync -l
```

* 方法二

通过 repo 拉取代码，此方法对网络要求较高，有条件可以使用

可选择获取完整 SDK 或者 BSP：

```bash
mkdir ~/proj/rk356x_linux_release_v1.3.0b_20221213/
cd ~/proj/rk356x_linux_release_v1.3.0b_20221213/

## 完整 SDK
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_release.xml

## BSP （ 只包含基础仓库和编译工具 ）
## BSP 包括 device/rockchip 、docs 、 kernel 、 u-boot 、 rkbin 、 tools 和交叉编译链
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_bsp_release.xml
```

### 同步代码

执行如下命令同步代码：

```bash
# 进入 SDK 根目录
cd ~/proj/rk356x_linux_release_v1.3.0b_20221213/

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
├── buildroot														# Buildroot 根文件系统编译目录
├── build.sh -> device/rockchip/common/build.sh					# 编译脚本
├── debian															# Debian 根文件系统编译目录
├── device															# 编译相关配置文件
├── docs															# 文档
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel															# Kernel
├── Makefile -> buildroot/build/Makefile
├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh	# 链接脚本
├── prebuilts														# 交叉编译工具链
├── rkbin
├── rkflash.sh -> device/rockchip/common/rkflash.sh				# 烧写脚本
├── tools															# 工具目录
└── u-boot															# U-Boot
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

## 编译 Yocto 固件

### 获取SDK

```bash
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_yocto_kirkstone_release.xml
.repo/repo/repo sync -c
```

### 编译

#### 选择映像
Yocto 项目提供了一些可用于不 layer 的映像。下表列出目前支持构建的映像和相关配方。

| 映像名字  | 描述 | 提供的layer |
| :--------: | :-------: | :-------: |
| core-image-minimal | A small image that only allows a device to boot | Poky |
| core-image-minimal-xfce | A XFCE minimal demo image | meta-openembedded/meta-xfce |
| core-image-sato | Image with Sato, a mobile environment and visual style for mobile devices. The image supports X11 with a Sato theme, Pimlico applications, and contains terminal, editor, and file manager | Poky |
| core-image-weston | A very basic Wayland image with a terminal | Poky |
| core-image-x11 | A very basic X11 image with a terminal | Poky |

### 编译映像文件

**使用 bitbake 命令构建的过程需要保证网络连接正常，如果是中国内陆客户需要保证能 ping 通外网**

进入目录 <path/to/yocto/poky> ，按顺序执行如下命令

```bash
# Install the required environment packages
# sudo apt install zstd
source oe-init-build-env

# 添加 layer（只需要执行一次）
bitbake-layers add-layer ../../meta-openembedded/meta-oe
bitbake-layers add-layer ../../meta-openembedded/meta-python
bitbake-layers add-layer ../../meta-openembedded/meta-networking
bitbake-layers add-layer ../../meta-openembedded/meta-multimedia
bitbake-layers add-layer ../../meta-openembedded/meta-gnome
bitbake-layers add-layer ../../meta-openembedded/meta-xfce
bitbake-layers add-layer ../../meta-clang
bitbake-layers add-layer ../../meta-browser/meta-chromium
bitbake-layers add-layer ../../meta-rockchip
```



选择其中之一命令来编译完整 core-image recipes 。以下是基于 x11 的 core-image 。

```bash
MACHINE= bitbake core-image-minimal
MACHINE= bitbake core-image-minimal-xfce
MACHINE= bitbake core-image-x11
MACHINE= bitbake core-image-sato
```



以下是基于 wayland 的 core-image 。需要在 ```/path/to/yocto/meta-rockchip/conf/machine/include/display.conf``` 修改 DISPLAY_PLATFORM 为 wayland 。修改如下：

```
DISPLAY_PLATFORM ?= "wayland"
# DISPLAY_PLATFORM ?= "x11"
```

完成上述修改后，执行命令编译 core-image-weston

```bash
MACHINE= bitbake core-image-weston
```

注意：如果在已经进行了完整编译一次 core-image 的基础上，需要更换编译的 core-image recipes 。需要将当前编译过 core-image 的清理掉，再开始编译新的 core-image 。

例如：当前编译的是 core-image-minimal 。需要更换成 core-image-sato 。

```bash
MACHINE= bitbake core-image-minimal -c clean
MACHINE= bitbake core-image-sato
```



如果想单独编译部分 recipes 可以参考以下内容：

```bash
# kernel
MACHINE= bitbake linux-rockchip
        
# u-boot
MACHINE= bitbake u-boot-rockchip
        
# rkmpp
MACHINE= bitbake rockchip-mpp
        
# rockchip-librga
MACHINE= bitbake rockchip-librga
        
# 参看更多编译对象
MACHINE= bitbake -s
```



### 调整编译速度

修改文件```/path/to/yocto/meta-rockchip/conf/machine/firefly-rk356x.conf``` 中的 BB_NUMBER_THREADS 和 PARALLEL_MAKE 变量配置。若线程数量设置过大可能会导致机器内存不足，导致编译失败。请根据编译机器的配置来设置编译速度。

```
BB_NUMBER_THREADS = "4"
PARALLEL_MAKE = "-j 4"
```

- [BB_NUMBER_THREADS](https://docs.yoctoproject.org/ref-manual/variables.html#term-BB_NUMBER_THREADS): The maximum number of threads BitBake simultaneously executes.
- [BB_NUMBER_PARSE_THREADS](https://docs.yoctoproject.org/ref-manual/variables.html#term-BB_NUMBER_PARSE_THREADS): The number of threads BitBake uses during parsing.
- [PARALLEL_MAKE](https://docs.yoctoproject.org/ref-manual/variables.html#term-PARALLEL_MAKE): Extra options passed to the `make` command during the [do_compile](https://docs.yoctoproject.org/ref-manual/tasks.html#ref-tasks-compile) task in order to specify parallel compilation on the local build host.
- [PARALLEL_MAKEINST](https://docs.yoctoproject.org/ref-manual/variables.html#term-PARALLEL_MAKEINST): Extra options passed to the `make` command during the [do_install](https://docs.yoctoproject.org/ref-manual/tasks.html#ref-tasks-install) task in order to specify parallel installation on the local build host.

### 更多 bitbake 选项

从根本上说，BitBake 是一个通用任务执行引擎，它允许 shell 和 Python 任务高效并行运行，同时在复杂的任务间依赖约束下工作。 BitBake 的主要用户之一，OpenEmbedded，利用这个核心并使用面向任务的方法构建嵌入式 Linux 软件堆栈。更多详细使用方法请查看[《bitbake-user-manual》](https://docs.yoctoproject.org/bitbake/2.0/bitbake-user-manual/bitbake-user-manual-intro.html#)。

```bash
MACHINE= bitbake <target> <paramater>
# e.g
MACHINE= bitbake u-boot-rockchip -c clean
MACHINE= bitbake u-boot-rockchip
```

| Bitbake paramater  | 描述 |
| :--------: | :-------: |
| -c fetch | 拉取目标所需要的代码 |
| -c clean | 清除目标的输出文件 |
| -c cleanall | 删除目标所有输出文件、共享高速缓存（shared state cache）和源代码 |
| -c compile -f | 使用此选项可在部署映像后强制重新编译，但不建议使用，除非 Yocto Project 不知道目标代码已经发生改变 |
| -c listtasks | 列出目标定义的所有 target |



### 分区固件烧写

编译生成的固件位于目录```<path/to/yocto>/build/tmp/deploy/images/<board>/```

```bash
$ sudo upgrade_tool di -boot boot.img
$ sudo upgrade_tool di -uboot uboot.img
$ sudo upgrade_tool di -misc misc.img
$ sudo upgrade_tool di -recovery recovery.img
```

* 分区烧写适用于调试阶段，固件验证请使用下文的统一固件烧写
* rootfs 不支持单独烧写，需要打包完整固件再烧写

### 统一固件烧写

编译生成的固件位于目录```<path/to/yocto>/build/tmp/deploy/images/<board>/```，待下载的文件为.wic与update.img，进入loader模式后执行如下命令：

```bash
$ sudo upgrade_tool wl 0 <IMAGE NAME>.wic
$ sudo upgrade_tool uf update.img
```
* **固件默认登录账号为：root，密码为：firefly 。固件含有普通用户账号名称为：firefly ，密码为：firefly 。**

注意：如果客户在 Windows PC 上开发，使用 RKdevtool 直接烧录 update.img 即可，**不需要烧录 `<IMAGE NAME>.wic`**。但是要注意一点是 update.img 是一个链接文件，实际得选择链接文件所指向的实际文件。

### 相关概述
Yocto Project 是一个专注于嵌入式 Linux® 操作系统开发的开源协作项目，它提供灵活的工具集和开发环境，允许全球的嵌入式设备开发人员通过共享技术，软件堆栈，配置和用于创建这些定制的Linux映像的最佳实践进行协作。有关 Yocto 项目的更多信息，请参阅 Yocto Project 官网：[www.yoctoproject.org/](www.yoctoproject.org/)。 Yocto Project 官网上有 [Yocto Project Reference Manual](https://docs.yoctoproject.org/current/ref-manual/index.html) 和 [Yocto Project Overview](https://docs.yoctoproject.org/) 等相关文档详细描述了如何构建系统。


### Yocto Project Release layer 介绍

| layer  | 路径 | 优先级(数字越大优先级越高) | 描述 |
| :--------: | :-------: | :-------: | :-------: |
| meta-oe | meta-openembedded/meta-oe | 6 | contains a large amount of additional recipes |
| meta-python | meta-openembedded/meta-python |  7 |  Provide Python recipes|
| meta-qt5 | meta-qt5 | 7 | Provides QT5 recipes |
| meta-clang | meta-clang |  7 |  clang compiler |
| meta-rockchip | meta-rockchip |  9 |  Rockchip board level support available|
| meta | meta |  5 |  Contains the OpenEmbedded-Core metadata|
| meta-poky | meta-poky |  5 |  Holds the configuration for the Poky reference distribution|
| meta-yocto-bsp | meta-yocto-bsp |  5 |  Configuration for the Yocto Project reference hardware board support package.|
| meta-chromium | meta-chromium |  7 |  Provide chromium browser recipe|

