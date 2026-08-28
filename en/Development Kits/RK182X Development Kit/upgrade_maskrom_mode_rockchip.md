# MaskRom Mode

The RK182X development kit does not provide a Loader mode. USB firmware upgrades and bootloader recovery must use the board's `MaskRom` key.

**Read the instructions carefully and operate with the board powered off.**

## Enter MaskRom Mode

1. Disconnect the development kit from the power supply.
2. Set the `USB SEL` switch to `1`.
3. Connect the OTG port to the host computer with a Type-A data cable.
4. Press and hold the board's `MaskRom` key.
5. Power on the development kit while holding the key.
6. Check the upgrade tool for a MaskRom device.
7. Release the key after the device is detected.

The board is now ready for firmware writing.

![](../../../gs1-n2_img/common/upgrade_maskrom_zh.png)

## Check MaskRom Mode

### Windows

AndroidTool should display a MaskRom device in the device list. If the device is not detected, check the USB driver, the Type-A data cable, the `USB SEL` switch, and the OTG connection.

### Linux

Run `upgrade_tool` and check whether the connected device is listed as MaskRom:

```shell
sudo upgrade_tool
```
