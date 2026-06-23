# Firmware Upgrade

AIBOX-186 uses a TF card to upgrade the firmware, and users must strictly follow the steps below.

## Preparation

- AIBOX-186
- PC
- USB card reader
- TF card, recommended Class 10 or higher, and choose 8/16/32 GB capacity based on firmware size
- Firmware to be upgraded

## Upgrade Steps

(1) The TF card needs to use MBR partition and be formatted to FAT32;

(2) Unzip all files from the firmware (in zip format) to the root directory of the TF card;

(3) Insert the TF card into the TF card slot of the AIBOX-186, then power on;

(4) During the upgrade process, the LED will flash intermittently, indicating that the upgrade is in progress;

(5) If the upgrade is successful, the green LED will flash continuously; at this time, remove the TF card and power cycle the device;

(6) If the upgrade fails, all LED lights will turn off.

Notes:

- If the TF card exceeds 32GB, it may not be formatable to FAT32 due to Windows system limitations, so it is best to choose a TF card smaller than 32GB.
- The upgrade process may take a while, approximately six minutes, so please be patient.