# 硬件功能使用
## 调试串口

整机板载 Debug 调试串口（3P-2.0mm），需要外接 USB 转 TTL 模块接入调试串口，用于查看启动日志与系统调试。调试串口的连接方式与使用方法详见：[调试串口](debug.md)。

调试串口连接如下图：

<center>

<img alt="" src="../../../rk356x_img/ROC-RK3568-PC-SE/debug_connection.jpg" width="800">
</center>

## UART（RS232 / RS485）

ROC-RK3568-PC-SE 提供 1 个控制串口（Control Port），引出 RS232 × 2 和 RS485 × 1 信号，接口位置见上文接口图。串口设备节点以实际系统为准，可通过 `ls /dev/ttyS*` 查看。

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

整机调试串口终端即可接收到字符串 `firefly RS485 test...`。RS232 信号的验证方法同理，将节点换成对应的 RS232 设备节点即可。

## SIM 卡

整机板载 SIM 卡槽用于配合 Mini PCIe 4G 模块实现移动网络连接。**4G 模块为选配**，需要先安装 4G 模块后，SIM 卡功能才能正常使用。插拔 SIM 卡前请先断电，插卡方向以卡槽丝印为准，联网状态以实际系统为准。

## 显示接口

ROC-RK3568-PC-SE 提供 1 个 HDMI 2.0 输出接口（最高支持 4K@60Hz 输出）和 1 个 MIPI-DSI 接口（30P-0.5mm）。Android 系统可在 `设置 -> 显示` 中调整分辨率与显示模式。

获取 HDMI 的 edid、支持的分辨率与连接状态：

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
cat /sys/class/drm/card0-HDMI-A-1/modes
cat /sys/class/drm/card0-HDMI-A-1/status
```

一般如果遇到 HDMI 无法显示的问题，都需要先执行上面的命令查看连接状态、edid 和分辨率是否正确。

## 以太网

ROC-RK3568-PC-SE 提供 2 个 RJ45 千兆网口，对应系统中的 `eth0`、`eth1` 两个设备。

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

## 存储

ROC-RK3568-PC-SE 提供 1 个 M.2 PCIe3.0 接口（NVMe SSD 2242）、1 个 SATA 3.0 接口（HDD/SSD）和 1 个 TF Card 卡槽。安装 SSD/HDD 后，系统下可通过 `lsblk` 查看识别到的存储设备：

```
lsblk
```
