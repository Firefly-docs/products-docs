# Boot mode description

## Preface

The IHC-3308GW is installed with the Buildroot operating system by default. If users want to run other operating systems, they need to use the corresponding firmware to program to the mainboard.

IHC-3308GW has a flexible startup mode. Generally, the IHC-3308GW development board will not turn brick unless the hardware is damaged.

If the accident appeared in the process of upgrading, bootloader damage, leading to unable to upgrade again, while still can enter ` MaskRom ` mode to repair.


## How to get the Firmwares
*   [Firmware download link](https://en.t-firefly.com/doc/download/92.html)


## Upgrade method
IHC-3308GW supports firmware update through the following two methods:

- Update firmware using USB cable

    Use the USB cable to connect the mainboard to the computer, and use the upgrade tool to program the firmware to the mainboard.




## boot media

* eMMC interface

In addition, IHC-3308GW supports downloading system codes from the Type-C data cable interface.

## Boot mode

IHC-3308GW has three startup modes:

* Normal mode
* Loader mode
* MaskRom mode


### Normal mode

Normal mode is the Normal startup process. Each component loads in turn and enters the system normally.


### Loader mode

In Loader mode, the bootloader will enter the upgrade state, waiting for the host command for firmware upgrade, etc. To enter the Loader mode, the bootloader must detect that the `RECOVERY` key has been pressed and the USB is connected.


The method to put the device into upgrade mode is as follows:

One way is to disconnect the power adapter



* Type-C data cable connects the device and the host.
* Press and hold the RECOVERY button on the device and hold it.
* Plug in the power
* After about two seconds, release the RECOVERY key.

Another way is to connect the power adapter

* The Type-C data cable connects the device and the host.
* Press and hold the RECOVERY button on the device and hold it.
* Briefly press the RESET button.
* After about two seconds, release the RECOVERY key.
### MaskRom mode

MaskRom mode is used for system repair when the bootloader is damaged.

Under normal circumstances, it is not necessary to enter MaskRom mode. Only when the bootloader verification fails (IDB block cannot be read, or the bootloader is damaged), the BootRom code will enter MaskRom mode. At this time, the BootRom code waits for the host to transmit the bootloader code through the USB interface, load and run it.

***Forcibly enter MaskRom mode, please refer to the chapter [MaskRom Mode](04-maskrom_mode.md).***