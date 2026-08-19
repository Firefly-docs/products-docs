# 编译 Linux AMP 固件

## 概述

AMP（Asymmetric Multi-Processing）系统是一种非对称多核异构系统，即在同一芯片内，
通过分组CPU，并在不同组的CPU内运行不同的系统，通过合理分配任务和资源，AMP系统
可以提供更好的性能表现。

rk3568芯片有Cortex-A55 * 4 + RISC-V * 1 两类CPU，四个Cortex-A55核心支持linux(kernel-4.19)、Baremetal(HAL)、RTOS(RT-Thread) , RISC-V支持Baremetal(HAL)。
利用AMP特性，可在Cortex-A55 * 4运行linux， RISC-V * 1作为辅助核心跑裸机系统；也可在Cortex-A55 * 3运行linux， Cortex-A55 * 1作为辅助核心跑裸机或实时系统等多种组合AMP构建。

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

**由于 Firefly_Linux_SDK 源码包比较大，部分用户电脑不支持4G以上文件或单个文件网络传输较慢, 所以我们采用分卷压缩的方法来打包 SDK。用户可以通过如下方式获取 Firefly_Linux_SDK 源码包：**[rk356x_amp_release_20240607_v0.0.1a](https://www.t-firefly.com/doc/download/103.html#other_447)

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

* 方法二

通过 repo 拉取代码，此方法对网络要求较高，有条件可以使用

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

### 同步代码

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

### 目录结构

```bash
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

### 安装依赖

* 方法一：

在 PC 中自行安装环境：
```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake \
unzip device-tree-compiler python-pip ncurses-dev python-pyelftools scons \
clang-format astyle libncurses5-dev build-essential python-configparser
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

[什么是 Ubuntu Minimal固件](ubuntu_minimal_support.md)？

[什么是 Ubuntu Desktop固件？](ubuntu_desktop_support.md)

### 编译 SDK

#### 编译前配置

不同板型的配置文件存放在`device/rockchip/rk356x/`目录下

回到 SDK 根目录执行`build.sh`选择配置文件：

```bash
./build.sh aio-3568j-amp-ubuntu.mk
```

#### 下载 Ubuntu 根文件系统

* 下载根文件系统：[Ubuntu 根文件系统(64位)](https://www.t-firefly.com/doc/download/103.html#other_448)，放到 SDK 路径下

* 解压文件

```bash
7z x ubuntu-aarch64-rootfs.7z
```

* 将根文件系统放到 `ubuntu_rootfs/` 目录下

```bash
mkdir ubuntu_rootfs
mv ubuntu-aarch64-rootfs.img ubuntu_rootfs/rk356x_ubuntu_rootfs.img
```

#### SDK统一编译与打包

RK3568 支持 Linux 、HAL 、RT-Thread的 AMP 混合架构设计，使得不同的 CPU 可以运行不同的系统，以满足灵活
的产品设计需求。该 SDK 目前默认使用 Linux + RT-Thread 的混合结构模型，其中运行 Linux
的 CPU 为主核，运行 rtt 的 CPU 为从核；0~2核心运行Linux，3核心运行RT-Thread。

##### 编译配置

SDK 的统一编译配置脚本位于device/rockchip/rk356x/目录下，编译配置脚本内容包括U-Boot、Kernel、
HAL、RT-Thread的配置，以及AMP相关的CPU分配，内存分配等配置。用户可以根据需求增加或者修改
配置脚本文件，以满足自己的编译需求。主要的AMP配置文件如下:

```bash
rk3568_amp_linux.its                    # linux+hal的AMP打包ITS配置文件
rk3568_amp_linux_rtt.its                # linux+rtt的AMP打包ITS配置文件
firefly-rk3568-amp.mk                   # linux+hal对应的AMP配置文件，也是AMP基础配置文件
firefly-rk3568-linux-rtt.mk             # linux+rtt对应的AMP配置文件
aio-3568j-amp-ubuntu.mk                 # AIO-3568J对应板级的配置脚本文件
```

统一编译脚本工具支持一键编译及打包U-Boot、Kernel、HAL、RT-Thread、ROOTFS等，并生成对应的
Image镜像。

```bash
./build.sh
```

#### 部分编译

* 编译 u-boot

```bash
./build.sh uboot
```

* 编译 kernel

注意：kernel 设备树位于kernel/arch/arm64/boot/dts/rockchip/目录下，它使用的设备树rk3568-firefly-aioj-amp.dts只适用于2G内存规格。
如果你需要适用于其他内存规格设备，需自行更改它的内存描述标签memory。

**AMP使用到的硬件资源需在内核设备树关闭，并在rk3568-amp.dtsi设备树的rockchip_amp标签下声明**

```bash
./build.sh kernel
```
生成的镜像为 kernel/boot.img


* 编译 recovery

```bash
./build.sh recovery
```

##### 编译AMP

编译hal或rt-thread制作amp镜像

###### 使用hal

* 编译hal
```bash
./build.sh hal 3
```
生成的镜像为 hal/project/rk3568/GCC/hal3.bin
会被链接到 rockdev/amp/cpu3.bin

###### 使用rt-thread

* 编译rt-thread

```bash
./build.sh rtthread 3
```
生成的镜像为 rt-thread/bsp/rockchip/rk3568-32/Image/rtt3.bin
会被链接到 rockdev/amp/cpu3.bin

###### 打包amp固件

* 打包amp镜像
```bash
./build.sh ampimg
```
使用rockdev/amp/amp.its配置文件打包出rockdev/amp/amp.img

###### 其他

hal或rt-thread默认使用uart4作为调试串口，波特率为115200

* 更加详细说明请阅读SDK/docs/amp/manuals/rk3568下的amp相关文档

#### 打包固件

* 更新链接

更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

* 打包固件

打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```

### 分区说明

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 parameter-amp-ubuntu-fit.txt 为例：

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

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

#### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

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
## 编译 Buildroot 固件

本章介绍 Buildroot 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

### 编译 SDK

#### 编译前配置

不同板型的配置文件存放在`device/rockchip/rk356x/`目录下

回到 SDK 根目录执行`build.sh`选择配置文件：

```bash
./build.sh aio-3568j-amp-buildroot.mk
```

#### SDK统一编译与打包

RK3568 支持 Linux 、HAL 、RT-Thread的 AMP 混合架构设计，使得不同的 CPU 可以运行不同的系统，以满足灵活
的产品设计需求。该 SDK 目前默认使用 Linux + RT-Thread 的混合结构模型，其中运行 Linux
的 CPU 为主核，运行 rtt 的 CPU 为从核；0~2核心运行Linux，3核心运行RT-Thread。

##### 编译配置

SDK 的统一编译配置脚本位于device/rockchip/rk356x/目录下，编译配置脚本内容包括U-Boot、Kernel、
HAL、RT-Thread的配置，以及AMP相关的CPU分配，内存分配等配置。用户可以根据需求增加或者修改
配置脚本文件，以满足自己的编译需求。主要的AMP配置文件如下:

```bash
rk3568_amp_linux.its                    # linux+hal的AMP打包ITS配置文件
rk3568_amp_linux_rtt.its                # linux+rtt的AMP打包ITS配置文件
firefly-rk3568-amp.mk                   # linux+hal对应的配置文件，也是AMP基础配置文件
firefly-rk3568-linux-rtt.mk             # linux+rtt对应的配置文件
aio-3568j-amp-buildroot.mk              # AIO-3568J对应的配置脚本文件
```

统一编译脚本工具支持一键编译及打包U-Boot、Kernel、HAL、RT-Thread、ROOTFS等，并生成对应的
Image镜像。

```bash
./build.sh
```

#### 部分编译

* 编译 u-boot

```bash
./build.sh uboot
```

* 编译 kernel

注意：kernel 设备树位于kernel/arch/arm64/boot/dts/rockchip/目录下，它使用的设备树rk3568-firefly-aioj-amp.dts只适用于2G内存规格。
如果你需要适用于其他内存规格设备，需自行更改它的内存描述标签memory。

**AMP使用到的硬件资源需在内核设备树关闭，并在rk3568-amp.dtsi设备树的rockchip_amp标签下声明**

```bash
./build.sh kernel
```

* 编译 recovery

```bash
./build.sh recovery
```

* 编译 Buildroot 根文件系统

编译 Buildroot 根文件系统，将会在 `buildroot/output` 生成编译输出目录：

```bash
./build.sh buildroot

# 注：确保作为普通用户编译 Buildroot 根文件系统，避免不必要的错误。
```

##### 编译AMP

编译hal或rt-thread制作amp镜像

###### 使用hal

* 编译hal
```bash
./build.sh hal 3
```
生成的镜像为 hal/project/rk3568/GCC/hal3.bin
会被链接到 rockdev/amp/cpu3.bin

###### 使用rt-thread

* 编译rt-thread

```bash
./build.sh rtthread 3
```
生成的镜像为 rt-thread/bsp/rockchip/rk3568-32/Image/rtt3.bin
会被链接到 rockdev/amp/cpu3.bin

###### 打包amp固件

* 打包amp镜像

```bash
./build.sh ampimg
```

使用rockdev/amp/amp.its配置文件打包出rockdev/amp/amp.img

###### 其他

hal或rt-thread默认使用uart4作为调试串口，波特率为115200

* 更加详细说明请阅读SDK/docs/amp/manuals/rk3568下的amp相关文档

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

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 `parameter-amp-buildroot-fit.txt` 为例：

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

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

#### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

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