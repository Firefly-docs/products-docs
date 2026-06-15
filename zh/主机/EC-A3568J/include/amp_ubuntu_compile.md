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

#### SDK统一编译与打包

RK3568 支持 Linux 、HAL 、RT-Thread的 AMP 混合架构设计，使得不同的 CPU 可以运行不同的系统，以满足灵活
的产品设计需求。该 SDK 目前默认使用 Linux + RT-Thread 的混合结构模型，其中运行 Linux
的 CPU 为主核，运行 rtt 的 CPU 为从核；0~2核心运行Linux，3核心运行RT-Thread。

##### 编译配置

SDK 的统一编译配置脚本位于device/rockchip/rk356x/目录下，编译配置脚本内容包括U-Boot、Kernel、
HAL、RT-Thread的配置，以及AMP相关的CPU分配，内存分配等配置。用户可以根据需求增加或者修改
配置脚本文件，以满足自己的编译需求。主要的AMP配置文件如下:

```bash
rk3568_amp_linux.its                    # linux+hal的AMP打包ITS配置文件
rk3568_amp_linux_rtt.its                # linux+rtt的AMP打包ITS配置文件
firefly-rk3568-amp.mk                   # linux+hal对应的AMP配置文件，也是AMP基础配置文件
firefly-rk3568-linux-rtt.mk             # linux+rtt对应的AMP配置文件
                 # AIO-3568J对应板级的配置脚本文件
```

统一编译脚本工具支持一键编译及打包U-Boot、Kernel、HAL、RT-Thread、ROOTFS等，并生成对应的
Image镜像。

```bash
./build.sh
```

#### 部分编译

* 编译 u-boot

```bash
./build.sh uboot
```

* 编译 kernel

注意：kernel 设备树位于kernel/arch/arm64/boot/dts/rockchip/目录下，它使用的设备树只适用于2G内存规格。
如果你需要适用于其他内存规格设备，需自行更改它的内存描述标签memory。

**AMP使用到的硬件资源需在内核设备树关闭，并在rk3568-amp.dtsi设备树的rockchip_amp标签下声明**

```bash
./build.sh kernel
```
生成的镜像为 kernel/boot.img


* 编译 recovery

```bash
./build.sh recovery
```

##### 编译AMP

编译hal或rt-thread制作amp镜像

###### 使用hal

* 编译hal
```bash
./build.sh hal 3
```
生成的镜像为 hal/project/rk3568/GCC/hal3.bin
会被链接到 rockdev/amp/cpu3.bin

###### 使用rt-thread

* 编译rt-thread

```bash
./build.sh rtthread 3
```
生成的镜像为 rt-thread/bsp/rockchip/rk3568-32/Image/rtt3.bin
会被链接到 rockdev/amp/cpu3.bin

###### 打包amp固件

* 打包amp镜像
```bash
./build.sh ampimg
```
使用rockdev/amp/amp.its配置文件打包出rockdev/amp/amp.img

###### 其他

hal或rt-thread默认使用uart4作为调试串口，波特率为115200

* 更加详细说明请阅读SDK/docs/amp/manuals/rk3568下的amp相关文档

#### 打包固件

* 更新链接

更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

* 打包固件

打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```

### 分区说明

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 parameter-amp-ubuntu-fit.txt 为例：

路径：`device/rockchip/rk356x/parameter-amp-ubuntu-fit.txt`

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
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00001000@0x00006000(misc),0x00001000@0x00007000(amp),0x00040000@0x00008000(boot:bootable),0x00020000@0x00048000(recovery),0x00010000@0x00068000(backup),0x49dc00@0x00078000(rootfs),-@0x515c00(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

#### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

路径：`tools/linux/Linux_Pack_Firmware/rockdev/rk356x-amp-ubuntu-package-file`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
uboot           Image/uboot.img
misc            Image/misc.img
amp             Image/amp.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
userdata        RESERVED
backup          RESERVED
```