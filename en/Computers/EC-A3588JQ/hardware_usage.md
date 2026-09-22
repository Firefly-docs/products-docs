# Hardware Function Usage
## Debug Serial

The EC-A3588JQ does not expose a debug serial port on the enclosure. You need to open the host to access the debug serial port on the core board, and connect an external USB to TTL serial module for debugging.

For the connection and usage of the debug serial port, see: [Debug Serial](usb_to_ttl.md).

## CAN

The EC-A3588JQ provides 1 CAN interface. When wiring, connect CAN_H to CAN_H and CAN_L to CAN_L.

The CAN device is recognized as `can0` in the system (the device node is subject to the actual system). The communication test commands are as follows:

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

The EC-A3588JQ provides 2 RS232 interfaces (RS232_0, RS232_1) and 1 RS485 interface. RS232_0 is expanded from the main control UART0, RS232_1 is expanded from the main control UART5, and RS485 is expanded from the main control UART1. It is recommended to use the official FC10 to DB9 serial cable. The pin order of serial cables from different manufacturers may be different, which will cause the serial port communication to fail.

The software nodes corresponding to each hardware interface:

```
RS232_0:  /dev/ttyS0
RS232_1:  /dev/ttyS5
RS485:    /dev/ttyS1
```

### Send and receive verification

Take RS485 (`/dev/ttyS1`) as an example, use the USB to serial adapter of the host computer to connect to the development board (the adapter node on the host side is subject to the actual one, such as `/dev/ttyUSB0`).

The development board sends, and the host receives:

```
# Run on the host terminal first
cat /dev/ttyUSB0
# Run on the debug serial terminal of the development board
echo "firefly RS485 test..." > /dev/ttyS1
```

The host terminal will receive the string `firefly RS485 test...`.

The host sends, and the development board receives:

```
# Run on the debug serial terminal of the development board first
busybox stty -echo -F /dev/ttyS1       # Turn off echo
cat /dev/ttyS1
# Run on the host terminal
echo "firefly RS485 test..." > /dev/ttyUSB0
```

The debug serial terminal of the development board will receive the string `firefly RS485 test...`. The verification methods of RS232_0 and RS232_1 are the same; just replace the nodes with the corresponding `/dev/ttyS0` and `/dev/ttyS5`.

## Display

The EC-A3588JQ provides multiple display output interfaces:

* HDMI2.1: HDMI output interface (software node `hdmi0`), supports the HDMI2.1 protocol, up to 7680x4320@60Hz (8K) output.
* Display Port1.4: Display Port output interface (software node `dp0`), up to 7680x4320@30Hz output.
* VGA: implemented by a DP to VGA conversion chip (software node `dp1`), up to 1080p@60Hz output.
* MIPI-DSI x2: two MIPI DSI output interfaces, both supporting DPHY2.0 and 4 Lane data output, up to 4096x2304@60Hz output (depending on the connected Portx).
* EDP: EDP output interface, which can drive an EDP panel (the supported panels are subject to the actual system).

The RK3588 has four Video output ports (Port0 ~ Port3), and each Video output port is bound to a fixed display controller (for example, Port0 can be used to connect to display controllers such as DP0, DP1, HDMI/eDP0 and HDMI/eDP1). The maximum resolutions that each port can output are as follows:

* Port0 can output up to 7680x4320@60Hz
* Port1 can output up to 4096x2304@60Hz
* Port2 can output up to 4096x2304@60Hz
* Port3 can output up to 1920x1080@60Hz

Notes on multi-screen usage: when the RK3588 does 8K output, it occupies the resources of both Port0 and Port1 at the same time. The default configuration of the SDK is HDMI0 (`hdmi0_in_vp0`) + DP0 (`dp0_in_vp2`). With this configuration, HDMI2.1 (8K) and Display Port (4K@60Hz) can output normally at the same time; do not reconfigure DP0 to vp1, otherwise the DP0 display will be abnormal when HDMI0 outputs 8K.

### Debugging methods

* Set the resolution in the system: on Android, adjust the resolution in `Settings -> Display -> HDMI -> Resolution settings`.
* Get the edid of the HDMI/Display Port (taking HDMI as an example):

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
```

* Get the resolutions supported by the HDMI/Display Port (taking HDMI as an example):

```
cat /sys/class/drm/card0-HDMI-A-1/modes
```

* Get the connection status of the HDMI/Display Port (taking HDMI as an example):

```
cat /sys/class/drm/card0-HDMI-A-1/status
```

* Get the information of the Video Portx currently in use in the system (and the connected display controller):

```
cat /d/dri/0/summary
```

Generally, if you encounter the problem that the HDMI/Display Port cannot display, you need to run the above commands first to check whether the connection status, edid and resolution are correct.

## Ethernet

The EC-A3588JQ provides 2 RJ45 Gigabit network ports, corresponding to the `eth0` and `eth1` devices in the system.

### Using dual Ethernet

For the Android dual Ethernet ports, one is for the external network and the other is for the internal network. The mapping between the primary and secondary ports is as follows:

| Hardware device name | dts node | Android system device name | Primary/secondary relationship |
|---|---|---|---|
| eth1 | gmac0 | Ethernet | Primary port, used for the external network |
| eth0 | gmac1 | Ethernet 2 | Secondary port, used for the internal network |

### Viewing IP addresses and connectivity test

After both network ports are connected to the network, you can view the IP addresses through the debug serial port or adb:

```
ifconfig eth0
ifconfig eth1
```

Connectivity test:

```
ping -I eth0 -c 10 www.baidu.com
ping -I eth1 -c 10 168.168.4.168
```

When testing the internal network port (`eth0`), please modify the target IP according to the actual internal network environment. Under Linux, you can use `ip addr` or `ifconfig` to view the status and IP address of the network ports.

## USB

The USB interfaces of the EC-A3588JQ are as follows:

* 1 x USB3.0 OTG (Type-C): used for firmware upgrade, adb debugging, etc.
* 4 x USB3.0, 3 x USB2.0: Host interfaces, which can be connected to USB keyboards, mice, U disks and other devices, plug and play.

After the USB device is inserted, you can use `lsusb` to check whether it is recognized successfully.

## TF Card

The EC-A3588JQ provides 1 TF Card slot. After inserting a TF card, the system will automatically recognize and mount it (the mount path is subject to the actual system).

## SIM Card

The SIM card slot of the EC-A3588JQ is used together with a 4G/5G module to provide mobile network connectivity. **The 4G/5G module is optional**, and the SIM card function is available only after the module has been installed inside the chassis. The SIM card insertion direction is shown in the figure below. Please power off the device before inserting or removing the SIM card.

<center>

<img alt="" src="../../../rk3588_img/EC-A3588JQ/sim_insert_direction.png" width="400">
</center>
