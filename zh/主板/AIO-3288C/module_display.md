# 屏幕模组

## [10.1寸LVDS屏幕模组](https://store.t-firefly.com/goods.php?id=80)

### 产品参数

* 品牌：Firefly
* 型号：HSX101H40C
* 尺寸：10.1寸
* 分辨率：1280x800
* 接口：LVDS
* 可视角度：170°
* 触摸屏：多点电容触摸

### 内核编译

```bash
make firefly_defconfig
make -j8 firefly-rk3288-aio-3288c_lvds.img
```

### 参考资料

[屏幕模组Datasheet](https://pan.baidu.com/s/1KZhcqUGRpl7sPhqWbtHOUA)

[转接板原理图](https://pan.baidu.com/s/1ucHOWxNe-JhbLoYGwjSVPg)

## 参考固件

[AIO-3288C_LVDS](http://www.t-firefly.com/doc/download/page/id/48.html#other_140)

### 接线方式

![](../../../rk3288_img/AIO-3288C/module_display.png)

* 注意：黄色座子为TP插座，红色座子为背光控制插座