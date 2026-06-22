# Upgrade the firmware via SD card

This article mainly introduces how to use a MicroSD card to update the firmware on the motherboard. <font color = "red">But only if the firmware is smaller than 4G</font>, otherwise, please check ["If the firmware is larger than 4G SD upgrade card making tutorial"](05-upgrade_firmware_sd_morethan_4g.md).

To use MicroSD to update the firmware, you need to write the unified firmware into the MicroSD card by using a card tool on the computer. Currently, this operation is only supported on the Windows operating system.

## Preparatory Tools

* AIO-3399C
* computer
* MicroSD card (ie TF card)
* USB card reader
* [SD_Firmware_Tool](http://en.t-firefly.com/doc/download/52.html#other_125)


## Run `SD_Firmware_Tool.exe`

1. Run `SD_Firmware_Tool.exe`
2. You can modify the **config.ini** file, set `Selected=1` to set the language to Chinese, set `Selected=2` to set the language to English.
3. Download the unified firmware that needs to be upgraded to the motherboard.
4. Insert the MicroSD into the USB card reader and then into the USB port of the computer.
5. Open SD_Firmware_Tool, select the correct removable disk device, check the `Upgrade Firmware` box, click `Firmware` to select the firmware you want to upgrade.

    ![](../../../rk3399_img/sdfirmwaretool.en.jpg)
6. After clicking `Create`, wait for the creation to finish.

    ![](../../../rk3399_img/sdfirmwaretool_done.en.jpg)
7. Take out the MicroSD card, insert it into the MicroSD card slot of the motherboard, power on the motherboard, and the motherboard will automatically start to upgrade.
8. After the upgrade is complete, take out the MicroSD card and the motherboard will automatically restart to complete the entire firmware update process.