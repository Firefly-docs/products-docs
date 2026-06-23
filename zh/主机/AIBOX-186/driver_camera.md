# CAMERA 使用

AIBOX-186 引出 3 个 `MIPI CSI` 接口（1*4 lanes / 2*2 lanes），单一 sensor 最大支持 8K@15fps 线性输入或 12M@60fps HDR 输入。

##  MIPI CSI 
[接口位置如图所示](interface_definition.html#zheng-ji-jie-kou-ding-yi)  

### 使用命令抓图

以 Firelfy 的 [CAM-8MS1M](https://wiki.t-firefly.com/CAM-8MS1M/cam-8ms1m.html])摄像头为例（连接 MIPI CSI0）：
进入系统之后，通过 `sudo -s`获取 root 权限，并执行以下抓图命令。
```
insmod /mnt/system/ko/v4l2_xc7160.ko # 安装驱动
systemctl stop SophonHDMI.service # 关闭 HDMI 的显示界面
/opt/sophon/sophon-soc-libisp_1.0.0/bin/ispv4l2_ut 5 # 抓取 v4l2 流
/opt/sophon/sophon-ffmpeg-latest/bin/ffmpeg -s 1920x1080 -pix_fmt yuyv422 -i v4l2_video0_7.yuv -hide_banner -loglevel error  -vf format=rgb24 -c:v rawvideo -f rawvideo -vframes 1 -  | sudo dd of=/dev/fb0 bs=1920x1080x3 # 这里假设你接的 HDMI 屏幕是 1920 x 1080 分辨率的
systemctl restart SophonHDMI.service  # 恢复 HDMI 的显示界面
```

目前系统默认只打开了 MIPI CSI0，如果需要使用 MIPI CSI1/2 ，则需要修改 SDK 配置以及 ko 文件。 

