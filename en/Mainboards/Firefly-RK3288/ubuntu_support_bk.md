# Ubuntu application layer support

The Firefly-RK3288 Ubuntu Desktop system is based on the Ubuntu 32bit system and currently maintains the Ubuntu 18.04 version. Has the following characteristics:

1. The desktop environment uses LXDE, Lubuntu + Firefly custom theme, simple and beautiful.
2. Xserver uses GPU + RGA for 2D acceleration, runs smoothly, and takes up less CPU resources.
3. Streamline system services for embedded platforms.
4. Provide OpenGL and OpenCL support based on Arm Mali GPU.
5. Provide video hard codec support based on Rockchip VPU + Mpp.
6. Adapt to QT, Docker, Electron and other development frameworks.
7. Provide a series of interfaces to operate onboard resource equipment.

## Basic information

### User and password

After the Firefly Linux Desktop system starts up, it automatically logs in to the firefly user.

If a debugging serial port is connected, the serial terminal automatically logs in to the root user.

- firefly user password: firefly

- root user password: firefly

## Video hardware codec support

RK3288 integrated VPU has excellent video encoding and decoding capabilities, Mpp is a set of video encoding and decoding APIs provided by Rockchip for VPU, and is based on mpp. Rockchip provides a set of gstreamer codec plug-ins. According to their own needs, users can use gstreamer to do video codec applications, or directly call mpp to achieve hardware codec acceleration.

The system provides a test video file, located in `/usr/local/test.mp4`, the test file is 1080P, 24Fps, H264 encoding, Mp4 format.

There are several ways to verify and develop video codec related applications.

### Gstreamer

- Ubuntu 16.04, gstreamer 1.12 Already installed in the `/opt/` directory
- Ubuntu 18.04, gstreamer 1.12 Already installed in the system

`/usr/local/bin/h264dec.sh` Test H264 decoding

`/usr/local/bin/h264enc.sh` Test H264 coding

Users can refer to these two scripts to configure their own gstreamer application.

### Mpv

The Mpv player provided by the system can directly call the rkmpp decoding plug-in.

### FFmpeg

FFmpeg only supports hardware decoding through Mpp for Rockchip for the time being, there is no support for hardware encoding for the time being.

The Ubuntu system released by Firefly has already installed FFmpeg, and users can use it directly.

- Confirm rkmpp decoder

```bash
$ ffmpeg -decoders | grep "rkmpp"

 V..... h264_rkmpp           h264 (rkmpp) (codec h264)
 V..... hevc_rkmpp           hevc (rkmpp) (codec hevc)
 V..... vp8_rkmpp            vp8 (rkmpp) (codec vp8)
 V..... vp9_rkmpp            vp9 (rkmpp) (codec vp9)
```

- Test command

```bash
$ ffmpeg -y -c:v h264_rkmpp -i test.mp4 -an -vf \
        hwdownload,format=nv12,format=yuv420p output.yuv
```

**pay attention:**FFmpeg h264_rkmpp decodes AV_PIX_FMT_DRM_PRIME, which is DRM frame data, if it is based on drm display, you can directly output the frame, otherwise, you need to use hwdownload to convert.

