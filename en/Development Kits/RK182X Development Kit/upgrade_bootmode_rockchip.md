# Boot Modes

RK182X Developer Kit is shipped with an operating system. To run another operating system, download the corresponding firmware from the [firmware download page](https://community.t-firefly.com/en/doc/download/358).

If an upgrade damages the bootloader and normal upgrades no longer work, use `MaskRom` mode to repair the board.
For complete upgrade procedures, see [Upgrade Firmware via USB Cable](upgrade_firmware_rockchip.md) or [Upgrade the firmware via SD card](upgrade_firmware_sd_rockchip.md).

## Boot Media

RK182X Developer Kit loads the system from the following media:

* eMMC interface
* SDMMC interface

## Boot Modes

RK182X Developer Kit has two boot modes:

* Normal mode
* MaskRom mode

### Normal Mode

Normal mode is the regular startup process. Each component loads in sequence and the system starts normally.


### MaskRom Mode

MaskRom mode is used to write firmware or repair the system when the bootloader is damaged. For this development kit, USB firmware upgrade always uses the board's `MaskRom` key.

For the hardware operation, see [MaskRom mode](upgrade_maskrom_mode_rockchip.md).