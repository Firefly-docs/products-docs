# Linux Device Tree (DTS) Manual

Sometimes we need to modify device tree to adjust functions of the board. For example: switch M2 slot function between SATA/PCIE, change CAN clock rate, etc. This tutorial will teach you how to find the right device tree file and some notes about modifying it.

## Find the Device Tree

First you need to learn [How to Build Linux SDK](linux_compile.md), after that you should understand that the configurations about building are defined in defconfig files under `SDK/device/rockchip/rk3576/`.

The defconfig format is: firefly_SOC_Model(-Accessory)_OS_defconfig

Assuming that you need to build an AIO-3576JD4 Ubuntu firmware, then the corresponding mk file is `firefly_rk3576_aio-3576jd4_ubuntu_defconfig`

cd to `SDK/device/rockhip/rk3576`, find and open `firefly_rk3576_aio-3576jd4_ubuntu_defconfig`, you can see the dts:
```bash
#include "firefly.config"
RK_EXTRA_PARTITION_1_DEV=""
RK_EXTRA_PARTITION_2_DEV=""
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk3576_ubuntu_rootfs.img"
RK_PARAMETER="parameter-ubuntu-fit.txt"
RK_KERNEL_DTS_NAME="rk3576-firefly-aio-3576jd4"
RK_PRODUCT_MODEL="AIO-3576JD4"
```
`RK_KERNEL_DTS_NAME` shows the dts. No suffix here, so the full name with suffix is `rk3576-firefly-aio-3576jd4.dts`

Some defconfig file may not have `RK_KERNEL_DTS_NAME`, It is because the file includes another file, So the `RK_KERNEL_DTS_NAME` may be defined in that included file.

When you get the name of dts, then you can find it under `SDK/kernel/arch/arm64/boot/dts/rockchip/`.

## Modify the Device Tree

This time we take rk3576-firefly-roc-rk3576-pc.dts as an example:

Open `SDK/kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc.dts` you will find this file is basically empty, where are the device tree nodes?
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
It included some other device tree files, note that the suffix is dtsi, open the dtsi files you can find many device tree nodes. This dtsi file will also include some other files.

So device tree is not a single file but a goup of dts/dtsi files:
```
The basic relationship is:
Upper layer <--------------------> Lower layer
A.dts include B.dtsi include C.dtsi ... N.dtsi
```
1. If you can't find a node in A.dts, then it may be in B.dtsi, or C.dtsi ... follow the inlcude relationship.
2. Configs in upper layer will overwrite that in lower layer. For example: disable uart0 in C.dtsi but enable uart0 in A.dts, the configs in A.dts will take effect, uart0 will be enabled.
3. A dtsi file may be included by many files, modify it will influence many models of boards. So check it before modifying.

Device tree config instructions of functions like Ethernet, PCIE, UART, etc. can be found under `SDK/docs/`.
Modifying device tree needs basic knowledge of kernel and dts grammar, and you need to understand what you are going to do. Improper modifications will cause abnormal functions or even boot failure.

## Compile and Download

Return to SDK root after modifying, use `build.sh` to config and compile:
```bash
# select mk file
./build.sh firefly_rk3576_aio-3576jd4_ubuntu_defconfig

# compile kernel
./build.sh extboot
```
The result will be under `SDK/kernel`: an image `extboot.img` and a directory `extboot`

Download to board(choose one method):

1. Copy all files under `SDK/kernel/extboot/` to the `/boot/` of board, and reboot the board.
2. Or follow the kernel part of [Upgrade Firmware](upgrade_firmware.md) to download extboot.img.