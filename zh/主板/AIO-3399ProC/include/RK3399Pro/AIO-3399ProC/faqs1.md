## HDMI 无法 4K 显示？

AIO-3399ProC 默认出厂固件是支持 LVDS+HDMI 1080P 的双屏显示，HDMI 分辨率最高只能到 1080P。HDMI 要支持到 4K 分辨率需要重新烧写[默认固件](http://www.t-firefly.com/doc/download/31.html#other_80)，或者重新编译内核 `make ARCH=arm64 rk3399pro-firefly-aioc.img` ，重新烧写 `resource.img` 即可。

## 如何确认固件是否支持 4K？

AIO-3399ProC 打开`设置`->`显示` ,显示设备处若出现 HDMI 和 DSI，代表固件是支持双屏显示的，不支持 4K。若只有 HDMI 则是默认固件，可以支持 4K。

## 如何调整HDMI输出分辨率？

AIO-3399ProC 的 HDMI 能自动识别显示的分辨率。假如无法读取显示器的 EDID (Extended Display Identification Data, 扩展显示器标识数据) ，HDMI 会默认设置为 1080P 的分辨率。你也可以进入系统设置对 HDMI 分辨率进行手动调整。

![](../../../rk3399_img/faqs_setting_resolution.jpg)