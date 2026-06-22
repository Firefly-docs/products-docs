## SDK 配置介绍

### 目录介绍

```bash
$ tree -L 1
.
├── app
├── buildroot                                               # buildroot
├── build.sh -> device/rockchip/common/scripts/build.sh     # 编译脚本
├── device                                                  # 编译系统，内含编译配置文件
├── docs                                                    # 开发文档
├── external                                                # 一些组件
├── kernel                                                  # 内核
├── Makefile -> device/rockchip/common/Makefile
├── prebuilt_rootfs                                         # 用于存放预编译好的文件系统
├── prebuilts                                               # 用于存放交叉编译工具链
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
├── tools                                                   # 工具，包含烧录工具等
└── u-boot                                                  # uboot
```

### 配置文件介绍

在 `device/rockchip/rk3562/` 目录下，有不同板型的配置文件(xxxx_defconfig)，用于管理 SDK 每个环节的编译配置。

配置文件命名规则：
```
<vendor>_<chip>_<model>-<extra>_<OS>_defconfig

vendor: 产品供应商
chip: 产品所用的芯片
model: 产品型号，表示这个配置文件是用于该型号的产品
extra: 额外属性，可以为空
OS: 产品将会运行的操作系统
```

配置文件内容介绍，比如 firefly_rk3562_aio-3562jq_debian_defconfig :

```bash
RK_WIFIBT_CHIP="AP6256"
RK_UBOOT_SPL=y
RK_KERNEL_DTS_NAME="rk3562-firefly-aio-3562jq"                   # 指定编译内核所使用的是设备树
RK_KERNEL_CFG_FRAGMENTS="firefly_linux.config"                   # 指定编译内核所使用的 config fragments
RK_UBOOT_CFG_FRAGMENTS="firefly-linux"                           # 指定编译 uboot 所使用的 config fragments
RK_PARAMETER="parameter-debian-fit.txt"                          # 指定固件分区表
RK_USE_FIT_IMG=y
RK_RAMDISK_IMG="kernel/ramdisk.img"
USE_EXTBOOT=y                                                    # 使用 exlinux 风格的 boot.img
RK_RECOVERY_RAMDISK="rk3562-recovery-arm64.cpio.gz"
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk3562_debian_rootfs.img"   # 指定要使用的预编译文件系统
RK_MISC_RECOVERY=y
RK_EXTRA_PARTITION_NUM=1
```

### 分区说明

parameter 文件中包含了固件的分区信息，比如 `device/rockchip/rk3562/parameter-xxxxxx-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3562
MACHINE_ID: 007
MANUFACTURER: RK3562
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
GROW_ALIGN: 0
CMDLINE: mtdparts=:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00040000@0x00048000(recovery),0x00010000@0x00088000(backup),0x00c00000@0x00098000(rootfs),-@0x00c98000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为 uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。分区的起始位置 + 分区大小 = 下一个分区的起始位置。

parameter 中数字的单位是 “块”，每块 512 字节。所以 uboot 分区大小为 0x00002000，即 8192 块，共 8192 x 512 / 1024 / 1024 = 4 MiB
