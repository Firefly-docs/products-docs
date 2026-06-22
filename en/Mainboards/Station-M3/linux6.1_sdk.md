## SDK Introduction

### SDK Structure

```bash
$ tree -L 1
.
├── app
├── buildroot                                               # buildroot
├── build.sh -> device/rockchip/common/scripts/build.sh     # building script
├── device                                                  # building system, includes building configs
├── docs                                                    # development documents
├── external                                                # some components
├── kernel -> kernel-6.1
├── kernel-6.1                                              # linux kernel
├── Makefile -> device/rockchip/common/Makefile
├── prebuilt_rootfs                                         # used to place pre-built rootfs
├── prebuilts                                               # used to place cross-build toolchain
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
├── tools                                                   # some tools
└── u-boot                                                  # uboot
```

### About Configurations

There are many config files (xxxx_defconfig) under `device/rockchip/rk3588/`, these files control the building system.

And their names will be like:
```
<vendor>_<chip>_<model>-<extra>_<OS>_defconfig

vendor: vendor of the product
chip: soc the product used
model: product model, means this config file is for that specific product.
extra: some extra features, could be empty
OS: the operating system that will be running in product
```

About config file contents, take firefly_rk3588_itx-3588j_debian_defconfig as an example:
```bash
RK_KERNEL_DTS_NAME="rk3588-firefly-itx-3588j"                    # set the dts to be used during kernel compilation
RK_KERNEL_CFG_FRAGMENTS="firefly-linux.config"                   # set the config fragments to be used during kernel compilation
RK_UBOOT_CFG_FRAGMENTS="firefly-linux"                           # set the config fragments to be used during uboot compilation
RK_USE_FIT_IMG=y
RK_BOOT_FIT_ITS_NAME="bootramdisk.its"
RK_PARAMETER="parameter-debian-fit.txt"                          # set the partition parameter
RK_PRODUCT_MODEL="ITX-3588J"                                     # product name/model
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk3588_debian_rootfs.img"   # set the prebuilt rootfs to be used
RK_MISC_RECOVERY=y
RK_RECOVERY_RAMDISK="rk3588-recovery-arm64.cpio.gz"
RK_RAMDISK_IMG="kernel/ramdisk.img"
USE_EXTBOOT=y                                                    # use exlinux style boot.img
RK_EXTRA_PARTITION_NUM=1
```

### Partitions

Parameter file records the firmware partitions, for example `device/rockchip/rk3588/parameter-xxxxxx-fit.txt`

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
GROW_ALIGN: 0
CMDLINE: mtdparts=:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00040000@0x00048000(recovery),0x00010000@0x00088000(backup),0x01c00000@0x00098000(rootfs),-@0x01c98000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
uuid:boot=7A3F0000-0000-446A-8000-702F00006273
```

CMDLINE is what we need to focus on, in "0x00002000@0x00004000(uboot)", 0x00004000 is the start position of uboot, and 0x00002000 is the size of uboot partition. Start position + Partition size = Start position of the next partition. All partitions follow the same rule.

And the unit of these hex numbers is "block", each block is 512 Byte, which means uboot partition size 0x00002000 is 8192 block = 8192 x 512 / 1024 / 1024 = 4 MiB