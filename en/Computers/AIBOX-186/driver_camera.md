# CAMERA Usage

The AIBOX-186 features 3 `MIPI CSI` interfaces (1*4 lanes / 2*2 lanes), with a single sensor supporting a maximum of 8K@15fps linear input or 12M@60fps HDR input.

## MIPI CSI
[Interface location as shown in the diagram](interface_definition.html#zheng-ji-jie-kou-ding-yi)

### Command for Capturing Images

Taking the Firefly's [CAM-8MS1M](https://wiki.t-firefly.com/CAM-8MS1M/cam-8ms1m.html) camera as an example (connected to MIPI CSI0):
After entering the system, gain root privileges using `sudo -s`, and execute the following capture command.
```
insmod /mnt/system/ko/v4l2_xc7160.ko # Install driver
systemctl stop SophonHDMI.service # Stop the HDMI display interface
/opt/sophon/sophon-soc-libisp_1.0.0/bin/ispv4l2_ut 5 # Capture v4l2 stream
/opt/sophon/sophon-ffmpeg-latest/bin/ffmpeg -s 1920x1080 -pix_fmt yuyv422 -i v4l2_video0_7.yuv -hide_banner -loglevel error  -vf format=rgb24 -c:v rawvideo -f rawvideo -vframes 1 -  | sudo dd of=/dev/fb0 bs=1920x1080x3 # Assuming your connected HDMI screen has a resolution of 1920 x 1080
systemctl restart SophonHDMI.service  # Restore the HDMI display interface
```

Currently, the system only has MIPI CSI0 enabled by default. If MIPI CSI1/2 is needed, the SDK configuration and ko files need to be modified.