# 硬件功能使用
## 调试串口

EC-R3588RT_2G5 整机未引出调试串口接口，需要拆下上盖后接入底板 ROC-RK3588-RT 上的调试串口，并外接 USB 转 TTL 串口模块进行调试。

调试串口的连接方式与使用方法详见：[调试串口](usb_to_ttl.md)。

## 显示接口

EC-R3588RT_2G5 提供 2 个 HDMI2.1 显示输出接口和 1 个 Display Port 1.4 显示输出接口（与 Type-C OTG 接口复用）：

* HDMI：HDMI 输出接口，支持 HDMI2.1 协议，单口最高支持 7680x4320@60Hz（8K）输出。
* Display Port：DP 输出接口（软件节点 `dp0`），最高支持 7680x4320@30Hz 输出。

RK3588 拥有四路 Video 输出端口（Port0 ~ Port3），每一个 Video 输出端口都绑定了固定的显示控制器（如 Port0 可以用于与 DP0、DP1、HDMI/eDP0 和 HDMI/eDP1 等显示控制器连接），各端口可输出的最大分辨率如下：

* Port0 最大可以输出 7680x4320@60Hz
* Port1 最大可以输出 4096x2304@60Hz
* Port2 最大可以输出 4096x2304@60Hz
* Port3 最大可以输出 1920x1080@60Hz

多屏使用注意事项：RK3588 做 8K 输出时会同时占用 Port0 与 Port1 的资源，此时与 Port1 连接的显示控制器输出会异常，多屏场景请注意各显示控制器与 Video Port 的分配。

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

EC-R3588RT_2G5 整机提供 3 个 RJ45 网口：2 个千兆网口和 1 个 2.5G 网口。2.5G 网口支持巨型帧，拥有高带宽和低时延等特点。各网口在系统中识别为 `ethX` 设备（编号以实际系统为准）。

此外，EC-R3588RT_2G5 还通过 PCIe 扩展提供 4 个 2.5G 网口，可满足多路有线网络接入场景。

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

Linux 系统下也可通过 `ip addr` 或 `ifconfig` 查看网口状态与 IP 地址。

## M.2 接口

EC-R3588RT_2G5 提供 2 个 M.2 扩展接口：

* 1 x PCIe2.0 (M.2 NVMe)：用于扩展 NVMe 固态硬盘。
* 1 x PCIe2.0 (M.2 E-KEY)：用于扩展 WiFi6 / BT5.0 无线模块。

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

也可以通过程序来控制看门狗：使用 `open` 打开 `/dev/watchdog` 后看门狗立刻开始计时，通过 `ioctl`（`WDIOC_SETTIMEOUT`/`WDIOC_GETTIMEOUT`）设置和获取超时时间，并循环 `write` 喂狗。需要注意的是：当用户没有设置超时时间时，驱动会应用默认请求的超时时间 30s。

参考文档：SDK/RKDocs（Linux 为 docs）/common/watchdog

* [设备树手册](linux_dts_manual.md)
