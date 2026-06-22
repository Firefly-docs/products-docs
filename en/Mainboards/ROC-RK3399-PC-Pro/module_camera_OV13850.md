## [OV13850 Camera Moudle](https://www.firefly.store/products)<font color=#ff0000>(Out of production)</font> <br />

### Product Parameter

* Brand：Omnivision
* Model：CMK-OV13850
* Interface：MIPI
* Pixels：1320W


### Reference firmware

* CMK-OV13850 camera module is supported by the default firmware of Android 7.1.
* The public firmware of Android 10.0 must be manually modified DTS.

#### The modified method of 10.0

```
--- a/kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus.dtsi
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus.dtsi
 &i2c1{

-    XC6130b@23{
+    ov13850b@10{
     status = "okay";
     };
-    XC7022b@1b{
+    ov13850f@10{
     status = "okay";
     };

-    /delete-node/ ov13850b@10;
-    /delete-node/ ov13850f@10;
+    /delete-node/ XC6130b@23;
+    /delete-node/ XC7022b@1b;

 };

```

Modify according to the patch, recompile the kernel, then burn boot.img and reboot.


### Datasheet

[DataSheet and schematic of OV13850 Camera Module](http://download.t-firefly.com/product/RK3288/Docs/Peripherals/OV13850%20datasheet/Sensor_OV13850-G04A_OmniVision_SpecificationV1.pdf).

### Picture

![](../../../rk3399_img/module_camera_ov13850-1.jpg)

![](../../../rk3399_img/module_camera_ov13850-2.jpg)

### Connection Method

![](../../../rk3399_img/ROC-RK3399-PC-Pro/module_camera_connection.jpg)

### Renderings
![](../../../rk3399_img/module_camera_photographs.png)
