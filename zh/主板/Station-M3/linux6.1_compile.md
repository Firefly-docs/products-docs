# 编译 Linux 固件 (内核版本 6.1)

## 获取 SDK

请联系销售(sales@t-firefly.com)获取 SDK 下载链接, 并且阅读下载链接的 readme 文档。

<font color=red>

**注意：**

**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**

**2. 编译环境请使用 Ubuntu20.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**

**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**

**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**
</font>## SDK 配置介绍

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
├── kernel -> kernel-6.1
├── kernel-6.1                                              # 内核
├── Makefile -> device/rockchip/common/Makefile
├── prebuilt_rootfs                                         # 用于存放预编译好的文件系统
├── prebuilts                                               # 用于存放交叉编译工具链
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
├── tools                                                   # 工具，包含烧录工具等
└── u-boot                                                  # uboot
```

### 配置文件介绍

在 `device/rockchip/rk3588/` 目录下，有不同板型的配置文件(xxxx_defconfig)，用于管理 SDK 每个环节的编译配置。

配置文件命名规则：
```
<vendor>_<chip>_<model>-<extra>_<OS>_defconfig

vendor: 产品供应商
chip: 产品所用的芯片
model: 产品型号，表示这个配置文件是用于该型号的产品
extra: 额外属性，可以为空
OS: 产品将会运行的操作系统
```

配置文件内容介绍，比如 firefly_rk3588_itx-3588j_debian_defconfig :

```bash
RK_KERNEL_DTS_NAME="rk3588-firefly-itx-3588j"                    # 指定编译内核所使用的设备树
RK_KERNEL_CFG_FRAGMENTS="firefly-linux.config"                   # 指定编译内核所使用的 config fragments
RK_UBOOT_CFG_FRAGMENTS="firefly-linux"                           # 指定编译 uboot 所使用的 config fragments
RK_USE_FIT_IMG=y
RK_BOOT_FIT_ITS_NAME="bootramdisk.its"
RK_PARAMETER="parameter-debian-fit.txt"                          # 指定固件分区表
RK_PRODUCT_MODEL="ITX-3588J"                                     # 产品名称/型号
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk3588_debian_rootfs.img"   # 指定要使用的预编译文件系统
RK_MISC_RECOVERY=y
RK_RECOVERY_RAMDISK="rk3588-recovery-arm64.cpio.gz"
RK_RAMDISK_IMG="kernel/ramdisk.img"
USE_EXTBOOT=y                                                    # 使用 exlinux 风格的 boot.img
RK_EXTRA_PARTITION_NUM=1
```

### 分区说明

parameter 文件中包含了固件的分区信息，比如 `device/rockchip/rk3588/parameter-xxxxxx-fit.txt`

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

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为 uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。分区的起始位置 + 分区大小 = 下一个分区的起始位置。

parameter 中数字的单位是 “块”，每块 512 字节。所以 uboot 分区大小为 0x00002000，即 8192 块，共 8192 x 512 / 1024 / 1024 = 4 MiB




