# 编译 Linux 固件 (内核版本 5.10)

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
## 编译 Ubuntu 固件

本章介绍 Ubuntu 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

### Ubuntu 固件简单介绍

[什么是 Ubuntu Minimal ？](ubuntu_minimal_support.md)

[什么是 Ubuntu Desktop ？](ubuntu_desktop_support.md)

### 编译 SDK

#### 编译前配置

不同板型的配置文件存放在`device/rockchip/rk3562/`目录下

回到 SDK 根目录执行`build.sh`选择配置文件：

```bash
./build.sh firefly_rk3562_aio-3562jq_ubuntu_defconfig
```

配置文件会链接到 `output/defconfig`，检查该文件可以验证是否配置成功。

相关配置介绍：

```bash
# 预编译的文件系统位置
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk3562_ubuntu_rootfs.img"
# 蓝牙用到的 UART
RK_WIFIBT_TTY="ttyS4"
# 内核配置文件
RK_KERNEL_CFG="firefly_linux_defconfig"
# 设备树名称
RK_KERNEL_DTS_NAME="rk3562-firefly-aio-3562jq"
# ramdisk 镜像
RK_RAMDISK_IMG="ramdisk.img"
# boot 分区镜像的 Image tree source
RK_BOOT_FIT_ITS="bootramdisk.its"
# 预编译的 recovery 镜像
RK_RECOVERY_RAMDISK="rk3562-recovery-arm64.cpio.gz"
# 分区表
RK_PARAMETER="parameter-ubuntu-fit.txt"
# 使用 FIT 格式镜像
RK_USE_FIT_IMG=y
# 使用 extlinux 方式引导系统
USE_EXTBOOT=y
```

#### 下载 Ubuntu 根文件系统

