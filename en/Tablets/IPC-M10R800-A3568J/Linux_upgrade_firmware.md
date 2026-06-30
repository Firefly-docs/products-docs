[Linux_adb_fastboot]: http://en.t-firefly.com/doc/download/108.html#other_536
[Androidtool_xxx (version number)]: http://en.t-firefly.com/doc/download/108.html#other_365
[Release_DriverAssistant.zip]: http://en.t-firefly.com/doc/download/108.html#other_366
[Linux_Upgrade_Tool]: http://en.t-firefly.com/doc/download/108.html#other_367
# Upgrading firmware on Linux

## Preface

This article describes how to burn the firmware file on the host computer to the flash memory of the IPC-M10R800-A3568J development board through 双公头USB数据线. When upgrading, you need to choose an appropriate upgrade method according to the host operating system and firmware type.

## Ready to work

* IPC-M10R800-A3568J board
* firmware
* host
* Good Double male USB data cable data cable


## Firmware file
There are two types of firmware files:

* A single unified firmware

	The unified firmware is a single file packaged and merged by all files such as the partition table, bootloader, uboot, kernel, system and so on. The firmware officially released by Firefly adopts a unified firmware format. Upgrading the unified firmware will update the data and partition table of all partitions on the motherboard, and erase all data on the motherboard.

* Multiple partition images

	That is, files with independent functions, such as partition table, bootloader, and kernel, are generated during the development phase. The independent partition image can only update the specified partition, while keeping other partition data from being destroyed, it will be very convenient to debug during the development process.

> Through the unified firmware unpacking / packing tool, the unified firmware can be unpacked into multiple partition images, or multiple partition images can be merged into a unified firmware.

## Flash on linux

There is no need to install device driver under Linux. Please refer to the Windows section to connect the device.

* 工具:[Linux_Upgrade_Tool]

### upgrade_tool

### Install the programming tool

* Download [Linux_Upgrade_Tool], And install it into the system as follows for easy invocation:

	```
	unzip Linux_Upgrade_Tool_xxxx.zip
	cd Linux_UpgradeTool_xxxx
	sudo mv upgrade_tool /usr/local/bin
	sudo chown root:root /usr/local/bin/upgrade_tool
	sudo chmod a+x /usr/local/bin/upgrade_tool
	```

* [Linux_adb_fastboot] tool

    Download [Linux_adb_fastboot], and install it into the system as follows for easy invocation:

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

Upgrade the unified firmware update.img:

```
sudo upgrade_tool uf update.img
```

<font color=#ff0000 >If the upgrade fails, try erasing before upgrading.</font>

```
# erase flash : Using the ef parameter requires the loader file or the corresponding update.img to be specified.
# update.img :The ubuntu firmware you need to upgrade.
sudo upgrade_tool ef update.img
# upgrade again
sudo upgrade_tool uf update.img
```

**Upgrade Partition image**

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

fastboot Burn dynamic partitions
```
adb reboot fastboot # enter bootloader
sudo fastboot flash vendor vendor.img
sudo fastboot flash system system.img
sudo fastboot reboot # After the burn is successful, restart
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

![](../../../rk356x_img/upgrade_downloadfail.jpg)