# Hardware Function Usage
## Debug Serial

The AIO-3588SJD4-AI does not expose a debug serial port on the enclosure. You need to open the host before you can access the debug serial port on the mainboard, and connect an external USB to TTL serial module for debugging.

For the connection and usage of the debug serial port, see: [Debug Serial](usb_to_ttl.md).

## CAN

The AIO-3588SJD4-AI provides 1 CAN interface. When wiring, connect CAN_H to CAN_H and CAN_L to CAN_L.

The CAN device is recognized as `can0` in the system. The communication test commands are as follows:

```
# Shut down the can0 device
ip link set can0 down
# Set the bitrate to 250Kbps
ip link set can0 type can bitrate 250000
# Bring up the can0 device
ip link set can0 up
# Run candump on the receiving end, blocking and waiting for frames
candump can0
# Run cansend on the sending end to send a frame
cansend can0 123#1122334455667788
```

The `candump` and `cansend` tools are included in the SDK, and can also be obtained from can-utils.

Debugging and verification notes:

* The bitrates of both communication ends must be configured to be the same, otherwise no frames can be received; you can use `ip -details link show can0` to view the detailed configuration and status of the CAN device.
* If no frames are received after sending, please check whether the bus CAN_H and CAN_L are loose or reversed.

## UART (RS232 / RS485)

The AIO-3588SJD4-AI provides 1 RS232 interface and 1 RS485 interface. RS232 is expanded from the main control UART9, and RS485 is expanded from the main control UART6. The DTS configuration is located in `kernel/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588sjd4-ai.dtsi`.

The software nodes corresponding to each hardware interface (subject to the actual system):

```
RS232:  /dev/ttyS9
RS485:  /dev/ttyS6
```

Usage notes:

* The pin GPIO1_A2 is used for the send/receive control of RS485. The pin pulled high means sending, and pulled low means receiving. It can be controlled through the `/sys/class/gpio` subsystem (the GPIO number is subject to the actual system).
* The default baud rate of the serial port is 9600, with 8 data bits, 1 stop bit and no flow control.

### Send and receive verification

Take RS485 (`/dev/ttyS6`) as an example, use the USB to serial adapter of the host computer to connect to the device (the adapter node on the host side is subject to the actual one, such as `/dev/ttyUSB0`).

The device sends, and the host receives:

```
# Run on the host terminal first
cat /dev/ttyUSB0
# Run on the debug serial terminal of the device
echo "firefly RS485 test..." > /dev/ttyS6
```

The host terminal will receive the string `firefly RS485 test...`.

The host sends, and the device receives:

```
# Run on the debug serial terminal of the device first
busybox stty -echo -F /dev/ttyS6       # Turn off echo
cat /dev/ttyS6
# Run on the host terminal
echo "firefly RS485 test..." > /dev/ttyUSB0
```

The debug serial terminal of the device will receive the string `firefly RS485 test...`. The verification method of RS232 is the same; just replace the node with the corresponding `/dev/ttyS9`.

## Display

The AIO-3588SJD4-AI provides 1 HDMI2.1 display output interface, which supports the HDMI2.1 protocol and outputs up to 7680x4320@60Hz (8K).

The RK3588 has four Video output ports (Port0 ~ Port3), and each Video output port is bound to a fixed display controller (for example, Port0 can be used to connect to display controllers such as DP0, DP1, HDMI/eDP0 and HDMI/eDP1). The maximum resolutions that each port can output are as follows:

* Port0 can output up to 7680x4320@60Hz
* Port1 can output up to 4096x2304@60Hz
* Port2 can output up to 4096x2304@60Hz
* Port3 can output up to 1920x1080@60Hz

### Debugging methods

* Set the resolution in the system: on Android, adjust the resolution in `Settings -> Display -> HDMI -> Resolution settings`.
* Get the edid of the HDMI:

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
```

* Get the resolutions supported by the HDMI:

```
cat /sys/class/drm/card0-HDMI-A-1/modes
```

* Get the connection status of the HDMI:

```
cat /sys/class/drm/card0-HDMI-A-1/status
```

* Get the information of the Video Portx currently in use in the system (and the connected display controller):

```
cat /d/dri/0/summary
```

Generally, if you encounter the problem that the HDMI cannot display, you need to run the above commands first to check whether the connection status, edid and resolution are correct.

## Ethernet

The AIO-3588SJD4-AI provides 1 RJ45 network port (support 1Gbps), corresponding to the `eth0` device in the system (the device number is subject to the actual system).

After the network port is connected to the network, you can view the IP address through the debug serial port or adb:

```
ifconfig eth0
```

Connectivity test:

```
ping -I eth0 -c 10 www.baidu.com
```

## USB Interfaces

The USB interfaces of the AIO-3588SJD4-AI include:

* 2 x USB3.0: used to connect USB peripherals.
* 1 x USB2.0: led out through the pin header.
* 1 x USB Type-C: download/debug interface, mainly used for firmware flashing and system debugging.

## TF Card

The AIO-3588SJD4-AI provides 1 TF card slot. Please power off the device before inserting or removing the card. The card insertion direction is subject to the silkscreen of the card slot.

## SIM Card

The SIM card slot of the AIO-3588SJD4-AI is used together with a 4G module to provide mobile network connectivity; WIFI and Bluetooth are provided by external modules. **The 4G module is optional**, and the SIM card function is available only after the module has been installed inside the chassis (subject to the actual configuration). The SIM card insertion direction is shown in the figure below. Please power off the device before inserting or removing the SIM card.

<center>

<img alt="" src="../../../rk3588_img/EC-A3588SJD4-AI/sim_insert_direction.png" width="400">
</center>

## Audio Interfaces

The AIO-3588SJD4-AI provides Line-Out, Line-In and Headphone interfaces, which can be used to connect audio devices such as power amplifiers, external audio sources and headphones respectively.

## NPU Usage

The RK3588S has a built-in NPU with computing power up to 6 TOPS, supporting INT4/INT8/INT16 hybrid computing. To use the NPU, you need to deploy models through the RKNN SDK. After converting the algorithm model to the `.rknn` format, it can run on the device. For details, please refer to: [NPU Usage](usage_npu.md).

* [Device Tree Manual](linux_dts_manual.md)
