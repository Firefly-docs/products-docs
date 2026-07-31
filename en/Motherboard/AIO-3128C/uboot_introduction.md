# U-Boot Use

## Introduction
RK U-Boot is based on the open source U-Boot. It supports boot loader mode and download mode. Boot loader mode is the normal mode of U-Boot. Device usually releases under this mode. This mode mainly loads the kernel from FLASH to SDRAM and boots the Operating System. User needs to flash the device under the download mode. Keep the recover key pressed while powering on the board, and it will enter download mode. This chapter mainly talks about U-Boot.  

## Compilation

U-Boot compilation is similar to kernel compilation: Write the default configuration to .config before compiling, and execute:
```
cd SDK/u-boot
make rk3128_defconfig
```
If you need to modify relevant options, you can also use: 
```
make menuconfig
```
Compile:
```
make
```
Generated after compilation：
```
RK3128MiniLoaderAll(L)_V2.20.bin
uboot.img
```
RK3128MiniLoaderAll(L)_V2.20.bin and uboot.img use the feature of two stages boot, which supports eMMC and NAND flash.  

## Burn

Open the burn tool, connect the USB OTG cable to the board, press and hold the Recovery button when connecting to power supply, so that the development board enters the U-Boot download mode. Select the required Loader file in the burn tool and click Run, which is shown as below: 

![](../../../rk3128_img/AIO-3128C/win_tool_uboot.png)

## Confirm whether the new Loader is properly programmed

If you flash the loader successfully, there will be some information output from the debug UART as the follow log shows:  
```
#Boot ver: 2015-01-05#2.19
```
If print time and version are consistent with that compiled by you, meaning you’ve successfully updated the Loader.

## Enter U-Boot command mode

Since Firefly is mainly used for development, there are default 1 second countdown before running the Operating System. During the 1 second, any input from the debug UART can make the UBOOT enter the command mode. If you need to set the UBOOT doesn’t enter the command mode default, you can edit the file:U-boot/include/configs/rk32plat.h Change the code
```
#define CONFIG_BOOTDELAY               1
```
to
```
#define CONFIG_BOOTDELAY               0
```
Change 0 to 3, then you'll get 3 seconds delay before a normal boot. Within this duration, press Enter key in the serial console to enter U-Boot's command line mode.