# Introduction

## Preface

The AIBOX-K3 comes pre-installed with Linux. If the user wants to run other operating systems, the corresponding firmware needs to be flashed onto the motherboard.

The AIBOX-K3 offers flexible boot options. Generally, unless there is hardware damage, the AIBOX-K3 development board will not become bricked.

If an accident occurs during the upgrade process and the bootloader is corrupted, making a re-upgrade impossible, it is still possible to force-enter the flashing mode to repair it.


## Firmware Acquisition

* [Download](https://en.t-firefly.com/doc/download/386.html)

## Upgrade Method

AIBOX-K3 supports firmware upgrades via the following methods:

* [Upgrade Firmware Using USB Cable](upgrade_firmware.md)<br><br>Use Type-C Cable to connect the motherboard to the computer, and use the upgrade tool to flash the firmware onto the motherboard. <br><br>

## Boot Storage

AIBOX-K3 Load the system from the following storage:

* UFS interface
* PCIe interface

## Boot Modes

AIBOX-K3 has the following boot modes:

* Normal Mode
* Loader Mode
* Hardware Flashing Mode

### Normal Mode

Normal mode is the normal boot process. Components load sequentially, and the system enters normally.

### Loader Mode

In Loader mode, the bootloader enters an upgrade state, waiting for host commands for firmware upgrades, etc.

### Hardware Flashing Mode

Hardware flashing mode is used for system repair when the bootloader is corrupted.

***To force entry into `Hardware Flashing Mode`, please refer to the chapter on [Hardware Flashing Mode](upgrade_boot_mode_spacemit.md).***