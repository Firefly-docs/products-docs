# 编译 Buildroot 固件

本章介绍 Buildroot 固件的编译流程，推荐在 Ubuntu 16.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

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
### 目录结构

```bash
$ tree -L 1
.
├── app
├── buildroot                                               # Buildroot 根文件系统编译目录
├── build.sh -> device/rockchip/common/build.sh             # 编译脚本
├── debian                                                  # Debian 根文件系统编译目录
├── device                                                  # 编译相关配置文件
├── distro
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

### 搭建编译环境

```bash
sudo apt-get install expect-dev repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler \
gcc-aarch64-linux-gnu mtools parted libudev-dev libusb-1.0-0-dev python-linaro-image-tools \
linaro-image-tools autoconf autotools-dev libsigsegv2 m4 intltool libdrm-dev curl sed make \
binutils build-essential gcc g++ bash patch gzip bzip2 perl tar cpio python unzip rsync file bc wget \
libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev libglade2-dev cvs git mercurial rsync openssh-client \
subversion asciidoc w3m dblatex graphviz python-matplotlib libc6:i386 libssl-dev texinfo \
liblz4-tool genext2fs lib32stdc++6 expect
```

## 编译 SDK

### 编译前配置

在 `device/rockchip/rk3399/` 目录下，有不同板型的配置文件，选择配置文件：

```bash
./build.sh aio-3399c-buildroot.mk
```

配置文件会链接到 `device/rockchip/.BoardConfig.mk`，检查该文件可以验证是否配置成功。

相关配置介绍：

```bash
# Target arch
export RK_ARCH=arm64                                             # 64位 ARM 架构
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly-rk3399                        # u-boot 配置文件
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly_linux_defconfig              # kernel 配置文件
# Kernel dts
export RK_KERNEL_DTS=rk3399-firefly-aioc                              # dts 文件
# Buildroot config
export RK_CFG_BUILDROOT=rockchip_rk3399                        # Buildroot 配置
# Recovery config
export RK_CFG_RECOVERY=rockchip_rk3399_recovery                # recovery 配置
# parameter for GPT table
export RK_PARAMETER=parameter-buildroot.txt                     # 分区表
# rootfs image path
export RK_ROOTFS_IMG=buildroot/output/$RK_CFG_BUILDROOT/images/rootfs.$RK_ROOTFS_TYPE   # 根文件系统路径
```

### 部分编译

* 编译 u-boot

```bash
./build.sh uboot
```

* 编译 kernel

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

### 打包固件

更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```

### 全自动编译

全自动编译会执行上述编译、打包操作，生成完整固件。

```bash
./build.sh
```

## 分区说明

### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 `parameter.txt` 为例：

路径：`device/rockchip/rk3288/parameter.txt`

```bash
FIRMWARE_VER: 8.1
MACHINE_MODEL: RK3399
MACHINE_ID: 007
MANUFACTURER: RK3399
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3399
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(trust),0x00002000@0x00008000(misc),0x00010000@0x0000a000(boot),0x00010000@0x0001a000(recovery),0x00010000@0x0002a000(backup),0x00020000@0x0003a000(oem),0x00c00000@0x0005a000(rootfs),-@0x00c5a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

路径：`tools/linux/Linux_Pack_Firmware/rockdev/rk3399-package-file`

```bash
# NAME		Relative path
#
#HWDEF		HWDEF
package-file	package-file
bootloader	Image/MiniLoaderAll.bin
parameter	Image/parameter.txt
trust		Image/trust.img
uboot		Image/uboot.img
misc		Image/misc.img
#resource	Image/resource.img
#kernel		Image/kernel.img
boot		Image/boot.img
recovery	Image/recovery.img
rootfs		Image/rootfs.img
oem		Image/oem.img
userdata:grow	Image/userdata.img
backup		RESERVED
#update-script	update-script
#recover-script	recover-script
```