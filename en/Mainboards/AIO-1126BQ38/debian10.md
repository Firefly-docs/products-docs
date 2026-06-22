# Firefly Debian Manual

## Firmware Download
Download the `Debian10` firmware from the [Resource Download](https://www.t-firefly.com/doc/download/101.html) page.

## System Introduction

Debian10 system built-in resources:
1. The system does not have built-in rockchip supporting IPC applications. There is no built-in ArcSoft Facial_Gate and UVC demo sample applications. If you need to use the above supporting resources, please download the corresponding buildroot firmware on the [Resource Download](https://en.t-firefly.com/doc/download/377.html) page.
2. The system has built-in rockchip npu library files and ArcSoft face recognition algorithm library files. And the supporting rkmedia development example demo is built in the system `/home/firefly/` directory. Help users quickly get started with developing product applications.

### Packages

The following software packages have been `default installed` to the Debian system. When using `apt install` to install the software, be careful not to `overwrite` the system software packages.
```
Gstreamer
libv4l
```

### Video hardware codec support
The `VPU` integrated in the RV1126 has excellent video codec capabilities. `MPP` is a set of video codec APIs provided by Rockchip for the `VPU`, and is based on `mpp`. Rockchip provides a set of codec plugins for `gstreamer`. Users can implement video codec applications based on `gstreamer` according to their own needs, or directly call `mpp` to achieve hardware codec acceleration.

Through the following two methods, verify and develop video codec-related applications.

1. Gstreamer
* By default `gstreamer1.0` is installed
```
# Test hardware H264 decoding.
gst-launch-1.0 filesrc location="/home/firefly/test.mp4" ! qtdemux ! h264parse ! mppvideodec ! kmssink

# Test hardware H264 encoding.
gst-launch-1.0 filesrc location=/home/firefly/video-640x360.yuv ! rawvideoparse use-sink-caps=false format=GST_VIDEO_FORMAT_I420 width=640 height=360 ! video/x-raw,width=640,height=360 ! mpph264enc ! queue ! h264parse ! qtmux ! filesink location=/home/firefly/yuv2h264.mp4
```
Users can refer to these two scripts to configure their own `gstreamer` applications.


2. MPP
* Under Debian system, mpp related deb packages have been installed in the system.

For more information, please refer to the relevant documents under `linux-sdk/docs/Linux/Multimedia`

### Source code compilation

#### RKNN_SSD_DEMO
* NPU is already installed in Debian system. Test the NPU as follows:
The test demo is `rknn_ssd_demo`. The source path is `/home/firefly/rknn_ssd_demo`. The following operations are performed on the `RV1126` side:
```
# Enter the rknn_ssd_demo directory
cd /home/firefly/rknn_ssd_demo

# compile
cmake ./
make

# execute program
sudo ./rknn_ssd_demo model/ssd_inception_v2_rv1109_rv1126.rknn model/road.bmp

# The effect is as follows:
root@firefly:/home/firefly/rknn_ssd_demo# sudo ./rknn_ssd_demo model/ssd_inception_v2_rv1109_rv1126.rknn model/road.bmp 
Loading model ...
model input num: 1, output num: 2
input tensors:
index=0 name= n_dims=4 dims=[1 300 300 3] n_elems=270000 size=270000 fmt=0 type=3 qnt_type=2 fl=127 zp=127 scale=0.007843
output tensors:
index=0 name= n_dims=4 dims=[1 1917 1 4] n_elems=7668 size=7668 fmt=0 type=3 qnt_type=2 fl=-76 zp=180 scale=0.089482
index=1 name= n_dims=3 dims=[0 1 1917 91] n_elems=174447 size=174447 fmt=0 type=3 qnt_type=2 fl=-66 zp=190 scale=0.137463
rknn_run
loadLabelName
ssd - loadLabelName ./model/coco_labels_list.txt
loadBoxPriors
person @ (13 125 58 212) 0.984076
bicycle @ (171 165 278 234) 0.972723
person @ (110 119 152 197) 0.968828
person @ (206 113 256 216) 0.964399
car @ (146 133 217 170) 0.959365
person @ (83 134 92 158) 0.634101
person @ (49 133 58 156) 0.601661
person @ (96 134 105 162) 0.465688
```

#### RKMedia
* RKMedia has been adapted and built into Debian system. The `librtsp` library is not supported by default.
* The following `4` demos are not supported by default, so comment the compilation configuration of the following `4` demos in `rkmedia/example/CMakeLists.txt`
```
rkmedia_vi_venc_rtsp_test
rkmedia_rga_crop_venc_test
rkmedia_vi_rockx_venc_rtsp_test
rkmedia_vi_rknn_venc_rtsp_test
```

* The source path is `/home/firefly/rkmedia`. The following operations are performed on the `RV1126` side:
```
# enter the rkmedia directory
cd /home/firefly/rkmedia

# cmake configuration
cmake ./   -DCMAKE_COLOR_MAKEFILE=OFF -DBUILD_DOC=OFF -DBUILD_DOCS=OFF -DBUILD_EXAMPLE=OFF -DBUILD_EXAMPLES=OFF -DBUILD_TEST=OFF -DBUILD_TESTS=OFF -DBUILD_TESTING=OFF -DBUILD_SHARED_LIBS=ON  -DWARNINGS_AS_ERRORS=ON -DUSE_RKAIQ=ON  -DRKMPP=ON -DRKMPP_LIB_NAME=rockchip_mpp -DRKMPP_ENCODER=ON -DRKMPP_ENCODER_OSD=ON -DRKMPP_DECODER=ON   -DAUDIO_ENCODER=ON -DAUDIO_DECODER=ON -DALSA_PLAYBACK=ON -DALSA_CAPTURE=ON -DAUDIO_ALGORITHM=ON -DV4L2_CAPTURE=ON  -DRKRGA=ON -DRKGUARD=ON -DRKNN=ON  -DROCKFACE=OFF -DFACE_RECOGNIZE=OFF -DROCKX=ON   -DDRM_DISPLAY=ON -DLIVE555=ON   -DLIVE555_SERVER=ON -DLIVE555_SERVER_H264=ON -DLIVE555_SERVER_H265=ON -DMOVE_DETECTION=ON -DOCCLUSION_DETECTION=ON  -DCOMPILES_EXAMPLES=ON -DCONFIG_OEM=1 -DCOMPILES_MINIMEDIA=ON -DRKAUDIO=ON


# compile
make -j4

# Install
make install
```
#### FIREFLY_FFMPEG_RTSP_DEMO
* Since Debian10 does not support `librtsp.a` library. Here, ffmpeg streaming is used instead of rkmedia streaming interface for data streaming. The source directory is in `/home/firefly/firefly_ffmpeg_rtsp_demo`.
```
# Switch to the firefly_ffmpeg_rtsp_demo directory
cd /home/firefly/firefly_ffmpeg_rtsp_demo

# compile
make

# Running the server in the background
/usr/share/rtsp_server/armhf/EasyDarwin-linux-8.1.0-21102107/easydarwin &

# run
./firefly_ffmpeg_rtsp_demo_test rtsp://xxx.xxx.xxx.xxx # Please replace your camera url address, Default H264 1080p 25fps
```
* PC preview streaming screen
```
vlc rtsp://<1126 ip address>:8554
```
* PC browser enters the server interface
```
http://<1126 ip address>:10008
```

#### FIREFLY_RKMEDIA_DEMO
* firefly_rkmedia_demo has been adapted and built into Debian system. The `librtsp` library is not supported by default.
Note: The Debian system does not support the rtsp streaming method of the following demos. For development needs, please modify the rtsp streaming method according to the source code `/home/firefly/firefly_ffmpeg_rtsp_demo/firefly_ffmpeg_rtsp_demo_test.cc`.
```
rkmedia_rtspget_vdec_venc_rtsp_test
rkmedia_vdec_venc_rtsp_test
rkmedia_vi_rknn_venc_rtsp_test
rkmedia_vi_venc_rtsp_test
```

* The source path is `/home/firefly/firefly_rkmedia_demo`. The following operations are performed on the `RV1126` side:
```
# Enter the firefly_rkmedia_demo directory
cd /home/firefly/firefly_rkmedia_demo

# cmake configuration
cmake ./ -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON  -DUSE_RKAIQ=ON -DDRM_DISPLAY=ON -DRKRGA=ON -DRKMPP=ON -DRKMPP_LIB_NAME=rockchip_mpp -DV4L2_CAPTURE=ON

# compile
make -j4

# Install
make install
```

## Use of ArcSoft face algorithm
If you want to use the ArcSoft face recognition algorithm, you need to specify the ArcSoft algorithm library and librknn_api.so library file to compile. Please refer to the following `/home/firefly/firefly_rkmedia_demo/CMakeLists.txt` to configure the ArcSoft algorithm link library.
```
set(ARCSOFT_LIBS
    /usr/lib/arm-linux-gnueabihf/arcsoft/libarcsoft_face.so
    /usr/lib/arm-linux-gnueabihf/arcsoft/libarcsoft_face_engine.so
    /usr/lib/arm-linux-gnueabihf/arcsoft/libgomp.so.1
    /usr/lib/arm-linux-gnueabihf/arcsoft/librknn_api.so
)
```

## Partition introduction

If you customize the Debian10 system yourself, you can upgrade after reading this chapter and modifying the partition.

Modify the partition table `parameter.txt` under the firmware directory, and modify the `rootfs` partition size according to the requirements. The default size is 3G. PS: Because there is no running application under Debian10, the `media` partition is not needed, so I deleted it.

```
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00010000@0x00008000(boot),0x00010000@0x00018000(recovery),0x00010000@0x00028000(backup),0x00600000@0x00038000(rootfs),0x00060000@0x00638000(oem),-@0x00698000(userdata:grow)
```

Partition table format: partition size@location address (partition name). The `-` at the last -@location address (partition name: grow) means to expand the remaining memory size of the partition, which can only be placed at the end of `CMDLINE`.
The size and address are both in units of block `(1 block = 512byte)`, and the following formula can be used to convert to MByte:

```
MByte = N * 512 / 1024 / 1024
```

For example, let's look at the `rootfs` line:
The partition size is 0x00600000, converted to decimal is

```
6291456 blocks * 512 bytes per block / 1024 / 1024 = 3072 MByte
```

So the rootfs partition size is 3072M.

* Special attention: `Partition size + address = address of the next partition`.

## System export

* Use the system's built-in `ff_export_rootfs` tool to export the system that has been customized on the board. Please use ext4 format for export. How to use it:
```shell
root@firefly:/# ff_export_rootfs

	ff_export_rootfs </path/to/store> [-t <ext4|btrfs>]
```
Note: If the internal memory of the system is not enough to export the system, there are two solutions.
* If the device supports access to external storage devices such as USB flash drives or SD cards, set the export path to the directory mounted on the corresponding external storage device.
* If the device does not support the connection of external storage devices such as USB flash drives or SD cards, but it supports the connection of network cables. Follow these steps to export:
1. Prepare a Linux PC device, and ensure that the PC and RV1126 device are on the same network segment. Make sure the PC and RV1126 can ping each other.
2. Use the `sshfs` tool to mount the storage of the external Linux PC to the RV1126 device directory. xxx represents the user name of the Linux PC, and XXX.XXX.XXX.XXX represents the IP address of the Linux PC. /save-img-path is the path to the file system where PPC stores exports. /pc-mount-path is the directory actually mounted on RV1126.
```shell
sshfs xxx@XXX.XXX.XXX /pc-mount-path/
ff_export_rootfs /pc-mount-path -t ext4
```

## Restore to Factory Settings

* Use recovery tool to restore factory settings. Please back up important data before restoring to factory settings. The command is as follows:
```
recovery reset
```