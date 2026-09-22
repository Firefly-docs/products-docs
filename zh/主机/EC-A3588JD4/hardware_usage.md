# 硬件功能使用
## 调试串口

EC-A3588JD4 整机未引出调试串口接口，需要拆开主机后接入核心板上的调试串口，并外接 USB 转 TTL 串口模块进行调试。

调试串口的连接方式与使用方法详见：[调试串口](usb_to_ttl.md)。

## CAN

AIO-3588JD4 提供 1 路 CAN 接口。接线时 CAN_H 接 CAN_H，CAN_L 接 CAN_L。

CAN 设备在系统中识别为 `can0`（设备节点以实际系统为准），通信测试命令如下：

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

AIO-3588JD4 提供 1 个 RS232 接口和 1 个 RS485 接口。其中 RS232 由主控 UART1 扩展，RS485 由主控 UART6 扩展。

各硬件接口对应的软件节点：

```
RS232:  /dev/ttyS1
RS485:  /dev/ttyS6
```

需要注意的是，RS485 是半双工通信，发送前需要通过 GPIO 控制收发方向（GPIO 编号以实际系统为准，下例以 GPIO34 为例）。

### 收发验证

使用主机的 USB 转串口适配器连接开发板（主机端适配器节点以实际为准，如 `/dev/ttyUSB0`）。

开发板发送，主机接收：

```
# 主机终端先执行
cat /dev/ttyUSB0
# 开发板调试串口终端执行
echo "firefly RS485 test..." > /dev/ttyS6
```

主机终端即可接收到字符串 `firefly RS485 test...`。

主机发送，开发板接收：

```
# 开发板调试串口终端执行（将 GPIO34 配置为输出，并拉低切换为接收方向）
echo 34 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio34/direction
echo 0 > /sys/class/gpio/gpio34/value

# 开发板调试串口终端先执行
busybox stty -echo -F /dev/ttyS6       # 关闭回显
cat /dev/ttyS6
# 主机终端执行（拉高切换为发送方向）
echo 1 > /sys/class/gpio/gpio34/value
echo "firefly RS485 test..." > /dev/ttyUSB0
```

开发板调试串口终端即可接收到字符串 `firefly RS485 test...`。RS232 的验证方法同理，将节点换成对应的 `/dev/ttyS1` 即可，RS232 为全双工，无需 GPIO 控制收发方向。

## 显示接口

AIO-3588JD4 提供一个 HDMI 显示输出接口：

* HDMI2.1：HDMI 输出接口，支持 HDMI2.1 协议（软件节点 `hdmi0`），最高支持 7680x4320@60Hz（8K）输出。

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

AIO-3588JD4 提供 2 个 RJ45 千兆网口，对应系统中的 `eth0`、`eth1` 两个设备。

### 使用双以太网

Android 双以太网口分内网和外网，主副网口对应关系如下：

| 硬件设备名 | dts 节点 | Android 系统设备名 | 主副关系 |
|---|---|---|---|
| eth1 | gmac0 | Ethernet | 主网口，用于外网 |
| eth0 | gmac1 | Ethernet 2 | 副网口，用于内网 |

### 查看 IP 地址与连通性测试

双网口接入网络后，可以通过调试串口或者 adb 查看 IP 地址：

```
ifconfig eth0
ifconfig eth1
```

连通性测试：

```
ping -I eth0 -c 10 www.baidu.com
ping -I eth1 -c 10 168.168.4.168
```

内网口（`eth0`）测试时，目标 IP 请根据实际内网环境修改。Linux 系统下可通过 `ip addr` 或 `ifconfig` 查看网口状态与 IP 地址。

## USB

AIO-3588JD4 的 USB 接口如下：

* 1 x USB Type-C（下载/Debug）：用于固件升级、adb 调试等。
* 2 x USB3.0：Host 接口，可外接 USB 键鼠、U 盘等设备，即插即用。

USB 设备插入后，可通过 `lsusb` 查看是否识别成功。

## TF 卡

AIO-3588JD4 提供 1 个 TF Card 卡槽，插入 TF 卡后系统会自动识别挂载（挂载路径以实际系统为准）。

## SIM 卡

AIO-3588JD4 的 SIM 卡槽用于配合 4G/5G 模块实现移动网络连接。**4G/5G 模块为选配**，需要在机箱内部安装 4G/5G 模块后，SIM 卡功能才能正常使用。SIM 卡插入方向如下图所示，插拔前请先断电。

<center>

<img alt="" src="../../../rk3588_img/EC-A3588JD4/sim_insert_direction.png" width="400">
</center>
