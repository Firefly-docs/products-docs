# Hardware Function Usage
## Debug Serial Port

The debug serial port form of the EC-A8550JD4 is to be supplemented. The AIO-8550JD4 motherboard it adopts provides two debug serial port forms: a 3pin TTL socket and a Type-C port (shared with the download port). If the 3pin TTL socket is used, an external USB to TTL serial module is required. For the connection and usage of the module, please refer to [Serial Module](usb_to_ttl.md).

## Display Interface

The EC-A8550JD4 provides 1 HDMI 2.0 display output interface, and the system display will be shown after connecting a monitor.

Common operations:

* Resolution and refresh rate: select the output resolution and refresh rate in the system display settings on the desktop.
* Multiple displays: the device only provides 1 HDMI display output interface, without multi-display output.
* Screen on/off: the screen-off timeout can be configured in the power options of the system settings.

You can also check the connection status of the display interface through the command line:

```bash
# connected means connected, the connector name is subject to the actual system
cat /sys/class/drm/*/status
```

## Serial Port (RS232 / RS485)

The EC-A8550JD4 provides RS232 and RS485 serial ports. The number of the ports, the wiring positions and the corresponding device nodes in the system are subject to the actual product.

The serial ports can be tested for transmitting and receiving as follows (`/dev/ttyX` refers to the corresponding device node, please confirm it with `ls /dev/tty*` first):

```bash
# list the serial devices in the system
ls /dev/tty*

# set the serial port parameters (baudrate 115200, 8 data bits, 1 stop bit, no parity check)
stty -F /dev/ttyX 115200 cs8 -cstopb -parenb

# after shorting the TX and RX of the serial port, test the loopback transmission
cat /dev/ttyX &
echo "serial_test" > /dev/ttyX
```

## USB Interface

The EC-A8550JD4 provides USB3.0 ports for USB devices such as USB keyboard, mouse and USB flash drive. In addition, the Type-C download port is used for firmware upgrade, please refer to [Upgrade Firmware](qfil_upgrade_firmware.md) and [Upgrade Partition Image](fastboot_upgrade_image.md).

Take the USB flash drive as an example, it can be checked and mounted in the system after inserting:

```bash
# list the recognized storage devices
lsblk

# mount the USB flash drive (the device name is subject to the actual recognition)
mkdir -p /mnt/udisk
mount /dev/sda1 /mnt/udisk

# unmount after use
umount /mnt/udisk
```

## Storage Expansion

The EC-A8550JD4 supports SSD storage expansion. The installation position is shown in the figure below.

<center>

<img alt="" src="../../../qcom_img/EC-A8550JD4/ec-a8550jd4-ssd-en.jpg" width="700">
</center>

## Audio

The EC-A8550JD4 is based on the AIO-8550JD4 motherboard: 1 I2S signal of the motherboard is connected to HDMI for audio playback, and another I2S signal is connected to the codec of the baseboard (3.5mm headphone jack); the audio interfaces actually led out by the device are subject to the actual product.

The low-level playback only supports wav audio with 48K sample rate and 16 bit. The agmplay / agmcap tools are required:

```bash
# set codec record path
amixer -q set "PGAL Select" "Line 2P"
amixer -q set "PGAR Select" "Line 2N"

# set record volume
amixer -q set "ADCL" "200"
amixer -q set "ADCR" "200"

# record for 5 sec
agmcap -D 100 -d 101 -i MI2S-LPAIF-TX-PRIMARY -dkv 0xA3000004 -c 2 -r 48000 -b 16 -T 5 test.wav

# set playback volume
amixer -q set "DACL" "200"
amixer -q set "DACR" "200"

# play audio through headphone
agmplay test.wav -D 100 -d 100 -i MI2S-LPAIF-RX-PRIMARY -dkv 0xA2000001

# play audio through HDMI
agmplay test.wav -D 100 -d 100 -i MI2S-LPAIF-RX-SECONDARY -dkv 0xA2000004
```

On the desktop, you can select the audio output device and adjust the volume in the system sound settings (subject to the audio service supported by the actual firmware).

## Video

The video codec capability of the EC-A8550JD4 is the same as that of the AIO-8550JD4 motherboard, and the default video framework is Gstreamer. You can use `gst-inspect-1.0` to list the available media plugins. The qti plugins such as `qtic2vdec` (Codec2 H.264/H.265/VP8/VP9/MPEG video decoder) and `qtic2venc` (Codec2 H.264/H.265/HEIC video encoder) have hardware acceleration:

```bash
gst-inspect-1.0 --plugin | grep "qti"
```

Codec capability: video decode up to 4K240 / 8K60, with AV1 decode support and native decode support for H.265 Main 10, H.265 Main, H.264 High and VP9 profile 2; video encode up to 4K120 / 8K30, with native encode support for H.265 Main 10, H.265 Main and H.264 High formats; concurrent 4K60 decode and 4K60 encode are supported.

Video playback (using the qtic2vdec plugin, for example, play h264 video):

```bash
export XDG_RUNTIME_DIR=/run/user/root
export WAYLAND_DISPLAY=wayland-1
gst-launch-1.0 filesrc location=/usr/local/test.mp4 ! qtdemux name=demux demux.video_0 ! queue ! h264parse ! qtic2vdec ! videoconvert ! waylandsink sync=true
```

Video encoding (using the qtic2venc plugin, use Ctrl+C to stop encoding):

```bash
gst-launch-1.0 videotestsrc ! qtic2venc ! h264parse ! qtmux ! filesink location=test.mp4 -e
```

## NPU (AI)

The QCS8550 integrates a 48 TOPS NPU (Hexagon DSP). For AI deployment and development, please refer to [AI Tutorial](usage_npu.md).
