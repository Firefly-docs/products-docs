# 屏幕模组

## [DM-M10R800 V2 MIPI屏模组](https://item.taobao.com/item.htm?ft=t&id=655100190974)

### 产品参数

* 型号：M101014_BE45_A1
* 尺寸：10.1寸
* 分辨率：800x1280
* 显示接口：MIPI
* 面板材料：IPS面板
* 可视角度：160°
* 触摸屏：多点电容触摸

###  参考固件


支持10.1寸屏的官方固件名带有`MIPI`字样，下面是固件的链接：[双MIPI屏固件链接](https://www.t-firefly.com/doc/download/125.html#other_543)  

<font color="#dd0000">注意：</font><br />由于RK3566双屏幕显示是使用同一个内部输入源，即`VOP`是`Same Source, Dual Display`,所以如果是使用MIPI屏幕作为主屏，会导致副屏HDMI画面会拉伸，所以想避免这个问题，主副屏幕都应该使用同分辨率同方向（横竖屏一致）的屏幕。


### 编译命令

用官网SDK编译支持的10.1 寸mipi屏的固件时使用以下命令：

```
./FFTools/make.sh -d rk3566-firefly-aiojd4-mipi101_M101014_BE45_A1 -j8 -l rk3566_firefly_aiojd4_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3566_firefly_aiojd4_mipi-userdebug
```


### 参考资料

[屏幕模组 Datasheet & 转接板原理图](https://www.t-firefly.com/doc/download/125.html#other_489)

### 实物图

#### 正面
* MIPI DSI0
![](../../img/Station-P2/mipi101_v2_M101014_BE45_A1_front0.jpg)
* MIPI DSI1
![](../../img/Station-P2/mipi101_v2_M101014_BE45_A1_front1.jpg)