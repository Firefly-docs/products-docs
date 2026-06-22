## 编译 Ubuntu 固件

本章介绍 Ubuntu 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

### Ubuntu 固件简单介绍

[什么是 Ubuntu Minimal固件](ubuntu_minimal_support.md)？

[什么是 Ubuntu Desktop固件？](ubuntu_desktop_support.md)

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
export RK_KERNEL_DEFCONFIG=firefly_linux_defconfig # kernel 配置文件
# Kernel dts
export RK_KERNEL_DTS= # dts 文件
# parameter for GPT table
export RK_PARAMETER=parameter-ubuntu-fit.txt # 分区表
# rootfs image path
export RK_ROOTFS_IMG=ubuntu_rootfs/rk356x_ubuntu_rootfs.img # 根文件系统路径
```

#### 选择编译配件

在 `device/rockchip/rk356x/` 目录下，除了之外，还有其他不同配件搭配的配置文件

配置文件的名称中会标明所使用的屏幕和摄像头。如果没有标明屏幕，说明使用的是默认 HDMI 显示；如果没有标明摄像头，说明使用的是默认单目摄像头

```bash
# 例如：
xxxx-ubuntu.mk                      # 使用 HDMI + 单目摄像头
xxxx-2cam-ubuntu.mk                 # 使用 HDMI + 双目摄像头
xxxx-mipi-ubuntu.mk                 # 使用 mipi + 单目摄像头
xxxx-mipi-2cam-ubuntu.mk            # 使用 mipi + 双目摄像头
```

选择好配置文件后，执行`build.sh`来使其生效：

```bash
# 例如：
./build.sh xxxx-mipi-2cam-ubuntu.mk
```

#### 下载 Ubuntu 根文件系统

* 下载根文件系统：[Ubuntu 根文件系统(64位)]()，放到 SDK 路径下

* 解压文件

```bash
7z x ubuntu-aarch64-rootfs.7z
```

* 将根文件系统放到 `ubuntu_rootfs/` 目录下

```bash
mkdir ubuntu_rootfs
mv ubuntu-aarch64-rootfs.img ubuntu_rootfs/rk356x_ubuntu_rootfs.img
```

#### 全自动编译

全自动编译会执行所有编译、打包操作，直接生成 RK 固件。

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
Linux SDK v1.2.4a 及之后版本采用了 extboot， 编译内核请执行`./build.sh extboot`

生成的文件为 SDK/kernel/extboot.img，取代之前的 boot.img

如何查看版本：
1. 版本格式为 vx.x.xx，例如 v1.2.4a
1. 固件文件名称中存在版本号(..._vx.x.xx_日期.img)
1. Buildroot 使用`cat /etc/version`获取版本(rk356x_linux_release_日期_vx.x.xx.xml)
1. Ubuntu 使用`ffgo version`获取版本(rk356x_linux_release_日期_vx.x.xx.xml)
1. SDK 中可以在 SDK 根目录通过命令查看：`ls -l .repo/manifests/rk356x_linux_release.xml`
1. 如果上述方法找不到格式为 vx.x.xx 的版本，说明是旧版本，不支持 extboot

**不要将 extboot.img 烧录进旧版本固件!**

除此之外，extboot ubuntu 还支持以安装包的形式更新内核，详情查看[Ubuntu 使用手册](https://wiki.t-firefly.com/zh_CN/Firefly-Linux-Guide/manual_ubuntu.html#linux-headers-he-linux-image)

* 编译 recovery

```bash
./build.sh recovery
```

#### 更新链接

更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

#### 打包固件

打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

##### RK 固件

RK 固件，是以 Rockchip 专有格式打包的固件，使用 Rockchip 提供的工具可以烧写到 eMMC 或者 SD 卡中(**注**：若无特殊说明，WIKI 上提及的固件默认为 RK 固件)。

```bash
# 打包 RK 固件
./build.sh updateimg
```

##### RAW 固件

RAW 固件，是一种能以逐位复制的方式烧写到存储设备的固件，是存储设备的原始映像。不同于 RK 固件，目前仅支持通过 Etcher 工具烧写至 SD 卡启动。

[Etcher 官方下载链接](https://www.balena.io/etcher/)

```bash
# 打包 RAW 固件
./build.sh rawimg
```

### 分区说明

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 parameter-ubuntu-fit.txt 为例：

路径：`device/rockchip/rk356x/parameter-ubuntu-fit.txt`

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
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00010000@0x00008000(boot),0x00010000@0x00018000(recovery),0x00010000@0x00028000(backup),0x00c00000@0x00038000(rootfs),-@0x00c38000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

#### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

路径：`tools/linux/Linux_Pack_Firmware/rockdev/rk356x-ubuntu-package-file`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
uboot           Image/uboot.img
misc            Image/misc.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
userdata        RESERVED
backup          RESERVED
```