ROC-RK3399-PC是一款迷你PC主板,体积只有小型手机的大小。
![](../../../rk3399_img/ROC-RK3399-PC/roc-rk3399-pc7.jpg)

![](../../../rk3399_img/ROC-RK3399-PC/roc-rk3399-pc6.jpg)

* Dual-core Cortex-A72 up to 1.8GHz & Quad-core Cortex-A53 up to 1.5GHz 六核处理器
* Mali-T864 GPU，支持 OpenGL ES1.1/2.0/3.0, OpenCL1.2, DirectX11.1.
* LPDDR4 RAM (4GB)
* 3 x USB 2.0
* 10/100/1000Mb 网口
* IR红外模组
* HDMI 2.0a with HDCP 2.2
* DisplayPort 1.2 (4K 60Hz)
* 3 x MIPI 接口:MIPI0 only for DSI, MIPI1 for DSI or CSI, MIPI2 only for CSI
* eDP 1.3
* I2S，支持 8 通道
* 高速 eMMC 扩展接口
* MicroSD (TF) 卡槽
* 2 x Type-C 接口:Type-C0 支持DP 1.2、PD 2.0、USB3.0，Type-C1支持DP 1.2、USB3.0
* RTC : 电池型号 -- CR1220 ; 座子 -- CR1220,BS-12-A3B0J002（2PIN电池座卧式灰色镀金1U）锦和泰
* 132 个扩展接口:I2C、SPI、Uart、I2S....

## 发货清单(仅供参考)

* 1 x ROC-RK3399-PC
* 1 x USB Type-A to Type-C数据线

## 配件

可能你还需要另外购买以下配件：

* PD电源15V/3A(45W)
* SD卡、emmc
* ROC-RK3399-PC扩展板
* 串口模块

**注意**：开发板使用PD(15V/3A,45W)电源供电，请搭配合适的电源使用。

## 开发板上电启动

在开发板上电启动前，确认以下事项：

- 可启动的 SD 卡或eMMC
- 15V/3A/45W PD 电源

由于开发板通过Type-C0供电，同时Type-C1支持DP视频信号输出，所以开机分为两种情况：

* 接独立PD电源：

1. 断电状态下插入可启动的 SD 卡或eMMC 之一。
2. 插入 HDMI 线、USB 鼠标或键盘（可选）。
3. 检查一切连接正常后，**Type-C0**接上PD电源上电。


* 接显示屏Type-C口，输出信号的同时通过显示屏给板子供电

1. 断电状态下插入可启动的 SD 卡或eMMC 之一。
2. 插入 HDMI 线、USB 鼠标或键盘（可选）。
3. 检查一切连接正常后，**Type-C0**接上显示屏的Type-C口上电。

**注意**：板子上有两个Type-C接口，**电源输入只能接Type-C0口**，不能接Type-C1。
![](../../../rk3399_img/ROC-RK3399-PC/roc-rk3399-pc3.jpg)