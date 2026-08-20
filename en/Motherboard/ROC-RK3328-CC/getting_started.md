# Getting Started

[ROC-RK3328-CC] supports booting from the following storage devices:

- SD card
- eMMC

You need to flash the firmware file to the SD card or eMMC, in order to make the board bootable.

[SDCard Installer] is the officially recommended SD card flashing tool, which is derived from Etcher / Rock64 Installer. It implements one-stop downloading and flashing of firmware file, which makes life easy.

## Firmware Format

There are two firmware file formats:

- Raw Firmware
- RK Firmware

<a id="raw-firmware-format"></a>

[Raw Firmware], when flashing, is copied to the storage device bit by bit. It is the raw image of the storage device. [Raw Firmware] is flashed to the SD card in common cases, but it can also be flashed to the eMMC. There are lots of flashing tools available:

- To flash to the SD card:
    + GUI:
        * [SDCard Installer] (Linux/Windows/Mac)
        * [Etcher] (Linux/Windows/Mac)
    + CLI:
        * [dd] (Linux)
- To flash to the eMMC:
    + GUI:
        * [AndroidTool] (Windows)
    + CLI:
        * [upgrade_tool] (Linux)
        * [rkdeveloptool] (Linux)

<a id="rk-firmware-format"></a>

[RK Firmware], is packed in Rockchip's proprietary format, it can be flashed into eMMC or SD Card with the tools provided by Rockchip. There are some flashing tools availble:

- To flash to the SD card:
    + GUI:
        * [SD Firmware Tool] (Windows)
- To flash to the eMMC:
    + GUI:
        * [AndroidTool] (Windows)
    + CLI:
        * [upgrade_tool] (Linux)

<a id="partition-image"></a>

[Partition Image], is the raw image of the partition. When you build the Android SDK, you'll get a list of `boot.img`, `kernel.img`, `system.img`, etc, which is called [Partition Image] and will be flashed to the corresponding partition. For example, `kernel.img` is to be flashed to `kernel` partition in the eMMC or SD card.

## Download & Flash

Here's the available OS list of firmware:

- Android 7.1.2
- Android 8.1.0
- Ubuntu 16.04
- Ubuntu 18.04
- Debian 9
- LibreELEC 9.0

Then choose the flashing tool according to your host PC's OS, storage devices, Firmware Format:

- To flash to the SD card: Refer to [《Flashing to the SD Card》](flash_sd.md)

- To flash to the eMMC: Refer to [《Flashing to the eMMC》](flash_emmc.md)

## System Boot Up

Before system boots up, make sure you have:

- A bootable SD card or eMMC
- 5V2A power adapter
- Micro USB cable

Then follow the procedures below:

1. Pull the power adapter out of the power socket.
2. Use the micro USB cable to connect the power adapter and the board.
3. Plug in the bootable SD card or eMMC (NOT BOTH).
4. Plug in the optional HDMI cable, USB mouse or keyboard.
5. Check everything is okay, then plug the power adapter into the power socket to power on the board.

[Getting Started]: getting_started.md
[FAQ]: faqs.md
[Serial Debug]: debug.md
[Building Linux Root Filesystem]: linux_build_ubuntu_rootfs.md
[Contact]: resource.md#community
[Raw Firmware]: getting_started.md#raw-firmware-format
[RK Firmware]: getting_started.md#rk-firmware-format
[Partition Image]: getting_started.md#partition-image
[SDCard Installer]: flash_sd.md#sdcard-installer
[Etcher]: flash_sd.md#etcher
[dd]: flash_sd.md#dd
[SD Firmware Tool]: flash_sd.md#sd-firmware-tool
[AndroidTool]: flash_emmc.md#androidtool
[upgrade_tool]: flash_emmc.md#upgrade-tool
[rkdeveloptool]: flash_emmc.md#rkdeveloptool
[Rockusb Mode]: flash_emmc.md#rockusb-mode
[Maskrom Mode]: flash_emmc.md#maskrom-mode
[Rockusb Driver]: flash_emmc.md#rockusb-driver
[ROC-RK3328-CC]: http://en.t-firefly.com/product/rocrk3328cc.html "ROC-RK3328-CC Official Website"
[Download Page]: https://community.t-firefly.com/en/doc/download/34
[Forum]: http://bbs.t-firefly.com/
[Facebook]: https://www.facebook.com/TeeFirefly
[Google+]: https://plus.google.com/u/0/communities/115232561394327947761
[Youtube]: https://www.youtube.com/channel/UCk7odZvUrTG0on8HXnBT7gA
[Twitter]: https://twitter.com/TeeFirefly
[USB Serial Adapter]: https://www.firefly.store/products/usb-to-uart-module-cp2104 
[5V2A US Adapter]: https://www.firefly.store/products/5v2a-us-adapter-3c-fcc-ce 
[eMMC Flash]: https://www.firefly.store/products
[Storage Map]: http://opensource.rock-chips.com/wiki_Partitions#Default_storage_map