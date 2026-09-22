# Hardware Function Usage
## Debug Serial Port

The AIBOX-8550 uses an on-board USB serial solution (the serial-to-USB chip is PL2303GL). You can debug the device by connecting the Console port of the device with a USB cable directly, no external USB to TTL serial module is required.

Serial port parameters: baudrate 115200, 8 data bits, 1 stop bit, no parity check.

For the connection method and Windows driver installation, please refer to: [Debug Console](debug.md).

## Display Interface

The AIBOX-8550 provides 1 HDMI 2.0 display output interface. The device runs Ubuntu 22.04 (Wayland + Weston desktop) by default, and the desktop will be displayed after connecting a monitor.

## Ethernet

The AIBOX-8550 provides 2 x 1000M RJ45 Ethernet ports, corresponding to the `eth0` and `eth1` devices in the system (subject to the actual system). After the Ethernet port is connected to the network, you can log in to the device through the debug serial port or SSH to check the IP address and test the connectivity:

```
ifconfig eth0
ping -I eth0 -c 10 www.baidu.com
```

## USB Interface

The AIBOX-8550 provides 2 x USB3.0 ports for USB keyboard, mouse, USB flash drive, etc. In addition, 1 Type-C port is used as the firmware download port. For firmware upgrade, please refer to [Upgrade Firmware](qfil_upgrade_firmware.md) and [Upgrade Partition Image](fastboot_upgrade_image.md).

## TF Card

The AIBOX-8550 provides 1 TF card slot for storage expansion.

## Audio

The AIBOX-8550 has 1 I2S signal for HDMI audio playback, and the audio of the device is output through HDMI.

The low-level playback only supports wav audio with 48K sample rate and 16 bit. The agmplay tool is required:

```bash
# play audio to HDMI
agmplay test.wav -D 100 -d 100 -i MI2S-LPAIF-RX-SECONDARY -dkv 0xA2000004
```

On the desktop, you can select the audio output device and adjust the volume in the system sound settings (subject to the audio service supported by the actual firmware).

## Video

The default video framework of the AIBOX-8550 is Gstreamer. You can use `gst-inspect-1.0` to list the available media plugins. The qti plugins such as `qtic2vdec` (Codec2 H.264/H.265/VP8/VP9/MPEG video decoder) and `qtic2venc` (Codec2 H.264/H.265/HEIC video encoder) have hardware acceleration:

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
