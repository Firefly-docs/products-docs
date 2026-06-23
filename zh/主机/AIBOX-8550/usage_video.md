# 视频教程

AIBOX-8550 默认的视频框架为 Gstreamer.

## 可用 gst 组件

使用 gst-inspect 可以查看可用的多媒体组件，这些 qti 组件享受硬件加速功能。

```bash
root@kalama:~# gst-inspect-1.0 --plugin | grep "qti"
qtibatch:  qtibatch: Batching stream buffers
qtic2adec:  qtic2adec: Codec2 AAC Audio Decoder
qtic2aenc:  qtic2aenc: Codec2 AAC Audio Encoder
qtic2vdec:  qtic2vdec: Codec2 H.264/H.265/VP8/VP9/MPEG Video Decoder
qtic2venc:  qtic2venc: Codec2 H.264/H.265/HEIC Video Encoder
qticamreproc:  qticamreproc: Camera Reprocess
qticvimgpyramid:  qticvimgpyramid: CV Image Pyramid Scaler
qticvoptclflow:  qticvoptclflow: CV Optical Flow
qtijpegenc:  qtijpegenc: Jpeg encoder
qtimetamux:  qtimetamux: Meta muxer
qtimldemux:  qtimldemux: Batching stream buffers
qtimlvclassification:  qtimlvclassification: Machine Learning image classification
qtimlvconverter:  qtimlvconverter: Machine Learning Video Converter
qtimlvdetection:  qtimlvdetection: Machine Learning image object detection
qtimlvpose:  qtimlvpose: Machine Learning Pose
qtimlvsegmentation:  qtimlvsegmentation: Machine Learning image segmentation
qtimlvsuperresolution:  qtimlvsuperresolution: Machine Learning image super resolution
qtiobjtracker:  qtiobjtracker: Object Tracker
qtioverlay:  qtioverlay: QTI Overlay
qtiqmmfsrc:  qtiqmmfsrc: QMMF Video Source
qtisocketsink:  qtisocketsink: QTI Socket Sink Element
qtisocketsrc:  qtisocketsrc: QTI Socket Source Element
qtivcomposer:  qtivcomposer: Video composer
qtivoverlay:  qtivoverlay: Video Overlay
qtivsplit:  qtivsplit: Video stream splitter
qtivtransform:  qtivtransform: Video transformer
```

## 编解码能力

Video decode up to 4K240/8K60

Video encode up to 4K120/8K30

Concurrent 4K60 decode and 4K60 encode for wireless display

AV1 decode

Native decode support for H.265 Main 10, H.265 Main, H.264 High, and VP9 profile 2

Native encode support for H.265 Main 10, H.265 Main, H.264 high formats

## 视频播放

使用 qtic2vdec 组件，例如播放 h264 视频：
```bash
export XDG_RUNTIME_DIR=/run/user/root
export WAYLAND_DISPLAY=wayland-1
gst-launch-1.0 filesrc location=/usr/local/test.mp4 ! qtdemux name=demux demux.video_0 ! queue ! h264parse ! qtic2vdec ! videoconvert ! waylandsink sync=true
```

## 视频编码

使用 qtic2venc 组件，例如编码 h264 视频：
```bash
gst-launch-1.0 videotestsrc ! qtic2venc ! h264parse ! qtmux ! filesink location=test.mp4 -e

# 按 Ctrl+C 停止编码
```