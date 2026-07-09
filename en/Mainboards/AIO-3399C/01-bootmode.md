# Boot mode description

## Preface


The AIO-3399C is installed with the Android operating system by default. If users want to run other operating systems, they need to use the corresponding firmware to program to the mainboard.

AIO-3399C has a flexible startup mode. Generally, the AIO-3399C development board will not turn brick unless the hardware is damaged.

If the accident appeared in the process of upgrading, bootloader damage, leading to unable to upgrade again, while still can enter `MaskRom` mode to repair.

## How to get the Firmwares

[Firmware download link](http://en.t-firefly.com/doc/download/52.html)

## Upgrade method

AIO-3399C supports firmware update through the following two methods:

* [Upgrade the firmware via USB cable](loader_mode.md)

Use the USB cable to connect the mainboard to the computer, and use the upgrade tool to program the firmware to the mainboard.

* [Upgrade the firmware via SD card](05-upgrade_firmware_sd.md)

Use the upgrade card creation tool to make the MicroSD card as an upgrade card, insert the upgrade card into the mainboard, power on, and the machine will automatically perform the upgrade.

## Boot media

* eMMC interface
* SDMMC interface

## Boot mode

AIO-3399C  has three startup modes:

* Normal mode
* Loader mode
* MaskRom mode

### Normal mode

Normal mode is the Normal startup process. Each component loads in turn and enters the system normally.

### Loader mode

In Loader mode, the bootloader will enter the upgrade state, waiting for the host command for firmware upgrade, etc. 

### MaskRom mode

MaskRom mode is used for system repair when the bootloader is damaged.

In general, there is no need to enter `MaskRom mode`. Only when the bootloader verification fails (the IDB block cannot be read, or the bootloader is damaged), the BootRom code will enter this mode. At this time, the BootRom code waits for the host to transmit the bootloader code through the USB interface, load and run it. When the board becomes bricked and cannot start or upgrade the program normally, you can also manually enter the `MaskRom mode`.

***To forcibly enter `MaskRom mode`, please refer to the chapter ["MaskRom mode"](04-maskrom_mode.md).***