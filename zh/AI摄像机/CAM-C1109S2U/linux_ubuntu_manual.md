# Firefly Ubuntu 使用手册

本使用手册适用与Firefly Ubuntu Desktop & Minimal 系统，部分与UI显示相关的介绍，只针对Desktop系统。

## 本地编译通用设备总线驱动

DEBIAN 包下载链接：[linux-headers](https://www.t-firefly.com/doc/download/98.html#other_485)

安装头文件：

```shell
sudo dpkg -i linux-headers-4.19.172_4.19.172-189_arm64.deb
sudo dpkg -i linux-image-4.19.172_4.19.172-189_arm64.deb

make headers_check
make headers_install

# make scripts 可能会出错但是不要紧
make scripts
```
## Docker 支持
Firefly 发布的普通固件一般不满足 Docker 的运行要求，如果有需求，可以使用 SDK 打开内核的相关配置，重新编译烧录内核以支持 Docker。SDK 的获取以及使用请查看编译 Ubuntu 固件。完成编译前配置之后，下面开始介绍支持 Docker 的方法：
### 检查 Kernel 配置
首先需要通过工具检查当前设备的内核缺少了哪些 Docker 需要的配置。检测脚本`check-config.sh`和现成的 RK356X Docker 配置文件可以前往[社区论坛](https://dev.t-firefly.com/thread-115083-1-1.html)获取。如果使用现成的配置文件可忽略以下内容。

获取到脚本之后，开始进行检测：
```bash
#将脚本拷贝到SDK的kernel目录下
cp check-config.sh PathToSDK/kernel/
cd PathToSDK/kernel
chmod +x check-config.sh

#获取当前内核配置
make ARCH=arm64 firefly_linux_defconfig

#检测
./check-config.sh .config
```
执行后的结果如下，主要是两部分：
```
Generally Necessary:
- cgroup hierarchy: properly mounted [/sys/fs/cgroup]
- apparmor: enabled and tools installed
- CONFIG_NAMESPACES: enabled
- CONFIG_NET_NS: enabled
- CONFIG_PID_NS: enabled
- CONFIG_IPC_NS: enabled
- CONFIG_UTS_NS: enabled
- CONFIG_CGROUPS: enabled
......

Optional Features:
- CONFIG_USER_NS: enabled
- CONFIG_SECCOMP: enabled
- CONFIG_SECCOMP_FILTER: enabled
- CONFIG_CGROUP_PIDS: enabled
- CONFIG_MEMCG_SWAP: enabled
......
```
Generally Necessary: 表示必要的配置，如果有显示 missing 的地方，就需要在内核配置中打开它。
Optional Features: 是可选配置，根据需要打开。
### 开启需要的配置
从上面的检测结果中得知需要打开哪些配置后，即可使用`make ARCH=arm64 menuconfig`进入菜单，搜索对应项目将其打开。请认真查看菜单中的操作说明，遇到不可选中的项目请注意依赖关系。

开启所有必要配置以及部分可选配置后，注意保存：
```bash
make ARCH=arm64 savedefconfig
mv defconfig arch/arm64/configs/firefly_linux_defconfig
```
之后进行编译内核：
```
#退回到SDK目录
cd ..
#编译内核
./build.sh kernel
```
### 安装 Docker
烧录完新内核之后，可以开始在设备上安装 Docker 并验证：
```bash
#这里仅介绍直接使用脚本快速安装
apt-get update
wget -qO- https://get.docker.com/ | sh
```
等待安装成功之后应该会看见 Docker 版本信息，然后运行 demo：
```
root@firefly:~# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
93288797bd35: Pull complete
Digest: sha256:cc15c5b292d8525effc0f89cb299f1804f3a725c8d05e158653a563f15e4f685
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm64v8)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

## Qt交叉编译环境支持
Firefly发布的Qt交叉编译工具链，适用于以下环境:
- Host:  x86-64 / Ubuntu 18.04- Target:  Firefly RV1126_RV1109 RK3588 RK3328 PX30 / Ubuntu 18.04 Minimal&Desktop
- Target:  Firefly  RK356X RK3588 RK3328 PX30 / Ubuntu 18.04 Minimal&Desktop
工具链完整支持wenEngine, 支持EGLFS LinuxFB XCB 等backend.

* 下载地址
```
链接：https://pan.baidu.com/s/1tNGqG4XSYuDxe_AdDegOAA
提取码：FFQT
```
* 部署

**详情参见Qt5.12.2_Release.md**
注意，文档中所有路径的名称不可更改，否则会导致编译或者运行出错。

* 编译

在host端，进入Qt工程目录，qmake  && make 即可.

* 运行

工具链中提供了两个测试Demo，分别对应EGLFS和LinuxFB Backend，用户在部署完成后，可以在host端build demo，在tartget端运行demo以测试部署是否成功。

## 导出设备系统

此章节适用于:
当用户已经在一台设备上完成工作环境的部署，需要将当前环境完整导出，以批量部署到其它同平台设备上。

导出设备系统分为两步
1. 导出设备rootfs
2. 二次打包完整固件，将rootfs与其它固件组成部分二次打包，生成完整固件

### 导出设备rootfs
1. 在ubuntu18.04环境下，安装 `fireflydev`

	```
	sudo apt update
	sudo apt install fireflydev
	```

2. 使用`ff_export_rootfs`导出根文件系统

	* 建议使用容量较大的移动硬盘

	* 导出工具会执行apt clean等操作以减小文件系统大小

	* 将根文件系统导出到`/media/firefly/AC91-C4AE/`目录下，实际例子如下：
	```
        root@firefly:~# ff_export_rootfs
        ff_export_rootfs </path/to/store> [-t <ext4|btrfs>]

	ff_export_rootfs /media/firefly/AC91-C4AE/
	```

### 二次打包完整固件
* 工具下载：[firefly-linux-repack](https://www.t-firefly.com/doc/download/98.html#other_479)
* 本工具适用于用户需要替换固件中任意一组成部分

```
firefly-linux-repack
    ├── bin
    │   ├── afptool
    │   └── rkImageMaker
    ├── pack.sh
    ├── Readme_en.md
    ├── Readme.md
    └── unpack.sh

```
1. 解包
    把官方发布的固件拷贝到当前目录，重命名为update.img , 执行unpack.sh
    解包完成后，生成的文件在output目录下.

2. 合包
    保持当前目录结构，文件名等不变，用客户自己的文件替换output/下同名的文件
    执行pack.sh, 执行完后，生成new_update.img，即为打包好的固件
	rootfs文件名必须为rootfs.img
	parameter.txt文件名必须为parameter.txt

* 注意：
	合包过程中，如果rootfs分区不是最后一个分区，那么程序会根据rootfs文件的大小，
	自动修改parameter.txt中rootfs分区的大小。
	如果用户自己有改动parameter.txt，请留意整个合包的流程。

```
tar -xzf firefly-linux-repack.tgz
cd firefly-linux-repack
mv /xxx/FIREFLY-RK3588-UBUNTU18.04-GPT-20210420-1510.img update.img
./unpack.sh

cp /customer/rootfs.img  output/Image/rootfs.img

./pack.sh
ls new_update.img
```

## 屏幕旋转

Firefly Ubuntu Desktop 使用ff_rotate脚本来控制屏幕旋转：

* ff_rotate

```
root@firefly:~# ff_rotate

        rotate screen and touchscreen
                run as root
                ff_rotate <orientation>
                        orientation: left, right, normal, inverted
#图像输出翻转
root@firefly:~# ff_rotate inverted
```

## 显示版本信息
* ffgo

通过Firefly 提供的ffgo命令可以方便的查看固件信息，便于开发者调试以及定位问题。

当用户需要向Firefly反馈信息时，需要附上ffgo version显示的版本信息。

```
root@firefly:~# ffgo
Usage:
         ffgo :         show this usage
         ffgo update:   update ffgo
         ffgo version:  get version
         ffgo cmdlist:  get support cmd list
         ffgo [cmd]:    run cmd in cmd list

root@firefly:~# ffgo update
update success
root@firefly:~#  ffgo version
OS:      Ubuntu 18.04.5 LTS
MODEL:   Firefly RK3588-ROC-PC HDMI(Linux)
FIREFLY: v2.04-1-g618089a
DATE:    20210316-1035
KERNEL:  Linux version 4.19.172 (liaoxt@tchip16) (gcc version 6.3.1 20170404 (Linaro GCC 6.3-2017.05), GNU ld (Linaro_Binutils-2017.05) 2.27.0.20161019) #107 SMP Mon Apr 19 09:01:32
```

## 恢复出厂设置

Firefly Ubuntu 支持恢复出厂设置。
**注意，此出厂设置表示恢复为设备最后一次升级固件之后的初始状态**
```
root@firefly:~# recovery
Usage: update <option>

Options are:
        ota | update [path]:
                update firmware, if no path, use default path
                "/userdata/update.img"
                "/sdcard/update.img"
        factory | reset:
                reset to factory
```

使用 `recovery reset` 可以恢复到出厂设置


## 开机自启动程序

Firefly Ubuntu Desktop 可以使用以下方法设置开机自启动程序，不过需要注意操作权限和当前环境变量：


```
$vim /home/firefly/.config/lxsession/Lubuntu/autostart
@/bin/mkdir /home/firefly/ssddd
```
每个命令使用`@`开头，以上面的例子为例会创建`/home/firefly/ssddd`目录



## 视频硬件编解码支持

RK356X集成的VPU具有优秀的视频编解码能力，Mpp是Rockchip为VPU提供的一套视频编解码的api, 并且基于mpp。Rockchip提供了一套gstreamer的编解码插件。用户可以根据自己的需求，基于gstreamer来做视频编解码的应用，或者直接调用mpp，来实现硬件的编解码加速。

系统提供了一个测试视频文件，位于/usr/local/test.mp4 ，测试文件为1080P, 24Fps, H264编码, Mp4格式。

可以通过一下几种方式，验证和开发视频编解码相关应用。

### Gstreamer

*   Ubuntu 18.04下， gstreamer 1.12 已经安装到系统中。

/usr/local/bin/h264dec.sh  测试硬件H264解码。

/usr/local/bin/h264enc.sh  测试硬件H264编码。

用户可以参照这两个脚本，配置自己的gstreamer应用。

### Mpv

系统提供的Mpv播放器，可以直接调用rkmpp解码插件。

### FFmpeg

FFmpeg 对于 Rockchip 暂时只支持通过 Mpp 实现硬件解码，暂时没有硬件编码的支持。

Firefly Ubuntu 已经安装了 FFmpeg，用户可以直接使用。

-   确认rkmpp解码器

```shell
$ ffmpeg -decoders | grep "rkmpp"

 V..... h264_rkmpp           h264 (rkmpp) (codec h264)
 V..... hevc_rkmpp           hevc (rkmpp) (codec hevc)
 V..... vp8_rkmpp            vp8 (rkmpp) (codec vp8)
 V..... vp9_rkmpp            vp9 (rkmpp) (codec vp9)
```

-   测试命令

```shell
$ ffmpeg -y -c:v h264_rkmpp -i test.mp4 -an -vf \
	hwdownload,format=nv12,format=yuv420p output.yuv
```

**特别注意：** FFmpeg h264_rkmpp 解码获得的是 AV_PIX_FMT_DRM_PRIME，也就是 DRM 帧数据，如果是基于drm显示，可以直接输出帧，否则，需要用 hwdownload 进行转换。

更多内容可以参考 [FFmpeg 官网](http://ffmpeg.org/about.html)。

### Mpp

*   Ubunut 系统下， mpp 相关dev包都已经安装到系统中。

    更多相关资料,可参考`linux-sdk/docs/Linux/Multimedia`下的相关文档

## OpenGL-ES

RK356X 支持 OpenGL ES1.1/2.0/3.0/3.1。

Firefly Ubuntu 已经提供了完整的OpenGL-ES支持。

-   测试命令

```shell
$ sudo test_glmark2_normal.sh
```

-   webGL支持

在Chromium浏览器中， 在地址栏输入：`chrome://gpu`可以查看chromium下硬件加速的支持。

Note:

1.  EGL 是用arm 平台上OpenGL针对x window  system的扩展，功能等效于x86下的glx库。

2.  由于Xorg使用的Driver modesettings 默认会加载libglx.so(禁用glx会导致某些通过检测glx环境的应用启动失败)， libglx.so会搜索系统中的dri实现库。但是RV1126_RV1109 Xorg 2D加速是直接基于DRM实现， 并未实现dri库，所以启动过程中，libglx.so会报告如下的错误 。

    ```
    (EE) AIGLX error: dlopen of /usr/lib/aarch64-linux-gnu/dri/rockchip_dri.so failed
    ```

    这个对系统运行没有任何影响，不需要处理。

3.  基于同样的道理，某些应用启动过程中，也会报告如下错误，不用处理，对应用的运行不会造成影响。

    ```
    libGL error: unable to load driver: rockchip_dri.so
    libGL error: driver pointer missing
    libGL error: failed to load driver: rockchip
    ```

4.  Firefly之前发布的某些版本的Ubuntu软件，默认关闭了加载libglx.so，在某些情况下，运行某些应用程序会出现下述错误：

    `GdkGLExt-WARNING **: Window system doesn't support OpenGL.`

    修正的方法如下：

    删除  `/etc/X11/xorg.conf.d/20-modesetting.conf` 中一下三行配置。

    ```shell
    Section "Module"
         Disable     "glx"
    EndSection
    ```



## OpenCL

Firefly Ubuntu已经添加了opencl1.2支持，可以运行系统内置的`clinfo`获取平台opencl相关参数。

```
firefly@firefly:~$ clinfo
Platform #0
 Name:                                  ARM Platform
 Version:                               OpenCL 1.2 v1.r14p0-01rel0-git(966ed26).f44c85cb3d2ceb87e8be88e7592755c3

 Device #0
   Name:                                Mali-T860
   Type:                                GPU
   Version:                             OpenCL 1.2 v1.r14p0-01rel0-git(966ed26).f44c85cb3d2ceb87e8be88e7592755c3
   Global memory size:                  1 GB 935 MB 460 kB
   Local memory size:                   32 kB
   Max work group size:                 256
   Max work item sizes:                 (256, 256, 256)
…
```

## QT双屏异显

### demo介绍

该demo目的在于演示在Ubuntu系统使用QT应用实现双屏显示和操作。
### 进入桌面环境
``` shell
export XAUTHORITY=/home/firefly/.Xauthority
export DISPLAY=:0
```

### 设置环境变量
[QT环境搭建参考](ubuntu_manual.html#qt-jiao-cha-bian-yi-huan-jing-zhi-chi)
```shell
export QT_QPA_PLATFORM=xcb
export QT_QPA_EGLFS_INTEGRATION=XCB_EGL
```
### 运行demo
```
./firefly_arm64_qt5.12.2_18.04/demo/double_panel_demo
```
### demo代码目录
```c
firefly_arm64_qt5.12.2_18.04/example/double_panel_demo
```
### 代码编译
```c
cd example
qmake
make
```
### 添加自己的QT工程
1. 在example目录下添加用户自己的QT项目工程
2. 编辑example目录下gui.pro文件
3. 假设工程目录名为double_panel_demo,则在gui.pro文件中追加 SUBDIRS += double_panel_demo
4. qmake && make
### 运行效果
![](../../../rv1126_img/common/linux_ubuntu_double_panel.jpg)

## 屏幕键盘

官方的 Ubuntu 系统中自带屏幕键盘，可以在菜单栏中点击打开：
![](../../../rv1126_img/common/linux_ubuntu_onboard.jpg)


## 声音配置

设备一般都有2个或以上的音频设备。常见的是耳机和 HDMI 这两个设备的音频输出。下面是音频设置的例子，供用户参考。

### 从命令行指定音频设备

系统自带的音频文件在 `/usr/share/sound/alsa/` 目录中，播放前请先查看声卡设备：

```
root@firefly:~# aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: rockchiphdmi [rockchip,hdmi], device 0: fe400000.i2s-i2s-hifi i2s-hifi-0 [fe400000.i2s-i2s-hifi i2s-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: rockchiprk809co [rockchip,rk809-codec], device 0: fe410000.i2s-rk817-hifi rk817-hifi-0 [fe410000.i2s-rk817-hifi rk817-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

然后指定声卡播放音频，其中声卡 card1 的设备0对应的是耳机口，card0 的设备0对应的是 HDMI。一般播放音频命令是 `aplay -Dhw:0,0 Fornt_Center.wav`，但系统自带的音频文件是单声道的，所以为了防止播放失败，可以按照下面的命令进行播放：

```
#选择耳机口输出音频文件
root@firefly:/usr/share/sounds/alsa# aplay -Dplughw:1,0 Front_Center.wav
Playing WAVE 'Front_Center.wav' : Signed 16 bit Little Endian, Rate 48000 Hz, Mono

#选择 HDMI 输出音频文件
root@firefly:/usr/share/sounds/alsa# aplay -Dplughw:0,0 Front_Center.wav
Playing WAVE 'Front_Center.wav' : Signed 16 bit Little Endian, Rate 48000 Hz, Mono
```

### 在图形界面中选择音频设备

在图形界面中，播放准备好的音频文件，然后点击声音图标，打开 `Sound Setting`，选择到 `Configuration` 。可以看到两个声卡设备，例如设置为 HDMI 输出音频，则把 HDMI 的声卡设备选择为 `Output`，另一个声卡设置为 `Off`。（如果 HDMI 无声或者声音小，可以试试按 HDMI 屏上的物理按键调高音量）

![](../../../rv1126_img/common/linux_ubuntu_sound_setting.jpg)


### 内核配置

在内核目录下，打开内核配置选项菜单：

```
make firefly_linux_defconfig
make menuconfig
```

进入内核配置菜单后依次选择：`Device Drivers` -> `USB Support` -> `USB Gadget Support`。

在 `USB Gadget Driver` 选项中选择到 `USB functions configurable through configfs`。

同时再选择上 `Function filesystem (FunctionFS)` 。

```
<*>   USB Gadget Drivers (USB functions configurable through configfs)  --->
        USB functions configurable through configfs
[*]       Function filesystem (FunctionFS)
```

接着在 `kernel` 目录下编译内核：

```
make rk3588-firefly.img -j12
```

编译完成后将内核烧录到设备中,烧录过程请参考维基教程：[升级固件](upgrade_firmware.md)中的分区镜像烧写部分。

### ADB 连接

安装了 ADB 后，用 Micro USB OTG 线连接设备和 PC 机。然后通过命令 `adb devices` 查看是否有设备连接:

```
firefly@Desktop:~$ adb devices
List of devices attached
0123456789ABCDEF	device
```

从返回的信息可以看到已经查找到了设备，表明设备已经成功连接。

设备连接成功后，输入命令 `adb shell` 即可进入命令行模式：

```
firefly@Desktop:~$ adb shell
#
```

此状态下是看不到命令行当前的路径，同时 Tab 键补全功能失效，需要进入一个用户中才可以正常操作。

```
firefly@Desktop:~$ adb shell
# su firefly
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

firefly@firefly:/$
#到这里命令行就可以正常使用了
```

用户也可以使用命令 `adb shell /bin/bash`，也可以进入正常的命令行模式。


可以输入 `adb help` 查看命令行帮助信息,注意并不是所有命令都可以使用，帮助信息只做参考。

## 串口

设备上的串口分两种，一种是普通串口另外一种是通过转换芯片把SPI转换而成的串口。转换而成的串口和普通串口功能完全一致但是需要注意它们的设备文件名是不一样的。下面是两者的区别:

```
//普通串口
root@firefly:~# ls /dev/ttyS*
ttyS0  ttyS1  ttyS2  ttyS3

//SPI转串口
root@firefly:~# ls /dev/ttysWK*
ttysWK0  ttysWK1  ttysWK2  ttysWK3
```

### 设置波特率

以 `ttyS4` 为例，查看串口波特率命令：

```
root@firefly:~# stty -F /dev/ttyS4
speed 9600 baud; line = 0;
-brkint -imaxbel
```

设置波特率命令：

```
//设置波特率为115200
root@firefly:~# stty -F /dev/ttyS4 ospeed 115200 ispeed 115200 cs8

//查看确认是否已经修改
root@firefly:~# stty -F /dev/ttyS4
speed 115200 baud; line = 0;
-brkint -imaxbel
root@firefly:~#
```

### 关闭回显

在做环回收发测试的时候回显功能会影响到我们的测试结果，所以在环回测试或者其他特殊情况下需要关闭回显。以下是关闭回显的命令：

```
//关闭回显示
root@firefly:~# stty -F /dev/ttyS4 -echo -echoe -echok

//查看所有功能的配置，检查是否已经关闭。“-”号代表该功能已经关闭
root@firefly:~# stty -F /dev/ttyS4 -a | grep echo
isig icanon iexten -echo -echoe -echok -echonl -noflsh -xcase -tostop -echoprt
echoctl echoke -flusho -extproc
```

### 发送接收原始数据

在实际应用中实际发送数据和我们希望发送的数据可能存在差别，例如多了回车或者其他特殊字符。利用 `stty` 可以把发送接收设置成 `raw` 模式确保发送接收的是原始数据，以下是设置命令：

```
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

```
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

硬件流控需要有硬件支持，设备的串口 `CTX` 和 `RTX` 脚需要和远端设备相连。

**注意：** 设备不是所有串口都支持硬件流控，请先从原理图上确认硬件是否支持

#### 设备树文件配置

```
uart3: serial@ff1b0000 {
        compatible = "rockchip,rk3588-uart", "snps,dw-apb-uart";
        reg = <0x0 0xff1b0000 0x0 0x100>;
        clocks = <&cru SCLK_UART3>, <&cru PCLK_UART3>;
        clock-names = "baudclk", "apb_pclk";
        interrupts = <GIC_SPI 101 IRQ_TYPE_LEVEL_HIGH 0>;
        reg-shift = <2>;
        reg-io-width = <4>;
        pinctrl-names = "default";
+       pinctrl-0 = <&uart3_xfer &uart3_cts &uart3_rts>;
        status = "disabled";
};
```

#### 应用层设置

上层也需要打开流控的设置，这里介绍 stty 如何打开流控如果你使用的是其他应用程序请从应用中打开流控。

```
//打开流控
root@firefly:~# stty -F /dev/ttyS4 crtscts

//检测流控是否打开,“-”号代表该功能已经关闭
root@firefly:~# stty -F /dev/ttyS4 -a | grep crtscts
-parenb -parodd -cmspar cs8 hupcl -cstopb cread clocal crtscts
```

 <!--以下功能未测试-->
