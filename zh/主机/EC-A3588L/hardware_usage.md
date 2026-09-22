# 硬件功能使用
## 调试串口

EC-A3588L 整机未引出调试串口接口，需要拆开主机后接入核心板上的调试串口，并外接 USB 转 TTL 串口模块进行调试。

调试串口的连接方式与使用方法详见：[调试串口](usb_to_ttl.md)。

## CAN

AIO-3588L 提供 1 路 CAN 接口。接线时 CAN_H 接 CAN_H，CAN_L 接 CAN_L。

需要注意的是，核心板 AIO-3588L 的 SDK 中 CAN1 与 UART3 复用，默认固件配置为 UART3；如需使用 CAN 功能，需要在设备树中把 `CAN1_OR_UART3` 配置为 1 使能 CAN1（详见核心板 Wiki），CAN 设备节点以实际系统为准。

CAN 通信测试命令如下（以 `can0` 为例，请按实际节点替换）：

```
# 关闭 CAN 设备
ip link set can0 down
# 设置比特率为 250Kbps
ip link set can0 type can bitrate 250000
# 打开 CAN 设备
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

AIO-3588L 整机对外提供 2 个 RS232 接口和 1 个 RS485 接口。核心板 AIO-3588L 的 SDK 中启用的串口为 UART3、UART6、UART7、UART8，对应的软件节点如下（RS232/RS485 接口与各 UART 的对应关系以实际系统为准）：

```
UART3:  /dev/ttyS3（与 CAN1 复用，默认配置为 UART3）
UART6:  /dev/ttyS6
UART7:  /dev/ttyS7
UART8:  /dev/ttyS8
```

### 收发验证

最简单的验证方式是短接串口的 TX、RX 引脚做自发自收，在调试串口或 adb 终端执行（以 `/dev/ttyS7` 为例，节点以实际系统为准）：

```
busybox stty -echo -F /dev/ttyS7          # 关闭回显
cat /dev/ttyS7 &                          # 后台获取 /dev/ttyS7 输入字符串
echo "firefly uart test..." > /dev/ttyS7  # 输入字符串
```

终端即可接收到字符串 `firefly uart test...`。RS232/RS485 的验证也可以使用主机的 USB 转串口适配器，一端接主机、一端接开发板串口，收发方法与上述相同。

## 显示接口

AIO-3588L 提供多个显示输出接口：

* HDMI2.1：HDMI 输出接口（软件节点 `hdmi0`），支持 HDMI2.1 协议，最高支持 7680x4320@60Hz（8K）输出。
* HDMI2.0：HDMI 输出接口（软件节点 `hdmi1`），支持 HDMI2.0 协议，最高支持 4096x2304@60Hz 输出。
* Display Port1.4：Display Port 输出接口（软件节点 `dp0`），最高支持 7680x4320@30Hz 输出。
* MIPI-DSI x2：两路 MIPI DSI 输出接口，最高可输出 4096x2304@60Hz（取决于连接的 Portx）。

RK3588 拥有四路 Video 输出端口（Port0 ~ Port3），每一个 Video 输出端口都绑定了固定的显示控制器（如 Port0 可以用于与 DP0、DP1、HDMI/eDP0 和 HDMI/eDP1 等显示控制器连接），各端口可输出的最大分辨率如下：

* Port0 最大可以输出 7680x4320@60Hz
* Port1 最大可以输出 4096x2304@60Hz
* Port2 最大可以输出 4096x2304@60Hz
* Port3 最大可以输出 1920x1080@60Hz

多屏使用注意事项：

* 目前一个 Portx 在同一时间只能输出一种分辨率格式，如果多个不同分辨率的显示接口配置在同一个 Portx 上，同一时间只能使用其中一个。
* RK3588 做 8K 输出时会同时占用 Port0 与 Port1 的资源。例如 HDMI0（接 Port0）做 8K 输出时，HDMI1（接 Port1）显示会异常；只有当 Port0 输出小于等于 4K@60Hz 时，Port1 才可以正常输出 4K@60Hz。

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

AIO-3588L 提供 2 个 RJ45 千兆网口，对应系统中的 `eth0`、`eth1` 两个设备。

### 查看 IP 地址与连通性测试

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

目标 IP 请根据实际网络环境修改。Linux 系统下可通过 `ip addr` 或 `ifconfig` 查看网口状态与 IP 地址；Android 系统下双网口的内外网主副关系以实际系统为准。

## USB

AIO-3588L 的 USB 接口如下：

* 1 x USB3.0 OTG（Type-C）：用于固件升级、adb 调试等。
* 4 x USB3.0、3 x USB2.0：Host 接口，可外接 USB 键鼠、U 盘等设备，即插即用。

USB 设备插入后，可通过 `lsusb` 查看是否识别成功。

## TF 卡

AIO-3588L 提供 1 个 TF Card 卡槽，插入 TF 卡后系统会自动识别挂载（挂载路径以实际系统为准）。

## SIM 卡

AIO-3588L 的 SIM 卡槽用于配合 4G/5G 模块实现移动网络连接。**4G/5G 模块为选配**，需要在机箱内部安装 4G/5G 模块后，SIM 卡功能才能正常使用。SIM 卡插入方向如下图所示，插拔前请先断电。

<center>

<img alt="" src="../../../rk3588_img/EC-A3588L/sim_insert_direction.png" width="400">
</center>
