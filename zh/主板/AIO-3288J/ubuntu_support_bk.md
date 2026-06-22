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

    更多相关资料和Demo，请参考

    https://github.com/rockchip-linux/mpp/tree/develop/test

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

## ADB 调试

ADB，全称 Android Debug Bridge，是 Android 的命令行调试工具，可以完成多种功能，如跟踪系统日志，上传下载文件，安装应用等。以 Firefly-RK3288 为例，为了在 Ubuntu 系统也能使用 ADB 工具进行调试，我们移植了 ADB 服务。但由于并非 Android 系统，很多 ADB 命令类似 `adb logcat`、`adb install` 等不能使用，仅作为普通的调试辅助工具，可以进行 shell 交互、上传下载文件等操作。同样的网络远程 ADB 调试也不能使用。

### 下载 ADB

在 Ubuntu 系统中，执行：

```bash
sudo apt-get install android-tools-adb
```

### 内核配置

在内核目录下，打开内核配置选项菜单：

```bash
make firefly_linux_defconfig
make menuconfig
```

进入内核配置菜单后依次选择：`Device Drivers` -> `USB Support` -> `USB Gadget Support`。

在 `USB Gadget Driver` 选项中选择到 `USB functions configurable through configfs`。

同时再选择上 `Function filesystem (FunctionFS)` 。

```c
<*>   USB Gadget Drivers (USB functions configurable through configfs)  --->
        USB functions configurable through configfs
[*]       Function filesystem (FunctionFS)
```

接着在 `kernel` 目录下编译内核：

```bash
make rk3288-firefly.img -j12
```

编译完成后将内核烧录到开发板中,烧录过程请参考维基教程：[升级固件](upgrade_firmware.md)中的分区镜像烧写部分。

### ADB 连接

安装了 ADB 后，用 Micro USB OTG 线连接开发板和 PC 机。然后通过命令 `adb devices` 查看是否有设备连接:

```bash
firefly@Desktop:~$ adb devices
List of devices attached
0123456789ABCDEF        device
```

从返回的信息可以看到已经查找到了设备，表明设备已经成功连接。

设备连接成功后，输入命令 `adb shell` 即可进入命令行模式：

```bash
firefly@Desktop:~$ adb shell
#
```

此状态下是看不到命令行当前的路径，同时 Tab 键补全功能失效，需要进入一个用户中才可以正常操作。

```bash
firefly@Desktop:~$ adb shell
# su firefly
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

firefly@firefly:/$
#到这里命令行就可以正常使用了
```

用户也可以使用命令 `adb shell /bin/bash`，也可以进入正常的命令行模式。

可以输入 `adb help` 查看命令行帮助信息，注意并不是所有命令都可以使用，帮助信息只做参考。

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

## 外部存储设备 rootfs

根文件系统除了可以使用在内部的 eMMC 中的，还可以使用外部存储设备的根文件系统，如 SD 卡，U 盘等。以下是以 SD 卡为例，在 Firefly-RK3288 开发板上实现挂载外部存储设备的根文件系统。

### 在 SD 卡上建立分区

在 PC 机上插入 SD 卡，用 `gdisk` 工具分出一个装载根文件系统的分区：

