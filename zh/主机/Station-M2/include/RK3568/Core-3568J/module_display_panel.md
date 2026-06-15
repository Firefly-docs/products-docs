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

官方的MIPI固件默认支持MIPI_DSI1 + HDMI显示，MIPI屏连接ROC-RK3566-PC的MIPI_DSI1接口，下面是固件的链接：[固件链接](https://www.t-firefly.com/doc/download/125.html#other_492)

**注意:** 接入HDMI时，HDMI的两边有可能会有黒边的现象，这是因为HDMI作为副屏会根据主屏MIPI的宽高比进行缩放，如果两者的宽高比不一致，就会导致黒边。


### 编译命令

#### MIPI_DSI1 + HDMI

用官网 SDK 编译支持的 10.1 寸屏的固件时使用以下命令, 该固件支持`MIPI_DSI1+HDMI`显示 + CAM-2MS2M摄像头模组：

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

如需使用 MIPI_DSI0 可以参考`kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj-mipi101_M101014_BE45_A1.dts`, 默认支持`MIPI_DSI0 + HDMI`+ CAM-8MS1M 摄像头模组,  编译命令：
```bash
./FFTools/make.sh -d rk3568-firefly-aioj-mipi101_M101014_BE45_A1 -j8 -l rk3568_firefly_aioj_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj_mipi-userdebug
```

**<font color=red>注意：如果需要支持 CAM-2MS2M 摄像头模组，需要先做如下修改，再编译。** </font>

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

如需使用 MIPI_DSI0，则选择以下配置文件中的一个编译：
```bash
# 按需选择其中一个
# MIPI_DSI0 + CAM-8MS1M，buildroot 系统
./build.sh aio-3568j-mipi-buildroot.mk

# MIPI_DSI0 + CAM-2MS2M，buildroot 系统
./build.sh aio-3568j-mipi-2cam-buildroot.mk

# MIPI_DSI0 + CAM-8MS1M，ubuntu 系统
./build.sh aio-3568j-mipi-ubuntu.mk

# MIPI_DSI0 + CAM-2MS2M，ubuntu 系统
./build.sh aio-3568j-mipi-2cam-ubuntu.mk

# 编译
./build.sh
```

### 参考资料

[屏幕模组 Datasheet & 转接板原理图](https://www.t-firefly.com/doc/download/125.html#other_489)

### 实物图

#### MIPI_DSI1 正面
![](../../img/Station-M2/module_display_mipi_DSI1_front.jpg)
#### MIPI_DSI1 背面
![](../../img/Station-M2/module_display_mipi_DSI1_back.jpg)
