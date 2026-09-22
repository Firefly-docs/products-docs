# 硬件功能使用
## 调试串口

EC-A3576JD4 整机未引出调试串口接口，需要拆开主机后，使用外接 USB 转 TTL 串口模块连接主板上的调试串口排针进行调试。

调试串口的连接方式与使用方法详见：[调试串口](usb_to_ttl.md)。

## CAN

AIO-3576JD4 提供 1 路 CAN 接口（凤凰端子座）。接线时 CAN_H 接 CAN_H，CAN_L 接 CAN_L。

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

`candump`、`cansend` 工具包含在 SDK 中，也可以从 can-utils 获取（Ubuntu 系统可使用 `apt update && apt install can-utils` 安装）。

调试与验证说明：

* 通信两端的比特率必须配置一致，否则会接收不到报文；可用 `ip -details link show can0` 查看 CAN 设备的详细配置与状态。
* 若发送后接收不到报文，请检查总线 CAN_H 和 CAN_L 是否松动或者接反。

## UART（RS232 / RS485）

AIO-3576JD4 提供 1 个 RS232 接口和 1 个 RS485 接口（均为凤凰端子座）。

串口设备在系统中为 `/dev/ttyS*` 形式的节点（具体编号以实际系统为准，可通过 `ls /dev/ttyS*` 查看）。以 RS485 为例，使用 USB 转串口适配器连接开发板（主机端适配器节点以实际为准，如 `/dev/ttyUSB0`）：

开发板发送，主机接收：

```
# 主机终端先执行
cat /dev/ttyUSB0
# 开发板调试串口终端执行
echo "firefly uart test..." > /dev/ttySx
```

主机发送，开发板接收：

```
# 开发板调试串口终端先执行
busybox stty -echo -F /dev/ttySx       # 关闭回显
cat /dev/ttySx
# 主机终端执行
echo "firefly uart test..." > /dev/ttyUSB0
```

将 `/dev/ttySx` 替换为对应串口节点即可。RS232 的验证方法同理。

## SIM 卡

AIO-3576JD4 的 SIM 卡槽用于配合 4G/5G 模块实现移动网络连接。**4G/5G 模块为选配**，需要在机箱内部安装 4G/5G 模块后，SIM 卡功能才能正常使用。SIM 卡插入方向如下图所示，插拔前请先断电。

**注意**：Mini PCIe（4G 模块）和 PCIe M.2（5G 模块）共用了一路 USB，不能同时使用。默认配置为 4G 模块，如需使用 5G 需要调整底板电阻位置。

<center>

<img alt="" src="../../../rk3576_img/EC-A3576JD4/sim_insert_direction.png" width="400">
</center>

## 显示接口

AIO-3576JD4 提供 1 个 HDMI 显示输出接口，支持 HDMI2.1 协议，分辨率最高可以支持 4K@120Hz。

若遇到 HDMI 无法显示的问题，可以先执行以下命令查看连接状态、edid 和分辨率是否正确：

```
cat /sys/class/drm/card0-HDMI-A-1/status
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
cat /sys/class/drm/card0-HDMI-A-1/modes
```

## 以太网

AIO-3576JD4 提供 2 个 RJ45 千兆网口，对应系统中的 `eth0`、`eth1` 两个设备。

网口接入网络后，可以通过调试串口或者 adb 查看 IP 地址并进行连通性测试：

```
ifconfig eth0
ping -I eth0 -c 10 www.baidu.com
```