```bash
#用户请用 `fdisk -l` 查询 SD 卡设备名是什么，再用 gdisk 命令进入对应的设备
sudo gdisk /dev/sdb

GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help):

-------------------------------------------------------------------

#输入 ? 查看帮助信息
# 按 p 显示 SD 卡当前已有分区，由于使用的 SD 卡暂未创建分区，所以没有查询到分区信息

Command (? for help): p
Disk /dev/sdb: 15278080 sectors, 7.3 GiB
Model: CARD-READER
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 5801AE61-92ED-42FB-A144-E522E8E15827
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 15278046
Partitions will be aligned on 2048-sector boundaries
Total free space is 15278013 sectors (7.3 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name

-------------------------------------------------------------------

#创建新的分区，请用户根据自身实际情况创建合适的分区大小给根文件系统

Command (? for help): n     #输入 n 创建新分区
Partition number (1-128, default 1): 1      #输入1创建第一个分区
First sector (34-15278046, default = 2048) or {+-}size{KMGTP}:   #直接按回车，使用默认值
Last sector (2048-15278046, default = 15278046) or {+-}size{KMGTP}: +3G     #创建分区大小为3G
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300):       #按回车使用默认值
Changed type of partition to 'Linux filesystem'

-------------------------------------------------------------------

#创建完成后，输入 i，可以查看分区的 unique GUID

Command (? for help): i
Using 1
Partition GUID code: 0FC63DAF-8483-4772-8E79-3D69D8477DE4 (Linux filesystem)
Partition unique GUID: 6278AF5D-BC6B-4213-BBF6-47D333B5CB53   #使用的是该分区的 unique GUID，记下前12位即可
First sector: 2048 (at 1024.0 KiB)
Last sector: 6293503 (at 3.0 GiB)
Partition size: 6291456 sectors (3.0 GiB)
Attribute flags: 0000000000000000
Partition name: 'Linux filesystem'

-------------------------------------------------------------------

#输入 wq 保存修改并退出

Command (? for help): wq

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot or after you
run partprobe(8) or kpartx(8)
The operation has completed successfully.

```

把新创建的分区进行格式化。

```bash
# 注意 sdb1 表示 SD 卡对应分区设备名，请用户根据实际情况填入
sudo mkfs.ext4 /dev/sdb1

```

格式化完成后，使用 `dd` 命令，将制作好的根文件系统烧写到 SD 卡刚新建的分区中。

```bash
# 用户请根据实际情况填写根文件系统镜像的路径和 SD 卡对应分区设备名
dd if=/rootfs_path/rootfs.img of=/dev/sdb1
```

### 挂载 SD 卡根文件系统

首先要修改设备树文件，打开对应的 dts 文件，在根节点下重写 `chosen` 节点下的 `bootargs` 参数的 `root` 值为写入了根文件系统的 SD 卡分区的 unique GUID：

```dts
//将 root 的值改为获取到的 unique GUID 值的前12位
chosen {
    bootargs = "earlycon=uart8250,mmio32,0xff1a0000 swiotlb=1 console=ttyFIQ0 rw root=PARTUUID=6278AF5D-BC6B  rootfstype=ext4 rootwait";
};

```

修改完成后编译并烧录至开发板中。

SD 卡的根文件系统烧录完成后，插入开发板的 TF 卡槽中，开机即可进入到 SD 的根文件系统中。

注意事项：

- 在操作前请注意备份 U 盘或者 SD 卡内的重要文件，避免丢失；
- 操作时请确认自己的 SD 卡对应的设备名；
- dts 文件中 `root` 参数的值使用的是 unique GUID，记下前12位即可。用户也可以根据 `gdisk` 的帮助信息自行修改这个值。

## 网络启动

网络启动，是用 TFTP 在服务器下载内核、dtb 文件到目标机的内存中，同时可以用 NFS 挂载网络根文件系统到目标机上，实现目标机的无盘启动。以下基于 Firefly-RK3288 作出一个示例，提供用户参考。

准备工作：

- Firefly-RK3288 开发板；
- 路由器、网线；
- 安装有 NFS 和 TFTP 的服务器；
- 一份制作好的根文件系统。

注：示例中使用的是 Ubuntu 系统的 PC 机作为服务器，通过路由器和网线实现与开发板的连接。用户可以根据自己的实际情况作调整，但如果是 PC 机直连开发板，请使用交叉网线。请确保服务器和目标机在同一局域网内。

### 服务器部署

1、在服务器上部署 TFTP 服务：

安装 TFTP 服务：

```bash
sudo apt-get  install  tftpd-hpa
```

创建 `/tftpboot` 目录并赋予权限：

```bash
mkdir /tftpboot
sudo chmod 777 /tftpboot
```

然后修改 TFTP 服务器的配置文件 `/etc/default/tftpd-hpa`，修改内容为：

```bash
# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/tftpboot"   #这个根据用户实际的 tftp 目录设置
TFTP_ADDRESS="0.0.0.0:69"
TDTP_OPTIONS="-c -s -l"
```

退出修改后重启 TFTP 服务器：

```bash
sudo service tftpd-hpa restart
```

在 `/tftpboot` 目录中创建一个文件，测试 TFTP 服务是否可用：

