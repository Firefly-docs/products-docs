# 硬件功能使用
## 调试串口

EC-A3588Q 整机未引出调试串口接口，需要拆开主机后才能接入主板上的调试串口进行调试。

调试串口的连接方式与使用方法详见：[调试串口](usb_to_ttl.md)。

## CAN

EC-A3588Q 提供 1 路 CAN 接口。接线时 CAN_H 接 CAN_H，CAN_L 接 CAN_L。

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

EC-A3588Q 提供 2 个 RS232 接口（RS232_0、RS232_1）和 1 个 RS485 接口。其中 RS232_0 由主控 UART0 扩展，RS232_1 由主控 UART5 扩展，RS485 由主控 UART1 扩展。推荐使用官方 FC10 转 DB9 串口线，不同厂商的串口线线序可能不同，会导致串口无法通信。

各硬件接口对应的软件节点：

```
RS232_0:  /dev/ttyS0
RS232_1:  /dev/ttyS5
RS485:    /dev/ttyS1
```

### 收发验证

以 RS485（`/dev/ttyS1`）为例，使用主机的 USB 转串口适配器连接开发板（主机端适配器节点以实际为准，如 `/dev/ttyUSB0`）。

开发板发送，主机接收：

```
# 主机终端先执行
cat /dev/ttyUSB0
# 开发板调试串口终端执行
echo "firefly RS485 test..." > /dev/ttyS1
```

主机终端即可接收到字符串 `firefly RS485 test...`。

主机发送，开发板接收：

```
# 开发板调试串口终端先执行
busybox stty -echo -F /dev/ttyS1       # 关闭回显
cat /dev/ttyS1
# 主机终端执行
echo "firefly RS485 test..." > /dev/ttyUSB0
```

开发板调试串口终端即可接收到字符串 `firefly RS485 test...`。RS232_0、RS232_1 的验证方法同理，将节点换成对应的 `/dev/ttyS0`、`/dev/ttyS5` 即可。

## SIM 卡

EC-A3588Q 的 SIM 卡槽用于配合 4G/5G 模块实现移动网络连接。**4G/5G 模块为选配**，需要在机箱内部安装 4G/5G 模块后，SIM 卡功能才能正常使用。SIM 卡插入方向如下图所示，插拔前请先断电。

<center>

<img alt="" src="../../../rk3588_img/EC-A3588Q/sim_insert_direction.png" width="400">
</center>

## 显示接口

EC-A3588Q 提供多个显示输出接口和一路视频输入接口：

* HDMI2.1：HDMI 输出接口，支持 HDMI2.1 协议，最高支持 7680x4320@60Hz（8K）输出。
* USB-C（DP1.4）：Display Port 输出接口（软件节点 `dp0`），最高支持 7680x4320@30Hz 输出。
* VGA：由 DP 转 VGA 芯片实现（软件节点 `dp1`），最高支持 1080p@60Hz 输出。
* HDMI-IN：HDMI 输入接口，支持 HDMI2.0 协议，最高支持 4K@60fps 输入。

RK3588 拥有四路 Video 输出端口（Port0 ~ Port3），每一个 Video 输出端口都绑定了固定的显示控制器（如 Port0 可以用于与 DP0、DP1、HDMI/eDP0 和 HDMI/eDP1 等显示控制器连接），各端口可输出的最大分辨率如下：

* Port0 最大可以输出 7680x4320@60Hz
* Port1 最大可以输出 4096x2304@60Hz
* Port2 最大可以输出 4096x2304@60Hz
* Port3 最大可以输出 1920x1080@60Hz

多屏使用注意事项：RK3588 做 8K 输出时会同时占用 Port0 与 Port1 的资源。SDK 默认配置为 HDMI0（`hdmi0_in_vp0`）+ DP0（`dp0_in_vp2`）+ HDMI-IN，此配置下 HDMI2.1（8K）与 USB-C（DP1.4，4K@60Hz）可以同时正常输出；请勿将 DP0 改配到 vp1，否则 HDMI0 8K 输出时 DP0 显示会异常。

HDMI-IN 的使用：Android 系统自带 **Live Tv** 和 **RockchipCamera2** 两个 APK，打开即可显示 HDMI-IN 的输入画面；Ubuntu 固件集成了测试脚本，直接运行 `/usr/local/bin/test_hdmirx.sh` 即可预览输入。

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

EC-A3588Q 提供 2 个 RJ45 千兆网口，对应系统中的 `eth0`、`eth1` 两个设备。

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

## RTC