More content can refer to[FFmpeg official website](http://ffmpeg.org/about.html).

### Mpp

- Under Ubuntu system, mpp related dev packages have been installed in the system.

        For more related information and Demo, please refer to

        https://github.com/rockchip-linux/mpp/tree/develop/test

## OpenGL-ES

RK3288 support OpenGL ES1.1/2.0/3.0/3.1。

The Ubuntu system released by Firefly has provided complete OpenGL-ES support.

- Test command

```bash
$ sudo test_glmark2_normal.sh
```

- webGL support

In the Chromium browser, type: `chrome://gpu` in the address bar to see hardware acceleration support.

Note:

1. EGL is an extension of the OpenGL for the x window system on the arm platform, which is equivalent to the glx library under x86.

2. Since the driver mode setting used by Xorg will load libglx.so by default (disabling glx will cause some applications failing to detect the glx environment), libglx.so will search the dri implementation library in the system. However, rk3288 Xorg 2D acceleration is directly based on DRM, and does not implement dri library, so libglx.so will report the following error during startup.

    ```bash
    (EE) AIGLX error: dlopen of /usr/lib/arm-linux-gnueabihf/dri/rockchip_dri.so failed
    ```

   This has no effect on system operation and does not need to be processed.

3. Based on the same reason, some applications will report the following errors during startup, without processing, and will not affect the operation of the application.

    ```bash
    libGL error: unable to load driver: rockchip_dri.so
    libGL error: driver pointer missing
    libGL error: failed to load driver: rockchip
    ```

4. Some versions of Ubuntu released by Firefly have turned off loading libglx.so by default. In some cases, some applications will run the following error.

    `GdkGLExt-WARNING **: Window system doesn't support OpenGL.`

    The method of correction is as follows：

    Delete  `/etc/X11/xorg.conf.d/20-modesetting.conf`  three-line configuration。

    ```bash
    Section "Module"
         Disable     "glx"
    EndSection
    ```

## OpenCL

The Ubuntu system released by Firefly has added opencl1.2 support, and can run the system's built-in `clinfo` to get the platform opencl related parameters.

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

## Screen rotation

The Ubuntu system released by Firefly, if you need to rotate the display direction of the system by default, you can modify the direction of the corresponding display device in `/etc/default/xrandr`.

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

For platforms with a touch screen, if you need to rotate the direction of the touch screen, you can modify the SwapAxes / InvertX / InvertY values in `/etc/X11/xorg.conf.d/05-gslX680.conf`.

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

## Virtual keyboard

In the Ubuntu system released by Firefly, you can execute the `onboard` at the menu to open a virtual keyboard.

![](../../../rk3288_img/onboard.png)

## Sound setting

The Firefly-RK3288 board generally has two or more audio devices. Headphone and HDMI are two common audio devices. Below is an example of audio settings for users to refer to.

### Specify audio devices in terminal

Audio files in the system are in `/usr/share/sound/alsa`, please check your sound card before play them:

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

And then specify a sound card to play an audio file. Among them, card0-device0 stands for headphone and card0-device1 stands for HDMI. We usually run command `aplay -Dhw:0,0 Fornt_Center.wav` to play an audio file, but in order to avoid playback failure because the audio files in the system are mono, so you can run as follow:

```bash
#choose headphone to play
root@firefly:/usr/share/sounds/alsa# aplay -Dplughw:0,0 Front_Center.wav
Playing WAVE 'Front_Center.wav' : Signed 16 bit Little Endian, Rate 48000 Hz, Mono

#choose HDMI to play
root@firefly:/usr/share/sounds/alsa# aplay -Dplughw:0,1 Front_Center.wav
Playing WAVE 'Front_Center.wav' : Signed 16 bit Little Endian, Rate 48000 Hz, Mono
```

## Start-up applications while booting up

In the system, the application program can be set up automatically according to the user's needs.

Select in turn from the System Menu Bar, `Preferences` -> `Default applications for LXSession`, open the setting interface, and you can set the default opening mode of the application in `Launching applocations`.

![](../../../rk3288_img/autostart_setting.png)

In the `Autostart` column, you can select applications that you need to boot. For example, Bluetooth does not start by default. If it needs to start automatically, it can start Bluetooth automatically by ticking the `Blueman Applet`.

![](../../../rk3288_img/autostart_bluetooth.png)

## ADB debug

ADB, called Android Debug Bridge, is a command line debugging tool for Android that can perform various functions such as tracking system logs, uploading and downloading files, and installing applications. Take Firefly-RK3288 as an example, in order to use the ADB tool for debugging on the Ubuntu system, we have transplanted the ADB service on the board. However, since it is not the Android system, many ADB commands such as `adb logcat`, `adb install`, etc. are not available, and are only used as ordinary debugging assistant tools to perform operations such as shell interaction, uploading and downloading of files, etc. Similarly, network remote ADB debugging is not available.

### Download ADB

In the Ubuntu system, run:

```bash
sudo apt-get install android-tools-adb
```

### ADB connect

After the ADB is installed, the development board and PC are connected with the Micro USB OTG cable. Then check whether there is a device connection by command `adb devices` :

```bash
firefly@Desktop:~$ adb devices
List of devices attached
0123456789ABCDEF        device
```

When the device is connected successfully, enter the command line mode by entering the command `adb shell` :

```bash
firefly@Desktop:~$ adb shell
#
```

In this state, the current path of the command line can not be seen, and the Tab key completion function is invalid. It needs to enter a user to operate normally.

```bash
firefly@Desktop:~$ adb shell
# su firefly
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

firefly@firefly:/$
```

It can also enter the normal command line by entering the command `adb shell /bin/bash`.

Command `adb help` can print help information. Note that not all commands are available. Help information just for reference only.

## Serial port

There are two types of serial ports on the development board: the normal ones that are built-in SoC UARTs, and the extended ones that are converted by SPI. The use of the converted serial port is exactly the same as of the normal one, but with different device file naming:

```bash
# uart
root@firefly:~# ls /dev/ttyS*
ttyS0  ttyS1  ttyS2  ttyS3

# spi to uart
root@firefly:~# ls /dev/ttysWK*
ttysWK0  ttysWK1  ttysWK2  ttysWK3
```

### Set baud rate

Take `ttyS4` as an example below. To view the serial port baud rate setting, run:

```bash
root@firefly:~# stty -F /dev/ttyS4
speed 9600 baud; line = 0;
-brkint -imaxbel
```

Set baud rate:

```bash
# Set the baud rate to 115200
root@firefly:~# stty -F /dev/ttyS4 ospeed 115200 ispeed 115200 cs8

# Confirm if it has been modified
root@firefly:~# stty -F /dev/ttyS4
speed 115200 baud; line = 0;
-brkint -imaxbel
```

### Turn off echo

Echo is useful in an interative serial terminal session but will affect the result of data transfer, for example, in a loopback test. To turn off echo, run:

```bash
# Close echo
root@firefly:~# stty -F /dev/ttyS4 -echo -echoe -echok

# Check the configuration of all features and check if it is closed. The "-" means the feature is off
root@firefly:~# stty -F /dev/ttyS4 -a | grep echo
isig icanon iexten -echo -echoe -echok -echonl -noflsh -xcase -tostop -echoprt
echoctl echoke -flusho -extproc
```

### Send and receive raw data

If the serial port is not in raw mode, there may be differences between the data sent and received (For example, the driver may do some line end converting, which is no good to binary data). Use `stty` to set it in `raw` mode to ensure that the same data is exactly sent and received:

```bash
root@firefly:~# stty -F /dev/ttyS4 raw
```

### Operating mode

The serial port has two operating modes: interrupt mode and DMA mode.

#### Interrupt mode

By default, the kernel configures the serial port into interrupt mode so there is no need to make any change. In interrupt mode, the transfer latency is low, but it tends to lose data in bulk data transfer. Therefore do not use interrupt mode in that case.

#### DMA mode

The DMA mode is mainly used when transferring a large amount of data. The kernel will provide a buffer space for the serial port to receive data to minimize the packet loss rate of the serial port transmission.

**WARNNING:** The default size of the cache space is "8K". If the transfer exceeds the cache size, the packet will be lost. Therefore, if the DMA mode is used, the sender needs to send multiple packets.

DTS setting:

```dts
&uart4 {
        status = "okay";
+       dmas = <&dmac_peri 8>, <&dmac_peri 9>;
+       dma-names = "tx", "rx";
};

```

DMA mode does not increase the transfer rate. Instead, the transfer rate is reduced because of the need for caching, so do not use DMA mode if you do not need to transfer large amounts of data.

### Flow control

No matter whether it is interrupt mode or DMA mode, data transmission is not foolproof, because DDR, CPU frequency conversion or high occupancy may cause processing data not to be timely when transferring large amounts of data . At this time, it is necessary to use flow control. There are two types of flow control, one is software flow control and the other is hardware flow control. The following only describes the use of hardware flow control.

#### Hardware support

Hardware flow control requires hardware support. The `CTX` and `RTX` pins of the development board need to be connected to the device.

**WARNNING:** Not all serial ports on the development board support hardware flow control. Please confirm the hardware support from the schematic.

#### DTS setting

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

**WARNNING:** Not all serial ports on the development board support hardware flow control. Please confirm the hardware support from the schematic.

#### Application settings

The application also needs to turn on flow control. Here is how to do that with `stty`:

```bash
# Turn on flow control
root@firefly:~# stty -F /dev/ttyS4 crtscts

# Check if the flow control is turned on, the "-" means the function has been turned off.
root@firefly:~# stty -F /dev/ttyS4 -a | grep crtscts
-parenb -parodd -cmspar cs8 hupcl -cstopb cread clocal crtscts
```

## MIPI Camera (OV13850)

The RK3288 board has added support for the MIPI camera `OV13850`, and examples of cameras have also been added to the application. The following describes the relevant configuration.

### DTS setting

Take `rk3288-firefly-aio.dts` as an example, these are the configurations that the camera needs to open.

```dts
// power
&vcc_mipi {
  status = "okay";
};

&dvdd_1v2 {
  status = "okay";
};

// mipi camera0
&ov13850 {
  status = "okay";
};

&mipi_phy_tx1rx1 {
  status = "okay";
};

// mipi camera1
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

**Note:** RK3288 only supports one ISP, so the two cameras cannot be used at the same time; for different RK3288 development boards, the corresponding GPIO properties may need to be modified.

### DEBUG

Check whether the ISP driver is loaded successfully, and you can see the following device information:

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

Check if the `OV13850` device is registered successfully. If successful, you can see the following print:

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
// Device ID successfully obtained
[    3.914401] ov13850 3-0010: Detected OV00d850 sensor, REVISION 0xb2
```

### Test

You can run the following script to test:

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

The result is shown in the figure:

![](../../../rk3288_img/mipi_csi.png)

## External storage device rootfs

In addition to using the root filesystem in the internal eMMC, you can also use the root filesystem of the external storage device, such as SD card. The following is an example of Firefly-RK3288 mounting the root filesystem of SD card.

### Create partitions on SD card

Insert SD card on PC and use `gdisk` tool to separate out a partition that loads the root filesystem.

Using `fdisk -l` to get the device name of the SD card and enter the corresponding device with gdisk command, and then run:

```bash
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

#enter '?' can print the help information
#enter 'p' to display all partitions of SD card

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

#now create a new partition, create the appropriate partition size to the root filesystem according to your actual situation.

Command (? for help): n     #entern 'n' to create a new partition
Partition number (1-128, default 1): 1      #create No.1 partition
First sector (34-15278046, default = 2048) or {+-}size{KMGTP}:   #press enter, use the default value.
Last sector (2048-15278046, default = 15278046) or {+-}size{KMGTP}: +3G     #partition size is 3G
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300):    #press enter, use the default value.
Changed type of partition to 'Linux filesystem'

-------------------------------------------------------------------

#then enter 'i' to get the partition's unique GUID, and mark down it.

Command (? for help): i
Using 1
Partition GUID code: 0FC63DAF-8483-4772-8E79-3D69D8477DE4 (Linux filesystem)
Partition unique GUID: 6278AF5D-BC6B-4213-BBF6-47D333B5CB53
First sector: 2048 (at 1024.0 KiB)
Last sector: 6293503 (at 3.0 GiB)
Partition size: 6291456 sectors (3.0 GiB)
Attribute flags: 0000000000000000
Partition name: 'Linux filesystem'

-------------------------------------------------------------------

#enter 'wq' save and exit gdisk

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

Format the newly created partition.

```bash
sudo mkfs.ext4 /dev/sdb1
```

When the formatting is completed, use the `dd` command to flash the root filesystem into the newly created partition of the SD card.

```bash
#Fill in the root filesystem path and device name according to your situation.
dd if=/rootfs_path/rootfs.img of=/dev/sdb1
```

### Mount root filesystem of SD card

Firstly, we need to modify the device tree file, open the corresponding DTS file, and rewrite the `root` value of the `bootargs` parameter under the `chosen` node as the unique GUID recorded before.

```bash
#Change the root value to the first 12 digits of the unique GUID.
chosen {
    bootargs = "earlycon=uart8250,mmio32,0xff1a0000 swiotlb=1 console=ttyFIQ0 rw root=PARTUUID=6278AF5D-BC6B  rootfstype=ext4 rootwait";
};
```

And then compile the kernel and flash it into the development board. After the root filesystem is flashed to SD card, insert SD card into the TF slot of the development board, and the root filesystem can be accessed on boot.

Note:

- Before operations, please backing up important files in USB disk or SD card to avoid data lost.
- When operating, please confirm the device name corresponding to your SD card.
- The value of the `root` parameter in the DTS file is unique GUID. Just write down the first 12 bits. You can also modify the value according to the help information of `gdisk`.

## Network startup

Network startup is to use TFTP to download the kernel and DTB file from the server to the memory of the target machine and mount the network root filesystem to the target machine through NFS server to achieve diskless startup. Following is an example based on Firefly-RK3288 for user reference.

Prerequisite:

- Firefly-Rk3288 development board.
- Router and network cable.
- A server with TFTP and NFS.
- A root filesystem.

### Server deployment

1. Deploy TFTP on the server

Install TFTP server.

```bash
sudo apt-get install tftpd-hpa
```

Create `/tftpboot` directory and grant permission:

```bash
mkdir /tftpboot
sudo chmod 777 /tftpboot
```

And then modify TFTP server configuration file `/etc/default/tftpd-hpa` :

```bash
# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/tftpboot"
TFTP_ADDRESS="0.0.0.0:69"
TDTP_OPTIONS="-c -s -l"
```

Then restart TFTP server.

```bash
sudo service tftpd-hpa restart
```

Create a new file in `/tftpboot` to test TFTP:

```bash
firefly@Desktop:~$ cd /tftpboot/
firefly@Desktop:/tftpboot$ touch test
firefly@Desktop:/tftpboot$ cd /tmp
firefly@Desktop:/tmp$ tftp 127.0.0.1
tftp> get test
tftp> q
```

And if you can see the `test` file in `tmp`, it means TFTP server is working.

2. Deloy NFS on the server

Install NFS server:

```bash
sudo apt-get install nfs-kernel-server
```

Create a shared directory:

```bash
sudo mkdir /nfs
sudo chmod 777 /nfs
cd /nfs
sudo mkdir rootfs
sudo chmod 777 rootfs
```

And then copy the root filesystem into the `/nfs/rootfs`.

Add the shared directory path in `/etc/exports`:

```bash
/nfs/rootfs *(rw,sync,no_root_squash,no_subtree_check)
```

The shared directory is set according to the user's actual situation, in which `*` represents all users' access.

Restart NFS server:

```bash
sudo /etc/init.d/nfs-kernel-server restart
```

Locally mount the shared directory to test whether the NFS server is available:

```bash
sudo mount -t nfs 127.0.0.1:/nfs/rootfs/ /mnt
```

Consistent content with `nfs/rootfs` seen in the `mnt` directory means NFS server is working. And then umount:

```bash
sudo umount /mnt
```

### Kernel configuration

If you want to mount the network root filesystem, you need to do the relevant configuration in the kernel and modify the configuration in DTS.

Firstly, setting kernel, run `make menuconfig` in kernel directory, and  select the relevant configuration:

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

Modify the device tree file `rk3288-firefly.dts`, and rewrite the `root` value of the `bootargs` parameter under the `chosen`:

```c
chosen {
    bootargs = "earlycon=uart8250,mmio32,0xff1a0000 swiotlb=1 console=ttyFIQ0 root=/dev/nfs rootfstype=ext4 rootwait";
};

```

`root=/dev/nfs` means mount network root filesystem by NFS.

Compile kernel:

```bash
make ARCH=arm rk3288-friefly.img -j12
```

After compile kernel, copy `zboot/img`, `rk3288-firefly.dtb` to the `/tftpboot`:

```bashj
cp zboot.img /tftpboot
cp arch/arm/boot/dts/rk3288-firefly.dtb /tftpboot
```

Detailed instructions can be referred to in the kernel directory: `kernel/Documentation/filesystems/nfs/nfsroot.txt`

### U-Boot setting

The target machine plugs in the wire and connects to the server. Then startup and enter U-Boot command line mode and set the following parameters:

```bash
=> setenv ipaddr 192.168.31.101     #target machine IP
=> setenv serverip 192.168.31.106   #set serverip as the server IP

#Set up to download the kernel and DTB files from TFTP. Modify the corresponding address according to actual target machine.
=> setenv bootcmd tftpboot 0x02007800 boot.img \; tftpboot 0x08300000 rk3288-firefly.dtb \; bootm 0x02007800 - 0x08300000

#Set up mount network root filesystem, ip=<client-ip>:<server-ip>:<gw-ip>:<netmask>:<hostname>:<device>:<autoconf>
=> setenv bootargs root=/dev/nfs rw nfsroot=192.168.31.106:/nfs/rootfs,v3 ip=192.168.31.101:192.168.31.106:192.168.31.1:255.255.255.0::eth0:off

#save the setting
=> saveenv
Saving Environment to MMC...
Writing to MMC(0)... done

#start up the target machine
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

You can see in the boot kernel log:

```cj
[   12.146297] VFS: Mounted root (nfs filesystem) on device 0:16.
```

This means that the network root filesystem has been mounted.

Note:

- Ensure that TFTP server and NFS server are working.
- Make sure that the target machine inserts the network cable first and then boot-up, and in the same LAN with the server. If it is directly connected to the target machine and the server, please use the crossover network cable.
- In the kernel configuration, `Root file system on NFS` depends on `IP: kernel level autoconfiguration`, please select `IP: kernel level autoconfiguration` first, and then you can find `Root file system on NFS` and select it.
-In the U-Boot command line, please confirm the burning address of `zboot.img` and the burning address of dtb file. (Hint: In the file structure of `zboot.img`, there is a 2k header file at the beginning, and then the kernel. So when TFTP downloads the kernel to the target machine, it must be downloaded to the corresponding kernel address minus 0x0800).
- When setting up the remote root file system for mounting, `nfsroot = 192.168.31.106:/nfs/rootfs,v3`, `v3` represents the version information of NFS, Please add it to avoid unsuccessful mounting.

## Update kernel and U-Boot online

This section describes a simple process for online updates. Package the kernel, U-Boot or other files that need to be updated into DEB, and then import them into the local package repository to download and update automatically on the development board.

### Prepare DEB installation package

Kernel and U-Boot update files are ready: `zboot.img`, `trust.img`, `uboot.img`.

DEB is a software package format of Debian Linux. The key to packaging is to create a `control` file in the `DEBIAN` directory. Now create DEB working directory:

```bash
mkdir deb
cd deb
mkdir firefly-firmware
cd firefly-firmware
mkdir DEBIAN
mkdir -p usr/share/{kernel,uboot}

# put the update files to corresponding directory
mv ~/zboot.img ~/deb/firefly-firmware/usr/share/kernel
mv ~/uboot.img ~/deb/firefly-firmware/usr/share/uboot
mv ~/trust.img ~/deb/firefly-firmware/usr/share/uboot
```

The files stored in the `DEBIAN` directory are the control files for DEB package installation and the corresponding script files.

Create control files `control` and script files `postinst` under the `DEBIAN` directory, the `control` file is used to record software package name, version number, platform, dependency information, and other data.

```bash
Package: firefly-firmware #directly name
Version: 1.0
Architecture: armhf
Maintainer: neg
Installed-Size: 1
Section: test
Priority: optional
Descriptionon: This is a deb test
```

The `postinst` file is as follows:

```bash
echo "-----------uboot updating------------"
dd if=/usr/share/uboot/uboot.img of=/dev/disk/by-partlabel/uboot

echo "-----------trust updating------------"
dd if=/usr/share/uboot/trust.img of=/dev/disk/by-partlabel/trust

echo "-----------kernel updating------------"
dd if=/usr/share/kernel/zboot.img of=/dev/disk/by-partlabel/boot
```

Note: `postinst` is a script that runs after unpacking data, and other correspondingly scripts:

- preinst: A script that runs before unpacking data.
- prerm: A script that runs before deleting files when uninstalled.
- postrm: A script that runs after deleting a file.

Only the `preinst` script is used here.

Here is the directory tree created:

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

Enter the deb directory and generate the DEB package with the `dpkg` command:

```bash
dpkg -b firefly-firmware firefly-firmware_1.0_armhf.deb
```

When generating DEB packages, they are usually named according to this specification: `package_version-reversion_arch.deb`.

### Create a local repository

Install the packages we need:

```bash
sudo apt-get install reprepro gnupg
```

Then use GnuPG tool to generate a GPG key. After executing the command, follow the prompt to operate.

```bash
gpg --gen-key
```

Run `sudo gpg --list-keys`, can see the key information:

```bash
sudo gpg --list-keys

gpg: WARNING: unsafe ownership on homedir '/home/firefly/.gnupg'
/home/firefly/.gnupg/pubring.kbx
--------------------------------
pub   rsa3072 2019-05-31 [SC] [expires: 2021-05-30]
      BCB65788541D632C057E696B8CBC526C05417B76
	  uid           [ultimate] firefly <firefly@t-chip.com>
	  sub   rsa3072 2019-05-31 [E] [expires: 2021-05-30]
```

Next, create the package repository:

```bash
cd /var/www
mkdir apt
mkdir -p ./apt/incoming
mkdir -p ./apt/conf
mkdir -p ./apt/key
```

Export the previously generated keys to the repository folder. Please correspond to the user name and mailbox address you created.

```bash
gpg --armor --export firefly firefly@t-chip.com > /var/www/apt/key/deb.gpg.key
```

Create a `distributions` file in the `conf` directory, which reads as follows:

```bash
Origin: Neg   #your name
Label: Mian     #package repository name
Suite: stable   #(stable or unstable)
Codename: bionic    #codename
Version: 1.0
Architectures: armhf
Components: main    #components name,such as main，universe.
Description: Deb source test
SignWith: BCB65788541D632C057E696B8CBC526C05417B76 #the key you generated
```

Establishment of repository tree:

```bash
reprepro --ask-passphrase -Vb /var/www/apt export
```

Add firefly-firmware_1.0_armhf.deb into repository:

```bash
reprepro --ask-passphrase -Vb /var/www/apt includedeb bionic ~/deb/firefly-firmware_1.0_armhf.deb
```

View the file in the repository:

```bash
root@Desktop:~# reprepro -b /var/www/apt/ list bionic
bionic|main|armhf: firefly-firmware 1.0
```

If you want to remove the file in the repository, just run:

```bash
reprepro --ask-passphrase -Vb /var/www/apt remove bionic firefly-firmware
```

And then install nginx server:

```bash
sudo apt-get install nginx
```

Modify nginx configuration file `/etc/nginx/sites-available/default`:

```c
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/apt;

    access_log /var/log/nginx/repo.access.log;
    error_log   /var/log/nginx/repo.error.log;

    location ~ /(db|conf) {
        deny all;
        return 404;
    }
}
```

Restart nginx server.

```bash
sudo service nginx restart
```

### Client update

In the client development board, first, add the source of the local package repository and add a new configuration file `bionic. list` under the directory `etc/apt/sources.list.d`, which is as follows:

```bash
deb http://192.168.31.106 bionic main
```

IP address is the nginx server address, `bionic` is the repository codename, `main` is the components name.

Get and add the GPG key from the server:

```bashj
wget -O - http://192.168.31.106/key/deb.gpg.key | apt-key add -
```

Update and install `firefly-firmware_1.0_armhf`:

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

You can see that the `poinst` script in the DEB package is executed during installation. After installation, restart the development board to take effect by loading the new images.

In `/usr/share` directory, we can see `kernel` and `uboot` directory with `zboot.img`, `uboot.img`, `trust.img` in it.

Note:

- In making DEB package, the directory at the same level as `DEBIAN` is regarded as the root directory. That is, the files placed in the same directory as `DEBIAN`, after the DEB package is installed on the client side, can be found in the root directory of the target system.
- The files and scripts in DEB package should be adjusted according to your actual situation.
- Every time the configuration file is modified in the repository, the repository directory tree should be re-imported.
- In nginx server configuration, the `root` parameter configures the path of the repository. Please modify it according to the actual situation.
- When the client adds a new download source file, pay attention to check the correct server IP address, package repository codename and component name. Note that the client needs to connect to the server.
- Clients need to add GPG keys with the `apt-key add` command before getting information about the local repository.

## Export root filesystem

This section describes how to export and package the root file system on a development board into an image file. Suitable for copying the environment of a certain development board to other development boards, or making upgrade firmware.

### Steps

1. Install `ssh`, `rsync` on the development board and PC:

    ```bash
        sudo apt-get install -y ssh rsync
    ```

2. Set up ssh on the development board to allow root user access:

   * Modify `#PermitRootLogin prohibit-password` in `/etc/ssh/sshd_config` file to `PermitRootLogin yes`
   * Restart the ssh service:

    ```bash
        sudo /etc/init.d/ssh restart
    ```

3. To synchronize the filesystem of the development board in the PC, please put the development board and the PC in the same local area network first. (Assuming the development board address is 192.168.100.100)

    ```bash
        mkdir rootfs
        sudo rsync -avx root@192.168.100.100:/ ./rootfs
    ```

4. At this point, all the files in the filesystem of the development board are saved in the rootfs directory, and you can start to make the root filesystem image:

    ```bash
        dd if=/dev/zero of=linuxroot.img bs=1M count=4000
        mkfs.ext4 linuxroot.img
        mkdir temp && sudo mount linuxroot.img ./temp
        sudo cp -rfp rootfs/*  temp/
        sudo umount temp/
        e2fsck -p -f linuxroot.img
        resize2fs  -M linuxroot.img
    ```

Finally, linuxroot.img is the finished root filesystem image, which is consistent with the filesystem of your development board.