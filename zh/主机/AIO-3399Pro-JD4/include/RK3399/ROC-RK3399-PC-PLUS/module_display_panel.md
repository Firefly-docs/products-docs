
# 屏幕模组

## 8寸MIPI液晶屏模组

详情参考 [LCD 驱动章节](driver_lcd.md)

###  产品参数

* 型号：CZNB080860T
* 尺寸：8寸
* 分辨率：800x1280
* 显示接口：MIPI
* 面板材料：IPS面板
* 可视角度：160°
* 触摸屏：多点电容触摸

### 编辑命令
* Android 7.1(industry)
```
  ./FFTools/make.sh -j8 -d rk3399-roc-pc-plus-mipi8 -l rk3399_roc_pc_plus_mipi-userdebug
  ./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus_mipi-userdebug
```

### 实物图

![](../../../rk3399_img/AIO-3399Pro-JD4/panel_mipi8.jpg)

## 10.1寸MIPI液晶屏模组

详情参考 [LCD 驱动章节](driver_lcd.md)

### 产品参数

* 型号：CZZC101863T05B
* 尺寸：10.1寸
* 分辨率：800x1280
* 显示接口：MIPI
* 面板材料：IPS面板
* 可视角度：160°
* 触摸屏：多点电容触摸

### 编辑命令
* Android 7.1 (industry)
```
  ./FFTools/make.sh -j8 -d rk3399-roc-pc-plus-mipi101_id_detect -l rk3399_roc_pc_plus_mipi101-userdebug
  ./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus_mipi101-userdebug
```

### 实物图

![](../../../rk3399_img/AIO-3399Pro-JD4/pc+_mipi101_2.jpg)

### 固件下载
* [网盘链接](https://pan.baidu.com/s/1H0wlejja0Jr0ilsAOK-htw) 
* 提取码：1234


## [DM-M10R800 V2 MIPI屏模组](https://item.taobao.com/item.htm?ft=t&id=655100190974)

### 产品参数

* 型号：M101014_BE45_A1
* 尺寸：10.1 寸
* 分辨率：800x1280
* 显示接口：MIPI
* 可视角度：160°
* 触摸屏：多点电容触摸

###  参考固件

**注意：** 支持 10.1 寸MIPI屏的官方固件名带有 `MIPI` 字样，下面是固件的链接：[固件链接](https://www.t-firefly.com/doc/download/125.html#other_499)

### 编译命令

用官网 SDK 编译支持的 10.1 寸MIPI屏的固件时使用以下命令：

```
./FFTools/make.sh -j8 -d rk3399-roc-pc-plus-mipi101-JDM101014_BC45_A1 -l rk3399_roc_pc_plus_mipi101-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus_mipi101-userdebug
```

### 实物图

![](../../../rk3399_img/AIO-3399Pro-JD4/pc+_mipi101_v2.jpg)

### 参考资料

[屏幕模组 Datasheet & 转接板原理图](https://www.t-firefly.com/doc/download/125.html#other_489)

## 10.1寸EDP液晶屏模组

详情参考 [LCD 驱动章节](driver_lcd.md)

### 产品参数

* 型号：LP079QX1
* 尺寸：10.1寸
* 分辨率：2560x1600
* 显示接口：EDP
* 面板材料：IPS面板
* 可视角度：160°
* 触摸屏：多点电容触摸

### 编辑命令
* Android 7.1 (industry)
```
 ./FFTools/make.sh -j8 -d rk3399-roc-pc-plus-edp -l rk3399_roc_pc_plus_edp-userdebug
  ./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus_edp-userdebug
```

![](../../../rk3399_img/AIO-3399Pro-JD4/panel_edp101.jpg)