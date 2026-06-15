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

![](../../../rk3399_img/AIO-3399ProC/module_camera_connection.jpg)

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
[Android 9.0 固件下载](https://www.t-firefly.com/doc/download/76.html#other_316)


### 实物图参考
![](../../../rk3399_img/cam_8ms1m_front.jpg)
![](../../../rk3399_img/cam_8ms1m_back.jpg)


### 连接方法
![](../../../rk3399_img/AIO-3399ProC/aio_3399proc_8ms1m.jpg)



### 实拍图片
![](../../../rk3399_img/camera_8ms1m_shoot.jpg)


## SV-TAYSH-TQ摄像头模组

### 产品参数

* 型号：XC7022(RGB)/XC6130(IR)

* 接口：MIPI

* 像素：200W

### 修改方法 (手动修改)
kernel/arch/arm64/boot/dts/rockchip/rk3399pro-firefly-aioc.dtsi
```
        xc7160b: xc7160b@1b {
+                status = "disabled";
        };

        xc7160f: xc7160f@1b {
+                status = "disabled";
        };

        XC7022b: XC7022b@1b{
+                status = "okay";
        };

        XC6130b: XC6130b@23{
+                status = "okay";
        };
```
修改上述补丁后重新[编译内核](compile_android9.0_firmware.html#shou-dong-bian-yi-aio-3399proc-android-9-0)并烧写 boot.img 后重启。



### 实物图
![](../../../rk3399_img/camera_SV-TAYSH-TQ.jpg)


### 连接方式

![](../../../rk3399_img/AIO-3399ProC/camera_SV-TAYSH-TQ_connect.jpg)

### 实拍图片

![](../../../rk3399_img/camera_SV-TAYSH-TQ_shoot.png)

