# Upgrade the firmware via USB cable

## Introduction

This article describes how to upgrade the firmware file on the host to the flash memory of the development board through the Type-C data cable. When upgrading, you need to choose the appropriate upgrade mode according to the host operating system and firmware type.

## Preparatory Tools

* ROC-RK3566-PC development board
* Firmware
* host computer
* Type-C data cable

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

<center>

<img alt="" src="../../../rk356x_img/upgrade_firmware_androidtool_zh.png" width="800">
</center>

#### Install RK USB drive

Download [Release_DriverAssistant.zip](), extract, and then run the DriverInstall.exe inside .
In order for all devices to use the updated driver, first select `Driver uninstall`(`驱动卸载`) and then select `Driver install`(`驱动安装`).

<center>

![](../../../rk356x_img/upgrade_firmware_install_RK_USB.png)
</center>

#### Connect devices

we can put the device into upgrade mode by hardware as follows:

1. Disconnect the device from the power supply
2. Press and hold the earphone hole on Station-M2 with a pin tool
3. The device is plugged into the power supply and powered on

The earphone hole position is shown in the figure below:

<center>

<img alt="" src="../../../rk3588_img/common/upgrade_hole.png" width="150">
</center>

At this point, the device enters Recovery mode.

<center>

<img alt="" src="../../../rk3588_img/common/upgrade_maskrom_zh.png" width="800">
</center>
put the device into upgrade mode by software as follows:

Type-C data cable is connected, use the command in the serial debugging terminal or adb shell

```shell
reboot loader
```


The host should prompt for new hardware and configure the driver. Open Device manager and you will see the new Device `Rockusb Device` appear as shown below. If not, you need to go back to the previous step and [reinstall the driver](03-upgrade_firmware.md).

<center>

<img alt="" src="../../../rk356x_img/upgrade_firmware_new_equipment.png" width="800">
</center>

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

## MaskRom Mode
`MasRrom` mode is the last line of defense against device being bricked. Forced entry `MaskRom` involved hardware operation, have certain risk, so only in the situation that deivce failed entering the `Loader` mode, you can try `MaskRom` mode.

**Please read carefully and operate carefully!**

The operation steps are as follows:

1. Disconnect all power supplies.
1. Unplug the SD card.
1. Connect the device and host PC with Type-C data cable.
1. Use metal a tweezer to short and hold the two test points as shown in the following figure on Station-M2 (as shown in the figure below).
1. Plug in the power supply.
1. Wait a few seconds, stop shorting.

<center>

<img alt="" src="../../../rk356x_img/Station-M2/maskrom_test_points.jpg" width="700">
</center>

At this point, the device should go into `MaskRom mode`.

<center>

<img alt="" src="../../../rk356x_img/maskrom_zh.png" width="700">
</center>


1. Disconnect the device from the power supply
2. Press and hold the MaskRom hole on Station-M2 with a pin tool
3. The device is plugged into the power supply and powered on
The device will then enter MaskRom mode.

The MaskRom hole position is shown in the figure below:

<center>

<img alt="" src="../../../rk3588_img/common/upgrade_hole.png" width="200">
</center>


## Upgrade the firmware


### Windows

#### Upgrade unified firmware - update.img

The steps to update the unified firmware `update.img` are as follows:

1. Switch to the "upgrade firmware" page.
2. Press the "firmware" button to open the firmware file to be upgraded. The upgrade tool displays detailed firmware information.
3. Press the "upgrade" button to start the upgrade.
4. If the upgrade fails, you can try methods in [Switching Upgrade Storage](03-upgrade_firmware_with_flash.md)

### Linux

#### Upgrade unified firmware - update.img

```
sudo upgrade_tool uf update.img
```
If the upgrade fails, you can try methods in [Switching Upgrade Storage](03-upgrade_firmware_with_flash.md)

[烧写须知]: 02-upgrade_table.md
[Station-M2 firmware]: https://community.t-firefly.com/en/doc/download/106
[Androidtool_xxx (version number)]: https://community.t-firefly.com/en/doc/download/106#windows_12
[Release_DriverAssistant.zip]: https://community.t-firefly.com/en/doc/download/106#windows_341
[Linux_Upgrade_Tool]: https://community.t-firefly.com/en/doc/download/106#linux_12
[upgrade_tool_xxx (version number)]: https://community.t-firefly.com/en/doc/download/106#linux_12
