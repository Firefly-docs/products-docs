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

		For more related information, please refer to the relevant documents under `linux-sdk/docs/Linux/Multimedia`.

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

## Compile the universal device bus driver locally

DEBIAN package download link: [https://drive.google.com/drive/folders/1GBl2n66fARqJLVEZ-LeW3bfbs3tyOW3C?usp=sharing](https://drive.google.com/drive/folders/1GBl2n66fARqJLVEZ-LeW3bfbs3tyOW3C?usp=sharing)

Install header files:

```shell
sudo dpkg -i linux-headers-4.4.194_4.4.194-93_armhf.deb
sudo dpkg -i linux-image-4.4.194_4.4.194-93_armhf.deb

cd /usr/src/linux-headers-4.4.194
make headers_check
make headers_install

# make scripts may go wrong but it doesn't matter
make scripts
```