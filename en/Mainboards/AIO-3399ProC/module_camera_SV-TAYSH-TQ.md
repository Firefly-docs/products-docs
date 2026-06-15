## SV-TAYSH-TQ Camera module

### Product parameters

* Model：XC7022(RGB)/XC6130(IR)

* Interface ：MIPI

* Pixel ：200W

### Patch
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
Modify the above patch and [complie kernel](https://wiki.t-firefly.com/en/AIO-3399ProC/compile_android9.0_firmware.html#manually-compile-aio-3399proc-android-9-0), then reboot after upgrading boot.img.

### Physical map
![](../../../rk3399_img/camera_SV-TAYSH-TQ.jpg)


### Connection method

![](../../../rk3399_img/AIO-3399ProC/camera_SV-TAYSH-TQ_connect.en.jpg)

### Real pictures

![](../../../rk3399_img/camera_SV-TAYSH-TQ_shoot.png)