```bash
firefly@Desktop:~$ cd /tftpboot/
firefly@Desktop:/tftpboot$ touch test
firefly@Desktop:/tftpboot$ cd /tmp
firefly@Desktop:/tmp$ tftp 127.0.0.1
tftp> get test
tftp> q
```

查看 `/tmp` 目录中的文件，如果看到了 `test` 文件表示 TFTP 服务器是可用的。

2、在服务器上部署 NFS 服务：

安装 NFS 服务器：

```bash
sudo apt-get install nfs-kernel-server
```

创建共享目录：

```bash
sudo mkdir /nfs
sudo chmod 777 /nfs
cd /nfs
sudo mkdir rootfs
sudo chmod 777 rootfs
```

然后将制作好的根文件系统复制到 `/nfs/rootfs` 目录中。

接着配置 NFS，在 `/etc/exports` 中添加共享目录位置：

```bash
/nfs/rootfs *(rw,sync,no_root_squash,no_subtree_check)
```

共享的目录根据用户实际情况设置，其中的 "*" 代表的是所有用户可访问。

重启 NFS 服务器：

```bash
sudo /etc/init.d/nfs-kernel-server restart
```

本地挂载共享目录，测试 NFS 服务器是否可用：

```bash
sudo mount -t nfs 127.0.0.1:/nfs/rootfs/ /mnt
```

在 `/mnt` 目录下查看到了与 `/nfs/rootfs` 一致的内容，则说明 NFS 服务器部署成功。然后可以解除挂载：

```bash
sudo umount /mnt
```

### 内核的配置

如果要做到挂载网络根文件系统，需要在内核中做相关配置，并在 dts 中修改相关挂载根文件系统的配置。

首先进行内核配置，在内核目录中执行 `make menuconfig`，选择相关配置：

```c
[*] Networking support  --->
         Networking options  --->
                [*]   IP: kernel level autoconfiguration
                [*]     IP: DHCP support
                [*]     IP: BOOTP support
                [*]     IP: RARP support


File systems  --->
        [*] Network File Systems  --->
                [*]   Root file system on NFS
```

修改 `rk3288-firefly.dts` 配置，在 dts 文件中修改 `chosen` 节点下的 `bootargs` 参数，选择使用 NFS 挂载远程根文件系统，内容如下。

```bash
chosen {
    bootargs = "earlycon=uart8250,mmio32,0xff1a0000 swiotlb=1 console=ttyFIQ0 root=/dev/nfs rootfstype=ext4 rootwait";
};

#主要是修改 root 的值，root=/dev/nfs 表示通过 NFS 挂载网络根文件系统
```

编译内核：

```bash
make ARCH=arm rk3288-firefly.img -j12
```

编译完成后，将编译好的内核文件 `zboot.img` 和 `rk3288-firefly.dtb` 文件复制到 `/tftpboot` 目录中：

```bash
cp zboot.img /tftpboot
cp arch/arm/boot/dts/rk3288-firefly.dtb /tftpboot
```

详细说明可以参考内核目录中的 `kernel/Documentation/filesystems/nfs/nfsroot.txt`

### U-Boot设置

请先确保目标机网线已插入，接入到服务器的局域网内。

目标机启动进入 U-Boot 命令行模式，设置以下参数：

