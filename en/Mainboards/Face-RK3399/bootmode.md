# Boot mode description

## Preface

Face-RK3399 has a flexible startup mode. Generally, the Face-RK3399 development board does not brick unless the hardware is damaged.

If the accident appeared in the process of upgrading, bootloader damage, leading to unable to upgrade again, while still can enter ` MaskRom ` mode to repair.

## Loading way

Face-RK3399 has 32KB BootRom and 200KB internal SRAM and supports loading the system from the following devices:

* SPI interface
* eMMC interface
* SDMMC interface

Face-RK3399 also supports downloading system code from USB OTG interface.

## Boot order

The order of startup:

1. Main control power on initialization.
2. BootRom code runs on SRAM and verifies the bootloader in the storage device.
3. Verify pass, load and run bootloader boot code.
4. Bootloader boot code is responsible for initializing DDR memory, loading the complete bootloader code into DDR memory and running it.
5. Bootloader loads the Linux kernel on the storage device and gives the execution power to the Linux kernel.

## Boot mode

Face-RK3399 has three startup modes:

* Normal mode
* Loader mode
* MaskRom mode

### Normal mode

Normal mode is the Normal startup process. Each component loads in turn and enters the system normally.

### Loader mode

In Loader mode, bootloader will enter the upgrade state and wait for the host command for firmware upgrade. To enter the Loader mode, must let the bootloader detected in startup `RECOVERY` key press, and USB connection status. The method to put the device into upgrade mode is as follows:

Disconnect the power adapter:

* Connect device OTG port and host with USB data cable.
* Press the RECOVERY button on the device and hold it.
* Plug in.
* About two seconds later, release the RECOVERY key.

### MaskRom mode

MaskRom mode is used for system repair when bootloader is damaged.

Generally, it is not necessary to enter MaskRom mode. BootRom code will enter MaskRom mode only if bootloader validation fails (IDR block cannot be read, or bootloader is damaged). At this point, BootRom code waits for the host to transfer the bootloader code through the USB interface and load and run it.

***force into MaskRom mode, please refer to the chapter of [MaskRom](maskrom_mode.md).***