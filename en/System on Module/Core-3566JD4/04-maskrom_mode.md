# MaskRom mode

***See startup mode for an introduction [startup mode](01-bootmode.md)***

`MasRrom` mode is the last line of defense against device being bricked. Forced entry `MaskRom` involved hardware operation, have certain risk, so only in the situation that deivce failed entering the `Loader` mode, you can try `MaskRom` mode.

**Please read carefully and operate carefully!**

The operation steps are as follows:

1. Disconnect all power supplies.
1. Connect the device and host PC with Type-C data cable.
1. Use a metal tweezer to short and hold the two test points as shown in the following figure on Core-3566JD4 (as shown in the figure below).
1. Connect the power.
1. Wait a few seconds, stop shorting.

<center>

<img alt="" src="../../../rk356x_img/Core-3566JD4/maskrom_test_points.jpg" width="700">
</center>

When the board has NOR flash at the same time, if EMMC is empty and there are burned files in NOR flash, it is necessary to short circuit the D0 and GND test points near NOR flash to enter Maskrom mode. And now we have to  refer to the chapter "[Switching Upgrade Storage](03-upgrade_firmware_with_flash)" for upgrade
<center>

<img alt="" src="../../../rk356x_img/Core-3566JD4/maskrom_test_points_flash.png" width="700">
</center>


At this point, the device should go into `MaskRom mode`.

<center>

<img alt="" src="../../../rk356x_img/maskrom_zh.png" width="700">
</center>