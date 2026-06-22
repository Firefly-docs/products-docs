## Linux SDK Configuration Introduction

### Directory

```bash
$ tree -L 1
.
├── app
├── buildroot	# Buildroot root file system compilation directory
├── build.sh -> device/rockchip/common/scripts/build.sh # Compilation script
├── Copyright_Statement.md -> docs/licenses/LICENSE
├── device		# Compilation-related configuration files
├── docs		# Documentation
├── external
├── firefly-release.sh -> device/rockchip/common/scripts/firefly-release.sh
├── firefly-update.sh -> device/rockchip/common/scripts/firefly-update.sh
├── hal
├── kernel -> kernel-6.1
├── kernel-6.1
├── Makefile -> device/rockchip/common/Makefile
├── prebuilts	# Cross-compilation toolchain
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
├── rtos
├── tools		# Tools directory
├── u-boot
└── yocto		# Compile yocto file system directory
```

### Introduction to configuration files

In the `device/rockchip/rv1126b/` directory, there are configuration files (xxxx.mk) for different board types, which are used to manage the compilation configuration of each project of the SDK. The relevant configuration introduction:

Configuration file naming conventions:

```
<vendor>_<chip>_<model>-<extra>_<OS>_defconfig

vendor: Product supplier
chip: Chip used in the product
model: Product model, indicating that this configuration file is for this product model
extra: Additional attributes, can be empty
OS: Operating system the product will run
```

Configuration file content description, for example firefly_rv1126b_aio-1126bjd4v0_debian_defconfig:

```bash
#include "firefly.config"
RK_UBOOT_INI="RV1126BMINIALL.ini"
RK_UBOOT_SPL=y
RK_KERNEL_DTS_NAME="rv1126b-firefly-aio-1126bjd4v0"		# Specifies the device tree used for compiling the kernel.
RK_KERNEL_CFG="rockchip_linux_defconfig"
RK_MISC_RECOVERY=y
RK_RECOVERY_RAMDISK="common-recovery-arm64.cpio.gz"		# Specifies the pre-compiled recovery file system
RK_BOOT_FIT_ITS_NAME="bootramdisk.its"
RK_RAMDISK_IMG="kernel/ramdisk-64.img"
RK_PRODUCT_MODEL="AIO-1126BJD4V0"				# Product name/model
RK_PARAMETER="parameter-debian-fit.txt"				# Specifies the firmware partition table
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rv1126b_debian_rootfs.img" # Specifies the pre-compiled file system to use
```

### Partition table

#### parameter

The parameter.txt file contains the partition information of the firmware. Take parameter-ubuntu-fit.txt as an example:

path: `device/rockchip/rv1126b/parameter-xxxxxx-fit.txt`

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

The CMDLINE attribute is what we focus on. Taking uboot as an example, in 0x00002000@0x00004000(uboot), 0x00004000 is the starting position of the uboot partition, 0x00002000 is the partition size, and so on. The starting position of the partition + the partition size = the starting position of the next partition.

The unit of the numbers in the parameter is "blocks," and each block is 512 bytes. Therefore, the uboot partition size is 0x00002000, which is 8192 blocks, totaling 8192 x 512 / 1024 / 1024 = 4 MiB.