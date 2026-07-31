# Linux Device Tree (DTS) Manual

Sometimes it's necessary to configure the device tree to adjust board functionality, such as modifying the CAN interface clock. This tutorial will show you how to find the corresponding device tree file for your board in the SDK and some important points to note when making modifications.

## Find the Device Tree

First you need to learn [How to Build Linux SDK](linux_compile.md), after that you should understand that the configurations about building are defined in defconfig files under `SDK/device/rockchip/rv1126b/`.

The defconfig format is: firefly_SOC_Model(-Accessory)_OS_defconfig

Assuming that you need to build an AIO-1126BJD4 Debian firmware, then the corresponding mk file is `firefly_rv1126b_aio-1126bjd4_debian_defconfig`

cd to `SDK/device/rockhip/rv1126b`, find and open `firefly_rv1126b_aio-1126bjd4_debian_defconfig`, you can see the dts:
```bash
#include "firefly.config"
RK_UBOOT_INI="RV1126BMINIALL.ini"
RK_UBOOT_SPL=y
RK_KERNEL_DTS_NAME="rv1126b-firefly-aio-1126bjd4"
RK_KERNEL_CFG="rockchip_linux_defconfig"
RK_MISC_RECOVERY=y
RK_RECOVERY_RAMDISK="common-recovery-arm64.cpio.gz"
RK_BOOT_FIT_ITS_NAME="bootramdisk.its"
RK_RAMDISK_IMG="kernel/ramdisk-64.img"
RK_PRODUCT_MODEL="AIO-1126BJD4V0"
RK_PARAMETER="parameter-debian-fit.txt"
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rv1126b_debian_rootfs.img"
```
`RK_KERNEL_DTS_NAME` shows the dts. No suffix here, so the full name with suffix is `rv1126b-firefly-aio-1126bjd4.dts`

Some defconfig file may not have `RK_KERNEL_DTS_NAME`, It is because the file includes another file, So the `RK_KERNEL_DTS_NAME` may be defined in that included file.

When you get the name of dts, then you can find it under `SDK/kernel/arch/arm64/boot/dts/rockchip/`.

## Modify the Device Tree

This time we take rv1126b-firefly-aio-1126bjd4.dts as an example:

Open `SDK/kernel/arch/arm64/boot/dts/rockchip/rv1126b-firefly-aio-1126bjd4.dts` you will find this file is basically empty, where are the device tree nodes?
```dts
// SPDX-License-Identifier: (GPL-2.0+ OR MIT)
/*
 * Copyright (c) 2020 Rockchip Electronics Co., Ltd.
 */

#define FF_SDCARD
#define FF_GMAC
#define FF_WIFI_BT
#define FF_LEDS
#define FF_ES8390
#define FF_MIC
#define FF_USB20_HOST0
#define FF_4G_MODULE
#define FF_RTC
#define FF_CAN_0
#define FF_CAN_1
#define FF_CAM_CSI0_8MS1M
#define FF_CAM_CSI1_8MS1M
#define FF_CAM_CSI0_IMX415
#define FF_CAM_CSI1_IMX415
#define FF_CAM_CSI0_IMX586
#define FF_CAM_CSI1_IMX586
#define FF_DSI_V3
#define FF_I2C3
#define FF_I2C5
#define FF_RS485_1
#define FF_RS485_2
#define FF_RS232_1
#define FF_RS232_2
#define FF_UART5
#define FF_OUTPUT

#include "rv1126b-firefly-aio-1126bjd4.dtsi"

#ifdef FF_CAM_CSI0_8MS1M
#include "rv1126b-firefly-aio-1126bjd4-csi0-8ms1m.dtsi"
#endif

#ifdef FF_CAM_CSI1_8MS1M
#include "rv1126b-firefly-aio-1126bjd4-csi1-8ms1m.dtsi"
#endif

#ifdef FF_CAM_CSI0_IMX415
#include "rv1126b-firefly-aio-1126bjd4-csi0-imx415.dtsi"
#endif

#ifdef FF_CAM_CSI1_IMX415
#include "rv1126b-firefly-aio-1126bjd4-csi1-imx415.dtsi"
#endif

#ifdef FF_CAM_CSI0_IMX586
#include "rv1126b-firefly-aio-1126bjd4-csi0-imx586.dtsi"
#endif

#ifdef FF_CAM_CSI1_IMX586
#include "rv1126b-firefly-aio-1126bjd4-csi1-imx586.dtsi"
#endif

#ifdef FF_DSI_V3
#include "rv1126b-firefly-aio-1126bjd4-mipi101-BSD1218-A101KL68.dtsi"
#endif

/ {
        model = "Firefly AIO-1126BJD4 mipi101-BSD1218-A101KL68 Board";
        compatible = "firefly,aio-1126bjd4", "rockchip,rv1126b";
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
# select defconfig file
./build.sh firefly_rv1126b_aio-1126bjd4_debian_defconfig

# compile kernel
./build.sh kernel
```
The compilation result will be located in `SDK/kernel`, and will be an image `boot.img`.

Download to board(choose one method):

1. Follow the kernel part of [Upgrade Firmware](upgrade_firmware.md) to download boot.img.