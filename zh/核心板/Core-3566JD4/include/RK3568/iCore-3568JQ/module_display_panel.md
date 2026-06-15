# 屏幕模组

## [DM-M10R800 V2 MIPI屏模组](https://item.taobao.com/item.htm?ft=t&id=655100190974)

### 产品参数

* 型号：M101014_BE45_A1
* 尺寸：10.1 寸
* 分辨率：800x1280
* 显示接口：MIPI
* 可视角度：160°
* 触摸屏：多点电容触摸

###  参考固件

官方的 MIPI 固件默认支持 MIPI_DSI1 + HDMI显示，MIPI屏连接 AIO-3566JD4 的MIPI_DSI1接口，下面是固件的链接：[固件链接](https://www.t-firefly.com/doc/download/125.html#other_604)

**注意:** 接入HDMI时，HDMI的两边有可能会有黒边的现象，这是因为HDMI作为副屏会根据主屏MIPI的宽高比进行缩放，如果两者的宽高比不一致，就会导致黒边。


### 编译命令

用官网 SDK 编译支持的 10.1 寸屏的固件时使用以下命令：

* Linux 直接指定对应的 mk 文件
```
./build.sh itx-3568q-mipi-ubuntu.mk
./build.sh
```
* Android

```
./FFTools/make.sh -d rk3568-firefly-itx-3568q-mipi101_M101014_BE45_A1 -j8 -l rk3568_firefly_itx_3568q-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_itx_3568q-userdebug
```

**<font color=red>注意：如果需要支持 CAM-8MS1M 摄像头模组，需要先做如下修改，再编译。** </font>

```
    diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-itx-3568q.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-itx-3568q.dts
    index d784287..fbe7b6b 100755
    --- a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-itx-3568q.dts
    +++ b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-itx-3568q.dts
    @@ -21,8 +21,8 @@
      * using dual camera gc2053/gc2093   ----> rk3568-firefly-itx-3568q-cam-2ms2m.dtsi
      * using hdmi-in module rk628d   ----> rk3568-firefly-itx-3568q-tf-hdmi-mipi-rk628.dtsi
      */
    -//#include "rk3568-firefly-itx-3568q-cam-8ms1m.dtsi"
    -#include "rk3568-firefly-itx-3568q-cam-2ms2m.dtsi"
    +#include "rk3568-firefly-itx-3568q-cam-8ms1m.dtsi"
    +//#include "rk3568-firefly-itx-3568q-cam-2ms2m.dtsi"
     //#include "rk3568-firefly-itx-3568q-tf-hdmi-mipi-rk628.dtsi"
```

### 参考资料

[屏幕模组 Datasheet & 转接板原理图](https://www.t-firefly.com/doc/download/125.html#other_489)

### 连接方式

![](../../img/Core-3566JD4/module_display_mipi_DSI.jpg)

