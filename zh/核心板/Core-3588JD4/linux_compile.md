# 编译 Linux 固件 (内核版本 5.10)

## 获取 SDK

首先准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以`~/proj`为例

<font color=red>

**注意：**

**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**

**2. 编译环境请使用 Ubuntu18.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**

**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**

**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**
</font>

### 安装工具
获取 SDK 需要先安装：
```
sudo apt update
sudo apt install -y repo git python
```
### 初始化仓库

* 方法一（推荐国内用户使用）

**SDK 源码存放于 gitlab，国内用户可能下载完整的 SDK 仓库速度比较慢，所以我们提供了一个 SDK 基础包([Linux SDK](https://www.t-firefly.com/doc/download/290.html))，国内用户只需要在此基础包上同步 gitlab 上的代码就可以了**

下载 SDK 基础包并且按照 README 文档操作：

```bash
└── rk3588_linux_release_xxx
    ├── linux_sdk_tar
    │   ├── rk3588_linux_release_xxx.sdk.split00
    │   ├── rk3588_linux_release_xxx.sdk.split01
    │   ├── rk3588_linux_release_xxx.sdk.split02
    │   ├── rk3588_linux_release_xxx.sdk.split03
    ├── md5sum.txt
    ├── README_EN.txt
    ├── README_ZH.txt
    └── sdk_tools.sh
```

* 方法二

通过 repo 拉取代码，此方法对网络要求较高，有条件可以使用

可选择获取完整 SDK 或者 BSP：

```bash
mkdir ~/proj/rk3588_sdk/
cd ~/proj/rk3588_sdk/

## 完整 SDK
repo init --no-clone-bundle --repo-url https://github.com/Firefly-rk-linux-utils/git-repo.git --no-repo-verify -u https://github.com/Firefly-rk-linux/manifests.git -b master -m rk3588_linux_release.xml

## BSP （ 只包含基础仓库和编译工具只适合编译 ubuntu、debian 固件）
## BSP 包括 device/rockchip 、docs 、 kernel 、 u-boot 、 rkbin 、 tools 和交叉编译链
repo init --no-clone-bundle --repo-url https://github.com/Firefly-rk-linux-utils/git-repo.git --no-repo-verify -u https://github.com/Firefly-rk-linux/manifests.git -b master -m rk3588_linux_bsp_release.xml
```

### 同步代码

执行如下命令同步代码：

```bash
# 进入 SDK 根目录
cd ~/proj/rk3588_sdk

# 同步
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

后续可以使用以下命令更新 SDK：

```bash
.repo/repo/repo sync -c --no-tags
```

因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行。

## Linux SDK 配置介绍

### 目录介绍

```bash
$ tree -L 1
.
├── app
├── buildroot                                               # Buildroot 根文件系统编译目录
├── build.sh -> device/rockchip/common/build.sh             # 编译脚本
├── device                                                  # 编译相关配置文件
├── docs                                                    # 文档
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel
├── Makefile -> buildroot/build/Makefile
├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh   # 链接脚本
├── prebuilts                                               # 交叉编译工具链
├── rkbin
├── rkflash.sh -> device/rockchip/common/rkflash.sh         # 烧写脚本
├── tools                                                   # 工具目录
├── u-boot
```

### 配置文件介绍

在 `device/rockchip/rk3588/` 目录下，有不同板型的配置文件(xxxx.mk)，用于管理 SDK 每个环节的编译配置，相关配置介绍：

```bash
# Target arch
export RK_ARCH=arm64 # 64位 ARM 架构
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=xxxx_defconfig # u-boot 配置文件
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=xxxx_defconfig # kernel 配置文件
# Kernel defconfig fragment
export RK_KERNEL_DEFCONFIG_FRAGMENT=xxxx.config # kernel 配置文件（fragment）
# Kernel dts
export RK_KERNEL_DTS=xxxx.dts # dts 文件
# parameter for GPT table
export RK_PARAMETER=parameter-xxxx.txt # 分区表
# rootfs image path
export RK_ROOTFS_IMG=ubuntu_rootfs/rootfs.img # 根文件系统路径
```

### 分区说明

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 parameter-ubuntu-fit.txt 为例：

路径：`device/rockchip/rk3588/parameter-xxxxxx-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3588
MACHINE_ID: 007
MANUFACTURER: RK3588
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00020000@0x00008000(boot:bootable),0x00040000@0x00028000(recovery),0x00010000@0x00068000(backup),0x00c00000@0x00078000(rootfs),0x00040000@0x00c78000(oem),-@0x00cb8000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

#### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

路径：`tools/linux/Linux_Pack_Firmware/rockdev/rk3588-package-file`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
uboot           Image/uboot.img
misc            Image/misc.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
userdata        RESERVED
backup          RESERVED
```
## 编译 Ubuntu 固件

本章介绍 Ubuntu 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

<font color=red>**本教程的编译部分适用于 v1.0.6e 以上 SDK 版本**</font>

```
$ readlink -f .repo/manifest.xml
/home/daijh/p/rk3588/.repo/manifests/rk3588/rk3588_linux_release_20230301_v1.0.6e.xml
```
### 准备工作

#### 搭建编译环境

```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip \
device-tree-compiler ncurses-dev \
```

### 编译 SDK

#### 编译前配置

在 `device/rockchip/rk3588/` 目录下，有不同板型的配置文件，选择配置文件：

```bash
./build.sh aio-3588jd4-ubuntu.mk 
```

#### 编译

##### 全自动编译

* 下载根文件系统：[Ubuntu 根文件系统(64位)](https://www.t-firefly.com/doc/download/161.html)，放到 SDK 路径下

```bash
7z x ubuntu-aarch64-rootfs.7z
mkdir ubuntu_rootfs
mv ubuntu-aarch64-rootfs.img ubuntu_rootfs/rootfs.img
```

* 开始编译
```bash
./build.sh
```

生成的完整固件会保存到 `rockdev/pack/` 目录。

##### 部分编译

* 编译 u-boot

```bash
./build.sh uboot
```

* 编译 kernel

```bash
./build.sh extboot
```

* 编译 recovery

```bash
./build.sh recovery
```

* 下载根文件系统：[Ubuntu 根文件系统(64位)](https://www.t-firefly.com/doc/download/161.html)，放到 SDK 路径下

```bash
7z x ubuntu-aarch64-rootfs.7z
mkdir ubuntu_rootfs
mv ubuntu-aarch64-rootfs.img ubuntu_rootfs/rootfs.img
```

* 更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

* 打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```

## 编译 Debian 固件

本章介绍 Debian 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

<font color=red>**本教程的编译部分适用于 v1.0.6e 以上 SDK 版本**</font>

```
$ readlink -f .repo/manifest.xml
/home/daijh/p/rk3588/.repo/manifests/rk3588/rk3588_linux_release_20230301_v1.0.6e.xml
```
### 准备工作

#### 搭建编译环境

```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip \
device-tree-compiler ncurses-dev \
```

### 编译 SDK

#### 编译前配置

在 `device/rockchip/rk3588/` 目录下，有不同板型的配置文件，选择配置文件：

```bash
./build.sh aio-3588jd4-debian.mk
```

#### 编译

##### 全自动编译

* 下载根文件系统：[Debian 根文件系统(64位)](https://www.t-firefly.com/doc/download/161.html)，放到 SDK 路径下

```bash
7z x debian_rk3588_rootfs_xxx.7z
mkdir debian
mv debianxx-rootfs.img debian/debian-rootfs.img
```

* 开始编译
```bash
./build.sh
```

生成的完整固件会保存到 `rockdev/pack/` 目录。

##### 部分编译

* 编译 u-boot

```bash
./build.sh uboot
```

* 编译 kernel

```bash
./build.sh extboot
```

* 编译 recovery

```bash
./build.sh recovery
```

* 下载根文件系统：[Debian 根文件系统(64位)](https://www.t-firefly.com/doc/download/161.html)，放到 SDK 路径下

```bash
7z x debian_rk3588_rootfs_xxx.7z
mkdir debian
mv debianxx-rootfs.img debian/rootfs.img
```

* 更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

* 打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```
## 编译 Buildroot 固件

本章介绍 Buildroot 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

<font color=red>**本教程的编译部分适用于 v1.0.6e 以上 SDK 版本**</font>

```
$ readlink -f .repo/manifest.xml
/home/daijh/p/rk3588/.repo/manifests/rk3588/rk3588_linux_release_20230301_v1.0.6e.xml
```
### 准备工作

#### 搭建编译环境

```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip \
device-tree-compiler ncurses-dev \
```

### 编译 SDK

#### 编译前配置

在 `device/rockchip/rk3588/` 目录下，有不同板型的配置文件，选择配置文件：

```bash
./build.sh aio-3588jd4-buildroot.mk
```

#### 编译

##### 全自动编译

全自动编译会执行上述编译、打包操作，生成 RK 固件。

```bash
./build.sh
```

打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

##### 部分编译

* 编译 u-boot

```bash
./build.sh uboot
```

* 编译 kernel

```bash
./build.sh extboot
```

* 编译 recovery

```bash
./build.sh recovery
```

* 编译 buildroot

```bash
./build.sh rootfs
```

* 更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

* 打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```
