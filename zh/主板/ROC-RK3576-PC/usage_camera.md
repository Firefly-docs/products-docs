# Camera 使用

* 接口效果图

![](../../../rk3576_img/ROC-RK3576-PC/usage_camera_mipicsi.jpg)

## PHY 介绍
RK3576 芯片有 1 个 DCPHY 和 2 个 DPHY, 两个 DPHY 可以工作在两个模式: full mode 和 split mode。DCPHY 则只有 full mode。

简单点来讲，如果用单目摄像头我们可以配置 full mode，若使用双目摄像头我们可以配置 split mode。

**目前ROC-RK3576-PC硬件只引出 1 个 DPHY。**

## Full Mode 配置

配置链路为：csi2_dphy0 --> mipi1_csi2 --> rkcif_mipi_lvds1

详情请查看设备树：rk3576-roc-pc-cam-8ms1m.dtsi


## Camera底层调试
查找摄像头节点
```shell
# 由于一款主板可能存在多个摄像头，对于使用RKISP的摄像头如 CAM-8MS1M(IMX415) 需要抓取rkisp_mainpath对应的video节点
# 对于自带ISP的摄像头如 CAM-8MS1M 则是抓取stream_cif_mipi_id0 对应的video节点
root@firefly:~# grep '' /sys/class/video4linux/video*/name
/sys/class/video4linux/video0/name:stream_cif_mipi_id0
/sys/class/video4linux/video1/name:stream_cif_mipi_id1
/sys/class/video4linux/video10/name:rkcif_tools_id2
/sys/class/video4linux/video11/name:stream_cif_mipi_id0
/sys/class/video4linux/video12/name:stream_cif_mipi_id1
/sys/class/video4linux/video13/name:stream_cif_mipi_id2
/sys/class/video4linux/video14/name:stream_cif_mipi_id3
/sys/class/video4linux/video15/name:rkcif_scale_ch0
/sys/class/video4linux/video16/name:rkcif_scale_ch1
/sys/class/video4linux/video17/name:rkcif_scale_ch2
/sys/class/video4linux/video18/name:rkcif_scale_ch3
/sys/class/video4linux/video19/name:rkcif_tools_id0
/sys/class/video4linux/video2/name:stream_cif_mipi_id2
/sys/class/video4linux/video20/name:rkcif_tools_id1
/sys/class/video4linux/video21/name:rkcif_tools_id2
/sys/class/video4linux/video22/name:rkisp_mainpath
/sys/class/video4linux/video23/name:rkisp_selfpath
/sys/class/video4linux/video24/name:rkisp_fbcpath
/sys/class/video4linux/video25/name:rkisp_iqtool
/sys/class/video4linux/video26/name:rkisp_rawrd0_m
/sys/class/video4linux/video27/name:rkisp_rawrd2_s
/sys/class/video4linux/video28/name:rkisp_rawrd1_l
/sys/class/video4linux/video29/name:rkisp-statistics
/sys/class/video4linux/video3/name:stream_cif_mipi_id3
/sys/class/video4linux/video30/name:rkisp-input-params
/sys/class/video4linux/video31/name:stream_hdmirx
/sys/class/video4linux/video4/name:rkcif_scale_ch0
/sys/class/video4linux/video5/name:rkcif_scale_ch1
/sys/class/video4linux/video6/name:rkcif_scale_ch2
/sys/class/video4linux/video7/name:rkcif_scale_ch3
/sys/class/video4linux/video8/name:rkcif_tools_id0
/sys/class/video4linux/video9/name:rkcif_tools_id1
```
使用v4l2-ctl抓取camera数据帧
```shell
v4l2-ctl --verbose -d /dev/video0 --set-fmt-video=width=1920,height=1080,pixelformat='NV12' --stream-mmap=4 --set-selection=target=crop,flags=0,top=0,left=0,width=1920,height=1080 --stream-to=/data/out.yuv
```

把out.yuv文件拷贝出来通过ubuntu去查看
```shell
ffplay -f rawvideo -video_size 1920x1080 -pix_fmt nv12 out.yuv
```

## Android 系统使用 Camera 应用
除了官方默认支持的摄像头外，Android系统使用camera的apk打开摄像头都需要配置camera3_profiles*.xml，具体可参考Android SDK `hardware/rockchip/camera/etc/camera`目录下的文件

## Linux 系统预览摄像头
Ubuntu 固件可以使用如下脚本进行预览。
```shell
#!/bin/bash

export DISPLAY=:0.0
export XDG_RUNTIME_DIR=/run/user/1000
echo "Start MIPI CSI Camera Preview!"

gst-launch-1.0 v4l2src device=/dev/video0 ! queue ! video/x-raw,width=800,height=600 ! videoconvert ! xvimagesink
```


