## Linux SDK 配置介绍

### 目录介绍

```bash
$ tree -L 1
.
├── app
├── buildroot	# Buildroot 根文件系统编译目录
├── build.sh -> device/rockchip/common/scripts/build.sh # 编译脚本
├── Copyright_Statement.md -> docs/licenses/LICENSE
├── device		# 编译相关配置文件
├── docs		# 文档
├── external
├── firefly-release.sh -> device/rockchip/common/scripts/firefly-release.sh
├── firefly-update.sh -> device/rockchip/common/scripts/firefly-update.sh
├── hal
├── kernel -> kernel-6.1
├── kernel-6.1
├── Makefile -> device/rockchip/common/Makefile
├── prebuilts	# 交叉编译工具链
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
├── rtos
├── tools		# 工具目录
├── u-boot
└── yocto		# 编译 yocto 文件系统目录
```

### 配置文件介绍

在 `device/rockchip/rv1126b/` 目录下，有不同板型的配置文件(xxxx_defconfig)，用于管理 SDK 每个环节的编译配置。

配置文件命名规则：
```
<vendor>_<chip>_<model>-<extra>_<OS>_defconfig

vendor: 产品供应商
chip: 产品所用的芯片
model: 产品型号，表示这个配置文件是用于该型号的产品
extra: 额外属性，可以为空
OS: 产品将会运行的操作系统
```

配置文件内容介绍，比如 firefly_rv1126b_aio-1126bjd4v0_debian_defconfig :
```bash
#include "firefly.config"
RK_UBOOT_INI="RV1126BMINIALL.ini"
RK_UBOOT_SPL=y
RK_KERNEL_DTS_NAME="rv1126b-firefly-aio-1126bjd4v0"		# 指定编译内核所使用的设备树
RK_KERNEL_CFG="rockchip_linux_defconfig"
RK_MISC_RECOVERY=y
RK_RECOVERY_RAMDISK="common-recovery-arm64.cpio.gz"		# 指定预编译好的 recovery 文件系统
RK_BOOT_FIT_ITS_NAME="bootramdisk.its"
RK_RAMDISK_IMG="kernel/ramdisk-64.img"
RK_PRODUCT_MODEL="AIO-1126BJD4V0"				# 产品名称/型号
RK_PARAMETER="parameter-debian-fit.txt"				# 指定固件分区表
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rv1126b_debian_rootfs.img" # 指定要使用的预编译文件系统
```

### 分区说明

#### parameter 分区表

parameter 文件中包含了固件的分区信息，比如 `device/rockchip/rv1126b/parameter-xxxxxx-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RV1126B
MACHINE_ID: 007
MANUFACTURER: RV1126B
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
GROW_ALIGN: 0
CMDLINE: mtdparts=:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot),0x00040000@0x00048000(recovery),0x00010000@0x00088000(backup),0x00a00000@0x00098000(rootfs),-@0x00a98000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
uuid:boot=7A3F0000-0000-446A-8000-702F00006273
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为 uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。分区的起始位置 + 分区大小 = 下一个分区的起始位置。

parameter 中数字的单位是 “块”，每块 512 字节。所以 uboot 分区大小为 0x00002000，即 8192 块，共 8192 x 512 / 1024 / 1024 = 4 MiB