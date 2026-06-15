## SV-TAYSH-TQ摄像头模组

### 产品参数

* 型号：XC7022(RGB)/XC6130(IR)

* 接口：MIPI

* 像素：200W

### 修改方法 (手动修改)
kernel/arch/arm64/boot/dts/rockchip/rk3399pro-firefly-aiojd4.dtsi
```
        xc7160b: xc7160b@1b {
+                status = "disabled";
                reset-gpios = <&gpio1 RK_PC4 GPIO_ACTIVE_HIGH>;
        };

        xc7160f: xc7160f@1b {
+                status = "disabled";
                reset-gpios = <&gpio1 RK_PC4 GPIO_ACTIVE_HIGH>;
        };

        XC7022b: XC7022b@1b{
+                status = "okay";
                reset-gpios = <&gpio0 RK_PB5 GPIO_ACTIVE_HIGH>;
        };

        XC6130b: XC6130b@23{
+                status = "okay";
                reset-gpios = <&gpio0 RK_PB5 GPIO_ACTIVE_HIGH>;
        };

```
修改上述补丁后重新[编译内核](compile_android9.0_firmware.html#shou-dong-bian-yi-aio-3399pro-jd4-android-9-0)并烧写 boot.img 后重启。


### 实物图
![](../../../rk3399_img/camera_SV-TAYSH-TQ.jpg)


### 连接方式

![](../../../rk3399_img/AIO-3399Pro-JD4/camera_SV-TAYSH-TQ_connect.jpg)

### 实拍图片

![](../../../rk3399_img/camera_SV-TAYSH-TQ_shoot.png)
