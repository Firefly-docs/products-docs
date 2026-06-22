# 摄像头模组

## [OV13850 摄像头模组](https://store.t-firefly.com/goods.php?id=6)<font color=#ff0000>(已停产)</font> <br />

### 产品参数

* 品牌：Omnivision
* 型号：CMK-OV13850
* 接口：MIPI
* 像素：1320W


### 参考固件

* Android 7.1 公版固件默认支持 CMK-OV13850 摄像头模组
* Android 10.0 公版固件须手动修改DTS

#### 10.0 固件修改方法

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
修改以上补丁后重新编译内核并烧写boot.img后重启。


### 技术资料

[OV13850 摄像头 DataSheet](http://download.t-firefly.com/product/RK3288/Docs/Peripherals/OV13850%20datasheet/Sensor_OV13850-G04A_OmniVision_SpecificationV1.pdf)

### 实物图

![](../../../rk3399_img/module_camera_ov13850-1.jpg)

![](../../../rk3399_img/module_camera_ov13850-2.jpg)

### 连接方法

![](../../../rk3399_img/ROC-RK3399-PC-PLUS/module_camera_connection.jpg)

### 实拍图片

![](../../../rk3399_img/module_camera_photographs.png)

## [CAM-8MS1M 单目摄像头模组](https://item.taobao.com/item.htm?ft=t&id=659032651408)

### 产品参数
* **品牌**：SV
* **ISP**：XC7160
* **Sensor**: SC8238
* **接口**: MIPI
* **像素**: 800W(当前仅支持1080P，4K仍在适配中)

### 规格书
[CAM-8MS1M_800万单目MIPI摄像模组_规格书](https://www.t-firefly.com/doc/download/131.html#other_515)

### 参考固件
公版固件默认支持 CAM-8MS1M 单目摄像头模组。若无法使用单目摄像头 CAM-8MS1M，请更新固件
[Android7.1固件下载](https://www.t-firefly.com/doc/download/85.html#other_339)

[Android10.0固件下载](https://www.t-firefly.com/doc/download/85.html#other_400)

### 实物图参考
![](../../../rk3399_img/cam_8ms1m_front.jpg)
![](../../../rk3399_img/cam_8ms1m_back.jpg)


### 连接方法
![](../../../rk3399_img/ROC-RK3399-PC-PLUS/roc_rk3399_pc_plus_8ms1m.jpg)



### 实拍图片
![](../../../rk3399_img/camera_8ms1m_shoot.jpg)


## SV-TAYSH-TQ摄像头模组

### 产品参数

* 型号：XC7022(RGB)/XC6130(IR)

* 接口：MIPI

* 像素：200W

### 修改方法 (手动修改)
「 Android 7.1 」device/rockchip/rk3399/rk3399_roc_pc_plus.mk

```
 BOARD_NFC_SUPPORT := false
 BOARD_HAS_GPS := false
+BOARD_XC7022_XC6130_SUPPORT := true

 #for 3G/4G modem dongle support
 BOARD_HAVE_DONGLE := false
```
修改上述补丁后重新 [编译Android](compile_android7.1_industry_firmware.html#zheng-ti-bian-yi)并烧写 system.img 后重启。

「 Android 10  」kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus.dtsi
```
    xc7160b@1b{
+    status = "disabled";
    };
    xc7160f@1b{
+    status = "disabled";
    };

    XC6130b@23{
+    status = "okay";
    };
    XC7022b@1b{
+    status = "okay";
    };
```
修改上述补丁后重新[编译内核](compile_android10.0_firmware.html#fen-bu-bian-yi)并烧写 boot.img 后重启。





### 实物图
![](../../../rk3399_img/camera_SV-TAYSH-TQ.jpg)


### 连接方式

![](../../../rk3399_img/ROC-RK3399-PC-PLUS/camera_SV-TAYSH-TQ_connect.jpg)

### 实拍图片

![](../../../rk3399_img/camera_SV-TAYSH-TQ_shoot.png)

