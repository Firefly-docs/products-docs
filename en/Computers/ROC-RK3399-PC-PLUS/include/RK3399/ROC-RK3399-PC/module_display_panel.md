# Screen module

## 10.1 "MIPI LCD module


Please refer to [LCD driver section](driver_lcd.md) for details

### Product parameters
* **model:** CZZC101863T05B
* **size:** 10.1"
* **resolution:** 800x1280
* **display interface:** MIPI
* **panel material:** IPS panel
* **visual Angle:** 160°
* **touch screen:** multi-point capacitive touch



### Compile command
 *  Android 7.1 Industry
```
 ./FFTools/make.sh -j8 -d rk3399-roc-pc-mipi101 -l rk3399_roc_pc_mipi-userdebug
 ./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_mipi-userdebug
```

### Real figure

![](../../../rk3399_img/ROC-RK3399-PC-PLUS/panel_mipi101.jpg)

## 8 "MIPI LCD module

Please refer to [LCD driver section](driver_lcd.md) for details


### Product parameters
* **model:** CZNB080860T
* **size:** 8"
* **resolution:** 800x1280
* **display interface:** MIPI
* **visual Angle:** 160°
* **touch screen:** multi-point capacitive touch


（<font color=#ff0000 size=3>Note：</font>The names of the 10.1-inch MIPI screen and the 8-inch MIPI screen are not different when they are generated. The firmware with MIPI101 or MIPI8 on the official website resources is a manually modified name after the package is generated, so that users can make a distinction）


### Compile command
Use the following command when compiling the 10.1-inch screen firmware supported by the official website SDK:
* Android 7.1 Industry
```
  ./FFTools/make.sh -j8 -d rk3399-roc-pc-mipi8 -l rk3399_roc_pc_mipi-userdebug
  ./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_mipi-userdebug
```


### Real figure
![](../../../rk3399_img/ROC-RK3399-PC-PLUS/panel_mipi8.jpg)

## 10.1"EDP LCD module

Please refer to [LCD driver section](driver_lcd.md) for details


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
* Android 7.1 Industry
```
 ./FFTools/make.sh -j8 -d rk3399-roc-pc-edp -l rk3399_roc_pc_edp-userdebug
 ./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_edp-userdebug
```



### Real figure
![](../../../rk3399_img/ROC-RK3399-PC-PLUS/panel_edp101.ipg)

