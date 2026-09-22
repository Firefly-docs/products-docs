# 硬件功能使用
## 调试串口

AIBOX-8550 采用板载 USB 串口方案（串口转 USB 芯片为 PL2303GL），使用 USB 数据线直连整机的 Console 口即可进行调试，无需外接 USB 转 TTL 串口模块。

串口参数：波特率 115200、数据位 8、停止位 1、无奇偶校验。

连接方式与 Windows 驱动安装详见：[调试串口](debug.md)。

## 显示接口

AIBOX-8550 提供 1 个 HDMI 2.0 显示输出接口。整机出厂默认运行 Ubuntu 22.04（Wayland + Weston 桌面），接上显示器即可看到桌面显示。

## 以太网

AIBOX-8550 提供 2 个 1000M RJ45 网口，对应系统中的 `eth0`、`eth1` 设备（以实际系统为准）。网口接入网络后，可以通过调试串口或 SSH 登录设备查看 IP 地址并测试连通性：

```
ifconfig eth0
ping -I eth0 -c 10 www.baidu.com
```

## USB 接口

AIBOX-8550 提供 2 个 USB3.0 接口，可外接 USB 键鼠、U 盘等设备。另有 1 个 Type-C 接口作为烧录口使用，固件升级方法请参阅[升级固件](qfil_upgrade_firmware.md)与[升级分区镜像](fastboot_upgrade_image.md)。

## TF 卡

AIBOX-8550 提供 1 个 TF 卡槽，可用于扩展存储。

## 音频

AIBOX-8550 有 1 路 I2S 接入了 HDMI 用于音频播放，整机音频通过 HDMI 输出。

底层播放仅支持 48K 采样率、16 bit 的 wav 音频，需要使用 agmplay 工具：

```bash
# 向 HDMI 播放声音
agmplay test.wav -D 100 -d 100 -i MI2S-LPAIF-RX-SECONDARY -dkv 0xA2000004
```

桌面环境下，可在系统的声音设置中选择音频输出设备并调节音量（以实际固件支持的音频服务为准）。

## 视频

AIBOX-8550 默认的视频框架为 Gstreamer。使用 `gst-inspect-1.0` 可以查看系统支持的多媒体组件，其中 `qtic2vdec`（H.264/H.265/VP8/VP9/MPEG 视频解码）、`qtic2venc`（H.264/H.265/HEIC 视频编码）等 qti 组件享有硬件加速：

```bash
gst-inspect-1.0 --plugin | grep "qti"
```

编解码能力：视频解码最高支持 4K240 / 8K60，支持 AV1 解码，原生支持 H.265 Main 10、H.265 Main、H.264 High、VP9 profile 2 格式；视频编码最高支持 4K120 / 8K30，原生支持 H.265 Main 10、H.265 Main、H.264 High 格式；可同时进行 4K60 解码与 4K60 编码。

视频播放（使用 qtic2vdec 组件，以播放 h264 视频为例）：

```bash
export XDG_RUNTIME_DIR=/run/user/root
export WAYLAND_DISPLAY=wayland-1
gst-launch-1.0 filesrc location=/usr/local/test.mp4 ! qtdemux name=demux demux.video_0 ! queue ! h264parse ! qtic2vdec ! videoconvert ! waylandsink sync=true
```

视频编码（使用 qtic2venc 组件，按 Ctrl+C 停止编码）：

```bash
gst-launch-1.0 videotestsrc ! qtic2venc ! h264parse ! qtmux ! filesink location=test.mp4 -e
```
