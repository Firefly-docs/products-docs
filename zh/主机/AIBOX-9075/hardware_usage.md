# 硬件功能使用
## 调试串口

AIBOX-9075 采用板载 USB 串口方案（串口转 USB 芯片为 CH342），使用 USB 数据线直连整机的 Type-C 调试口即可进行调试，无需外接 USB 转 TTL 串口模块。

Windows 下安装驱动后，设备管理器会出现 "USB-Enhanced-SERIAL-A CH342" 和 "USB-Enhanced-SERIAL-B CH342" 两个串口设备：SERIAL-A 是主系统（Linux）的 Debug 串口，SERIAL-B 是子系统（RTOS）的 Debug 串口。串口参数：波特率 115200、数据位 8、停止位 1、无奇偶校验。

连接方式与 Windows 驱动安装详见：[调试串口](debug.md)。

## 显示接口

AIBOX-9075 提供 1 个 HDMI 2.0 显示输出接口。整机出厂默认运行 Ubuntu 24.04（Wayland + Gnome 桌面），接上显示器即可看到桌面显示。

## 以太网

AIBOX-9075 提供 2 个 2.5G RJ45 网口，对应系统中的 `eth0`、`eth1` 设备（以实际系统为准）。网口接入网络后，可以通过调试串口或 SSH 登录设备查看 IP 地址并测试连通性：

```
ifconfig eth0
ping -I eth0 -c 10 www.baidu.com
```

## USB 接口

AIBOX-9075 提供 2 个 USB3.0 接口，可外接 USB 键鼠、U 盘等设备。

## RS485

AIBOX-9075 提供 2 个 RS485 接口：RS485_1 对应系统中的 `/dev/ttyHS1`，RS485_2 对应系统中的 `/dev/ttyHS3`。RS485 收发需要额外 GPIO 控制方向（RS485_1 使用 695 号引脚，RS485_2 使用 693 号引脚），收发测试方法详见[串口教程](usage_serial.md)。

## CAN-FD

AIBOX-9075 提供 2 路 CAN-FD 接口（光耦隔离），可连接 CAN 总线设备进行通信，详细使用方法待补充。

## GMSL2

AIBOX-9075 提供 2 个 GMSL2 4Pin Mini FAKRA 接口，可用于接入 GMSL2 摄像头等设备，详细使用方法待补充。

## IO

AIBOX-9075 引出 6 路 IO 输入（IO in）与 6 路 IO 输出（IO out），详细使用方法待补充。

## SIM 卡

AIBOX-9075 的 SIM 卡槽用于配合无线模块实现移动网络连接。**无线模块为选配**，需要在机箱内部安装无线模块后，SIM 卡功能才能正常使用。SIM 卡插入方向如下图所示，插拔前请先断电。

<center>

<img alt="" src="../../../qcom_img/AIBOX-9075/sim_insert_direction.png" width="400">
</center>

## RTC

AIBOX-9075 支持 RTC 实时时钟，时间的读取、写入与同步方法详见[RTC 教程](usage_rtc.md)。

## 看门狗

AIBOX-9075 支持硬件看门狗，使能与喂狗方法详见[看门狗教程](usage_watchdog.md)。

## 视频

AIBOX-9075 默认的视频框架为 Gstreamer，编解码能力与播放、编码命令详见[视频教程](usage_video.md)。

## NPU（AI）

IQ-9075 峰值算力 200 TOPS，支持端侧大模型私有化部署，AI 开发方法详见[AI 教程](usage_npu.md)。
