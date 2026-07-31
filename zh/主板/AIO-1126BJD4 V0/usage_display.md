# Display 使用

* 底板与 MIPI 屏幕的硬件连接方式如图所示：

![](../../../rv1126b_img/AIO-1126BJD4V0/mipi_screen.jpg)

Debian 固件默认开机会进入到 xfce 桌面系统。由于 RV1126B 芯片没有 GPU 做硬件加速渲染，桌面环境使用 CPU 来渲染，所以在高负载作业情况下，显示可能会有卡顿的现象。

* QT 应用屏幕转向

```
#指定 QT 输出平台并使屏幕顺时针转向 90°
export QT_QPA_PLATFORM=linuxfb:rotation=90
```
* 如果屏幕出现花屏，或者不显示的情况。请检查LCD排线是否有问题，更换优质材质的短排线能解决花屏问题。