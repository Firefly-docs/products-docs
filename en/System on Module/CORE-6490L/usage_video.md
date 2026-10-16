# Video Usage

The default video framework of Firefly-Q6490A is Gstreamer.

## Available gst plugins

Use gst-inspect to check the available multimedia plugins. These v4l plugins enjoy hardware acceleration.

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

## Codec capability

* decode

1x 4K60, 2x 4K30, 4x 1080p60

Formats: H.264, H.265, VP9

* encode

1x 4K30, 4x 1080p30

Formats: H.264, H.265

## Video Playback

For example, to play an h264 video, use the v4l2h264dec plugin:
```bash
export XDG_RUNTIME_DIR=/run/user/1000
export WAYLAND_DISPLAY=wayland-0
gst-launch-1.0 filesrc location=/usr/local/test.mp4 ! qtdemux name=demux demux.video_0 ! queue ! h264parse ! v4l2h264dec ! glimagesink sync=true
```

## Video Encoding

For example, to encode an h264 video, use the v4l2h264enc plugin:
```bash
gst-launch-1.0 videotestsrc ! v4l2h264enc ! h264parse ! qtmux ! filesink location=test.mp4 -e

# Press Ctrl+C to stop encoding
```