```bash
=> setenv ipaddr 192.168.31.101     #设置目标机 IP 地址
=> setenv serverip 192.168.31.106   #设置 serverip 为服务器 IP 地址

#设置从 TFTP 下载内核和 dtb 文件到相应地址，用户请根据自己实际的目标机修改相应地址
=> setenv bootcmd tftpboot 0x02007800 zboot.img \; tftpboot 0x08300000 rk3288-firefly.dtb \; bootm 0x02007800 - 0x08300000

#设置挂载网络根文件系统，IP 参数依次为：目标机 IP:服务器 IP:网关:网络掩码:设备名:off，可以更简单的设置 ip=dhcp，通过 DHXP 自动分配 IP
=> setenv bootargs root=/dev/nfs rw nfsroot=192.168.31.106:/nfs/rootfs,v3 ip=192.168.31.101:192.168.31.106:192.168.31.1:255.255.255.0::eth0:off

#如果不想每次开机都设置一遍，可以保存上面配置的变量
=> saveenv
Saving Environment to MMC...
Writing to MMC(0)... done

#启动目标机
=> boot
ethernet@fe300000 Waiting for PHY auto negotiation to complete. done
Speed: 100, full duplex
Using ethernet@fe300000 device
TFTP from server 192.168.31.106; our IP address is 192.168.31.101
Filename 'zboot.img'.
Load address: 0x2007800
Loading: #################################################################
         #################################################################
         #################################################################
         #################################################################
         #################################################################
         ##################################################
         475.6 KiB/s
done
Bytes transferred = 20072448 (1324800 hex)
Speed: 100, full duplex
Using ethernet@fe300000 device
TFTP from server 192.168.31.106; our IP address is 192.168.31.101
Filename 'rk3288-firefly.dtb'.
Load address: 0x8300000
Loading: #######
         645.5 KiB/s
done
Bytes transferred = 97212 (17bbc hex)
## Booting Android Image at 0x02007800 ...
Kernel load addr 0x02008000 size 19377 KiB
## Flattened Device Tree blob at 08300000
   Booting using the fdt blob at 0x8300000
   XIP Kernel Image ... OK
   Loading Device Tree to 0000000073edc000, end 0000000073ef6bbb ... OK
Adding bank: 0x00000000 - 0x08400000 (size: 0x08400000)
Adding bank: 0x09300000 - 0x80000000 (size: 0x76d00000)
Total: 912260.463 ms

Starting kernel ...

...
```

在开机内核日志中可见：

```c
[   12.146297] VFS: Mounted root (nfs filesystem) on device 0:16.
```

说明已经挂载上了网络根文件系统。

注意事项

- 确保 TFTP 服务器、NFS 服务器可用;
- 确保目标机先插入网线后在开机，且和服务器在同一局域网内，如果是直连目标机和服务器，请使用交叉网线;
- 内核配置中，`Root file system on NFS` 依赖于 `IP: kernel level autoconfiguration` 选项，请先选择 `IP: kernel level autoconfiguration`，之后才可以找到 `Root file system on NFS` 选项;
- 在 U-Boot 命令行中，请确认 `zboot.img` 烧录地址和 dtb 文件烧录地址。（提示：`zboot.img` 的文件结构中，开头有2k的头文件，然后才是 kernel。所以在 TFTP 下载内核到目标机时，要下载到对应 kernel 地址减去0x0800的地址上）;
- 在设置挂载远程根文件系统时，`nfsroot=192.168.31.106:/nfs/rootfs,v3` 中的 `v3` 代表 NFS 版本信息，请添加上以避免出现挂载不成功的问题。

## 在线更新内核和 U-Boot

本小节介绍了在线更新的一个简单的流程。将内核、U-Boot 或者其他需要更新的文件打包成 deb 安装包，然后导入到本地包仓库，实现在开发板上下载并自动更新。仅供用户参考。

### 准备 deb 安装包

操作中需要升级内核和 U-Boot，事先已经准备好了修改好的相关文件：`uboot.img` 、`trust.img` 、`zboot.img` 。

deb 是 Debian Linux 的软件包格式，打包最关键的是在 `DEBIAN` 目录下创建一个 `control` 文件。首先创建 deb 工作目录，然后在 deb 目录中创建相应的目录和文件：

```bash
mkdir deb
cd deb
mkdir firefly-firmware    #创建 firefly-firmware 目录
cd firefly-firmware
mkdir DEBIAN    #创建 DEBIAN 目录，这个目录是必须要有的。
mkdir -p usr/share/{kernel,uboot}
mv ~/zboot.img ~/deb/firefly-firmware/usr/share/kernel  #将相应文件放到相应的目录
mv ~/uboot.img ~/deb/firefly-firmware/usr/share/uboot
mv ~/trust.img ~/deb/firefly-firmware/usr/share/uboot
```

`DEBIAN` 目录下存放的文件是 deb 包安装的控制文件以及相应的脚本文件。

在 `DEBIAN` 目录下创建控制文件 `control` 和脚本文件 `postinst`。

`control` 文件内容如下,用于记录软件标识，版本号，平台，依赖信息等数据。

