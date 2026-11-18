# Linux Device Tree (DTS) Manual

Sometimes we need to modify device tree to adjust functions of the board. For example: adjust the pin multiplexing of UART/SPI, enable an I2C device, etc. This tutorial will teach you how to find the right device tree file of Firefly-R3572A and some notes about modifying it.

## Find the Device Tree [step]

First you need to learn [How to Build Linux Firmware](linux_compile.md), after that you should understand that the configurations about building are defined in defconfig files under `SDK/device/rockchip/.chips/rk3572/`.

Assuming that you need to build a Debian firmware for Firefly-R3572A, then the corresponding defconfig file is `firefly_rk3572_r3572a_debian_defconfig`.

cd to `SDK/device/rockchip/.chips/rk3572`, find and open `firefly_rk3572_r3572a_debian_defconfig`, you can see the dts:

```
# Kernel dts
export RK_KERNEL_DTS=rk3572-firefly-r3572a
```

`RK_KERNEL_DTS` shows the dts. No suffix here, so the full name with suffix is `rk3572-firefly-r3572a.dts`.

Some defconfig files may not have `RK_KERNEL_DTS`, because the file sourced another file, and the `RK_KERNEL_DTS` is actually defined in the sourced file. Follow the sourcing relationship to find it.

When you get the name of dts, then you can find it under `SDK/kernel-6.12/arch/arm64/boot/dts/rockchip/`.

## Modify the Device Tree [step]

Open `SDK/kernel-6.12/arch/arm64/boot/dts/rockchip/rk3572-firefly-r3572a.dts` you will find this file is basically empty, where are the device tree nodes?

```
// SPDX-License-Identifier: (GPL-2.0+ OR MIT)
/*
 * Copyright (c) 2026 Rockchip Electronics Co., Ltd.
 *
 */
/dts-v1/;
#include "rk3572-firefly-r3572a.dtsi"

/ {
        model = "Firefly R3572A";
        compatible = "rockchip,rk3572-firefly-r3572a", "rockchip,rk3572";

};
```

It included another file `rk3572-firefly-r3572a.dtsi`, note that the suffix is dtsi, open this dtsi file you can find many device tree nodes. This dtsi file will also include some other files.

So device tree is not a single file but a group of dts/dtsi files:

```
The basic relationship is:
Upper layer <--------------------> Lower layer
A.dts include B.dtsi include C.dtsi ... N.dtsi
```

1. If you can't find a node in A.dts, then it may be in B.dtsi, or C.dtsi ... follow the include relationship.
2. Configs in upper layer will overwrite that in lower layer. For example: disable uart0 in C.dtsi but enable uart0 in A.dts, the configs in A.dts will take effect, uart0 will be enabled.
3. A dtsi file may be included by many files, modify it will influence many models of boards. So check it before modifying.

Device tree config instructions of functions like Ethernet, PCIE, UART, etc. can be found under `SDK/docs/`. Modifying device tree needs basic knowledge of kernel and dts grammar, and you need to understand what you are going to do. Improper modifications will cause abnormal functions or even boot failure.

## Compile and Download [step]

Return to SDK root after modifying, use `build.sh` to config and compile:

```
# select defconfig file
./build.sh firefly_rk3572_r3572a_debian_defconfig

# compile kernel
./build.sh extboot
```

The result will be under `SDK/kernel-6.12`: an image `extboot.img` and a directory `extboot`.

Download to board (choose one method):

1. Copy all files under `SDK/kernel-6.12/extboot/` to the `/boot/` of board, and reboot the board.
2. Or follow the kernel part of [Upgrade Firmware](../../getting_started/firmware_upgrade/upgrade_firmware.md) to download extboot.img.