* 下载根文件系统：[Ubuntu 根文件系统(64位)](https://www.t-firefly.com/doc/download/247.html#other_774)，放到 SDK 路径下

* 解压文件

```bash
# 假设下载的压缩包叫 Ubuntu20.04-xxx_RK3562_xxx.7z
7z x Ubuntu20.04-xxx_RK3562_xxx.7z
```

* 将解压出的根文件系统放到 `prebuilt_rootfs/` 目录下

```bash
mkdir prebuilt_rootfs
# 假设解压出的文件系统叫做 Ubuntu20.04-xxx_RK3562_xxx.img
mv Ubuntu20.04-xxx_RK3562_xxx.img prebuilt_rootfs/
cd prebuilt_rootfs
ln -sf Ubuntu20.04-xxx_RK3562_xxx.img rk3562_ubuntu_rootfs.img
```

#### 全自动编译

全自动编译会执行所有编译、打包操作，直接生成 RK 固件。

```bash
./build.sh all
```
完整固件会保存到 `output/update/` 目录。

#### 部分编译

* 编译 u-boot

```bash
./build.sh uboot
```
生成的镜像为 u-boot/uboot.img

* 编译 kernel

注意：Firefly kernel 没有开启全部的内核功能，有需求请查看[Kernel 使用](kernel_introduction.md)

```bash
./build.sh extboot
```
生成的镜像为 kernel/extboot.img

* 打包固件

```bash
./build.sh updateimg
```
打包固件，生成的完整固件会保存到 `output/update/` 目录。

### 分区说明

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 parameter-ubuntu-fit.txt 为例：

路径：`device/rockchip/rk3562/parameter-ubuntu-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3562
MACHINE_ID: 007
MANUFACTURER: RK3562
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
GROW_ALIGN: 0
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00040000@0x00048000(recovery),0x00010000@0x00088000(backup),0x00c00000@0x00098000(rootfs),-@0x00c98000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。单位是块，每块 512 字节。
## 编译 Yocto 固件

### 获取SDK

```bash
git clone https://gitlab.com/firefly-linux/git-repo.git
./git-repo/repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3562_yocto_kirkstone_release.xml
.repo/repo/repo sync -c --no-tags
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
bitbake-layers add-layer ../../meta-lts-mixins
bitbake-layers add-layer ../../meta-clang
bitbake-layers add-layer ../../meta-browser/meta-chromium
bitbake-layers add-layer ../../meta-rockchip
```



选择其中之一命令来编译完整 core-image recipes 。以下是基于 x11 的 core-image 。

```bash
MACHINE=aio-3562jq bitbake core-image-minimal
MACHINE=aio-3562jq bitbake core-image-minimal-xfce
MACHINE=aio-3562jq bitbake core-image-x11
MACHINE=aio-3562jq bitbake core-image-sato
```



以下是基于 wayland 的 core-image 。需要在 ```/path/to/yocto/meta-rockchip/conf/machine/include/display.conf``` 修改 DISPLAY_PLATFORM 为 wayland 。修改如下：

```
DISPLAY_PLATFORM ?= "wayland"
# DISPLAY_PLATFORM ?= "x11"
```

完成上述修改后，执行命令编译 core-image-weston

```bash
MACHINE=aio-3562jq bitbake core-image-weston
```

注意：如果在已经进行了完整编译一次 core-image 的基础上，需要更换编译的 core-image recipes 。需要将当前编译过 core-image 的清理掉，再开始编译新的 core-image 。

例如：当前编译的是 core-image-minimal 。需要更换成 core-image-sato 。

```bash
MACHINE=aio-3562jq bitbake core-image-minimal -c clean
MACHINE=aio-3562jq bitbake core-image-sato
```



如果想单独编译部分 recipes 可以参考以下内容：

```bash
# kernel
MACHINE=aio-3562jq bitbake linux-rockchip
        
# u-boot
MACHINE=aio-3562jq bitbake u-boot-rockchip
        
# rkmpp
MACHINE=aio-3562jq bitbake rockchip-mpp
        
# rockchip-librga
MACHINE=aio-3562jq bitbake rockchip-librga
        
# 参看更多编译对象
MACHINE=aio-3562jq bitbake -s
```



### 调整编译速度

修改文件```/path/to/yocto/meta-rockchip/conf/machine/firefly-rk3562.conf``` 中的 BB_NUMBER_THREADS 和 PARALLEL_MAKE 变量配置。若线程数量设置过大可能会导致机器内存不足，导致编译失败。请根据编译机器的配置来设置编译速度。

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
MACHINE=aio-3562jq bitbake <target> <paramater>
# e.g
MACHINE=aio-3562jq bitbake u-boot-rockchip -c clean
MACHINE=aio-3562jq bitbake u-boot-rockchip
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

## 编译 Buildroot 固件

本章介绍 Buildroot 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

### 编译 SDK

#### 编译前配置

不同板型的配置文件存放在`device/rockchip/rk3562/`目录下

回到 SDK 根目录执行`build.sh`选择配置文件：

```bash
./build.sh firefly_rk3562_aio-3562jq_buildroot_defconfig
```

配置文件会链接到 `output/defconfig`，检查该文件可以验证是否配置成功。

相关配置介绍：

```bash
# 设备树名称
RK_KERNEL_DTS_NAME="rk3562-firefly-aio-3562jq"
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

#### 全自动编译

全自动编译会执行所有编译、打包操作，生成完整固件。

```bash
./build.sh all
```
完整固件会保存到 `output/update/` 目录。

#### 部分编译

* 编译 u-boot

```bash
./build.sh uboot
```
生成的镜像为 u-boot/uboot.img

* 编译 kernel

注意：Firefly kernel 没有开启全部的内核功能，有需求请查看[Kernel 使用](kernel_introduction.md)

```bash
./build.sh extboot
```
生成的镜像为 kernel/extboot.img

* 编译 recovery

```bash
./build.sh recovery
```
生成结果在 output/recovery 下

* 编译 Buildroot 根文件系统

```bash
./build.sh buildroot

# 注：确保作为普通用户编译 Buildroot 根文件系统，避免不必要的错误。
```
编译目录在 buildroot/output/ 下

#### 打包固件

```bash
./build.sh updateimg
```
打包固件，生成的完整固件会保存到 `output/update/` 目录。

### 分区说明

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 `parameter-buildroot-fit.txt` 为例：

路径：`device/rockchip/rk3562/parameter-buildroot-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3562
MACHINE_ID: 007
MANUFACTURER: RK3562
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00040000@0x00048000(recovery),0x00010000@0x00088000(backup),0x00040000@0x00098000(oem),0x00c00000@0x000d8000(rootfs),-@0x00cd8000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。单位是块，每块 512 字节。