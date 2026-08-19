# Firmware upgrade

## Introduction

This page describes how to flash the image file from the host to the development board's flash memory, via the Dual male USB data cable.  
Please choose the right way according to the host OS and image file type.

## Ready to work

What you need:

* [AIO-3128C board](https://www.firefly.store/products/aio-3128c-quad-core-high-performance-board)
* [Firmware](https://community.t-firefly.com/en/doc/download/37)
* Dual male USB data cable

Typically firmware files comprise two kinds:

* Monolithic image, often known as update.img, which contains the bootloader, parameter and all the partition image files. It is used for firmware distribution.
* Multiple partition image, like kernel.img, boot.img, recovery.img, etc, which are created during development.

You can download compiled update.img, or you can reference Build Android to compile your own image file.  
Supported host OS:

* Windows XP （32/64bit)
* Windows 7 (32/64bit)
* Windows 8 (32/64bit)
* Linux (32/64bit)

## Flash on windows

We used to need two utilities to flash rockchip image file:  

* RKBatchTool, used to flash update.img
* RKDevelopTool, used to flash partition image file separately.

Later, Rockchip release new tool: AndroidTool. It is based on RKDevelopTool, but adds flashing support for update.img. Therefore, one utility is sufficient.  
Before using the flashing utility, you need to install RK USB driver. If the driver is already installed, you can skip to next step.  

### RK USB driver installation

Download [Release_DriverAssistant.zip](https://community.t-firefly.com/en/doc/download/37), uncompress it, then run DriverInstall.exe inside.  
In order to use new driver for all the rockchip devices, please select "驱动卸载"(Driver uninstall), then "驱动安装"(Driver install).  

![](../../../rk3128_img/AIO-3128C/win_tool_devices.png)

### Devices connection

There are two ways to put the device into upgrade mode.  
The first one, device is cut off all the power sources, such as power adapter and Dual male USB data cabl connection:  

* Keep Dual male USB data cabl connected with host PC.
* Press and hold RECOVERY key.
* Connect Dual male USB data cable to the device.
* After around two seconds, release RECOVERY key.

The other way:

* Use Dual male USB data cabl to connect host and device together.
* Press and hold RECOVERY key.
* Shortly press RESET key.
* After around two seconds, release RECOVERY key.

RECOVERY button and RESET button and OTG interface as shown:

![](../../../rk3128_img/AIO-3128C/loader.png)

When programming the firmware, you should use the dual male USB data cable. The connection interface is as shown:

![](../../../rk3128_img/AIO-3128C/AIO-3128C-OTG&USB.jpg)

The host will prompt to have new device detected and configured. Open the Device Management, you'll find a new device name "Rockusb Device", as shown below. Return to previous step to reinstall driver if it is not shown.

![](../../../rk3128_img/AIO-3128C/win_rockusb_driver.png)

### Firmware burning

Download [AndroidTool_Release_v2.35.rar](https://community.t-firefly.com/en/doc/download/37). Uncompress it and change to directory AndroidTool_Release_v2.35.  
 
Now, run AndroidTool.exe: (Note: If using Windows 7/8, you'll need to right click it, select to run it as Administrator)

![](../../../rk3128_img/AIO-3128C/win_3128_tool_download.png)

#### Burn the unified firmware update.img

Steps of flashing update.img:  

* Switch to "Upgrade Firmware" tab page.
* Click "Firmware" button and open the image file. Detail information of the image file, like version and chip, is shown.
* Click "Upgrade" button to start flash.
* If upgrade fails, please try "LowerFormat" in the "Download Image" tab page first, then try again.

***WARNING: If you flash firmware laoder different version of the original machine, please click "Erase Flash" before upgrading the firmware.***

![](../../../rk3128_img/AIO-3128C/win_3128_tool_upgrade.png)

#### Burn partition image

Steps of flashing partition images:

* Switch to "Download Image" tab page.
* Check the partitions you want.
* Make sure the image file's path is correct. Click the rightmost empty table cell to select new path if needed.
* Click "Run" button to start flashing. Device will reboot automatically when finish.

![](../../../rk3128_img/AIO-3128C/win_3128_tool_download.png)

## Flash on linux

Rockchip provides a command line utility named "upgrade_tool" under Linux, which support flashing of both update.img and partition images.  
We have two choices with regard to open source tools:

* rkflashtool [https://github.com/Galland/rkflashtool_rk3066](https://github.com/Galland/rkflashtool_rk3066)
* rkflashkit [https://github.com/linuxerwang/rkflashkit]( https://github.com/linuxerwang/rkflashkit)

Both of them only support flashing partition images, not update.img. rkflashtool is a command line tool, and flashkit has a nice and easy to use GUI with command line support lately added. We only introduce rkflashkit below.  
There is no need to install device driver. Just connect the device and host as described in the Windows section.  

### upgrade_tool

Download [Linux_Upgrade_Tool](https://community.t-firefly.com/en/doc/download/37), and install it to host filesystem:

```
unzip Linux_Upgrade_Tool_v1.21.zip
cd Linux_UpgradeTool_v1.21 
sudo mv upgrade_tool /usr/local/bin
sudo chown root:root /usr/local/bin/upgrade_tool
```

Burn unified firmware update.img:

```
sudo upgrade_tool uf update.img
```

Burn partition image:

```
sudo upgrade_tool di -b /path/to/boot.img
sudo upgrade_tool di -k /path/to/kernel.img
sudo upgrade_tool di -s /path/to/system.img
sudo upgrade_tool di -r /path/to/recovery.img
sudo upgrade_tool di -m /path/to/misc.img
sudo upgrade_tool di resource /path/to/resource.img
sudo upgrade_tool di -p paramater   # flash parameter
sudo upgrade_tool ul bootloader.bin # flash bootloader
```

If flash issue causes error while upgrading, you may try low-level formatting and erasing nand flash:

```
upgrade_tool lf   # low format flash
upgrade_tool ef   # erase flash
```

### rkflashkit

* Install:

```
sudo apt-get install build-essential fakeroot 
git clone https://github.com/linuxerwang/rkflashkit
cd rkflashkit
./waf debian
sudo apt-get install python-gtk2
sudo dpkg -i rkflashkit_0.1.4_all.deb
```

* Graphic interface:

![](../../../rk3128_img/AIO-3128C/Fireprime_rkflashkit.png)

* Command line:

```
$ rkflashkit --help
Usage: <cmd> [args] [<cmd> [args]...]

part                              List partition
flash @<PARTITION> <IMAGE FILE>   Flash partition with image file
cmp @<PARTITION> <IMAGE FILE>     Compare partition with image file
backup @<PARTITION> <IMAGE FILE>  Backup partition to image file
erase  @<PARTITION>               Erase partition
reboot                            Reboot device

For example, flash device with boot.img and kernel.img, then reboot:

sudo rkflashkit flash @boot boot.img @kernel.img kernel.img reboot
```

See the example above, which is really handy to flash multiple images then reboot device in one command, especially good for developers who will compile and flash kernel again and again.

## Common Problem

### How to enter MaskRom mode

If the board cannot enter Loader mode, the SD card fails to start, and you can try to enter MaskRom mode. Please refer to [《How to enter MaskRom mode》](maskrom_mode.md)。
