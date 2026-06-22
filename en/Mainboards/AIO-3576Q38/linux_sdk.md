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
├── kernel                                                  # linux kernel
├── Makefile -> device/rockchip/common/Makefile
├── prebuilt_rootfs                                         # used to place pre-built rootfs
├── prebuilts                                               # used to place cross-build toolchain
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
├── tools                                                   # some tools
└── u-boot                                                  # uboot
```

### About Configurations

There are many config files (xxxx_defconfig) under `device/rockchip/rk3576/`, these files control the building system.

And their names will be like:
```
<vendor>_<chip>_<model>-<extra>_<OS>_defconfig

vendor: vendor of the product
chip: soc the product used
model: product model, means this config file is for that specific product.
extra: some extra features, could be empty
OS: the operating system that will be running in product
```

About config file contents, take firefly_rk3576_aio-3576jd4_debian_defconfig as an example:

```bash
#include "firefly.config"                                             # includes firefly.config
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk3576_debian_rootfs.img"        # set the prebuilt rootfs to be used
RK_PARAMETER="parameter-ubuntu-fit.txt"                               # set the partition parameter
RK_KERNEL_DTS_NAME="rk3576-firefly-aio-3576jd4"                       # set the dts to be used during kernel compilation
RK_PRODUCT_MODEL="AIO-3576JD4"                                        # product name/model
UBOOT_DEBUG_BAUDRATE="115200"                                         # set debug serial baudrate as 115200
```

And firefly.config mainly has:

```bash
RK_KERNEL_CFG_FRAGMENTS="firefly-linux.config firefly-docker.config"  # set the config fragments to be used during kernel compilation
RK_UBOOT_CFG_FRAGMENTS="firefly-linux"                                # set the config fragments to be used during uboot compilation
USE_EXTBOOT=y                                                         # use exlinux style boot.img
```

### Partitions

Parameter file records the firmware partitions, for example `device/rockchip/rk3576/parameter-xxxxxx-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK357
MACHINE_ID: 007
MANUFACTURER: RK3576
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
GROW_ALIGN: 0
CMDLINE: mtdparts=:0x00004000@0x00004000(uboot),0x00002000@0x00008000(misc),0x00040000@0x0000a000(boot:bootable),0x00040000@0x0004a000(recovery),0x00010000@0x0008a000(backup),0x00c00000@0x0009a000(rootfs),-@0x00c9a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
uuid:boot=7A3F0000-0000-446A-8000-702F000062736
```

CMDLINE is what we need to focus on, in "0x00040000@0x0000a000(boot:bootable)", 0x0000a000 is the start position of boot, and 0x00040000 is the size of boot partition. Start position + Partition size = Start position of the next partition. All partitions follow the same rule.

And the unit of these hex numbers is "block", each block is 512 Byte, which means boot partition size 0x00040000 is 262144 block = 262144 x 512 / 1024 / 1024 = 128 MiB