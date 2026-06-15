# 编译 Linux 固件

为了方便用户的使用与开发，官方提供了 Linux 开发的整套 SDK，本章详细的说明 SDK 的具体用法。

## 准备工作

### 搭建 SDK 编译环境

**以下文件请务必确认安装！**

这里使用Ubuntu18.04进行测试(推荐使用ubuntu18.04系统进行开发，或者使用 docker 部署 Ubuntu18.04 容器，否则无法安装好环境包)：
```
sudo apt-get update

sudo apt-get install repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler \
gcc-aarch64-linux-gnu mtools parted libudev-dev libusb-1.0-0-dev python-linaro-image-tools \
linaro-image-tools gcc-arm-linux-gnueabihf libssl-dev liblz4-tool genext2fs lib32stdc++6 \
gcc-aarch64-linux-gnu g+conf autotools-dev libsigsegv2 m4 intltool libdrm-dev curl sed make \
binutils build-essential gcc g++ bash patch gzip bzip2 perl tar cpio python unzip rsync file bc wget \
libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev libglade2-dev cvs git mercurial rsync openssh-client \
subversion asciidoc w3m dblatex graphviz python-matplotlib libssl-dev texinfo fakeroot \
libparse-yapp-perl default-jre patchutils swig chrpath diffstat gawk time expect-dev
```

**注意：** Ubuntu17.04 或者更高的系统还需要如下依赖包：

```
sudo apt-get install lib32gcc-7-dev g++-7 libstdc++-7-dev
```

### 下载 Firefly_Linux_SDK 分卷压缩包

**由于 Firefly_Linux_SDK 源码包比较大，部分用户电脑不支持4G以上文件或单个文件网络传输较慢, 所以我们采用分卷压缩的方法来打包SDK。用户可以通过如下方式获取 Firefly_Linux_SDK源码包：**[Firefly_Linux_SDK源码包]

下载完成后按照 README 文档操作：

```
└── rk3399_linux_release_xxx
    ├── linux_sdk_tar
    │   ├── rk3399_linux_release_xxx.sdk.split00
    │   ├── rk3399_linux_release_xxx.sdk.split01
    ├── md5sum.txt
    ├── README_EN.txt
    ├── README_ZH.txt
    └── sdk_tools.sh

```

### 更新 Firefly_Linux_SDK

后续可以使用以下命令更新 SDK
```
.repo/repo/repo sync -c --no-tags
```

[下载链接]: http://www.t-firefly.com/doc/download/page/id/54.html
[Firefly_Linux_SDK源码包]: http://www.t-firefly.com/doc/download/page/id/54.html#other_186
### Linux_SDK 目录介绍

目录：

```
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
└── yocto
```

## 编译 SDK
<a id="configuration"></a>
### 编译前配置

配置文件 `aio-3399c-ubuntu.mk`:

```
./build.sh aio-3399c-ubuntu.mk

#文件路径在 `device/rockchip/rk3399/aio-3399c-ubuntu.mk`
```

如果配置文件生效会连接到 `device/rockchip/.BoardConfig.mk` ,检查该文件可以验证是否配置成功

**注意**: `aio-3399c-ubuntu.mk` 为编译生成 Ubuntu 固件的配置文件。同时用户也可以通过参考该配置生成新的配置文件来适配自己所需要的固件。

```bash
# uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly-rk3399      # 编译 uboot 配置文件

# kernel defconfig
export LINUX_KERNEL_DEFCONFI=firefly_linux_defconfig   # 编译 kernel 配置文件

# kernel dts
export RK_KERNEL_DTS=rk3399-firefly-aioc                       # 编译 kernel 用到的 dts

# parameter for GPT table
export RK_PARAMETER=parameter-ubuntu.txt                 # 分区信息(十分重要)

# packagefile for make update image
export RK_PACKAGE_FILE=rk3399-ubuntu-package-file        # 打包配置文件

# rootfs image path
export RK_ROOTFS_IMG=xxxx/xxxx.img                       # 根文件系统镜像路径
```

**<font color=#ff0000 >注意，以下步骤十分重要！！</font>**

