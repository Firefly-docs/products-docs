## 编译 Ubuntu 固件

本章介绍 Ubuntu 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

### Ubuntu 固件简单介绍

[什么是 Ubuntu Minimal固件？](ubuntu_minimal_support.md)


### 编译 SDK

#### 编译前配置

不同板型的配置文件存放在`device/rockchip/rk3308/`目录下

回到 SDK 根目录执行`build.sh`选择配置文件：

```bash
./build.sh roc-rk3308b-cc-plus-ubuntu.mk
```

配置文件会链接到 `device/rockchip/.BoardConfig.mk`，检查该文件可以验证是否配置成功。

相关配置介绍：

```bash
# Target arch
export RK_ARCH=arm64 # 64位 ARM 架构
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly-rk3308-debug-uart4 # u-boot 配置文件
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly-rk3308b_linux_defconfig # kernel 配置文件
# Kernel dts
export RK_KERNEL_DTS=rk3308b-roc-cc-plus-amic_emmc # dts 文件
# parameter for GPT table
export RK_PARAMETER=parameter-64bit-ubuntu.txt # 分区表
# rootfs image path
export RK_ROOTFS_IMG=ubuntu_rootfs/rk3308-ubuntu_rootfs.img # 根文件系统路径
```

#### 下载 Ubuntu 根文件系统

* 下载根文件系统：[Ubuntu 根文件系统(64位)](https://www.t-firefly.com/doc/download/73.html#other_601)，放到 SDK 路径下

* 将根文件系统放到SDK的 `ubuntu_rootfs/` 目录下

```bash
mkdir ubuntu_rootfs
mv ubuntu-aarch64-rootfs.img ubuntu_rootfs/rk3308-ubuntu_rootfs.img
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

```bash
./build.sh kernel
```

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

```bash
# 打包 RK 固件
./build.sh updateimg
```

### 分区说明

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 parameter-ubuntu-fit.txt 为例：

路径：`device/rockchip/rk3308/parameter-64bit-ubuntu.txt`

```bash
FIRMWARE_VER:8.1
MACHINE_MODEL:RK3308
MACHINE_ID:007
MANUFACTURER: RK3308
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3308
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00001000@0x00002000(uboot),0x00001000@0x00003000(trust),0x00000800@0x00004000(misc),0x0000A000@0x00004800(recovery),0x0000A000@0x0000E800(boot),0x00100000@0x00018800(rootfs),-@0x118800(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00001000@0x00002000(uboot) 中 0x00001000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

#### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

路径：`tools/linux/Linux_Pack_Firmware/rockdev/rk3308-package-file-ubuntu`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
trust       Image/trust.img
uboot       Image/uboot.img
boot        Image/boot.img
rootfs      Image/rootfs.img
recovery        Image/recovery.img
#oem                    Image/oem.img
userdata:grow    RESERVED
misc            Image/misc.img
backup          RESERVED
#update-script  update-script
#recover-script recover-script
```

### 更多内容

 [《Firefly Ubuntu使用手册》](https://wiki.t-firefly.com/zh_CN/Firefly-Linux-Guide/manual_ubuntu.html)
