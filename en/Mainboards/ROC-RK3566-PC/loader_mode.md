# Loader upgrade mode

***For the introduction of boot mode, please refer to the chapter ["Introduction to updating firmware"](01-bootmode.md)***



## Introduction

In Loader mode, the bootloader will enter the upgrade state, waiting for the host command for firmware upgrade and so on. To enter Loader mode, the bootloader must detect a `RECOVERY` key press at startup and the USB is connected.

Here's how to put your device into upgrade mode:

One way is to disconnect the Type-C data cable


* Press and hold the RECOVERY button(near by the phone jack) on the device and hold it.
* Plug in the power(Type-C data cable)
* After about two seconds, release the RECOVERY key.

Another way is to connect the Type-C data cable

* The Type-C data cable connects the device and the host.
* Press and hold the RECOVERY button on the device and hold it.
* After about two seconds, release the RECOVERY key.



## upgrade firmware

Developers can choose different firmware upgrade tools according to their different PC operating systems.
* Windows environment

**If developers use `Windows` host for development, they can use `RKDevTool` to upgrade the tool. For details, please refer to ["Using RKDevTool to upgrade firmware"](Windows_upgrade_firmware.html).**

* Linux environment

**If developers use `Linux` host for development, they can use `upgrade_tool` to upgrade the tool. For details, please refer to ["Using upgrade_tool to upgrade firmware"](Linux_upgrade_firmware.html).**