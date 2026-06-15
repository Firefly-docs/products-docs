# Screen module

## [DM-M10R800 V2 MIPI module](https://item.taobao.com/item.htm?ft=t&id=655100190974)

### Product parameters

* **model:** M101014_BE45_A1
* **size:** 10.1 inch
* **resolution:** 800x1280
* **display interface:** MIPI
* **panel material:** IPS panel
* **visual Angle:** 160°
* **touch screen:** multi-point capacitive touch

### Refer to the firmware

The official firmware name supporting the 10.1 screen has the word `MIPI`. Below is the link to the firmware: [Dual MIPI Firmware link](https://en.t-firefly.com/doc/download/109.html#other_447)  

<font color="#dd0000">Note: </font><br />Because RK3566 Dual screen Display uses the same internal input Source, that is, `VOP` is the `Same Source, Dual Display`, so if MIPI screen is used as the main screen, the HDMI screen of the secondary screen will stretch, so if you  want to avoid this problem.  Both the primary and secondary screens should be in the same direction (horizontal and vertical) with the same resolution.  

### Compile command

```
./FFTools/make.sh -d rk3566-firefly-aiojd4-mipi101_M101014_BE45_A1 -j8 -l rk3566_firefly_aiojd4_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3566_firefly_aiojd4_mipi-userdebug
```

### Reference data

[[schematic of screen module Datasheet& adapter board]](http://en.t-firefly.com/doc/download/109.html#other_417)

### Real figure

#### Front
* MIPI DSI0
![](../../img/IPC-M10R800-A3568J/mipi101_v2_M101014_BE45_A1_front0.jpg)
* MIPI DSI1
![](../../img/IPC-M10R800-A3568J/mipi101_v2_M101014_BE45_A1_front1.jpg)
