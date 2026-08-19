# Firefly Debian 使用手册

## 固件下载
在[资源下载](https://www.t-firefly.com/doc/download/101.html)页面下载` Debian10 `固件。

## 系统介绍

Debian10 系统内置资源：
1. 系统没有内置 rockchip 配套的 IPC 应用。也没有内置虹软 ArcSoft Facial_Gate 和 UVC 的 demo 示例应用。如果需要使用上述配套资源，请在[资源下载](https://www.t-firefly.com/doc/download/101.html)页面下载对应的 buildroot 固件。
2. 系统内置了 rockchip npu 的库文件和虹软 ArcSoft 人脸识别算法的库文件。并且在系统 `/home/firefly/` 目录内置了配套的 rkmedia 开发实例 demo 。帮助用户快速上手开发产品应用。

### 软件包

以下软件包已`默认安装`到 Debian 系统，使用 `apt install` 安装软件的时候注意`不要覆盖`掉系统的软件包。
```
Gstreamer
libv4l
```

### 视频硬件编解码支持
RV1126 集成的 `VPU` 具有优秀的视频编解码能力， `MPP` 是 Rockchip 为 `VPU` 提供的一套视频编解码的 api , 并且基于 `mpp` 。 Rockchip 提供了一套 `gstreamer` 的编解码插件。用户可以根据自己的需求，基于 `gstreamer` 来做视频编解码的应用，或者直接调用 `mpp` ，来实现硬件的编解码加速。

通过以下两种方式，验证和开发视频编解码相关应用。

一、Gstreamer
* 默认安装 `gstreamer1.0`
```
#测试硬件 H264 解码。
gst-launch-1.0 filesrc location="/home/firefly/test.mp4" ! qtdemux ! h264parse ! mppvideodec ! kmssink

#测试硬件 H264 编码。
gst-launch-1.0 filesrc location=/home/firefly/video-640x360.yuv ! rawvideoparse use-sink-caps=false format=GST_VIDEO_FORMAT_I420 width=640 height=360 ! video/x-raw,width=640,height=360 ! mpph264enc ! queue ! h264parse ! qtmux ! filesink location=/home/firefly/yuv2h264.mp4
```
用户可以参照这两个脚本，配置自己的 `gstreamer` 应用。


二、MPP
* Debian 系统下， mpp 相关 deb 包都已经安装到系统中。

更多相关资料,可参考`linux-sdk/docs/Linux/Multimedia`下的相关文档

### 源码编译

#### RKNN_SSD_DEMO
* NPU 已经内置安装到 Debian 系统。测试 NPU 如下：
测试 demo 为 `rknn_ssd_demo` 。源码路径为 `/home/firefly/rknn_ssd_demo`。以下操作均在 `RV1126` 端执行：
```
# 进入到 rknn_ssd_demo 目录
cd /home/firefly/rknn_ssd_demo

#编译
cmake ./
make

#执行程序
sudo ./rknn_ssd_demo model/ssd_inception_v2_rv1109_rv1126.rknn model/road.bmp

#效果如下：
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
* RKMedia 已经适配并内置到 Debian 系统。默认不支持 `librtsp` 库。
* 默认不支持以下 `4` 个 demo，所以在 `rkmedia/example/CMakeLists.txt`注释以下 `4` 个 demo 的编译配置
```
rkmedia_vi_venc_rtsp_test
rkmedia_rga_crop_venc_test
rkmedia_vi_rockx_venc_rtsp_test
rkmedia_vi_rknn_venc_rtsp_test
```

* 源码路径为 `/home/firefly/rkmedia`。以下操作均在 `RV1126` 端执行：
```
# 进入到 rkmedia 目录
cd /home/firefly/rkmedia

# cmake 配置
cmake ./   -DCMAKE_COLOR_MAKEFILE=OFF -DBUILD_DOC=OFF -DBUILD_DOCS=OFF -DBUILD_EXAMPLE=OFF -DBUILD_EXAMPLES=OFF -DBUILD_TEST=OFF -DBUILD_TESTS=OFF -DBUILD_TESTING=OFF -DBUILD_SHARED_LIBS=ON  -DWARNINGS_AS_ERRORS=ON -DUSE_RKAIQ=ON  -DRKMPP=ON -DRKMPP_LIB_NAME=rockchip_mpp -DRKMPP_ENCODER=ON -DRKMPP_ENCODER_OSD=ON -DRKMPP_DECODER=ON   -DAUDIO_ENCODER=ON -DAUDIO_DECODER=ON -DALSA_PLAYBACK=ON -DALSA_CAPTURE=ON -DAUDIO_ALGORITHM=ON -DV4L2_CAPTURE=ON  -DRKRGA=ON -DRKGUARD=ON -DRKNN=ON  -DROCKFACE=OFF -DFACE_RECOGNIZE=OFF -DROCKX=ON   -DDRM_DISPLAY=ON -DLIVE555=ON   -DLIVE555_SERVER=ON -DLIVE555_SERVER_H264=ON -DLIVE555_SERVER_H265=ON -DMOVE_DETECTION=ON -DOCCLUSION_DETECTION=ON  -DCOMPILES_EXAMPLES=ON -DCONFIG_OEM=1 -DCOMPILES_MINIMEDIA=ON -DRKAUDIO=ON


# 编译
make -j4

# 安装
make install
```
#### FIREFLY_FFMPEG_RTSP_DEMO
* 由于 Debian10 不支持 `librtsp.a` 库。在此使用 ffmpeg 推流的方式替代 rkmedia 推流接口进行数据推流。源码目录在 `/home/firefly/firefly_ffmpeg_rtsp_demo`。具体使用请参照目录下的 `/home/firefly/firefly_ffmpeg_rtsp_demo/README.md` 文件
```
# 切换到 firefly_ffmpeg_rtsp_demo 目录
cd /home/firefly/firefly_ffmpeg_rtsp_demo

# 编译
make

# 后台运行服务器
/usr/share/rtsp_server/armhf/EasyDarwin-linux-8.1.0-21102107/easydarwin &

# 运行
./firefly_ffmpeg_rtsp_demo_test rtsp://xxx.xxx.xxx.xxx # Please replace your camera url address, Default H264 1080p 25fps
```
* PC 预览推流画面
```
vlc rtsp://<1126 ip address>:8554
```
* PC 浏览器进入服务器界面
```
http://<1126 ip address>:10008
```

#### FIREFLY_RKMEDIA_DEMO
* firefly_rkmedia_demo 已经适配并内置到 Debian 系统。默认不支持 `librtsp` 库。
注意：Debian 系统不支持以下 demo 的 rtsp 推流方式。如有开发需要，请根据源码 `/home/firefly/firefly_ffmpeg_rtsp_demo/firefly_ffmpeg_rtsp_demo_test.cc` 修改 rtsp 推流方式。
```
rkmedia_rtspget_vdec_venc_rtsp_test
rkmedia_vdec_venc_rtsp_test
rkmedia_vi_rknn_venc_rtsp_test
rkmedia_vi_venc_rtsp_test
```

* 源码路径为 `/home/firefly/firefly_rkmedia_demo`。具体使用请参照目录下的 `/home/firefly/firefly_rkmedia_demo/README.md` 文件。
* 以下操作均在 `RV1126` 端执行：
```
# 进入到 firefly_rkmedia_demo 目录
cd /home/firefly/firefly_rkmedia_demo

# cmake 配置
cmake ./ -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON  -DUSE_RKAIQ=ON -DDRM_DISPLAY=ON -DRKRGA=ON -DRKMPP=ON -DRKMPP_LIB_NAME=rockchip_mpp -DV4L2_CAPTURE=ON


# 编译
make -j4

# 安装
make install
```

## 虹软人脸算法使用
如需使用虹软人脸识别算法，则需要指定虹软算法库和 librknn_api.so 库文件编译。请参照以下 `/home/firefly/firefly_rkmedia_demo/CMakeLists.txt` 配置虹软算法链接库。
```
set(ARCSOFT_LIBS
    /usr/lib/arm-linux-gnueabihf/arcsoft/libarcsoft_face.so
    /usr/lib/arm-linux-gnueabihf/arcsoft/libarcsoft_face_engine.so
    /usr/lib/arm-linux-gnueabihf/arcsoft/libgomp.so.1
    /usr/lib/arm-linux-gnueabihf/arcsoft/librknn_api.so
)
```

## 分区介绍

如果自己定制 Debian10 系统可以阅读此章节修改分区后进行升级。

修改固件目录下的分区表` parameter.txt `，根据需求修改` rootfs `分区大小。默认为大小为 3G 。PS：因为 Debian10 下没有跑应用,不需要` media `分区所以删掉了。

```
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00010000@0x00008000(boot),0x00010000@0x00018000(recovery),0x00010000@0x00028000(backup),0x00600000@0x00038000(rootfs),0x00060000@0x00638000(oem),-@0x00698000(userdata:grow)
```

分区表格式：分区大小@所在地址(分区名)。最后的-@所在地址(分区名:grow)的 `-` 代表将剩余的内存大小拓展该分区，该写法只能放在 `CMDLINE` 最后。
其中大小和地址都是以块为单位`（ 1 块 = 512byte）`，换算成 MByte 可以用以下公式：

```
MByte = N * 512 / 1024 / 1024
```

例如我们看 `rootfs` 这行：
分区大小是 0x00600000 ，换算成十进制也就是

```
6291456  块 * 512 字节每块 / 1024 / 1024 = 3072 MByte
```

所以 rootfs 分区大小就是 3072M 。

* 特别注意：`分区大小 + 所在地址 = 下一个分区的所在地址`。

## 系统导出

* 使用系统内置的 `ff_export_rootfs` 工具对已经在板子上定制好的系统进行导出，请使用 ext4 格式进行导出。使用方法如下：
```shell
root@firefly:/# ff_export_rootfs

	ff_export_rootfs </path/to/store> [-t <ext4|btrfs>]
```
注：如果系统内部的内存不足以将系统导出，那么有两种解决办法。
* 如果设备支持接入 U 盘或者 SD 卡等外部存储设备，就将导出路径设置为对应外部存储设备挂载的目录。
* 如果设备不支持接入 U 盘或者 SD 卡等外部存储设备，但是支持连接网线。根据以下步骤进行导出：
1. 准备一台 Linux PC 设备，并且保证 PC 和 RV1126 设备在同一网段。保证 PC 和 RV1126 可以互相 ping 通。
2. 使用 `sshfs` 工具将外部 Linux PC 的存储挂载到 RV1126 设备目录上。xxx 代表着 Linux PC 的用户名称，XXX.XXX.XXX.XXX 代表 Linux PC 的 ip 地址。/save-img-path 是PPC 存储导出的文件系统的路径。/pc-mount-path 是在 RV1126 实际挂载的目录。
```shell
sshfs xxx@XXX.XXX.XXX:/save-img-path  /pc-mount-path/
ff_export_rootfs /pc-mount-path -t ext4
```

## 恢复出厂设置

* 使用 recovery 工具进行恢复出厂设置。恢复出厂设置之前请备份好重要的数据。命令如下：
```
recovery reset
```