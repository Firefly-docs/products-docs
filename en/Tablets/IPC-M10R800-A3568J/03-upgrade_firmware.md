# Upgrade the firmware via USB cable

## Introduction

This article describes how to upgrade the firmware file on the host to the flash memory of the development board through the Double male USB data cable. When upgrading, you need to choose the appropriate upgrade mode according to the host operating system and firmware type.

## Preparatory Tools

* AIO-3568J development board
* Firmware
* host computer
* Double male USB data cable

There are two types of firmware files:

* A single unified firmware

	The unified firmware is a single file packaged and merged by all files such as the partition table, bootloader, uboot, kernel, system and so on. The firmware officially released by Firefly adopts a unified firmware format. Upgrading the unified firmware will update the data and partition table of all partitions on the motherboard, and erase all data on the motherboard.

* Multiple partition images

	That is, files with independent functions, such as partition table, bootloader, and kernel, are generated during the development phase. The independent partition image can only update the specified partition, while keeping other partition data from being destroyed, it will be very convenient to debug during the development process.

> Through the unified firmware unpacking / packing tool, the unified firmware can be unpacked into multiple partition images, or multiple partition images can be merged into a unified firmware.

In order to avoid the burning problem caused by the upgrade tool version, it is recommended to use the tool packaged inside the public firmware package for burning. After decompressing the public firmware package, it is as follows:
```shell
XXXX_Android11_HDMI_XXXX
├── XXXX_Android11_HDMI_XXXX.img
├── linux
│   └── Linux_Upgrade_Tool_v1.59.zip
└── windows
    ├── DriverAssitant_v5.1.1.zip
    └── RKDevTool_Release_v2.81.zip
```

### Windows

* Tool: Use tools in the firmware package or download here [Androidtool_xxx (version number)]()

AndroidTool defaults to display in Chinese. We need to change it to English. Open `config.ini` with an text editor (like notepad). The starting lines are:

```
#Language Selection: Selected=1(Chinese); Selected=2(English)
[Language]
Kinds=2
Selected=1
LangPath=Language\
```

Change `Selected=1` to `Selected=2`, and save. From now on, AndroidTool will display in English.Now, run AndroidTool.exe: (Note: If using Windows 7/8, you’ll need to right click it, select to run it as Administrator)

![](../../../rk356x_img/upgrade_firmware_androidtool_zh.png)

#### Install RK USB drive

Download [Release_DriverAssistant.zip](), extract, and then run the DriverInstall.exe inside .
In order for all devices to use the updated driver, first select `Driver uninstall`(`驱动卸载`) and then select `Driver install`(`驱动安装`).

<center>

![](../../../rk356x_img/upgrade_firmware_install_RK_USB.png)
</center>

#### Connect devices

we can put the device into upgrade mode by hardware as follows:


put the device into upgrade mode by software as follows:

Double male USB data cable is connected, use the command in the serial debugging terminal or adb shell

```shell
reboot loader
```


