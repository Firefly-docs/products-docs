# 摄像头教程

AIO-6490JD4 支持 3 个 IMX577 摄像头

## 预览
```bash
export XDG_RUNTIME_DIR=/run/user/1000
export WAYLAND_DISPLAY=wayland-0

# 打开摄像头 0
gst-launch-1.0 -e qtiqmmfsrc camera=0 ! \
        video/x-raw,format=NV12,width=1280,height=720,framerate=30/1 ! \
        videoconvert ! waylandsink sync=true &

# 打开摄像头 1
gst-launch-1.0 -e qtiqmmfsrc camera=1 ! \
        video/x-raw,format=NV12,width=1280,height=720,framerate=30/1 ! \
        videoconvert ! waylandsink sync=true &
# 打开摄像头 2
gst-launch-1.0 -e qtiqmmfsrc camera=2 ! \
        video/x-raw,format=NV12,width=1280,height=720,framerate=30/1 ! \
        videoconvert ! waylandsink sync=true &
```