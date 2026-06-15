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

在 `device/rockchip/rk3576/` 目录下，有不同板型的配置文件(xxxx_defconfig)，用于管理 SDK 每个环节的编译配置。

配置文件命名规则：
```
<vendor>_<chip>_<model>-<extra>_<OS>_defconfig

vendor: 产品供应商
chip: 产品所用的芯片
model: 产品型号，表示这个配置文件是用于该型号的产品
extra: 额外属性，可以为空
OS: 产品将会运行的操作系统
```

配置文件内容介绍，比如 firefly_rk3576_aio-3576jd4_debian_defconfig :

```bash
#include "firefly.config"                                             # 引用 firefly.config
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk3576_debian_rootfs.img"        # 指定要使用的预编译文件系统
RK_PARAMETER="parameter-ubuntu-fit.txt"                               # 指定固件分区表
RK_KERNEL_DTS_NAME="rk3576-firefly-aio-3576jd4"                       # 指定编译内核所使用的设备树
RK_PRODUCT_MODEL="AIO-3576JD4"                                        # 产品名称/型号
UBOOT_DEBUG_BAUDRATE="115200"                                         # 设置 debug 串口波特率为 115200
```

firefly.config 中的主要内容为：

```bash
RK_KERNEL_CFG_FRAGMENTS="firefly-linux.config firefly-docker.config"  # 指定编译内核所使用的 config fragments
RK_UBOOT_CFG_FRAGMENTS="firefly-linux"                                # 指定编译 uboot 所使用的 config fragments
USE_EXTBOOT=y                                                         # 使用 exlinux 风格的 boot.img
```

### 分区说明

parameter 文件中包含了固件的分区信息，比如 `device/rockchip/rk3576/parameter-xxxxxx-fit.txt`

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

CMDLINE 属性是我们关注的地方，以 boot 为例， 0x00040000@0x0000a000(boot:bootable) 中 0x0000a000 为 boot 分区的起始位置，0x00040000 为分区的大小，以此类推。分区的起始位置 + 分区大小 = 下一个分区的起始位置。

parameter 中数字的单位是 “块”，每块 512 字节。所以 boot 分区大小为 0x00040000，即 262144 块，共 262144 x 512 / 1024 / 1024 = 128 MiB
