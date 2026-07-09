# Display

* The screen that comes with RV1126/RV1109 is [10.1-inch IPS full-vision screen module MIPI multi-touch 1280x800 pixels](https://item.taobao.com/item.htm?spm=a1z10.5-c-s.w4002-24147662620.19.187cce42Nq5fe8&id=655100190974). The MIPI screen is shown in the figure:

![](../../../rv1126_img/CAM-C1109S2U/screen.jpg)

* Screen rotation

```
#Specify the QT output platform and rotate the screen 90° clockwise
export QT_QPA_PLATFORM=linuxfb:rotation=90
```

* The hardware connection method between the intelligent binocular module and the MIPI screen is shown in the figure:

![](../../../rv1126_img/CAM-C1109S2U/cam-c1126s2u-dsi.jpg)