# Camera Usage

Firefly-Q6490A supports 2 IMX577 cameras

## Preview
```bash
export XDG_RUNTIME_DIR=/run/user/1000
export WAYLAND_DISPLAY=wayland-0

# Open camera 0
gst-launch-1.0 -e qtiqmmfsrc camera=0 ! \
        video/x-raw,format=NV12,width=1280,height=720,framerate=30/1 ! \
        videoconvert ! waylandsink sync=true &

# Open camera 1
gst-launch-1.0 -e qtiqmmfsrc camera=1 ! \
        video/x-raw,format=NV12,width=1280,height=720,framerate=30/1 ! \
        videoconvert ! waylandsink sync=true &
```