```bash
Package: firefly-firmware #文件目录名
Version: 1.0    #版本号
Architecture: armhf #架构
Maintainer: neg   #维护人员，自定义即可
Installed-Size: 1
Section: test
Priority: optional
Descriptionon: This is a deb test
```

`postinst` 文件内容如下,就是将需要更新的内核和 U-Boot 文件用 `dd` 命令写进对应分区的脚本：

```bash
echo "-----------uboot updating------------"
dd if=/usr/share/uboot/uboot.img of=/dev/disk/by-partlabel/uboot

echo "-----------trust updating------------"
dd if=/usr/share/uboot/trust.img of=/dev/disk/by-partlabel/trust

echo "-----------kernel updating------------"
dd if=/usr/share/kernel/zboot.img of=/dev/disk/by-partlabel/boot
```

说明：`postinst` 脚本，是在解包数据后运行的脚本，对应的还有:

- preinst：在解包数据前运行的脚本;
- prerm：卸载时，在删除文件之前运行的脚本;
- postrm：在删除文件之后运行的脚本;

其他相关知识点用户可以自行了解。这里只用到了 `preinst` 脚本。

以下是创建好的目录树：

```bash
deb
└── firefly-firmware
    ├── DEBIAN
    │   ├── control
    │   └── postinst
    └── usr
        └── share
            ├── kernel
            │   └── zboot.img
            └── uboot
                ├── trust.img
                └── uboot.img
```

进入 deb 目录，用 `dpkg` 命令生成 deb 包：

```bash
dpkg -b firefly-firmware firefly-firmware_1.0_armhf.deb
```

生成 deb 包时，一般按照这种规范进行命名：`package_version-reversion_arch.deb` 。

### 创建本地仓库

首先安装需要的包：

```bash
sudo apt-get install reprepro gnupg
```

然后用 GnuPG 工具生成一个 GPG 密匙，执行命令后请根据提示操作：

```bash
gpg --gen-key
```

执行 `sudo gpg --list-keys` 可以查看刚刚生成的密匙信息:

```bash
sudo gpg --list-keys

gpg: WARNING: unsafe ownership on homedir '/home/firefly/.gnupg'
gpg: 正在检查信任度数据库
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: 深度：0 有效性：  4 已签名：  0 信任度：0-，0q，0n，0m，0f，4u
gpg: 下次信任度数据库检查将于 2021-05-29 进行
/home/firefly/.gnupg/pubring.kbx
--------------------------------
pub   rsa3072 2019-05-31 [SC] [有效至：2021-05-30]
      BCB65788541D632C057E696B8CBC526C05417B76
uid           [ 绝对 ] firefly <firefly@t-chip.com>
sub   rsa3072 2019-05-31 [E] [有效至：2021-05-30]
```

接下来创建包仓库，首先创建目录:

```bash
cd /var/www
mkdir apt   #包仓库目录
mkdir -p ./apt/incoming
mkdir -p ./apt/conf
mkdir -p ./apt/key
```

把前面生成的密匙导出到仓库文件夹，请用户对应好自己创建的用户名和邮箱地址。

```bash
gpg --armor --export firefly firefly@t-chip.com > /var/www/apt/key/deb.gpg.key
```

在 `conf` 目录下创建 `distributions` 文件，其内容如下：

```bash
Origin: Neg   #你的名字
Label: Mian     #库的名字
Suite: stable   #(stable 或 unstable)
Codename: bionic    #发布代码名，可自定义
Version: 1.0    #发布版本
Architectures: armhf    #架构
Components: main    #组件名，如main，universe等
Description: Deb source test        #相关描述
SignWith: BCB65788541D632C057E696B8CBC526C05417B76  #上面步骤中生成的 GPG 密匙
```

建立仓库树：

```bash
reprepro --ask-passphrase -Vb /var/www/apt export
```

将第2步做好的 deb 包加入到仓库中：

```bash
reprepro --ask-passphrase -Vb /var/www/apt includedeb bionic ~/deb/firefly-firmware_1.0_armhf.deb
```

可以查看库中添加的文件：

```bash
root@Desktop:~# reprepro -b /var/www/apt/ list bionic
bionic|main|armhf: firefly-firmware 1.0
```

