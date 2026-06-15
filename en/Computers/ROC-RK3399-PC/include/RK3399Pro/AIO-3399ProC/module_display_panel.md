# Screen module

## [10.1" LVDS module](https://www.firefly.store/products)

### Product parameters

* **model:** HSX101H40C-L28A
* **size:** 10.1 inch
* **resolution:** 800x1280
* **display interface:** LVDS
* **visual Angle:** 170°
* **touch screen:** multi-point capacitive touch

### Refer to the firmware

* Note: The official firmware name supporting the 10.1 screen has the word "LVDS". Below is the link to the firmware: [Firmware link](http://en.t-firefly.com/doc/download/page/id/45.html#other_283)

### Compile command

```
./FFTools/make.sh -j8 -d rk3399pro-firefly-aioc-lvds-HSX101H40C -l rk3399pro_firefly_aioc_lvds-userdebug
./FFTools/mkupdate/mkupdate.sh
```

### Reference data

[[schematic of screen module Datasheet& adapter board]](http://en.t-firefly.com/doc/download/page/id/45.html#other_101)

### Real figure

* Note: The voltage jumper in the figure below should use 12V

![](../../../rk3399_img/ROC-RK3399-PC/module_display_lvds.en.jpg)

## [DM-M10R800 V2 MIPI module](https://www.firefly.store/products/dm-m10r800-v2)

### Product parameters

* **model:** M101014_BE45_A1
* **size:** 10.1 inch
* **resolution:** 800x1280
* **display interface:** MIPI
* **visual Angle:** 160°
* **touch screen:** multi-point capacitive touch

### Refer to the firmware

* Note: The official firmware name supporting the 10.1 MIPI screen has the word "MIPI". Below is the link to the firmware: [Firmware link](http://en.t-firefly.com/doc/download/109.html)

### Compile command

```
./FFTools/make.sh -j8 -d rk3399pro-firefly-aioc-mipi_M101014_BE45_A1 -l rk3399pro_firefly_aioc_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aioc_mipi-userdebug
```

### Reference data

[[schematic of screen module Datasheet& adapter board]](http://en.t-firefly.com/doc/download/109.html#other_417)

### Real figure

![](../../../rk3399_img/ROC-RK3399-PC/module_display_mipi.jpg)