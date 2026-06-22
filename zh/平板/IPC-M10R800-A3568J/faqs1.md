## 如何手动切换HDMI 4K分辨率

除了在`设置->显示->HDMI->分辨率设置`中可以调节分辨率，也 可以使用以下命令手动设置 4K 分辨率后,重新插拔 HDMI 线：

```
# 设置4k60：
setprop persist.vendor.resolution.main 3840x2160@60
# 每次设置完更新sys.display.timeline(每次加1)使分辨率生效
setprop vendor.display.timeline 1
```

## 3.5mm耳机座子接国标耳机有异常？

目前IPC-M10R800-A3568J仅支持美标类型(CTIA)的耳机,对于国标类型(OMTP)的耳机硬件不兼容,会出现左右声道同时存在双声道的现象。

