# Boot mode description

## Preface

There is a flexible boot method for the AIO-RK3328-JD4. In general, it is impossible to brick for the AIO-RK3328-JD4 development board unless the hardware is damaged.

If an accident occurs during the upgrade, the bootloader is damaged and cannot be re-upgraded, at this time, you can enter the `MaskRom` mode to fix it.
## Loading mode

There is 20KB of BootRom and 36KB of internal SRAM for AIO-RK3328-JD4, which supports loading the system from the following devices:

* 8-bit Async Nand Flash
* 8-bit toggle Nand Flash
* SPI interface
* eMMC interface
* SDMMC interface

The AIO-RK3328-JD4 supports downloading the firmware from the Type-C interface.

## Boot order

The boot order is:

* Master control board power-up initialization
* Run the BootRom code on SRAM to verify the bootloader in the storage device
* After verification, load and run the bootloader boot code
* The bootloader boot code is responsible for initializing the DDR memory and loading the complete code of bootloader into DDR memory and running it
* The bootloader loads the Linux kernel on the storage device and passes the execution power to the Linux kernel

## Boot mode

There are three boot modes for the AIO-RK3328-JD4: 
* Normal mode
* Loader mode
* MaskRom mode

### Normal mode

Normal mode is the normal startup process, and each component is loaded in order to enter the system normally.

### Loader mode

In Loader mode, the bootloader goes into the upgrade state and waits for host commands for firmware upgrades and so on.   
If you want to enter the Loader mode,the bootloader must detect that the `RECOVERY` key is pressed and the USB is connected at startup. There are two ways to bring the device into the upgrade mode:

One way is to disconnect the power adapter

*  The USB Type-C cable is connected to the device and the host.
* Press and hold the `RECOVERY` key on the device.
* Plug in the power supply
* After approximately two seconds, release the `RECOVERY` key.

The other way is to plug in the power adapter

* Connect the device and the host with a USB Type-C cable.
* Press and hold the `RECOVERY` key on the device.
* Short press the `RESET` key.
* After approximately two seconds, release the `RECOVERY` key.

### MaskRom mode

The MaskRom mode is used for system recovery when the bootloader is damaged.

In general, the BootRom code will not enter the MaskRom mode, and only if bootloader validation fails (the IDR is not readable or the bootloader is damaged), the BootRom code will enter the MaskRom mode . At this time, the BootRom code waits for the host to pass the bootloader code through the USB interface, and load and run the code.   

***If you want to forcibly enter the MaskRom mode, please refer to the chapter [MaskRom].***

[Getting Started]: started.md
[FAQ]: faqs.md
[Serial Debug]: debug.md
[Building Linux Root Filesystem]: linux_build_ubuntu_rootfs.md
[ADB introduction]: adb_use.md
[Contact]: resource.md#community
[RK Firmware]: 02-upgrade_table.md#rk-firmware-format
[Compile Linux Firmware]:linux_compile_gpt.md
[Compile Android Firmware]:compile_android8.1_firmware.md
[MaskRom]:04-maskrom_mode.md
[Flashing Notes]:02-upgrade_table.md
[Boot Mode]:01-bootmode.md
[ROC-RK3328-PC]: http://en.t-firefly.com/product/rocrk3328pc.html "ROC-RK3328-PC Official Website"
[Download Page]: http://en.t-firefly.com/doc/download/34.html
[Forum]: http://bbs.t-firefly.com/
[Facebook]: https://www.facebook.com/TeeFirefly
[Google+]: https://plus.google.com/u/0/communities/115232561394327947761
[Youtube]: https://www.youtube.com/channel/UCk7odZvUrTG0on8HXnBT7gA
[Twitter]: https://twitter.com/TeeFirefly
[Shop]: http://shop.t-firefly.com/
[USB Serial Adapter]: https://www.firefly.store/products/usb-to-uart-module-cp2104 
[5V2A US Adapter]: https://www.firefly.store/products/5v2a-us-adapter-3c-fcc-ce
[eMMC Flash]: https://www.firefly.store/products
[Storage Map]: http://opensource.rock-chips.com/wiki_Partitions#Default_storage_map