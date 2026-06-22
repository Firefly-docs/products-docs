## Linux SDK 配置介绍

### 目录介绍

```bash
$ tree -L 1
.
├── app
├── buildroot									# Buildroot 根文件系统编译目录
├── build.sh -> device/rockchip/common/scripts/build.sh				# 编译脚本
├── device									# 编译配置脚本目录
├── docs									# 开发文档
├── external
├── firefly-release.sh -> device/rockchip/common/scripts/firefly-release.sh
├── firefly-update.sh -> device/rockchip/common/scripts/firefly-update.sh
├── hal										# hal 固件开发目录
├── kernel									# kernel 开发目录
├── Makefile -> device/rockchip/common/Makefile
├── prebuilts									# 交叉编译工具链
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh			# 烧写脚本
├── rtos									# RT-Thread 开发目录
├── tools									# 开发工具
├── u-boot									# uboot 开发目录
└── yocto									# yocto 文件系统构建
```

### 配置文件介绍

在 `device/rockchip/rk3506/` 目录下，有不同板型的配置文件(xxxx_defconfig)，用于管理 SDK 每个环节的编译配置。firefly.config 作为基础的配置。其他版型的配置文件会引用 firefly.config 并且覆盖其中的一些变量配置。 firefly.config 相关配置介绍：

```bash
RK_UBOOT_CFG="rk3506"				# uboot 编译配置
RK_UBOOT_CFG_FRAGMENTS="rk3506b firefly-linux"	# uboot 编译追加配置
RK_UBOOT_SPL=y
RK_KERNEL_ARM32=y				# kernel 使用 ARM32 架构编译
RK_KERNEL_CFG_FRAGMENTS="firefly-linux.config"	# kernel 编译追加配置
RK_KERNEL_CFG="rk3506_defconfig"		# kernel 编译配置
RK_KERNEL_DTS_NAME="rk3506b-firefly-roc-rk3506b-cc"	# 板型 dts 名称
RK_BOOT_COMPRESSED=y					# kernel 使用 boot 压缩方式（编译出来内核固件名称为 zboot.img）
RK_RECOVERY_FIT_ITS_NAME="zboot4recovery.its"	# recovery 固件编译打包使用的 its 配置
RK_FLASH_SIZE=2048
RK_EXTRA_PARTITION_1_FSTYPE="ubi"		# partition 1 使用 ubi 作为文件系统格式 (ubi 格式只适用于 spi flash 存储)
RK_EXTRA_PARTITION_1_SRC="rk3506_oem"		# partition 1 分区的源文件目录
RK_EXTRA_PARTITION_2_FSTYPE="ubi"		# partition 2 使用 ubi 作为文件系统格式 (ubi 格式只适用于 spi flash 存储)
RK_PARAMETER="parameter-buildroot-fit.txt"	# 打包使用的分区表
```

### 分区说明

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 parameter-buildroot-emmc-fit.txt 为例：

路径：`device/rockchip/rk3506/parameter-buildroot-emmc-fit.txt`

```bash
FIRMWARE_VER:8.1
MACHINE_MODEL:RK3506
MACHINE_ID:007
MANUFACTURER: RK3506
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3506
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
GROW_ALIGN: 0
CMDLINE:mtdparts=:0x00002000@0x00002000(uboot),0x00000800@0x00004000(misc),0x0000A000@0x00004800(recovery),0x00010000@0x0000E800(boot),0x00010000@0x0001E800(backup),0x00008000@0x0002E800(oem),0x00c00000@0x00036800(rootfs),-@0x00c36800(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
uuid:boot=7A3F0000-0000-446A-8000-702F00006273
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00002000(uboot) 中 0x00002000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

#### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

路径：`/path/to/sdk/output/firmware/package-file`

```bash
# NAME  PATH
package-file    package-file
parameter       parameter.txt
bootloader      MiniLoaderAll.bin
uboot   uboot.img
misc    misc.img
recovery        recovery.img
boot    boot.img
backup  RESERVED
oem     oem.img
rootfs  rootfs.img
userdata        userdata.img
```