
# Camera 使用





## Camera 底层调试


* v4l2 接口操作 MIPI-CSI 摄像头

查找摄像头节点


```
$ grep '' /sys/class/video4linux/video*/name
/sys/class/video4linux/video0/name:stream_cif_mipi_id0
/sys/class/video4linux/video1/name:stream_cif_mipi_id1
/sys/class/video4linux/video10/name:rkcif_tools_id2
/sys/class/video4linux/video11/name:rkisp_mainpath
/sys/class/video4linux/video12/name:rkisp_selfpath
/sys/class/video4linux/video13/name:rkisp_iqtool
/sys/class/video4linux/video14/name:rkisp_rawrd0_m
/sys/class/video4linux/video15/name:rkisp_rawrd2_s
/sys/class/video4linux/video16/name:rkisp-statistics
/sys/class/video4linux/video17/name:rkisp-input-params
/sys/class/video4linux/video18/name:rkisp-pdaf
/sys/class/video4linux/video19/name:rkvpss-offline
/sys/class/video4linux/video2/name:stream_cif_mipi_id2
/sys/class/video4linux/video20/name:rkvpss_scale0
/sys/class/video4linux/video21/name:rkvpss_scale1
/sys/class/video4linux/video22/name:rkvpss_scale2
/sys/class/video4linux/video23/name:rkvpss_scale3
/sys/class/video4linux/video24/name:rkvpss_scale4
/sys/class/video4linux/video25/name:rkvpss_scale5
/sys/class/video4linux/video3/name:stream_cif_mipi_id3
/sys/class/video4linux/video4/name:rkcif_scale_ch0
/sys/class/video4linux/video5/name:rkcif_scale_ch1
/sys/class/video4linux/video6/name:rkcif_scale_ch2
/sys/class/video4linux/video7/name:rkcif_scale_ch3
/sys/class/video4linux/video8/name:rkcif_tools_id0
/sys/class/video4linux/video9/name:rkcif_tools_id1
```

* 确定抓取的节点

1. 对于使用 RKISP 的摄像头如 SC3336 需要抓取 rkisp_mainpath 对应的 video 节点。从上述的输出信息来看，rkisp_mainpath 对应 video11 节点。

   如果使用 IMX415 摄像头，则使用 v4l2-ctl 抓取 camera 数据帧并保存在 /data/out.yuv 。
   ```
   v4l2-ctl --verbose -d /dev/video11 --set-fmt-video=width=1920,height=1080,pixelformat='NV12' --stream-mmap=3 --stream-skip=3 --stream-to=/data/out.yuv
   ```

   将 out.yuv 文件拷贝出来通过 ubuntu 去查看
   ```
   ffplay -f rawvideo -video_size 1920x1080 -pix_fmt nv12 out.yuv
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
```
.
├── etc
│   ├── iqfiles
│   │   ├── imx335_default_default.json
│   │   └── imx415_CMK-OT2022-PX1_IR0147-50IRC-8M-F20.json
│   │   
│   └── rc.local
└── usr
    ├── lib
    │   └── aarch64-linux-gnu
    │       └── libff_media.so
    └── local
        └── bin
            ├── demo
            └── ffmedia_test.sh

7 directories, 7 files
```
上述文件在固件中已经添加,连接网络后将会固定IP(eth0)为:192.168.1.100,网关:192.168.1.1

2.Linux系统命令方式打开预览:
```
ffplay -rtsp_transport tcp -x 640 -y 480 -an \
  "rtsp://192.168.1.100:8554/live/test"
```

3.Windows/Linux系统下载VLC媒体播放器
```
打开 媒体 -> 打开网络串流
输入: rtsp://192.168.1.100:8554/live/test
点击播放预览
```

