# U-Boot 使用

## 前言

RK U-Boot 基于开源的 U-Boot 进行开发，工作模式有启动加载模式和下载模式。启动加载模式是 U-Boot 的正常工作模式，嵌入式产品发布时，U-Boot 都工作在此模式下，主要用于开机时把存储器中的内核加载到内存中，启动操作系统；下载模式主要用于将固件下载到存储器，开机时长按 Recovery 键可进入下载模式。本文简单说明 U-Boot 的使用，详细请参考[Uboot开发](https://download.t-firefly.com/product/Board/RK356X/Document/Developer/Rockchip_Developer_Guide_UBoot_Nextdev_CN.pdf)

## 编译

编译 U-Boot 与编译内核类似，编译前把默认配置写入 .config，执行：

```
make 
```

如果需要修改相关选项，也可以用:

```
make menuconfig
```

编译：

```
./make.sh rk3568
```

编译后生成：

```
u-boot/uboot.img
u-boot/
```

## 烧录

打开烧录工具，板子接好 USB OTG 线，接通电源时按住 Recovery 键，使开发板进入 U-Boot 的下载模式，在烧录工具中选择编译好的 Loader 文件，点击执行即可，如下图：
![](../../../rk356x_img/uboot_download.jpg)

## 确认是否正确烧写新的 Loader

如果你已经成功烧写你最新编译的 Loader，在开机的串口输出中可以看到类似如下信息：

```
Android 11.0, Build 2021.2, v2
```

如果打印的时间及版本与你编译的一致，说明你成功更新了 Loader。

## 进入 U-Boot 命令行模式

系统启动时通过调试串口长按`ctrl+c`即可进入U-Boot命令模式
```shell
=> <INTERRUPT>
=> <INTERRUPT>
=>
=> help
?       - alias for 'help'
android_print_hdr- print android image header
base    - print or set address offset
bdinfo  - print Board Info structure
bidram_dump- Dump bidram layout
boot    - boot default, i.e., run 'bootcmd'
boot_android- Execute the Android Bootloader flow.
boot_fit- Boot FIT Image from memory or boot/recovery partition
bootavb - Execute the Android avb a/b boot flow.
bootd   - boot default, i.e., run 'bootcmd'
bootefi - Boots an EFI payload from memory
booti   - boot arm64 Linux Image image from memory
bootm   - boot application image from memory
bootp   - boot image via network using BOOTP/TFTP protocol
bootrkp - Boot Linux Image from rockchip image type
bootz   - boot Linux zImage image from memory
cmp     - memory compare

```

## 二级 Loader

RK3568 使用二级 Loader，编译生成的镜像为：

```
u-boot/uboot.img
u-boot/
```

其中 `V` 是发布的版本号，rockchip 定义 U-Boot loader 的版本，其中 `` 是根据存储版本定义的，客户务必不要修改这个版本。`uboot.img` 包含 uboot 固件。