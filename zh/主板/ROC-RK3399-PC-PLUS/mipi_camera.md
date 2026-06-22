## MIPI 摄像头
带有 `MIPI CSI` 接口的 RK3399 板子都添加了双 MIPI 摄像头 `OV13850` 的支持，应用中也添加了摄像头的例子。下面介绍一下相关配置。

### 设备树文件配置

以 `rk3399-firefly-aiojd4.dts` 为例，这些都是摄像头需要打开的配置。由于内核已经添加了双 MIPI 摄像头的支持，所以这里配置了两个摄像头，如果只使用一个摄像只需要打开其中一个摄像头就可以了。

```
//电源管理
&vcc_mipi {
        status = "okay";
};

&dvdd_1v2 {
        status = "okay";
};

//MIPI 摄像头1
&ov13850 {
        status = "okay";
};
&rkisp1_0 {
        status = "okay";
};

&mipi_dphy_rx0 {
        status = "okay";
};

&isp0_mmu {
        status = "okay";
};
//MIPI 摄像头2
&ov13850_1 {
        status = "okay";
};
&rkisp1_1 {
        status = "okay";
};
&mipi_dphy_tx1rx1 {
        status = "okay";
};

&isp1_mmu {
        status = "okay";
};
```


**注意：** 如果使用的是 `core-3399` 核心板加上 DIY 底板，可能需要修改相应的 GPIO 属性。

### 调试

在运行脚本之前先确认一下 `OV13850` 设备是否注册成功，下面是成功的内核日志：

```
root@firefly:~# dmesg | grep ov13850
//MIPI 摄像头1
[    1.276762] ov13850 1-0036: GPIO lookup for consumer reset
[    1.276771] ov13850 1-0036: using device tree for GPIO lookup
[    1.276803] of_get_named_gpiod_flags: parsed 'reset-gpios' property of node '/i2c@ff110000/ov13850@36[0]' - status (0)
[    1.276855] ov13850 1-0036: Looking up avdd-supply from device tree
[    1.277034] ov13850 1-0036: Looking up dovdd-supply from device tree
[    1.277170] ov13850 1-0036: Looking up dvdd-supply from device tree
[    1.277535] ov13850 1-0036: GPIO lookup for consumer pwdn
[    1.277544] ov13850 1-0036: using device tree for GPIO lookup
[    1.277575] of_get_named_gpiod_flags: parsed 'pwdn-gpios' property of node '/i2c@ff110000/ov13850@36[0]' - status (0)
[    1.281862] ov13850 1-0036: Detected OV00d850 sensor, REVISION 0xb2
//MIPI 摄像头2
[    1.284442] ov13850 1-0046: GPIO lookup for consumer pwdn
[    1.284461] ov13850 1-0046: using device tree for GPIO lookup
[    1.284523] of_get_named_gpiod_flags: parsed 'pwdn-gpios' property of node '/i2c@ff110000/ov13850@46[0]' - status (0)
[    1.288235] ov13850 1-0046: Detected OV00d850 sensor, REVISION 0xb2
```

在 `/dev` 下应该生成相关设备文件：

```
root@firefly:~# ls /dev/video
//MIPI 摄像头1
video0  video1  video2  video3
//MIPI 摄像头2
video4  video5  video6  video7

```
**注意：** 同理，如果只使用一个摄像头，只需要注册一个摄像头设备就可以了。

### 测试预览摄像头

#### v4l2
使用v4l2方式预览摄像头：
```
apt-get install -y git libopencv-dev cmake libdrm-dev g++ librga-dev

git clone https://github.com/T-Firefly/rkisp-v4l2.git

cd rkisp-v4l2/rkisp_demo/
cmake ./
make

sudo -u firefly DISPLAY=:0 ./rkisp_demo -c 300 -d /dev/video0 -w 640 -h 480
or
sudo -u firefly DISPLAY=:0 ./rkisp_demo -c 300 -d /dev/video5 -w 640 -h 480
```


#### gstreamer
使用gstreamer方式预览摄像头：
```
gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,format=NV12,width=640,height=480, framerate=30/1 ! videoconvert ! kmssink &
```



运行脚本即可，结果如图所示：

![](../../../rk3399_img/mipi_csi.jpg)

### 开发MIPI 摄像头


#### FFMPEG读取MIPI摄像头

* 使用v4l2loopback和FFMPEG多路投影

```
apt-get install -y git libopencv-dev cmake libdrm-dev g++ librga-dev python-dev

git clone https://github.com/umlaeute/v4l2loopback.git

cd v4l2loopback/

#参考 《安装linux-headers和linux-image》
make install
...

//使用v4l2loopback创建 video10、video11和video12设备
sudo insmod v4l2loopback.ko videonr=10,11,12

#打开读取video0设备，并把图像信息写入到video10中
sudo -u firefly DISPLAY=:0  ./rkisp_demo -c 300  -d /dev/video0 -w 640  -h 480 -D /dev/video10

// 使用ffmpeg拷贝功能将video10中的内容，拷贝到video11和video12设备中
ffmpeg -loglevel quiet -f v4l2 -video_size 640x480 -r 10 -i /dev/video10 -preset fast -codec copy -f v4l2 /dev/video11 -preset fast -codec copy -f v4l2 /dev/video12

// 另起一个终端，使用ffplay工具任意预览video11或video12
ffplay -i /dev/video11
```


#### OpenCV读取MIPI摄像头

MIPI摄像头是**不支持**OpenCV中 **capture.open(index)** 方式读取的，在此提供2中读取摄像头的方法：

这种方式**推荐OpenCV的C++版本使用**，详细使用参考前面的v4l2预览一节。v4l2_simple_demo是rkisp_demo.cpp的精简版本，详细可参考rkisp_demo.cpp。

* C++

    **推荐OpenCV的C++版本使用**，详细使用参考前面的v4l2预览一节。v4l2_simple_demo是rkisp_demo.cpp的精简版本，详细可参考rkisp_demo.cpp。
    ```
    参考：
    https://github.com/T-Firefly/rkisp-v4l2.git

    git clone https://github.com/T-Firefly/rkisp-v4l2.git
    cd rkisp-v4l2/mipi_video_demo/v4l2_simple_demo
    make
    # open /dev/video0
    sudo -u firefly DISPLAY=:0 ./opencv
    ```

* Python

    使用 OpenCV-Python 方式读取 MIPI 摄像头需要添加Gstreamer支持，并需要重现编译安装OpenCV。参考[《OpenCV 编译和安装》](OpenCV_support.md)
    ```
    git clone https://gitlab.com/firefly-linux/test_code/rkisp-v4l2.git
    cd mipi_video_demo/OpenCV_Python
    # Please read rkisp-v4l2/mipi_video_demo/OpenCV_Python/README.md carefully.
    python3 opencv_gst_test.py
    ```