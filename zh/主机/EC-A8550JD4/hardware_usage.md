# 硬件功能使用
## 调试串口

EC-A8550JD4 整机的调试串口接口形式待补充。其采用的 AIO-8550JD4 主板提供 3pin TTL 插槽和 Type-C 接口（与烧录口共用）两种调试串口接口形式；如需使用 3pin TTL 插槽，要外接 USB 转 TTL 串口模块，模块的连接与使用方法详见[串口模块](usb_to_ttl.md)。

## 显示接口

EC-A8550JD4 提供 1 个 HDMI 2.0 显示输出接口，接上显示器即可显示系统画面。

常用操作如下：

* 分辨率与刷新率：在桌面的系统显示设置中选择输出分辨率与刷新率。
* 多屏：整机仅提供 1 个 HDMI 显示输出接口，无多屏输出。
* 亮灭屏：可在系统设置的电源选项中配置自动息屏时间。

也可以通过命令行查看显示接口的连接状态：

```bash
# connected 表示已连接，连接器名称以实际系统为准
cat /sys/class/drm/*/status
```

## 串口（RS232 / RS485）

EC-A8550JD4 提供 RS232、RS485 串口接口。串口的数量、接线位置与系统设备节点的对应关系以实际产品为准。

串口可按如下方法进行收发测试（`/dev/ttyX` 代指对应的设备节点，请先通过 `ls /dev/tty*` 确认）：

```bash
# 查看系统中的串口设备
ls /dev/tty*

# 设置串口参数（波特率 115200、数据位 8、停止位 1、无奇偶校验）
stty -F /dev/ttyX 115200 cs8 -cstopb -parenb

# 短接串口的发送（TX）与接收（RX）后，进行自发自收测试
cat /dev/ttyX &
echo "serial_test" > /dev/ttyX
```

## USB 接口

EC-A8550JD4 提供 USB3.0 接口，可外接 USB 键鼠、U 盘等 USB 设备。另有 Type-C 烧录口，固件升级方法请参阅[升级固件](qfil_upgrade_firmware.md)与[升级分区镜像](fastboot_upgrade_image.md)。

以 U 盘为例，插入后可在系统中查看并挂载：

```bash
# 查看识别到的存储设备
lsblk

# 挂载 U 盘（设备名以实际识别为准）
mkdir -p /mnt/udisk
mount /dev/sda1 /mnt/udisk

# 使用完成后卸载
umount /mnt/udisk
```

## 存储扩展

EC-A8550JD4 支持 SSD 存储扩展，安装位置如下图所示。

<center>

<img alt="" src="../../../qcom_img/EC-A8550JD4/ec-a8550jd4-ssd-zh.jpg" width="700">
</center>

## 音频

EC-A8550JD4 基于 AIO-8550JD4 主板：主板 1 路 I2S 接入 HDMI 用于音频播放，另一路 I2S 接入底板声卡（3.5mm 耳机孔）；整机实际引出的音频接口以实际产品为准。

底层播放仅支持 48K 采样率、16 bit 的 wav 音频，需要使用 agmplay / agmcap 工具：

```bash
# 设置声卡录音通路
amixer -q set "PGAL Select" "Line 2P"
amixer -q set "PGAR Select" "Line 2N"

# 设置录音音量
amixer -q set "ADCL" "200"
amixer -q set "ADCR" "200"

# 录音 5 秒
agmcap -D 100 -d 101 -i MI2S-LPAIF-TX-PRIMARY -dkv 0xA3000004 -c 2 -r 48000 -b 16 -T 5 test.wav

# 设置播放音量
amixer -q set "DACL" "200"
amixer -q set "DACR" "200"

# 向耳机孔播放声音
agmplay test.wav -D 100 -d 100 -i MI2S-LPAIF-RX-PRIMARY -dkv 0xA2000001

# 向 HDMI 播放声音
agmplay test.wav -D 100 -d 100 -i MI2S-LPAIF-RX-SECONDARY -dkv 0xA2000004
```

桌面环境下，可在系统的声音设置中选择音频输出设备并调节音量（以实际固件支持的音频服务为准）。

## 视频

EC-A8550JD4 的视频编解码能力与 AIO-8550JD4 主板一致，默认的视频框架为 Gstreamer。使用 `gst-inspect-1.0` 可以查看系统支持的多媒体组件，其中 `qtic2vdec`（H.264/H.265/VP8/VP9/MPEG 视频解码）、`qtic2venc`（H.264/H.265/HEIC 视频编码）等 qti 组件享有硬件加速：

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

## NPU（AI）

QCS8550 集成 48 TOPS NPU（Hexagon DSP），AI 部署与开发方法详见[AI 教程](usage_npu.md)。
