# 屏幕模组

## [DM-M10R800 V2 MIPI屏模组](https://item.taobao.com/item.htm?ft=t&id=655100190974)

### 产品参数

* 型号：M101014_BE45_A1
* 尺寸：10.1寸
* 分辨率：800x1280
* 显示接口：MIPI
* 面板材料：IPS面板
* 可视角度：160°
* 触摸屏：多点电容触摸

###  参考固件


支持10.1寸屏的官方固件名带有`MIPI`字样，下面是固件的链接：[固件链接](https://www.t-firefly.com/doc/download/125.html#other_490)  
  
如需使用双屏显示，请先参考[FAQ](faqs.html#rk3566-shuang-ping-xian-shi-wen-ti)

### 编译命令

用官网SDK编译支持的10.1 寸mipi屏的固件时使用以下命令：

```
./FFTools/make.sh -d rk3566-roc-pc-mipi101_M101014_BE45_A1 -j8 -l rk3566_roc_pc_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3566_roc_pc_mipi-userdebug
```

**<font color=red>注意：如果需要支持 CAM-8MS1M 摄像头模组，需要先做如下修改，再编译。** </font>

```
diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3566-roc-pc-mipi101_M101014_BE45_A1.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3566-roc-pc-mipi101_M101014_BE45_A1.dts
index ebbb5d1123f..71e82f8d9c0 100644
--- a/kernel/arch/arm64/boot/dts/rockchip/rk3566-roc-pc-mipi101_M101014_BE45_A1.dts
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk3566-roc-pc-mipi101_M101014_BE45_A1.dts
@@ -7,6 +7,13 @@
  /dts-v1/;

 #include "rk3566-roc-pc.dtsi"
+/*
+ * Select one of the two
+ * using single camera xc7160 ----> rk3566-roc-pc-cam-8ms1m.dtsi
+ * using dual camera gc2053/gc2093   ----> rk3566-roc-pc-cam-2ms2m.dtsi
+ */
+#include "rk3566-roc-pc-cam-8ms1m.dtsi"
+//#include "rk3566-roc-pc-cam-2ms2m.dtsi"

 / {
     model = "ROC-RK3566-PC MIPI101 M101014-BE45-A1(Android)";
```

### 参考资料

[屏幕模组 Datasheet & 转接板原理图](https://www.t-firefly.com/doc/download/125.html#other_489)

### 实物图

#### 正面

![](../../img/ROC-RK3568-PC/mipi101_v2_M101014_BE45_A1_front.jpg)

#### 背面

![](../../img/ROC-RK3568-PC/mipi101_v2_M101014_BE45_A1_back.jpg)