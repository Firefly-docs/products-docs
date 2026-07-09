# Linux Device Tree (DTS) Manual

Sometimes we need to modify device tree to adjust functions of the board. This tutorial will teach you how to find the right device tree file and some notes about modifying it.

## Find the Device Tree

First you need to learn [How to Build Linux SDK](linux_compile.md), after that you should understand that the configurations about building are defined in mk files under `SDK/device/rockchip/rv1126_rv1109/`.

Suppose I need to compile the Buildroot firmware of AIO-1126JD4 to support mipi screen, then the corresponding configuration file is: `aio-rv1126-jd4-mipi101-BSD1218-A101KL68.mk`

cd to `SDK/device/rockhip/rv1126_rv1109/`, find and open `aio-rv1126-jd4-mipi101-BSD1218-A101KL68.mk`, you can see the dts:
```bash
#!/bin/bash

CMD=`realpath $BASH_SOURCE`
CUR_DIR=`dirname $CMD`

source $CUR_DIR/aio-rv1126-jd4.mk

# Kernel dts
export RK_KERNEL_DTS=rv1126-firefly-jd4-mipi101-BSD1218-A101KL68
```
`RK_KERNEL_DTS` shows the dts. No suffix here, so the full name with suffix is `rv1126-firefly-jd4-mipi101-BSD1218-A101KL68.dts`

Some mk files may not have the `RK_KERNEL_DTS` attribute, such as the configuration `aio-rv1126-jd4-mipi101-BSD1218-A101KL68-debian.mk` for compiling Debian10 firmware:
```bash
#!/bin/bash

CMD=`realpath $BASH_SOURCE`
CUR_DIR=`dirname $CMD`

source $CUR_DIR/aio-rv1126-jd4-mipi101-BSD1218-A101KL68.mk

# parameter for GPT table
export RK_PARAMETER=parameter-firefly-debian-fit.txt

# Set rootfs type, including ext2 ext4 squashfs
export RK_ROOTFS_TYPE=ext4

# rootfs image path
export RK_ROOTFS_IMG=ubuntu_rootfs/rootfs.img

# Buildroot config
export RK_CFG_BUILDROOT=

# rootfs_system
export RK_ROOTFS_SYSTEM=ubuntu
```

The reason is that it sources other files. In fact, the device tree information is recorded in `aio-rv1126-jd4-mipi101-BSD1218-A101KL68.mk`. After knowing the device tree name, you can find the device tree under `SDK/kernel/arch/arm/boot/dts/`.

## Modify the Device Tree

Open `SDK/kernel/arch/arm/boot/dts/rv1126-firefly-jd4-mipi101-BSD1218-A101KL68.dts` you will find this file is basically empty, where are the device tree nodes?
```dts
// SPDX-License-Identifier: (GPL-2.0+ OR MIT)
/*
 * Copyright (c) 2020 Rockchip Electronics Co., Ltd.
 */
/dts-v1/;
#include "rv1126.dtsi"
#include "rv1126-firefly-jd4-mipi101-BSD1218-A101KL68.dtsi"
/ {
        model = "Firefly Core-RV1126-JD4 mipi101-BSD1218-A101KL68 Board";
        compatible = "rockchip,rv1126-evb-ddr3-v12", "rockchip,rv1126";
};
```
It included another file `rv1126-firefly-jd4-mipi101-BSD1218-A101KL68.dtsi`, note that the suffix is dtsi, open this dtsi file you can find many device tree nodes. This dtsi file will also include some other files.

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
./build.sh aio-rv1126-jd4-mipi101-BSD1218-A101KL68.mk

# compile kernel
./build.sh kernel
```
The compiled result will be under `SDK/kernel`, which is a mirror image `zboot.img`.

Download to board(choose one method):

1. Or follow the kernel part of [Upgrade Firmware](upgrade_firmware.md) to download zboot.img.