# User manual

## V1 hardware version

The  Face-RK3399 V1 includes the following accessories:

* Face-RK3399 main board ×1
* 12V-2A Power adapter ×1
* 20pin integrated transfer cable ×1

Other optional accessories:

*   Firefly Serial Module

In addition, you may need the following accessories during use:

*   Display device
      - Monitor or TV with MIPI-DSI interface
*   Network
      - 100M Ethernet cable, and wired router
      - WiFi router
*   Firmware upgrade and debugging
      - Dual male USB data cable
      - Serial port to USB adapter

*   [Consulting mall customer service getting to know detailed shopping list.](https://www.firefly.store/goods.php?id=105)


## Integrated transfer cable connection 

Face-RK3399 is equipped with a 20pin integrated transfer cable, and the connection line sequence with the board is as follows:

![](../../../rk3399_img/Face-RK3399/weixian1.jpg)


## V2 hardware version

[Consulting mall customer service getting to know detailed shopping list.](https://www.firefly.store/goods.php?id=105)


 <a id="firmware-format"></a>

## Firmware type

Firmware comes in two formats:
 * Raw firmware
 * RK firmware(Rockchip firmware)

<a id="raw-firmware-format"></a>
    [Raw firmware] is a firmware that can be upgraded to a storage device in a bit-by-bit replication mode. It is the original image of the storage device. Raw firmware is generally upgraded to an SD card, but it can also be upgraded to an eMMC. There are many tools available to upgrade the original firmware:

<a id="rk-firmware-format"></a>
    [RK firmware] is a firmware packaged in Rockchip proprietary format and upgraded to eMMC flash memory using *upgrade_tool(Linux)* or *AndroidTool(Windows)* tools provided by Rockchip. RK firmware is Rockchip's traditional firmware packaging format and is often used on Android devices. In addition, Android's RK Firmware can be upgraded to an SD card using the [SD Firmware Tool].

<a id="partition-image"></a>
    [Partition image] is the image data of a partition and is used to store the upgrade of the corresponding partition of the device. For example, the compilation of the Android SDK will construct `boot.img`, ` kernel.img ` and ` system.img ` etc. Partition image file ,the `kernel.img` will be written in eMMC or SD card "the kernel" partition.

## Download and update firmware

The following is a list of supported systems:

- Android 7.1
- Ubuntu 16.04 
- Ubuntu 18.04
- Debian 9

Choose the right tool to upgrade the firmware according to the operating system used:

- upgrade to SD
  + Graphical interface upgrade tool:
	* [Etcher] (windows/linux/Mac)
  + Command line upgrade tool:
	* [dd] (Linux)
- upgrade to eMMC
  + Graphical interface upgrade tool:
	* [AndroidTool] (Windows)
  + Command line upgrade tool:
	* [upgrade_tool] (Linux)

## Boot

After confirming the correct connection of motherboard accessories, insert the power adapter into a live socket and the power line interface into the development board. The development board will automatically start up after the first power on. Maintain power supply of development board after shutdown, then we can boot Face-RK3399 by follow ways:

* Press the power button for 3 seconds (extension button)
 <a id="firmware-format"></a>