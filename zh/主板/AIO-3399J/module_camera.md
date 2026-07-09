# 摄像头模组

## [OV13850 摄像头模组](https://store.t-firefly.com/goods.php?id=6)<font color=#ff0000>(已停产)</font> <br />

### 产品参数

* 品牌：Omnivision
* 型号：CMK-OV13850
* 接口：MIPI
* 像素：1320W


### 参考固件

公版固件默认支持 CMK-OV13850 摄像头模组


### 技术资料

[OV13850 摄像头 DataSheet](http://download.t-firefly.com/product/RK3288/Docs/Peripherals/OV13850%20datasheet/Sensor_OV13850-G04A_OmniVision_SpecificationV1.pdf)

### 实物图

![](../../../rk3399_img/module_camera_ov13850-1.jpg)

![](../../../rk3399_img/module_camera_ov13850-2.jpg)

### 连接方法

![](../../../rk3399_img/AIO-3399J/module_camera_connection.jpg)

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
[Android7.1固件下载](https://www.t-firefly.com/doc/download/31.html#other_322)

[Android10.0固件下载](https://www.t-firefly.com/doc/download/31.html#other_426)

[Ubuntu18.04固件下载](https://www.t-firefly.com/doc/download/31.html#other_174)


### 实物图参考
![](../../../rk3399_img/cam_8ms1m_front.jpg)
![](../../../rk3399_img/cam_8ms1m_back.jpg)


### 连接方法
![](../../../rk3399_img/AIO-3399J/aio_3399j_8ms1m.jpg)



### 实拍图片
![](../../../rk3399_img/camera_8ms1m_shoot.jpg)


## SV-TAYSH-TQ摄像头模组

### 产品参数

* 型号：XC7022(RGB)/XC6130(IR)

* 接口：MIPI

* 像素：200W

### 修改方法 (手动修改)
<font color=#ff0000 size=4>**注意：**</font> <br />
<font color=#ff0000>该摄像头只支持AIO-3399J标准版，不支持HDMI_IN版本</font><br />
「 Android 7.1 」device/rockchip/rk3399/rk3399_firefly_aio.mk

```
 BOARD_NFC_SUPPORT := false
 BOARD_HAS_GPS := false
+BOARD_XC7022_XC6130_SUPPORT := true

 #for 3G/4G modem dongle support
 BOARD_HAVE_DONGLE := false
```
修改上述补丁后重新 [编译Android](compile_android7.1_industry_firmware.html#shou-dong-bian-yi-aio-3399j)并烧写 system.img 后重启。

「 Android 10  」kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-aio.dtsi
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
修改上述补丁后重新[编译内核](compile_android10.0_firmware.html#shou-dong-bian-yi-aio-3399j)并烧写 boot.img 后重启。




### 实物图
![](../../../rk3399_img/camera_SV-TAYSH-TQ.jpg)


### 连接方式

![](../../../rk3399_img/AIO-3399J/camera_SV-TAYSH-TQ_connect.jpg)

### 实拍图片

![](../../../rk3399_img/camera_SV-TAYSH-TQ_shoot.png)

