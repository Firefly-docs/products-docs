# Upgrade System Firmware Using TF Card

## Preparation

- AIBOX-1688.
- PC.
- USB card reader.
- TF card, recommended Class 10 or higher, and choose 8/16/32 GB capacity based on firmware size.
- To upgrade firmware, please download it from the [Download Center].

## Upgrade Steps

1. Format the TF card with MBR partition and FAT32 format.
2. Extract all files from the upgrade firmware file (zip format) to the TF card.
3. Insert the TF card into the TF card slot of the development board, then power it on.

4. During the upgrade process, the LED will briefly flash, indicating that the upgrade is in progress.
5. If the upgrade is successful, the green LED will keep flashing.
6. If the upgrade fails, all LED lights will turn off.

Notes:

- If the TF card exceeds 32GB, it may not be possible to format it to FAT32 due to Windows system limitations, so it is best to choose a TF card smaller than 32GB.
- The upgrade process may take a while, approximately six minutes, please be patient.

- Ensure that the cooling fan is installed correctly.

[Download Center]: https://www.t-firefly.com/doc/download/266.html