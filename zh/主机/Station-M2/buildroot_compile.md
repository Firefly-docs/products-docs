## 编译 Buildroot 固件

本章介绍 Buildroot 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

### 编译 SDK

#### 编译前配置

不同板型的配置文件存放在`device/rockchip/rk356x/`目录下

回到 SDK 根目录执行`build.sh`选择配置文件：

```bash
./build.sh 
```

配置文件会链接到 `device/rockchip/.BoardConfig.mk`，检查该文件可以验证是否配置成功。

相关配置介绍：

```bash
# Target arch
export RK_ARCH=arm64 # 64位 ARM 架构
# Uboot defconfig
export RK_UBOOT_DEFCONFIG= # u-boot 配置文件
# Kernel defconfig
export RK_KERNEL_DEFCONFIG= # kernel 配置文件
# Kernel dts
export RK_KERNEL_DTS= # dts 文件
# Buildroot config
export RK_CFG_BUILDROOT= # Buildroot 配置
# Recovery config
export RK_CFG_RECOVERY=rockchip_rk356x_recovery # recovery 配置
# parameter for GPT table
export RK_PARAMETER=parameter-buildroot-fit.txt # 分区表
# rootfs image path
export RK_ROOTFS_IMG=rockdev/rootfs.${RK_ROOTFS_TYPE} # 根文件系统路径
```

#### 选择编译配件

在 `device/rockchip/rk356x/` 目录下，除了之外，还有其他不同配件搭配的配置文件

配置文件的名称中会标明所使用的屏幕和摄像头。如果没有标明屏幕，说明使用的是默认 HDMI 显示；如果没有标明摄像头，说明使用的是默认单目摄像头

```bash
# 例如：
xxxx-buildroot.mk              # 使用 HDMI + 单目摄像头
xxxx-2cam-buildroot.mk         # 使用 HDMI + 双目摄像头
xxxx-mipi-buildroot.mk         # 使用 mipi + 单目摄像头
xxxx-mipi-2cam-buildroot.mk    # 使用 mipi + 双目摄像头
```

选择好配置文件后，执行`build.sh`来使其生效：

```bash
# 例如：
./build.sh xxxx-mipi-2cam-buildroot.mk
```

#### 全自动编译

全自动编译会执行所有编译、打包操作，生成完整固件。

```bash
./build.sh
```

#### 部分编译

* 编译 u-boot

```bash
./build.sh uboot
```

* 编译 kernel

注意：Firefly kernel 没有开启全部的内核功能，有需求请查看[Kernel 使用](kernel_introduction.md)

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

#### 打包固件

更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```

### 分区说明

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 `parameter-buildroot-fit.txt` 为例：

路径：`device/rockchip/rk356x/parameter-buildroot-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3568
MACHINE_ID: 007
MANUFACTURER: RK3568
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00010000@0x00008000(boot),0x00010000@0x00018000(recovery),0x00010000@0x00028000(backup),0x00040000@0x00038000(oem),0x00c00000@0x00078000(rootfs),-@0x00c78000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

#### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

路径：`tools/linux/Linux_Pack_Firmware/rockdev/rk356x-package-file`

```bash
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
uboot           Image/uboot.img
misc            Image/misc.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
oem             Image/oem.img
userdata        Image/userdata.img
backup          RESERVED
```