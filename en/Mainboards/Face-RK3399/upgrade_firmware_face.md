# Face-X series upgrade firmware

## Preface 

Before reading this chapter, please read the [upgrade firmware](upgrade_firmware.md) chapter above. 

The way Face-X series upgrades firmware is basically the same as that in the [upgrade firmware](upgrade_firmware.md) chapter. However, Face-X series is equipped with a shell, and the recovery button is not exposed, so here we use another way to enter the upgrade mode:

1. After Face-X series is powered on, click the gear pattern on the upper right corner, and enter the password `123456` to enter the setting.
2. Open `USB OTG` options.
3. Connect OTG port of the Face-X series and computer with dual male USB date cable.
4. After the ADB is installed, enter the `adb reboot loader` in CMD, and the machine will enter the upgrade mode. Follow the [upgrade firmware](upgrade_firmware.md) chapter to complete the remaining steps. 