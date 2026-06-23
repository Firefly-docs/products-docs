# Use TF card to upgrade firmware

## Preparation

- AIBOX-1684
- PC
- USB card reader
- TF card (Class10 or higher is recommended, and choose 8/16/32 GB capacity according to firmware size)
- System firmware (Please go to [Download Center] to download)

## Upgrade steps

1. Format TF card using MBR partition and FAT32 format.
2. Extract all files in the upgrade firmware package directly to the TF card (TF card does not contain folders).
3. Insert the TF card into the TF card slot of the development board, then power on.
4. After the machine upgrade is completed, remove the TF card and disconnect the power supply.
5. Power on and start the system again to complete the upgrade.

## Indicator status

1. During the upgrade process, the LED light will flash briefly, indicating that the upgrade is in progress.
2. If the upgrade is successful, the green LED light will keep flashing.
3. If the upgrade fails, all LEDs will be off.

Precautions:

- If the TF card exceeds 32GB, it may not be formatted into FAT32 format due to the limitation of Windows system, so it is best to choose a TF card with a size below 32GB.
- The upgrade time is a bit long, about six minutes, please be patient.
- The cooling fan must be installed correctly.

[Download Center]: https://en.t-firefly.com/doc/download/290.html