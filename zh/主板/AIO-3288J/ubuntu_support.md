# Firefly Ubuntu Desktop 使用

Firefly-RK3288 Ubuntu Desktop 系统基于 Ubuntu 32bit 系统构建，目前主要维护 Ubuntu18.04 版本。具有以下特性：

1. 桌面环境采用 LXDE，Lubuntu+Firefly 定制主题，简洁美观。
2. Xserver 使用 GPU + RGA 进行 2D 加速，运行流畅，占用 CPU 资源少。
3. 针对嵌入式平台，精简系统服务。
4. 提供基于 Arm Mali GPU 的 OpenGL、OpenCL 支持。
5. 提供基于 Rockchip VPU + Mpp 的视频硬编解码支持。
6. 适配 QT、Docker、Electron 等开发框架。
7. 提供一系列接口，以操作板载资源设备。

## 基础信息

### 用户和密码

Firefly Linux Desktop 系统开机启动后，自动登录到 firefly 用户。

如果有连接调试串口，串口终端自动登录 root 用户。

- firefly 用户密码： firefly
- root 用户密码： firefly

## 视频硬件编解码支持

RK3288 集成的 VPU 具有优秀的视频编解码能力，Mpp 是 Rockchip 为 VPU 提供的一套视频编解码的 api，并且基于 mpp。Rockchip 提供了一套 gstreamer 的编解码插件。用户可以根据自己的需求，基于 gstreamer 来做视频编解码的应用，或者直接调用 mpp，来实现硬件的编解码加速。

系统提供了一个测试视频文件，位于 `/usr/local/test.mp4`，测试文件为 1080P, 24Fps, H264 编码，Mp4 格式。

可以通过一下几种方式，验证和开发视频编解码相关应用。

### Gstreamer

- Ubuntu 16.04 下，gstreamer 1.12 已经安装在 `/opt/` 目录下。
- Ubuntu 18.04 下， gstreamer 1.12 已经安装到系统中。

`/usr/local/bin/h264dec.sh` 测试硬件 H264 解码。

`/usr/local/bin/h264enc.sh` 测试硬件 H264 编码。

用户可以参照这两个脚本，配置自己的 gstreamer 应用。

### Mpv

系统提供的 Mpv 播放器，可以直接调用 rkmpp 解码插件。

### FFmpeg

FFmpeg 对于 Rockchip 暂时只支持通过 Mpp 实现硬件解码，暂时没有硬件编码的支持。

Firefly 发布的 Ubuntu 系统已经安装了 FFmpeg，用户可以直接使用。

- 确认 rkmpp 解码器

```bash
$ ffmpeg -decoders | grep "rkmpp"

 V..... h264_rkmpp           h264 (rkmpp) (codec h264)
 V..... hevc_rkmpp           hevc (rkmpp) (codec hevc)
 V..... vp8_rkmpp            vp8 (rkmpp) (codec vp8)
 V..... vp9_rkmpp            vp9 (rkmpp) (codec vp9)
```

- 测试命令

```bash
$ ffmpeg -y -c:v h264_rkmpp -i test.mp4 -an -vf \
        hwdownload,format=nv12,format=yuv420p output.yuv
```

**特别注意：** FFmpeg h264_rkmpp 解码获得的是 AV_PIX_FMT_DRM_PRIME，也就是 DRM 帧数据，如果是基于 drm 显示，可以直接输出帧，否则，需要用 hwdownload 进行转换。

