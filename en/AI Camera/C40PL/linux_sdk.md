## Linux SDK Configuration Introduction

### Directory

```bash
$ tree -L 1
.
├── app
├── buildroot # Buildroot root filesystem build directory
├── build.sh -> device/rockchip/common/build.sh # Compile script
├── device # Compile related configuration files
├── docs # Documentation
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel
├── Makefile -> buildroot/build/Makefile
├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh # Link script
├── prebuilts # Cross compilation toolchain
├── rkbin
├── rkflash.sh -> device/rockchip/common/rkflash.sh # Flash script
├── tools # Tools directory
├── u-boot
```

### Introduction to configuration files

In the `device/rockchip/rv1126_rv1109/` directory, there are configuration files (xxxx.mk) for different board types, which are used to manage the compilation configuration of each project of the SDK. The relevant configuration introduction:

```bash
# Target arch
export RK_ARCH=arm # 32-bit ARM architecture
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=xxxx_defconfig # u-boot configuration file
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=xxxx_defconfig # kernel configuration file
# Kernel defconfig fragment
export RK_KERNEL_DEFCONFIG_FRAGMENT=xxxx.config # kernel configuration file (fragment)
# Kernel dts
export RK_KERNEL_DTS=xxxx.dts # dts file
# parameter for GPT table
export RK_PARAMETER=parameter-xxxx.txt # partition table
# rootfs image path
export RK_ROOTFS_IMG=ubuntu_rootfs/rootfs.img # root file system path
```

### Partition table

#### parameter

The parameter.txt file contains the partition information of the firmware. Take parameter-ubuntu-fit.txt as an example:

path: `device/rockchip/rv1126_rv1109/parameter-xxxxxx-fit.txt`

```bash
FIRMWARE_VER: 8.1
MACHINE_MODEL: RV1126
MACHINE_ID: 007
MANUFACTURER: RV1126
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00010000@0x00008000(boot),0x00010000@0x00018000(recovery),0x00010000@0x00028000(backup),0x00300000@0x00038000(rootfs),0x00060000@0x00338000(oem),0x00400000@0x00398000(userdata),-@0x00798000(media:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is where we are concerned. Take uboot as an example. In 0x00002000@0x00004000(uboot), 0x00004000 is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

#### package-file

The package-file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt.

path: `tools/linux/Linux_Pack_Firmware/rockdev/rv1126_rv1109-package-file`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
#
# if uboot.img is fit, uboot.img had include uboot and trust,
# so ignore trust.img
# file Image/uboot.img
# Image/uboot.img: Device Tree Blob version 17
#
# trust         Image/trust.img
#
uboot           Image/uboot.img
misc            Image/misc.img
#resource       Image/resource.img
#kernel         Image/kernel.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
oem             Image/oem.img
userdata        Image/userdata.img
# 要写入backup分区的文件就是自身（update.img）
# SELF 是关键字，表示升级文件（update.img）自身
# 在生成升级文件时，不加入SELF文件的内容，但在头部信息中有记录
# 在解包升级文件时，不解包SELF文件的内容。
backup          RESERVED
#update-script  update-script
#recover-script recover-script
```