# 编译 Linux 固件

<CodeBlockTabs defaultValue="linux510">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="linux510">Linux 5.10</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="amp">Linux AMP</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="linux510">
## 编译环境搭建 [step]

本章介绍 Linux SDK 的编译环境搭建。

<Callout title="注意" type="warn">

1. 推荐在 X86_64 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。
2. 使用普通用户进行编译，不要使用 root 用户权限进行编译。

</Callout>


**获取 SDK**

首先准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以 `~/proj` 为例。

<Callout title="注意" type="warn">

不要在虚拟机共享文件夹以及非英文目录存放、解压 SDK，避免产生不必要的错误。

</Callout>

获取 SDK 需要先安装：

```bash
sudo apt update
sudo apt install -y repo git python
```

- 方法一（推荐）

由于 Firefly_Linux_SDK 源码包比较大，部分用户电脑不支持 4 GB 以上文件或单个文件网络传输较慢，所以采用分卷压缩的方法来打包 SDK。用户可以通过如下方式获取 Firefly_Linux_SDK 源码包：[rk356x_linux5.10_release_20241220_v1.4.0c](https://community.t-firefly.com/doc/download/180)

下载 SDK 基础包并且按照 README 文档操作：

```text
└── rk356x_linux5.10_release_xxx
    ├── linux_sdk_tar
    │   ├── rk356x_linux5.10_release_xxx.sdk.split00
    │   ├── rk356x_linux5.10_release_xxx.sdk.split01
    │   ├── rk356x_linux5.10_release_xxx.sdk.split02
    │   ├── rk356x_linux5.10_release_xxx.sdk.split03
    ├── md5sum.txt
    ├── README_EN.txt
    ├── README_ZH.txt
    └── sdk_tools.sh
```

- 方法二

通过 repo 拉取代码，此方法对网络要求较高，有条件可以使用。

可选择获取完整 SDK 或者 BSP：

```bash
# 创建 SDK 目录
mkdir ~/proj/rk356x_sdk-linux5.10
cd ~/proj/rk356x_sdk-linux5.10

## 完整 SDK
repo init --no-clone-bundle --repo-url https://github.com/Firefly-rk-linux-utils/git-repo.git --no-repo-verify -u https://github.com/Firefly-rk-linux/manifests.git -b master -m rk356x_linux5.10_release.xml

## BSP （ 只包含基础仓库和编译工具只适合编译 ubuntu、debian 固件 ）
## BSP 包括 device/rockchip 、docs 、 kernel 、 u-boot 、 rkbin 、 tools 和交叉编译链
repo init --no-clone-bundle --repo-url https://github.com/Firefly-rk-linux-utils/git-repo.git --no-repo-verify -u https://github.com/Firefly-rk-linux/manifests.git -b master -m rk356x_linux5.10_bsp_release.xml
```

**同步代码**

执行如下命令同步代码：

```bash
# 进入 SDK 根目录
cd ~/proj/rk356x_sdk-linux5.10

# 同步
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

后续可以使用以下命令更新 SDK：

```bash
.repo/repo/repo sync -c --no-tags
```

因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行。

**目录结构**

```text
.
├── app
├── buildroot														# Buildroot 根文件系统编译目录
├── build.sh -> device/rockchip/common/build.sh					# 编译脚本
├── device															# 编译相关配置文件
├── docs															# 文档
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel															# Kernel
├── Makefile -> device/rockchip/common/Makefile
├── prebuilts														# 交叉编译工具链
├── rkbin
├── tools															# 工具目录
└── u-boot															# U-Boot
```
**安装依赖**

- 方法一：

在 PC 中自行安装环境：
```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake \
unzip device-tree-compiler python-pip ncurses-dev python-pyelftools
```
- 方法二：使用 Docker

使用 dockerfile 创建容器，在容器中进行编译，完美解决编译环境问题，并且与主机环境隔离，互不影响。

首先在主机中安装 docker，请参考：[安装教程](https://wiki.t-firefly.com/zh_CN/Firefly-Linux-Guide/first_use.html#an-zhuang-docker)

创建一个目录作为 docker 工作目录，例如 `~/docker/`，在其中创建文件 `dockerfile`，内容如下：
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

创建镜像：

```bash
cd ~/docker
docker build -t sdkcompiler .
# sdkcompiler 是镜像名称，可随意更改，注意命令最后有一个‘.’
# 此过程需要一段时间，请耐心等待
```

镜像创建完毕后，创建容器并启动：

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
## 编译固件 [step]

<CodeBlockTabs defaultValue="ubuntu">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="ubuntu">ubuntu</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="yocto">yocto</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="buildroot">buildroot</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="ubuntu">
## 编译 Ubuntu 固件

本章介绍 Ubuntu 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

### Ubuntu 固件简单介绍

<Accordions>

<Accordion title="什么是 Ubuntu Minimal ？" type="single" id="ubuntu_intro_minimal">

ROC-RK3568-PC-SE Ubuntu Minimal 系统基于 Ubuntu 64bit 系统构建，目前主要维护 Ubuntu 20.04 版本：没有桌面环境、占用资源少（简化网络管理后仅需 40 MB 内存），针对嵌入式平台精简系统服务，适配 QT、Docker 等开发框架，并提供一系列操作板载资源的接口；系统采用 overlayfs 文件系统，支持导出 rootfs、二次打包、恢复出厂设置等功能。

详细介绍请参阅 [《Firefly Ubuntu Minimal 介绍》](../../ubuntu_minimal_support.md)。

</Accordion>

<Accordion title="什么是 Ubuntu Desktop ？" type="single" id="ubuntu_intro_desktop">

ROC-RK3568-PC-SE Ubuntu Desktop 系统基于 Ubuntu 64bit 系统构建，目前主要维护 Ubuntu 20.04 版本：桌面环境采用 LXQT 或 XFCE4（Firefly 定制主题），Xserver 使用 GPU + RGA 进行 2D 加速，提供基于 Arm Mali GPU 的 OpenGL、OpenCL 支持与基于 Rockchip VPU + Mpp 的视频硬编解码支持，适配 QT、Docker 等开发框架。

详细介绍请参阅 [《Firefly Ubuntu Desktop 介绍》](../../ubuntu_desktop_support.md)。

</Accordion>

</Accordions>

### 编译 SDK

#### 编译前配置 [step]

不同板型的配置文件存放在 `device/rockchip/rk3566_rk3568/` 目录下。

回到 SDK 根目录执行 `build.sh` 选择配置文件：

```bash
./build.sh firefly_rk3568_roc-rk3568-pc-se_ubuntu_defconfig
```

配置文件会链接到 `output/defconfig`，检查该文件可以验证是否配置成功。

相关配置介绍：

```bash
# 预编译的文件系统位置
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk356x_ubuntu_rootfs.img"
# 蓝牙用到的 UART
RK_WIFIBT_TTY="ttyS8"
# 内核配置文件
RK_KERNEL_CFG="firefly_linux_defconfig"
# 设备树名称
RK_KERNEL_DTS_NAME="rk3568-firefly-roc-pc-se"
# ramdisk 镜像
RK_RAMDISK_IMG="ramdisk.img"
# boot 分区镜像的 Image tree source
RK_BOOT_FIT_ITS="bootramdisk.its"
# 预编译的 recovery 镜像
RK_RECOVERY_RAMDISK="rk356x-recovery-arm64.cpio.gz"
# 分区表
RK_PARAMETER="parameter-ubuntu-fit.txt"
# 使用 FIT 格式镜像
RK_USE_FIT_IMG=y
# 使用 extlinux 方式引导系统
USE_EXTBOOT=y
```

#### 选择编译配件 [step]

在 `device/rockchip/rk3566_rk3568/` 目录下，除了 firefly_rk3568_roc-rk3568-pc-se_ubuntu_defconfig 之外，还有其他不同配件搭配的配置文件。

配置文件的名称中会标明所使用的屏幕和摄像头。如果没有标明屏幕，说明使用的是默认 HDMI 显示；如果没有标明摄像头，说明使用的是默认单目摄像头。

```bash
# 例如：
xxxx-ubuntu_defconfig                      # 使用 HDMI + 单目摄像头
xxxx-2cam_ubuntu_defconfig                 # 使用 HDMI + 双目摄像头
xxxx-mipi_ubuntu_defconfig                 # 使用 mipi + 单目摄像头
xxxx-mipi_ubuntu_defconfig                 # 使用 mipi + 双目摄像头
```

选择好配置文件后，执行 `build.sh` 来使其生效：

```bash
# 例如：
./build.sh xxxx-mipi_ubuntu_defconfig
```

#### 下载 Ubuntu 根文件系统 [step]

- 下载根文件系统：[Ubuntu 根文件系统（64 位）](https://community.t-firefly.com/doc/download/180)，放到 SDK 路径下。

- 解压文件：

```bash
# 假设下载的压缩包叫 Ubuntu20.04-xxx_RK3568_KERNEL-5.10_xxx.7z
7z x Ubuntu20.04-xxx_RK3568_KERNEL-5.10_xxx.7z
```

- 将解压出的根文件系统放到 `prebuilt_rootfs/` 目录下：

```bash
mkdir prebuilt_rootfs
# 假设解压出的文件系统叫做 Ubuntu20.04-xxx_RK3568_KERNEL-5.10_xxx.img
mv Ubuntu20.04-xxx_RK3568_KERNEL-5.10_xxx.img prebuilt_rootfs/
cd prebuilt_rootfs
ln -sf Ubuntu20.04-xxx_RK3568_KERNEL-5.10_xxx.img rk356x_ubuntu_rootfs.img
```

#### 全自动编译 [step]

全自动编译会执行所有编译、打包操作，直接生成 RK 固件。

```bash
./build.sh all
```

完整固件会保存到 `output/update/` 目录。

#### 部分编译 [step]

- 编译 u-boot：

```bash
./build.sh uboot
```

生成的镜像为 `u-boot/uboot.img`。

- 编译 kernel：

注意：Firefly kernel 没有开启全部的内核功能，有需求请查看 [Kernel 使用指南](kernel_introduction.md)。

```bash
./build.sh extboot
```

生成的镜像为 `kernel/extboot.img`。

- 打包固件：

```bash
./build.sh updateimg
```

打包固件，生成的完整固件会保存到 `output/update/` 目录。

### 分区说明

#### parameter 分区表

`parameter.txt` 文件中包含了固件的分区信息，以 `parameter-ubuntu-fit.txt` 为例：

路径：`device/rockchip/rk3566_rk3568/parameter-ubuntu-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3568
MACHINE_ID: 007
MANUFACTURER: RK3568
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
GROW_ALIGN: 0
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00020000@0x00048000(recovery),0x00010000@0x00068000(backup),0x00c00000@0x00078000(rootfs),-@0x00c78000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是需要关注的地方，以 uboot 为例，0x00002000@0x00004000(uboot) 中 0x00004000 为 uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。单位是块，每块 512 字节。
  </CodeBlockTab>

  <CodeBlockTab value="yocto">
## 编译 Yocto 固件

### 获取SDK [step]

```bash
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_yocto_kirkstone_release.xml
.repo/repo/repo sync -c
```

### 编译 [step]

#### 选择映像

Yocto 项目提供了一些无需 layer 即可使用的映像。下表列出目前支持构建的映像和相关配方。

| 映像名字 | 描述 | 提供的 layer |
| :--------: | :-------: | :-------: |
| core-image-minimal | A small image that only allows a device to boot | Poky |
| core-image-minimal-xfce | A XFCE minimal demo image | meta-openembedded/meta-xfce |
| core-image-sato | Image with Sato, a mobile environment and visual style for mobile devices. The image supports X11 with a Sato theme, Pimlico applications, and contains terminal, editor, and file manager | Poky |
| core-image-weston | A very basic Wayland image with a terminal | Poky |
| core-image-x11 | A very basic X11 image with a terminal | Poky |

### 编译映像文件 [step]

<Callout title="注意" type="warn">

使用 bitbake 命令构建的过程需要保证网络连接正常，如果是中国内陆客户需要保证能 ping 通外网。

</Callout>

进入目录 `<path/to/yocto/poky>`，按顺序执行如下命令：

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

选择其中一条命令来编译完整的 core-image recipes。以下是基于 x11 的 core-image。

```bash
MACHINE=roc-rk3568-pc-se-kernel5-10 bitbake core-image-minimal
MACHINE=roc-rk3568-pc-se-kernel5-10 bitbake core-image-minimal-xfce
MACHINE=roc-rk3568-pc-se-kernel5-10 bitbake core-image-x11
MACHINE=roc-rk3568-pc-se-kernel5-10 bitbake core-image-sato
```

以下是基于 wayland 的 core-image。需要在 `/path/to/yocto/meta-rockchip/conf/machine/include/display.conf` 修改 DISPLAY_PLATFORM 为 wayland。修改如下：

```ini
DISPLAY_PLATFORM ?= "wayland"
# DISPLAY_PLATFORM ?= "x11"
```

完成上述修改后，执行命令编译 core-image-weston。

```bash
MACHINE=roc-rk3568-pc-se-kernel5-10 bitbake core-image-weston
```

<Callout title="注意" type="warn">

如果在已经完整编译过一次 core-image 的基础上，需要更换编译的 core-image recipes，需要将当前编译过的 core-image 清理掉，再开始编译新的 core-image。

例如：当前编译的是 core-image-minimal，需要更换成 core-image-sato。

</Callout>

```bash
MACHINE=roc-rk3568-pc-se-kernel5-10 bitbake core-image-minimal -c clean
MACHINE=roc-rk3568-pc-se-kernel5-10 bitbake core-image-sato
```

如果想单独编译部分 recipes，可以参考以下内容：

```bash
# kernel
MACHINE=roc-rk3568-pc-se-kernel5-10 bitbake linux-rockchip

# u-boot
MACHINE=roc-rk3568-pc-se-kernel5-10 bitbake u-boot-rockchip

# rkmpp
MACHINE=roc-rk3568-pc-se-kernel5-10 bitbake rockchip-mpp

# rockchip-librga
MACHINE=roc-rk3568-pc-se-kernel5-10 bitbake rockchip-librga

# 参看更多编译对象
MACHINE=roc-rk3568-pc-se-kernel5-10 bitbake -s
```

### 调整编译速度

修改文件 `/path/to/yocto/meta-rockchip/conf/machine/firefly-rk356x-kernel5-10.conf` 中的 `BB_NUMBER_THREADS` 和 `PARALLEL_MAKE` 变量配置。若线程数量设置过大可能会导致机器内存不足，导致编译失败。请根据编译机器的配置来设置编译速度。

```ini
BB_NUMBER_THREADS = "4"
PARALLEL_MAKE = "-j 4"
```

- [BB_NUMBER_THREADS](https://docs.yoctoproject.org/ref-manual/variables.html#term-BB_NUMBER_THREADS): The maximum number of threads BitBake simultaneously executes.
- [BB_NUMBER_PARSE_THREADS](https://docs.yoctoproject.org/ref-manual/variables.html#term-BB_NUMBER_PARSE_THREADS): The number of threads BitBake uses during parsing.
- [PARALLEL_MAKE](https://docs.yoctoproject.org/ref-manual/variables.html#term-PARALLEL_MAKE): Extra options passed to the `make` command during the [do_compile](https://docs.yoctoproject.org/ref-manual/tasks.html#ref-tasks-compile) task in order to specify parallel compilation on the local build host.
- [PARALLEL_MAKEINST](https://docs.yoctoproject.org/ref-manual/variables.html#term-PARALLEL_MAKEINST): Extra options passed to the `make` command during the [do_install](https://docs.yoctoproject.org/ref-manual/tasks.html#ref-tasks-install) task in order to specify parallel installation on the local build host.

### 更多 bitbake 选项

从根本上说，BitBake 是一个通用任务执行引擎，它允许 shell 和 Python 任务高效并行运行，同时在复杂的任务间依赖约束下工作。BitBake 的主要用户之一，OpenEmbedded，利用这个核心并使用面向任务的方法构建嵌入式 Linux 软件堆栈。更多详细使用方法请查看 [《bitbake-user-manual》](https://docs.yoctoproject.org/bitbake/2.0/bitbake-user-manual/bitbake-user-manual-intro.html#)。

```bash
MACHINE=roc-rk3568-pc-se-kernel5-10 bitbake <target> <paramater>
# e.g
MACHINE=roc-rk3568-pc-se-kernel5-10 bitbake u-boot-rockchip -c clean
MACHINE=roc-rk3568-pc-se-kernel5-10 bitbake u-boot-rockchip
```

| BitBake parameter | 描述 |
| :--------: | :-------: |
| -c fetch | 拉取目标所需要的代码 |
| -c clean | 清除目标的输出文件 |
| -c cleanall | 删除目标所有输出文件、共享高速缓存（shared state cache）和源代码 |
| -c compile -f | 使用此选项可在部署映像后强制重新编译，但不建议使用，除非 Yocto Project 不知道目标代码已经发生改变 |
| -c listtasks | 列出目标定义的所有 target |

### 分区固件烧写

编译生成的固件位于目录 `<path/to/yocto>/build/tmp/deploy/images/<board>/`。

```bash
$ sudo upgrade_tool di -boot boot.img
$ sudo upgrade_tool di -uboot uboot.img
$ sudo upgrade_tool di -misc misc.img
$ sudo upgrade_tool di -recovery recovery.img
```

- 分区烧写适用于调试阶段，固件验证请使用下文的统一固件烧写。
- rootfs 不支持单独烧写，需要打包完整固件再烧写。

### 统一固件烧写

编译生成的固件位于目录 `<path/to/yocto>/build/tmp/deploy/images/<board>/`，待下载的文件为 `.wic` 与 `update.img`，进入 Loader 模式后执行如下命令：

```bash
$ sudo upgrade_tool wl 0 <IMAGE NAME>.wic
$ sudo upgrade_tool uf update.img
```

- 固件默认登录账号为 root，密码为 firefly。固件含有普通用户账号，名称为 firefly，密码为 firefly。

<Callout title="注意" type="warn">

如果客户在 Windows PC 上开发，使用 RKDevTool 直接烧录 update.img 即可，不需要烧录 `<IMAGE NAME>.wic`。需要注意 update.img 是一个链接文件，实际应选择链接文件所指向的实际文件。

</Callout>

### 相关概述

Yocto Project 是一个专注于嵌入式 Linux® 操作系统开发的开源协作项目，它提供灵活的工具集和开发环境，允许全球的嵌入式设备开发人员通过共享技术，软件堆栈，配置和用于创建这些定制的 Linux 映像的最佳实践进行协作。有关 Yocto 项目的更多信息，请参阅 Yocto Project 官网：[www.yoctoproject.org/](https://www.yoctoproject.org/)。Yocto Project 官网上有 [Yocto Project Reference Manual](https://docs.yoctoproject.org/current/ref-manual/index.html) 和 [Yocto Project Overview](https://docs.yoctoproject.org/) 等相关文档详细描述了如何构建系统。

### Yocto Project Release layer 介绍

| layer | 路径 | 优先级（数字越大优先级越高） | 描述 |
| :--------: | :-------: | :-------: | :-------: |
| meta-oe | meta-openembedded/meta-oe | 6 | contains a large amount of additional recipes |
| meta-python | meta-openembedded/meta-python | 7 | Provide Python recipes |
| meta-qt5 | meta-qt5 | 7 | Provides QT5 recipes |
| meta-clang | meta-clang | 7 | clang compiler |
| meta-rockchip | meta-rockchip | 9 | Rockchip board level support available |
| meta | meta | 5 | Contains the OpenEmbedded-Core metadata |
| meta-poky | meta-poky | 5 | Holds the configuration for the Poky reference distribution |
| meta-yocto-bsp | meta-yocto-bsp | 5 | Configuration for the Yocto Project reference hardware board support package. |
| meta-chromium | meta-chromium | 7 | Provide chromium browser recipe |
  </CodeBlockTab>


  <CodeBlockTab value="buildroot">
## 编译 Buildroot 固件

本章介绍 Buildroot 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

### 编译 SDK

#### 编译前配置 [step]

不同板型的配置文件存放在 `device/rockchip/rk3566_rk3568/` 目录下。

回到 SDK 根目录执行 `build.sh` 选择配置文件：

```bash
./build.sh firefly_rk3568_roc-rk3568-pc-se_buildroot_defconfig
```

配置文件会链接到 `output/defconfig`，检查该文件可以验证是否配置成功。

相关配置介绍：

```bash
# 设备树名称
RK_KERNEL_DTS_NAME="rk3568-firefly-roc-pc-se"
# 内核配置文件
RK_KERNEL_CFG="firefly_linux_defconfig"
# 使用 FIT 格式镜像
RK_USE_FIT_IMG=y
# 蓝牙用到的 UART
RK_WIFIBT_TTY="ttyS8"
# 分区表
RK_PARAMETER="parameter-buildroot-fit.txt"
# boot 分区镜像的 Image tree source
RK_BOOT_FIT_ITS="bootramdisk.its"
# ramdisk 镜像
RK_RAMDISK_IMG="ramdisk.img"
# 使用 extlinux 的方式引导系统
USE_EXTBOOT=y
```

#### 选择编译配件 [step]

在 `device/rockchip/rk3566_rk3568/` 目录下，除了 firefly_rk3568_roc-rk3568-pc-se_buildroot_defconfig 之外，还有其他不同配件搭配的配置文件。

配置文件的名称中会标明所使用的屏幕和摄像头。如果没有标明屏幕，说明使用的是默认 HDMI 显示；如果没有标明摄像头，说明使用的是默认单目摄像头。

```bash
# 例如：
xxxx-buildroot_defconfig             # 使用 HDMI + 单目摄像头
xxxx-2cam-buildroot_defconfig         # 使用 HDMI + 双目摄像头
xxxx-mipi-buildroot_defconfig        # 使用 mipi + 单目摄像头
xxxx-mipi-2cam-buildroot_defconfig    # 使用 mipi + 双目摄像头
```

选择好配置文件后，执行 `build.sh` 来使其生效：

```bash
# 例如：
./build.sh xxxx-mipi_buildroot_defconfig
```

#### 全自动编译 [step]

全自动编译会执行所有编译、打包操作，生成完整固件。

```bash
./build.sh all
```

完整固件会保存到 `output/update/` 目录。

#### 部分编译 [step]

- 编译 u-boot：

```bash
./build.sh uboot
```

生成的镜像为 `u-boot/uboot.img`。

- 编译 kernel：

注意：Firefly kernel 没有开启全部的内核功能，有需求请查看 [Kernel 使用指南](kernel_introduction.md)。

```bash
./build.sh extboot
```

生成的镜像为 `kernel/extboot.img`。

- 编译 recovery：

```bash
./build.sh recovery
```

生成结果在 `output/recovery` 下。

- 编译 Buildroot 根文件系统：

```bash
./build.sh buildroot

# 注：确保作为普通用户编译 Buildroot 根文件系统，避免不必要的错误。
```

编译目录在 `buildroot/output/` 下。

#### 打包固件 [step]

```bash
./build.sh updateimg
```

打包固件，生成的完整固件会保存到 `output/update/` 目录。

### 分区说明

#### parameter 分区表

`parameter.txt` 文件中包含了固件的分区信息，以 `parameter-buildroot-fit.txt` 为例：

路径：`device/rockchip/rk3566_rk3568/parameter-buildroot-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3568
MACHINE_ID: 007
MANUFACTURER: RK3568
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
GROW_ALIGN: 0
CMDLINE: mtdparts=:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00020000@0x00048000(recovery),0x00010000@0x00068000(backup),0x00040000@0x00078000(oem),0x00c00000@0x000b8000(rootfs),-@0x00cb8000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是需要关注的地方，以 uboot 为例，0x00002000@0x00004000(uboot) 中 0x00004000 为 uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。单位是块，每块 512 字节。
  </CodeBlockTab>
</CodeBlockTabs>

  </CodeBlockTab>

  <CodeBlockTab value="amp">
## 概述

AMP（Asymmetric Multi-Processing）系统是一种非对称多核异构系统，即在同一芯片内，通过分组 CPU，并在不同组的 CPU 内运行不同的系统，通过合理分配任务和资源，AMP 系统可以提供更好的性能表现。

RK3568 芯片有 Cortex-A55 * 4 + RISC-V * 1 两类 CPU，四个 Cortex-A55 核心支持 Linux (kernel-4.19)、Baremetal (HAL)、RTOS (RT-Thread)，RISC-V 支持 Baremetal (HAL)。
利用 AMP 特性，可在 Cortex-A55 * 4 运行 Linux，RISC-V * 1 作为辅助核心跑裸机系统；也可在 Cortex-A55 * 3 运行 Linux，Cortex-A55 * 1 作为辅助核心跑裸机或实时系统等多种组合 AMP 构建。

## 编译环境搭建 [step]

本章介绍 Linux SDK 的编译环境搭建。

<Callout title="注意" type="warn">

1. 推荐在 X86_64 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。
2. 使用普通用户进行编译，不要使用 root 用户权限进行编译。

</Callout>


**获取 SDK**

首先准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以 `~/proj` 为例。

<Callout title="注意" type="warn">

不要在虚拟机共享文件夹以及非英文目录存放、解压 SDK，避免产生不必要的错误。

</Callout>

获取 SDK 需要先安装：

```bash
sudo apt update
sudo apt install -y repo git python
```

- 方法一（推荐）

由于 Firefly_Linux_SDK 源码包比较大，部分用户电脑不支持 4 GB 以上文件或单个文件网络传输较慢，所以采用分卷压缩的方法来打包 SDK。用户可以通过如下方式获取 Firefly_Linux_SDK 源码包：[rk356x_amp_release_20240607_v0.0.1a](https://community.t-firefly.com/doc/download/180)

下载完成后，提取 SDK：

```bash
# 给脚本添加执行权限
cd /path/to/rk356x_amp_release_20240607_v0.0.1a
chmod +x ./sdk_tools.sh

# 创建 SDK 目录
mkdir -p ~/proj/rk356x_amp_sdk

# 校验并解压
./sdk_tools.sh --unpack -C ~/proj/rk356x_amp_sdk

# 还原工作目录
./sdk_tools.sh --sync -C ~/proj/rk356x_amp_sdk
```

- 方法二

通过 repo 拉取代码，此方法对网络要求较高，有条件可以使用。

可选择获取完整 SDK 或者 BSP：

```bash
# 创建 SDK 目录
mkdir ~/proj/rk356x_amp_sdk
cd ~/proj/rk356x_amp_sdk

## 完整 SDK
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_amp_release.xml

## BSP （ 只包含基础仓库和编译工具 ）
## BSP 包括 device/rockchip 、docs 、 kernel 、 u-boot 、hal、 rt-thread 、 rkbin 、 tools 和交叉编译链
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_amp_bsp_release.xml
```

**同步代码**

执行如下命令同步代码：

```bash
# 进入 SDK 根目录
cd ~/proj/rk356x_amp_sdk

# 同步
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

后续可以使用以下命令更新 SDK：

```bash
.repo/repo/repo sync -c --no-tags
```

因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行。

**目录结构**

```text
.
├── app
├── buildroot   # Buildroot 根文件系统编译目录
├── build.sh    # 编译脚本
├── device      # 编译相关配置文件
├── docs        # SDK 相关文档
├── envsetup.sh
├── external
├── hal
│ ├── doc       # HAL 相关文档
│ ├── html      # HAL doxygen 生成驱动文档
│ ├── lib       # HAL 驱动代码
│ ├── project   # HAL 工程目录
│ ├── test      # HAL 测试代码
│ └── tools     # HAL 相关工具
├── kernel      # Kernel相关代码
├── rt-thread   # RT-Thread相关代码
├── prebuilts   # SDK 编译工具链(包括编译U-Boot、Kernel、HAL编译工具链)
├── rkbin       # SDK 相关Binary文件（包括Loader、Trust等打包用基础bin文件）
├── tools
│ ├── linux     # Linux 平台烧录工具
│ ├── mac       # Mac 平台烧录工具
│ └── windows   # Windows 平台烧录工具及驱动
└── u-boot      # U-boot相关代码
```
**安装依赖**

- 方法一：

在 PC 中自行安装环境：
```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake \
unzip device-tree-compiler python-pip ncurses-dev python-pyelftools scons \
clang-format astyle libncurses5-dev build-essential python-configparser
```
- 方法二：使用 Docker

使用 dockerfile 创建容器，在容器中进行编译，完美解决编译环境问题，并且与主机环境隔离，互不影响。

首先在主机中安装 docker，请参考：[安装教程](https://wiki.t-firefly.com/zh_CN/Firefly-Linux-Guide/first_use.html#an-zhuang-docker)

创建一个目录作为 docker 工作目录，例如 `~/docker/`，在其中创建文件 `dockerfile`，内容如下：
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
	qemu-user-static live-build bison flex fakeroot cmake scons \
	unzip device-tree-compiler python-pip ncurses-dev python-pyelftools \
	subversion asciidoc w3m dblatex graphviz python-matplotlib cpio \
	libparse-yapp-perl default-jre patchutils swig expect-dev u-boot-tools \
    clang-format astyle libncurses5-dev build-essential python-configparser

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

创建镜像：

```bash
cd ~/docker
docker build -t sdkcompiler .
# sdkcompiler 是镜像名称，可随意更改，注意命令最后有一个‘.’
# 此过程需要一段时间，请耐心等待
```

镜像创建完毕后，创建容器并启动：

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
## 编译固件 [step]

<CodeBlockTabs defaultValue="ubuntu">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="ubuntu">ubuntu</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="buildroot">buildroot</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="ubuntu">
## 编译 Ubuntu 固件

本章介绍 Ubuntu 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

<Accordions>

<Accordion title="什么是 Ubuntu Minimal 固件？" type="single" id="amp_ubuntu_intro_minimal">

[什么是 Ubuntu Minimal 固件](../../ubuntu_minimal_support.md)？

</Accordion>

<Accordion title="什么是 Ubuntu Desktop 固件？" type="single" id="amp_ubuntu_intro_desktop">

[什么是 Ubuntu Desktop 固件？](../../ubuntu_desktop_support.md)

</Accordion>

</Accordions>

### 编译 SDK

#### 编译前配置 [step]

不同板型的配置文件存放在 `device/rockchip/rk356x/` 目录下。

回到 SDK 根目录执行 `build.sh` 选择配置文件：

```bash
./build.sh roc-rk3568-pc-se-amp-ubuntu.mk
```

#### 下载 Ubuntu 根文件系统 [step]

- 下载根文件系统：[Ubuntu 根文件系统（64 位）](https://community.t-firefly.com/doc/download/180)，放到 SDK 路径下。

- 解压文件：

```bash
7z x ubuntu-aarch64-rootfs.7z
```

- 将根文件系统放到 `ubuntu_rootfs/` 目录下：

```bash
mkdir ubuntu_rootfs
mv ubuntu-aarch64-rootfs.img ubuntu_rootfs/rk356x_ubuntu_rootfs.img
```

#### SDK统一编译与打包 [step]

RK3568 支持 Linux、HAL、RT-Thread 的 AMP 混合架构设计，使得不同的 CPU 可以运行不同的系统，以满足灵活的产品设计需求。该 SDK 目前默认使用 Linux + RT-Thread 的混合结构模型，其中运行 Linux 的 CPU 为主核，运行 rtt 的 CPU 为从核；0~2 核心运行 Linux，3 核心运行 RT-Thread。

##### 编译配置

SDK 的统一编译配置脚本位于 `device/rockchip/rk356x/` 目录下，编译配置脚本内容包括 U-Boot、Kernel、HAL、RT-Thread 的配置，以及 AMP 相关的 CPU 分配、内存分配等配置。用户可以根据需求增加或者修改配置脚本文件，以满足自己的编译需求。主要的 AMP 配置文件如下：

```bash
rk3568_amp_linux.its                    # linux+hal的AMP打包ITS配置文件
rk3568_amp_linux_rtt.its                # linux+rtt的AMP打包ITS配置文件
firefly-rk3568-amp.mk                   # linux+hal对应的AMP配置文件，也是AMP基础配置文件
firefly-rk3568-linux-rtt.mk             # linux+rtt对应的AMP配置文件
roc-rk3568-pc-se-amp-ubuntu.mk                 # ROC-RK3568-PC-SE对应板级的配置脚本文件
```

统一编译脚本工具支持一键编译及打包 U-Boot、Kernel、HAL、RT-Thread、ROOTFS 等，并生成对应的 Image 镜像。

```bash
./build.sh
```

#### 部分编译 [step]

- 编译 u-boot：

```bash
./build.sh uboot
```

- 编译 kernel：

注意：kernel 设备树位于 `kernel/arch/arm64/boot/dts/rockchip/` 目录下，它使用的设备树 rk3568-firefly-roc-pc-se-amp.dts 只适用于 2G 内存规格。如果需要适用于其他内存规格设备，需自行更改它的内存描述标签 `memory`。

<Callout title="注意" type="warn">

AMP 使用到的硬件资源需在内核设备树关闭，并在 `rk3568-amp.dtsi` 设备树的 `rockchip_amp` 标签下声明。

</Callout>

```bash
./build.sh kernel
```

生成的镜像为 `kernel/boot.img`。

- 编译 recovery：

```bash
./build.sh recovery
```

- 编译AMP：编译 hal 或 rt-thread 制作 amp 镜像。

<CodeBlockTabs defaultValue="hal">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="hal">使用 hal</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="rtthread">使用 rt-thread</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="hal">

- 编译 hal：

```bash
./build.sh hal 3
```

生成的镜像为 `hal/project/rk3568/GCC/hal3.bin`，会被链接到 `rockdev/amp/cpu3.bin`。

  </CodeBlockTab>

  <CodeBlockTab value="rtthread">

- 编译 rt-thread：

```bash
./build.sh rtthread 3
```

生成的镜像为 `rt-thread/bsp/rockchip/rk3568-32/Image/rtt3.bin`，会被链接到 `rockdev/amp/cpu3.bin`。

  </CodeBlockTab>
</CodeBlockTabs>

**打包 amp 固件**

- 打包 amp 镜像：

```bash
./build.sh ampimg
```

使用 `rockdev/amp/amp.its` 配置文件打包出 `rockdev/amp/amp.img`。

**其他**

hal 或 rt-thread 默认使用 uart4 作为调试串口，波特率为 115200。

- 更加详细说明请阅读 `SDK/docs/amp/manuals/rk3568` 下的 amp 相关文档。

#### 打包固件

- 更新链接：

更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

- 打包固件：

打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```

### 分区说明

<Accordions>

<Accordion title="parameter 分区表" type="single" id="amp_ubuntu_partition_parameter">

`parameter.txt` 文件中包含了固件的分区信息，以 `parameter-amp-ubuntu-fit.txt` 为例：

路径：`device/rockchip/rk356x/parameter-amp-ubuntu-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3568
MACHINE_ID: 007
MANUFACTURER: RK3568
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00001000@0x00006000(misc),0x00001000@0x00007000(amp),0x00040000@0x00008000(boot:bootable),0x00020000@0x00048000(recovery),0x00010000@0x00068000(backup),0x49dc00@0x00078000(rootfs),-@0x515c00(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是需要关注的地方，以 uboot 为例，0x00002000@0x00004000(uboot) 中 0x00004000 为 uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

</Accordion>

<Accordion title="package-file" type="single" id="amp_ubuntu_partition_packagefile">

`package-file` 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 `parameter.txt` 文件保持一致。

路径：`tools/linux/Linux_Pack_Firmware/rockdev/rk356x-amp-ubuntu-package-file`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
uboot           Image/uboot.img
misc            Image/misc.img
amp             Image/amp.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
userdata        RESERVED
backup          RESERVED
```

</Accordion>

</Accordions>
  </CodeBlockTab>

  <CodeBlockTab value="buildroot">
## 编译 Buildroot 固件

本章介绍 Buildroot 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

### 编译 SDK

#### 编译前配置 [step]

不同板型的配置文件存放在 `device/rockchip/rk356x/` 目录下。

回到 SDK 根目录执行 `build.sh` 选择配置文件：

```bash
./build.sh roc-rk3568-pc-se-amp-buildroot.mk
```

#### SDK统一编译与打包 [step]

RK3568 支持 Linux、HAL、RT-Thread 的 AMP 混合架构设计，使得不同的 CPU 可以运行不同的系统，以满足灵活的产品设计需求。该 SDK 目前默认使用 Linux + RT-Thread 的混合结构模型，其中运行 Linux 的 CPU 为主核，运行 rtt 的 CPU 为从核；0~2 核心运行 Linux，3 核心运行 RT-Thread。

##### 编译配置

SDK 的统一编译配置脚本位于 `device/rockchip/rk356x/` 目录下，编译配置脚本内容包括 U-Boot、Kernel、HAL、RT-Thread 的配置，以及 AMP 相关的 CPU 分配、内存分配等配置。用户可以根据需求增加或者修改配置脚本文件，以满足自己的编译需求。主要的 AMP 配置文件如下：

```bash
rk3568_amp_linux.its                    # linux+hal的AMP打包ITS配置文件
rk3568_amp_linux_rtt.its                # linux+rtt的AMP打包ITS配置文件
firefly-rk3568-amp.mk                   # linux+hal对应的配置文件，也是AMP基础配置文件
firefly-rk3568-linux-rtt.mk             # linux+rtt对应的配置文件
roc-rk3568-pc-se-amp-buildroot.mk              # ROC-RK3568-PC-SE对应的配置脚本文件
```

统一编译脚本工具支持一键编译及打包 U-Boot、Kernel、HAL、RT-Thread、ROOTFS 等，并生成对应的 Image 镜像。

```bash
./build.sh
```

#### 部分编译 [step]

- 编译 u-boot：

```bash
./build.sh uboot
```

- 编译 kernel：

注意：kernel 设备树位于 `kernel/arch/arm64/boot/dts/rockchip/` 目录下，它使用的设备树 rk3568-firefly-roc-pc-se-amp.dts 只适用于 2G 内存规格。如果需要适用于其他内存规格设备，需自行更改它的内存描述标签 `memory`。

<Callout title="注意" type="warn">

AMP 使用到的硬件资源需在内核设备树关闭，并在 `rk3568-amp.dtsi` 设备树的 `rockchip_amp` 标签下声明。

</Callout>

```bash
./build.sh kernel
```

- 编译 recovery：

```bash
./build.sh recovery
```

- 编译 Buildroot 根文件系统：

编译 Buildroot 根文件系统，将会在 `buildroot/output` 生成编译输出目录：

```bash
./build.sh buildroot

# 注：确保作为普通用户编译 Buildroot 根文件系统，避免不必要的错误。
```

- 编译AMP：编译 hal 或 rt-thread 制作 amp 镜像。

<CodeBlockTabs defaultValue="hal">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="hal">使用 hal</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="rtthread">使用 rt-thread</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="hal">

- 编译 hal：

```bash
./build.sh hal 3
```

生成的镜像为 `hal/project/rk3568/GCC/hal3.bin`，会被链接到 `rockdev/amp/cpu3.bin`。

  </CodeBlockTab>

  <CodeBlockTab value="rtthread">

- 编译 rt-thread：

```bash
./build.sh rtthread 3
```

生成的镜像为 `rt-thread/bsp/rockchip/rk3568-32/Image/rtt3.bin`，会被链接到 `rockdev/amp/cpu3.bin`。

  </CodeBlockTab>
</CodeBlockTabs>

**打包 amp 固件**

- 打包 amp 镜像：

```bash
./build.sh ampimg
```

使用 `rockdev/amp/amp.its` 配置文件打包出 `rockdev/amp/amp.img`。

**其他**

hal 或 rt-thread 默认使用 uart4 作为调试串口，波特率为 115200。

- 更加详细说明请阅读 `SDK/docs/amp/manuals/rk3568` 下的 amp 相关文档。

#### 打包固件

更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```

### 分区说明

<Accordions>

<Accordion title="parameter 分区表" type="single" id="amp_buildroot_partition_parameter">

`parameter.txt` 文件中包含了固件的分区信息，以 `parameter-amp-buildroot-fit.txt` 为例：

路径：`device/rockchip/rk356x/parameter-amp-buildroot-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3568
MACHINE_ID: 007
MANUFACTURER: RK3568
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00010000@0x00008000(boot),0x00010000@0x00018000(recovery),0x00010000@0x00028000(backup),0x00040000@0x00038000(oem),0x00c00000@0x00078000(rootfs),-@0x00c78000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是需要关注的地方，以 uboot 为例，0x00002000@0x00004000(uboot) 中 0x00004000 为 uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

</Accordion>

<Accordion title="package-file" type="single" id="amp_buildroot_partition_packagefile">

`package-file` 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 `parameter.txt` 文件保持一致。

路径：`tools/linux/Linux_Pack_Firmware/rockdev/rk356x-amp-package-file`

```bash
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
uboot           Image/uboot.img
misc            Image/misc.img
amp             Image/amp.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
oem             Image/oem.img
userdata        Image/userdata.img
backup          RESERVED
```

</Accordion>

</Accordions>
  </CodeBlockTab>
</CodeBlockTabs>

  </CodeBlockTab>
</CodeBlockTabs>