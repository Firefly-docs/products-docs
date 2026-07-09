
# Camera 使用




**AIO-1126BQ38** 底板可使用 24pin 的 MIPI CSI0 接口和 30pin 的 MIPI CSI1/2 接口。MIPI CSI3/4 接口为无功能接口。



## Camera 底层调试


* v4l2 接口操作 MIPI-CSI 摄像头

查找摄像头节点

```
$ grep '' /sys/class/video4linux/video*/name
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
/sys/class/video4linux/video24/name:rkisp_iqtool
/sys/class/video4linux/video25/name:rkisp_rawrd0_m
/sys/class/video4linux/video26/name:rkisp_rawrd2_s
/sys/class/video4linux/video27/name:rkisp-statistics
/sys/class/video4linux/video28/name:rkisp-input-params
/sys/class/video4linux/video29/name:rkisp-pdaf
/sys/class/video4linux/video3/name:stream_cif_mipi_id3
/sys/class/video4linux/video30/name:rkisp_mainpath
/sys/class/video4linux/video31/name:rkisp_selfpath
/sys/class/video4linux/video32/name:rkisp_iqtool
/sys/class/video4linux/video33/name:rkisp_rawrd0_m
/sys/class/video4linux/video34/name:rkisp_rawrd2_s
/sys/class/video4linux/video35/name:rkisp-statistics
```

* 确定抓取的节点

  由于一款主板可能存在多个摄像头，分两种情况：

1. 对于自带 ISP 的摄像头如 CAM-8MS1M 则是抓取 stream_cif_mipi_id0 对应的 video 节点。从上述的输出信息来看，stream_cif_mipi_id0 对应 video0和videod11节点。
   如果使用 CAM-8MS1M 摄像头，则使用 v4l2-ctl 抓取 camera 数据帧并保存在 /data/out.yuv 。
   ```
   v4l2-ctl --verbose -d /dev/video0 --set-fmt-video=width=1920,height=1080,pixelformat='NV12' --stream-mmap=3 --stream-skip=3 --stream-to=/data/out.yuv
   ```
   将 out.yuv 文件拷贝出来通过 ubuntu 去查看
   ```
   ffplay -f rawvideo -video_size 1920x1080 -pix_fmt nv12 out.yuv
   ```
   
2. 对于使用 RKISP 的摄像头如 IMX415 需要抓取 rkisp_mainpath 对应的 video 节点。从上述的输出信息来看，rkisp_mainpath 对应 video22和video30节点，22对应csi0,30对应csi1。s

   如果使用 IMX415 摄像头，则使用 v4l2-ctl 抓取 camera 数据帧并保存在 /data/out.yuv 。
   ```
   v4l2-ctl --verbose -d /dev/video22 --set-fmt-video=width=3840,height=2160,pixelformat='NV12' --stream-mmap=3 --stream-skip=3 --stream-to=/data/out.yuv
   ```
   将 out.yuv 文件拷贝出来通过 ubuntu 去查看
   ```
   ffplay -f rawvideo -video_size 3840x2160 -pix_fmt nv12 out.yuv
   ```
   如果带有屏幕的话，也可以使用gst-launch预览摄像头
   ```
   gst-launch-1.0 v4l2src device=/dev/video22 ! queue ! video/x-raw,width=640,height=480 ! kmssink sync=false 
   ```


## PHY 介绍
RV1126B 芯片 2 个 DPHY, 两个 DPHY 可以工作在两个模式: full mode 和 split mode。

简单点来讲，如果用单目摄像头我们可以配置 full mode，若使用双目摄像头我们可以配置 split mode。

硬件设计决定软件链路，配置如下：

csi2_dphy0 -> csi0(rx0) clk0 + 4 lane

csi2_dphy1 -> csi0(rx0) clk0 + 2 lane 0/1

csi2_dphy2 -> csi0(rx0) clk1 + 2 lane 2/3

csi2_dphy3 -> csi1(rx1) clk0 + 4 lane

csi2_dphy4 -> csi1(rx1) clk0 + 2 lane 0/1

csi2_dphy5 -> csi1(rx1) clk1 + 2 lane 2/3

## Full Mode 配置

配置链路为：

csi2_dphy0 –> mipi0_csi2 –> rkcif_mipi_lvds

csi2_dphy3 –> mipi2_csi2 –> rkcif_mipi_lvds2

详情请查看设备树：

rv1126b-firefly-aio-1126bjd4-csi0-imx415.dtsi

rv1126b-firefly-aio-1126bjd4-csi1-imx415.dtsi

## Split Mode 配置

配置链路为：

csi2_dphy1 –> mipi0_csi2 –> rkcif_mipi_lvds

csi2_dphy2 –> mipi0_csi2 –> rkcif_mipi_lvds

csi2_dphy4 –> mipi2_csi2 –> rkcif_mipi_lvds2

csi2_dphy5 –> mipi2_csi2 –> rkcif_mipi_lvds2

详情请查看设备树：

rv1126b-evb-dual-cam-csi0.dtsi

rv1126b-evb-dual-cam-csi1.dtsi

## Linux 系统预览摄像头

摄像头画面可以使用 ffmedia 进行预览。ffmedia 安装点击跳转：[ffmedia 教程](https://wiki.t-firefly.com/zh_CN/Firefly-Linux-Guide/manual_ubuntu.html#ffmedia)

安装好 ffmedia 后，根据上述描述选好 rkisp_mainpath 所对应的 video 节点进行 rtsp 推流命令。假设 rkisp_mainpath 对应的是 video11 节点。以 video11 进行推流 rtsp 为例。
```bash
./demo /dev/video11 -e h264 --port 8554  --push_type rtsp --push_path /live/test
```

PC 端使用 vlc 软件打开以下 rtsp 流媒体链接进行预览
```
rtsp://<设备IP>:8554/live/test
```

