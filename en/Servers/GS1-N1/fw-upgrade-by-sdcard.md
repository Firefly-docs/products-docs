# Use TF card to upgrade firmware

## Preparation

- GS1-N1.
- Host computer.
- USB card reader.
- TF card, Class10 or higher is recommended, and choose 8/16/32 GB capacity according to firmware size.
- To upgrade the firmware, please go to [Download Center] to download.

## Upgrade steps

1. The TF card is partitioned by MBR and formatted into FAT32 format .
2. Extract all the files in the upgrade firmware file (zip compressed format) to the TF card.
3. Insert the TF card into the TF card slot of the development board, then power on.
4. During the upgrade process, the LED light will flash briefly, indicating that the upgrade is in progress.
5. If the upgrade is successful, the green LED light will keep flashing.
6. If the upgrade fails, all LEDs will be off.

Precautions:

- If the TF card exceeds 32GB, it may not be formatted into FAT32 format due to the limitation of Windows system, so it is best to choose a TF card with a size below 32GB.
- The upgrade time is a bit long, about six minutes, please be patient.
- The cooling fan must be installed correctly.

[Download Center]: https://en.t-firefly.com/doc/download/342.html