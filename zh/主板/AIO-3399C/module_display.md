
# 屏幕模组

## 7.85 寸 MIPI 液晶屏模组

注意：默认的 AIO-3399C 主板不带 mipi_dsi 接口，如需要此功能需修改硬件。详情参考 [LCD 驱动章节](driver_lcd.md)

###  产品参数

* 型号：B080XAN01
* 尺寸： 7.85 寸
* 分辨率：1024x768
* 显示接口： MIPI
* 面板材料：IPS 面板
* 可视角度：160°
* 触摸屏：多点电容触摸

### 编译命令

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


### 实物图

**注意：** 下图中电压跳线要使用 12V。

![](../../../rk3399_img/AIO-3399C/module_display_mipi_connection.jpg)

## [10.1 寸LVDS屏模组](https://store.t-firefly.com/goods.php?id=80)

### 产品参数

* 型号：HSX101H40C-L28A
* 尺寸：10.1 寸
* 分辨率：800x1280
* 显示接口：LVDS
* 可视角度：170°
* 触摸屏：多点电容触摸

###  参考固件

**注意：** 支持 10.1 寸屏的官方固件名带有 `LVDS` 字样，下面是固件的链接：[固件链接](http://www.t-firefly.com/doc/download/page/id/48.html#other_147)

### 编译命令

用官网 SDK 编译支持的 10.1 寸屏的固件时使用以下命令：

*  Android 7.1 (Industry)

1. AIOC 标准版

        ```
        ./FFTools/make.sh -j8 -d rk3399-firefly-aioc-lvds-HSX101H40C -l rk3399_firefly_aioc_lvds-userdebug
        ./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_lvds-userdebug
        ```

2. AIOC AI 版

        ```
        ./FFTools/make.sh -j8 -d rk3399-firefly-aioc-ai-lvds-HSX101H40C -l rk3399_firefly_aioc_ai_lvds-userdebug
        ./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_lvds-userdebug
        ```

*  Android 7.1 (Box)

1. AIOC 标准版

	```
	./FFTools/make.sh -j8 -d rk3399-firefly-aioc-lvds-HSX101H40C -l rk3399_firefly_aioc_lvds_box-userdebug
	./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_lvds_box-userdebug
	```

2. AIOC AI 版

	```
	./FFTools/make.sh -j8 -d rk3399-firefly-aioc-ai-lvds-HSX101H40C -l rk3399_firefly_aioc_ai_lvds_box-userdebug
	./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_lvds_box-userdebug
	```

*    Android 8.1

```
./FFTools/make.sh -j8 -d rk3399-firefly-aioc-lvds-HSX101H40C -l rk3399_firefly_aioc_lvds_mid-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_lvds_mid-userdebug
```


### 参考资料

[屏幕模组 Datasheet & 转接板原理图](http://www.t-firefly.com/doc/download/page/id/48.html#other_137)

### 实物图
**注意：** 下图中电压跳线要使用 12V。

![](../../../rk3399_img/AIO-3399C/module_display_lvds.png)

## 7.85 寸 EDP 液晶屏模组

### 产品参数

* 型号：LP079QX1
* 尺寸：7.85 寸
* 分辨率：2048x1563
* 显示接口：EDP
* 面板材料：IPS 面板
* 可视角度：160°
* 触摸屏：多点电容触摸

### 固件参考

**注意：** 支持 7.85 寸屏的官方固件名带有 `EDP785` 字样，下面是固件的链接：[固件链接](http://www.t-firefly.com/doc/download/page/id/54.html#other_385)

### 编译命令

用官网 SDK 编译支持的 7.85 寸屏的固件时需用如下编译命令：

* Andorid 7.1 (Industry)

```
./FFTools/make.sh -j8 -d rk3399-firefly-aioc-edp -l rk3399_firefly_aioc_edp-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_edp-userdebug
```

* Andorid 7.1 (Box)

```
./FFTools/make.sh -j8 -d rk3399-firefly-aioc-edp -l rk3399_firefly_aioc_edp_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_edp_box-userdebug
```

### 技术资料

7.85 寸屏幕模组 DataSheet、底板原理图(https://pan.baidu.com/s/1mhXH4fu#list/path=%2F)

### 实物图

![](../../../rk3399_img/AIO-3399C/module_display_edp.jpg)

## 连接方法

* 请注意，下图中电压跳线要使用 5V ，接错到 12V 会烧掉屏幕芯片

![](../../../rk3399_img/AIO-3399C/module_display_edp_connection.jpg)
