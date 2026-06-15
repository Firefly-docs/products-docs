# Screen module

## 7.85 MIPI LCD module

**Note :** the AIO-3399Pro-JD4 motherboard does not carry the mipi_dsi interface by default, and the hardware needs to be modified if this function is needed. Please refer to [LCD driver section](https://wiki.t-firefly.com/en/AIO-3399J/driver_lcd.html) for details.

### Product parameters

* **model:** B080XAN01
* **size:** 7.85 inch
* **resolution:** 1024x768
* **display interface:** MIPI
* **panel material:** IPS panel
* **visual Angle:** 160°
* **touch screen:** multi-point capacitive touch

### Compile command

Using official SDK to compile firmware that support 7.85 inch screen firmware need the patch ([0001-AIO-SDK-mipi-support-7.85-MIPI-lcd.patch](https://drive.google.com/drive/folders/1LXI8JA_zVUKJ-XHH3mZ5rC09DWkBzZBv?usp=sharing) ) , and then, execute the following command：

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

![](../../../rk3399_img/AIO-3399Pro-JD4/module_display_mipi_connection.en.jpg)

## [10.1" LVDS module](https://www.firefly.store/products)

### Product parameters

* **model:** HSX101H40C-L28A
* **size:** 10.1 inch
* **resolution:** 800x1280
* **display interface:** LVDS
* **visual Angle:** 170°
* **touch screen:** multi-point capacitive touch

### Refer to the firmware

* Note: The official firmware name supporting the 10.1 screen has the word "LVDS". Below is the link to the firmware: [Firmware link](https://en.t-firefly.com/doc/download/31.html#other_233)

### Compile command

Use the following command when compiling the 10.1-inch screen firmware supported by the official website SDK:

* Android 7.1 (Industry)
```
./FFTools/make.sh -j8 -d rk3399-firefly-aio-lvds-HSX101H40C -l rk3399_firefly_aio_lvds-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aio_lvds-userdebug
```

* Android 7.1 (Box)
```
./FFTools/make.sh -j8 -d rk3399-firefly-aio-lvds-HSX101H40C -l rk3399_firefly_aio_lvds_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aio_lvds_box-userdebug
```

### Reference data

[[schematic of screen module Datasheet& adapter board]](http://en.t-firefly.com/doc/download/page/id/45.html#other_101)

### Real figure

**Note:** the voltage jumper in the figure below should use 12V.

* Old version wiring diagram:

![](../../../rk3399_img/AIO-3399Pro-JD4/module_display_lvds_old.en.jpg)

* New version wiring diagram:

![](../../../rk3399_img/AIO-3399Pro-JD4/module_display_lvds_new.en.jpg)
