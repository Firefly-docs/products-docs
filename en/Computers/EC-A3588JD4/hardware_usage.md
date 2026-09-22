# Hardware Function Usage
## Debug Serial

The EC-A3588JD4 does not expose a debug serial port on the enclosure. You need to open the host to access the debug serial port on the core board, and connect an external USB to TTL serial module for debugging.

For the connection and usage of the debug serial port, see: [Debug Serial](usb_to_ttl.md).

## CAN

The AIO-3588JD4 provides 1 CAN interface. When wiring, connect CAN_H to CAN_H and CAN_L to CAN_L.

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

The AIO-3588JD4 provides 1 RS232 interface and 1 RS485 interface. The RS232 is expanded from the main control UART1, and the RS485 is expanded from the main control UART6.

The software nodes corresponding to each hardware interface:

```
RS232:  /dev/ttyS1
RS485:  /dev/ttyS6
```

Note that RS485 is half-duplex communication. Before sending, the transmit/receive direction needs to be controlled by GPIO (the GPIO number is subject to the actual system; the following example uses GPIO34).

### Send and receive verification

Use the USB to serial adapter of the host computer to connect to the development board (the adapter node on the host side is subject to the actual one, such as `/dev/ttyUSB0`).

The development board sends, and the host receives:

```
# Run on the host terminal first
cat /dev/ttyUSB0
# Run on the debug serial terminal of the development board
echo "firefly RS485 test..." > /dev/ttyS6
```

The host terminal will receive the string `firefly RS485 test...`.

The host sends, and the development board receives:

```
# Run on the debug serial terminal of the development board (configure GPIO34 as output and pull it low to switch to the receive direction)
echo 34 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio34/direction
echo 0 > /sys/class/gpio/gpio34/value

# Run on the debug serial terminal of the development board first
busybox stty -echo -F /dev/ttyS6       # Turn off echo
cat /dev/ttyS6
# Run on the host terminal (pull it high to switch to the transmit direction)
echo 1 > /sys/class/gpio/gpio34/value
echo "firefly RS485 test..." > /dev/ttyUSB0
```

The debug serial terminal of the development board will receive the string `firefly RS485 test...`. The verification method of RS232 is the same; just replace the node with the corresponding `/dev/ttyS1`. RS232 is full-duplex and does not need GPIO to control the transmit/receive direction.

## Display

The AIO-3588JD4 provides one HDMI display output interface:

* HDMI2.1: HDMI output interface, supports the HDMI2.1 protocol (software node `hdmi0`), up to 7680x4320@60Hz (8K) output.

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

The AIO-3588JD4 provides 2 RJ45 Gigabit network ports, corresponding to the `eth0` and `eth1` devices in the system.

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

The USB interfaces of the AIO-3588JD4 are as follows:

* 1 x USB Type-C (download/Debug): used for firmware upgrade, adb debugging, etc.
* 2 x USB3.0: Host interfaces, which can be connected to USB keyboards, mice, U disks and other devices, plug and play.

After the USB device is inserted, you can use `lsusb` to check whether it is recognized successfully.

## TF Card

The AIO-3588JD4 provides 1 TF Card slot. After inserting a TF card, the system will automatically recognize and mount it (the mount path is subject to the actual system).

## SIM Card

The SIM card slot of the AIO-3588JD4 is used together with a 4G/5G module to provide mobile network connectivity. **The 4G/5G module is optional**, and the SIM card function is available only after the module has been installed inside the chassis. The SIM card insertion direction is shown in the figure below. Please power off the device before inserting or removing the SIM card.

<center>

<img alt="" src="../../../rk3588_img/EC-A3588JD4/sim_insert_direction.png" width="400">
</center>