你的包已经加入了仓库，如果要移除它的话采用如下命令:

```bash
reprepro --ask-passphrase -Vb /var/www/apt remove bionic firefly-firmware
```

安装 nginx 服务器：

```bash
sudo apt-get install nginx
```

修改nginx的配置文件 `/etc/nginx/sites-available/default` 为：

```c
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/apt;      //本地包仓库的路径

    access_log /var/log/nginx/repo.access.log;
    error_log   /var/log/nginx/repo.error.log;

    location ~ /(db|conf) {
        deny all;
        return 404;
    }
}
```

重启 nginx 服务器：

```bash
sudo service nginx restart
```

### 客户端更新安装

在客户端开发板中，首先要添加本地包仓库的源，在目录 `/etc/apt/sources.list.d` 下添加一个新的配置文件 `bionic.list`，内容如下：

```bash
deb http://192.168.31.106 bionic main
```

IP 地址是服务器地址, `bionic` 是仓库发布代码名， `main` 是组件名。

从服务器中获取并添加 GPG 密匙：

```bash
wget -O - http://192.168.31.106/key/deb.gpg.key | apt-key add -
```

更新后即可安装自定义软件源里的 `firefly-firmware_1.0_armhf` 包：

```bash
root@firefly:/home/firefly# apt-get update
Hit:1 http://192.168.31.106 bionic InRelease
Hit:2 http://wiki.t-firefly.com/firefly-rk3288-repo bionic InRelease
Hit:3 http://ports.ubuntu.com/ubuntu-ports bionic InRelease
Hit:4 http://archive.canonical.com/ubuntu bionic InRelease
Hit:5 http://ports.ubuntu.com/ubuntu-ports bionic-updates InRelease
Hit:6 http://ports.ubuntu.com/ubuntu-ports bionic-backports InRelease
Hit:7 http://ports.ubuntu.com/ubuntu-ports bionic-security InRelease
Reading package lists... Done

root@firefly:/home/firefly# apt-get install firefly-firmware
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libllvm6.0
Use 'apt autoremove' to remove it.
The following NEW packages will be installed:
  firefly-firmware
0 upgraded, 1 newly installed, 0 to remove and 3 not upgraded.
Need to get 0 B/6982 kB of archives.
After this operation, 1024 B of additional disk space will be used.
Selecting previously unselected package firefly-firmware.
(Reading database ... 117088 files and directories currently installed.)
Preparing to unpack .../firefly-firmware_1.0_armhf.deb ...
Unpacking firefly-firmware (1.0) ...
Setting up firefly-firmware (1.0) ...
-----------uboot updating------------
8192+0 records in
8192+0 records out
4194304 bytes (4.2 MB, 4.0 MiB) copied, 0.437281 s, 9.6 MB/s
-----------trust updating------------
8192+0 records in
8192+0 records out
4194304 bytes (4.2 MB, 4.0 MiB) copied, 0.565762 s, 7.4 MB/s
-----------kernel updating------------
39752+0 records in
39752+0 records out
20353024 bytes (20 MB, 19 MiB) copied, 0.1702 s, 120 MB/s
```

可以看到安装过程中执行了 deb 包中的 `poinst` 脚本。安装完成后，重启开发板，更新完成。

在 `/usr/share` 目录中还可以看到 `kernel` 和 `uboot` 目录，分别存放了 `zboot.img`，`uboot.img` 和 `trust.img` 。

注意事项

- 在制作 deb 包中，与 `DEBIAN` 同级的目录视为根目录，即放在与 `DEBIAN` 同目录下的文件，在客户端安装 deb 包后，可以在客户端的根目录下找到；
- deb 包中的文件和脚本，用户要根据自己的实际情况做调整；
- 每次在仓库中修改了配置文件后，都要重新导入仓库目录树 ；
- nginx 服务器配置中，`root` 参数配置的是仓库的地址，请用户根据自己的实际情况修改；
- 客户端添加的新下载源的文件时，注意核对正确的服务器地址，包仓库代码名和组件名。注意客户端要连通服务器；
- 客户端要用 `apt-key add` 命令添加 GPG 密匙之后，才可以获取本地的仓库的信息。

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