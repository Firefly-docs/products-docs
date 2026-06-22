# Upgrade the firmware via USB cable

## Introduction

This article describes how to upgrade the firmware file on the host to the flash memory of the development board through the USB data cable. When upgrading, you need to choose the appropriate upgrade mode according to the host operating system and firmware type.

## Preparatory Tools

* AIO-1126BQ38 development board
* [Firmware](https://en.t-firefly.com/doc/download/377.html)
* host computer
* USB data cable

## Prepare Firmware

The firmware can be obtained by compiling the SDK, or you can download the public firmware (unified firmware) from the [Resource download](https://en.t-firefly.com/doc/download/377.html). There are two types of firmware files:

* A single unified firmware

	The unified firmware is a single file packaged and merged by all files such as the partition table, bootloader, uboot, kernel, system and so on. The firmware officially released by Firefly adopts a unified firmware format. Upgrading the unified firmware will update the data and partition table of all partitions on the motherboard, and erase all data on the motherboard.

* Multiple partition images

	That is, files with independent functions, such as partition table, bootloader, and kernel, are generated during the development phase. The independent partition image can only update the specified partition, while keeping other partition data from being destroyed, it will be very convenient to debug during the development process.

>    Through the unified firmware unpacking / packing tool, the unified firmware can be unpacked into multiple partition images, or multiple partition images can be merged into a unified firmware.

## Install the Upgrade Tool
### Windows Operating System

* Install RK USB driver

Download [Release_DriverAssistant.zip](https://en.t-firefly.com/doc/download/377.html#other_11), extract, and then run the DriverInstall.exe inside . 
In order for all devices to use the updated driver, first select Driver uninstall(驱动卸载) and then select Driver install(驱动安装).

<center>

![](../../../rv1126b_img/common/upgrade_firmware_install_rk_usb.jpg)
</center>

![](../../../rv1126b_img/common/upgrade_firmware_androidtool_zh.png)

### Linux Operating System

There is no need to install device driver under Linux.

* [Linux_Upgrade_Tool](https://en.t-firefly.com/doc/download/377.html#other_502)

Download [Linux_Upgrade_Tool](https://en.t-firefly.com/doc/download/377.html#other_502), And install it into the system as follows for easy invocation:

```
unzip Linux_Upgrade_Tool_xxxx.zip
cd Linux_UpgradeTool_xxxx
sudo mv upgrade_tool /usr/local/bin
sudo chown root:root /usr/local/bin/upgrade_tool
sudo chmod a+x /usr/local/bin/upgrade_tool
```

* [Linux_adb_fastboot](https://en.t-firefly.com/doc/download/140.html#other_565)

Download [Linux_adb_fastboot](https://en.t-firefly.com/doc/download/140.html#other_565), And install it into the system as follows for easy invocation:

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

## Enter Upgrade mode

Usually we upgrade firmware in two modes, namely `Loader` mode and `MaskRom` mode. Before we can write the firmware, we need to connect the device and put the board into upgradable mode.

### Maskrom mode

#### Hardware way into Maskrom mode

Connect the device and press the **MASKROM** button to enter the Loader mode. The steps are as follows:

* Disconnect the power adapter first
* Use USB data cable to connect one end to the host and the other end to the development board

![](../../../rv1126b_img/AIO-1126BQ38/upgrade_otg_interface.jpg)

* Press and hold the MASKROM button on the device
* The button position may vary slightly depending on the hardware version, please refer to the actual silk screen
* Connect the power
* After about two seconds, release the MASKROM button


#### Software way into Loader mode

USB data cable is connected, use the command in the serial debugging terminal or `adb shell`:

```shell
reboot loader
```

#### Check the Loader mode

How to determine whether the board is in Loader mode, we can check through the tool

**Windows Operating System**

The AndroidTool displays the prompt Found One LOADER Device at the bottom

![](../../../rv1126b_img/common/upgrade_firmware_androidtool_zh.png)

if the “Enter Loader mode” operation is performed, and the LOADER prompted by the upgrade tool is not displayed, check whether the Windows host computer you to discover new hardware and configure the driver. Open the Device manager and a new device `Rockusb Device` will appear, as shown below. If not, you can go back to the previous step to [reinstall the driver](upgrade_firmware.html#install-the-upgrade-tool).

![](../../../rv1126b_img/common/upgrade_firmware_new_equipment.jpg)

**Linux Operating System**

After running upgrade_tool, you can see a Loader prompt in the connected device:

```
firefly@T-chip:~/severdir/down_firmware$ sudo upgrade_tool
List of rockusb connected
DevNo=1 Vid=0x2207,Pid=0x330c,LocationID=106    Loader
Found 1 rockusb,Select input DevNo,Rescan press <R>,Quit press <Q>:q
```

### MaskRom mode

To enter MaskRom mode, please refer to [MaskRom mode](upgrade_maskrom_mode.md).

## Upgrade the firmware
### Windows Operating System
#### Upgrade unified firmware - update.img

The steps to update the unified firmware `update.img` are as follows:

1. Switch to the `Upgrade Firmware` page.
2. Press the `Firmware` button to open the firmware file to be upgraded. The upgrade tool displays detailed firmware information.
3. Press the `Upgrade` button to start the upgrade.
4. <font color=#ff0000 >If the upgrade fails, you can try to erase the Flash by pressing the `EraseFlash` button first, and then upgrade. </font>

![](../../../rv1126b_img/common/upgrade_firmware_erase_flash_zh.png)

#### Upgrade Partition image

The steps to upgrade the partition image are as follows:
1. Switch to the `Download Image` page.

2. Check the partition to be burned, and select multiple.

3. Make sure the path of the image file is correct. If necessary, click the blank table cell on the right side of the path to select it again.

4. Click `Run` button to start the upgrade, and the device will restart automatically after the upgrade.

![](../../../rv1126b_img/common/upgrade_firmware_androidtool_zh.png)

Tips for using the flashing tool:

1. Get device partition table information

	Put the rv1126 device into loader mode. Then click the Dev Partition button in the picture to get the device partition information.
   ![](../../../rv1126b_img/common/upgrade_tools_get_partition.png)

   Click Update to get the device partition information as follows:

   ![](../../../rv1126b_img/common/upgrade_tools_get_partition_ok.png)



2. Export device partition table information

   Right-click the mouse to pop up a menu, select export config to export the partition table configuration:

   ![](../../../rv1126b_img/common/upgrade_tools_export_config.png)



3. Load device partition table information

   Right-click the mouse to pop up a menu, select load config to load the partition table configuration:

   ![](../../../rv1126b_img/common/upgrade_tools_load_config.png)


### Linux Operating System

#### Upgrade unified firmware - update.img

```
sudo upgrade_tool uf update.img
```

If the upgrade fails, try erasing before upgrading. 

```
# erase flash : Using the ef parameter requires the loader file or the corresponding update.img to be specified.
# update.img :The ubuntu firmware you need to upgrade.
sudo upgrade_tool ef update.img
# upgrade again
sudo upgrade_tool uf update.img
```

#### Upgrade Partition image

View the SDK compilation output directory output. You can see that boot.img in this directory is linked to /path/to/sdk/kernel/boot.img.
```bash
lvsx@amd-epyc:~/project/rv1126b $ ll output/firmware/
total 12
drwxrwxr-x  2 lvsx lvsx 4096 Dec 25 17:40 ./
drwxrwxr-x 11 lvsx lvsx 4096 Dec 25 20:06 ../
lrwxrwxrwx  1 lvsx lvsx   25 Dec 25 17:35 boot.img -> ../../kernel-6.1/boot.img
lrwxrwxrwx  1 lvsx lvsx   45 Dec 25 15:51 MiniLoaderAll.bin -> ../../u-boot/rv1126b_spl_loader_v1.06.105.bin
lrwxrwxrwx  1 lvsx lvsx   11 Dec 25 15:50 misc.img -> ../misc.img
lrwxrwxrwx  1 lvsx lvsx   22 Dec 25 15:53 oem.img -> ../extra-parts/oem.img
lrwxrwxrwx  1 lvsx lvsx   61 Dec 25 15:53 parameter.txt -> ../../device/rockchip/.chips/rv1126b/parameter-debian-fit.txt
lrwxrwxrwx  1 lvsx lvsx   23 Dec 25 17:38 recovery.img -> ../recovery/ramboot.img
lrwxrwxrwx  1 lvsx lvsx   47 Dec 25 15:52 rootfs.img -> ../../prebuilt_rootfs/rv1126b_debian_rootfs.img
lrwxrwxrwx  1 lvsx lvsx   22 Dec 25 15:51 uboot.img -> ../../u-boot/uboot.img
lrwxrwxrwx  1 lvsx lvsx   26 Dec 25 17:40 update.img -> ../update/Image/update.img
lrwxrwxrwx  1 lvsx lvsx   27 Dec 25 15:53 userdata.img -> ../extra-parts/userdata.img
```

Therefore, the actual kernel firmware that needs to be flashed is boot.img. If it's in the output/firmware/ directory, execute the following command to flash the kernel firmware:
```bash
sudo upgrade_tool di -b /path/to/sdk/output/firmware/boot.img
```

If it is in the /path/to/sdk/kernel directory, you need to execute the following command to burn the kernel firmware:
```bash
sudo upgrade_tool di -b /path/to/sdk/kernel/boot.img
```

Commonly used programming commands are as follows:
```
sudo upgrade_tool di -b /path/to/boot.img
sudo upgrade_tool di -r /path/to/recovery.img
sudo upgrade_tool di -m /path/to/misc.img
sudo upgrade_tool di -u /path/to/uboot.img
sudo upgrade_tool di -p paramater   #upgrade parameter
sudo upgrade_tool ul bootloader.bin #upgrade bootloader
```

If the upgrade fails due to flash problems, you can try low-level formatting and erase nand flash:

```
sudo upgrade_tool lf update.img	# low-level formatting
sudo upgrade_tool ef update.img	# erase
```

fastboot Burn dynamic partitions:

```
adb reboot fastboot # enter bootloader
sudo fastboot flash vendor vendor.img
sudo fastboot flash system system.img
sudo fastboot reboot # After the burn is successful, restart
```


## FAQs

### 1. How to forcibly enter MaskRom mode

**A1 :** If the board does not enter Loader mode, you can try to force your way into MaskRom mode. See operation method ["How to enter MaskRom mode"](upgrade_maskrom_mode.md).


### 2. Analysis of programming failure

If Download Boot Fail occurs during the programming process, or an error occurs during the programming process, as shown in the figure below, it is usually caused by the poor connection of the USB cable, the inferior cable, or the insufficient drive capability of the USB port of the computer. Troubleshoot the computer USB port.

![](../../../rv1126b_img/common/upgrade_firmware_download_fail.png)

[烧写须知]: 02-upgrade_table.md
[AIO-1126BQ38 firmware]: https://en.t-firefly.com/doc/download/377.html
[Androidtool_xxx (version number)]: https://en.t-firefly.com/doc/download/377.html#windows_12
[Release_DriverAssistant.zip]: https://en.t-firefly.com/doc/download/377.html#windows_341
[Linux_Upgrade_Tool]: https://en.t-firefly.com/doc/download/377.html#linux_12
[upgrade_tool_xxx (version number)]: https://en.t-firefly.com/doc/download/377.html#linux_12
