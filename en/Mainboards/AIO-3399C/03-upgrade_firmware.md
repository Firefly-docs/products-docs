# Upgrade the firmware

## Introduction

This article describes how to upgrade the firmware file on the host to the flash memory of the development board through the Type-C data cable. When upgrading, you need to choose the appropriate upgrade mode according to the host operating system and firmware type.

## Preparatory work

* AIO-3399C development board
* Firmware
* Host computer
* Type-C data cable

There are two types of firmware files:

* A single unified firmware

	The unified firmware is a single file packaged and merged by all files such as the partition table, bootloader, uboot, kernel, system and so on. The firmware officially released by Firefly adopts a unified firmware format. Upgrading the unified firmware will update the data and partition table of all partitions on the motherboard, and erase all data on the motherboard.

* Multiple partition images

	That is, files with independent functions, such as partition table, bootloader, and kernel, are generated during the development phase. The independent partition image can only update the specified partition, while keeping other partition data from being destroyed, it will be very convenient to debug during the development process.

> Through the unified firmware unpacking / packing tool, the unified firmware can be unpacked into multiple partition images, or multiple partition images can be merged into a unified firmware.



## Flash on windows

* Tool: [Androidtool_xxx (version number)]

**<font color=#ff0000 >Note :</font>** Different firmware may use different versions of tools, please download the corresponding version according to the [Instructions for writing with USB cable (important)][烧写须知].

### Install RK USB drive

Download [Release_DriverAssistant.zip], extract, and then run the DriverInstall.exe inside .
In order for all devices to use the updated driver, first select `Driver uninstall` and then select `Driver install`.

![](../../../rk3399_img/upgrade_firmware_install_RK_USB.jpg)

### Connected devices

You can put the device into upgrade mode as follows:

* Disconnect the power adapter first:
   * Dual male usb data cable connects one end to the host and the other end to the development board.
   * Press the `RECOVERY` button on the device and hold.
   * Connect to the power supply.
   * About two seconds later, release the `RECOVERY` button.

The host should prompt for new hardware and configure the driver. Open Device manager and you will see the new Device `Rockusb Device` appear as shown below. If not, you need to go back to the previous step and reinstall the driver.

![](../../../rk3399_img/upgrade_firmware_new_equipment.jpg)

### Upgrade the firmware

Download [Androidtool_xxx (version number)]. AndroidTool defaults to display in Chinese. We need to change it to English. Open `config.ini` with an text editor (like notepad). The starting lines are:

```
#选择工具语言:Selected=1(Chinese);Selected=2(English)
[Language]
Kinds=2
Selected=1
LangPath=Language\
```

Change `Selected=1` to `Selected=2`, and save. From now on, AndroidTool will display in English.Now, run AndroidTool.exe: (Note: If using Windows 7/8, you’ll need to right click it, select to run it as Administrator)

![](../../../rk3399_img/upgrade_firmware_androidtool.jpg)

#### Upgrade unified firmware - update.img

The steps to update the unified firmware `update.img` are as follows:

1. Switch to the "upgrade firmware" page.
2. Press the "firmware" button to open the firmware file to be upgraded. The upgrade tool displays detailed firmware information.
3. Press the "upgrade" button to start the upgrade.
4. <font color=#ff0000 >If the upgrade fails, you can try to erase the Flash by pressing the `erase Flash` button first, and then upgrade. Be sure to erase and upgrade according to [Upgrade instructions](02-upgrade_table.md).</font>

**Note: if the firmware loader you wrote is inconsistent with the original one, please execute `wipe Flash` before upgrading the firmware.**

![](../../../rk3399_img/upgrade_firmware_erase_flash.jpg)

#### Upgrade Partition image

Each firmware partition may be different, please note the following two points:

1. When upgrading `ubuntu(MBR)` and `Android7.1` firmware with `Androidtool_2.38`, the default configuration can be used.
2. When upgrading `ubuntu(GPT)` with `Androidtool_2.58`, the default configuration can be used. Please first perform the following actions to upgrade `Android8.1` firmware:<br />
<font color=#ff0000 >Switch to "download image page "; Right click on the table and select "import configuration "; Select the "rk3399-Android81.cfg".</font><br />
3. When upgrading `Android9.0` and `Android10.0` firmware with `Androidtool_2.49`, the default configuration can be used.

