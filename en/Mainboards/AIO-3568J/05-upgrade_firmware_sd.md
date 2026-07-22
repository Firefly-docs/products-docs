# Upgrade the firmware via SD card

## Introduction

This article mainly introduced how to upgrade the firmware on the host to the SD card, which needs to be selected according to the host operating system.

## Preparatory Tools

* AIO-3568J
* firmware
* host computer
* USB Card Reader 
* MIrcoSD card
* [**SD_Firmware_Tool**](http://en.t-firefly.com/doc/download/93.html#other_368)
## Operation Steps
 
* Insert microSD card into USB card reader and then into USB port of host computer 
* Run SD_Firmware_Tool, check the "Upgrade Firmware" box and select the correct removable disk device
* Choose firmware we want to upgrade into the AIO-3568J
* Click  button `Create` to make it and wait until it is finshed


<center>

![](../../../rk356x_img/SD_Firmware_Tool_zh.png)

</center>



* Remove the microSD card, insert it into the microSD card slot of the motherboard, power on the board, it will start upgrading automatically
*  After the upgrade, remove the microSD card and restart the motherboard automatically to complete the whole process of firmware update 