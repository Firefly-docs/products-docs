# 屏幕模组

## 7.85 寸 EDP 液晶屏模组

###  产品参数

* 型号：LP079QX1
* 尺寸： 7.85 寸
* 分辨率：2048x1563
* 显示接口：EDP
* 面板材料：IPS面板
* 可视角度：160°
* 触摸屏：多点电容触摸

### 参考固件

**注意：**  支持 7.85 寸屏的官方固件名带有 `EDP 785` 字样，下面是固件的链接：[固件链接](https://pan.baidu.com/s/1ULLpq50iaRg9O0YbzWWi-A) (提取码: 1234)

### 编译命令

用官网 SDK 编译支持的 7.85 寸屏的固件时需用如下编译命令：

* Android 7.1 (Box)

```
  ./FFTools/make.sh -j8 -d rk3399-firefly-edp -l rk3399_firefly_edp_box-userdebug
  ./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_edp_box-userdebug
```

### 技术资料

[ 7.85 寸屏幕模组 DataSheet、底板原理图](https://pan.baidu.com/s/1mhXH4fu#list/path=%2F)

### 实物图

![](../../../rk3399_img/Firefly-RK3399/module_display_edp.jpg)

### 连接方法

![](../../../rk3399_img/Firefly-RK3399/module_display_edp_connection.jpg)

##  7.85 寸 MIPI 液晶屏模组

### 产品参数

* 型号：B080XAN01
* 尺寸：7.85 寸
* 分辨率：1024x768
* 显示接口：MIPI
* 面板材料：IPS面板
* 可视角度：160°
* 触摸屏：多点电容触摸

### 参考固件

**注意：** 支持 7.85 寸屏的官方固件名带有 `MIPI785` 字样，下面是固件的链接：[固件链接](https://pan.baidu.com/s/1slNUe8P#list/path=%2F)

### 编译命令

用官网 SDK 编译支持的 7.85 寸屏的固件时需用如下编译命令：

* Android 7.1 (Box)

```
  ./FFTools/make.sh -j8 -d rk3399-firefly-mipi -l rk3399_firefly_mipi_box-userdebug
  ./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_mipi_box-userdebug
```

### 实物图

![](../../../rk3399_img/Firefly-RK3399/module_display_mipi_connection.jpg)