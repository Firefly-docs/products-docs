## HDMI 无法 4K 显示？

暂时无法支持自动切换 4K, 默认 1080P 显示, 后续会更新支持。可以使用以下命令手动设置 4K 分辨率后,重新插拔 HDMI 线：

```
# 设置4k60：
setprop persist.sys.resolution.main 3840x2160@60
# 每次设置完更新sys.display.timeline(每次加1)使分辨率生效
setprop sys.display.timeline 1
```

## 3.5mm耳机座子接国标耳机有异常？

目前iCore-3568JQ仅支持美标类型(CTIA)的耳机,对于国标类型(OMTP)的耳机硬件不兼容,会出现左右声道同时存在双声道的现象。




