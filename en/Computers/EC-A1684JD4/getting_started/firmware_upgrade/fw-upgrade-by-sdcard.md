# SD Card Upgrade Firmware

If the system firmware is damaged due to improper operations or the system firmware version needs to be updated, perform the following operations to upgrade the system firmware of EC-A1684JD4.

## Preparation [step]

- EC-A1684JD4
- PC
- USB card reader
- TF card (Class10 or higher is recommended, and choose 8/16/32 GB capacity according to firmware size)
- System firmware (Please go to [Download Center] to download)

## Upgrade steps [step]

1. Format TF card using MBR partition and FAT32 format.
2. Extract all files in the upgrade firmware package directly to the TF card (TF card does not contain folders).
3. Insert the TF card into the TF card slot of the development board, then power on.
4. After the machine upgrade is completed, remove the TF card and disconnect the power supply.
5. Power on and start the system again to complete the upgrade.

## Indicator status [step]

1. During the upgrade process, the LED light will flash briefly, indicating that the upgrade is in progress.
2. If the upgrade is successful, the green LED light will keep flashing.
3. If the upgrade fails, all LEDs will be off.

Precautions:

- The cooling fan must be installed correctly.

[Download Center]: https://community.t-firefly.com/en/doc/download/177
