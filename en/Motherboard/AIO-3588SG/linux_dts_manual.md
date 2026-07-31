# Linux Device Tree (DTS) Manual

Sometimes we need to modify device tree to adjust functions of the board. For example: switch M2 slot function between SATA/PCIE, change CAN clock rate, etc. This tutorial will teach you how to find the right device tree file and some notes about modifying it.

## Find the Device Tree

First you need to learn [How to Build Linux SDK](linux_compile.md), after that you should understand that the configurations about building are defined in mk files under `SDK/device/rockchip/rk3588/`.

Assuming that you need to build an ITX-3588J Ubuntu firmware that supports mipi screen, then the corresponding mk file is `itx-3588j-BE45-A1-ubuntu.mk`

cd to `SDK/device/rockhip/rk3588`, find and open `itx-3588j-BE45-A1-ubuntu.mk`, you can see the dts:
```bash
CMD=`realpath $BASH_SOURCE`
CUR_DIR=`dirname $CMD`

source $CUR_DIR/itx-3588j-ubuntu.mk

# Kernel dts
export RK_KERNEL_DTS=rk3588-firefly-itx-3588j-mipi101-M101014-BE45-A1
```
`RK_KERNEL_DTS` shows the dts. No suffix here, so the full name with suffix is `rk3588-firefly-itx-3588j-mipi101-M101014-BE45-A1.dts`

Some mk file may not have `RK_KERNEL_DTS`, such as the `itx-3588j-kylin.mk` for building kylin firmware:
```bash
CMD=`realpath $BASH_SOURCE`
CUR_DIR=`dirname $CMD`

DEVICE_NAME=$(echo ${CMD} | awk -F '/' '{print $NF}')
DEVICE_NAME=${DEVICE_NAME%\-*}
source $CUR_DIR/${DEVICE_NAME}-ubuntu.mk

# parameter for GPT table
export RK_PARAMETER=parameter-kylin.txt
```
It is because the file sourced another file `${DEVICE_NAME}-ubuntu.mk`, `${DEVICE_NAME}` is a variable, it means the model of the board. So the `RK_KERNEL_DTS` actually defined in `itx-3588j-ubuntu.mk`.

When you get the name of dts, then you can find it under `SDK/kernel/arch/arm64/boot/dts/rockchip/`.

## Modify the Device Tree

Open `SDK/kernel/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j-mipi101-M101014-BE45-A1.dts` you will find this file is basically empty, where are the device tree nodes?
```dts
// SPDX-License-Identifier: (GPL-2.0+ OR MIT)
/*
 * Copyright (c) 2021 Rockchip Electronics Co., Ltd.
 *
 */
/dts-v1/;
#include "rk3588-firefly-itx-3588j-mipi101-M101014-BE45-A1.dtsi"

/ {
        model = "Firefly ITX-3588J MIPI(Linux)";
        compatible = "rockchip,rk3588-firefly-itx-3588j-mipi101-M101014-BE45-A1", "rockchip,rk3588";

};
```
It included another file `rk3588-firefly-itx-3588j-mipi101-M101014-BE45-A1.dtsi`, note that the suffix is dtsi, open this dtsi file you can find many device tree nodes. This dtsi file will also include some other files.

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
./build.sh itx-3588j-BE45-A1-ubuntu.mk

# compile kernel
./build.sh extboot
```
The result will be under `SDK/kernel`: an image `extboot.img` and a directory `extboot`

Download to board(choose one method):

1. Copy all files under `SDK/kernel/extboot/` to the `/boot/` of board, and reboot the board.
2. Or follow the kernel part of [Upgrade Firmware](upgrade_firmware.md) to download extboot.img.