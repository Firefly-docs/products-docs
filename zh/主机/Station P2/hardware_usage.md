# 硬件功能使用
## 调试串口

Station P2 整机面板未引出调试串口接口，需要拆开主机后才能接入主板上的调试串口进行调试，调试时需外接 USB 转 TTL 串口模块。

调试串口的连接方式与使用方法详见：[调试串口](debug.md)。

## UART（RS232 / RS485）

Station P2 整机面板提供 1 个控制串口（Control Port），引出 RS232 × 2 和 RS485 × 1 信号，信号定义如下：

```
1. RS232_TX3    2. RS232_RX3
3. RS232_TX2    4. GND
5. GND          6. RS232_RX2
7. RS485_A      8. RS485_B
```

串口设备节点以实际系统为准，可通过 `ls /dev/ttyS*` 查看。

### 收发验证

以 RS485（`RS485_A`/`RS485_B`）为例，使用 USB 转 RS485 适配器连接主机与整机（主机端适配器节点以实际为准，如 `/dev/ttyUSB0`，整机端 RS485 节点以实际系统为准，下述以 `/dev/ttyS1` 为例）。

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

## 显示接口

Station P2 提供 1 个 HDMI 2.0 输出接口，最高支持 4K@60Hz 输出。Android 系统可在 `设置 -> 显示` 中调整分辨率与显示模式。

获取 HDMI 的 edid、支持的分辨率与连接状态：

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
cat /sys/class/drm/card0-HDMI-A-1/modes
cat /sys/class/drm/card0-HDMI-A-1/status
```

一般如果遇到 HDMI 无法显示的问题，都需要先执行上面的命令查看连接状态、edid 和分辨率是否正确。

## 以太网

Station P2 提供 2 个 RJ45 千兆网口，对应系统中的 `eth0`、`eth1` 两个设备。

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

## USB 接口

Station P2 提供 1 个 USB 3.0 接口和 2 个 USB 2.0 接口，可外接 U 盘、USB 键鼠等设备。插入设备后，可通过 `lsusb` 查看设备是否被识别：

```
lsusb
```

此外，整机面板上的 USB-C 接口为 OTG 接口，可用于连接主机进行 adb 调试或固件升级。

## SATA 硬盘安装

Station P2 整机内部提供 1 个 SATA 3.0 接口，支持 2.5 寸、7mm 厚的 HDD/SSD，安装方式如下图：

<center>

<img alt="" src="../../../rk356x_img/Station-P2/station_p2_zh_Install.png" width="700">
</center>
