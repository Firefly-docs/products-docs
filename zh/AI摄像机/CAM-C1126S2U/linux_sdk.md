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

在 `device/rockchip/rv1126_rv1109/` 目录下，有不同板型的配置文件(xxxx.mk)，用于管理 SDK 每个环节的编译配置，相关配置介绍：

```bash
# Target arch
export RK_ARCH=arm # 32 位 ARM 架构
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

路径：`device/rockchip/rv1126_rv1109/parameter-xxxxxx-fit.txt`

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

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

#### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

路径：`tools/linux/Linux_Pack_Firmware/rockdev/rv1126_rv1109-package-file`

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