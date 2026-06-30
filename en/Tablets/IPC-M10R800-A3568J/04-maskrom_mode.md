# MaskRom mode

***See startup mode for an introduction [startup mode](01-bootmode.md)***

`MasRrom` mode is the last line of defense against device being bricked. Forced entry `MaskRom` involved hardware operation, have certain risk, so only in the situation that deivce failed entering the `Loader` mode, you can try `MaskRom` mode.

**Please read carefully and operate carefully!**

The operation steps are as follows:

1. Disconnect all power supplies.
1. Unplug the SD card.
1. Connect the device and host PC with Double male USB data cable.
1. Use metal a tweezer to short and hold the two test points as shown in the following figure on IPC-M10R800-A3568J (as shown in the figure below).
1. Plug in the power supply.
1. Wait a few seconds, stop shorting.

![](../../../rk356x_img/IPC-M10R800-A3568J/maskrom_test_points.jpg)

At this point, the device should go into `MaskRom mode`.

![](../../../rk356x_img/maskrom_zh.png)