*  [前往下载页面 下载对应的Ubuntu 根文件系统镜像](http://www.t-firefly.com/doc/download/page/id/54.html)
*  因为RK3399属于64位处理器，所以在对应`Linux Rootfs/arm64`文件夹下选择需要的文件系统，本文用`rk3399_ubuntu18.04.img.7z`进行举例。
*  把得到的镜像放到 SDK 的指定目录:

```
#解压
7z x rk3399_ubuntu18.04.img.7z

#sdk根目录下
mkdir ubuntu_rootfs
mv rk3399_ubuntu18.04.img ubuntu_rootfs/

#修改aio-3399c-ubuntu.mk文件
vim device/rockchip/rk3399/aio-3399c-ubuntu.mk

#把RK_ROOTFS_IMG属性改成ubuntu文件系统镜像得路径(也就是rk3399_ubuntu18.04.img)
RK_ROOTFS_IMG=ubuntu_rootfs/rk3399_ubuntu18.04.img
```

**<font color=#ff0000 >注意：</font>** Ubuntu 根文件系统镜像存放路径不能错。

运行 `./mkfirmware.sh` 会自动更新 `rockdev/rootfs.img` 的链接。

### 选择编译配件

选择屏幕

选择摄像头

### 全自动编译

全自动编译会执行所有编译、打包操作，直接生成 RK 固件。

```
./build.sh
```

### 部分编译

#### 编译 kernel

```
./build.sh kernel
```

Linux SDK v2.5.1c 及之后版本采用了 extboot， 编译内核请执行`./build.sh extboot`

生成的文件为 SDK/kernel/extboot.img，取代之前的 boot.img

如何查看版本：

1. 版本格式为 vx.x.xx，例如 v2.5.1c
2. 固件文件名称中存在版本号(…_vx.x.xx_日期.img)
3. Buildroot 使用 `cat /etc/version` 获取版本(rk3399_linux_release_日期_vx.x.xx.xml)
4. Ubuntu 使用 `ffgo version` 获取版本(rk3399_linux_release_日期_vx.x.xx.xml)
5. SDK 中可以在 SDK 根目录通过命令查看：`ls -l .repo/manifests/rk3399_linux_release.xml`
6. 如果上述方法找不到格式为 vx.x.xx 的版本，说明是旧版本，不支持 extboot

**不要将 extboot.img 烧录进旧版本固件!**

除此之外，extboot ubuntu 还支持以安装包的形式更新内核，详情查看[Ubuntu 使用手册](https://wiki.t-firefly.com/zh_CN/Firefly-Linux-Guide/manual_ubuntu.html#linux-headers-he-linux-image)

#### 编译 u-boot

```
./build.sh uboot
```

#### 编译 recovery

recovery分区可省略，若有需要，编译recovery：

```
./build.sh recovery
```

#### 编译 rootfs

* buildroot:

```
./build.sh rootfs
```

运行 `./mkfirmware.sh` 会自动更新 `rockdev/rootfs.img` 的链接。

## 固件打包

### 同步更新各部分镜像

每次打包固件前先确保 `rockdev/` 目录下文件链接是否正确:

```
ls -l

├── boot.img -> ~/project/linux_sdk/kernel/boot.img
├── idbloader.img -> ~/project/linux_sdk/u-boot/idbloader.img
├── linaro-rootfs.img
├── MiniLoaderAll.bin -> ~/project/linux_sdk/u-boot/rk3399_loader_v1.14.115.bin
├── misc.img -> ~/project/linux_sdk/device/rockchip/rockimg/wipe_all-misc.img
├── oem.img
├── parameter.txt -> ~/project/linux_sdk/device/rockchip/RK3399/parameter-ubuntu.txt
├── recovery.img -> ~/project/linux_sdk/buildroot/output/rockchip_rk3399_recovery/images/recovery.img
├── rootfs.img -> ~/project/linux_sdk/ubuntu_rootfs/rk3399_ubuntu18.04.img
├── trust.img -> ~/project/linux_sdk/u-boot/trust.img
├── uboot.img -> ~/project/linux_sdk/u-boot/uboot.img
└── userdata.img
```

可以运行 `./mkfirmware.sh` 更新链接

```
./mkfirmware.sh
```

提示：若不是编译全部的分区镜像，在运行 `./mkfirmware` 时，会遇到如下类似情况：

```
error: /home/ljh/proj/linux-sdk/buildroot/output/rockchip_rk3399_recovery/images/recovery.img not found!
表示recovery分区没有编译出镜像，其他的情况类似，如oem.img、userdata.img
上文提到，这些属于可省略分区镜像，可以不用理会。
```

### 打包统一固件

**注意：** 打包前请确认 `tools/linux/Linux_Pack_Firmware/rockdev/package-file` 是否正确。打包会根据此文件进行分区打包。此文件链接会在 `./build.sh aio-3399c-ubuntu.mk` 命令时更新，如果配置不对请返回`编译前配置`一节重新配置一次。

打包生成的完整固件会保存到 `rockdev/pack/` 目录。

#### RK 固件

RK 固件，是以 Rockchip 专有格式打包的固件，使用 Rockchip 提供的工具可以烧写到 eMMC 或者 SD 卡中(**注**：若无特殊说明，WIKI 上提及的固件默认为 RK 固件)。

```bash
# 打包 RK 固件
./build.sh updateimg
```

#### RAW 固件

RAW 固件，是一种能以逐位复制的方式烧写到存储设备的固件，是存储设备的原始映像。不同于 RK 固件，目前仅支持通过 Etcher 工具烧写至 SD 卡启动。

[Etcher 官方下载链接](https://www.balena.io/etcher/)

```bash
# 打包 RAW 固件
./build.sh rawimg
```

## 分区介绍

### parameter

`parameter.txt` 包含了固件的分区信息十分重要,你可以在 `device/rockchip/rk3399` 目录下找到一些 `parameter.txt` 文件,下面以 `parameter.txt` 为例子做介绍:

```
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3399
MACHINE_ID: 007
MANUFACTURER: RK3399
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(trust),0x00002000@0x00008000(misc),0x00040000@0x0000a000(boot:bootable),0x00040000@0x0004a000(recovery),0x00010000@0x0008a000(backup),0x00c00000@0x0009a000(rootfs),0x00040000@0x00c9a000(oem),-@0x00d1a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

`CMDLINE` 属性是我们关注的地方。以 Uboot 为例 `0x00002000@0x00004000(uboot)` 中 `0x00004000` 为 Uboot 分区的起始位置 `0x00002000` 为分区的大小。后面的分区规则相同。用户可以根据自己需要增减或者修改分区信息，但是请最少保留 uboot, trust, boot, rootfs 分区，这是机器能正常启动的前提条件。`parameter-ubuntu.txt` 中使用的就是这样的最简分区方案。

分区介绍：

```
uboot 分区:     烧写 uboot 编译出来的 uboot.img。
trust 分区:     烧写 uboot 编译出来的 trust.img。
misc 分区:      烧写 misc.img。开机检测进入 recovery 模式。（可省略）
boot 分区:      烧写 kernel 编译出来的 boot.img 包含 kernel 和设备树信息。
recovery 分区:  烧写 recovery.img。（可省略）
backup 分区:    预留,暂时没有用。后续跟 android 一样作为 recovery 的 backup 使用。（可省略）
oem 分区:       给厂家使用,存放厂家的 app 或数据。只读。代替原来音箱的 data 分区。挂载在/oem 目录。（可省略）
rootfs 分区:    存放 buildroot 编出来的 rootfs.img,只读.
userdata 分区 : 存放 app 临时生成的文件或者是给最终用户使用。可读写,挂载在 /userdata 目录下。（可省略）
```

### package-file

此文件应当与 `parameter` 保持一致，用于固件打包。可以在 `tools/linux/Linux_Pack_Firmware/rockdev` 下找到相关文件。以 `rk3399-ubuntu-package-file` 为例介绍:

```
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
trust           Image/trust.img
uboot           Image/uboot.img
boot            Image/boot.img
rootfs:grow     Image/rootfs.img
backup          RESERVED
```

以上是 SDK 编译后生成的镜像文件。根据 `parameter.txt` 只打包自己用到的 img 文件。

## 常见问题

### 如何进入升级模式

操作方法见[《升级固件》](03-upgrade_firmware.md)


[Linux_SDK.7z]: http://www.t-firefly.com/doc/download/page/id/54.html