# 硬件功能使用
## 调试串口

ITX-3588J  整机未引出调试串口接口，需要拆开机箱后接入主板上的调试串口，并外接 USB 转 TTL 串口模块进行调试。

调试串口的连接方式与使用方法详见：[调试串口](usb_to_ttl.md)。

## CAN

ITX-3588J  提供 1 路 CAN 接口（V1.1 版本新增；V1.1 版本 H 与 L 标反，V1.2 及后续版本丝印正确，接线前请核对端子丝印）。接线时 CAN_H 接 CAN_H，CAN_L 接 CAN_L。

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

ITX-3588J  提供 1 个 RS232 接口和 1 个 RS485 接口。其中 RS232 由主控 UART0 扩展（与 UART0 通过跳帽二选一使用），RS485 由主控 UART1 扩展（与 UART1 通过跳帽二选一使用），DTS 配置位于 `kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dtsi`。RS232、RS485 推荐使用官方的 FC10 转 DB9 串口线，不同厂商的串口线线序可能不同，会导致串口无法通信。

各硬件接口对应的软件节点（以实际系统为准）：

```
RS232:  /dev/ttyS0
RS485:  /dev/ttyS1
```

### 收发验证

以 RS485（`/dev/ttyS1`）为例，使用主机的 USB 转串口适配器连接整机（主机端适配器节点以实际为准，如 `/dev/ttyUSB0`）。

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

整机调试串口终端即可接收到字符串 `firefly RS485 test...`。RS232 的验证方法同理，将节点换成对应的 `/dev/ttyS0` 即可。

## 显示接口

ITX-3588J  提供多个显示输出接口、一路视频输入接口和 2 路 MIPI DSI 显示接口：

* HDMI0：HDMI 输出接口，支持 HDMI2.1 协议，最高支持 7680x4320@60Hz（8K）输出。
* HDMI1：HDMI 输出接口，支持 HDMI2.0 协议，最高支持 4096x2304@60Hz 输出。
* Display Port1.4：DP 输出接口，最高支持 7680x4320@30Hz（8K）输出。
* VGA：由 DP 转 VGA 芯片实现（软件节点 `dp1`），受限于 VGA 协议，最高支持 1080p@60Hz 输出。
* HDMI-IN：HDMI 输入接口，支持标准 HDMI2.0 协议，最高支持 4K@60fps 输入。
* 2 x MIPI DSI：MIPI DSI 显示输出接口。

RK3588 拥有四路 Video 输出端口（Port0 ~ Port3），每一个 Video 输出端口都绑定了固定的显示控制器（如 Port0 可以用于与 DP0、DP1、HDMI/eDP0 和 HDMI/eDP1 等显示控制器连接），各端口可输出的最大分辨率如下：

* Port0 最大可以输出 7680x4320@60Hz
* Port1 最大可以输出 4096x2304@60Hz
* Port2 最大可以输出 4096x2304@60Hz
* Port3 最大可以输出 1920x1080@60Hz

多屏使用注意事项：RK3588 做 8K 输出时会同时占用 Port0 与 Port1 的资源，此时与 Port1 连接的显示控制器输出会异常；只有当 Port0 输出小于等于 4K@60Hz 时，Port1 才可以正常输出 4K@60Hz。另外，目前一个 Portx 在同一时间只能输出一种分辨率格式，请合理分配各显示控制器与 Video Port。

### HDMI-IN 的使用

Android 系统自带 **Live Tv** 和 **RockchipCamera2** 两个 APK，打开即可显示 HDMI-IN 的输入画面；Ubuntu 固件集成了 `test_hdmirx.sh` 测试脚本，直接运行 `/usr/local/bin/test_hdmirx.sh` 即可预览输入。

### 调试手段

* 在系统中设置分辨率：Android 在 `设置 -> 显示 -> HDMI -> 分辨率设置` 中调整分辨率。
* 获取 HDMI/Display Port 的 edid（以 HDMI 为例）：

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
```

* 获取 HDMI/Display Port 所支持的分辨率（以 HDMI 为例）：

```
cat /sys/class/drm/card0-HDMI-A-1/modes
```

* 获取 HDMI/Display Port 的连接状态（以 HDMI 为例）：

```
cat /sys/class/drm/card0-HDMI-A-1/status
```

* 获取系统中正在使用的 Video Portx（与所连接的显示控制器）信息：

```
cat /d/dri/0/summary
```

一般如果遇到 HDMI/Display Port 无法显示的问题，都需要先执行上面的命令查看连接状态、edid 和分辨率是否正确。

## 以太网

ITX-3588J  提供 2 个 RJ45 网口（支持 1Gbps），对应系统中的 `eth0`、`eth1` 两个设备（设备编号以实际系统为准）。

网口接入网络后，可以通过调试串口或者 adb 查看 IP 地址：

```
ifconfig eth0
ifconfig eth1
```

连通性测试：

```
ping -I eth0 -c 10 www.baidu.com
ping -I eth1 -c 10 168.168.4.168
```

## USB 接口

ITX-3588J  的 USB 接口包括：

* 4 x USB3.0：用于连接 USB 外设。
* 4 x USB2.0：用于连接 USB 外设。
* 1 x USB3.0 OTG（Type-C）：OTG/下载接口，主要用于烧写固件，也可作 OTG 使用。

## 存储接口

ITX-3588J  提供丰富的存储扩展接口：

* 4 x SATA3.0：可连接 SATA 硬盘/固态硬盘。
* 1 x M.2（SATA3.0）：可连接 M.2 SATA 协议固态硬盘。
* 1 x PCIe3.0x4：可扩展 NVMe 协议固态硬盘等 PCIe 设备。
* 1 x TF Card：TF 卡槽，插拔卡前请先断电，插卡方向以卡槽丝印为准。

以 SATA 设备为例，设备在系统中识别为 `sdX`（以实际识别的设备为准），手动挂载：

```
# 格式化为 EXT4 文件格式
mkfs.ext4 /dev/sda
# 挂载
mount /dev/sda /mnt/
# 查看挂载路径
df -h
```

## SIM 卡

ITX-3588J  的 SIM 卡槽用于配合 4G/5G 模块实现移动网络连接；WIFI（支持 WiFi6）与蓝牙由板载/外置模块提供。**4G/5G 模块为选配**，需要在机箱内部安装 4G/5G 模块后，SIM 卡功能才能正常使用。SIM 卡插入方向如下图所示，插拔前请先断电。

<center>

<img alt="" src="../../../rk3588_img/EC-I3588J/sim_insert_direction.png" width="400">
</center>

## 音频接口

ITX-3588J  提供 Speaker（扬声器）和 Line-In（线路输入）接口，分别用于连接无源扬声器等发声设备和外部音源。

* [设备树手册](linux_dts_manual.md)
