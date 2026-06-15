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

官方的MIPI固件默认支持MIPI_DSI0 + HDMI显示，MIPI屏连接CQ38W-3576的MIPI_DSI0接口，下面是固件的链接：[固件链接](https://www.t-firefly.com/doc/download/125.html#other_492)

**注意:** 接入HDMI时，HDMI的两边有可能会有黒边的现象，这是因为HDMI作为副屏会根据主屏MIPI的宽高比进行缩放，如果两者的宽高比不一致，就会导致黒边。


### 编译命令

用官网 SDK 编译支持的 10.1 寸屏的固件时使用以下命令, 默认编译的固件为MIPI_DSI0+HDMI显示：

```
./FFTools/make.sh -d rk3588-firefly-aioj-mipi101_M101014_BE45_A1 -j8 -l rk3568_firefly_aioj_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3588_firefly_aioj_mipi-userdebug
```

**<font color=red>注意：如果需要支持 CAM-8MS1M 摄像头模组，需要先做如下修改，再编译。** </font>

```
diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3588-firefly-aioj-mipi101_M101014_BE45_A1.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj-mipi101_M101014_BE45_A1.dts
index 20f1c1af636..4730a4e8dce 100755
--- a/kernel/arch/arm64/boot/dts/rockchip/rk3588-firefly-aioj-mipi101_M101014_BE45_A1.dts
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk3588-firefly-aioj-mipi101_M101014_BE45_A1.dts
@@ -7,6 +7,15 @@
  /dts-v1/;

 #include "rk3588-firefly-aioj.dtsi"
+/*
+ * Select one of the three
+ * using single camera xc7160 ----> rk3588-firefly-aioj-cam-8ms1m.dtsi
+ * using dual camera gc2053/gc2093   ----> rk3588-firefly-aioj-cam-2ms2m.dtsi
+ * using hdmi-in module rk628d   ----> rk3588-firefly-aioj-tf-hdmi-mipi-rk628.dtsi
+ */
+#include "rk3588-firefly-aioj-cam-8ms1m.dtsi"
+//#include "rk3588-firefly-aioj-cam-2ms2m.dtsi"
+//#include "rk3588-firefly-aioj-tf-hdmi-mipi-rk628.dtsi"

 / {
     model = "ITX-3588J MIPI M101014-BE45-A1 + HDMI(Android)";
```

### 参考资料

[屏幕模组 Datasheet & 转接板原理图](https://www.t-firefly.com/doc/download/125.html#other_489)

### 实物图

#### MIPI_DSI0 正面
![](../../../rk3576_img/CQ38W-3576/module_display_mipi_DSI0_front.jpg)
#### MIPI_DSI0 背面
![](../../../rk3576_img/CQ38W-3576/module_display_mipi_DSI0_back.jpg)

