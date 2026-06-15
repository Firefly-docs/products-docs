## HDMI can’t display 4K?

The default firmware of the AIO-3399J is a dual-screen display that supports LVDS+HDMI 1080P. The HDMI resolution can only be up to 1080P. HDMI to support 4K resolution needs to re-upgrade the [default firmware](http://en.t-firefly.com/doc/download/52.html#other_118), or recompile the kernel `make ARCH=arm64 rk3399-firefly-aio.img`, re-upgrade `resource.img`.

## How to confirm whether the firmware supports 4K?

AIO-3399J open `Settings` -> `Display`. If HDMI and DSI appear on the display device, the firmware supports dual-screen display and does not support 4K. If only HDMI is the default firmware, it can support 4K.

## How to adjust the HDMI output resolution?

AIO-3399J HDMI can automatically identify the display resolution. If you can not read the monitor’s EDID (Extended Display Identification Data, extended display identification data), HDMI will default to 1080P resolution. You can also enter the system settings to adjust the HDMI resolution manually.

![](../../../rk3399_img/faqs_setting_resolution.jpg)