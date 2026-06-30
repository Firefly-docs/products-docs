# Introduction to updating firmware

## Preface



AIO-3568J has 2 working modes. Under normal circumstances, boot directly into `Normal mode` to start the system normally. If you need to upgrade the board subsystem, you can choose the appropriate `Upgrade mode` to upgrade the firmware according to the situation.

* Note: The AIO-3568J is installed with the Android operating system by default. If users want to run other operating systems, they need to use the corresponding firmware to program to the mainboard.You can go to the [Firmware download page](http://en.t-firefly.com/doc/download/108.html) to download more firmware.


![](../../../rk356x_img/working_mode_en.png)

**AIO-3568J is classified according to the working mode and can be divided into two working modes:**

## Normal mode

| Working Mode | Normal Mode | Upgrade Mode |
| :--------: | :-------: | :------- |
| Boot Media | eMMC Interface/SDMMC Interface| | √ |
| Description | Normal mode is the normal startup process, <br /> each component is loaded in sequence and enters the system normally. | There are currently 3 upgrade modes supported, each with their own advantages and disadvantages:<br />1. [MaskRom Upgrade Mode](04-maskrom_mode.html)<br />2. [Loader upgrade mode](loader_mode.html)<br />3. [SD upgrade mode](05-upgrade_firmware_sd.html)|

## Upgrade mode



**Among the upgrade modes, the comparison between different upgrade modes:**

| Upgrade mode  | [MaskRom Upgrade Mode](04-maskrom_mode.html) | [Loader upgrade mode](loader_mode.html) | [SD upgrade mode](05-upgrade_firmware_sd.html) |
| :--------: | :------- | :------- | :------- |
| Quick description | 1. Use the USB cable to connect the motherboard to the computer;<br />2. The hardware operation makes the board enter the upgrade mode;<br />3. Use USB to upgrade the board firmware on the PC.  |  1. Use the USB cable to connect the motherboard to the computer;<br />2. Software or key operation makes the board enter the upgrade mode;<br />3. Use USB to upgrade the board firmware on the PC. | 1. Use the upgrade card making tool to make the MicroSD card as an upgrade card;<br />2. Insert the upgrade card into the motherboard, power on, and the machine will automatically perform the upgrade.|
| Connection method | USB | USB | TF card (a few are SD card slots) |
| Upgrade Tool<br />Windows PC<br />Linux PC | <br />[Upgrading firmware on Windows](Windows_upgrade_firmware.html)<br />[Upgrading firmware on Linux](Linux_upgrade_firmware.html) | <br />[Upgrading firmware on Windows](Windows_upgrade_firmware.html)<br />[Upgrading firmware on Linux](Linux_upgrade_firmware.html) | <br />[Making an Upgrade Card on Windows](05-upgrade_firmware_sd.html)<br />Not support |
| Entry method | Requires hardware operation |  Key or software entry| Power on and enter directly|
| Conditions of Use | Hardware operation entry |  Can use uboot normally| Nothing|
| Recommended usage scenarios | 1. When the board cannot be started normally;<br />2. When switching between programming Linux and Android firmware. |  1. Have a complete uboot or can enter the system normally;<br />2. Need to burn the partition separately (uboot or boot partition, etc.).| 1. It is convenient for workers to operate, suitable for mass production of products;<br />2. The products can be upgraded after finalization, which is convenient for end customers to operate.|
| Advantage | <font color = "red">1. The most basic programming method; </font><br />2. Non-firmware and hardware problems can generally be successfully programmed;<br />3. No uboot support is required , to save the bricked board;<br />4. Support cross-system upgrade (Linux and Android, etc.). | 1. The programming effect is similar to MaskRom Upgrade Mode;<br />2. It can write partitions separately;<br />3. It is convenient to enter the loader mode. | 1. Easy to operate, just plug in the card to start;<br />2. It combines the advantages of MaskRom Upgrade Mode. |
| Disadvantages | 1. It is difficult to enter the product, and it is difficult to remove the shell;<br />2. It is troublesome to program the partition table, and it is difficult to program the partitions individually;<br />3. The device needs to be completely erased and then programmed. | 1. A complete loader (usually uboot) is required;<br />2. Cross-system upgrade requires a complete erasing of the device and then programming. | 1. Requires to synthesize full firmware. |



#### MaskRom Upgrade Mode

In general, there is no need to enter `MaskRom Upgrade Mode`. Only when the bootloader verification fails (the IDB block cannot be read, or the bootloader is damaged), the BootRom code will enter this mode. At this time, the BootRom code waits for the host to transmit the bootloader code through the USB interface, load and run it. When the board becomes bricked and cannot start or upgrade the program normally, you can also manually enter the `MaskRom Upgrade Mode`.

***To forcibly enter `MaskRom Upgrade Mode`, please refer to the chapter ["MaskRom Upgrade Mode"](04-maskrom_mode.md).***


#### Loader upgrade mode

In `Loader upgrade mode`, the bootloader will enter an upgrade state, waiting for host commands for firmware upgrades, etc. To enter this mode, the bootloader must detect a `RECOVERY` key press at startup and the USB is connected.

***To force into `Loader upgrade mode`, see the chapter ["Loader upgrade mode"](loader_mode.md).***

#### SD upgrade mode

Using SD upgrade is essentially to make a bootable SD-boot upgrade firmware, let the board SD boot, erase and program the EMMC.

***To forcibly enter `SD upgrade mode`, please refer to the chapter ["Upgrade Firmware Using SD Card"](05-upgrade_firmware_sd.md).***
