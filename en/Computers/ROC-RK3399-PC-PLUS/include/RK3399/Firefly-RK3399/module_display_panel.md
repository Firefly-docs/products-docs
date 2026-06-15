# Screen module

## 7.85-Inches EDP display module

### Product parameter

* Model: LP079QX1
* Size: 7.85-inches
* Resolution: 2048x1563
* Interface: EDP
* Panel Type: IPS
* Viewing Angle: 160°
* Touch Screen: Capacitive multi-touch

### Firmware follow

<font color=#ff0000>The name of official firmware to support the EDP7.85 Display Module with the words `EDP785`, the firmware link as followː</font> [Firmware link](https://drive.google.com/drive/folders/12EGY48-JtD3NPhTr60xrWpQ4ANw1d-2w?usp=sharing).

### Compilation command

Using official SDK to compile firmware that support 7.85 inch screen firmware need the following command：

* Android 7.1 (Box)

```
 ./FFTools/make.sh -j8 -d rk3399-firefly-edp -l rk3399_firefly_edp_box-userdebug
 ./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_edp_box-userdebug
```

### Datasheet

[Datasheet and schematic of 7.85 inch edp lcd module](https://pan.baidu.com/s/1mhXH4fu#list/path=%2F)

### Picture

![](../../../rk3399_img/Firefly-RK3399/module_display_edp.en.jpg)

### Connection method

![](../../../rk3399_img/Firefly-RK3399/module_display_edp_connection.en.jpg)

## 7.85-Inches MIPI display module

### Product parameter

* Model: B080XAN01
* Size: 7.85-inches
* Resolution: 1024x768
* Interface: MIPI
* Panel Type: IPS
* Viewing Angle: 160°
* Touch Screen: Capacitive multi-touch

### Firmware follow

<font color=#ff0000>The name of official firmware to support the MIPI7.85 Display Module with the words `MIPI785`, the firmware link as followː</font> [Firmware link](https://drive.google.com/drive/folders/1oSKLSGZiXIy15Im0dGGdRofU0L3z1nu8?usp=sharing).

### Compilation command

Using official SDK to compile firmware that support 7.85 inch screen firmware need the following command：

* Android 7.1 (Box)

```
 ./FFTools/make.sh -j8 -d rk3399-firefly-mipi -l rk3399_firefly_mipi_box-userdebug
 ./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_mipi_box-userdebug
```

### Picture

![](../../../rk3399_img/Firefly-RK3399/module_display_mipi_connection.en.jpg)