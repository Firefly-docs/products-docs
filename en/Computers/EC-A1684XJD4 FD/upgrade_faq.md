# FAQ

## Q: What firmware upgrade methods does the EC-A1684XJD4 FD support?

**A:** The EC-A1684XJD4 FD upgrades the firmware via a TF card:

* The TF card must use an MBR partition table and be formatted as FAT32;
* Extract all files from the firmware package to the root directory of the TF card (no folders on the TF card);
* Insert the TF card into the card slot and power on the device, the upgrade will complete automatically.

For EC series computers, see [Firmware upgrade](fw_upgrade.md); for AIBOX series, see [System firmware upgrade](fw-upgrade-by-sdcard.md).

## Q: How to tell the upgrade status during the upgrade?

**A:** Check the on-board LED indicators:

1. During the upgrade, the LEDs flash briefly, which means the upgrade is in progress.
2. If the upgrade is successful, the green LED keeps flashing.
3. If the upgrade fails, all LEDs go out.

## Q: What to do if the TF card cannot be formatted as FAT32?

**A:** If the TF card is larger than 32GB, it may not be able to be formatted as FAT32 due to Windows system limitations. It is recommended to choose a TF card of 32GB or less.

## Q: How long does the upgrade take? What to do if the upgrade fails?

**A:** The upgrade takes about six minutes, please be patient. If the upgrade fails, please make sure the TF card uses an MBR partition and FAT32 format, and all files in the firmware package have been extracted directly to the root directory of the TF card, then power on and upgrade again.
