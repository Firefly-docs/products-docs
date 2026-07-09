# Display 使用

* RV1126/RV1109 配套的屏幕为[10.1 寸 IPS 全视觉屏幕模组 MIPI 多点触摸 1280x800 像素](https://item.taobao.com/item.htm?spm=a1z10.5-c-s.w4002-24147662620.19.187cce42Nq5fe8&id=655100190974)。MIPI 屏幕如图所示：

![](../../../rv1126_img/CAM-C1126S2U/screen.jpg)

* 屏幕转向

```
#指定 QT 输出平台并使屏幕顺时针转向 90°
export QT_QPA_PLATFORM=linuxfb:rotation=90
```

* 智能双目模组与 MIPI 屏幕的硬件连接方式如图所示：

![](../../../rv1126_img/CAM-C1126S2U/cam-c1126s2u-dsi.jpg)