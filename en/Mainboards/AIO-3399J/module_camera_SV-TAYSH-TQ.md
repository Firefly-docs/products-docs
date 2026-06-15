## SV-TAYSH-TQ Camera module

### Product parameters

* Model：XC7022(RGB)/XC6130(IR)

* Interface ：MIPI

* Pixel ：200W

### Patch

<font color=#ff0000 size=4>**note:**</font> <br />
<font color=#ff0000>The camera only supports aio-3399j Standard Edition and does not support HDMI_In version</font><br />

「 Android 7.1 」device/rockchip/rk3399/rk3399_firefly_aio.mk

```
 BOARD_NFC_SUPPORT := false
 BOARD_HAS_GPS := false
+BOARD_XC7022_XC6130_SUPPORT := true

 #for 3G/4G modem dongle support
 BOARD_HAVE_DONGLE := false
```
 Modify the above patch and [complie Android](compile_android7.1_industry_firmware.html#overall-compilation), then reboot after upgrading system.img.

 [Android 10] kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-aio.dtsi
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
Modify the above patch and [complie kernel](compile_android10.0_firmware.html#step-by-step-compilation), then reboot after upgrading boot.img.

### Physical map
![](../../../rk3399_img/camera_SV-TAYSH-TQ.jpg)


### Connection method

![](../../../rk3399_img/AIO-3399J/camera_SV-TAYSH-TQ_connect.en.jpg)

### Real pictures

![](../../../rk3399_img/camera_SV-TAYSH-TQ_shoot.png)
