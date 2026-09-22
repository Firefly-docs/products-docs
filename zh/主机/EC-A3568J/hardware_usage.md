# 硬件功能使用
## 调试串口

EC-A3568J 整机面板未引出调试串口接口，需要拆开主机后才能接入主板上的调试串口进行调试，调试时需外接 USB 转 TTL 串口模块。

调试串口的连接方式与使用方法详见：[调试串口](debug.md)。

## CAN

AIO-3568J 提供 1 路 CAN 接口（支持 CAN/CANFD）。接线时 CAN_H 接 CAN_H，CAN_L 接 CAN_L。

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

AIO-3568J 整机后面板提供 2 个 RS232 接口和 2 个 RS485 接口（接线端子），接口位置参见上文接口图。串口设备节点以实际系统为准，可通过 `ls /dev/ttyS*` 查看。

### 收发验证

以 RS485 为例，使用 USB 转 RS485 适配器连接主机与整机（主机端适配器节点以实际为准，如 `/dev/ttyUSB0`，整机端 RS485 节点以实际系统为准，下述以 `/dev/ttyS1` 为例）。

整机发送，主机接收：

```
# 主机终端先执行
cat /dev/ttyUSB0
# 整机调试串口终端执行
echo "firefly RS485 test..." > /dev/ttyS1
```

主机终端即可接收到字符串 `firefly RS485 test...`。

主机发送，整机接收：

```
# 整机调试串口终端先执行
busybox stty -echo -F /dev/ttyS1       # 关闭回显
cat /dev/ttyS1
# 主机终端执行
echo "firefly RS485 test..." > /dev/ttyUSB0
```

整机调试串口终端即可接收到字符串 `firefly RS485 test...`。RS232 接口的验证方法同理，将节点换成对应的 RS232 设备节点即可。

## SIM 卡

AIO-3568J 的 SIM 卡槽用于配合 4G 模块实现移动网络连接。**4G 模块为选配**，需要在机箱内部安装 4G 模块后，SIM 卡功能才能正常使用。SIM 卡插入方向如下图所示，插拔前请先断电。

<center>

<img alt="" src="../../../rk356x_img/EC-A3568J/sim_insert_direction.png" width="400">
</center>

## 显示接口

AIO-3568J 提供 1 个 HDMI 2.0 输出接口，最高支持 4K 输出。Android 系统可在 `设置 -> 显示` 中调整分辨率与显示模式。

获取 HDMI 的 edid、支持的分辨率与连接状态：

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
cat /sys/class/drm/card0-HDMI-A-1/modes
cat /sys/class/drm/card0-HDMI-A-1/status
```

一般如果遇到 HDMI 无法显示的问题，都需要先执行上面的命令查看连接状态、edid 和分辨率是否正确。

## 以太网

AIO-3568J 提供 2 个 RJ45 千兆网口（WAN 口支持 PoE），对应系统中的以太网设备（如 `eth0`、`eth1`，以实际系统为准）。

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

测试时，目标 IP 请根据实际网络环境修改。Linux 系统下可通过 `ip addr` 或 `ifconfig` 查看网口状态与 IP 地址。

## SATA 硬盘安装

AIO-3568J 整机内部提供 1 个 SATA 3.0 接口，支持 2.5 寸 SSD/HDD，安装方式如下图：

<center>

<img alt="" src="../../../rk356x_img/EC-A3568J/ec-a3568j_sata.png" width="700">
</center>
