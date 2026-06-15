# Screen module

## 7.85 MIPI LCD module

**Note:** the AIO-3399J motherboard does not carry the mipi_dsi interface by default, and the hardware needs to be modified if this function is needed. Please refer to [LCD driver section](https://wiki.t-firefly.com/en/Core-3399-JD4/driver_lcd.html) for details.

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

*  Android 7.1 (Box)

```
./FFTools/make.sh -j8 -d rk3399-firefly-aiojd4-mipi -l rk3399_firefly_aiojd4_mipi_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aiojd4_mipi_box-userdebug
```

### Real figure

* **Note:** the voltage jumper in the figure below should use 12V.

![](../../../rk3399_img/AIO-3399J/module_display_mipi_connection.jpg)

## [10.1" LVDS module](https://www.firefly.store/products)

### Product parameters

* **model:** HSX101H40C-L28A
* **size:** 10.1 inch
* **resolution:** 800x1280
* **display interface:** LVDS
* **visual Angle:** 170°
* **touch screen:** multi-point capacitive touch

### Refer to the firmware

* Note: The official firmware name supporting the 10.1 screen has the word "LVDS". Below is the link to the firmware: 

[[Box Firmware link]](https://drive.google.com/drive/folders/1Qsf9x_XCCoK7LSQCe2VDBj1xJw4wyZ6w?usp=sharing)
[[Industry Firmware link]](https://drive.google.com/drive/folders/1wrfYkPdkBqWlD6a5kLyfg87c2Kwmmo36?usp=sharing)



### Compile command

Use the following command when compiling the 10.1-inch screen firmware supported by the official website SDK:

* Android 7.1 (Industry)
```
./FFTools/make.sh -j8 -d rk3399-firefly-aiojd4-lvds-HSX101H40C -l rk3399_firefly_aiojd4_lvds-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aiojd4_lvds-userdebug
```

* Android 7.1 (Box)
```
./FFTools/make.sh -j8 -d rk3399-firefly-aiojd4-lvds-HSX101H40C -l rk3399_firefly_aiojd4_lvds_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aiojd4_lvds_box-userdebug
```

### Reference data

[[schematic of screen module Datasheet& adapter board](https://drive.google.com/open?id=0B7HO8lbGgAqAcWlsQ3VSQzVTWTA)

### Real figure

* **Note:** the voltage jumper in the figure below should use 12V.
* Connection principle diagram:

![](../../../rk3399_img/AIO-3399J/module_display_lvds_new.jpg)


## 7.85 inch EDP LCD module

Note: The default AIO-3399J motherboard does not have a mipi_dsi interface. If you need this function, you need to modify the hardware. For details, please refer to [LCD Driver Chapter](driver_lcd.md)

###  Product parameter

* Model: LP079QX1
* Size: 7.85 inches
* Resolution: 2048x1563
* Display interface: EDP
* Panel material: IPS panel
* Viewing angle: 160°
* Touch screen: multi-point capacitive touch

###  Reference firmware

**Note:** The official firmware name that supports 7.85-inch screens has the words "EDP785", the following is the firmware link: [Firmware link](https://drive.google.com/drive/folders/19QYR0ZScFNywlS7_pXl_6H2y9L8X4vAI?usp=sharing)

### Compile command

When compiling the supported 7.85-inch screen firmware with the official website SDK, you need to use the following compile command:

* Android 7.1 (Box)

```
./FFTools/make.sh -j8 -d rk3399-firefly-aiojd4-edp -l rk3399_firefly_aiojd4_edp_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aiojd4_edp_box-userdebug
```

### Physical map

![](../../../rk3399_img/module_display_edp.jpg)