## Differences between V1.0 and V1.1 devices
### V1.0 version
On V1.0 version devices, I2C2_M0 and I2C2_M1 are simultaneously drawn out by hardware, resulting in multiplexing. Therefore, MIPI CSI using I2C2_M1 and RTC and MIPI DSI touch parts using I2C2_M0 cannot be used at the same time. The default firmware for the V1.0 version of the device is enabling I2C2_M0 (it supports RTC and MIPI DSI and turns off MIPI CSI). If you need to enable MIPI CSI, then the SDK needs the following modifications
```
&i2c2{
	pinctrl-0 = <&i2c2m1_xfer>;
};

```

This makes the I2C2 use the M1 channel, while the touch part of the MIPI DSI and RTC is temporarily unavailable.

### V1.1 and above version
On V1.1 and above version,MIPI CSI using I2C4_M0 and RTC and MIPI DSI touch parts using I2C2_M0.Therefore,there are no conflicts, and do not need to use the previous step to distinguish the use of the configuration.

## HDMI can't display 4K?

Automatically switching to 4K is temporarily not supported. The default display is 1080P. Support will be updated in the future.After setting the 4K resolution manually, you can use the following command to replug the HDMI cable:

```
# 4k60 setting：
setprop persist.sys.resolution.main 3840x2160@60
# Update sys.display.timeline every time you set it (add 1 each time) for resolution to take effect
setprop sys.display.timeline 1
```

## The 3.5mm headphone jack is connected to the national standard headphone abnormal?

At present, Station-M2 only supports the American standard type of headphones(CTIA). For the national standard type of headphones(OMTP), the hardware is not compatible, and there will be the phenomenon of dual sound channels in both the left and right channels.

