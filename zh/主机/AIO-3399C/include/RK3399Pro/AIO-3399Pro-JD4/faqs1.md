## AIO-3399Pro-JD4 Android9如何使用底板上的ALC5640 codec？
系统默认使用RK809 codec，若使用ALC5640则需要修改硬件。
I2S1信号脚位改接I2S0(Core-3399Pro-JD4没有引出I2S1)，I2C和I2S_CLK不要动，需要跳开R89~R93的电阻。

![](../../../rk3399_img/AIO-3399C/5640_sch.png)
![](../../../rk3399_img/AIO-3399C/5640_pcb.png)

软件修改如下:

```
**基于最新代码进行修改**
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

## HDMI 无法 4K 显示？

暂时无法支持自动切换 4K, 默认 1080P 显示, 后续会更新支持。可以使用以下命令手动设置 4K 分辨率后,重新插拔 HDMI 线：

```
# 设置4k60：
setprop persist.sys.resolution.main 3840x2160@60
# 每次设置完更新sys.display.timeline(每次加1)使分辨率生效
setprop sys.display.timeline 1
```