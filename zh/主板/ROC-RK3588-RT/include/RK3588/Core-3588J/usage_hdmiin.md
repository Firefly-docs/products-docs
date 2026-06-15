# HDMI-IN 使用  
ROC-RK3588-RT 有一个 HDMI-IN 接口, 支持标准的 HDMI2.0 协议，它具备以下的特性：  

* HDMI 1.4b/2.0 RX: Up to 4K@60fps
* Support FMT: RGB888/YUV420/YUV422/YUV444 8bit
* Pixel clock: Up to 600MHz
* HDCP1.4/2.3
* CEC hardware engine
* E-EDID configuration
* S/PDIF 2channel output
* I2S 2/4/6/8channel output


接口图如下：
![](../../../rk3588_img/ROC-RK3588-RT/usage_hdmiin_interface.jpg)  

目前有 APK 及命令的方式来进行 HDMI-IN 的功能调试。

## Android 使用 HDMI-IN

默认 ROC-RK3588-RT 的 Android 系统中会带有两个 APK ，一个名为 **Live Tv**，另一个名为 **RockchipCamera2**，如下图所示：
![](../../../rk3588_img/common/usage_hdmiin_rk3588_apk.png)  

直接点击 APK 便可进行 HDMI-IN 的视频输入显示，HDMI-IN 的音频输入会从 ROC-RK3588-RT 的耳机、HDMI、Display Port 等接口输出。

需要注意的是，在软件层上 **Live Tv** 默认走的是 Tv Hal 层，**RockchipCamera2** 默认走的是 USB Camera Hal 层。

且 **RockchipCamera2** 默认只支持 NV12，NV16，RGB3 三种输入格式（所以如果输入源格式不对的话，会导致该 APK 打开黑屏），支持的最大输入分辨率为 3840x2160@30fps。而 **Live Tv** 则没有这样的限制。

SDK 中 APK 对应的源码路径：  
```
Live Tv ： path/to/sdk/packages/apps/TV/  
RockchipCamera2 ： path/to/sdk/packages/apps/rkCamera2/
```


## Linux 使用 HDMI-IN
Ubuntu 固件集成了 `test_hdmirx.sh` 测试脚本，直接运行该脚本就可以了，脚本路径：`/usr/local/bin/test_hdmirx.sh`。

```shell
#!/bin/bash

device_id=$(v4l2-ctl --list-devices | grep -A1 hdmirx | grep -v hdmirx | awk -F ' ' '{print $NF}')

v4l2-ctl -d $device_id --set-dv-bt-timings query

width=$(v4l2-ctl -d $device_id --get-dv-timings | grep "Active width" |awk -F ' ' '{print $NF}')
heigh=$(v4l2-ctl -d $device_id --get-dv-timings | grep "Active heigh" |awk -F ' ' '{print $NF}')

trap 'onCtrlC' INT
function onCtrlC () {
        echo 'Ctrl+C is captured'
        killall gst-launch-1.0
        exit 0
}

export XDG_RUNTIME_DIR=/run/user/1000
gst-launch-1.0 alsasrc device=hw:2,0 ! audioconvert ! audioresample ! queue !  alsasink device="hw:1,0" &
gst-launch-1.0 v4l2src device=$device_id ! queue ! video/x-raw,format=RGB ! capssetter replace = true caps="video/x-raw,format=BGR,width=$width,height=$heigh" ! glimagesink &

echo "[Ctrl + C] exit"
while true
do
        sleep 10
done
```

## 调试命令

### 视频调试  

HDMI-IN 设备在内核中会被注册为 `video` 设备，生成的节点如：`/dev/video8`，可以通过 `v4l2-ctl` 命令来获取设备信息和抓帧。  

* 获取设备信息  
	```
	:/ # v4l2-ctl -d /dev/video8  -V -D
	Driver Info:
	        Driver name      : rk_hdmirx
	        Card type        : rk_hdmirx
	        Bus info         : fdee0000.hdmirx-controller
	        Driver version   : 5.10.66
	        Capabilities     : 0x84201000
	                Video Capture Multiplanar
	                Streaming
	                Extended Pix Format
	                Device Capabilities
	        Device Caps      : 0x04201000
	                Video Capture Multiplanar
	                Streaming
	                Extended Pix Format

	```
* 获取外部设备输入的分辨率信息
	```
	:/ # v4l2-ctl -d /dev/video8  -V                 
	Format Video Capture Multiplanar:
	        Width/Height      : 3840/2160
	        Pixel Format      : 'NV12'
	        Field             : None
	        Number of planes  : 1
	        Flags             : premultiplied-alpha, 000000fe
	        Colorspace        : Unknown (1220e180)
	        Transfer Function : Unknown (00000020)
	        YCbCr Encoding    : Unknown (000000ff)
	        Quantization      : Default
	        Plane 0           :
	           Bytes per Line : 3840
	           Size Image     : 16588800
	```

