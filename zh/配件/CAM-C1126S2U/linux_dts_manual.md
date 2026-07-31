# Linux 设备树 (DTS) 指南

有时候需要配置设备树来调整板子功能，比如需要对外设进行适配与修改等。这篇教程会告诉你如何在 SDK 中寻找到板子对应的设备树文件以及有关修改的注意事项。

## 找到设备树

首先需要学习如何[编译 Linux SDK](linux_compile.md)，之后你应该可以理解，所有的编译配置信息记录在 `SDK/device/rockchip/rv1126_rv1109/` 下的众多 mk 文件里。

假设我需要编译 AIO-1126JD4 支持 mipi 屏幕的 Buildroot 固件，那么对应的配置文件为：`aio-rv1126-jd4-mipi101-BSD1218-A101KL68.mk`

前往 `SDK/device/rockhip/rv1126_rv1109/` 下寻找 `aio-rv1126-jd4-mipi101-BSD1218-A101KL68.mk` 并打开，可以很直观看到对应的 dts:
```bash
#!/bin/bash

CMD=`realpath $BASH_SOURCE`
CUR_DIR=`dirname $CMD`

source $CUR_DIR/aio-rv1126-jd4.mk

# Kernel dts
export RK_KERNEL_DTS=rv1126-firefly-jd4-mipi101-BSD1218-A101KL68
```
`RK_KERNEL_DTS` 属性就记录着对应的 dts 文件名称，此处是不带后缀的，所以完整的 dts 名称为 `rv1126-firefly-jd4-mipi101-BSD1218-A101KL68.dts`

有些 mk 文件可能没有 `RK_KERNEL_DTS` 属性，例如编译 Debian10 固件的配置 `aio-rv1126-jd4-mipi101-BSD1218-A101KL68-debian.mk`:
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
原因是它 source 了其他文件，其实设备树信息是记录在了 `aio-rv1126-jd4-mipi101-BSD1218-A101KL68.mk` 中。知道设备树名称后，就可以前往 `SDK/kernel/arch/arm/boot/dts/` 下找到该设备树。

## 修改设备树

打开 `SDK/kernel/arch/arm/boot/dts/rv1126-firefly-jd4-mipi101-BSD1218-A101KL68.dts` 后你可能发现，这个文件基本是空的，如何修改？
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
原因是该文件 include 了 `rv1126-firefly-jd4-mipi101-BSD1218-A101KL68.dtsi`，注意后缀为 `dtsi`，再次打开这个文件就可以看到设备树节点。同样的，该文件也会 include 其他文件。

所以设备树是多个 dts/dtsi 文件共同组成的，并不是单个文件：
```
设备树基本关系为：
上层 <-----------------------------------> 底层
A.dts include B.dtsi include C.dtsi ... N.dtsi
```
1. 如果在 A.dts 里面找不到某个节点，那么就可能在 B.dtsi 中，或者在 C.dtsi 中...根据包含关系总能找到。
2. 上层的配置会覆盖底层的配置，例如 C.dtsi 中将 uart0 关闭，但 A.dts 中配置为开启，那么 A.dts 中的配置会生效，uart0 开启。
3. 一个 dtsi 可能被多个文件 include，修改会导致多个板型同步修改，所以修改前请明确包含关系。

以太网、USB、UART 等各功能的配置指南一般可以在 `SDK/docs/` 下的文档中找到。修改设备树需要基本的 kernel 知识、设备树语法，并且你需要明确的知道你在干什么。修改不当会导致功能不正常，甚至无法开机。

## 编译烧录

修改完成后回到 SDK 根目录，使用 `build.sh` 进行配置与编译：
```bash
# 选择配置文件
./build.sh aio-rv1126-jd4-mipi101-BSD1218-A101KL68.mk

# 编译内核
./build.sh kernel
```
编译的结果会在 `SDK/kernel` 下，是一个镜像 `zboot.img`。

烧录方法：

1. 根据[固件烧录](upgrade_firmware.md)中烧写 kernel 分区的方法烧录 zboot.img 。