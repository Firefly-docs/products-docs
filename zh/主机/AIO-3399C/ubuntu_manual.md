# Firefly Ubuntu 使用手册

本使用手册适用与Firefly Ubuntu Desktop & Minimal 系统，部分与UI显示相关的介绍，只针对Desktop系统。

**强烈建议使用Ubuntu 18.04, 如使用其它版本系统，本文档可能部分章节不能完全适用。**

* [《创建 Ubuntu 根文件系统》](linux_build_ubuntu_rootfs.md)
* [《编译 Ubuntu 固件(GPt)》](linux_compile_gpt.md)

## adb 使用

### ADB
AIO-3399C用Type-C数据线连接设备和主机;

```
adb devices
adb shell
```

### 网络ADB
查看开发板 IP 地址，PC 端通过网络访问：
```
adb connect + IP
adb shell
```


## 安装linux-headers和linux-image
DEBIAN包下载链接:
链接：[https://pan.baidu.com/s/1HH_1M1QqdQQAVIXp3NnPvw](https://pan.baidu.com/s/1HH_1M1QqdQQAVIXp3NnPvw)
取码：ddd6

安装头文件
```
sudo dpkg -i linux-headers-4.4.194+_4.4.194+-5_arm64.deb
sudo dpkg -i linux-image-4.4.194+_4.4.194+-5_arm64.deb

cd /usr/src/linux-headers-4.4.194+
make headers_check
make headers_install

#make scripts 可能会出错但是不要紧
make scripts
```

## Qt交叉编译环境支持
Firefly发布的Qt交叉编译工具链，适用于以下环境:
- Host:  x86-64 / Ubuntu 18.04
- Target:  Firefly RK3399 RK3328 PX30 / Ubuntu 18.04 Minimal&Desktop

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

## 更换设备开机和桌面LOGO

* 开机的LOGO
开机LOGO是存放在SDK/kernel里面的，修改完之后重新编译kernel
```
linux-sdk/kenel/logo.bmp
```

* 桌面的LOGO
更换后重启系统即可
```
/usr/share/lubuntu/
├── images
│   └── lubuntu-ff-logo.png             #桌面左上角的小图标
└── wallpapers
    └── firefly-default-wallpaper.png   #桌面壁纸
```

## 创建WIFI热点

硬件要求：
* 需要有无线网卡，且支持移动热点功能

点击桌面右上角`Ethernet Network`图标，选择`Edit Connection...`
![](../../../rk3399_img/Hostspot1.png)

选择`+`图标，Add a new connection
![](../../../rk3399_img/Hostspot2.png)

选择`Wi-Fi`，后点击`Create`
![](../../../rk3399_img/Hostspot3.png)

Wi-Fi设置：
* 设置SSID
* Mode：选择Hotspot
* Device: 选择无线网卡（wlan）
![](../../../rk3399_img/Hostspot4.png)

选择适当的加密方式：
![](../../../rk3399_img/Hostspot5.png)

点击`Save`**即刻完成热点创建**


## 设置静态IP （使用netplan）


使用netplan设置静态IP

* addresses
* gateway4：设置网关
* nameservers： DNS nameservers
* ethernets: 以太网配置项
* wifis: wifi配置项


```
root@firefly:~# vim /etc/netplan/netplan.yaml
root@firefly:~# netplan apply
```


netplan的内容如下:
```
network:
    ethernets:
        eth0:
            addresses: [168.168.4.3/24]
            gateway4: 168.168.0.1
            dhcp4: yes
            optional: true
    wifis:
        wlan0:
            addresses: [168.168.4.23/24]
            gateway4: 168.168.0.1
            dhcp4: yes
            dhcp6: no
            access-points:
            "ssid":
                password: "password"
    version: 2
```
or
```
network:
    ethernets:
        eth0:
            addresses: [168.168.4.3/24]
            gateway4: 168.168.0.1
            dhcp4: no
            optional: true
            nameservers:
              addresses: [202.96.128.166]
    wifis:
        wlan0:
            addresses: [168.168.4.23/24]
            gateway4: 168.168.0.1
            dhcp4: no
            dhcp6: no
            access-points:
            "ssid":
                password: "password"
    version: 2
```

netplan更多配置示例：[https://netplan.io/examples](https://netplan.io/examples)




## USB 以太网

USB 以太网，主要实现的是将设备的 OTG 接口做外设模式，模拟成一个网络接口，然后主机通过 USB 连接设备并通过设备访问互联网。以下是基于 Firefly-RK3399 设备进行的具体操作。

操作环境：

* Ubuntu 系统的 PC 机
* Firefly-RK3399 设备


### 内核设置

在内核目录下，打开内核配置选项菜单：

```
make firefly_linux_defconfig
make menuconfig
```

进入内核配置菜单后依次选择：`Device Drivers` -> `USB Support` -> `USB Gadget Support`。

将 `USB Gadget Driver` 设置成编译成模块，然后可以在下边找到 `Ethernet Gadget (with CDC Ethernet support)` 选项，同样选择编译成模块。同时要选择上 `RNDIS support`。

```
<M>   USB Gadget Drivers
<M>     USB functions configurable through configfs
<M>     Ethernet Gadget (with CDC Ethernet support)
[*]       RNDIS support (NEW)
```

接着在 `kernel` 目录下编译内核：

```
make rk3399-firefly.img -j12
```

编译完成后将内核烧录到设备中,烧录过程请参考维基教程：[升级固件](03-upgrade_firmware.md)中的分区镜像烧写部分。然后将内核目录下生成的下列模块复制到设备中：

* drivers/usb/gadget/function/u_ether.ko
* drivers/usb/gadget/function/usb_f_ecm_subset.ko
* drivers/usb/gadget/function/usb_f_ecm.ko
* drivers/usb/gadget/function/usb_f_rndis.ko
* drivers/usb/gadget/function/usb_f_eem.ko
* drivers/usb/gadget/legacy/g_ether.ko
* drivers/usb/gadget/libcomposite.ko

然后在设备上，依次加载上述模块：

```
insmod libcomposite.ko
insmod u_ether.ko
insmod usb_f_ecm_subset.ko
insmod usb_f_rndis.ko
insmod usb_f_ecm.ko
insmod usb_f_eem.ko
insmod g_ether.ko
```

**注意：** 要先加载 `libcomposite.ko` 和 `u_ether.ko`，后面的模块才可以加载进去。

### IP 地址设置

用数据线连接 PC 机和设备的 OTG 接口，在 PC 机中执行 `lsusb` 命令可以查看到 USB 以太网设备，即说明连接成功。

```
firefly@Desktop:~$ lsusb
Bus 002 Device 003: ID 09da:5814 A4Tech Co., Ltd.
Bus 002 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 005: ID 04f2:b2ea Chicony Electronics Co., Ltd Integrated Camera [ThinkPad]
Bus 001 Device 004: ID 0a5c:21e6 Broadcom Corp. BCM20702 Bluetooth 4.0 [ThinkPad]
Bus 001 Device 003: ID 147e:1002 Upek Biometric Touchchip/Touchstrip Fingerprint Sensor
Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 003: ID 0525:a4a2 Netchip Technology, Inc. Linux-USB Ethernet/RNDIS Gadget
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

输出信息中 ID 0525:a4a2 Netchip Technology, Inc. Linux-USB Ethernet/RNDIS Gadget 即为 USB 网卡设备。

设备插入网线，使设备可以连接外网。

* 在设备中 IP 的设置：

输入执行 `ifconfig -a` 命令，可以查看到以下信息：

```
root@firefly:~# ifconfig -a

# eth0 是有线网卡
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 168.168.100.48  netmask 255.255.0.0  broadcast 168.168.255.255
        inet6 fe80::1351:ae2f:442e:e436  prefixlen 64  scopeid 0x20<link>
        ether 8a:4f:c3:77:94:ac  txqueuelen 1000  (Ethernet)
        RX packets 9759  bytes 897943 (897.9 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 236  bytes 35172 (35.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 42  base 0x8000

        ...

# usb0 是虚拟的 usb 网卡
usb0: flags=4098<BROADCAST,MULTICAST>  mtu 1500
        ether 4a:81:b1:34:d2:ad  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

然后给 usb0 网卡自定义一个适当的 IP：

注意要设置 usb0 的 IP 和有线网卡 eth0 的 IP 不在同一网段！！

```
ifconfig usb0 192.168.1.101
```

* 在 PC 机中 IP 的设置：

```
# 先查看 USB 虚拟网卡
firefly@Desktop:~$ ifconfig
enp0s20u2i1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.2.90  netmask 255.255.255.0  broadcast 192.168.2.255
        inet6 fe80::871c:b87e:1327:7fd4  prefixlen 64  scopeid 0x20<link>
        ether 46:fe:6e:97:ee:a6  txqueuelen 1000  (以太网)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1  bytes 54 (54.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
...


# 设置 USB 网卡的 IP 地址和设备的 usb0 的 IP 地址在同一网段
firefly@Desktop:~$ sudo ifconfig enp0s20u2i1 192.168.1.100

#设置默认网关：要设置为设备 usb0 的 ip 地址，因为后面要通过 usb0 来进行流量的转发
firefly@Desktop:~$ sudo route add default gw 192.168.1.101
```

设置完设备和 PC 机的 IP 后，互相是可以 ping 通的，PC 机也可以用 `ssh` 命令登录到设备。

### 网络共享实现 PC 机上网

在设备上：首先打开 IPv4 的转发功能：

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

如果希望每次重启设备后都自动打开转发功能，请直接修改 `/etc/sysctl.conf` 文件的 `net.ipv4.ip_forward` 值为1。修改文件参数后要执行 `sysctl -p` 命令重新载入 `/etc/sysctl.conf` 文件，使 IPv4 转发功能生效。

添加规则进行流量转发：

```
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j SNAT --to-source 168.168.100.48
```

现在 PC 主机就可以访问网络了。如果 PC 机可以 ping 通设备的 usb0 和 eth0，但是无法上外网，则需要修改 PC 机的 DNS，在 `/etc/resolv.conf` 中做以下添加：

```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

配置过程中要注意以下几点：

* 对应好上述步骤中自己设备上的各个 IP 地址,注意设备上的 USB 虚拟网卡 IP 和有线网络 IP 不在同一网段;
* PC 机虚拟 USB 网卡的网关要设置为设备虚拟 USB 网卡的 IP 地址;
* 设备上的虚拟网卡 IP 地址、IP 转发功能、流量转发规则等设置会在设备重启后恢复，用户可以自行查找资料如何开机自动设置这些配置。

## 导出设备系统

此章节适用于:
当用户已经在一台设备上完成工作环境的部署，需要将当前环境完整导出，以批量部署到其它同平台设备上。用户也可以通过导出设备文件系统来备份当前的开发环境。

* [《导出设备系统》](export_dev_sf.md)

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
root@firefly:~# ffgo version
OS:      Ubuntu 18.04.4 LTS
MODEL:   Firefly-RK3399 Board (Linux Opensource)
FIREFLY: v2.03-1-g2ed565c
DATE:    20200617-1042
KERNEL:  Linux version 4.4.194-59119-g968ba5005831 (jincheng@jincheng-System-Product-Name) (local/firefly-linux/rk3399/firefly: 968ba5005831c5f69d68d95984451fd2a7c457c0) (gcc version 6.3.1 20170404 (Lina0
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

## 使用root登录系统界面

* autologin-user=root

```
root@firefly:~# cat /etc/lightdm/lightdm.conf.d/20-autologin.conf
[Seat:*]
user-session=Lubuntu
autologin-user=root
```


## 视频硬件编解码支持

RK3399集成的VPU具有优秀的视频编解码能力，Mpp是Rockchip为VPU提供的一套视频编解码的api, 并且基于mpp。Rockchip提供了一套gstreamer的编解码插件。用户可以根据自己的需求，基于gstreamer来做视频编解码的应用，或者直接调用mpp，来实现硬件的编解码加速。

系统提供了一个测试视频文件，位于/usr/local/test.mp4 ，测试文件为1080P, 24Fps, H264编码, Mp4格式。

可以通过一下几种方式，验证和开发视频编解码相关应用。

### Gstreamer

*   Ubuntu 16.04 下，gstreamer 1.12 已经安装在/opt/目录下。
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
$ ffmpeg -y -c:v h264_rkmpp -i test.mp4 -an  output.yuv
```

**特别注意：** FFmpeg h264_rkmpp 解码获得的是 AV_PIX_FMT_DRM_PRIME，也就是 DRM 帧数据，如果是基于drm显示，可以直接输出帧，否则，需要用 hwdownload 进行转换。

更多内容可以参考 [FFmpeg 官网](http://ffmpeg.org/about.html)。

### Mpp

*   Ubunut 系统下， mpp 相关dev包都已经安装到系统中。

    更多相关资料,可参考`linux-sdk/docs/Linux/Multimedia`下的相关文档

## OpenGL-ES

RK3399 支持 OpenGL ES1.1/2.0/3.0/3.1。

Firefly Ubuntu 已经提供了完整的OpenGL-ES支持。

-   测试命令

```shell
$ sudo test_glmark2_normal.sh
```

-   webGL支持

在Chromium浏览器中， 在地址栏输入：`chrome://gpu`可以查看chromium下硬件加速的支持。

Note:

1.  EGL 是用arm 平台上OpenGL针对x window  system的扩展，功能等效于x86下的glx库。

2.  由于Xorg使用的Driver modesettings 默认会加载libglx.so(禁用glx会导致某些通过检测glx环境的应用启动失败)， libglx.so会搜索系统中的dri实现库。但是rk3399 Xorg 2D加速是直接基于DRM实现， 并未实现dri库，所以启动过程中，libglx.so会报告如下的错误 。

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



## TensorFlow Lite

RK3399 支持神经网络的GPU加速方案LinuxNN， Firefly Ubuntu已经添加了LinuxNN的支持。

在opt/tensorflowbin/下，运行test.sh， 即可测试MobileNet 模型图像分类器的 Demo和MobileNet-SSD 模型的目标检测 Demo

```
firefly@firefly:/opt/tensorflowbin$ ./test.sh
Loaded model mobilenet_ssd.tflite
resolved reporter
nn version: 1.0.0
findAvailableDevices
filename:libarmnn-driver.so     d_info:40432     d_reclen:40s
[D][ArmnnDriver]: Register Service: armnn (version: 1.0.0)!
first invoked time: 1919.17 ms
invoked
average time: 108.4 ms
validCount: 26
car     @ (546, 501) (661, 586)
car     @ (1, 549) (51, 618)
person  @ (56, 501) (239, 854)
person  @ (332, 530) (368, 627)
person  @ (391, 541) (434, 652)
person  @ (418, 477) (538, 767)
person  @ (456, 487) (602, 764)
car     @ (589, 523) (858, 687)
person  @ (826, 463) (1034, 873)
bicycle @ (698, 644) (1128, 925)
write out.jpg succ!
```

## 屏幕键盘

官方的 Ubuntu 系统中自带屏幕键盘，可以在菜单栏中点击打开：
![](../../../rk3399_img/onboard.jpg)

## HDMI_IN 图像抓取

**这项功能仅仅能够在AIO-3399J板子上使用**
* 需要焊接 HDMI_IN 输入接口

手动编译需要注意：
* 默认情况下这个功能是关闭,需要在kernel仓库中执行以下操作：```git revert 8998afb927200ad42b27beacc22cf1e86bfca442```
* 确保存在这个补丁:
        [firefly: AIO-3399J: enable HDMI-IN function: The HDMI-IN function is enabled, but other camera and audio functions may not be used normally.](https://gitlab.com/firefly-linux/kernel/-/commit/30e174dc173fba89c9b84a2651ad46cbc68bb0ac)

**测试脚本 test_hdmiin-3399_new.sh**
```
#!/bin/bash

export DISPLAY=:0
export XAUTHORITY=/home/firefly/.Xauthority
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
export LD_LIBRARY_PATH=/usr/lib/gstreamer-1.0

WIDTH=1920
HEIGHT=1080
SINK=kmssink
#SINK=gtksink

echo autospawn = no > /home/firefly/.config/pulse/client.conf
pkill pulseaudio
sleep 1
amixer -c 1 sset 'IN1 Boost' 0
amixer -c 1 sset "RECMIXL BST1" on
amixer -c 1 sset "RECMIXR BST1" on
amixer -c 1 sset 'Mono ADC MIXR ADC1' on
amixer -c 1 sset 'Mono ADC MIXL ADC1' on

gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,format=NV12,width=${WIDTH},height=${HEIGHT}, framerate=30/1 ! videoconvert ! $SINK & # &>/dev/null &

GST_PID=$!

sleep 2
alsaloop -P hw:1,0 -C hw:1,0 -c 2 -t 500000 &
ALSA_PID=$!

trap '{ kill -9 $GST_PID; kill -9 $ALSA_PID; }' EXIT INT TERM

wait

```

## 声音配置

设备一般都有2个或以上的音频设备。常见的是耳机和 HDMI 这两个设备的音频输出。下面是音频设置的例子，供用户参考。

### 从命令行指定音频设备

系统自带的音频文件在 `/usr/share/sound/alsa/` 目录中，播放前请先查看声卡设备：

```
root@firefly:~# aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: rockchiprt5640c [rockchip,rt5640-codec], device 0: ff890000.i2s-rt5640-aif1 rt5640-aif1-0 []
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: rkhdmidpsound [rk-hdmi-dp-sound], device 0: HDMI-DP multicodec-0 []
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

然后指定声卡播放音频，其中声卡 card0 的设备0对应的是耳机口，card1 的设备0对应的是 HDMI。一般播放音频命令是 `aplay -Dhw:0,0 Fornt_Center.wav`，但系统自带的音频文件是单声道的，所以为了防止播放失败，可以按照下面的命令进行播放：

```
#选择耳机口输出音频文件
root@firefly:/usr/share/sounds/alsa# aplay -Dplughw:0,0 Front_Center.wav
Playing WAVE 'Front_Center.wav' : Signed 16 bit Little Endian, Rate 48000 Hz, Mono

#选择 HDMI 输出音频文件
root@firefly:/usr/share/sounds/alsa# aplay -Dplughw:1,0 Front_Center.wav
Playing WAVE 'Front_Center.wav' : Signed 16 bit Little Endian, Rate 48000 Hz, Mono
```

### 在图形界面中选择音频设备

在图形界面中，播放准备好的音频文件，然后点击声音图标，打开 `Sound Setting`，选择到 `Configuration` 。可以看到两个声卡设备，例如设置为 HDMI 输出音频，则把 HDMI 的声卡设备选择为 `Output`，另一个声卡设置为 `Off`。（如果 HDMI 无声或者声音小，可以试试按 HDMI 屏上的物理按键调高音量）

![](../../../rk3399_img/sound_setting.jpg)


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
make rk3399-firefly.img -j12
```

编译完成后将内核烧录到设备中,烧录过程请参考维基教程：[升级固件](03-upgrade_firmware.md)中的分区镜像烧写部分。

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
        compatible = "rockchip,rk3399-uart", "snps,dw-apb-uart";
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

## MIPI 摄像头 (OV13850)

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

cd rkisp-v4l2/
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

MIPI摄像头是**不支持**OpenCV中**capture.open(index)**方式读取的，在此提供2中读取摄像头的方法：

* 使用V4L2方法读取摄像头：
这种方式**推荐OpenCV的C++版本使用**，详细使用参考前面的v4l2预览一节。v4l2_simple_demo是rkisp_demo.cpp的精简版本，详细可参考rkisp_demo.cpp。

```
参考：
https://github.com/T-Firefly/rkisp-v4l2.git

git clone https://github.com/T-Firefly/rkisp-v4l2.git
cd firefly-rkisp-v4l2/v4l2_simple_demo
make
# open /dev/video0
sudo -u firefly DISPLAY=:0 ./opencv
```


## 外部存储设备 rootfs

根文件系统除了可以使用在内部的 eMMC 中的，还可以使用外部存储设备的根文件系统，如 SD 卡，U 盘等。以下是以 SD 卡为例，在 Firefly-RK3399 设备上实现挂载外部存储设备的根文件系统。


### 在 SD 卡上建立分区

在 PC 机上插入 SD 卡，用 `gdisk` 工具分出一个装载根文件系统的分区：

```
#用户请用 `fdisk -l` 查询 SD 卡设备名是什么，在用 gdisk 命令进入对应的设备
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

```
# 注意 sdb1 表示 SD 卡对应分区设备名，请用户根据实际情况填入
sudo mkfs.ext4 /dev/sdb1

```

格式化完成后，使用 `dd` 命令，将制作好的根文件系统烧写到 SD 卡刚新建的分区中。（根文件系统的制作可以参考[Ubuntu 根文件系统的制作](linux_build_ubuntu_rootfs.md)）

```
# 用户请根据实际情况填写根文件系统镜像的路径和 SD 卡对应分区设备名
dd conv=fsync,notrunc if=/rootfs_path/rootfs.img of=/dev/sdb1
```

### 挂载 SD 卡根文件系统

首先要修改设备树文件，打开对应的 dts 文件，在根节点下重写 `chosen` 节点下的 `bootargs` 参数的 `root` 值为写入了根文件系统的 SD 卡分区的 unique GUID：

```
#将 root 的值改为获取到的 unique GUID 值的前12位
chosen {
    bootargs = "earlycon=uart8250,mmio32,0xff1a0000 swiotlb=1 console=ttyFIQ0 rw root=PARTUUID=6278AF5D-BC6B  rootfstype=ext4 rootwait";
};

```

修改完成后编译并烧录至设备中。

SD 卡的根文件系统烧录完成后，插入设备的 TF 卡槽中，开机即可进入到 SD 的根文件系统中。


注意事项：
*   在操作前请注意备份 U 盘或者 SD 卡内的重要文件，避免丢失；
*   操作时请确认自己的 SD 卡对应的设备名；
*   dts 文件中 `root` 参数的值使用的是 unique GUID，记下前12位即可。用户也可以根据 `gdisk` 的帮助信息自行修改这个值。


## 网络启动

网络启动，是用 TFTP 在服务器下载内核、dtb 文件到目标机的内存中，同时可以用 NFS 挂载网络根文件系统到目标机上，实现目标机的无盘启动。以下基于 Firefly-RK3399 作出一个示例，提供用户参考。

准备工作：
* Firefly-RK3399 设备；
* 路由器、网线；
* 安装有 NFS 和 TFTP 的服务器；
* 一份制作好的根文件系统。

注：示例中使用的是 Ubuntu 系统的 PC 机作为服务器，通过路由器和网线实现与设备的连接。用户可以根据自己的实际情况作调整，但如果是 PC 机直连设备，请使用交叉网线。请确保服务器和目标机在同一局域网内。

### 服务器部署

1、在服务器上部署 TFTP 服务：

安装 TFTP 服务：

```
sudo apt-get  install  tftpd-hpa
```

创建 `/tftpboot` 目录并赋予权限：

```
mkdir /tftpboot
sudo chmod 777 /tftpboot
```

然后修改 TFTP 服务器的配置文件 `/etc/default/tftpd-hpa`，修改内容为：

```
# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/tftpboot"   #这个根据用户实际的 tftp 目录设置
TFTP_ADDRESS="0.0.0.0:69"
TDTP_OPTIONS="-c -s -l"
```

退出修改后重启 TFTP 服务器：

```
sudo service tftpd-hpa restart
```

在 `/tftpboot` 目录中创建一个文件，测试 TFTP 服务是否可用：

```
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

```
sudo apt-get install nfs-kernel-server
```

创建共享目录：

```
sudo mkdir /nfs
sudo chmod 777 /nfs
cd /nfs
sudo mkdir rootfs
sudo chmod 777 rootfs
```

然后将制作好的根文件系统复制到 `/nfs/rootfs` 目录中。（根文件系统的制作可以参考：[Ubuntu 根文件系统的制作](linux_build_ubuntu_rootfs.md)）

接着配置 NFS，在 `/etc/exports` 中添加共享目录位置：

```
/nfs/rootfs *(rw,sync,no_root_squash,no_subtree_check)
```

共享的目录根据用户实际情况设置，其中的"*"代表的是所有用户可访问。

重启 NFS 服务器：

```
sudo /etc/init.d/nfs-kernel-server restart
```

本地挂载共享目录，测试 NFS 服务器是否可用：

```
sudo mount -t nfs 127.0.0.1:/nfs/rootfs/ /mnt
```

在 `/mnt` 目录下查看到了与 `/nfs/rootfs` 一致的内容，则说明 NFS 服务器部署成功。然后可以解除挂载：

```
sudo umount /mnt
```

### 内核的配置

如果要做到挂载网络根文件系统，需要在内核中做相关配置，并在 dts 中修改相关挂载根文件系统的配置。

首先进行内核配置，在内核目录中执行 `make menuconfig`，选择相关配置：

```
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

修改 `rk3399-firefly.dts` 配置，在 dts 文件中修改 `chosen` 节点下的 `bootargs` 参数，选择使用 NFS 挂载远程根文件系统，内容如下。

```
chosen {
    bootargs = "earlycon=uart8250,mmio32,0xff1a0000 swiotlb=1 console=ttyFIQ0 root=/dev/nfs rootfstype=ext4 rootwait";
};

#主要是修改 root 的值，root=/dev/nfs 表示通过 NFS 挂载网络根文件系统
```

编译内核：

```
make ARCH=arm64 rk3399-firefly.img -j12
```

编译完成后，将编译好的内核文件 `boot.img` 和 `rk3399-firefly.dtb` 文件复制到 `/tftpboot` 目录中：

```
cp boot.img /tftpboot
cp arch/arm64/boot/dts/rockchip/rk3399-firefly.dtb /tftpboot
```

详细说明可以参考内核目录中的 `kernel/Documentation/filesystems/nfs/nfsroot.txt`


### U-Boot设置

请先确保目标机网线已插入，接入到服务器的局域网内。

目标机启动进入 U-Boot 命令行模式，设置以下参数：

```
=> setenv ipaddr 192.168.31.101     #设置目标机 IP 地址
=> setenv serverip 192.168.31.106   #设置 serverip 为服务器 IP 地址

#设置从 TFTP 下载内核和 dtb 文件到相应地址，用户请根据自己实际的目标机修改相应地址
=> setenv bootcmd tftpboot 0x0027f800 boot.img \; tftpboot 0x08300000 rk3399-firefly.dtb \; bootm 0x0027f800 - 0x08300000

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
Filename 'boot.img'.
Load address: 0x27f800
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
Filename 'rk3399-firefly.dtb'.
Load address: 0x8300000
Loading: #######
         645.5 KiB/s
done
Bytes transferred = 97212 (17bbc hex)
## Booting Android Image at 0x0027f800 ...
Kernel load addr 0x00280000 size 19377 KiB
## Flattened Device Tree blob at 08300000
   Booting using the fdt blob at 0x8300000
   XIP Kernel Image ... OK
   Loading Device Tree to 0000000073edc000, end 0000000073ef6bbb ... OK
Adding bank: 0x00200000 - 0x08400000 (size: 0x08200000)
Adding bank: 0x0a200000 - 0x80000000 (size: 0x75e00000)
Total: 912260.463 ms

Starting kernel ...

...
```

在开机内核日志中可见：

```
[   12.146297] VFS: Mounted root (nfs filesystem) on device 0:16.
```

说明已经挂载上了网络根文件系统。

注意事项

*   确保 TFTP 服务器、NFS 服务器可用;
*   确保目标机先插入网线后在开机，且和服务器在同一局域网内，如果是直连目标机和服务器，请使用交叉网线;
*   内核配置中，`Root file system on NFS` 依赖于 `IP: kernel level autoconfiguration` 选项，请先选择 `IP: kernel level autoconfiguration`，之后才可以找到 `Root file system on NFS` 选项;
*   在 U-Boot 命令行中，请确认 `boot.img` 烧录地址和 dtb 文件烧录地址。（提示：`boot.img` 的文件结构中，开头有2k的头文件，然后才是 kernel。所以在 TFTP 下载内核到目标机时，要下载到对应 kernel 地址减去0x0800的地址上）;
*   在设置挂载远程根文件系统时，`nfsroot=192.168.31.106:/nfs/rootfs,v3` 中的 `v3` 代表 NFS 版本信息，请添加上以避免出现挂载不成功的问题。

## 在线更新内核和 U-Boot

本小节介绍了在线更新的一个简单的流程。将内核、U-Boot 或者其他需要更新的文件打包成 deb 安装包，然后导入到本地包仓库，实现在设备上下载并自动更新。仅供用户参考。



### 准备 deb 安装包

操作中需要升级内核和 U-Boot，事先已经准备好了修改好的相关文件：`uboot.img` 、`trust.img` 、`boot.img` 。

deb 是 Debian Linux 的软件包格式，打包最关键的是在 `DEBIAN` 目录下创建一个 `control` 文件。首先创建 deb 工作目录，然后在 deb 目录中创建相应的目录和文件：

```
mkdir deb
cd deb
mkdir firefly-firmware    #创建 firefly-firmware 目录
cd firefly-firmware
mkdir DEBIAN    #创建 DEBIAN 目录，这个目录是必须要有的。
mkdir -p usr/share/{kernel,uboot}
mv ~/boot.img ~/deb/firefly-firmware/usr/share/kernel  #将相应文件放到相应的目录
mv ~/uboot.img ~/deb/firefly-firmware/usr/share/uboot
mv ~/trust.img ~/deb/firefly-firmware/usr/share/uboot
```

`DEBIAN` 目录下存放的文件是 deb 包安装的控制文件以及相应的脚本文件。

在 `DEBIAN` 目录下创建控制文件 `control` 和脚本文件 `postinst`。

`control` 文件内容如下,用于记录软件标识，版本号，平台，依赖信息等数据。

```
Package: firefly-firmware #文件目录名
Version: 1.0    #版本号
Architecture: arm64 #架构
Maintainer: neg   #维护人员，自定义即可
Installed-Size: 1
Section: test
Priority: optional
Descriptionon: This is a deb test
```

`postinst` 文件内容如下,就是将需要更新的内核和 U-Boot 文件用 `dd` 命令写进对应分区的脚本：

```
echo "-----------uboot updating------------"
dd conv=fsync,notrunc if=/usr/share/uboot/uboot.img of=/dev/disk/by-partlabel/uboot

echo "-----------trust updating------------"
dd conv=fsync,notrunc if=/usr/share/uboot/trust.img of=/dev/disk/by-partlabel/trust

echo "-----------kernel updating------------"
dd conv=fsync,notrunc if=/usr/share/kernel/boot.img of=/dev/disk/by-partlabel/boot
```

说明：`postinst` 脚本，是在解包数据后运行的脚本，对应的还有:

* preinst：在解包数据前运行的脚本;
* prerm：卸载时，在删除文件之前运行的脚本;
* postrm：在删除文件之后运行的脚本;

其他相关知识点用户可以自行了解。这里只用到了 `preinst` 脚本。

以下是创建好的目录树：

```
deb
└── firefly-firmware
    ├── DEBIAN
    │   ├── control
    │   └── postinst
    └── usr
        └── share
            ├── kernel
            │   └── boot.img
            └── uboot
                ├── trust.img
                └── uboot.img
```

进入 deb 目录，用 `dpkg` 命令生成 deb 包：

```
dpkg -b firefly-firmware firefly-firmware_1.0_arm64.deb
```

生成 deb 包时，一般按照这种规范进行命名：`package_version-reversion_arch.deb` 。

### 创建本地仓库

首先安装需要的包：

```
sudo apt-get install reprepro gnupg
```

然后用 GnuPG 工具生成一个 GPG 密匙，执行命令后请根据提示操作：

```
gpg --gen-key
```

执行 `sudo gpg --list-keys` 可以查看刚刚生成的密匙信息:

```
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

```
cd /var/www
mkdir apt   #包仓库目录
mkdir -p ./apt/incoming
mkdir -p ./apt/conf
mkdir -p ./apt/key
```

把前面生成的密匙导出到仓库文件夹，请用户对应好自己创建的用户名和邮箱地址。

```
gpg --armor --export firefly firefly@t-chip.com > /var/www/apt/key/deb.gpg.key
```

在 `conf` 目录下创建 `distributions` 文件，其内容如下：

```
Origin: Neg   #你的名字
Label: Mian     #库的名字
Suite: stable   #(stable 或 unstable)
Codename: bionic    #发布代码名，可自定义
Version: 1.0    #发布版本
Architectures: arm64    #架构
Components: main    #组件名，如main，universe等
Description: Deb source test        #相关描述
SignWith: BCB65788541D632C057E696B8CBC526C05417B76  #上面步骤中生成的 GPG 密匙
```

建立仓库树：

```
reprepro --ask-passphrase -Vb /var/www/apt export
```

将第2步做好的 deb 包加入到仓库中：

```
reprepro --ask-passphrase -Vb /var/www/apt includedeb bionic ~/deb/firefly-firmware_1.0_arm64.deb
```

可以查看库中添加的文件：

```
root@Desktop:~# reprepro -b /var/www/apt/ list bionic
bionic|main|arm64: firefly-firmware 1.0
```

你的包已经加入了仓库，如果要移除它的话采用如下命令:

```
reprepro --ask-passphrase -Vb /var/www/apt remove bionic firefly-firmware
```

安装 nginx 服务器：

```
sudo apt-get install nginx
```

修改nginx的配置文件 `/etc/nginx/sites-available/default` 为：

```
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

```
sudo service nginx restart
```

### 客户端更新安装

在客户端设备中，首先要添加本地包仓库的源，在目录 `/etc/apt/sources.list.d` 下添加一个新的配置文件 `bionic.list`，内容如下：

```
deb http://192.168.31.106 bionic main
```

IP 地址是服务器地址, `bionic` 是仓库发布代码名， `main` 是组件名。

从服务器中获取并添加 GPG 密匙：

```
wget -O - http://192.168.31.106/key/deb.gpg.key | apt-key add -
```

更新后即可安装自定义软件源里的 `firefly-firmware_1.0_arm64` 包：

```
root@firefly:/home/firefly# apt-get update
Hit:1 http://192.168.31.106 bionic InRelease
Hit:2 http://wiki.t-firefly.com/firefly-rk3399-repo bionic InRelease
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
Preparing to unpack .../firefly-firmware_1.0_arm64.deb ...
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

可以看到安装过程中执行了 deb 包中的 `poinst` 脚本。安装完成后，重启设备，更新完成。

在 `/usr/share` 目录中还可以看到 `kernel` 和 `uboot` 目录，分别存放了 `boot.img`，`uboot.img` 和 `trust.img` 。

注意事项

* 在制作 deb 包中，与 `DEBIAN` 同级的目录视为根目录，即放在与 `DEBIAN` 同目录下的文件，在客户端安装 deb 包后，可以在客户端的根目录下找到；
* deb 包中的文件和脚本，用户要根据自己的实际情况做调整；
* 每次在仓库中修改了配置文件后，都要重新导入仓库目录树 ；
* nginx 服务器配置中，`root` 参数配置的是仓库的地址，请用户根据自己的实际情况修改；
* 客户端添加的新下载源的文件时，注意核对正确的服务器地址，包仓库代码名和组件名。注意客户端要连通服务器；
* 客户端要用 `apt-key add` 命令添加 GPG 密匙之后，才可以获取本地的仓库的信息。

## 用Docker搭建分布式编译环境

`distcc` 是个通过网络上的若干台机器用来分布式编译 C、C++、Objective-C 或 Objective-C++ 代码的程序。`distcc` 不要求所有的机器共享文件系统，有同步的时钟，或者安装有同样的库和头文件，只要作为服务器的机器有合适的编译工具即可进行编译。本例在两块 Firefly-RK3399 设备 (arm64) 和一台 PC 机 (x86_64) 上利用 Docker 技术来布署 distcc 分布式编译服务，进而实现在其中一块 Firefly-RK3399 设备上利用 distcc 的分布式编译特性来加速内核的编译过程。

### 准备工作

*   两块 Firefly-RK3399 设备；
*   路由器、网线；
*   PC 机。

将两块设备和 PC 机都连接到同一个局域网内。连接好后对应的 IP 地址别是：

*   PC 机：192.168.1.100
*   设备 A：192.168.1.101
*   设备 B：192.168.1.102

### PC 机部署 Docker

使用脚本安装 Docker ：

```
wget -qO- https://get.docker.com/ | sh
```

为了可以使当前普通用户可以执行 docker 相关命令，需要将当前用户添加到 dokcer 用户组中：

```
sudo groupadd docker            #添加 docker 用户组
sudo gpasswd -a $USER docker    #将当前用户添加到 docker 用户组中
newgrp docker                   #更新 docker 用户组
```

启动 Docker 后台服务：

```
sudo service docker start
```

新建一个 `Dockerfile_distcc.x86_64` 文件。其文件内容如下：

```
FROM ubuntu:bionic
MAINTAINER Firefly <service@t-firefly.com>

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests\
	gcc-aarch64-linux-gnu distcc\
    && apt-get clean \
    && rm -rf /var/lib/apt/lists

RUN ln -s aarch64-linux-gnu-gcc /usr/bin/gcc &&\
ln -s aarch64-linux-gnu-gcc /usr/bin/cc

EXPOSE 3632

ENTRYPOINT ["/usr/bin/distccd"]
CMD ["--no-detach" , "--log-stderr" , "--log-level", "debug", "--allow" , "0.0.0.0/0"]
```

生成 Docker 镜像：

```
docker build -t distcc_server:x86_64 -f Dockerfile_distcc.x86_64 .
```

可以用命令 `docker images` 查看生成的 Docker 镜像：

```
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
distcc_server       x86_64              138c0b7e3801        9 minutes ago       66.1MB
```

用新建的镜像启动 Docker 容器，在主机网络的 3632 TCP 端口提供 distcc 服务：

```
docker run -d -p 3632:3632 distcc_server:x86_64
```

用命令 `docker ps` 可以查看正在运行中的容器：

```
docker ps
CONTAINER ID    IMAGE                  COMMAND                  CREATED         STATUS          PORTS                    NAMES
fa468d068185    distcc_server:x86_64   "/usr/bin/distccd --…"   9 minutes ago   Up 9 minutes    0.0.0.0:3632->3632/tcp   epic_chatterjee
```

### 设备部署 Docker

```
apt-get update
wget -qO- https://get.docker.com/ | sh
```

将当前用户添加到 dokcer 用户组中：

```
sudo groupadd docker
sudo gpasswd -a $USER docker
newgrp docker	#更新 docker
```

启动 Docker 后台服务：

```
service docker start
```

新建 `Dockerfile_distcc.arm64` 文件，内容如下：

```
FROM ubuntu:bionic
MAINTAINER Firefly <service@t-firefly.com>

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
	gcc distcc\
    && apt-get clean \
    && rm -rf /var/lib/apt/lists

EXPOSE 3632

ENTRYPOINT ["/usr/bin/distccd"]
CMD ["--no-detach" , "--log-stderr" , "--log-level", "debug", "--allow" , "0.0.0.0/0"]
```

生成 Docker 镜像：

```
docker build -t distcc_server:arm64 -f Dockerfile_distcc.arm64 .
```

用新建的镜像启动 Docker 容器，在主机网络的 3632 TCP 端口提供 distcc 服务：

```
docker run -d -p 3632:3632 distcc_server:arm64
```

用 `docker save` 命令导出新建的镜像：

```
docker save -o distcc_server.tar distcc_server:arm64
```

将生成的 `distcc_server.tar` 文件复制到另一块设备上，然后导入镜像：

```
docker load -i distcc_server.tar
```

导入镜像后，在这个设备上也有了 `distcc_server:arm64` 镜像，然后可以运行一个容器：

```
docker run -d -p 3632:3632 distcc_server:arm64
```

**提示：** 用户如果注册了 Docker Hub 帐号，可以用 `docker push` 推送镜像到远程仓库，在另一设备上用 `docker pull` 拉取远程仓库的镜像。用户可以自行了解相关操作。

### 客户端用 distcc 编译内核

到这里，三个机器都部署了分布式编译环境。可以选择任意一台机器作为客户端，剩下两个机器作为服务器。这里选择其中一个设备作为客户端。

在客户端中创建 `Dockerfile_compile.arm64` 文件，生成一个编译内核的 Docker 镜像，文件内容如下：

```
FROM ubuntu:bionic
MAINTAINER Firefly <service@t-firefly.com>

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get install -y --no-install-recommends --no-install-suggests \
	bc make python sed libssl-dev binutils build-essential distcc\
    liblz4-tool gcc \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists
```

然后生成镜像：

```
docker build -t compile:arm64 -f Dockerfile_compile.arm64 .
```

在启动容器前先将内核文件拉到客户端中。然后创建 `/etc/distcc/hosts` 文件，其内容是列举出所有提供 distcc 服务的机器的 IP 地址。内容如下：

```
# As described in the distcc manpage, this file can be used for a global
# list of available distcc hosts.
#
# The list from this file will only be used, if neither the
# environment variable DISTCC_HOSTS, nor the file $HOME/.distcc/hosts
# contains a valid list of hosts.
#
# Add a list of hostnames in one line, seperated by spaces, here.
192.168.1.100
192.168.1.101
193.168.1.102
```

为了测试结果更准确，先清除客户端设备上的缓存：

```
echo 3 > /proc/sys/vm/drop_caches
```

用 `compile:arm64` 镜像启动一个 Docker 容器，同时将主机当前内核目录挂载到容器中的 `/mnt` 目录，将主机的 `/etc/distcc/hosts` 文件挂载到容器的 `/etc/distcc/hosts` 中，并用参数 `-it` 以交互模式启动容器：

```
docker run -it --rm -v $(pwd):/mnt -v /etc/distcc/hosts:/etc/distcc/hosts compile:arm64 /bin/bash
root@f4415264351b:/# ls
bin  boot  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@f4415264351b:/# cd mnt/
```

进入到容器中的 `/mnt` 目录，然后开始用 distcc 编译内核，加入 `time` 命令可以查看编译命令执行耗时，`CC` 参数指明用 `distcc` 进行编译：

```
time make ARCH=arm64 rk3399-firefly.img -j32 CC="distcc"
```

**注意：** 如果用 PC 机作为客户端，则需要用以下命令进行编译：

```
time make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- rk3399-firefly.img -j32 CC="distcc aarch64-linux-gnu-gcc"
```

编译过程中，可以在用于编译的容器内新的窗口中用命令 `distccmon-text 1` 查看编译情况：

```
distccmon-text 1
                    ...
15713  Compile     perf_regs.c                              192.168.1.101[0]
15327  Compile     fork.c                                   192.168.1.101[2]
15119  Compile     dma-mapping.c                            192.168.1.101[3]
15552  Compile     signal32.c                               192.168.1.100[0]
15644  Compile     open.c                                   192.168.1.100[2]
15112  Compile     traps.c                                  192.168.1.100[3]
15670  Compile     arm64ksyms.c                             192.168.1.102[0]
15629  Compile     mempool.c                                192.168.1.102[2]
15606  Compile     filemap.c                                192.168.1.102[3]
15771  Preprocess                                                localhost[0]
15573  Preprocess                                                localhost[1]
15485  Preprocess                                                localhost[2]
                    ...
```

最后编译命令完成后可以看到编译所用时间：

```
real    15m44.809s
user    16m0.029s
sys     6m22.317s
```

下边是单独使用一块设备进行内核编译所耗费的时间:

```
real    23m33.002s
user    113m2.615s
sys     9m29.348s
```

对比可见，用采用 distcc 实现的分布式编译可以有效提高编译速度。

**注意：**

* 平台不同，所需的编译器也不同。如在 x86_64 平台上，需要安装对应的交叉编译工具 `gcc-aarch64-linux-gnu`。在 arm64 平台则只需要安装 `gcc` 编译工具即可。用户需要根据实际情况在对应的 `Dockerfile` 文件里指定安装正确的工具。
* 如果用 PC 机做客户端，则在用于编译的容器中编译内核的时候，要用参数 `CC="distcc aarch64-linux-gnu-gcc"` 指明用 `aarch64-linux-gnu-gcc` 交叉编译工具编译。


## Android 7.1 升级到 Ubuntu

**此升级方法仅适用于 RK3399，且板子上当前运行的固件为 Android 7.1**

如果 RK3399 板出厂的固件是 Android 7.1，在使用 TF 卡升级到 Ubuntu 时候，由于 Maskrom 中 eMMC 启动优先级高于 TF 卡，所以会先启动 eMMC 中的 miniloader，然后由 miniloader 再启动 TF 卡中的 U-Boot。但是 Android 7.1 的 miniloader 与 Ubuntu 的 U-Boot 不兼容，会导致出错。解决方案是：使用 Android 7.1 的 Bootloader、用户 Ubuntu 固件的内核及定制的 recovery 内存盘来制作 Ubuntu 的升级卡， 成功实现启动链 “MaskRom -> eMMC Android miniloader -> TF Android U-Boot -> TF Ubuntu Kernel -> TF recovery 系统” 的正常工作。

### 材料准备

1. 一张 TF 卡。
2. 待进行系统升级的设备。
3. 对应板型的 Ubuntu 固件。
4. 下载操作所需的相关文件和脚本：[ubuntu_sd_update.zip]。

### 升级卡的制作流程

1. 解压 `ubuntu_sd_update.zip`，进入 `ubuntu_sd_update` 目录；
2. 拷贝板型对应的 Ubuntu 固件到本目录下，重命名为 `update.img`（或者 `ln -sf xxx update.img` ）；
3. 执行 `./unpack.sh` 解包 Ubuntu 固件；
4. 然后执行 `./mkSDCreate.sh`，生成 `sdCreate.img`；
5. 使用 [SD_Firmware_Tool] 工具，选择 `sdCreate.img`，完成升级卡的制作；
6. 拷贝 Ubuntu 固件到 TF 卡中，并重命名为 `sdupdate.img`。

### 设备升级流程

1. 对设备断电。
2. 插入 TF 卡，上电开机，设备自动进入升级操作，期间机器的一个 LED 灯会持续闪烁，视固件大小，升级时间较长。
3. 升级完成，LED 灯停止闪烁。
4. 拔出 TF 卡，对开板重新上电。

### 特殊情况

* 在实际应用中，有些用户的升级固件大小超过 4G，由于 [SD_Firmware_Tool] 制作完升级卡之后，是以 FAT32 来格式化 TF 卡的，导致用户固件不能放入 TF 卡。此时，在制作升级卡完成后，先将 TF 卡内 `rksdfw.tag` 和 `sd_boot_config.config` 两个文件备份，然后在 Windows 上将 TF 卡的 FAT32 分区重新格式化为 NTFS 或者 exFAT，然后再将备份出来的文件和用户固件拷贝到 TF 卡即可。

* 如果出现插卡开机后设备进入了 recovery 模式却没有自动进行升级操作，请在 recovery 模式下重新拔插 TF 卡，升级操作即可正常继续进行。

[ubuntu_sd_update.zip]: https://download.t-firefly.com/product/RK3399/Tools/ubuntu_sd_update/ubuntu_sd_update.zip
[SD_Firmware_Tool]: http://www.t-firefly.com/doc/download/page/id/54.html#other_401