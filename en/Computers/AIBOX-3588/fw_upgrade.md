# Firmware upgrade

AIBOX-3588 uses TF card to upgrade firmware, users need to strictly follow the steps below.

## Preparation

- AIBOX-3588
- PC
- USB card reader
- TF card, Class10 or higher is recommended, and choose 8/16/32 GB capacity according to the firmware size
- Firmware to be upgraded

## Upgrade steps

(1) The TF card needs to use MBR partition and format it into FAT32 format;

(2) Decompress all the files in the firmware to be upgraded (zip compression format) to the TF card;

(3) Insert the TF card into the TF card slot of AIBOX-3588, and then turn on the power;

(4) During the upgrade process, the LED light will flash briefly, indicating that the upgrade is in progress;

(5) If the upgrade is successful, the green LED light will keep flashing;

(6) If the upgrade fails, all LED lights will be off.

Precautions:

- If the TF card exceeds 32GB, it may not be formatted into FAT32 format due to the limitations of the Windows system, so it is best to choose a TF card with a size below 32GB
- The upgrade time is a bit long, about six minutes, please be patient