* 抓取帧率（需要设置好分辨率和像素格式）
	```
	:/ # v4l2-ctl  -d /dev/video8 --set-fmt-video=width=3840,height=2160,pixelformat='NV12' --stream-mmap=4  --stream-skip=10  --stream-poll --

	VIDIOC_REQBUFS: ok
	VIDIOC_QUERYBUF: ok
	VIDIOC_QUERYBUF: ok
	VIDIOC_QBUF: ok
	VIDIOC_QUERYBUF: ok
	VIDIOC_QBUF: ok
	VIDIOC_QUERYBUF: ok
	VIDIOC_QBUF: ok
	VIDIOC_QUERYBUF: ok
	VIDIOC_QBUF: ok
	VIDIOC_STREAMON: ok
	idx: 0 seq:      0 bytesused: 16588800 ts: 617.869692
	idx: 1 seq:      1 bytesused: 16588800 ts: 617.888339 delta: 18.647 ms
	idx: 2 seq:      2 bytesused: 16588800 ts: 617.904531 delta: 16.192 ms
	idx: 3 seq:      3 bytesused: 16588800 ts: 617.921665 delta: 17.134 ms
	idx: 0 seq:      4 bytesused: 16588800 ts: 617.937899 delta: 16.234 ms fps: 58.65
	idx: 1 seq:      5 bytesused: 16588800 ts: 617.954531 delta: 16.632 ms fps: 58.94
	idx: 2 seq:      6 bytesused: 16588800 ts: 617.971662 delta: 17.131 ms fps: 58.84
	idx: 3 seq:      7 bytesused: 16588800 ts: 617.987897 delta: 16.235 ms fps: 59.22
	idx: 0 seq:      8 bytesused: 16588800 ts: 618.004531 delta: 16.634 ms fps: 59.33
	idx: 1 seq:      9 bytesused: 16588800 ts: 618.021230 delta: 16.699 ms fps: 59.39
	idx: 2 seq:     10 bytesused: 16588800 ts: 618.037864 delta: 16.634 ms fps: 59.46
	idx: 3 seq:     11 bytesused: 16588800 ts: 618.054561 delta: 16.697 ms fps: 59.50
	idx: 0 seq:     12 bytesused: 16588800 ts: 618.071196 delta: 16.635 ms fps: 59.55
	idx: 1 seq:     13 bytesused: 16588800 ts: 618.088322 delta: 17.126 ms fps: 59.46
	idx: 2 seq:     14 bytesused: 16588800 ts: 618.104528 delta: 16.206 ms fps: 59.62
	idx: 3 seq:     15 bytesused: 16588800 ts: 618.121653 delta: 17.125 ms fps: 59.53
	idx: 0 seq:     16 bytesused: 16588800 ts: 618.137861 delta: 16.208 ms fps: 59.66
	idx: 1 seq:     17 bytesused: 16588800 ts: 618.154989 delta: 17.128 ms fps: 59.59
	idx: 2 seq:     18 bytesused: 16588800 ts: 618.171227 delta: 16.238 ms fps: 59.69

	```

* 把抓取到的帧保存为文件,这里表示抓一帧并保存在 `data/4kp60_nv12.yuv`

	```
	:/ # v4l2-ctl  -d /dev/video8 --set-fmt-video=width=3840,height=2160,pixelformat='NV12' --stream-mmap=4  --stream-skip=10  --stream-to=/data/4kp60_nv12.yuv --stream-count=1 --stream-poll
	<<<<<<<<<<<

	```

* 在 PC 端查看抓取到的帧文件
	* Window 端可以使用 7yuv 工具

	* Linux 端可以使用 ffplay 命令
		```
		ffplay -f rawvideo -video_size 3840x2160 -pixel_format  nv12 4kp60_nv12.yuv
		```

### 音频调试  

可以通过 `tinycap` 命令去录制输入的音频。

* 查看声卡设备
	``` 
	:/ # cat /proc/asound/card*

	 0 [rockchipdp0    ]: rockchip_dp0 - rockchip,dp0
	                      rockchip,dp0
	 1 [rockchipes8388 ]: rockchip-es8388 - rockchip-es8388
	                      rockchip-es8388
	 2 [rockchiphdmiin ]: rockchip_hdmiin - rockchip,hdmiin
	                      rockchip,hdmiin
	 3 [rockchiphdmi0  ]: rockchip-hdmi0 - rockchip-hdmi0
	                      rockchip-hdmi0
	 4 [rockchiphdmi1  ]: rockchip-hdmi1 - rockchip-hdmi1
	                      rockchip-hdmi1

	```
可以看到，hdminrx（hdmiin）的声卡号为`2`，可以在 HDMI-IN 有音频输入的时候运行以下命令来录制和播放音频。

* 录制音频
	```
	:/ # tinycap /sdcard/test.wav -D 2 -d 0
	```


* 播放录制到的音频（指定 HDMI0 作声卡输出）

	```
	:/ # tinyplay /sdcard/test.wav -D 3 -d 0
	```