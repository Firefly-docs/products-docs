## V1.0版本的设备和V1.1版本的设备的区别
### V1.0版本设备
在V1.0版本设备中，I2C2_M0和I2C2_M1被硬件同时引出，造成了复用，所以使用I2C2_M1的MIPI CSI和使用I2C2_M0的RTC和MIPI DSI触摸部分是不能同时使用，V1.0版本设备出厂固件默认是使能I2C2_M0（即支持RTC和MIPI DSI关闭MIPI CSI），如需要使能MIPI CSI，则SDK需要进行以下修改即可
```
&i2c2{
	pinctrl-0 = <&i2c2m1_xfer>;
};

```

这样I2C2就使用了M1通道，而RTC和MIPI DSI的触摸部分暂时用不了。

### V1.1及以上版本设备
在V1.1及以上版本的设备中，MIPI CSI单独使用的I2C4_M0和RTC,MIPI DSI所使用的I2C2_M0是不冲突的，共用的，不需要使用上一步的配置来进行区分使用。

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




