# Camera 使用


* 接口效果图

![](../../../rv1126_img/C40PL/usage_camera_mipicsi.jpg)

* 注：摄像头上的红线和黑线用作切换滤光片使用。
* 命令切换滤光片
```
# 查看 SDK 版本
cd path-to-sdk/
realpath .repo/manifest.xml
```
* 若上述命令获取到的 SDK 版本`是` rv1126_rv1109_linux_release_20220324_v2.2.5b.xml 版本及日期为 2022.03.24 以上版本的则使用以下命令进行 IRCUT （滤光片切换）：
```
# 关闭滤光片
v4l2-ctl -d /dev/v4l-subdev5 --set-ctrl 'band_stop_filter=0'

# 打开滤光片
v4l2-ctl -d /dev/v4l-subdev5 --set-ctrl 'band_stop_filter=1'
```

* 若上述命令获取到的 SDK 版本`不是` rv1126_rv1109_linux_release_20220324_v2.2.5b.xml 版本及日期为 2022.03.24 以上版本的则使用以下命令进行 IRCUT （滤光片切换）：
```
# 关闭滤光片
v4l2-ctl -d /dev/v4l-subdev4 --set-ctrl 'band_stop_filter=0'

# 打开滤光片
v4l2-ctl -d /dev/v4l-subdev4 --set-ctrl 'band_stop_filter=1'
```


## MIPI CSI用法
* MIPI-CSI摄像头需要购买转接板
* v4l2接口操作MIPI-CSI摄像头
```
#判断驱动 probe 状态 RKISP 如果 probe 成功,会有 video 及 media 设备存在于 /dev/ 目录下。
#系统中可能存在多个 /dev/video 设备,通过 /sys 可以查询到RKISP注册的 video 节点。
#与之对应的是 rkispp_scale0 节点，如下打印也就是 /dev/video19
[root@RV1126_RV1109:/]# grep '' /sys/class/video4linux/video*/name
/sys/class/video4linux/video0/name:stream_cif_mipi_id0
/sys/class/video4linux/video1/name:stream_cif_mipi_id1
/sys/class/video4linux/video10/name:rkisp_rawwr3
/sys/class/video4linux/video11/name:rkisp_rawrd0_m
/sys/class/video4linux/video12/name:rkisp_rawrd2_s
/sys/class/video4linux/video13/name:rkisp_rawrd1_l
/sys/class/video4linux/video14/name:rkisp-statistics
/sys/class/video4linux/video15/name:rkisp-input-params
/sys/class/video4linux/video16/name:rkisp-mipi-luma
/sys/class/video4linux/video17/name:rkispp_input_image
/sys/class/video4linux/video18/name:rkispp_m_bypass
/sys/class/video4linux/video19/name:rkispp_scale0
/sys/class/video4linux/video2/name:stream_cif_mipi_id2
/sys/class/video4linux/video20/name:rkispp_scale1
/sys/class/video4linux/video21/name:rkispp_scale2
/sys/class/video4linux/video22/name:rkispp_iqtool
/sys/class/video4linux/video23/name:rkispp_input_params
/sys/class/video4linux/video24/name:rkispp-stats
/sys/class/video4linux/video3/name:stream_cif_mipi_id3
/sys/class/video4linux/video4/name:rkcif-mipi-luma
/sys/class/video4linux/video5/name:rkisp_mainpath
/sys/class/video4linux/video6/name:rkisp_selfpath
/sys/class/video4linux/video7/name:rkisp_rawwr0
/sys/class/video4linux/video8/name:rkisp_rawwr1
/sys/class/video4linux/video9/name:rkisp_rawwr2

# v4l2-ctl 抓取一帧图片并保存在 /tmp/cif.out
# /dev/video19 不是固定的，需要按照上面匹配得到相应的节点来配置
#抓取图片之前需要关闭默认的摄像头应用
/oem/RkLunch-stop.sh
/oem/usr/bin/dbserver &
/usr/bin/ispserver &

#开启 dbserver 和 ispserver 之后就可以进行抓图了
v4l2-ctl -d /dev/video19 \
--set-fmt-video=width=1920,height=1080,pixelformat=NV12 \
--stream-mmap=3 \
--stream-skip=3 \
--stream-to=/tmp/cif.out \
--stream-count=1 \
--stream-poll

# v4l2-ctl 抓取 10 帧连续图片作为一段视频并保存在 /tmp/test.yuv
v4l2-ctl -d /dev/video19  --try-fmt-video=width=1920,height=1080,pixelformat=NV12 --stream-mmap=3 --stream-to=/tmp/test.yuv --stream-count=10 --stream-poll

# Linux PC 使用 ffplay 播放抓取到的视频
ffplay -f rawvideo -video_size 1920x1080 -pix_fmt nv12 test.yuv
```

* 访问网络码流
使用支持 RTSP 或 RTMP 的播放器访问,例如( VLC 播放器)。
* RTSP访问地址:
```
rtsp://设备IP地址/live/mainstream
rtsp://设备IP地址/live/substream
rtsp://设备IP地址/live/thirdstream
```
* RTMP 访问地址:
```
rtmp://设备IP地址:1935/live/substream
```

例：VLC 播放器预览 RTSP 流
```
#注：需要安装 VLC 播放器
vlc rtsp://设备IP地址/live/mainstream
```
* 通过网页访问设备信息
打开 Web 浏览器(推荐 Chrome 浏览器)访问地址:
```
http://设备IP地址
#用户名和密码均是 admin
```
网页端详细的操作说明请参考 SDK 目录 `docs` 下的文档 `Rockchip_Instructions_Linux_Web_Configuration_CN.pdf`。


