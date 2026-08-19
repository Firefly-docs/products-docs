# Boot mode description

## Preface

CT36L/CT36B is shipped with the Linux operating system installed by default.

CT36L/CT36B has flexible startup methods. Under normal circumstances, the CT36L/CT36B development board will not become bricked unless the hardware is damaged.

If an accident occurs during the upgrade process and the bootloader is damaged, making it impossible to re-upgrade, you can still enter `MaskRom` mode to repair it.


## Firmware acquisition
* CT36L [Download link](https://community.t-firefly.com/en/doc/download/214.html)
* CT36B [Download link](https://community.t-firefly.com/en/doc/download/214.html)

## Upgrade method
CT36L/CT36B supports firmware upgrade through the following two methods:

* [Use USB cable to upgrade firmware](upgrade_firmware.md)<br><br>
Use CT36L/CT36B to connect the motherboard to the computer, and burn the firmware to the motherboard through the upgrade tool. <br><br>
* [Upgrade firmware using SD card](upgrade_firmware_sd.md)<br><br>
* Note 1: `SocToolKit` tool version 1.7 or higher is required to support the SD card upgrade boot function.
* Note 2: This function requires the user to run `SocToolKit.exe` as an administrator (it will ask by default when opening the tool).
* Insert the successfully created SD card into the device and then restart. The device will enter the U-Boot terminal in the SD card first.
* If the SD card has an upgrade function, the device will be automatically upgraded.
* After the upgrade is completed, you need to remove the SD card and restart the device to enter the device system.


## Start memory

CT36L  loads the system from memory:

* SPI FLASH interface
* SDMMC interface

CT36B  loads the system from memory:

* EMMC interface
* SDMMC interface

## Startup mode

CT36L/CT36B has two startup modes:

* Normal mode
* MaskRom mode

### Normal mode

Normal mode is the normal startup process. Each component is loaded in sequence and the system is entered normally.

### MaskRom mode

MaskRom mode is used for firmware programming.

***To forcefully enter `MaskRom mode`, please refer to the chapter ["MaskRom Mode"](upgrade_maskrom_mode.md). ***