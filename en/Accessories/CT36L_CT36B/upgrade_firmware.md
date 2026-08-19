# Upgrade the firmware via USB cable

## Preface

This article describes how to burn the firmware on the host computer into the memory of the CT36L development board through USB data cable. When upgrading, you need to choose the appropriate upgrade method based on the host operating system and firmware type.

## Preparation tools
* CT36L/CT36B
* [Firmware](https://community.t-firefly.com/en/doc/download/214.html)
* Host
* Good USB data cable

## Prepare firmware
The firmware can be obtained by compiling the SDK, or you can download the public version firmware (unified firmware) from [Resource Download](https://community.t-firefly.com/en/doc/download/214.html). There are generally two types of firmware files:

* Single unified firmware

     Unified firmware is a single file that is packaged and merged from all files such as partition table, bootloader, uboot, kernel, system, etc. The firmware officially released by Firefly adopts a unified firmware format. Upgrading the unified firmware will update the data and partition table of all partitions on the motherboard, and erase all data on the motherboard.

* Multiple partition mirrors

     That is, files with independent functions, such as partition table, bootloader, kernel, etc., are generated during the development stage. Independent partition mirroring can only update the specified partition while keeping the data in other partitions from being destroyed, which is very convenient for debugging during the development process.

> Through the unified firmware unpacking/packaging tool, the unified firmware can be unpacked into multiple partition images, or multiple partition images can be merged into a unified firmware.



## Install the programming tool

### Windows operating system

* Install RK USB driver

Download [Release_DriverAssistant.zip](https://community.t-firefly.com/en/doc/download/214.html#other_663), unzip it, and then run DriverInstall.exe inside. In order for all devices to use updated drivers, please select `Driver Uninstall` first, and then select `Driver Installation`.

![](../../../rv1106_img/common/upgrade_firmware_install_rk_usb.jpg)

* Run SocToolKit's SocToolKit.exe



![](../../../rv1106_img/CT36L/SocToolKit_upgrade_update-img-1.png)

### Linux operating system
No need to install device driver under Linux
* [Linux_Upgrade_Tool](https://community.t-firefly.com/en/doc/download/214.html#other_665) tool

Download [Linux_Upgrade_Tool](https://community.t-firefly.com/en/doc/download/214.html#other_665), and install it into the system as follows for easy calling:

```
cd Linux_Upgrade_Tool
cp 88-rockusb.rules /etc/udev/rules.d/

# restart udev command:
sudo udevadm control --reload-rules
sudo service udev restart

sudo cp upgrade_tool /usr/local/bin
sudo chown root:root /usr/local/bin/upgrade_tool
sudo chmod a+x /usr/local/bin/upgrade_tool
```

## Enter upgrade mode
Usually we upgrade the firmware in MaskRom mode. Before burning the firmware, we need to connect the device and put the board into upgradeable mode.

### MaskRom Mode
For how to enter MaskRom mode, please refer to [MaskRom Mode](upgrade_maskrom_mode.md)


## Flash firmware

* **<font color=#ff0000 >Note: Burning firmware requires disassembling the machine casing</font>**

### windows operating system

#### Burn unified firmware update.img
Note: SocToolKit tool version 1.8 or higher is required to support the update.img burning function.

The steps to burn unified firmware update.img are as follows:

1. The device enters MaskRom mode.
2. Select RV1106 chip platform.
3. Click the Download button in the upper left corner to switch to the Download page.
4. Select the burning method, which supports serial port burning and USB burning. It is recommended to use USB burning.
5. Select the update.img unified firmware file.
6. Click the `Upgrade` button in the lower right corner to start the upgrade.

![](../../../rv1106_img/CT36L/SocToolKit_upgrade_update-img-1.png)
![](../../../rv1106_img/CT36L/SocToolKit_upgrade_update-img-2.png)

#### Burn partition image
The steps to burn a partition image are as follows:

1. The device enters MaskRom mode.
2. Select RV1106 chip platform.
3. Click the Download button in the upper left corner to switch to the Download page.
4. Select the burning method, which supports serial port burning and USB burning. It is recommended to use USB burning.
5. Select the corresponding partition firmware file.
6. Click the `Download` button in the lower right corner to start the upgrade.

![](../../../rv1106_img/CT36L/Partition_writing-1.png)
![](../../../rv1106_img/CT36L/Partition_writing-2.png)

### Linux operating system

#### Burn unified firmware update.img

There is no tool on Linux to flash unified firmware. However, partition image burning can be performed through scripts, and the burning effect is the same as unified firmware. Please refer to the partition image burning method described below.

#### Burn partition image

Use the rkdownload.sh script in the Linux_Upgrade_Tool directory to burn the partition image:

For example, if the partition image is placed in the output/image directory, check the partition image in the output/image compilation output directory.
```shell
ls output/image/
boot.img download.bin env.img idblock.img oem.img rootfs.img sd_update.txt tftp_update.txt uboot.img update.img userdata.img
```

Copy the rkdownload.sh script to the /usr/local/bin directory for global calling
```shell
sudo cp rkdownload.sh /usr/local/bin/
```

Specify the partition image in the output/image directory for burning
```shell
sudo rkdownload.sh -d output/image
```

If an error occurs during the upgrade due to flash problems, you can try low-level formatting and erasing operations:
```bash
sudo upgrade_tool ef output/image/download.bin
```
Wait for the formatting and erasing operations to be completed before trying to flash the image.

## Common problem
### How to force into MaskRom mode

For the operation method, see ["MaskRom Mode"] (upgrade_maskrom_mode.md).

### Analysis of programming failure

If Download Boot Fail occurs during the programming process, or an error occurs during the programming process, it is usually caused by poor connection of the USB cable used, inferior wire material, or insufficient driver capability of the computer USB port. Please replace the USB cable or computer USB port to troubleshoot.

[Androidtool_xxx(version number)]: http://www.t-firefly.com/share/index/index/id/2ea171f2235fe841e89734ca5189da8b.
[AndroidTool]: http://www.t-firefly.com/share/index/index/id/2ea171f2235fe841e89734ca5189da8b.html
[Release_DriverAssistant.zip]: http://www.t-firefly.com/share/index/index/id/1f98ebd663ed09a32e9ebf3fa893dfc0.html
[Linux_Upgrade_Tool]: http://www.t-firefly.com/share/index/index/id/f756718dd2bbf82eb405926549e75ef3.html
[Linux_adb_fastboot]: http://www.t-firefly.com/share/index/index/id/c64b7d743d9368de521a6ced87813dc5.html
