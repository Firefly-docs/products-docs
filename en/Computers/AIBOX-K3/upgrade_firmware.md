# Upgrading Firmware Using a USB Cable

## Introduction

This article describes how to burn the firmware from the host device to the BOARD with space using Type-C 数据线. During the upgrade, you need to select the appropriate upgrade method based on the host operating system and firmware type.

Please read the "Flashing Tool User Manual" carefully before flashing. If it doesn't display, please refresh the page.

* Flashing Tool User Manual Link: https://spacemit.com/community/document/info?lang=en&nodepath=tools/user_guide/flasher_user_guide.md

## Preparation Tools

* AIBOX-K3
* [Firmware](https://en.t-firefly.com/doc/download/386.html)
* Main Unit
* Good Type-C Cable

## Preparing firmware

Firmware can be obtained by compiling the SDK or downloaded from [Resource Download](https://en.t-firefly.com/doc/download/386.html). Firmware files generally come in two types:

* Single Unified Firmware

Unified firmware is a single file packaged and merged from all files including the partition table, bootloader, uboot, kernel, and system. Officially released Firefly firmware uses the unified firmware format. Upgrading to unified firmware will update the data and partition table of all partitions on the motherboard and erase all data on the motherboard.

* Multiple Partition Images

These are files with independent functions, such as the partition table, bootloader, and kernel, generated during the development phase. Independent partition images can update only specified partitions while keeping other partitions' data intact, which is very convenient for debugging during development.

## Installing the Flashing Tool

### Windows Operating System

1. Download the latest version of the flashing tool titantools_for_windows.exe

2. Double-click the downloaded flashing tool titantools_for_windows_last to begin installation;

3. After installation, run titanflasher.exe

### Linux Operating System

1. Download the latest version of the flashing tool titantools_for_linux.AppImage
2. Double-click the downloaded flashing tool titantools_for_linux_last to begin installation;
3. After installation, run titanflasher.exe

![](../../../aibox_img/AIBOX-K3/upgrade_firmware_titanflasher_en.png)


* [Linux_adb_fastboot](https://en.t-firefly.com/doc/download/338.html#other_565)工具

Download [Linux_adb_fastboot](https://en.t-firefly.com/doc/download/338.html#other_565) and install it on your system using the following method for easy access:

```
sudo mv fastboot /usr/local/bin
sudo chown root:root /usr/local/bin/fastboot
sudo chmod a+x /usr/local/bin/fastboot
```

## Entering Upgrade Mode

There are generally two modes for upgrading firmware: Loader mode and hardware flashing mode. Before flashing the firmware, we need to connect the device and put the board into upgradeable mode.

### Loader Mode

#### Entering Loader Mode via Software

After connecting the Type-C Cable, run the following command on the board in the serial port debugging terminal. Do not use the adb tool to enter loader mode, as software entry into loader mode requires operation of the uboot terminal.

```bash
reboot
```

After the board system reboots to the uboot stage, press and hold the 's' key to enter the uboot debugging terminal. Once in the uboot debugging terminal, delete any unnecessary 's' characters from the input information. Execute the `fastboot` command to enter flashing mode:

```bash
fastboot 0
```
On the PC, in the titanflasher flashing tool, click:
Dev Tools --> USB Download --> Scan Devices --> Local File --> Select Zip File --> Start Flashing.

Note: At this point, the titanflasher tool will flash a portion of the necessary firmware to the board, but not all of it. You need to press Enter again in the board's uboot terminal to complete the subsequent firmware flashing.

### Hardware Flashing Mode

For instructions on entering hardware flashing mode, please refer to [Hardware Flashing Mode](upgrade_boot_mode_spacemit.md)

## Flashing Firmware

#### Flashing Unified Firmware

The steps for flashing the unified firmware package are as follows:

Click the following steps: Dev Tools --> USB Download --> Scan Devices --> Local File --> Select Zip File --> Reboot After Flashing --> Start Flashing.

![](../../../aibox_img/AIBOX-K3/upgrade_firmware_en.png)

#### Burning Partition Image

Click Steps: Dev Tools --> USB Download --> Scan Devices --> Local File --> Select Zip File --> Reboot After Flashing --> Configure Partition File --> Start Flashing.

Configuring partition files requires selecting the partition file.

1. `partition_4M.json` updates the NOR flash on the core board.
2. `partition_universal.json` updates the UFS partition image on the core board.

![](../../../aibox_img/AIBOX-K3/upgrade_firmware_part_en.png)

## Frequently Asked Questions

### 1. How to Force Enter Hardware Flashing Mode

If the board cannot enter Loader mode, you can try to force enter hardware flashing mode. See [Hardware Flashing Mode](upgrade_boot_mode_spacemit.md) for instructions.

### 2. Flashing Failure Analysis

If an error occurs during the flashing process, it is usually due to a faulty USB cable connection, a low-quality cable, or insufficient driver capability of the computer's USB port. Please replace the USB cable or check the computer's USB port.

[Androidtool_xxx(版本号)]: http://www.t-firefly.com/share/index/index/id/2ea171f2235fe841e89734ca5189da8b.
[AndroidTool]: http://www.t-firefly.com/share/index/index/id/2ea171f2235fe841e89734ca5189da8b.html
[Release_DriverAssistant.zip]: http://www.t-firefly.com/share/index/index/id/1f98ebd663ed09a32e9ebf3fa893dfc0.html
[Linux_Upgrade_Tool]: http://www.t-firefly.com/share/index/index/id/f756718dd2bbf82eb405926549e75ef3.html
[Linux_adb_fastboot]: http://www.t-firefly.com/share/index/index/id/c64b7d743d9368de521a6ced87813dc5.html