The host should prompt for new hardware and configure the driver. Open Device manager and you will see the new Device `Rockusb Device` appear as shown below. If not, you need to go back to the previous step and [reinstall the driver](03-upgrade_firmware.html#install-rk-usb-drive).

![](../../../rk356x_img/upgrade_firmware_new_equipment.png)

### Linux

There is no need to install device driver under Linux. Please refer to the Windows section to connect the device.

* Tool : Use tools in the firmware package or download here [upgrade_tool_xxx (version number)]()
* Tool : [Linux_adb_fastboot]


#### Upgrade_tool

Install it into the system as follows for easy invocation:

```
unzip Linux_Upgrade_Tool_xxxx.zip
cd Linux_UpgradeTool_xxxx
sudo mv upgrade_tool /usr/local/bin
sudo chown root:root /usr/local/bin/upgrade_tool
sudo chmod a+x /usr/local/bin/upgrade_tool
```

## Upgrade the firmware


Notice：**Linux SDK v1.2.4a** and later using extboot, please use extboot.img instead of boot.img in the following instructions（Linux only, ignore it if using Android）

How to check SDK version:
1. The version format is vx.x.xx, eg: v1.2.4a
1. Firmware filename has SDK version(..._vx.x.xx_date.img)
1. Buildroot use`cat /etc/version`to get version(rk356x_linux_release_date_vx.x.xx.xml)
1. Ubuntu use`ffgo version`to get(rk356x_linux_release_date_vx.x.xx.xml)
1. In SDK check the link:`ls -l .repo/manifests/rk356x_linux_release.xml`
1. If you can't get version by methods above, that means you are using old version, no support for extboot

**Do not burn extboot.img into old version firmware!**

Besides, extboot ubuntu support update kernel by deb package, please see [Ubuntu Manual](https://wiki.t-firefly.com/en/Firefly-Linux-Guide/manual_ubuntu.html#linux-headers-and-linux-image)

### Windows

#### Upgrade unified firmware - update.img

The steps to update the unified firmware `update.img` are as follows:

1. Switch to the "upgrade firmware" page.
2. Press the "firmware" button to open the firmware file to be upgraded. The upgrade tool displays detailed firmware information.
3. Press the "upgrade" button to start the upgrade.
4. If the upgrade fails, you can try methods in [Switching Upgrade Storage](03-upgrade_firmware_with_flash.html#download-to-emmc)

#### Upgrade Partition image
The steps to upgrade the partition image are as follows:
1. Switch to the "download image" page.

2. Click `Dev Partition`

3. Check the partition to be burned, and select multiple.

4. Make sure the path of the image file is correct. If necessary, click the blank table cell on the right side of the path to select it again.

5. Click "Run" button to start the upgrade, and the device will restart automatically after the upgrade.

![](../../../rk356x_img/upgrade_firmware_androidtool_zh.png)

### Linux

#### Upgrade unified firmware - update.img

```
sudo upgrade_tool uf update.img
```

If the upgrade fails, you can try methods in [Switching Upgrade Storage](03-upgrade_firmware_with_flash.html#download-to-emmc)

#### Upgrade Partition image

```
sudo upgrade_tool di -b /path/to/boot.img
sudo upgrade_tool di -r /path/to/recovery.img
sudo upgrade_tool di -m /path/to/misc.img
sudo upgrade_tool di -u /path/to/uboot.img
sudo upgrade_tool di -dtbo /path/to/dtbo.img
sudo upgrade_tool di -p paramater   #upgrade parameter
sudo upgrade_tool ul bootloader.bin #upgrade bootloader
```


#### Android fastboot

Download [Linux_adb_fastboot](http://en.t-firefly.com/doc/download/108.html#other_536), And according to the following method to install into the system, easy to call：

```
sudo mv adb /usr/local/bin
sudo chown root:root /usr/local/bin/adb
sudo chmod a+x /usr/local/bin/adb
```
```
sudo mv fastboot /usr/local/bin
sudo chown root:root /usr/local/bin/fastboot
sudo chmod a+x /usr/local/bin/fastboot
```

**fastboot Burn dynamic partitions**

```
adb reboot fastboot # enter bootloader
sudo fastboot flash vendor vendor.img
sudo fastboot flash system system.img
sudo fastboot reboot # After the burn is successful, restart
```


## FAQs

### 1. How to forcibly enter MaskRom mode

**A1 :** If the board does not enter Loader mode, you can try to force your way into MaskRom mode. See operation method ["How to enter MaskRom mode"](04-maskrom_mode.md).




### 2. Analysis of programming failure

If Download Boot Fail occurs during the programming process, or an error occurs during the programming process, as shown in the figure below, it is usually caused by the poor connection of the USB cable, the inferior cable, or the insufficient drive capability of the USB port of the computer. Troubleshoot the computer USB port.

![](../../../rk356x_img/upgrade_downloadfail.png)

### 3. Has Spi Flash(Nor Flash)，After MaskRom，Download failure
If board has Spi Flash and eMMC，After MaskRom，need select Storage，See operation method [ "Switching Upgrade Storage" ](03-upgrade_firmware_with_flash.md)。

[烧写须知]: 02-upgrade_table.md
[IPC-M10R800-A3568J firmware]: http://en.t-firefly.com/doc/download/108.html
[Androidtool_xxx (version number)]: http://en.t-firefly.com/doc/download/108.html#windows_12
[Release_DriverAssistant.zip]: http://en.t-firefly.com/doc/download/108.html#windows_341
[Linux_Upgrade_Tool]: http://en.t-firefly.com/doc/download/108.html#linux_12
[upgrade_tool_xxx (version number)]: http://en.t-firefly.com/doc/download/108.html#linux_12
