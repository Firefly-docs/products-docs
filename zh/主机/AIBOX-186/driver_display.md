# DISPLAY 使用

AIBOX-186 引出了一路 HDMI 接口（支持 4K@60fps）以及 MIPS DSI 接口（最大输出能力 1440P (2560 * 1440)@60fps，[接口位置如图所示](interface_definition.html#zheng-ji-jie-kou-ding-yi)  。

需要注意的是，` /mnt/system/ko/soph_drm.ko` 为整个显示框架的驱动，诸如 HDMI以及 MIPI  DSI 等模块驱动都是打包进这个驱动里面的。

## HDMI
AIBOX-186 在开机之后，没有内置桌面环境，只跑一个显示程序：
```
insmod /mnt/system/ko/soph_drm.ko # 安装显示框架驱动
systemctl stop SophonHDMI.service # 关闭 HDMI 的显示界面
systemctl restart SophonHDMI.service  # 恢复 HDMI 的显示界面
```

## MIPI DSI

目前系统只能存在一个显示，当你接着 HDMI 的时候，MIPI DSI 默认没有输出。
但你拔掉 HDMI ，你所接入的 MIPI 显示模组会以播放彩条的形式进行显示。

目前已适配 Firelfy 的 [DM-M10R800 V2](https://wiki.t-firefly.com/DM-M10R800-V2/dm-m10r800-v2.html) 显示模组。