更多内容可以参考 [FFmpeg 官网](http://ffmpeg.org/about.html)。

### Mpp

- Ubunut 系统下， mpp 相关 dev 包都已经安装到系统中。

	更多相关资料,可参考`linux-sdk/docs/Linux/Multimedia`下的相关文档。

## OpenGL-ES

RK3288 支持 OpenGL ES1.1/2.0/3.0/3.1。

Firefly 发布的Ubuntu 系统， 都已经提供了完整的 OpenGL-ES 支持。

- 测试命令

```bash
$ sudo test_glmark2_normal.sh
```

- webGL 支持

在 Chromium 浏览器中， 在地址栏输入：`chrome://gpu` 可以查看 chromium 下硬件加速的支持。

Note:

1. EGL 是用 arm 平台上 OpenGL 针对 x window system 的扩展，功能等效于 x86 下的 glx 库。

2. 由于 Xorg 使用的 Driver modesettings 默认会加载 libglx.so(禁用 glx 会导致某些通过检测 glx 环境的应用启动失败)， libglx.so 会搜索系统中的 dri 实现库。但是 rk3288 Xorg 2D 加速是直接基于 DRM 实现，并未实现 dri 库，所以启动过程中，libglx.so 会报告如下的错误。

    ```bash
    (EE) AIGLX error: dlopen of /usr/lib/arm-linux-gnueabihf/dri/rockchip_dri.so failed
    ```

    这个对系统运行没有任何影响，不需要处理。

3. 基于同样的道理，某些应用启动过程中，也会报告如下错误，不用处理，对应用的运行不会造成影响。

    ```bash
    libGL error: unable to load driver: rockchip_dri.so
    libGL error: driver pointer missing
    libGL error: failed to load driver: rockchip
    ```

4. Firefly 之前发布的某些版本的 Ubuntu 软件，默认关闭了加载 libglx.so，在某些情况下，运行某些应用程序会出现下述错误：

    `GdkGLExt-WARNING **: Window system doesn't support OpenGL.`

    修正的方法如下：

    删除  `/etc/X11/xorg.conf.d/20-modesetting.conf` 中一下三行配置。

    ```bash
    Section "Module"
         Disable     "glx"
    EndSection
    ```

## OpenCL

Firefly 发布的 Ubuntu 系统，已经添加了 opencl1.2 支持，可以运行系统内置的 `clinfo` 获取平台opencl 相关参数。

```bash
firefly@firefly:~$ clinfo
Number of platforms                               1
  Platform Name                                   ARM Platform
  Platform Vendor                                 ARM
  Platform Version                                OpenCL 1.2 v1.r14p0-01rel0-git(a79caef).004e51a1e8b44487ebc4c7bfea5f60ee
  Platform Profile                                FULL_PROFILE
  Platform Extensions                             cl_khr_global_int32_base_atomics cl_khr_global_int32_extended_atomics cl_khr_local_int32_base_atomics cl_khr_local_int32_extended_atoy
  Platform Extensions function suffix             ARM

  Platform Name                                   ARM Platform
Number of devices                                 1
  Device Name                                     Mali-T760
  Device Vendor                                   ARM
  Device Vendor ID                                0x7501000
  Device Version                                  OpenCL 1.2 v1.r14p0-01rel0-git(a79caef).004e51a1e8b44487ebc4c7bfea5f60ee
  Driver Version                                  1.2
  Device OpenCL C Version                         OpenCL C 1.2 v1.r14p0-01rel0-git(a79caef).004e51a1e8b44487ebc4c7bfea5f60ee
  Device Type                                     GPU
…
```

## 屏幕旋转

Firefly 发布的 Ubuntu 系统，如果需要默认对系统的显示方向做旋转，可以在

`/etc/default/xrandr` 中修改对应的显示设备的方向即可。

```bash
firefly@firefly:~$ cat /etc/default/xrandr
#!/bin/sh

# Rotation can be one of 'normal', 'left', 'right' or 'inverted'.

# xrandr --output HDMI-1 --rotate normal
# xrandr --output LVDS-1 --rotate normal
# xrandr --output EDP-1 --rotate normal
# xrandr --output MIPI-1 --rotate normal
# xrandr --output VGA-1 --rotate normal
# xrandr --output DP-1 --rotate normal
```

对于配有触摸屏的平台，如果需要对触摸屏的方向做旋转，可以在 `/etc/X11/xorg.conf.d/05-gslX680.conf` 中修改 SwapAxes / InvertX / InvertY 三个值。

```bash
firefly@firefly:~$ cat /etc/X11/xorg.conf.d/05-gslX680.conf
Section "InputClass"
        Identifier "gslX680"
        MatchIsTouchscreen "on"
        MatchProduct  "gslX680"
        Driver "evdev"
        Option "SwapAxes" "off"
       # Invert the respective axis.
        Option "InvertX" "off"
        Option "InvertY" "off"
EndSection
```

## 屏幕键盘

官方的 Ubuntu 系统中自带屏幕键盘，可以在菜单栏中点击打开：
![](../../../rk3288_img/onboard.png)

## 声音配置

开发板一般都有2个或以上的音频设备。常见的是耳机和 HDMI 这两个设备的音频输出。下面是音频设置的例子，供用户参考。

### 从命令行指定音频设备

系统自带的音频文件在 `/usr/share/sound/alsa/` 目录中，播放前请先查看声卡设备：

```bash
root@firefly:~# aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: rockchipfirefly [rockchip,firefly-codec], device 0: ff890000.i2s-ES8323 HiFi ES8323 HiFi-0 []
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 0: rockchipfirefly [rockchip,firefly-codec], device 1: ff890000.i2s-i2s-hifi i2s-hifi-1 []
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

然后指定声卡播放音频，其中声卡 card0 的设备0 对应的是耳机口，card0 的设备1 对应的是 HDMI。一般播放音频命令是 `aplay -Dhw:0,0 Fornt_Center.wav`，但系统自带的音频文件是单声道的，所以为了防止播放失败，可以按照下面的命令进行播放：

```bash
#选择耳机口输出音频文件
root@firefly:/usr/share/sounds/alsa# aplay -Dplughw:0,0 Front_Center.wav
Playing WAVE 'Front_Center.wav' : Signed 16 bit Little Endian, Rate 48000 Hz, Mono

#选择 HDMI 输出音频文件
root@firefly:/usr/share/sounds/alsa# aplay -Dplughw:0,1 Front_Center.wav
Playing WAVE 'Front_Center.wav' : Signed 16 bit Little Endian, Rate 48000 Hz, Mono
```

## 开机启动应用程序

在系统中，可以根据用户需求自行设置应用程序开机自动启动。

在系统菜单栏中依次选择：`Preferences` -> `Default applications for LXSession`，打开设置界面。

在设置界面的 `Launching applocations` 一栏中可以设置应用默认的打开方式：

![](../../../rk3288_img/autostart_setting.png)

在 `Autostart` 一栏中就可以选择用户需要开机启动的应用。例如蓝牙默认是开机不启动的，如果需要让它开机自动启动，就把 `Blueman Applet` 的勾打上，就可以实现开机自动启动蓝牙：

![](../../../rk3288_img/autostart_bluetooth.png)

## 串口

开发板上的串口分两种，一种是普通串口另外一种是通过转换芯片把 SPI 转换而成的串口。转换而成的串口和普通串口功能完全一致但是需要注意它们的设备文件名是不一样的。下面是两者的区别:

```bash
# 普通串口
root@firefly:~# ls /dev/ttyS*
ttyS0  ttyS1  ttyS2  ttyS3

# SPI 转串口
root@firefly:~# ls /dev/ttysWK*
ttysWK0  ttysWK1  ttysWK2  ttysWK3
```

### 设置波特率

以 `ttyS4` 为例，查看串口波特率命令：

```bash
root@firefly:~# stty -F /dev/ttyS4
speed 9600 baud; line = 0;
-brkint -imaxbel
```

设置波特率命令：

```bash
# 设置波特率为 115200
root@firefly:~# stty -F /dev/ttyS4 ospeed 115200 ispeed 115200 cs8

# 查看确认是否已经修改
root@firefly:~# stty -F /dev/ttyS4
speed 115200 baud; line = 0;
-brkint -imaxbel
root@firefly:~#
```

### 关闭回显

在做环回收发测试的时候回显功能会影响到我们的测试结果，所以在环回测试或者其他特殊情况下需要关闭回显。以下是关闭回显的命令：

```bash
# 关闭回显示
root@firefly:~# stty -F /dev/ttyS4 -echo -echoe -echok

# 查看所有功能的配置，检查是否已经关闭。“-” 号代表该功能已经关闭
root@firefly:~# stty -F /dev/ttyS4 -a | grep echo
isig icanon iexten -echo -echoe -echok -echonl -noflsh -xcase -tostop -echoprt
echoctl echoke -flusho -extproc
```

### 发送接收原始数据

在实际应用中实际发送数据和我们希望发送的数据可能存在差别，例如多了回车或者其他特殊字符。利用 `stty` 可以把发送接收设置成 `raw` 模式确保发送接收的是原始数据，以下是设置命令：

```bash
root@firefly:~# stty -F /dev/ttyS4 raw
```

### 工作模式

串口有 2 种工作模式分别是中断模式和 DMA 模式。

#### 中断模式

内核默认把串口配置成中断模式所以不需要做任何修改。中断模式下传输速率比较快但是在传大量数据的时候很容易丢包或者出错，所以当数据量比较大的时候请不要使用中断模式。

#### DMA 模式

DMA 模式主要是传输大量数据的时候使用。内核会为串口提供一个缓存空间接收数据来尽量降低串口传输的丢包率。

**注意：** 缓存空间限默认大小为 `8K` 如果一次传输超过缓存大小就会丢包，所以使用 DMA 模式的话需要发送端需要分包发送。

设备树文件配置

```dts
&uart4 {
        status = "okay";
+       dmas = <&dmac_peri 8>, <&dmac_peri 9>;
+       dma-names = "tx", "rx";
};

```

DMA 模式并不能提高传输速率，相反因为需要缓存，传输速率会有所下降，所以如果不是需要传输大量数据的话不要使用 DMA 模式。

### 流控

不管是中断模式还有 DMA 模式都无法保证数据传输万无一失，因为长时间传输大量数据的时候 DDR、CPU 变频或者占用率过高都可能导致上层处理数据不及时导致丢包，这个时候就需要是用流控了。流控分两种，一种是软件流控另一种为硬件流控，下面只介绍硬件流控的使用。

#### 硬件支持

硬件流控需要有硬件支持，开发板的串口 `CTX` 和 `RTX` 脚需要和设备相连。

**注意：** 开发板不是所有串口都支持硬件流控，请先从原理图上确认硬件是否支持

#### 设备树文件配置

```dts
uart4: serial@ff1c0000 {
        compatible = "rockchip,rk3288-uart", "snps,dw-apb-uart";
        reg = <0x0 0xff1c0000 0x0 0x100>;
        interrupts = <GIC_SPI 59 IRQ_TYPE_LEVEL_HIGH>;
        reg-shift = <2>;
        reg-io-width = <4>;
        clocks = <&cru SCLK_UART4>, <&cru PCLK_UART4>;
        clock-names = "baudclk", "apb_pclk";
        pinctrl-names = "default";
-       pinctrl-0 = <&uart4_xfer>;
+       pinctrl-0 = <&uart4_xfer &uart4_cts &uart4_rts>;
        status = "disabled";
};
```

#### 应用层设置

上层也需要打开流控的设置，这里介绍 stty 如何打开流控如果你使用的是其他应用程序请从应用中打开流控。

```bash
# 打开流控
root@firefly:~# stty -F /dev/ttyS4 crtscts

# 检测流控是否打开，“-” 号代表该功能已经关闭
root@firefly:~# stty -F /dev/ttyS4 -a | grep crtscts
-parenb -parodd -cmspar cs8 hupcl -cstopb cread clocal crtscts
```

## MIPI 摄像头 (OV13850)

RK3288 板子添加了 MIPI 摄像头 `OV13850` 的支持，应用中也添加了摄像头的例子。下面介绍一下相关配置。

### 设备树文件配置

以 `rk3288-firefly-aio.dts` 为例，这些都是摄像头需要打开的配置。

```dts
// 电源管理
&vcc_mipi {
  status = "okay";
};

&dvdd_1v2 {
  status = "okay";
};

// MIPI 摄像头0
&ov13850 {
  status = "okay";
};

&mipi_phy_tx1rx1 {
  status = "okay";
};

// MIPI 摄像头1
/*&ov13850_1 {
  status = "okay";
};

&mipi_phy_rx0 {
  status = "okay";
};*/

// ISP
&rkisp1 {
  status = "okay";
};

&isp_mmu {
  status = "okay";
};
```

**注意：** RK3288 只支持一路 ISP，所以两个摄像头不能同时使用；对于不同的 RK3288 开发板，可能需要修改相应的 GPIO 属性。

### 调试

查看 ISP 驱动是否加载成功，成功可以看到如下设备信息：

```bash
root@firefly:~# grep '' /sys/class/video4linux/video*/name
/sys/class/video4linux/video0/name:rkisp1_mainpath
/sys/class/video4linux/video1/name:rkisp1_selfpath
/sys/class/video4linux/video2/name:rkisp1_dmapath
/sys/class/video4linux/video3/name:rkisp1-statistics
/sys/class/video4linux/video4/name:rkisp1-input-params

root@firefly:~# ls /dev/video*
video0  video1  video2  video3 video4
```

查看 `OV13850` 设备是否注册成功，成功可以看到如下打印：

```bash
root@firefly:~# dmesg |grep 13850
[    1.113850] NET: Registered protocol family 2
[    3.911130] ov13850 3-0010: driver version: 00.01.03
[    3.911168] ov13850 3-0010: GPIO lookup for consumer reset
[    3.911171] ov13850 3-0010: using device tree for GPIO lookup
[    3.911187] of_get_named_gpiod_flags: parsed 'reset-gpios' property of node '/i2c@ff150000/ov13850@10[0]' - status (0)
[    3.911370] ov13850 3-0010: could not get default pinstate
[    3.911373] ov13850 3-0010: could not get sleep pinstate
[    3.911376] ov13850 3-0010: GPIO lookup for consumer pwdn
[    3.911379] ov13850 3-0010: using device tree for GPIO lookup
[    3.911403] of_get_named_gpiod_flags: parsed 'pwdn-gpios' property of node '/i2c@ff150000/ov13850@10[0]' - status (0)
// 成功获取设备 ID
[    3.914401] ov13850 3-0010: Detected OV00d850 sensor, REVISION 0xb2
```

### 测试

可以运行如下脚本进行测试：

```bash
#!/bin/bash

export DISPLAY=:0
export XAUTHORITY=/home/firefly/.Xauthority

WIDTH=640
HEIGHT=480
SINK=gtksink

gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,format=NV12,width=${WIDTH},height=${HEIGHT}, framerate=30/1 ! videoconvert ! $SINK &

wait
```

结果如图所示：

![](../../../rk3288_img/mipi_csi.png)

## 导出开发板根文件系统

此节讲述将如何将一块开发板上的根文件系统导出并打包成镜像文件。适用于将某一开发板的环境完整地复制到其他开发板上，或制作升级固件。

### 操作步骤

1. 在开发板，PC 机安装 `ssh`, `rsync`:

    ```bash
        sudo apt-get install -y ssh rsync
    ```

2. 在开发板上设置 ssh 允许用 root 用户访问

   * 修改 `/etc/ssh/sshd_config` 文件中的 `#PermitRootLogin prohibit-password` 为 `PermitRootLogin yes`
   * 重新启动 ssh 服务：

    ```bash
        sudo /etc/init.d/ssh restart
    ```

3. 在 PC 机中同步开发板的文件系统，请先将开发板和 PC 机置于同一局域网内。（假设开发板地址为 192.168.100.100）

    ```bash
        mkdir rootfs
        sudo rsync -avx root@192.168.100.100:/ ./rootfs
    ```

4. 此时，rootfs 目录中保存的就是开发板文件系统中的所有文件，可以开始制作根文件系统镜像:

    ```bash
        dd if=/dev/zero of=linuxroot.img bs=1M count=4000
        mkfs.ext4 linuxroot.img
        mkdir temp && sudo mount linuxroot.img ./temp
        sudo cp -rfp rootfs/*  temp/
        sudo umount temp/
        e2fsck -p -f linuxroot.img
        resize2fs  -M linuxroot.img
    ```

最终 linuxroot.img 就是做好的根文件系统镜像，和你的开发板的文件系统是一致的。

## 本地编译通用设备总线驱动

DEBIAN 包下载链接：[https://pan.baidu.com/s/1l_2s8gCvpDUb13hTpH8UCA](https://pan.baidu.com/s/1l_2s8gCvpDUb13hTpH8UCA)(提取码：1234)。

安装头文件：

```shell
sudo dpkg -i linux-headers-4.4.194_4.4.194-93_armhf.deb
sudo dpkg -i linux-image-4.4.194_4.4.194-93_armhf.deb

cd /usr/src/linux-headers-4.4.194
make headers_check
make headers_install

# make scripts 可能会出错但是不要紧
make scripts
```