The steps to upgrade the partition image are as follows:
	1. Switch to the "download image" page.
	2. Check the partition to be burned, and select multiple.
	3. Make sure the path of the image file is correct. If necessary, click the blank table cell on the right side of the path to select it again.
	4. Click "Run" button to start the upgrade, and the device will restart automatically after the upgrade.

![](../../../rk3399_img/upgrade_firmware_androidtool.jpg)

## Flash on linux

There is no need to install device driver under Linux. Please refer to the Windows section to connect the device.

* Tool : [upgrade_tool_xxx (version number)]

**<font color=#ff0000 >Note :</font>** Different firmware may use different versions of tools, please download the corresponding version according to the [upgrade instructions][烧写须知].

### Upgrade_tool

Download [Linux_Upgrade_Tool] (**Android8.1 need Linux_Upgrade_Tool_for_android8.1**), and install it to host filesystem:
Download [Linux_Upgrade_Tool], And install it into the system as follows for easy invocation:

```
unzip Linux_Upgrade_Tool_xxxx.zip
cd Linux_UpgradeTool_xxxx
sudo mv upgrade_tool /usr/local/bin
sudo chown root:root /usr/local/bin/upgrade_tool
sudo chmod a+x /usr/local/bin/upgrade_tool
```

### Upgrade unified firmware - *update.img*：

```
sudo upgrade_tool uf update.img
```

<font color=#ff0000 >If the upgrade fails, try erasing before upgrading. Be sure to erase and upgrade against the table (upgrade_table.md) in [upgrade instructions][烧写须知].</font>

```
# erase flash : Using the ef parameter requires the loader file or the corresponding update.img to be specified.
# update.img :The ubuntu firmware you need to upgrade.
sudo upgrade_tool ef update.img
# upgrade again
sudo upgrade_tool uf update.img
```

**Upgrade Partition image**

Android7.1, Android8.1 use the following methods:
```
sudo upgrade_tool di -b /path/to/boot.img
sudo upgrade_tool di -k /path/to/kernel.img
sudo upgrade_tool di -s /path/to/system.img
sudo upgrade_tool di -r /path/to/recovery.img
sudo upgrade_tool di -m /path/to/misc.img
sudo upgrade_tool di -re /path/to/resource.img
sudo upgrade_tool di -p paramater   # upgrade parameter
sudo upgrade_tool ul bootloader.bin # upgrade bootloader
```

Android9.0、Android10.0 use the following way:
```
sudo upgrade_tool di -b boot.img
sudo upgrade_tool di -dtbo dtbo.img  
sudo upgrade_tool di -misc misc.img
sudo upgrade_tool di -parameter parameter.txt
sudo upgrade_tool di -r recovery.img
sudo upgrade_tool di -super super.img
sudo upgrade_tool di -trust trust.img
sudo upgrade_tool di -uboot uboot.img
sudo upgrade_tool di -vbmeta vbmeta.img
```

Ubuntu (GPT) uses the following methods:
```
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

```
sudo upgrade_tool lf update.img	# low-level formatting
sudo upgrade_tool ef update.img	# erase
```

## FAQs

### 1. How to forcibly enter MaskRom mode

**A1 :** If the board does not enter Loader mode, you can try to force your way into MaskRom mode. See operation method ["How to enter MaskRom mode"](04-maskrom_mode.md).




### 2. Analysis of programming failure

If Download Boot Fail occurs during the programming process, or an error occurs during the programming process, as shown in the figure below, it is usually caused by the poor connection of the USB cable, the inferior cable, or the insufficient drive capability of the USB port of the computer. Troubleshoot the computer USB port.

![](../../../rk3399_img/upgrade_downloadfail.jpg)

[烧写须知]: 02-upgrade_table.md
[AIO-3399C firmware]: http://en.t-firefly.com/doc/download/52.html
[Androidtool_xxx (version number)]: http://en.t-firefly.com/doc/download/52.html#other_343
[Release_DriverAssistant.zip]: http://en.t-firefly.com/doc/download/52.html#windows_341
[Linux_Upgrade_Tool]: http://en.t-firefly.com/doc/download/52.html#linux_12
[upgrade_tool_xxx (version number)]: http://en.t-firefly.com/doc/download/52.html#linux_12
