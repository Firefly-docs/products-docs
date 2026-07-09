# Linux 设备树 (DTS) 指南

有时候需要配置设备树来调整板子功能，比如 M.2 接口 SATA/PCIE 功能的切换、CAN 接口时钟的修改等。这篇教程会告诉你如何在 SDK 中寻找到板子对应的设备树文件以及有关修改的注意事项。

## 找到设备树

首先需要学习如何[编译 Linux SDK](linux_compile.md)，之后你应该可以理解，所有的编译配置信息记录在 `SDK/device/rockchip/rk3576/` 下的众多 defconfig 文件里。

格式为：firefly_芯片_板型(-配件)_系统_defconfig

假设我需要编译 AIO-3576JD4 的 Ubuntu 固件，则配置文件为 firefly_rk3576_aio-3576jd4_ubuntu_defconfig

前往 `SDK/device/rockhip/rk3576` 下打开 `firefly_rk3576_aio-3576jd4_ubuntu_defconfig`，可以很直观看到对应的 dts:
```bash
#include "firefly.config"
RK_EXTRA_PARTITION_1_DEV=""
RK_EXTRA_PARTITION_2_DEV=""
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk3576_ubuntu_rootfs.img"
RK_PARAMETER="parameter-ubuntu-fit.txt"
RK_KERNEL_DTS_NAME="rk3576-firefly-aio-3576jd4"
RK_PRODUCT_MODEL="AIO-3576JD4"
```
`RK_KERNEL_DTS_NAME` 属性就记录着对应的 dts 文件名称，此处是不带后缀的，所以完整的 dts 名称为 `rk3576-firefly-aio-3576jd4.dts`

有些 defconfig 文件可能没有 `RK_KERNEL_DTS_NAME` 属性，原因是 defconfig 支持 include，可能 `RK_KERNEL_DTS_NAME` 在 include 的文件中已经配置过了，所以此处不用再配置。

知道设备树名称后，就可以前往 `SDK/kernel/arch/arm64/boot/dts/rockchip/` 下找到该设备树。

## 修改设备树

这次我们使用 rk3576-firefly-roc-rk3576-pc.dts 举例：

打开 `SDK/kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc.dts` 后你可能发现，这个文件基本是空的，如何修改？
```dts
// SPDX-License-Identifier: (GPL-2.0+ OR MIT)
/*
 * Copyright (c) 2024 Rockchip Electronics Co., Ltd.
 *
 */

/dts-v1/;

#include "rk3576-firefly-roc-rk3576-pc.dtsi"
#include "rk3576-roc-pc-cam-8ms1m.dtsi"
#include "rk3576-firefly-roc-rk3576-pc-mipi101-M101014-BE45-A1.dtsi"
//#include "rk3576-firefly-roc-rk3576-pc-ext.dtsi"
/ {
        model = "Firefly ROC-RK3576-PC Linux HDMI";
        compatible = "rockchip,rk3576-firefly-roc-rk3576-pc", "rockchip,rk3576";
};
```
原因是该文件 include 了其他几个设备树文件，注意后缀为 `dtsi`，再次打开这些文件就可以看到设备树节点。同样的，该文件也会 include 其他文件。

所以设备树是多个 dts/dtsi 文件共同组成的，并不是单个文件：
```
设备树基本关系为：
上层 <-----------------------------------> 底层
A.dts include B.dtsi include C.dtsi ... N.dtsi
```
1. 如果在 A.dts 里面找不到某个节点，那么就可能在 B.dtsi 中，或者在 C.dtsi 中...根据包含关系总能找到。
2. 上层的配置会覆盖底层的配置，例如 C.dtsi 中将 uart0 关闭，但 A.dts 中配置为开启，那么 A.dts 中的配置会生效，uart0 开启。
3. 一个 dtsi 可能被多个文件 include，修改会导致多个板型同步修改，所以修改前请明确包含关系。

以太网、PCIE、UART 等各功能的配置指南一般可以在 `SDK/docs/` 下的文档中找到。修改设备树需要基本的 kernel 知识、设备树语法，并且你需要明确的知道你在干什么。修改不当会导致功能不正常，甚至无法开机。

## 编译烧录

修改完成后回到 SDK 根目录，使用 `build.sh` 进行配置与编译：
```bash
# 选择配置文件
./build.sh firefly_rk3576_aio-3576jd4_ubuntu_defconfig

# 编译内核
./build.sh extboot
```
编译的结果会在 `SDK/kernel` 下，是一个镜像 `extboot.img` 和一个文件夹 `extboot`

烧录方法（二选一）：

1. 可以将 `SDK/kernel/extboot/` 下的所有东西复制到板子的 `/boot/` 下，然后重启板子。
2. 或者根据[固件烧录](upgrade_firmware.md)中烧写 kernel 分区的方法烧录 extboot.img