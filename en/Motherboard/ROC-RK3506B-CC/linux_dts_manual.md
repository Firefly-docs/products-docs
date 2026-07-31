# Linux Device Tree (DTS) Manual

Sometimes you need to configure the device tree to adjust the board functions, such as modifying the CAN interface clock, etc. This tutorial will tell you how to find the device tree file corresponding to the board in the SDK and the precautions for modification.

## Find the Device Tree

First you need to learn [How to Build Linux SDK](linux_compile.md), after that you should understand that the configurations about building are defined in defconfig files under `SDK/device/rockchip/rk3506/`.

The defconfig format is: firefly_SOC_Model(-Accessory)_OS_defconfig

Suppose I need to compile the eMMC Buildroot firmware of ROC-RK3506B-CC, the configuration file is `firefly_rk3506_roc-rk3506b-cc-emmc_buildroot_defconfig`

cd to `SDK/device/rockhip/rk3506`, find and open `firefly_rk3506_roc-rk3506b-cc-emmc_buildroot_defconfig`, you can see the dts:
```bash
#include "firefly.config"

RK_WIFIBT=n
RK_ROOTFS_TYPE="ext4"
RK_ROOTFS_EXT4=y
RK_PRODUCT_MODEL="ROC-RK3506B-CC"
RK_EXTRA_PARTITION_1_FSTYPE="ext4"
RK_EXTRA_PARTITION_2_FSTYPE="ext4"
RK_PARAMETER="parameter-buildroot-emmc-fit.txt"
RK_RECOVERY=y
RK_MISC_RECOVERY=y
RK_RECOVERY_RAMDISK="rk3506-recovery-arm-ext4.cpio.gz"
```
The `RK_KERNEL_DTS_NAME` property records the corresponding dts file name, which is defined in the included firefly.config file. There is no suffix here, so the full dts name is `rk3506b-firefly-roc-rk3506b-cc.dts`

Some defconfig file may not have `RK_KERNEL_DTS_NAME`, It is because the file includes another file, So the `RK_KERNEL_DTS_NAME` may be defined in that included file.

When you get the name of dts, then you can find it under `SDK/kernel/arch/arm/boot/dts/`.

## Modify the Device Tree

This time we take rk3506b-firefly-roc-rk3506b-cc.dts as an example:

Open `SDK/kernel/arch/arm/boot/dts/rk3506b-firefly-roc-rk3506b-cc.dts` you will find this file is basically empty, where are the device tree nodes?
```dts
// SPDX-License-Identifier: (GPL-2.0+ OR MIT)
/*
 * Copyright (c) 2024 Rockchip Electronics Co., Ltd.
 */

/dts-v1/;

#define N_LED 1
#define N_ES8390 1
#define N_RTC 1
#define N_WATCH_DOG 1
#define N_PCA9555 1
#define RECOVERY_KEY 1
#define EMMC_OR_SDCARD 1 /* 1 = EMMC, 0 = SDCARD */
#define WIFI_BT 1
#define GMAC0 1
#define GMAC1 1
#define CAN0 0
#define CAN1 0

#include "rk3506.dtsi"
#include "rk3506b-firefly-roc-rk3506b-cc.dtsi"
#include "rk3506b-firefly-roc-rk3506b-cc-mipi101-BSD1218-A101KL68.dtsi"

/ {
        model = "Firefly ROC-RK3506B-CC Linux(MIPI)";
        compatible = "rockchip,rk3506b-firefly-roc-rk3506b-cc", "rockchip,rk3506";
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

Device tree config instructions of functions like Ethernet, UART, etc. can be found under `SDK/docs/`.
Modifying device tree needs basic knowledge of kernel and dts grammar, and you need to understand what you are going to do. Improper modifications will cause abnormal functions or even boot failure.

## Compile and Download

Return to SDK root after modifying, use `build.sh` to config and compile:
```bash
# Select Profile
./build.sh firefly_rk3506_roc-rk3506b-cc-emmc_buildroot_defconfig

# compile kernel
./build.sh kernel
```
The result will be under `SDK/kernel`: an image `zboot.img`

Download to board(choose one method):

1. Burn zboot.img according to the method of burning kernel partition in [Upgrade Firmware](upgrade_firmware.md).