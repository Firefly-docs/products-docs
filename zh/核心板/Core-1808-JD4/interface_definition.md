# 接口定义

## 整机接口定义

AIO-1808-JD4 提供了丰富的接口，主要包括：电源接口， 1 x USB3.0（host/device），6 x USB2.0（接口×5，座子×1），以太网，LVDS屏幕接口，TP触摸接口，屏电压跳线接口，背光接口，WIFI天线，蓝牙天线，电源按键，MIC接口，3.5mm耳机接口，RTC电源接口，12v电源接口，TF卡槽，SIM卡卡槽，扩展按键接口，喇叭接口，recovery按键，调试串口，工业级串口(RS485,RS232,2TTL)，MIPI屏接口(双lvds复用)。

**说明：CORE-1808-JD4的底板与CORE-3399-JD4以及CORE-3399Pro-JD4的底板兼容，驱动开发以及底板原理图可以参考CORE-3399-JD4以及CORE-3399Pro-JD4的维基**

**注意：底板为了兼容其它核心板，CORE-1808-JD4不支持底板上的某些功能，所以图片中有些功能没有注明，说明CORE-1808-JD4并不支持底板的这些功能**

具体如下图：

![](../../../rk1808_img/interface.png)
![](../../../rk1808_img/back.png)

#### 特殊接口说明
目前版本暂不支持的功能：
* LINE_IN/LINE_OUT，IR。
* 硬件不支持USB_HOST2, eDP屏接口, MINI-PCIE，HDMI。
* USB接口：固件默认USB3.0使用OTG功能，无法使用host功能，USB2.0接口只支持HUB接口，即USB2.0最上层的接口与其它5个外接端口。