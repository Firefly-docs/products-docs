# MaskRom mode

***See startup mode for an introduction [Boot mode description](01-bootmode.md)***

`MaskRom` pattern is the last line of defense equipment burn out. Forced entry `MaskRom` involved hardware operation, have certain risk, so only in the equipment into the `Loader` mode, can try `MaskRom` mode.

**Please read carefully and operate carefully!**

The operation steps are as follows:

1. Disconnect all power supplies.
2. Unplug the SD card.
3. Connect the equipment and host machine with Type-C data cable.
4. Use metal tweezers to connect and hold the two test points as shown in the following figure on AIO-3399C (as shown in the figure below).
5. Plug the device into the power supply.
6. Wait a moment, then loosen the tweezers.

![](../../../rk3399_img/AIO-3399C/maskrom_test_points.jpg)

At this point, the device should go into `MaskRom mode`.

![](../../../rk3399_img/maskrom.jpg)