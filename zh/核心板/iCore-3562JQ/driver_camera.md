# Camera 使用

* 接口效果图
![](../../../rk3562_img/iCore-3562JQ/mipicsi.jpg)

## MIPI CSI 用法
RK3562 平台有 2 个 4 lane dphy，每 lane 最高 2.5Gbps

### dphy

RK3562 有两个 4 lane 的 dphy，csi2_dphy0 和 csi2_dphy3，直接使用就是 full mode。

如果拆分使用就是 split mode：

csi2_dphy0 可以拆成 2 个 2 lane 的 dphy，分别为 csi2_dphy1 和 csi2_dphy2；

csi2_dphy3 可以拆成 2 个 2 lane 的 dphy，分别为 csi2_dphy4 和 csi2_dphy5。

### mipi_csi

RK3562 有 4 个 mipi_csi

来自 csi2_dphy0 (csi2_dphy1+csi2_dphy2) 的数据使用 mipi0_csi2 和 mipi1_csi2；

来自 csi2_dphy3 (csi2_dphy4+csi2_dphy5) 的数据使用 mipi2_csi2 和 mipi3_csi2。

### vicap

同理 vicap 有 4 个节点

来自 mipi0_csi2 和 mipi1_csi2 的数据使用 rkcif_mipi_lvds 和 rkcif_mipi_lvds1；

来自 mipi2_csi2 和 mipi3_csi2 的数据使用 rkcif_mipi_lvds2 和 rkcif_mipi_lvds3。

### rkisp_vir

isp 只有一个但支持 4 个节点：rkisp_vir0~3

![](../../../rk3562_img/iCore-3562JQ/rk3562_mipi_csi_mode.png)

## 配置举例
教程文档位于 SDK/docs/cn/Common/ISP/ISP32-lite/Rockchip_Driver_Guide_VI_CN_v1.1.4.pdf

### 双路 8MS1M 摄像头

800W 摄像头 8MS1M (XC7160_SC8238)，该摄像头自带 isp，所以链路走到 vicap 即可，如下：
```
xc7160_0 -> csi2_dphy0 -> mipi0_csi2 -> rkcif_mipi_lvds
xc7160_1 -> csi2_dphy3 -> mipi2_csi2 -> rkcif_mipi_lvds2
```
详细配置请查看 SDK/kernel/arch/arm64/boot/dts/rockchip/rk3562-firefly-aio-3562jq-dual-xc7160.dtsi

### 双路 IMX415 摄像头

IMX415 是 RAW 摄像头，因此最终需要接入 rkisp，链路如下:
```
imx415_0 -> csi2_dphy0 -> mipi0_csi2 -> rkcif_mipi_lvds -> rkcif_mipi_lvds_sditf -> rkisp
imx415_1 -> csi2_dphy3 -> mipi2_csi2 -> rkcif_mipi_lvds2 -> rkcif_mipi_lvds2_sditf ->rkisp
```
详细配置请查看 SDK/kernel/arch/arm64/boot/dts/rockchip/rk3562-firefly-aio-3562jq-dual-imx415.dtsi

rkisp 需要应用层 rkaiq_3A_server 服务处理数据才能出图（目前未完善）

## 摄像头配置的选择

设备树 SDK/kernel/arch/arm64/boot/dts/rockchip/rk3562-firefly-aio-3562jq.dts 中默认 include rk3562-firefly-aio-3562jq-dual-xc7160.dtsi

也就是默认使用双路 8MS1M，可修改成其他摄像头的 dtsi，并重新编译。

## Camera 底层调试
使用 v4l2-ctl 抓取 camera 数据帧
```shell
# 首先 v4l2-ctl 可以列出所有可用 video 设备
v4l2-ctl --list-devices

# 抓帧到文件 out.yuv
v4l2-ctl --verbose -d /dev/video0 --set-fmt-video=width=1920,height=1080,pixelformat='NV12' --stream-mmap=4 --set-selection=target=crop,flags=0,top=0,left=0,width=1920,height=1080 --stream-to=/data/out.yuv
```

把 out.yuv 文件拷贝出来可通过 ffmpeg 去播放
```shell
ffplay -f rawvideo -video_size 1920x1080 -pix_fmt nv12 out.yuv
```

## Android 系统使用 camera 应用
Android 系统使用 camera 的 apk 打开摄像头即可。

## Linux 系统预览摄像头

摄像头预览可以使用如下脚本，可自行修改其中 video 设备：
```bash
#!/bin/bash

export DISPLAY=:0
export XAUTHORITY=/home/firefly/.Xauthority
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/aarch64-linux-gnu/gstreamer-1.0
WIDTH=1920
HEIGHT=1080
SINK=xvimagesink

gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,format=NV12,width=${WIDTH},height=${HEIGHT}, framerate=30/1 ! videoconvert ! $SINK &

wait
```

## IQ 文件
raw 摄像头支持的 iq 文件路径`external/camera_engine_rkaiq/rkaiq/iqfiles/isp32_lite/`

若使用 raw 摄像头 sensor，请留意该摄像头对应的 iq 文件需要在设备`/etc/iqfiles/`下

