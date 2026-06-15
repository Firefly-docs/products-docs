
# 接口定义

## 整机接口定义

AIO-3399C 提供了丰富的接口，主要包括：

* 电源接口
* Type-C (OTG)
* 1 x USB3.0
* 4 x USB2.0（接口×2，座子×2）
* HDMI
* 以太网
* 双 LVDS 屏幕接口
* EDP 屏接口
* TP 触摸接口
* 屏电压跳线接口
* 背光接口
* WIFI 天线
* 蓝牙天线
* 电源按键
* MIC 接口
* 音频输入输出 (linein,lineout)
* 3.5mm 耳机接口
* RTC 电源接口
* 12V 电源接口
* IR 接口
* TF 卡槽
* SIM 卡卡槽
* 扩展按键接口
* I2C
* I2S
* 喇叭接口
* USB2.0 HOST
* recovery 按键
* EDP 屏接口
* 调试串口
* 工业级串口 (RS485,RS232,2TTL)
* RTC 电源接入口
* MIPI 屏接口(双 LVDS 复用)
* MINI-PCIE

具体如下图：

![](../../../rk3399_img/AIO-3399C/interface_front.jpg)

![](../../../rk3399_img/AIO-3399C/interface_reverse.jpg)

*另外* 客户也可以根据需要定制相关功能接口的板子，如下图所示：

![](../../../rk3399_img/AIO-3399C/interface_customized_front.jpg)

![](../../../rk3399_img/AIO-3399C/interface_customized_reverse.jpg)

除了上述接口，AIO-3399C(AI)（已贴 SPR2801S 芯片）支持 NPU 加速功能，但是不支持 TYPEC 功能(包括 ADB OTG DP 显示)，TYPEC 只能用于烧固件。

***注意：AIO-3399C(AI) 中的 SPR2801S 芯片是选贴的。如果需要 NPU 加速功能，请注意选购带有 SPR2801S 芯片的硬件版本。***

![](../../../rk3399_img/AIO-3399C/interface_with_npu.jpg)