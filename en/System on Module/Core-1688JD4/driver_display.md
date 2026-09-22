# DISPLAY usage

The Core-1688JD4 features an HDMI interface (supports 4K\@60fps) and a MIPS DSI interface (maximum output capability of 1440P (2560 \* 1440)@60fps,[The interface location is shown in the figure](interface_definition.md).

It is important to note that,` /mnt/system/ko/soph_drm.ko` the driver for the entire display framework, such as HDMI and MIPI DSI module drivers, is packaged within this driver.

## HDMI

After booting, the Core-1688JD4 does not have a built-in desktop environment and only runs a display program:

```
insmod /mnt/system/ko/soph_drm.ko # Install the display framework driver
systemctl stop SophonHDMI.service # Stop the HDMI display interface
systemctl restart SophonHDMI.service  # Restart the HDMI display interface
```

## MIPI DSI

Currently, the system can only have one display; when you connect HDMI, the MIPI DSI does not output by default.&#x20;
However, if you unplug the HDMI, the connected MIPI display module will show a test pattern in the form of color bars.

Currently adapted for Firefly's [DM-M10R800 V2](../../Accessories/DM-M10R800-V3S/started.md) display module.