EC-A3588Q 搭载的核心板 iCore-3588Q 采用 HYM8563 作为 RTC（Real Time Clock）。HYM8563 是一款低功耗 CMOS 实时时钟/日历芯片，所有的地址和数据都通过 I2C 总线接口串行传递，可计时基于 32.768kHz 晶体的秒、分、小时、星期、天、月和年。驱动参考：`kernel-5.10/drivers/rtc/rtc-hym8563.c`。

Linux 提供了三种用户空间调用接口，对应的路径为（设备编号以实际系统为准）：

* SYSFS 接口：/sys/class/rtc/rtc0/
* PROCFS 接口：/proc/driver/rtc
* IOCTL 接口：/dev/rtc0

### SYSFS 接口

可以直接使用 `cat` 和 `echo` 操作 `/sys/class/rtc/rtc0/` 下面的接口。

比如查看当前 RTC 的日期和时间：

```
# cat /sys/class/rtc/rtc0/date
2022-06-21
# cat /sys/class/rtc/rtc0/time
06:52:08
```

设置开机时间，如设置 120 秒后开机：

```
# 120秒后定时开机
echo +120 > /sys/class/rtc/rtc0/wakealarm
# 查看开机时间
cat /sys/class/rtc/rtc0/wakealarm
# 关机
reboot -p
```

### PROCFS 接口

打印 RTC 相关的信息：

```
# cat /proc/driver/rtc
rtc_time        : 06:53:50
rtc_date        : 2022-06-21
alrm_time       : 06:55:05
alrm_date       : 2022-06-21
alarm_IRQ       : yes
alrm_pending    : no
update IRQ enabled      : no
periodic IRQ enabled    : no
periodic IRQ frequency  : 1
max user IRQ frequency  : 64
24hr            : yes
```

### IOCTL 接口

可以使用 `ioctl` 控制 `/dev/rtc0`，详细使用说明请参考文档 `kernel-5.10/Documentation/admin-guide/rtc.rst`。

### FAQs

#### Q: 上电后时间不同步？

A: 检查 RTC 电池是否正确接入（整机 RTC 供电配置以实际产品为准）。

## Watchdog

### 简介

看门狗（watchdog）实际是一个定时器，启动之后会开始计时。系统或者软件需要在规定时间内与看门狗通信（俗称喂狗）重置计时，如此反复下去，以此来确定系统和软件正常运行。

如果规定时间内没有喂狗，看门狗超时，说明系统或应用陷入循环、卡死，此时看门狗会发出复位信号让主控复位，脱离卡死。

### DTS 配置

RK3588 的 watchdog 的 DTS 节点在 `kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588s.dtsi` 文件中定义：

```
wdt: watchdog@feaf0000 {
    compatible = "snps,dw-wdt";
    reg = <0x0 0xfeaf0000 0x0 0x100>;
    clocks = <&cru TCLK_WDT0>, <&cru PCLK_WDT0>;
    clock-names = "tclk", "pclk";
    interrupts = <GIC_SPI 315 IRQ_TYPE_LEVEL_HIGH>;
    status = "disabled";
};
```

watchdog 默认是关闭的，需在 DTS 文件中打开 wdt 节点方能使用：

```
&wdt{
    status = "okay";
};
```

### 使用

watchdog 的驱动文件为 `kernel-5.10/drivers/watchdog/dw_wdt.c`。内部看门狗的设备名称为 `/dev/watchdog`，用户可通过 `echo` 命令来控制该设备：

```
# 写入任意内容（大写字母'V'除外），开启看门狗，每 44 秒内需要写入一次（喂狗）
echo A > /dev/watchdog

# 开启看门狗，并且内核会每隔 22 秒自动喂一次狗
echo V > /dev/watchdog
```

也可以通过程序来控制看门狗：使用 `open` 打开 `/dev/watchdog` 后看门狗立刻开始计时，通过 `ioctl`（`WDIOC_SETTIMEOUT`/`WDIOC_GETTIMEOUT`）设置和获取超时时间，并循环 `write` 喂狗。需要注意的是：当用户没有设置超时时间时，驱动会应用默认请求的超时时间 30s；驱动内部维护一个预置的超时时间列表，会在列表中找到一个合适的时间作为最终设置的超时时间，因此最终生效的超时时间并不一定等于应用层传入的时间。

参考文档：SDK/RKDocs（Linux 为 docs）/common/watchdog

如整机存在外部硬件看门狗，`/dev/` 下会生成 `wdt_XXX` 形式的设备文件（可通过 `ls /dev/wdt_*` 确认），通过写设备文件即可完成使能和喂狗（是否支持以实际系统为准）。
