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

The official MIPI firmware default support MIPI_DSI1+HDMI display, MIPI screen connected to ROC-RK3566-PC MIPI_DSI1 interface. Below is the link to the firmware: [Firmware link](http://en.t-firefly.com/doc/download/109.html#other_420)

**NOTE:** When using HDMI display, there may be black edges on both sides of HDMI. The reason is HDMI as a secondary screen, will be scaled according to the aspect ratio of the main screen MIPI. If the aspect ratio of the two is inconsistent, it will lead to black edges.

### Compile command

#### MIPI_DSI1 + HDMI

Using official SDK to compile firmware that support 10.1 inches screen firmware need the following command, this compiled firmware is MIPI_DSI1 + HDMI display + CAM-2MS2M camera:

* Android
```bash
./FFTools/make.sh -d rk3568-firefly-aioj-ipc-mipi_M101014_BE45_A1 -j8 -l rk3568_firefly_aioj_ipc-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj_ipc-userdebug
```

* Linux
```bash
./build.sh ipc-m10r800-a3568j-ubuntu.mk
./build.sh
```

#### MIPI_DSI0

* Android

If you need to use MIPI_DSI0 or MIPI_DSI1, you can refer to `kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj-mipi101_M101014_BE45_A1.dts`. By default, it supports `MIPI_DSI0 + HDMI` + CAM-8MS1M camera module. The compilation command is:
```bash
./FFTools/make.sh -d rk3568-firefly-aioj-mipi101_M101014_BE45_A1 -j8 -l rk3568_firefly_aioj_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj_mipi-userdebug
```

**<font color=red>Note：If need to use CAM-2MS2M camera，first to add patch。** </font>

```diff
diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj-mipi101_M101014_BE45_A1.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj-mipi101_M101014_BE45_A1.dts
index c3e14e5c031..71f39d23c3f 100755
--- a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj-mipi101_M101014_BE45_A1.dts
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj-mipi101_M101014_BE45_A1.dts
@@ -13,8 +13,8 @@
  * using dual camera gc2053/gc2093   ----> rk3568-firefly-aioj-cam-2ms2m.dtsi
  * using hdmi-in module rk628d   ----> rk3568-firefly-aioj-tf-hdmi-mipi-rk628.dtsi
  */
-#include "rk3568-firefly-aioj-cam-8ms1m.dtsi"
-//#include "rk3568-firefly-aioj-cam-2ms2m.dtsi"
+//#include "rk3568-firefly-aioj-cam-8ms1m.dtsi"
+#include "rk3568-firefly-aioj-cam-2ms2m.dtsi"
 //#include "rk3568-firefly-aioj-tf-hdmi-mipi-rk628.dtsi"
```

* Linux

If you need to use MIPI_DSI0, then select one of the following mk files:
```bash
# Select one as you need:
# MIPI_DSI0 + CAM-8MS1M, buildroot
./build.sh aio-3568j-mipi-buildroot.mk

# MIPI_DSI0 + CAM-2MS2M, buildroot
./build.sh aio-3568j-mipi-2cam-buildroot.mk

# MIPI_DSI0 + CAM-8MS1M, ubuntu
./build.sh aio-3568j-mipi-ubuntu.mk

# MIPI_DSI0 + CAM-2MS2M, ubuntu
./build.sh aio-3568j-mipi-2cam-ubuntu.mk

# Start build
./build.sh
```

### Reference data

[[schematic of screen module Datasheet& adapter board]](http://en.t-firefly.com/doc/download/109.html#other_417)

### Real figure

#### MIPI_DSI1 FRONT
![](../../img/ROC-RK3566-PC/module_display_mipi_DSI1_front.jpg)
#### MIPI_DSI1 BACK
![](../../img/ROC-RK3566-PC/module_display_mipi_DSI1_back.jpg)
