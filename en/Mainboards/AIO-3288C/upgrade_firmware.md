# Upgrade the firmware

## Introduction

This article describes how to upgrade the firmware file on the host to the flash memory of the development board through the Dual male USB data cale. When upgrading, you need to choose the appropriate upgrade mode according to the host operating system and firmware type.

## Preparatory work

* AIO-3288C development board
* Firmware
* host computer
* Dual male USB data cale

## Firmware

There are two types of firmware files:

* A single unified firmware

    The unified firmware is a single file packaged and merged by all files such as the partition table, bootloader, uboot, kernel, system and so on. The firmware officially released by Firefly adopts a unified firmware format. Upgrading the unified firmware will update the data and partition table of all partitions on the motherboard, and erase all data on the motherboard.

* Multiple partition images

    That is, files with independent functions, such as partition table, bootloader, and kernel, are generated during the development phase. The independent partition image can only update the specified partition, while keeping other partition data from being destroyed, it will be very convenient to debug during the development process.

> Through the unified firmware unpacking / packing tool, the unified firmware can be unpacked into multiple partition images, or multiple partition images can be merged into a unified firmware.

## Windows

* Tool: [Androidtool_xxx (version number)]

**<font color=#ff0000 >Note :</font>** Different firmware may use different versions of tools, please download the corresponding version according to the ["Instruction of upgrade"].

### Install RK USB drive

Download [Release_DriverAssistant.zip], extract, and then run the DriverInstall.exe inside.
In order for all devices to use the updated driver, first select `Driver uninstall` and then select `Driver install`.

![](../../../rk3288_img/upgrade_firmware_install_RK_USB.jpg)

### Connected devices

You can put the device into upgrade mode as follows:

!INCLUDE "upgrade_firmware_enter_loader.mdpp"

AIO-3288C:

![](../../../rk3288_img/AIO-3288C/download_otg.png)



The host should prompt for new hardware and configure the driver. Open Device manager and you will see the new Device `Rockusb Device` appear as shown below. If not, you need to go back to the previous step and reinstall the driver.

![](../../../rk3288_img/upgrade_firmware_new_equipment.jpg)

### Upgrade the firmware

Download [AndroidTool]. AndroidTool defaults to display in Chinese. We need to change it to English. Open `config.ini` with an text editor (like notepad). The starting lines are:

```bash
#选择工具语言:Selected=1(Chinese);Selected=2(English)
[Language]
Kinds=2
Selected=1
LangPath=Language\
```

Change `Selected=1` to `Selected=2`, and save. From now on, AndroidTool will display in English. Now, run AndroidTool.exe: (Note: If using Windows 7/8, you’ll need to right click it, select to run it as Administrator)

![](../../../rk3288_img/upgrade_firmware_androidtool.jpg)

#### Upgrade unified firmware - update.img

The steps to update the unified firmware `update.img` are as follows:

1. Switch to the "upgrade firmware" page.
2. Press the "firmware" button to open the firmware file to be upgraded. The upgrade tool displays detailed firmware information.
3. Press the "upgrade" button to start the upgrade.
4. <font color=#ff0000 >If the upgrade fails, you can try to erase the Flash by pressing the `Erase Flash` button first, and then upgrade. Be sure to erase and upgrade according to ["Instruction of upgrade"]</font>.

**Note: if the firmware loader you wrote is inconsistent with the original one, please execute `Erase Flash` before upgrading the firmware.**

![](../../../rk3288_img/upgrade_firmware_erase_flash.jpg)

#### Upgrade Partition image

The partition of each firmware may be different, please make sure the address information of the partition and the `parameter` file are consistent.

The steps to upgrade the partition image are as follows:

1. Switch to the "download image" page.
2. Check the partition to be download, and select multiple.
3. Make sure the path of the image file is correct. If necessary, click the blank table cell on the right side of the path to select it again.
4. Click "Run" button to start the upgrade, and the device will restart automatically after the upgrade.

