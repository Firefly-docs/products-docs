# Boot mode description

## Preface

The AIO-3288J is installed with the Android operating system by default. If users want to run other operating systems, they need to use the corresponding firmware to program to the motherboard.

AIO-3288J has a flexible startup mode. Generally, the AIO-3288J development board does not brick unless the hardware is damaged.

If the accident appeared in the process of upgrading, bootloader damage, leading to unable to upgrade again, while still can enter ` MaskRom ` mode to repair.

## Firmware information

* [Firmware update instructions](https://github.com/T-Firefly/firmware_doc)
* [Firmware download link](http://en.t-firefly.com/doc/download/page/id/16.html)

## Upgrade method

AIO-3288J supports firmware update through the following two methods:

* Update firmware using USB cable

    Use the USB cable to connect the motherboard to the computer, and use the upgrade tool to program the firmware to the motherboard.

* Use MicroSD card

    Use the upgrade card creation tool to make the MicroSD card as an upgrade card, insert the upgrade card into the motherboard, power on, and the machine will automatically perform the upgrade.

## boot media

* eMMC interface
* SDMMC interface

## Boot mode

AIO-3288J has three startup modes:

* Normal mode
* Loader mode
* MaskRom mode

### Normal mode

Normal mode is the Normal startup process. Each component loads in turn and enters the system normally.

### Loader mode

In Loader mode, the bootloader will enter the upgrade state, waiting for the host command for firmware upgrade, etc. To enter the Loader mode, the bootloader must detect that the `RECOVERY` key has been pressed and the USB is connected.

The method to put the device into upgrade mode is as follows:

!INCLUDE "upgrade_firmware_enter_loader.mdpp"

### MaskRom mode

MaskRom mode is used for system repair when the bootloader is damaged.

Under normal circumstances, it is not necessary to enter MaskRom mode. Only when the bootloader verification fails (IDR block cannot be read, or the bootloader is damaged), the BootRom code will enter MaskRom mode. At this time, the BootRom code waits for the host to transmit the bootloader code through the USB interface, load and run it.

***Forcibly enter MaskRom mode, please refer to the chapter [MaskRom Mode](maskrom_mode.md).***