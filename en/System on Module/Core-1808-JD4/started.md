
# User manual
## Accessories

AIO-1808-JD4 standard set includes the following accessories:
* AIO-1808-JD4 main board ×1
* brass antenna ×1  
* 12V-2A Power adapter ×1  
* Dual male usb data cable ×1

Other optional accessories :
* Firefly Serial Module 

In addition, you may need the following accessories during use:
*    Display device
     *   10.1 inch LVDS display module (***Note: In order to be compatible with other core boards, the HDMI interface on the backplane does not support the HDMI display function***)
*    Network
     *   100M/1000M Ethernet cable, and wired router
     *   WiFi router
*    Input devices
     *   USB wireless/wired mouse/keyboard
     *   infrared remote control (need to connect to the infrared receiver)
*    firmware upgrade and debugging
     *   Dual male usb data cable
     *   serial port to USB adapter.
*    Shipping list reference   
![](../../../rk1808_img/started1_en.jpg)

 <a id="firmware-format"></a>
## Firmware type

Firmware comes in two formats:
 * Original firmware(raw firmware)
 * RK firmware(Rockchip firmware)

<a id="raw-firmware-format"></a>
    [Original firmware][原始固件] is a firmware that can be upgraded to a storage device in a bit-by-bit replication mode. It is the original image of the storage device. Raw firmware is generally upgraded to an SD card, but it can also be upgraded to an eMMC. There are many tools available to upgrade the original firmware:
		
<a id="rk-firmware-format"></a>
    [RK firmware][RK固件] is a firmware packaged in Rockchip proprietary format and upgraded to eMMC flash memory using *upgrade_tool(Linux)* or *AndroidTool(Windows)* tools provided by Rockchip. RK firmware is Rockchip's traditional firmware packaging format and is often used on Android devices. In addition, Android's RK Firmware can be upgraded to an SD card using the SD Firmware Tool.

<a id="partition-image"></a>
    [Partition image][分区映像] is the image data of a partition and is used to store the upgrade of the corresponding partition of the device. For example, the compilation of the Android SDK will construct `boot.img`, ` kernel.img ` and ` system.img ` etc. Partition image file ,the `kernel.img` will be written in eMMC or SD card "the kernel" partition.

## Download and update firmware

The following is a list of supported systems:
- Buildroot
- Ubuntu 18.04

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
After confirming the correct connection of motherboard accessories, insert the power adapter into a live socket and the power line interface into the development board. The development board will automatically start up after the first power on. After the shutdown of the Android system is selected, the power supply of the development board is maintained. At this time,  AIO-1808-JD4 is as follows:
*  Long press the power button for 3 seconds (extension button)

When the machine is turned on, the blue power indicator light will be on.

[RK固件]:started.html#rk-firmware-format
[原始固件]:started.html#raw-firmware-format
[分区映像]:started.html#partition-image
[固件类型]:started.html#firmware-format
[SD Firmware Tool]: upgrade_firmware_rk.html#SD_Firmware_Tool
[Etcher]: upgrade_firmware_sd.html#Etcher
[dd]: upgrade_firmware_sd.html#dd
[AndroidTool]: upgrade_firmware.html#Androidtool
[upgrade_tool]: upgrade_firmware.html#upgrade_and_upgrade_tool
[rkdeveloptool]: upgrade_firmware.html#upgrade_and_rkdeveloptoo