![](../../../rk3288_img/upgrade_firmware_androidtool.jpg)

## Linux

There is no need to install device driver under Linux. Please refer to the Windows section to connect the device.

* Tool : [upgrade_tool_xxx (version number)]

**<font color=#ff0000 >Note :</font>** Different firmware may use different versions of tools, please download the corresponding version according to the ["Instruction of upgrade"].

### Upgrade_tool

Download [Linux_Upgrade_Tool], And install it into the system as follows for easy invocation:

```bash
unzip Linux_Upgrade_Tool_xxxx.zip
cd Linux_UpgradeTool_xxxx
sudo mv upgrade_tool /usr/local/bin
sudo chown root:root /usr/local/bin/upgrade_tool
sudo chmod a+x /usr/local/bin/upgrade_tool
```

### Upgrade unified firmware - *update.img*

```bash
sudo upgrade_tool uf update.img
```

<font color=#ff0000 >If the upgrade fails, try erasing before upgrading. Be sure to erase and upgrade against the table in ["Instruction of upgrade"]</font>.

```bash
# erase flash : Using the ef parameter requires the loader file or the corresponding update.img to be specified.
# update.img :The ubuntu firmware you need to upgrade.
sudo upgrade_tool ef update.img
# upgrade again
sudo upgrade_tool uf update.img
```

### Upgrade Partition image

Linux(MBR)、Android5.1: Using the following methods:

```bash
sudo upgrade_tool di -b /path/to/boot.img
sudo upgrade_tool di -k /path/to/kernel.img
sudo upgrade_tool di -s /path/to/system.img
sudo upgrade_tool di -r /path/to/recovery.img
sudo upgrade_tool di -m /path/to/misc.img
sudo upgrade_tool di -re /path/to/resource.img
sudo upgrade_tool di -p paramater   # upgrade parameter
sudo upgrade_tool ul bootloader.bin # upgrade bootloader
```

Linux(GPT): Using the following methods:

```bash
sudo upgrade_tool ul $LOADER
sudo upgrade_tool di -p $PARAMETER
sudo upgrade_tool di -uboot $UBOOT
sudo upgrade_tool di -trust $TRUST
sudo upgrade_tool di -boot $BOOT
sudo upgrade_tool di -recovery $RECOVERY
sudo upgrade_tool di -misc $MISC
sudo upgrade_tool di -oem $OEM
sudo upgrade_tool di -userdata $USERDATA
sudo upgrade_tool di -rootfs $ROOTFS
```

If the upgrade fails due to flash problems, you can try low-level formatting and erase nand flash:

```bash
sudo upgrade_tool lf update.img # low-level formatting
sudo upgrade_tool ef update.img # erase
```

## FAQs

### 1. How to forcibly enter MaskRom mode

**A1 :** If the board does not enter Loader mode, you can try to force your way into MaskRom mode. See operation method ["How to enter MaskRom mode"](maskrom_mode.md).

### 2. Analysis of programming failure

If Download Boot Fail occurs during the programming process, or an error occurs during the programming process, as shown in the figure below, it is usually caused by the poor connection of the USB cable, the inferior cable, or the insufficient drive capability of the USB port of the computer. Troubleshoot the computer USB port.

![](../../../rk3288_img/upgrade_downloadfail.jpg)

["Instruction of upgrade"]: upgrade_table.md
[AIO-3288C firmware]: http://en.t-firefly.com/doc/download/page/id/47.html
[Androidtool_xxx (version number)]: http://en.t-firefly.com/doc/download/47.html#windows_22
[Androidtool]: http://en.t-firefly.com/doc/download/47.html#windows_22
[Release_DriverAssistant.zip]: http://en.t-firefly.com/doc/download/47.html#other_23
[Linux_Upgrade_Tool]: http://en.t-firefly.com/doc/download/47.html#linux_22
[upgrade_tool_xxx (version number)]: http://en.t-firefly.com/doc/download/47.html#linux_22