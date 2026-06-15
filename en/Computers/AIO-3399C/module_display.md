
# Screen module

## 7.85 MIPI LCD module

**Note :** the AIO-3399C motherboard does not carry the mipi_dsi interface by default, and the hardware needs to be modified if this function is needed. Please refer to [LCD driver section](http://wiki.t-firefly.com/AIO-3399C/driver_lcd.md) for details.

### Product parameters

* **model:** B080XAN01
* **size:** 7.85 inch
* **resolution:** 1024x768
* **display interface:** MIPI
* **panel material:** IPS panel
* **visual Angle:** 160°
* **touch screen:** multi-point capacitive touch

### Compile command

*  Android 7.1 (Industry)

```
./FFTools/make.sh -j8 -d rk3399-firefly-aioc-mipi -l rk3399_firefly_aioc_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_mipi-userdebug
```

*  Android 7.1 (Box)

```
./FFTools/make.sh -j8 -d rk3399-firefly-aioc-mipi -l rk3399_firefly_aioc_mipi_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_mipi_box-userdebug
```

### Real figure

* **Note:** the voltage jumper in the figure below should use 12V.

![](../../../rk3399_img/AIO-3399C/module_display_mipi_connection.en.jpg)

## [10.1" LVDS module](https://www.firefly.store/products)

### Product parameters

* **model:** HSX101H40C-L28A
* **size:** 10.1 inch
* **resolution:** 800x1280
* **display interface:** LVDS
* **visual Angle:** 170°
* **touch screen:** multi-point capacitive touch

### Refer to the firmware

* Note: The official firmware name supporting the 10.1 screen has the word "LVDS". Below is the link to the firmware: [Firmware link](http://en.t-firefly.com/doc/download/page/id/45.html#other_120)

### Compile command

Use the following command when compiling the 10.1-inch screen firmware supported by the official website SDK:

* Android 7.1 (Industry)

1.std version

```
./FFTools/make.sh -j8 -d rk3399-firefly-aioc-lvds-HSX101H40C -l rk3399_firefly_aioc_lvds-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_lvds-userdebug
```

2.AI version

```
./FFTools/make.sh -j8 -d rk3399-firefly-aioc-ai-lvds-HSX101H40C -l rk3399_firefly_aioc_ai_lvds-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_lvds-userdebug
```

* Android 7.1 (Box)

1.std version

```
./FFTools/make.sh -j8 -d rk3399-firefly-aioc-lvds-HSX101H40C -l rk3399_firefly_aioc_lvds_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_lvds_box-userdebug
```

2.AI version

```
./FFTools/make.sh -j8 -d rk3399-firefly-aioc-ai-lvds-HSX101H40C -l rk3399_firefly_aioc_ai_lvds_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_lvds_box-userdebug
```

* Android 8.1

```
./FFTools/make.sh -j8 -d rk3399-firefly-aioc-lvds-HSX101H40C -l rk3399_firefly_aioc_lvds_mid-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_lvds_mid-userdebug
```

### Reference data

[[schematic of screen module Datasheet& adapter board]](http://en.t-firefly.com/doc/download/page/id/45.html#other_101)

### Real figure

* **Note:** the voltage jumper in the figure below should use 12V.
![](../../../rk3399_img/AIO-3399C/module_display_lvds.en.png)

## 7.85 "EDP LCD module

### Product parameters

* **model:** LP079QX1
* **size:** 7.85 inch
* **resolution:** 2048x1563
* **display interface:** EDP
* **panel material:** IPS panel
* **visual Angle:** 160°
* **touch screen:** multi-point capacitive touch

###  Refer to the firmware

**Note:** The official firmware name supporting the 7.85 screen has the word "EDP785". Below is the link to the firmware : [Firmware link](https://pan.baidu.com/s/1k69vkfYrDJCKBTinsT1FaA)

###  Compile command

Use the following command when compiling the 7.85-inch screen firmware supported by the official website SDK:

* Android 7.1 (Industry)

```
 ./FFTools/make.sh -j8 -d rk3399-firefly-aioc-edp -l rk3399_firefly_aioc_edp-userdebug
  /FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_edp-userdebug
```

* Android 7.1 (Box)

```
 ./FFTools/make.sh -j8 -d rk3399-firefly-aioc-edp -l rk3399_firefly_aioc_edp_box-userdebug
  /FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_edp_box-userdebug
```

### Technical data

[7.85" screen module DataSheet & Floor schematic diagram](https://pan.baidu.com/s/1mhXH4fu#list/path=%2F)

### Real figure

![](../../../rk3399_img/AIO-3399C/module_display_edp.en.jpg)

### Connection methods

**Note:** the voltage jumper in the figure below will use 5V, and a wrong connection to 12V will burn the screen chip

![](../../../rk3399_img/AIO-3399C/module_display_edp_connection.en.png)
