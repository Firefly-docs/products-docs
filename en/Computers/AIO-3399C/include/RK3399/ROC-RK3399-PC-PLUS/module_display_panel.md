# Screen module

## 8 "MIPI LCD module

[LCD driver section](https://wiki.t-firefly.com/en/ROC-RK3399-PC-PLUS/driver_lcd.html) for details

### Product parameters
* **model:** CZNB080860T
* **size:** 8"
* **resolution:** 800x1280
* **display interface:** MIPI
* **visual Angle:** 160°
* **touch screen:** multi-point capacitive touch


### Compile command
Use the following command when compiling the 10.1-inch screen firmware supported by the official website SDK:
* Android 7.1 (industry)
```
  ./FFTools/make.sh -j8 -d rk3399-roc-pc-plus-mipi8 -l rk3399_roc_pc_plus_mipi-userdebug
  ./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus_mipi-userdebug
```


### Real figure
![](../../../rk3399_img/AIO-3399C/panel_mipi8.jpg)

## 10.1 "MIPI LCD module


Please refer to [LCD driver section](https://wiki.t-firefly.com/en/ROC-RK3399-PC-PLUS/driver_lcd.html) for details

### Product parameters
* **model:** CZZC101863T05B
* **size:** 10.1"
* **resolution:** 800x1280
* **display interface:** MIPI
* **panel material:** IPS panel
* **visual Angle:** 160°
* **touch screen:** multi-point capacitive touch



### Compile command
 *  Android 7.1 (industry)
```
 ./FFTools/make.sh -j8 -d rk3399-roc-pc-plus-mipi101_id_detect -l rk3399_roc_pc_plus_mipi101-userdebug
 ./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus_mipi101-userdebug
```

### Real figure

![](../../../rk3399_img/AIO-3399C/pc+_mipi101_2.jpg)

### Firmware Download 
* [Google Driver](https://drive.google.com/drive/folders/1q3O9P9b2ELUSuL4bNWTxoU7YoEg2FnTX?usp=sharing)


## [DM-M10R800 V2 MIPI module](https://www.firefly.store/products/dm-m10r800-v2)

### Product parameters

* **model:** M101014_BE45_A1
* **size:** 10.1 inch
* **resolution:** 800x1280
* **display interface:** MIPI
* **visual Angle:** 160°
* **touch screen:** multi-point capacitive touch

### Refer to the firmware

* Note: The official firmware name supporting the 10.1 MIPI screen has the word "MIPI". Below is the link to the firmware: [Firmware link](https://en.t-firefly.com/doc/download/109.html#other_428)

### Compile command
```
./FFTools/make.sh -j8 -d rk3399-roc-pc-plus-mipi101-JDM101014_BC45_A1 -l rk3399_roc_pc_plus_mipi101-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus_mipi101-userdebug
```

### Real figure
	
![](../../../rk3399_img/AIO-3399C/pc+_mipi101_v2.jpg)

### Reference data

[[schematic of screen module Datasheet& adapter board]](http://en.t-firefly.com/doc/download/109.html#other_417)

## 10.1"EDP LCD module

[LCD driver section](driver_lcd.md) for details

### Product parameters
* **model:** LP079QX1
* **size:** 10.1"
* **resolution:** 2560x1600
* **display interface:** EDP
* **panel material:** IPS panel
* **visual Angle:** 160°
* **touch screen:** multi-point capacitive touch



###  Compile command
Use the following command when compiling the 10.1-inch screen firmware supported by the official website SDK:
* Android 7.1 (industry)
```
 ./FFTools/make.sh -j8 -d rk3399-roc-pc-plus-edp -l rk3399_roc_pc_plus_edp-userdebug
 ./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus_edp-userdebug
```



### Real figure
![](../../../rk3399_img/AIO-3399C/panel_edp101.jpg)
