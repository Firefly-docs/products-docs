# Upgrade the firmware via SD card

## Introduction

This article mainly introduced how to upgrade the firmware on the host to the SD card, which needs to be selected according to the host operating system.

## Preparatory Tools

* CAM-C1126S2U
* firmware
* host computer
* USB Card Reader 
* MIrcoSD card
* [**SD_Firmware_Tool**](https://en.t-firefly.com/doc/download/140.html#other_499)
## Operation Steps
 
* Insert microSD card(Suggest using a card with a capacity of 32GB or less) into USB card reader and then into USB port of host computer 
* Run SD_Firmware_Tool, check the "Upgrade Firmware" box and select the correct removable disk device
* Choose firmware we want to upgrade into the CAM-C1126S2U
* Click  button `Create` to make it and wait until it is finshed


<center>

![](../../../rv1126_img/common/upgrade_firmware_sd_tool_zh.png)

</center>



* Remove the microSD card, insert it into the microSD card slot of the motherboard, power on the board, it will start upgrading automatically
*  After the upgrade, remove the microSD card and restart the motherboard automatically to complete the whole process of firmware update 