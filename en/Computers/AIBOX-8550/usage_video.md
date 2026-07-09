# Video Usage

AIBOX-8550 default video framework is Gstreamer.

## Available Gst Plugins

Use gst-inspect to list available media plugins, these qti plugins have hardware acceleration.

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

## VPU Capability

Video decode up to 4K240/8K60

Video encode up to 4K120/8K30

Concurrent 4K60 decode and 4K60 encode for wireless display

AV1 decode

Native decode support for H.265 Main 10, H.265 Main, H.264 High, and VP9 profile 2

Native encode support for H.265 Main 10, H.265 Main, H.264 high formats

## Video Play

Use qtic2vdec plugin, for example, play h264 video:
```bash
export XDG_RUNTIME_DIR=/run/user/root
export WAYLAND_DISPLAY=wayland-1
gst-launch-1.0 filesrc location=/usr/local/test.mp4 ! qtdemux name=demux demux.video_0 ! queue ! h264parse ! qtic2vdec ! videoconvert ! waylandsink sync=true
```

## Video Encode

Use qtic2venc plugin, for example, encode h264 stream:
```bash
gst-launch-1.0 videotestsrc ! qtic2venc ! h264parse ! qtmux ! filesink location=test.mp4 -e

# Use Ctrl+C to stop encoding.
```