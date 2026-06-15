# 接口定义

## 整机接口定义

Firefly-RK3399 提供了丰富的接口，主要包括：


* 电源接口
* 1 x USB3.0（host/device）
* 8 x USB2.0（接口×6，座子×2）
* HDMI，以太网
* LVDS屏幕接口
* eDP屏接口
* TP触摸接口
* 屏电压跳线接口
* 背光接口
* WIFI天线
* 蓝牙天线
* 电源按键
* MIC接口
* 音频输入输出(linein,lineout)
* 3.5mm耳机接口
* RTC电源接口
* 12v电源接口
* IR接口
* TF卡槽
* SIM卡卡槽
* 扩展按键接口
* 喇叭接口
* USB2.0 HOST
* recovery按键
* 调试串口
* 工业级串口(RS485,RS232,2TTL)
* MIPI屏接口(双lvds复用)
* MINI-PCIE

具体如下图：

![](../../../rk3399_img/Firefly-RK3399/interface.png)

除了上述接口，AIO-3399JD4（已贴SPR5801S芯片）支持NPU加速功能。

**注意：AIO-3399JD4 中的SPR5801S芯片是选贴的。如果需要NPU加速功能，请注意选购带有SPR5801S芯片的硬件版本。(硬件NPU已经全部替换成SPR5801S，但软件仍支持旧版本SPR2801S)**

## 版本信息
**注意： V2.2以上版本使用NPU SPR5801S，低于V2.2的是SPR2801S**

![](../../../rk3399_img/Firefly-RK3399/NPU_v22_cn.png)

**注意： 版本低于V2.2的是SPR2801S**

![](../../../rk3399_img/Firefly-RK3399/NPU.jpg)