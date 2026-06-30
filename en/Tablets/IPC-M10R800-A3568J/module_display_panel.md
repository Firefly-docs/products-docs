# Screen module

## [DM-M10R800 V2 MIPI module](https://www.firefly.store/products/dm-m10r800-v2)

### Product parameters

* **model:** M101014_BE45_A1
* **size:** 10.1 inch
* **resolution:** 800x1280
* **display interface:** MIPI
* **visual Angle:** 160°
* **touch screen:** multi-point capacitive touch

### Refer to the firmware

The official MIPI firmware default support MIPI_DSI0+HDMI display, MIPI screen connected to AIO-3568J MIPI_DSI0 interfacei. Below is the link to the firmware: [Firmware link](https://en.t-firefly.com/doc/download/109.html#other_561)

**NOTE:** When using HDMI display, there may be black edges on both sides of HDMI. The reason is HDMI as a secondary screen, will be scaled according to the aspect ratio of the main screen MIPI. If the aspect ratio of the two is inconsistent, it will lead to black edges.

### Compile command

Using official SDK to compile firmware that support 10.1 inches screen firmware need the following command, the default compiled firmware is MIPI_DSI0+HDMI display:
```
./FFTools/make.sh -d rk3568-firefly-roc-pc-se-mipi101_M101014_BE45_A1 -j8 -l rk3568_firefly_roc_pc_se-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_roc_pc_se-userdebug
```

**<font color=red>Note：If need to use CAM-8MS1M camera，first to add patch。** </font>

```
diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc-se-mipi101_M101014_BE45_A1.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc-se-mipi101_M101014_BE45_A1.dts
index ae57644a105..0cb9124bec8 100644
--- a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc-se-mipi101_M101014_BE45_A1.dts
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc-se-mipi101_M101014_BE45_A1.dts
@@ -7,6 +7,15 @@
  /dts-v1/;

 #include "rk3568-firefly-roc-pc-se.dtsi"
+/*
+ * Select one of the three
+ * using single camera xc7160 ----> rk3568-firefly-roc-pc-se-cam-8ms1m.dtsi
+ * using dual camera gc2053/gc2093   ----> rk3568-firefly-roc-pc-se-cam-2ms2m.dtsi
+ */
+#include "rk3568-firefly-roc-pc-se-cam-8ms1m.dtsi"
+//#include "rk3568-firefly-roc-pc-se-cam-2ms2m.dtsi"

 / {
     model = "ROC-RK3568-PC-SE MIPI M101014_BE45_A1 + HDMI(Android)";
```

### Reference data

[[schematic of screen module Datasheet& adapter board]](http://en.t-firefly.com/doc/download/109.html#other_417)

### Real figure

![](../../../rk356x_img/IPC-M10R800-A3568J/module_display_mipi_DSI0.jpg)