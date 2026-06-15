# 屏幕模组

## [10.1 寸LVDS屏模组](https://store.t-firefly.com/goods.php?id=80)

### 产品参数

* 型号：HSX101H40C-L28A
* 尺寸：10.1 寸
* 分辨率：800x1280
* 显示接口：LVDS
* 可视角度：170°
* 触摸屏：多点电容触摸

###  参考固件

**注意：** 支持 10.1 寸屏的官方固件名带有 `LVDS` 字样，下面是固件的链接：[固件链接](http://www.t-firefly.com/doc/download/page/id/48.html#other_317)

### 编译命令

用官网 SDK 编译支持的 10.1 寸屏的固件时使用以下命令：

```
./FFTools/make.sh -j8 -d rk3399pro-firefly-aioc-lvds-HSX101H40C -l rk3399pro_firefly_aioc_lvds-userdebug
./FFTools/mkupdate/mkupdate.sh
```


### 参考资料

[屏幕模组 Datasheet & 转接板原理图](http://www.t-firefly.com/doc/download/page/id/48.html#other_137)

### 实物图

* 注意：下图中电压跳线要使用12V


![](../../../rk3399_img/AIO-3399Pro-JD4/module_display_lvds.jpg)

## [DM-M10R800 V2 MIPI屏模组](https://item.taobao.com/item.htm?ft=t&id=655100190974)

### 产品参数

* 型号：M101014_BE45_A1
* 尺寸：10.1 寸
* 分辨率：800x1280
* 显示接口：MIPI
* 可视角度：160°
* 触摸屏：多点电容触摸

###  参考固件

**注意：** 支持 10.1 寸MIPI屏的官方固件名带有 `MIPI` 字样，下面是固件的链接：[固件链接](https://www.t-firefly.com/doc/download/125.html)

### 编译命令

用官网 SDK 编译支持的 10.1 寸MIPI屏的固件时使用以下命令：

```
./FFTools/make.sh -j8 -d rk3399pro-firefly-aioc-mipi_M101014_BE45_A1 -l rk3399pro_firefly_aioc_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aioc_mipi-userdebug
```


### 参考资料

[屏幕模组 Datasheet & 转接板原理图](https://www.t-firefly.com/doc/download/125.html#other_489)

### 实物连接图

![](../../../rk3399_img/AIO-3399Pro-JD4/module_display_mipi.jpg)