

# 接口定义

## 整机接口定义

AIO-3399Pro-JD4 提供了丰富的接口，主要包括：

* 电源接口,
* 1 x USB3.0（host/device）,
* 7 x USB2.0（接口×5,座子×2）,
* HDMI,以太网,LVDS 屏幕接口,
* EDP 屏接口,
* TP 触摸接口,
* 屏电压跳线接口,
* 背光接口,
* WIFI 天线,
* 蓝牙天线,
* 电源按键,
* MIC 接口,
* 音频输入输出 (lineout),
* 3.5mm 耳机接口,
* RTC 电源接口,
* 12V 电源接口,
* IR 接口,
* TF 卡槽,
* SIM 卡卡槽,
* 扩展按键接口,
* 喇叭接口,
* Recovery 按键,
* 调试串口,
* 工业级串口 (RS485, RS232, 2TTL),
* MIPI 屏接口(双 LVDS 复用),
* MINI-PCIE.

具体如下图：

![](../../../rk3399_img/AIO-3399Pro-JD4/interface_front.png)

![](../../../rk3399_img/AIO-3399Pro-JD4/interface_reverse.png)

## 特殊接口说明

目前软件版本暂不支持 LINE_IN/LINE_OUT，硬件不支持 USB_HOST2，其中工程机型(底板 JD4-1.0 版本) SPERAKER 只支持单声道。
## 发货清单(仅供参考)

* 1 x AIO-3399Pro-JD4
* 1 x USB Type-A to Type-C数据线

## 配件

可能你还需要另外购买以下配件：

* 12V电源
* SD卡、emmc
* 串口模块


## 开发板上电启动

在开发板上电启动前，确认以下事项：

- 可启动的 SD 卡或eMMC
- 12V 电源适配器