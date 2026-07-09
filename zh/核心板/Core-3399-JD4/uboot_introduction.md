# U-Boot 使用

## 前言

RK U-Boot 基于开源的 U-Boot 进行开发，工作模式有启动加载模式和下载模式。启动加载模式是 U-Boot 的正常工作模式，嵌入式产品发布时，U-Boot 都工作在此模式下，主要用于开机时把内存中的内核加载到内存中，启动操作系统；下载模式主要用于将固件下载到闪存，开机时长按 Recovery 键可进入下载模式。本文简单说明 U-Boot 的使用，更多相关文档请看 SDK 下面的 `RKDocs/common/uboot/RockChip_Uboot` 开发文档 `V3.0.pdf`。

## 编译

编译 U-Boot 与编译内核类似，编译前把默认配置写入 .config，执行：

* Android：

```
make rk3399_box_defconfig
```

* linux：

```
make firefly-rk3399
```

如果需要修改相关选项，也可以用:

```
make menuconfig
```

编译：

```
make ARCHV=aarch64
```

编译后生成：

```
u-boot/uboot.img
u-boot/trust.img
u-boot/RK3399MiniLoaderAll_Vx.xx.bin
```

## 烧录

打开烧录工具，板子接好 USB OTG 线，接通电源时按住 Recovery 键，使开发板进入 U-Boot 的下载模式，在烧录工具中选择编译好的 Loader 文件，点击执行即可，如下图：

![](../../../rk3399_img/uboot_download.jpg)

## 确认是否正确烧写新的 Loader

如果你已经成功烧写你最新编译的 Loader，在开机的串口输出中可以看到类似如下信息：

```
#Boot ver: 2016-12-19#1.05
```

如果打印的时间及版本与你编译的一致，说明你成功更新了 Loader。

## 进入 U-Boot 命令行模式

由于 Firefly 产品主要用于开发，所以我们默认设置开机时有1秒的倒计时，如果这时候在串口输入任意键即可进入 U-Boot 命令行模式。 发布的产品是不需要进入 U-Boot 命令行模式的，如果需要设置U-Boot默认不进入命令行模式的，可以做如下修改：

在文件 `u-boot/include/configs/rk33plat.h`

```
/* mod it to enable console commands.  */
#define CONFIG_BOOTDELAY               0
```

把宏 `CONFIG_BOOTDELAY` 改为 0 即默认不进入命令行模式。

## 一级 Loader(RK3328)

U-BOOT 作为一级 Loader 模式，那么仅支持 EMMC 存储设备，编译完成后生成的镜像：

```
RK3288LoaderU-BOOT_V2.17.01.bin
```

其中 `V2.17.01` 是发布的版本号，rockchip 定义U-Boot loader 的版本，其中 `2.17` 是根据存储版本定义的，客户务必不要修改这个版本，`01` 是 U-Boot 定义的小版本，用户根据实际需求在 `Makefile` 中修改。

## 二级 Loader(RK3399/RK3399PRO)

U-Boot 作为二级 Loader 模式，那么固件支持所有的存储设备，该模式下，需要 MiniLoader 支持，通过宏 `CONFIG_MERGER_MINILOADER` 进行配置生成。同时引入 `Arm Trusted`，Firmware 会生成 `trust image`，这个通过宏 `CONFIG_MERGER_TRUSTIMAGE` 进行配置生成。RK3399 使用二级 Loader，编译生成的镜像为：

```
u-boot/uboot.img
u-boot/trust.img
u-boot/RK3399MiniLoaderAll_V1.05.bin
```

其中 `V1.05` 是发布的版本号，rockchip 定义 U-Boot loader 的版本，其中 `1.05` 是根据存储版本定义的，客户务必不要修改这个版本。`uboot.img` 是 U-Boot 作为二级 loader 的打包。 `trust.img` 是 U-Boot 作为二级 loader 的打包。