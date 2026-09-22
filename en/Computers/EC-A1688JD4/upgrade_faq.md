# FAQ

## Q: What are the requirements for the TF card used for firmware upgrade?

**A:** The TF card needs to use an MBR partition and be formatted to FAT32. Class 10 or higher is recommended, with 8/16/32 GB capacity selected according to the firmware size.

Note: If the TF card exceeds 32GB, it may not be formatable to FAT32 due to Windows system limitations, so it is best to choose a TF card smaller than 32GB.

## Q: What are the steps for TF card upgrade? How to tell whether the upgrade succeeded?

**A:** The steps are as follows:

1. Unzip all files from the firmware (in zip format) to the root directory of the TF card
2. Insert the TF card into the TF card slot of the device, then power on
3. During the upgrade process, the LED will flash intermittently, indicating that the upgrade is in progress; the upgrade takes about six minutes, so please be patient
4. If the upgrade is successful, the green LED will flash continuously. At this time, remove the TF card and power cycle the device
5. If the upgrade fails, all LED lights will turn off

For details, please refer to [Firmware Upgrade](fw_upgrade.md).

