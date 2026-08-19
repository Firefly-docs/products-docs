# 编译 Ubuntu 固件

本章介绍 Ubuntu 固件的编译使用。

## 准备工作

如果编译过 Buildroot 固件，则下载源码可以省略，可在更新源代码后在 Buildroot 源码的基础上进行编译。

### 下载 Firefly_Linux_SDK

#### 方式一（国内用户）

* 从[资源下载](https://community.t-firefly.com/doc/download/73)页面下载 `REPO_SDK` 软件包。</br>
* 比较 REPO_SDK 软件包的 MD5 码校验完整性，然后解压。
```
md5sum rk1808_linux_release_20210306.tgz
be79c2e877970e4e0fc2bc0be44a8d02  rk1808_linux_release_20210306.tgz
tar xvf rk1808_linux_release_20210306.tgz
```
* 解压后进入文件夹，完成同步工作。
```
#本压缩包内包含一个.repo目录，解压之后，在当前目录下执行以下操作
.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all

#后续可以使用以下命令更新SDK
.repo/repo/repo sync -c --no-tags

#因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行
```

#### 方式二（国外用户）
```
repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk1808_linux_release.xml

repo sync -c
#需反复确认代码下载是否成功
```

### firefly-sdk 目录介绍

目录：
```
firefly-sdk/
├── app
├── buildroot                                              buildroot根文件系统的编译目录
├── build.sh -> device/rockchip/common/build.sh            全自动编译脚本
├── debian                                                 debian根文件系统生成目录
├── device                                                 编译相关配置文件
├── distro
├── docs                                                   文档
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel                                                 内核
├── Makefile -> buildroot/build/Makefile
├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh  rockdev链接更新脚本
├── prebuilts
├── rkbin
├── rkflash.sh -> device/rockchip/common/rkflash.sh        烧写脚本 
├── tools                                                  烧写、打包工具
├── u-boot                                                 uboot
└── yocto
```

### 搭建 SDK 编译环境

* 编译 Ubuntu 固件

```
sudo apt-get install repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler \
gcc-aarch64-linux-gnu mtools parted libudev-dev libusb-1.0-0-dev python-linaro-image-tools \
linaro-image-tools gcc-4.8-multilib-arm-linux-gnueabihf gcc-arm-linux-gnueabihf libssl-dev \
gcc-aarch64-linux-gnu g+conf autotools-dev libsigsegv2 m4 intltool libdrm-dev curl sed make \
binutils build-essential gcc g++ bash patch gzip bzip2 perl tar cpio python unzip rsync file bc wget \
libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev libglade2-dev cvs git mercurial rsync openssh-client \
subversion asciidoc w3m dblatex graphviz python-matplotlib libc6:i386 libssl-dev texinfo \
liblz4-tool genext2fs lib32stdc++6
```

**注意**: Ubuntu17.04 或者更高的系统还需要如下依赖包。
```
sudo apt-get install lib32gcc-7-dev g++-7 libstdc++-7-dev
```

## 编译 SDK

### 编译前配置

在 `device/rockchip/rK1808/` 目录下，选择对应的板型的配置文件。

配置文件 aio-rk1808-jd4-ubuntu.mk ：

```
./build.sh aio-rk1808-jd4-ubuntu.mk

#文件路径在 device/rockchip/rk1808/aio-rk1808-jd4-ubuntu.mk
```
配置文件生效会连接到 `device/rockchip/.BoardConfig.mk` ，检查该文件可以验证是否配置成功。

**注意**： `aio-rk1808-jd4-ubuntu.mk` 为编译生成 ubuntu 固件的配置文件，同时用户也可以通过参考该配置生成新的配置文件来适配自己所需要的固件，配置文件使用 `source` 命令包含其它文件的方式，一些配置选项在其包含的文件中。

重要配置介绍：

```
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly_rk1808                          编译uboot配置文件

# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly_rk1808_ubuntu_defconfig        编译kernel配置文件

# Kernel dts
export RK_KERNEL_DTS=rk1808-firefly-aiojd4                        编译kernel用到的dts

# parameter for GPT table
export RK_PARAMETER=parameter-ubuntu.txt                          分区信息(十分重要)

# packagefile for make update image 
export RK_PACKAGE_FILE=rk1808-ubuntu-package-file                 打包配置文件

# rootfs image path
export RK_ROOTFS_IMG=ubuntu_rootfs/rk1808_ubuntu18.04_rootfs.img  根文件系统路径
```
**<font color=#ff0000 >注意,十分重要！！</font>**

*  [下载 Ubuntu 根文件系统镜像](https://community.t-firefly.com/doc/download/73)
*  把得到的镜像放到 firefly-sdk 的指定目录：

```
# 解压
7z x xxxxx_rootfs.img.7z

# 在 firefly-sdk 根目录下新建存放根文件系统的文件夹
mkdir ubuntu_rootfs
mv xxxxx_rootfs.img ubuntu_rootfs/

# 修改 aio-rk1808-jd4-ubuntu.mk 文件
vim device/rockchip/rk1808/aio-rk1808-jd4-ubuntu.mk.mk

# 添加或者修改根文件系统镜像 RK_ROOTFS_IMG 的路径
RK_ROOTFS_IMG=ubuntu_rootfs/xxxxx_rootfs.img
```
**<font color=#ff0000 >注意</font>**： Ubuntu 根文件系统镜像存放路径不能错。

### 全自动编译

在配置和搭建环境的工作都做好的前提下：

```
./build.sh aio-rk1808-jd4-ubuntu.mk
./build.sh
```
全自动编译将会在 `rockdev/pack` 目录下生成统一固件。

### 部分编译

#### 配置
```
./build.sh aio-rk1808-jd4-ubuntu.mk
```
#### kernel
```
./build.sh kernel
```
#### u-boot
```
./build.sh uboot
```
#### recovery

```
./build.sh recovery
```

#### rootfs

* Ubuntu18.04 根文件系统通过网盘下载。

1. [下载 Ubuntu 根文件系统镜像](https://community.t-firefly.com/doc/download/73)
2. 把得到的镜像放到 firefly-sdk 的指定目录：

```
# 解压
7z x xxxxx.img.7z

# 在 firefly-sdk 根目录下新建存放根文件系统的文件夹
mkdir ubuntu_rootfs
mv rootfs.img ubuntu_rootfs/

# 修改 aio-rk1808-jd4-ubuntu.mk 文件
vim device/rockchip/rk1808/aio-rk1808-jd4-ubuntu.mk

# 添加或者修改根文件系统镜像 RK_ROOTFS_IMG 的路径
RK_ROOTFS_IMG=ubuntu_rootfs/xxxxx_rootfs.img
```
**<font color=#ff0000 >注意</font>**：Ubuntu 根文件系统镜像存放路径不能错。

运行 `./mkfirmware.sh` 会自动更新 `rockdev/rootfs.img` 的链接。

## 固件打包

### 同步更新各部分镜像

每次打包固件前先确保 `rockdev/` 目录下文件链接是否正确：

```
rockdev/
├── boot.img -> ../kernel/boot.img
├── MiniLoaderAll.bin -> ../u-boot/rk1808_loader_v1.03.104.bin
├── misc.img -> ../device/rockchip/rockimg/wipe_all-misc.img
├── oem.img
├── pack
├── parameter.txt -> ../device/rockchip/rk1808/parameter-ubuntu.txt
├── recovery.img
├── rootfs.img -> ../ubuntu_rootfs/rk1808-ubuntu1804-arm64-rootfs.img
├── trust.img -> ../u-boot/trust.img
├── uboot.img -> ../u-boot/uboot.img
└── userdata.img
```

运行 `./mkfirmware.sh` 更新链接。

```
./mkfirmware.sh
```
### 打包统一固件

注意：打包前请确认 `tools/linux/Linux_Pack_Firmware/rockdev/package-file` 是否正确，打包会根据此文件进行分区打包。此文件链接会在 `./build.sh aio-rk1808-jd4-ubuntu.mk` 命令时更新，如果配置不对请返回配置这一节重新配置一次。

整合统一固件，每次打包前都需要运行 `./mkfirmware.sh` 更新链接。

```
./build.sh updateimg
```
输入该命令后将会提醒是否需要重命名，输入 `y` 可以将固件重新命名，输入 `n` 使用默认的固件名字，在 `rockdev/pack` 目录下生成该固件。

## 分区介绍

### parameter

parameter.txt 包含了固件的分区信息十分重要，你可以在 `device/rockchip/rk1808` 目录下找到一些 parameter.txt 文件，下面以 parameter-ubuntu.txt 为例子做介绍:
```
FIRMWARE_VER: 8.1
MACHINE_MODEL: RK1808
MACHINE_ID: 007
MANUFACTURER: RK1808
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 1808
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(trust),0x00002000@0x00008000(misc),0x00010000@0x0000a000(boot),0x00010000@0x0001a000(recovery),0x00010000@0x0002a000(backup),0x00600000@0x0005a000(rootfs),-@0x0065a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

`CMDLINE` 属性是我们关注的地方，以 uboot 为例 `0x00002000@0x00004000(uboot)` 中 `0x00004000 `为 uboot 分区的起始位置 `0x00002000` 为分区的大小，后面的分区规则相同。用户可以根据自己需要增减或者修改分区信息，但是请最少保留 uboot ，trust ，boot ，rootfs 分区，这是机器能正常启动的前提条件。

分区介绍:
```
uboot 分区: 烧写 uboot 编译出来的 uboot.img 。
trust 分区: 烧写 uboot 编译出来的 trust.img 。
misc 分区: 烧写 misc.img。开机检测进入recovery 模式。
boot 分区: 烧写 kernel 编译出来的 boot.img ，包含 kernel 和设备树信息。
recovery 分区: 烧写 recovery.img 。
backup 分区: 预留,暂时没有用。
rootfs 分区: 存放 rootfs.img 。
```

### package-file
此文件应当与 parameter.txt 文件保持一致，用于固件打包。可以在 `tools/linux/Linux_Pack_Firmware/rockdev` 下找到相关文件。以 rk1808-ubuntu-package-file 为例介绍：
```
# NAME      Relative path
#
#HWDEF      HWDEF
package-file    package-file
bootloader  Image/MiniLoaderAll.bin
parameter   Image/parameter.txt
trust       Image/trust.img
uboot       Image/uboot.img
misc        Image/misc.img
boot        Image/boot.img
recovery    Image/recovery.img
rootfs      Image/rootfs.img
userdata:grow RESERVED
backup      RESERVED
```
以上是SDK编译后生成的镜像文件，根据 `parameter.txt` 只打包自己用到的 img 文件。
