# Camera
* Hardware interface

![](../../../rk3576_img/EC-A3576JD4/usage_camera_mipicsi.jpg)

## MIPI CSI
RK3576 platform has one DCPHY and two DPHY. DPHY can work in two modes: full mode and split mode; DCPHY only has full mode.

In short, if we use single-camera, we can set dphy to full mode, if we use dual-camera, we can set dphy to split mode.


## Full Mode Configuration

## Camera Debug
Find the camera node
```shell
# Since a board may have multiple cameras, for cameras using RKISP such as 8Mega Pixel HD Camera(IMX415), 
# it is necessary to use the video node named rkisp_mainpath.
# For cameras with their own ISP, such as 8Mega Pixel HD Camera, 
# it is necessary to use the video node named stream_cif_mipi_id0.
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

Use v4l2-ctl to capture camera frame data
```shell
v4l2-ctl --verbose -d /dev/video0 --set-fmt-video=width=1920,height=1080,pixelformat='NV12' --stream-mmap=4 --set-selection=target=crop,flags=0,top=0,left=0,width=1920,height=1080 --stream-to=/data/out.yuv
```

Copy file out.yuv to ubuntu to play
```shell
ffplay -f rawvideo -video_size 1920x1080 -pix_fmt nv12 out.yuv
```

## Android Use Camera App
In addition to the official default supported cameras, To open camera with camera app in Android needs configuring in camera3_profiles*.xml. For details please refer to files in Android SDK under `hardware/rockchip/camera/etc/camera`.

## Linux Preview Camera
Ubuntu firmware can use this script to view camera stream:
```shell
#!/bin/bash

export DISPLAY=:0.0
export XDG_RUNTIME_DIR=/run/user/1000
echo "Start MIPI CSI Camera Preview!"

gst-launch-1.0 v4l2src device=/dev/video0 ! queue ! video/x-raw,width=800,height=600 ! videoconvert ! xvimagesink
```


