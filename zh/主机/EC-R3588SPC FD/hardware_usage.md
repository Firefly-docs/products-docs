# 硬件功能使用
## 调试串口

EC-R3588SPC-FD 整机未引出调试串口接口，需要拆下上盖后接入底板（ROC-RK3588S-PC）上的调试串口，并外接 USB 转 TTL 串口模块进行调试。

调试串口的连接方式与使用方法详见：[调试串口](usb_to_ttl.md)。

## 显示接口

EC-R3588SPC-FD 提供 1 路 HDMI2.1（最高支持 7680x4320@60Hz 输出）、1 路 Display Port 1.4（最高支持 7680x4320@30Hz 输出，软件节点 `dp0`）和 2 路 MIPI DSI 显示输出接口，支持多屏同显/异显。

RK3588S 拥有四路 Video 输出端口（Port0 ~ Port3），每一个 Video 输出端口都绑定了固定的显示控制器，各端口可输出的最大分辨率如下：

* Port0 最大可以输出 7680x4320@60Hz
* Port1 最大可以输出 4096x2304@60Hz
* Port2 最大可以输出 4096x2304@60Hz
* Port3 最大可以输出 1920x1080@60Hz

多屏使用注意事项：RK3588S 做 8K 输出时会同时占用 Port0 与 Port1 的资源，此时与 Port1 连接的显示控制器输出会有异常；只有当 Port0 输出小于等于 4K@60Hz 时，Port1 才可以正常输出 4K@60Hz。

显示接口的软件配置参考：`kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi`。

### 调试手段

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

EC-R3588SPC-FD 提供 1 个 RJ45 网口（支持 1Gbps），对应系统中的 `eth0` 设备。

接入网络后，可以通过调试串口或者 adb 查看 IP 地址：

```
ifconfig eth0
```

连通性测试：

```
ping -I eth0 -c 10 www.baidu.com
```

## SATA

EC-R3588SPC-FD 提供 1 个 M.2 接口，默认配置为 M.2 SATA3.0（支持 SATA 协议 SSD），也可以软件配置为 M.2 PCIe2.0（支持 NVMe 协议 SSD），修改配置后需重启系统生效。

DTS 配置位于 `kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi`：

```
#define M2_SATA_OR_PCIE 1 /*1 = SATA , 0 = PCIe */

/* default use sata3.0 , pcie2.0 optional*/
&combphy0_ps {
    status = "okay";
};

#if M2_SATA_OR_PCIE
&sata0 {
    pinctrl-names = "default";
    pinctrl-0 = <&sata_reset>;
    status = "okay";
};
#else
&pcie2x1l2 {
    reset-gpios = <&gpio3 RK_PD1 GPIO_ACTIVE_HIGH>;
    vpcie3v3-supply = <&vcc3v3_pcie20>;
    status = "okay";
};
#endif
```

以 SATA3.0 为例，设备在系统中识别为 `sda`（Android 下为 `/dev/block/sda`），手动挂载：

```
# 格式化为 EXT4 文件格式
mkfs.ext4 /dev/sda
# 挂载
mount /dev/sda /mnt/
# 查看挂载路径
df -h
```

## RTC

EC-R3588SPC-FD 采用 HYM8563 作为 RTC（Real Time Clock）。HYM8563 是一款低功耗 CMOS 实时时钟/日历芯片，所有的地址和数据都通过 I2C 总线接口串行传递，可计时基于 32.768kHz 晶体的秒、分、小时、星期、天、月和年。驱动参考：`kernel-5.10/drivers/rtc/rtc-hym8563.c`。

Linux 提供了三种用户空间调用接口，对应的路径为（设备编号以实际系统为准）：

* SYSFS 接口：/sys/class/rtc/rtc0/
* PROCFS 接口：/proc/driver/rtc
* IOCTL 接口：/dev/rtc0

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

## Watchdog

看门狗（watchdog）实际是一个定时器，启动之后会开始计时。系统或者软件需要在规定时间内与看门狗通信（俗称喂狗）重置计时，如果规定时间内没有喂狗，看门狗超时后会发出复位信号让主控复位，脱离卡死。

RK3588S 的 watchdog 的 DTS 节点在 `kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588s.dtsi` 文件中定义，watchdog 默认是关闭的，需在 DTS 文件中打开 wdt 节点方能使用：

```
&wdt{
    status = "okay";
};
```

内部看门狗的设备名称为 `/dev/watchdog`，用户可通过 `echo` 命令来控制该设备：

```
# 写入任意内容（大写字母'V'除外），开启看门狗，每 44 秒内需要写入一次（喂狗）
echo A > /dev/watchdog

# 开启看门狗，并且内核会每隔 22 秒自动喂一次狗
echo V > /dev/watchdog
```

* [设备树手册](linux_dts_manual.md)
