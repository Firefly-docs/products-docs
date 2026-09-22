# 硬件功能使用
## 调试串口

EC-R3576PC-FD 整机未引出调试串口接口，需要拆开主机后，使用外接 USB 转 TTL 串口模块连接主板上的调试串口排针进行调试。

调试串口的连接方式与使用方法详见：[调试串口](usb_to_ttl.md)。

## 显示接口

ROC-RK3576-PC 提供 HDMI、Display Port 两种显示输出接口，可以做到多屏同显/异显：

* HDMI：支持 HDMI2.1 协议，分辨率最高可以支持 4K@120Hz。
* Display Port：软件上表示为 `dp0`，支持 DP TX 1.4a 协议，分辨率最高可以支持 4K@120Hz。

RK3576 拥有 3 路 Video 输出端口，每一个 Video 输出端口都绑定了固定的显示控制器，各端口可输出的最大分辨率如下：

* Port0 最大可以输出 4K@120Hz
* Port1 最大可以输出 2560x1600@60Hz
* Port2 最大可以输出 1920x1080@60Hz

SDK 默认配置将 HDMI 连接在 Port0、dp0 连接在 Port2；如需 dp0 输出更高分辨率，可将 dp0 改配到 Port0（修改 `rk3576-firefly-roc-rk3576-pc.dtsi` 中的 `dp0_in_vp2` 为 `dp0_in_vp0`）。

### 调试手段

* 获取 HDMI/Display Port 的 edid、所支持的分辨率与连接状态（以 HDMI 为例）：

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
cat /sys/class/drm/card0-HDMI-A-1/modes
cat /sys/class/drm/card0-HDMI-A-1/status
```

* 获取系统中正在使用的 Video Portx（与所连接的显示控制器）信息：

```
cat /sys/kernel/debug/dri/0/summary
```

一般如果遇到 HDMI/Display Port 无法显示的问题，都需要先执行上面的命令查看连接状态、edid 和分辨率是否正确。

## 以太网

ROC-RK3576-PC 提供 1 个 RJ45 千兆网口，对应系统中的 `eth0` 设备。

网口接入网络后，可以通过调试串口或者 adb 查看 IP 地址并进行连通性测试：

```
ifconfig eth0
ping -I eth0 -c 10 www.baidu.com
```

## 存储（M.2 SATA3.0 / PCIe2.0）

ROC-RK3576-PC 带有 1 个 M.2 接口，可以软件配置成 M.2 SATA3.0 接口（支持 SATA 协议的 SSD），也可以软件配置成 M.2 PCIe2.0 接口（支持 NVMe 协议的 SSD）。默认软件配置成 M.2 SATA3.0 接口。

SATA 与 PCIe 的切换通过 DTS 中的 `M2_SATA_OR_PCIE` 宏控制（位于 `rk3576-firefly-roc-rk3576-pc.dtsi`）：**默认值为 1 即配置成 SATA3.0，如果需要配置成 PCIe2.0，需修改为 0**。

系统识别到的设备节点一般为 `/dev/block/sda`，格式化与挂载命令如下：

```
mkfs.ext4 /dev/block/sda
mount /dev/block/sda /mnt/media_rw/
df -h
```

整机还带有 1 个 TF Card 卡槽，插入 TF 卡后即可在系统中使用。
