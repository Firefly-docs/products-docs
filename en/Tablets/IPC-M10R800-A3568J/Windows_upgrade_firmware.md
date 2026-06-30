[Linux_adb_fastboot]: http://en.t-firefly.com/doc/download/108.html#other_536
[Androidtool_xxx (version number)]: http://en.t-firefly.com/doc/download/108.html#other_365
[Release_DriverAssistant.zip]: http://en.t-firefly.com/doc/download/108.html#other_366
[Linux_Upgrade_Tool]: http://en.t-firefly.com/doc/download/108.html#other_367# Upgrading firmware on Windows

## Preface

This article describes how to burn the firmware file on the host computer to the flash memory of the IPC-M10R800-A3568J development board through 双公头USB数据线. When upgrading, you need to choose an appropriate upgrade method according to the host operating system and firmware type.

## Preparatory work

* IPC-M10R800-A3568J development board
* Firmware
* Host computer
* Double male USB data cable


## Firmware file
There are two types of firmware files:

* A single unified firmware

	The unified firmware is a single file packaged and merged by all files such as the partition table, bootloader, uboot, kernel, system and so on. The firmware officially released by Firefly adopts a unified firmware format. Upgrading the unified firmware will update the data and partition table of all partitions on the motherboard, and erase all data on the motherboard.

* Multiple partition images

	That is, files with independent functions, such as partition table, bootloader, and kernel, are generated during the development phase. The independent partition image can only update the specified partition, while keeping other partition data from being destroyed, it will be very convenient to debug during the development process.

> Through the unified firmware unpacking / packing tool, the unified firmware can be unpacked into multiple partition images, or multiple partition images can be merged into a unified firmware.

## Flash on windows

* Tool: [Androidtool_xxx (version number)]

### Install RK USB drive

Download [Release_DriverAssistant.zip], extract, and then run the DriverInstall.exe inside .
In order for all devices to use the updated driver, first select `Driver uninstall` and then select `Driver install`.

![](../../../rk356x_img/upgrade_firmware_install_RK_USB.jpg)

### Connected devices

You can put the device into upgrade mode as follows:



The host should prompt for new hardware and configure the driver. Open Device manager and you will see the new Device `Rockusb Device` appear as shown below. If not, you need to go back to the previous step and reinstall the driver.

![](../../../rk356x_img/upgrade_firmware_new_equipment.jpg)

## Upgrade the firmware

Download [Androidtool_xxx (version number)]. AndroidTool defaults to display in Chinese. We need to change it to English. Open `config.ini` with an text editor (like notepad). The starting lines are:

```
#选择工具语言:Selected=1(Chinese);Selected=2(English)
[Language]
Kinds=2
Selected=1
LangPath=Language\
```

Change `Selected=1` to `Selected=2`, and save. From now on, AndroidTool will display in English.Now, run AndroidTool.exe: (Note: If using Windows 7/8, you’ll need to right click it, select to run it as Administrator)

![](../../../rk356x_img/upgrade_firmware_androidtool.jpg)

#### Upgrade unified firmware - update.img

The steps to update the unified firmware `update.img` are as follows:

1. Switch to the "upgrade firmware" page.
2. Press the "firmware" button to open the firmware file to be upgraded. The upgrade tool displays detailed firmware information.
3. Press the "upgrade" button to start the upgrade.
4. <font color=#ff0000 >If the upgrade fails, you can try to erase the Flash by pressing the `erase Flash` button first, and then upgrade. </font>

**Note: if the firmware loader you wrote is inconsistent with the original one, please execute `wipe Flash` before upgrading the firmware.**

![](../../../rk356x_img/upgrade_firmware_erase_flash.jpg)

### Upgrade Partition image
The steps to upgrade the partition image are as follows:
	1. Switch to the "download image" page.
	2. Check the partition to be burned, and select multiple.
	3. Make sure the path of the image file is correct. If necessary, click the blank table cell on the right side of the path to select it again.
	4. Click "Run" button to start the upgrade, and the device will restart automatically after the upgrade.

![](../../../rk356x_img/upgrade_firmware_androidtool.jpg)


## FAQs

### 1. How to forcibly enter MaskRom mode

**A1 :** If the board does not enter Loader mode, you can try to force your way into MaskRom mode. See operation method ["How to enter MaskRom mode"](04-maskrom_mode.md).




### 2. Analysis of programming failure

If Download Boot Fail occurs during the programming process, or an error occurs during the programming process, as shown in the figure below, it is usually caused by the poor connection of the USB cable, the inferior cable, or the insufficient drive capability of the USB port of the computer. Troubleshoot the computer USB port.

![](../../../rk356x_img/upgrade_downloadfail.jpg)