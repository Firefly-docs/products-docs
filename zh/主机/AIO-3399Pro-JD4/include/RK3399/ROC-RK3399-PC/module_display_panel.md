
# 屏幕模组

（<font color=#ff0000 size=3>注意：</font>10.1寸MIPI屏幕与8寸MIPI屏幕在生成的时候名字并无不同，在官网资源上面的带MIPI101或MIPI8字样的固件是在打包生成之
后，手动修改的名字，用以用户做区分）

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
* Android 7.1 (Industry)
```
  ./FFTools/make.sh -j8 -d rk3399-roc-pc-mipi8 -l rk3399_roc_pc_mipi-userdebug
  ./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_mipi-userdebug
```

### 实物图

![](../../../rk3399_img/AIO-3399Pro-JD4/panel_mipi8.jpg)


## 10.1寸MIPI液晶屏模组

详情参考 [LCD 驱动章节](driver_lcd.md)

## 产品参数

* 型号：CZZC101863T05B
* 尺寸：10.1寸
* 分辨率：800x1280
* 显示接口：MIPI
* 面板材料：IPS面板
* 可视角度：160°
* 触摸屏：多点电容触摸

### 编辑命令
* Android 7.1 (Industry)
```
  ./FFTools/make.sh -j8 -d rk3399-roc-pc-mipi101 -l rk3399_roc_pc_mipi-userdebug
  ./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_mipi-userdebug
```

![](../../../rk3399_img/AIO-3399Pro-JD4/panel_mipi101.jpg)



## 10.1寸EDP液晶屏模组

详情参考 [LCD 驱动章节](driver_lcd.md)

## 产品参数

* 型号：LP079QX1
* 尺寸：10.1寸
* 分辨率：2560x1600
* 显示接口：EDP
* 面板材料：IPS面板
* 可视角度：160°
* 触摸屏：多点电容触摸

### 编辑命令
* Android 7.1 Industry
```
  ./FFTools/make.sh -j8 -d rk3399-roc-pc-edp -l rk3399_roc_pc_edp-userdebug
  ./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_edp-userdebug
```

![](../../../rk3399_img/AIO-3399Pro-JD4/panel_edp101.ipg)