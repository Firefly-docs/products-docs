# 视频教程

AIBOX-9075 默认的视频框架为 Gstreamer.

## 可用 gst 组件

使用 gst-inspect 可以查看可用的多媒体组件，这些 v4l 组件享受硬件加速功能。

```bash
firefly@ubuntu:~$ gst-inspect-1.0 --plugin | grep v4l2
video4linux2:  v4l2av1dec: V4L2 AV1 Decoder
video4linux2:  v4l2deviceprovider (GstDeviceProviderFactory)
video4linux2:  v4l2h264dec: V4L2 H264 Decoder
video4linux2:  v4l2h264enc: V4L2 H.264 Encoder
video4linux2:  v4l2h265dec: V4L2 H265 Decoder
video4linux2:  v4l2h265enc: V4L2 H.265 Encoder
video4linux2:  v4l2radio: Radio (video4linux2) Tuner
video4linux2:  v4l2sink: Video (video4linux2) Sink
video4linux2:  v4l2src: Video (video4linux2) Source
video4linux2:  v4l2vp9dec: V4L2 VP9 Decoder
```

## 编解码能力

* decode

1x 8K60, 2x 8K30, 4x 4K60, 8x 4K30, 16x 1080p60, 32x 1080p30

Formats: AV1, H.264, H.265, VP9, MPEG2

* encode

2x 4K60, 4x 4K30, 8x 1080p60, 16x 1080p30

Formats: H.264, H.265, HEIF/ HEIC

## 视频播放

例如播放 h264 视频，使用 v4l2h264dec 组件：
```bash
export XDG_RUNTIME_DIR=/run/user/1000
export WAYLAND_DISPLAY=wayland-0
gst-launch-1.0 filesrc location=/usr/local/test.mp4 ! qtdemux name=demux demux.video_0 ! queue ! h264parse ! v4l2h264dec ! glimagesink sync=true
```

## 视频编码

例如编码 h264 视频，使用 v4l2h264enc 组件：
```bash
gst-launch-1.0 videotestsrc ! v4l2h264enc ! h264parse ! qtmux ! filesink location=test.mp4 -e

# 按 Ctrl+C 停止编码
```