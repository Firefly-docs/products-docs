# 硬件功能使用
## 调试串口

EC-R3566PC 整机面板未引出调试串口接口，需要拆开主机后才能接入主板上的调试串口进行调试，调试时需外接 USB 转 TTL 串口模块。

调试串口的连接方式与使用方法详见：[调试串口](debug.md)。

## 显示接口

ROC-RK3566-PC 提供 1 个 HDMI 2.0 输出接口，最高支持 4K@60fps 输出。Android 系统可在 `设置 -> 显示` 中调整分辨率与显示模式。

获取 HDMI 的 edid、支持的分辨率与连接状态：

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
cat /sys/class/drm/card0-HDMI-A-1/modes
cat /sys/class/drm/card0-HDMI-A-1/status
```

一般如果遇到 HDMI 无法显示的问题，都需要先执行上面的命令查看连接状态、edid 和分辨率是否正确。

## 以太网

ROC-RK3566-PC 提供 1 个 RJ45 千兆网口，对应系统中的以太网设备（如 `eth0`，以实际系统为准）。

网口接入网络后，可以通过调试串口或者 adb 查看 IP 地址：

```
ifconfig eth0
```

连通性测试：

```
ping -I eth0 -c 10 www.baidu.com
```

测试时，目标域名/IP 请根据实际网络环境修改。Linux 系统下可通过 `ip addr` 或 `ifconfig` 查看网口状态与 IP 地址。

## USB 接口

ROC-RK3566-PC 提供 1 个 USB 3.0 接口和 1 个 USB 2.0 接口，可外接 U 盘、USB 键鼠等设备。插入设备后，可通过 `lsusb` 查看设备是否被识别：

```
lsusb
```

另外，整机面板上的 Type-C 接口为 OTG & 供电复用接口：默认作为电源输入（DC 5V），在需要 adb/OTG 场景下可作为 USB OTG 接口使用（OTG 与供电的复用方式以实际硬件设计为准）。

## 音频接口

ROC-RK3566-PC 提供 1 个 3.5mm Audio 接口，可外接耳机或音箱。Android 系统可在 `设置 -> 声音` 中调整音量与输出设备；Linux 系统下可通过 `aplay -l` 查看声卡设备，使用 `aplay` 播放音频：

```
aplay -l
aplay /usr/share/sounds/alsa/Front_Center.wav
```
