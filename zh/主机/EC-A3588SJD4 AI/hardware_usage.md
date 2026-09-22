# 硬件功能使用
## 调试串口

AIO-3588SJD4-AI 整机未引出调试串口接口，需要拆开主机后接入主板上的调试串口，并外接 USB 转 TTL 串口模块进行调试。

调试串口的连接方式与使用方法详见：[调试串口](usb_to_ttl.md)。

## CAN

AIO-3588SJD4-AI 提供 1 路 CAN 接口。接线时 CAN_H 接 CAN_H，CAN_L 接 CAN_L。

CAN 设备在系统中识别为 `can0`，通信测试命令如下：

```
# 关闭 can0 设备
ip link set can0 down
# 设置比特率为 250Kbps
ip link set can0 type can bitrate 250000
# 打开 can0 设备
ip link set can0 up
# 接收端执行 candump，阻塞等待报文
candump can0
# 发送端执行 cansend，发送报文
cansend can0 123#1122334455667788
```

`candump`、`cansend` 工具包含在 SDK 中，也可以从 can-utils 获取。

调试与验证说明：

* 通信两端的比特率必须配置一致，否则会接收不到报文；可用 `ip -details link show can0` 查看 CAN 设备的详细配置与状态。
* 若发送后接收不到报文，请检查总线 CAN_H 和 CAN_L 是否松动或者接反。

## UART（RS232 / RS485）

AIO-3588SJD4-AI 提供 1 个 RS232 接口和 1 个 RS485 接口。其中 RS232 由主控 UART9 扩展，RS485 由主控 UART6 扩展，DTS 配置位于 `kernel/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588sjd4-ai.dtsi`。

各硬件接口对应的软件节点（以实际系统为准）：

```
RS232:  /dev/ttyS9
RS485:  /dev/ttyS6
```

使用说明：

* 引脚 GPIO1_A2 用于 RS485 的收发控制，该引脚拉高为发送，拉低为接收，可通过 `/sys/class/gpio` 子系统控制（GPIO 编号以实际系统为准）。
* 串口默认波特率为 9600，数据位 8 位，停止位 1 位，无流控。

### 收发验证

以 RS485（`/dev/ttyS6`）为例，使用主机的 USB 转串口适配器连接整机（主机端适配器节点以实际为准，如 `/dev/ttyUSB0`）。

整机发送，主机接收：

```
# 主机终端先执行
cat /dev/ttyUSB0
# 整机调试串口终端执行
echo "firefly RS485 test..." > /dev/ttyS6
```

主机终端即可接收到字符串 `firefly RS485 test...`。

主机发送，整机接收：

```
# 整机调试串口终端先执行
busybox stty -echo -F /dev/ttyS6       # 关闭回显
cat /dev/ttyS6
# 主机终端执行
echo "firefly RS485 test..." > /dev/ttyUSB0
```

整机调试串口终端即可接收到字符串 `firefly RS485 test...`。RS232 的验证方法同理，将节点换成对应的 `/dev/ttyS9` 即可。

## 显示接口

AIO-3588SJD4-AI 提供 1 个 HDMI2.1 显示输出接口，支持 HDMI2.1 协议，最高支持 7680x4320@60Hz（8K）输出。

RK3588 拥有四路 Video 输出端口（Port0 ~ Port3），每一个 Video 输出端口都绑定了固定的显示控制器（如 Port0 可以用于与 DP0、DP1、HDMI/eDP0 和 HDMI/eDP1 等显示控制器连接），各端口可输出的最大分辨率如下：

* Port0 最大可以输出 7680x4320@60Hz
* Port1 最大可以输出 4096x2304@60Hz
* Port2 最大可以输出 4096x2304@60Hz
* Port3 最大可以输出 1920x1080@60Hz

### 调试手段

* 在系统中设置分辨率：Android 在 `设置 -> 显示 -> HDMI -> 分辨率设置` 中调整分辨率。
* 获取 HDMI 的 edid：

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
```

* 获取 HDMI 所支持的分辨率：

```
cat /sys/class/drm/card0-HDMI-A-1/modes
```

* 获取 HDMI 的连接状态：

```
cat /sys/class/drm/card0-HDMI-A-1/status
```

* 获取系统中正在使用的 Video Portx（与所连接的显示控制器）信息：

```
cat /d/dri/0/summary
```

一般如果遇到 HDMI 无法显示的问题，都需要先执行上面的命令查看连接状态、edid 和分辨率是否正确。

## 以太网

AIO-3588SJD4-AI 提供 1 个 RJ45 网口（支持 1Gbps），对应系统中的 `eth0` 设备（设备编号以实际系统为准）。

网口接入网络后，可以通过调试串口或者 adb 查看 IP 地址：

```
ifconfig eth0
```

连通性测试：

```
ping -I eth0 -c 10 www.baidu.com
```

## USB 接口

AIO-3588SJD4-AI 的 USB 接口包括：

* 2 x USB3.0：用于连接 USB 外设。
* 1 x USB2.0：由排针引出。
* 1 x USB Type-C：下载/Debug 接口，主要用于烧写固件和系统调试。

## TF 卡

AIO-3588SJD4-AI 提供 1 个 TF Card 卡槽。插拔卡前请先断电，插卡方向以卡槽丝印为准。

## SIM 卡

AIO-3588SJD4-AI 的 SIM 卡槽用于配合 4G 模块实现移动网络连接；WIFI 与蓝牙由外置模块提供。**4G 模块为选配**，需要在机箱内部安装 4G 模块后，SIM 卡功能才能正常使用（以实际配置为准）。SIM 卡插入方向如下图所示，插拔前请先断电。

<center>

<img alt="" src="../../../rk3588_img/EC-A3588SJD4-AI/sim_insert_direction.png" width="400">
</center>

## 音频接口

AIO-3588SJD4-AI 提供 Line-Out（线路输出）、Line-In（线路输入）和 Headphone（耳机）接口，可分别用于连接功放等音频设备、外部音源和耳机。

## NPU 使用

RK3588S 内置 NPU，算力可达 6 TOPS，支持 INT4/INT8/INT16 混合运算。使用 NPU 需要通过 RKNN SDK 部署模型，将算法模型转换为 `.rknn` 格式后即可在整机上运行，具体请参考：[NPU 使用](usage_npu.md)。

* [设备树手册](linux_dts_manual.md)
