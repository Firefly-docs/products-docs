# Firefly Ubuntu User Manual

This user manual is applicable to Firefly Ubuntu Desktop & Minimal system. Some introductions related to UI display are only for Desktop system.

## Compile the universal device bus driver locally

DEBIAN package download link: [linux-headers](https://en.t-firefly.com/doc/download/377.html#other_413)

Install header files:

```shell
sudo dpkg -i linux-headers-4.19.172_4.19.172-189_arm64.deb
sudo dpkg -i linux-image-4.19.172_4.19.172-189_arm64.deb

make headers_check
make headers_install

# make scripts may go wrong but it doesn't matter
make scripts
```

## Support Docker
Firefly normal firmware generally does not meet Docker operation requirements. If there is a need, you can use SDK to enable related kernel configs and rebuild the kernel to support Docker. How to get and use SDK please refer to Compile Ubuntu firmware. After pre-compile configuration, here is how to support Docker:
### Check Kernel Configuration
First use script to see which configuration is needed by Docker but missing in current kernel. You can get the check script from [GitHub](https://github.com/moby/moby/blob/master/contrib/check-config.sh).

After get the script, begin to check:
```bash
#Copy the script to SDK/kernel/
cp check-config.sh PathToSDK/kernel/
cd PathToSDK/kernel
chmod +x check-config.sh

#Get current kernel configuration
make ARCH=arm64 firefly_linux_defconfig

#Check
./check-config.sh .config
```
The result looks like this, mainly two parts:
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
Generally Necessary: All features in this part need to be enabled in kernel.

Optional Features: Enable them as you needed.
### Enable Needed Features
From the result you can find out which features are need to be enabled, then use `make ARCH=arm64 menuconfig` to enter config menu, search and enable them. Please read the instructions in the config menu, if some features can't be selected, check the dependency.

Enabled all the necessary features and some optional features, remember to save:
```bash
make ARCH=arm64 savedefconfig
mv defconfig arch/arm64/configs/firefly_linux_defconfig
```
Compile the kernel:
```
#Back to SDK dir
cd ..
#Compile
./build.sh kernel
```
### Install Docker
After burning new kernel, install Docker on device to verify:
```bash
#Here only introduces the quick-install using script
apt-get update
wget -qO- https://get.docker.com/ | sh
```
After successful installation, you will see the Docker version, then run a demo:
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

## Qt cross compilation environment support
The Qt cross-compilation tool chain released by Firefly is suitable for the following environments:
* Host: x86-64 / Ubuntu 18.04
* Target: Firefly RV1126B RK3399 RK3328 PX30 / Ubuntu 18.04 Minimal&Desktop

The tool chain fully supports wenEngine and backends such as EGLFS LinuxFB XCB.

* download link
```
Link: https://drive.google.com/drive/folders/1sQdnuZe2onTxvguUUV0LeVsOGWXb-d_q?usp=sharing
```
* Deployment

**See Qt5.12.2_Release.md for details**
Note that the names of all paths in the document cannot be changed, otherwise it will cause compilation or running errors.

* Compile

On the host side, enter the Qt project directory, qmake && make.

* Run

Two test demos are provided in the tool chain, corresponding to EGLFS and LinuxFB Backend. After the deployment is completed, the user can build the demo on the host side and run the demo on the tartget side to test whether the deployment is successful.

## Export device system

This chapter applies to:
When the user has completed the deployment of the work environment on one device, the current environment needs to be completely exported for batch deployment to other devices on the same platform.

The export device system is divided into two steps
1. Export device rootfs
2. Re-package the complete firmware, re-package rootfs and other firmware components to generate complete firmware

### Export device rootfs
1. In ubuntu18.04 environment, install `fireflydev`

```
sudo apt update
sudo apt install fireflydev
```

2. Use `ff_export_rootfs` to export the root file system

* It is recommended to use a mobile hard disk with a larger capacity

* The export tool will perform operations such as apt clean to reduce the file system size

* Export the root file system to the `/media/firefly/AC91-C4AE/` directory, the actual example is as follows:
```
        root@firefly:~# ff_export_rootfs
        ff_export_rootfs </path/to/store> [-t <ext4|btrfs>]

ff_export_rootfs /media/firefly/AC91-C4AE/
```

### Second package complete firmware
* Tool download: [firefly-linux-repack](https://en.t-firefly.com/doc/download/377.html#other_402)
* This tool is suitable for users who need to replace any part of the firmware349

```
firefly-linux-repack
    ├── bin
    │ ├── afptool
    │ └── rkImageMaker
    ├── pack.sh
    ├── Readme_en.md
    ├── Readme.md
    └── unpack.sh

```
1. Unpack
    Copy the officially released firmware to the current directory, rename it to update.img, execute unpack.sh
    After unpacking is complete, the generated file is in the output directory.

2. Combined package
    Keep the current directory structure, file name, etc. unchanged, replace the file with the same name under output/ with the customer's own file
    Execute pack.sh, after execution, generate new_update.img, which is the packaged firmware
The rootfs file name must be rootfs.img
The file name of parameter.txt must be parameter.txt

* Note:
During the package process, if the rootfs partition is not the last partition, the program will follow the size of the rootfs file,
Automatically modify the size of the rootfs partition in parameter.txt.
If the user himself changes parameter.txt, please pay attention to the entire package process.

```
tar -xzf firefly-linux-repack.tgz
cd firefly-linux-repack
mv /xxx/FIREFLY-RK3588-UBUNTU18.04-GPT-20210420-1510.img update.img
./unpack.sh

cp /customer/rootfs.img output/Image/rootfs.img

./pack.sh
ls new_update.img
```

## Screen rotation

Firefly Ubuntu Desktop uses the ff_rotate script to control screen rotation:

* ff_rotate

```
root@firefly:~# ff_rotate

        rotate screen and touchscreen
                run as root
                ff_rotate <orientation>
                        orientation: left, right, normal, inverted
#Image output flip
root@firefly:~# ff_rotate inverted
```

## Display version information
* ffgo

The ffgo command provided by Firefly can easily view the firmware information, which is convenient for developers to debug and locate problems.

When users need to feedback information to Firefly, they need to attach the version information displayed by ffgo version.

```
root@firefly:~# ffgo
Usage:
         ffgo: show this usage
         ffgo update: update ffgo
         ffgo version: get version
         ffgo cmdlist: get support cmd list
         ffgo [cmd]: run cmd in cmd list

root@firefly:~# ffgo update
update success
root@firefly:~#  ffgo version
OS:      Ubuntu 18.04.5 LTS
MODEL:   Firefly RK3588-ROC-PC HDMI(Linux)
FIREFLY: v2.04-1-g618089a
DATE:    20210316-1035
KERNEL:  Linux version 4.19.172 (liaoxt@tchip16) (gcc version 6.3.1 20170404 (Linaro GCC 6.3-2017.05), GNU ld (Linaro_Binutils-2017.05) 2.27.0.20161019) #107 SMP Mon Apr 19 09:01:32
```

## reset

Firefly Ubuntu supports restoring factory settings.
**Note that this factory setting means that the device is restored to its initial state after the last firmware upgrade**
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

Use `recovery reset` to restore to factory settings


## Startup program

Firefly Ubuntu Desktop can use the following methods to set the boot-up program, but you need to pay attention to the operating permissions and current environment variables:


```
$vim /home/firefly/.config/lxsession/Lubuntu/autostart
@/bin/mkdir /home/firefly/ssddd
```
Each command starts with `@`, and the above example will create the `/home/firefly/ssddd` directory



## Video hardware codec support

The integrated VPU of RV1126B has excellent video codec capabilities. Mpp is a set of video codec APIs provided by Rockchip for VPU, and is based on mpp. Rockchip provides a set of gstreamer codec plug-ins. Users can do video codec applications based on gstreamer according to their needs, or directly call mpp to achieve hardware codec acceleration.

The system provides a test video file located at /usr/local/test.mp4, the test file is 1080P, 24Fps, H264 encoding, Mp4 format.

There are several ways to verify and develop video codec related applications.

### Gstreamer

* Under Ubuntu 18.04, gstreamer 1.12 has been installed in the system.

/usr/local/bin/h264dec.sh Test hardware H264 decoding.

/usr/local/bin/h264enc.sh Test hardware H264 encoding.

Users can refer to these two scripts to configure their own gstreamer application.

### Mpv

The Mpv player provided by the system can directly call the rkmpp decoding plug-in.

### FFmpeg

FFmpeg only supports hardware decoding through Mpp for Rockchip temporarily, and there is no hardware encoding support for the time being.

Firefly Ubuntu has installed FFmpeg, and users can use it directly.

-Confirm rkmpp decoder

```shell
$ ffmpeg -decoders | grep "rkmpp"

 V..... h264_rkmpp h264 (rkmpp) (codec h264)
 V..... hevc_rkmpp hevc (rkmpp) (codec hevc)
 V..... vp8_rkmpp vp8 (rkmpp) (codec vp8)
 V..... vp9_rkmpp vp9 (rkmpp) (codec vp9)
```

-Test commands

```shell
$ ffmpeg -y -c:v h264_rkmpp -i test.mp4 -an -vf \
hwdownload,format=nv12,format=yuv420p output.yuv
```

**Special attention:** FFmpeg h264_rkmpp decodes AV_PIX_FMT_DRM_PRIME, which is DRM frame data. If it is based on drm display, you can directly output the frame, otherwise, you need to use hwdownload to convert.

For more information, please refer to [FFmpeg official website](http://ffmpeg.org/about.html).

### Mpp

* Under Ubunut system, mpp related dev packages have been installed in the system.

    For more related information, please refer to the relevant documents under `linux-sdk/docs/Linux/Multimedia`

## OpenGL-ES

RV1126B supports OpenGL ES1.1/2.0/3.0/3.1.

Firefly Ubuntu has provided complete OpenGL-ES support.

-Test commands

```shell
$ sudo test_glmark2_normal.sh
```

-webGL support

In the Chromium browser, type in the address bar: `chrome://gpu` to view the hardware acceleration support under Chromium.

Note:

1. EGL is an extension of OpenGL on the arm platform for the x window system, and its function is equivalent to the glx library under x86.

2. Because the Driver modesettings used by Xorg will load libglx.so by default (disabling glx will cause some applications that detect glx environment to fail to start), libglx.so will search for the dri implementation library in the system. However, RV1126B Xorg 2D acceleration is implemented directly based on DRM and does not implement the dri library, so during the startup process, libglx.so will report the following error.

    ```
    (EE) AIGLX error: dlopen of /usr/lib/aarch64-linux-gnu/dri/rockchip_dri.so failed
    ```

    This has no effect on the operation of the system and does not need to be processed.

3. Based on the same reasoning, some applications will report the following errors during the startup process, and there is no need to deal with them, which will not affect the operation of the application.

    ```
    libGL error: unable to load driver: rockchip_dri.so
    libGL error: driver pointer missing
    libGL error: failed to load driver: rockchip
    ```

4. In some versions of Ubuntu software previously released by Firefly, the loading of libglx.so is disabled by default. In some cases, the following errors will occur when running certain applications:

    `GdkGLExt-WARNING **: Window system doesn't support OpenGL.`

    The correction method is as follows:

    Delete the following three lines of configuration in `/etc/X11/xorg.conf.d/20-modesetting.conf`.

    ```shell
    Section "Module"
         Disable "glx"
    EndSection
    ```



## OpenCL

Firefly Ubuntu has added opencl1.2 support, you can run the built-in `clinfo` of the system to get platform opencl related parameters.

```
firefly@firefly:~$ clinfo
Platform #0
 Name: ARM Platform
 Version: OpenCL 1.2 v1.r14p0-01rel0-git(966ed26).f44c85cb3d2ceb87e8be88e7592755c3

 Device #0
   Name: Mali-T860
   Type: GPU
   Version: OpenCL 1.2 v1.r14p0-01rel0-git(966ed26).f44c85cb3d2ceb87e8be88e7592755c3
   Global memory size: 1 GB 935 MB 460 kB
   Local memory size: 32 kB
   Max work group size: 256
   Max work item sizes: (256, 256, 256)
…
```

## QT Dual screen different display
### Demo introduction
The object is to demonstrate the use of QT demo application Ubuntu system implementation and operation of dual-screen display.
### Into the desktop environment
```shell
export XAUTHORITY=/home/firefly/.Xauthority
export DISPLAY=:0
```
### Set environment variables
[Environment to build reference](ubuntu_manual.html#qt-cross-compilation-environment-support)
```shell
export QT_QPA_PLATFORM=xcb
export QT_QPA_EGLFS_INTEGRATION=XCB_EGL
```
### Run the demo
```shell
./firefly_arm64_qt5.12.2_18.04/demo/double_panel_demo
```
### Demo code directory
```shell
firefly_arm64_qt5.12.2_18.04/example/double_panel_demo
```
### Code compilation
```shell
cd example
qmake
make
```
### Add your own QT project
1. Add your own QT project in the example directory.

2. Edit the gui.pro file in the example directory.

3. Assuming that the project directory is named double_panel_demo, add SUBDIRS += double_panel_demo to the gui.pro file.

4. qmake && make
### Running effect
![](../../../rv1126b_img/common/linux_ubuntu_double_panel.jpg)

## On-screen keyboard

The official Ubuntu system comes with an on-screen keyboard, you can click to open it in the menu bar:
![](../../../rv1126b_img/common/linux_ubuntu_onboard.jpg)


## Sound configuration

Equipment generally has 2 or more audio devices. The most common ones are the audio output of headphones and HDMI. The following is an example of audio settings for users' reference.

### Specify audio device from the command line

The audio files that come with the system are in the `/usr/share/sound/alsa/` directory, please check the sound card device before playing:

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

Then specify the sound card to play audio, where device 0 of sound card card1 corresponds to the headphone port, and device 0 of card0 corresponds to HDMI. The general audio playback command is `aplay -Dhw:0,0 Fornt_Center.wav`, but the audio file that comes with the system is mono, so in order to prevent playback failure, you can follow the following command to play:

```
#Select the headphone port to output audio files
root@firefly:/usr/share/sounds/alsa# aplay -Dplughw:1,0 Front_Center.wav
Playing WAVE'Front_Center.wav': Signed 16 bit Little Endian, Rate 48000 Hz, Mono

#Select HDMI output audio file
root@firefly:/usr/share/sounds/alsa# aplay -Dplughw:0,0 Front_Center.wav
Playing WAVE'Front_Center.wav': Signed 16 bit Little Endian, Rate 48000 Hz, Mono
```

### Select audio device in the graphical interface

In the graphical interface, play the prepared audio file, then click the sound icon to open the `Sound Setting`, and select the `Configuration`. You can see two sound card devices, for example, set to HDMI output audio, then set the HDMI sound card device to `Output`, and set the other sound card to `Off`. (If the HDMI is silent or low, you can try to increase the volume by pressing the physical button on the HDMI screen)

![](../../../rv1126b_img/common/linux_ubuntu_sound_setting.jpg)


### Kernel configuration

In the kernel directory, open the kernel configuration options menu:

```
make firefly_linux_defconfig
make menuconfig
```

After entering the kernel configuration menu, select in turn: `Device Drivers` -> `USB Support` -> `USB Gadget Support`.

Select `USB functions configurable through configfs` in the `USB Gadget Driver` option.

At the same time, select `Function filesystem (FunctionFS)`.

```
<*> USB Gadget Drivers (USB functions configurable through configfs) --->
        USB functions configurable through configfs
[*] Function filesystem (FunctionFS)
```

Then compile the kernel in the `kernel` directory:

```
make rk3588-firefly.img -j12
```

After the compilation is completed, burn the kernel to the device. For the burn process, please refer to the partition image burning part in the Wiki tutorial: [Upgrade Firmware](upgrade_firmware.md).

### ADB connection

After installing ADB, connect the device and PC with a Micro USB OTG cable. Then use the command `adb devices` to see if there is a device connected:

```
firefly@Desktop:~$ adb devices
List of devices attached
0123456789ABCDEF device
```

From the returned information, you can see that the device has been found, indicating that the device has been successfully connected.

After the device is successfully connected, enter the command `adb shell` to enter the command line mode:

```
firefly@Desktop:~$ adb shell
#
```

In this state, the current path of the command line cannot be seen, and the Tab key completion function is invalid. You need to enter a user to operate normally.

```
firefly@Desktop:~$ adb shell
# su firefly
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

firefly@firefly:/$
#The command line can be used normally here
```

The user can also use the command `adb shell /bin/bash`, or enter the normal command line mode.


You can enter `adb help` to view the command line help information. Note that not all commands are available. The help information is for reference only.

## Serial port

There are two types of serial ports on the device, one is a common serial port and the other is a serial port converted from SPI through a conversion chip. The converted serial port has the same function as the ordinary serial port, but it is necessary to note that their device file names are different. The following is the difference between the two:

```
//Ordinary serial port
root@firefly:~# ls /dev/ttyS*
ttyS0 ttyS1 ttyS2 ttyS3

//SPI to serial port
root@firefly:~# ls /dev/ttysWK*
ttysWK0 ttysWK1 ttysWK2 ttysWK3
```

### Set the baud rate

Take `ttyS4` as an example to view the serial port baud rate command:

```
root@firefly:~# stty -F /dev/ttyS4
speed 9600 baud; line = 0;
-brkint -imaxbel
```

Set the baud rate command:

```
//Set the baud rate to 115200
root@firefly:~# stty -F /dev/ttyS4 ospeed 115200 ispeed 115200 cs8

//Check to confirm whether it has been modified
root@firefly:~# stty -F /dev/ttyS4
speed 115200 baud; line = 0;
-brkint -imaxbel
root@firefly:~#
```
### Turn off echo

The echo function will affect our test results when doing the loopback test, so you need to turn off the echo in the loopback test or other special circumstances. The following is the command to turn off echo:

```
//Close back to display
root@firefly:~# stty -F /dev/ttyS4 -echo -echoe -echok

//Check the configuration of all functions and check whether it has been closed. "-" means the function has been closed
root@firefly:~# stty -F /dev/ttyS4 -a | grep echo
isig icanon iexten -echo -echoe -echok -echonl -noflsh -xcase -tostop -echoprt
echoctl echoke -flusho -extproc
```

### Send and receive raw data

In actual applications, there may be differences between the actual data sent and the data we want to send, such as a carriage return or other special characters. Use `stty` to set the send and receive to `raw` mode to ensure that the raw data is sent and received. The following is the setting command:

```
root@firefly:~# stty -F /dev/ttyS4 raw
```

### Operating mode

The serial port has two working modes: interrupt mode and DMA mode.

#### Interrupt mode

The kernel configures the serial port as interrupt mode by default, so no modification is required. The transmission rate in interrupt mode is relatively fast, but it is easy to lose packets or make mistakes when transferring large amounts of data, so please do not use interrupt mode when the amount of data is relatively large.

#### DMA mode

DMA mode is mainly used when transferring large amounts of data. The kernel will provide a buffer space for the serial port to receive data to minimize the packet loss rate of the serial port transmission.

**Note:** The default size of the buffer space limit is `8K`. If one transfer exceeds the buffer size, packets will be lost, so if you use DMA mode, the sender needs to send in packets.

Device tree file configuration

```
&uart4 {
        status = "okay";
+ dmas = <&dmac_peri 8>, <&dmac_peri 9>;
+ dma-names = "tx", "rx";
};

```

The DMA mode cannot increase the transfer rate. On the contrary, the transfer rate will be reduced because of the need for buffering, so do not use the DMA mode if you do not need to transfer a large amount of data.

### Flow Control

Whether it is interrupt mode or DMA mode, data transmission cannot be guaranteed to be foolproof, because when a large amount of data is transmitted for a long time, DDR, CPU frequency conversion or high occupancy rate may cause the upper layer to process the data in time and cause packet loss. At this time, it is necessary to use Flow control. There are two types of flow control, one is software flow control and the other is hardware flow control. Only the use of hardware flow control is introduced below.

#### Hardware Support

Hardware flow control needs hardware support, and the device's serial port `CTX` and `RTX` pins need to be connected to the remote device.

**Note:** Not all serial ports of the device support hardware flow control, please confirm from the schematic diagram whether the hardware supports

#### Device tree file configuration

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
+ pinctrl-0 = <&uart3_xfer &uart3_cts &uart3_rts>;
        status = "disabled";
};
```

#### Application layer settings

The upper layer also needs to open the flow control setting. Here is how to open the flow control in stty. If you are using other applications, please turn on the flow control from the application.

```
//Turn on flow control
root@firefly:~# stty -F /dev/ttyS4 crtscts

//Check whether the flow control is turned on, the "-" sign means that the function has been turned off
root@firefly:~# stty -F /dev/ttyS4 -a | grep crtscts
-parenb -parodd -cmspar cs8 hupcl -cstopb cread clocal crtscts
```

<!--以下功能未测试-->
