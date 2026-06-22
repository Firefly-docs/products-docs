# Boot mode description

## Preface

The CAM-C1126S2U is installed with the Linux operating system by default. If users want to run other operating systems, they need to use the corresponding firmware to program to the mainboard.

CAM-C1126S2U has a flexible startup mode. Generally, the CAM-C1126S2U development board will not turn brick unless the hardware is damaged.

If the accident appeared in the process of upgrading, bootloader damage, leading to unable to upgrade again, while still can enter ` MaskRom ` mode to repair.


## How to get the Firmwares
*   [Firmware download link](https://en.t-firefly.com/doc/download/86.html)


## Upgrade method
CAM-C1126S2U supports firmware update through the following two methods:

* [Upgrade the firmware via USB cable](upgrade_firmware.md)<br><br>     Use the USB cable to connect the mainboard to the computer, and use the upgrade tool to program the firmware to the mainboard.<br><br>


## Boot media

CAM-C1126S2U loads the system from the following memory:

* eMMC interface

## Boot mode

CAM-C1126S2U has three startup modes:

* Normal mode
* Loader mode
* MaskRom mode


### Normal mode

Normal mode is the Normal startup process. Each component loads in turn and enters the system normally.


### Loader mode

In Loader mode, the bootloader will enter the upgrade state, waiting for the host command for firmware upgrade, etc.

### MaskRom mode

MaskRom mode is used for system repair when the bootloader is damaged.

Under normal circumstances, it is not necessary to enter `MaskRom mode`. Only when the bootloader verification fails (IDB block cannot be read, or the bootloader is damaged), the BootRom code will enter `MaskRom mode`. At this time, the BootRom code waits for the host to transmit the bootloader code through the USB interface, load and run it.

***Forcibly enter `MaskRom mode`, please refer to the chapter [MaskRom Mode](upgrade_maskrom_mode.md).***