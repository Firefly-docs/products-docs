# U-Boot use

## Introduction

RK U-Boot is developed based on the open source U-Boot. It has two working modes: Bootloading mode and download mode, and the former is the usual working mode. U-Boot works under this mode when the embedded product is released, mainly to load the kernel in memory into the memory and start the operating system when booting. Download mode is designed to download the firmware to the flash memory. Press and hold the Recovery button to enter the download mode when booting. This article briefly explains the use of U-Boot. For more information, please read the `RKDocs/common/uboot/RockChip_Uboot Development document V3.0.pdf` under the SDK.

## Compilation

U-Boot compilation is similar to kernel compilation: Write the default configuration to .config before compiling, and execute:  

```bash
cd SDK/u-boot
make rk3288_defconfig
```

If you need to modify relevant options, you can also use:

```bash
make menuconfig
```

Compile：

```bash
make
```

Generated after compilation：

```bash
RK3288Loader_uboot_Vx.xx.xx.bin
```

## Burn

Open the burn tool, connect the USB OTG cable to the board, press and hold the Recovery button when connecting to power supply, so that the development board enters the U-Boot download mode. Select the required Loader file in the burn tool and click Run, which is shown as below:

![](../../../rk3288_img/uboot_download.jpg)  

## Confirm whether the new Loader is properly programmed

If you have successfully programmed your newly compiled Loader, you’ll see information or similar information in the serial output when booting:

```bash
#Boot ver: 2015-01-05#2.19
```

If print time and version are consistent with that compiled by you, meaning you’ve successfully updated the Loader.

## Enter U-Boot command line mode

Normally you do not need to enter the command line mode of U-Boot. But if it is required to schedule U-Boot, you may modify include/configs/rk32plat.h:

```c
/* mod it to enable console commands.  */
#define CONFIG_BOOTDELAY               0
```

Change 0 to 3 to have a delay of 3 seconds when booting. Enter any key in the serial port during this time to enter the u-boot command line mode. 0 means not to enter the command line mode by default.

## Secondary Loader

Since the Firefly development board does not use nand flash, it is by default not to use the secondary loader. Here only gives a brief description about the secondary Loader.

RK3288Loader_uboot_Vx.xx.xx.bin is a Level 1 Loader mode that only supports emmc.
RK3288Loader_uboot_Vx.xx.xx.bin and uboot.img combination is secondary loader mode that supports emmc and nand flash. Under the second loader mode, you need to add the following definitions to the rk32xx.h configuration file:

```c
#define CONFIG_SECOND_LEVEL_BOOTLOADER
```

Recompile Uboot after adding, and generate: RK3288Loader_uboot_Vx.xx.xx.bin and uboot.img. Write RK3288Loader_uboot_Vx.xx.xx.bin and uboot.img into the board.