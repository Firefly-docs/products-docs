# Loader upgrade mode

***For the introduction of boot mode, please refer to the chapter ["Introduction to updating firmware"](01-bootmode.md)***

## Introduction

In Loader mode, the bootloader will enter the upgrade state, waiting for the host command for firmware upgrade and so on. To enter Loader mode, the bootloader must detect a `RECOVERY` key press at startup and the USB is connected.

Here's how to put your device into upgrade mode:


* Disconnect all power supply
* Press and hold RECOVERY button on board
* Connect the power supply
* After few seconds, release RECOVERY button


## upgrade firmware

Developers can choose different firmware upgrade tools according to their different PC operating systems.

Please refer to [Use USB cable to upgrade](03-upgrade_firmware.html)