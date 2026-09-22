# 硬件功能使用
## 调试串口

Station P1 Pro 整机未引出调试串口接口，需要拆开主机后接入主板 ROC-RK3399-PC Pro 上的调试串口，并外接 USB 转 TTL 串口模块进行调试。串口参数为：波特率 1500000、数据位 8、停止位 1、无奇偶校验、无流控。

调试串口的连接方式与使用方法详见：[调试串口](usb_to_ttl.md)。

## 显示接口

Station P1 Pro 提供两种显示输出接口，支持双屏显示：

* HDMI 2.0：4K@60Hz 输出。
* DP 1.2：通过 USB-C 接口输出，最高 4K@60Hz。

双屏显示时支持 DP1.2（2K 输出）+ HDMI（4K 输出）组合，可在系统的显示设置中配置多屏显示方式。

## 以太网

Station P1 Pro 提供 1 个 RJ45 千兆网口（1000Mbps），对应系统中的 `eth0` 设备。

网口接入网络后，可以通过调试串口或者 adb 查看 IP 地址：

```
ifconfig eth0
```

连通性测试：

```
ping -I eth0 -c 10 www.baidu.com
```

## USB 接口

Station P1 Pro 对外提供以下 USB 接口：

* 1 x USB3.0（限流 1000mA）
* 1 x USB2.0（限流 500mA）
* 1 x USB-C（USB3.0 / OTG / DP1.2），OTG 方式主要用于连接主机进行固件烧写

## 存储

Station P1 Pro 提供以下存储扩展：

* TF Card：面板上的 TF 卡槽，支持 TF 卡扩展存储，插拔前请先断电。
* NVMe SSD：机内主板提供 1 x PCIe2.1，可扩展 2242 NVMe SSD，需要拆开主机后安装。安装后可在系统中对 SSD 进行格式化与挂载：

```
mkfs.ext4 /dev/nvme0n1
mount /dev/nvme0n1 /mnt/
df -h
```

## 音频

Station P1 Pro 提供 1 个 3.5mm Audio Jack，支持 Mic 录音；Recovery 升级按键位于耳机孔内。此外还支持 HDMI 音频输出与 DP1.2 音频输出（通过 USB-C 接口输出）。

## 红外与按键

Station P1 Pro 支持红外遥控功能；面板上提供 1 个 Power 按键。
