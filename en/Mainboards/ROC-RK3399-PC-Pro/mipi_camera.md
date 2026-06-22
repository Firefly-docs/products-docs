## MIPI camera

The RK3399 board with `MIPI CSI` interface has added the support of dual MIPI camera `OV13850`, and the camera example is also added in the application. The following describes the relevant configuration.

### Device tree file configuration

Take `rk3399-firefly-aiojd4.dts` as an example, these are the configurations that the camera needs to open. Since the kernel has added support for dual MIPI cameras, there are two cameras configured here. If you only use one camera, you only need to open one of the cameras.

```
//Power Management
&vcc_mipi {
        status = "okay";
};

&dvdd_1v2 {
        status = "okay";
};

//MIPI camera 1
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
//MIPI camera 2
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

**Note:** If you are using the `core-3399` core board plus a DIY backplane, you may need to modify the corresponding GPIO attributes.

### Debug

Before running the script, confirm whether the `OV13850` device is successfully registered. The following is the successful kernel log:

```
root@firefly:~# dmesg | grep ov13850
//MIPI camera 1
[1.276762] ov13850 1-0036: GPIO lookup for consumer reset
[1.276771] ov13850 1-0036: using device tree for GPIO lookup
[1.276803] of_get_named_gpiod_flags: parsed'reset-gpios' property of node'/i2c@ff110000/ov13850@36[0]'-status (0)
[1.276855] ov13850 1-0036: Looking up avdd-supply from device tree
[1.277034] ov13850 1-0036: Looking up dovdd-supply from device tree
[1.277170] ov13850 1-0036: Looking up dvdd-supply from device tree
[1.277535] ov13850 1-0036: GPIO lookup for consumer pwdn
[1.277544] ov13850 1-0036: using device tree for GPIO lookup
[1.277575] of_get_named_gpiod_flags: parsed'pwdn-gpios' property of node'/i2c@ff110000/ov13850@36[0]'-status (0)
[1.281862] ov13850 1-0036: Detected OV00d850 sensor, REVISION 0xb2
//MIPI camera 2
[1.284442] ov13850 1-0046: GPIO lookup for consumer pwdn
[1.284461] ov13850 1-0046: using device tree for GPIO lookup
[1.284523] of_get_named_gpiod_flags: parsed'pwdn-gpios' property of node'/i2c@ff110000/ov13850@46[0]'-status (0)
[1.288235] ov13850 1-0046: Detected OV00d850 sensor, REVISION 0xb2
```

Related device files should be generated under `/dev`:

```
root@firefly:~# ls /dev/video
//MIPI camera 1
video0 video1 video2 video3
//MIPI camera 2
video4 video5 video6 video7

```
**Note:** Similarly, if you only use one camera, you only need to register one camera device.

### Test the preview camera

#### v4l2
Use the v4l2 method to preview the camera:
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
Use the gstreamer method to preview the camera:
```
gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,format=NV12,width=640,height=480, framerate=30/1 ! videoconvert ! kmssink &
```

Just run the script, and the result is as shown in the figure:

![](../../../rk3399_img/mipi_csi.jpg)



#### FFMPEG read MIPI camera

* Multiplexing using v4l2loopback and FFMPEG
```
apt-get install -y git libopencv-dev cmake libdrm-dev g++ librga-dev python-dev

git clone https://github.com/umlaeute/v4l2loopback.git

cd v4l2loopback/

#Refer to "Installing linux-headers and linux-image"
make install
...

//Create video10, video11 and video12 devices using v4l2loopback
sudo insmod v4l2loopback.ko videonr=10,11,12

#Open and read the video0 device, and write the image information into video10
sudo -u firefly DISPLAY=:0  ./rkisp_demo -c 300  -d /dev/video0 -w 640  -h 480 -D /dev/video10

// Use the ffmpeg copy function to copy the content in video10 to video11 and video12 devices
ffmpeg -loglevel quiet -f v4l2 -video_size 640x480 -r 10 -i /dev/video10 -preset fast -codec copy -f v4l2 /dev/video11 -preset fast -codec copy -f v4l2 /dev/video12

// Start another terminal and use the ffplay tool to preview video11 or video12 at will
ffplay -i /dev/video11
```


#### OpenCV reading MIPI camera

The MIPI camera is **not supported** OpenCV **capture.open(index)** The way to read, here are the methods to read the camera in 2:

In this way, **it is recommended to use the C++ version of OpenCV**. For details, please refer to the previous v4l2 preview section. v4l2_simple_demo is a simplified version of rkisp_demo.cpp, please refer to rkisp_demo.cpp for details.

* C++

    **Recommended to use the C++ version of OpenCV**, please refer to the previous v4l2 preview section for details. v4l2_simple_demo is a simplified version of rkisp_demo.cpp, please refer to rkisp_demo.cpp for details.
    ```
    Refer：
    https://github.com/T-Firefly/rkisp-v4l2.git

    git clone https://github.com/T-Firefly/rkisp-v4l2.git
    cd rkisp-v4l2/mipi_video_demo/v4l2_simple_demo
    make
    # open /dev/video0
    sudo -u firefly DISPLAY=:0 ./opencv
    ```

* Python

    Using OpenCV-Python to read MIPI cameras needs to add Gstreamer support, and needs to reproduce the compilation and installation of OpenCV. Refer to ["OpenCV Compilation and Installation"](OpenCV_support.md)
    ```
    git clone https://gitlab.com/firefly-linux/test_code/rkisp-v4l2.git
    cd mipi_video_demo/OpenCV_Python
    # Please read rkisp-v4l2/mipi_video_demo/OpenCV_Python/README.md carefully.
    python3 opencv_gst_test.py
    ```