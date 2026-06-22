## 编译 Buildroot 固件

本章介绍 Buildroot 固件的编译流程，推荐在 Ubuntu 22.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

### 编译 SDK

#### 编译前配置

不同板型的配置文件存放在 `/path/to/sdk/device/rockchip/rk3506/` 目录下

回到 SDK 根目录执行`build.sh`选择配置文件：

```bash
./build.sh firefly_rk3506_roc-rk3506b-cc-amp-emmc_buildroot_defconfig
```

#### SDK统一编译与打包

RK3506 支持 Linux 、HAL 、RT-Thread的 AMP 混合架构设计，使得不同的 CPU 可以运行不同的系统，以满足灵活的产品设计需求。该 SDK 目前默认使用 Linux + RT-Thread 的混合结构模型，其中运行 Linux 的 CPU 为主核，运行 rtt 的 CPU 为从核；0~1 核心运行 Linux ，2 核心运行 RT-Thread 。

##### 编译配置

SDK 的统一编译配置脚本位于 device/rockchip/rk3506/ 目录下，编译配置脚本内容包括 U-Boot、Kernel、HAL、RT-Thread 的配置，以及 AMP 相关的 CPU 分配，内存分配等配置。用户可以根据需求增加或者修改配置脚本文件，以满足自己的编译需求。主要的 AMP 配置文件如下:

```bash
amp_linux.its								# linux + rtt的 AMP 打包 ITS 配置文件
firefly_rk3506_roc-rk3506b-cc-amp-emmc_buildroot_defconfig		# linux + rtt 对应的配置文件
```

统一编译脚本工具支持一键编译及打包 U-Boot、Kernel、HAL、RT-Thread、ROOTFS等，并生成对应的 Image 镜像。

```bash
./build.sh
```

#### 部分编译

* 编译 u-boot

```bash
./build.sh uboot
```

* 编译 kernel

注意：kernel 设备树位于 kernel/arch/arm/boot/dts 目录下，它使用的设备树rk3506b-firefly-roc-rk3506b-cc-amp-emmc.dts只适用于2G内存规格。
如果你需要适用于其他内存规格设备，需自行更改它的内存描述标签memory。

**AMP 使用到的硬件资源需在内核设备树关闭，并在 rk3506-amp.dtsi 设备树的 rockchip_amp 标签下声明**

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

##### 编译 AMP

编译 rt-thread 制作 amp 镜像

###### 打包 amp 固件

* 打包amp镜像

```bash
./build.sh amp
```

使用 /path/to/sdk/device/rockchip/rk3506/amp_linux.its 配置文件打包出 output/firmware/amp.img

###### 其他

hal 或 rt-thread 默认使用 uart5 作为调试串口，波特率为 1.5M 。

#### 打包固件

打包固件，生成的完整固件会保存到 `output/update/` 目录。

```bash
./build.sh updateimg
```

### 分区说明

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 `parameter-buildroot-amp-emmc-fit.txt` 为例：

路径：`device/rockchip/rk3506/parameter-buildroot-amp-emmc-fit.txt`

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
CMDLINE:mtdparts=:0x00002000@0x00002000(uboot),0x00000800@0x00004000(misc),0x0000A000@0x00004800(recovery),0x00010000@0x0000E800(boot),0x00010000@0x0001E800(backup),0x00008000@0x0002E800(oem),0x00002000@0x00036800(amp),0x00c00000@0x00038800(rootfs),-@0x00c36800(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
uuid:boot=7A3F0000-0000-446A-8000-702F00006273
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00002000(uboot) 中 0x00002000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

#### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

路径：`output/firmware/package-file`

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
amp     amp.img
rootfs  rootfs.img
userdata        userdata.img
```