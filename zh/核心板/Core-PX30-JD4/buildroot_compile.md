# 编译 Buildroot 固件

本章介绍 Buildroot 固件的编译流程。

## 编译 SDK

### 编译前配置

在 `device/rockchip/px30/` 目录下，有不同板型的配置文件，选择配置文件：

```bash
./build.sh px30-lvds-buildroot.mk
```

配置文件会链接到 `device/rockchip/.BoardConfig.mk`，检查该文件可以验证是否配置成功。

相关配置介绍：

```bash
# Target arch
export RK_ARCH=arm64                                            # 64位 ARM 架构
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=evb-px30                              # u-boot 配置文件
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=px30_linux_defconfig                 # kernel 配置文件
# Kernel dts
export RK_KERNEL_DTS=px30-firefly-lvds                          # dts 文件
# Buildroot config
export RK_CFG_BUILDROOT=rockchip_px30_64                        # Buildroot 配置
# Recovery config
export RK_CFG_RECOVERY=rockchip_px30_recovery                   # recovery 配置
# parameter for GPT table
export RK_PARAMETER=parameter.txt                               # 分区表
# rootfs image path
export RK_ROOTFS_IMG=buildroot/output/$RK_CFG_BUILDROOT/images/rootfs.$RK_ROOTFS_TYPE     # 根文件系统路径
```

### 部分编译

* 编译 u-boot

```bash
./build.sh uboot
```

* 编译 kernel

```bash
./build.sh kernel
```

* 编译 recovery

```bash
./build.sh recovery
```

* 编译 Buildroot 根文件系统

编译 Buildroot 根文件系统，将会在 `buildroot/output` 生成编译输出目录：

```bash
./build.sh buildroot

# 注：确保作为普通用户编译 Buildroot 根文件系统，避免不必要的错误。
```

### 打包固件

更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```

### 全自动编译

全自动编译会执行上述编译、打包操作，生成完整固件。

```bash
./build.sh
```

## 分区说明

### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 parameter.txt 为例：

路径：`device/rockchip/px30/parameter.txt`

```bash
FIRMWARE_VER: 8.1
MACHINE_MODEL: PX30
MACHINE_ID: 007
MANUFACTURER: PX30
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: px30
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(trust),0x00002000@0x00008000(misc),0x00010000@0x0000a000(boot),0x00010000@0x0001a000(recovery),0x00010000@0x0002a000(backup),0x00020000@0x0003a000(oem),0x00c00000@0x0005a000(rootfs),-@0x00c5a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

路径：`tools/linux/Linux_Pack_Firmware/rockdev/px30-package-file`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
trust           Image/trust.img
uboot           Image/uboot.img
misc            Image/misc.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
oem             Image/oem.img
userdata:grow   Image/userdata.img
backup          RESERVED
```
