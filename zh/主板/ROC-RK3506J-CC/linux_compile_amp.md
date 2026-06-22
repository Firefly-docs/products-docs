# 编译 Linux AMP 固件

## 概述

AMP（Asymmetric Multi-Processing）系统是一种非对称多核异构系统，即在同一芯片内，通过分组 CPU ，并在不同组的 CPU 内运行不同的系统，通过合理分配任务和资源，AMP 系统可以提供更好的性能表现。

rk3506 芯片有 Cortex-A7 * 3 + Cortex-M0 * 1 两类 CPU ，三个 Cortex-A7 核心支持 linux(kernel-6.1)、Baremetal(HAL)、RTOS(RT-Thread) , Cortex-M0 支持 Baremetal(HAL) 。
利用 AMP 特性，可在 Cortex-A7 * 3 运行 linux， Cortex-M0 * 1作为辅助核心跑裸机系统；也可在 Cortex-A7 * 2 运行linux， Cortex-A7 * 1作为辅助核心跑裸机或实时系统等多种组合 AMP 构建。

## 编译环境搭建

本章介绍 Linux SDK 的编译环境搭建

<font color=red>

**注意：**

**（1）推荐在 X86_64 Ubuntu 22.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。**

**（2）使用普通用户进行编译，不要使用 root 用户权限进行编译。**</font>

## 获取 SDK

首先准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以`~/proj`为例

<font color=red>

**注意：**

**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**

**2. 编译环境请使用 Ubuntu22.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**

**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**

**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**
</font>

### 安装工具
获取 SDK 需要先安装：
```
sudo apt update
sudo apt install -y repo git python p7zip-full
```
### 初始化仓库

**我们提供了一个 SDK 基础包。请联系业务 sales@t-firefly.com 获取。**

下载完成后先验证一下 MD5 码：

```bash
$ md5sum rk3506_linux_release_20250117_v1.0.0a/*
a5df458069569e9288b1d96097b43397  rk3506_linux_release_20250117_v1.0.0a.7z.001
7c1a1b050ae4f0daf64fe7488a7f4b02  rk3506_linux_release_20250117_v1.0.0a.7z.002
c7998713e7ef3512635ad2f79730d888  rk3506_linux_release_20250117_v1.0.0a.7z.003
```

确认无误后，就可以解压：
```bash
# 解压
mkdir -p ~/proj/rk3506_sdk
cd ~/proj/rk3506_sdk
7z x /path/to/rk3506_linux_release_20250117_v1.0.0a/rk3506_linux_release_20250117_v1.0.0a.7z.001
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
FROM ubuntu:22.04
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
# ubuntu22 是容器名，firefly 是容器 hostname，均可随意更改
# sdkcompiler 是上一步的镜像名
docker run --privileged --mount type=bind,source=/home/fierfly/proj,target=/home/firefly/proj --name="ubuntu22" -h firefly -it sdkcompiler
```
现在就可以在容器中进行 SDK 的编译了。

退出容器、重启容器的方法：
```bash
# 在容器内输入 exit 即可退出

# 查看所有容器（包括已退出的）
docker ps -a

# 重启一个退出的容器并连接
docker start ubuntu22 # 容器名
docker attach ubuntu22
```

## 编译 Buildroot 固件

本章介绍 Buildroot 固件的编译流程，推荐在 Ubuntu 22.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

### 编译 SDK

#### 编译前配置

不同板型的配置文件存放在 `/path/to/sdk/device/rockchip/rk3506/` 目录下

回到 SDK 根目录执行`build.sh`选择配置文件：

```bash
./build.sh firefly_rk3506_roc-rk3506b-cc-amp-emmc_buildroot_defconfig
```

#### SDK统一编译与打包

RK3506 支持 Linux 、HAL 、RT-Thread的 AMP 混合架构设计，使得不同的 CPU 可以运行不同的系统，以满足灵活的产品设计需求。该 SDK 目前默认使用 Linux + RT-Thread 的混合结构模型，其中运行 Linux 的 CPU 为主核，运行 rtt 的 CPU 为从核；0~1 核心运行 Linux ，2 核心运行 RT-Thread 。

##### 编译配置

SDK 的统一编译配置脚本位于 device/rockchip/rk3506/ 目录下，编译配置脚本内容包括 U-Boot、Kernel、HAL、RT-Thread 的配置，以及 AMP 相关的 CPU 分配，内存分配等配置。用户可以根据需求增加或者修改配置脚本文件，以满足自己的编译需求。主要的 AMP 配置文件如下:

```bash
amp_linux.its								# linux + rtt的 AMP 打包 ITS 配置文件
firefly_rk3506_roc-rk3506b-cc-amp-emmc_buildroot_defconfig		# linux + rtt 对应的配置文件
```

统一编译脚本工具支持一键编译及打包 U-Boot、Kernel、HAL、RT-Thread、ROOTFS等，并生成对应的 Image 镜像。

```bash
./build.sh
```

#### 部分编译

* 编译 u-boot

```bash
./build.sh uboot
```

* 编译 kernel

注意：kernel 设备树位于 kernel/arch/arm/boot/dts 目录下，它使用的设备树rk3506b-firefly-roc-rk3506b-cc-amp-emmc.dts只适用于2G内存规格。
如果你需要适用于其他内存规格设备，需自行更改它的内存描述标签memory。

**AMP 使用到的硬件资源需在内核设备树关闭，并在 rk3506-amp.dtsi 设备树的 rockchip_amp 标签下声明**

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

##### 编译 AMP

编译 rt-thread 制作 amp 镜像

###### 打包 amp 固件

* 打包amp镜像

```bash
./build.sh amp
```

使用 /path/to/sdk/device/rockchip/rk3506/amp_linux.its 配置文件打包出 output/firmware/amp.img

###### 其他

hal 或 rt-thread 默认使用 uart5 作为调试串口，波特率为 1.5M 。

#### 打包固件

打包固件，生成的完整固件会保存到 `output/update/` 目录。

```bash
./build.sh updateimg
```

### 分区说明

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 `parameter-buildroot-amp-emmc-fit.txt` 为例：

路径：`device/rockchip/rk3506/parameter-buildroot-amp-emmc-fit.txt`

```bash
FIRMWARE_VER:8.1
MACHINE_MODEL:RK3506
MACHINE_ID:007
MANUFACTURER: RK3506
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3506
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
GROW_ALIGN: 0
CMDLINE:mtdparts=:0x00002000@0x00002000(uboot),0x00000800@0x00004000(misc),0x0000A000@0x00004800(recovery),0x00010000@0x0000E800(boot),0x00010000@0x0001E800(backup),0x00008000@0x0002E800(oem),0x00002000@0x00036800(amp),0x00c00000@0x00038800(rootfs),-@0x00c36800(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
uuid:boot=7A3F0000-0000-446A-8000-702F00006273
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00002000(uboot) 中 0x00002000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

#### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

路径：`output/firmware/package-file`

```bash
# NAME  PATH
package-file    package-file
parameter       parameter.txt
bootloader      MiniLoaderAll.bin
uboot   uboot.img
misc    misc.img
recovery        recovery.img
boot    boot.img
backup  RESERVED
oem     oem.img
amp     amp.img
rootfs  rootfs.img
userdata        userdata.img
```