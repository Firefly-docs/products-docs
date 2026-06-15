## AIO-3399Pro-JD4 How does Android9 use the ALC5640 codec on the baseboard?
The system uses RK809 codec by default. If you use ALC5640, you need to modify the hardware.
I2S1 signal pin is changed to I2S0 (Core-3399Pro-JD4 does not lead to I2S1), I2C and I2S_CLK do not move, need to jump off the resistance of R89~R93.

![](../../../rk3399_img/Firefly-RK3399/5640_sch.png)
![](../../../rk3399_img/Firefly-RK3399/5640_pcb.png)

The software is modified as follows:

```
**Modified according to the latest code**
commit 809d8d77b59996770a05f81abff9ff2553773ff7
Author: Firefly <service@t-firefly.com>
Date:   Tue Oct 20 15:41:26 2020 +0800

    1.add alc5640 support
```

```
--- a/kernel/arch/arm64/boot/dts/rockchip/rk3399pro-firefly-aiojd4.dtsi
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk3399pro-firefly-aiojd4.dtsi
@@ -20,11 +20,11 @@
        };

        rk809-sound {
-               status = "okay";
+               status = "disabled";
        };

         rt5640-sound {
-               status = "disabled";
+               status = "okay";
                 compatible = "simple-audio-card";
                 simple-audio-card,format = "i2s";
                 simple-audio-card,name = "rockchip,rt5640-codec";

```

## How to adjust HDMI output resolution?

AIO-3399Pro-JD4 HDMI cannot automatically identify display resolution temporary. Use command below to switch 4K resoultion and flash screen:

```
# 4k60:
setprop persist.sys.resolution.main 3840x2160@60
# Set sys.display.timeline base on the default num to plus one for updating resolution
setprop sys.display.timeline 1
```