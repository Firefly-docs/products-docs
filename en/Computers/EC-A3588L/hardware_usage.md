# Hardware Function Usage
## Debug Serial

The EC-A3588L does not expose a debug serial port on the enclosure. You need to open the host to access the debug serial port on the core board, and connect an external USB to TTL serial module for debugging.

For the connection and usage of the debug serial port, see: [Debug Serial](usb_to_ttl.md).

## CAN

The AIO-3588L provides 1 CAN interface. When wiring, connect CAN_H to CAN_H and CAN_L to CAN_L.

Note that in the SDK of the AIO-3588L core board, CAN1 is multiplexed with UART3, and the default firmware is configured as UART3. To use the CAN function, you need to configure `CAN1_OR_UART3` to 1 in the device tree to enable CAN1 (see the core board Wiki for details). The CAN device node is subject to the actual system.

The CAN communication test commands are as follows (taking `can0` as an example, please replace it with the actual node):

```
# Shut down the CAN device
ip link set can0 down
# Set the bitrate to 250Kbps
ip link set can0 type can bitrate 250000
# Bring up the CAN device
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

The AIO-3588L provides 2 RS232 interfaces and 1 RS485 interface externally. In the SDK of the AIO-3588L core board, the enabled serial ports are UART3, UART6, UART7 and UART8, and the corresponding software nodes are as follows (the mapping between the RS232/RS485 interfaces and each UART is subject to the actual system):

```
UART3:  /dev/ttyS3 (multiplexed with CAN1, configured as UART3 by default)
UART6:  /dev/ttyS6
UART7:  /dev/ttyS7
UART8:  /dev/ttyS8
```

### Send and receive verification

The simplest way of verification is to short-circuit the TX and RX pins of the serial port for a self-loopback test, and run the following commands on the debug serial or adb terminal (taking `/dev/ttyS7` as an example, the node is subject to the actual system):

```
busybox stty -echo -F /dev/ttyS7          # Turn off echo
cat /dev/ttyS7 &                          # Get the input string of /dev/ttyS7 in the background
echo "firefly uart test..." > /dev/ttyS7  # Send the string
```

The terminal will receive the string `firefly uart test...`. The RS232/RS485 can also be verified with the USB to serial adapter of the host computer: one end is connected to the host, and the other end is connected to the serial port of the development board. The sending and receiving methods are the same as above.

## Display

The AIO-3588L provides multiple display output interfaces:

* HDMI2.1: HDMI output interface (software node `hdmi0`), supports the HDMI2.1 protocol, up to 7680x4320@60Hz (8K) output.
* HDMI2.0: HDMI output interface (software node `hdmi1`), supports the HDMI2.0 protocol, up to 4096x2304@60Hz output.
* Display Port1.4: Display Port output interface (software node `dp0`), up to 7680x4320@30Hz output.
* MIPI-DSI x2: two MIPI DSI output interfaces, up to 4096x2304@60Hz output (depending on the connected Portx).

The RK3588 has four Video output ports (Port0 ~ Port3), and each Video output port is bound to a fixed display controller (for example, Port0 can be used to connect to display controllers such as DP0, DP1, HDMI/eDP0 and HDMI/eDP1). The maximum resolutions that each port can output are as follows:

* Port0 can output up to 7680x4320@60Hz
* Port1 can output up to 4096x2304@60Hz
* Port2 can output up to 4096x2304@60Hz
* Port3 can output up to 1920x1080@60Hz

Notes on multi-screen usage:

* Currently, one Portx can only output one resolution format at the same time. If multiple display interfaces with different resolutions are configured on the same Portx, only one of them can be used at the same time.
* When the RK3588 does 8K output, it occupies the resources of both Port0 and Port1 at the same time. For example, when HDMI0 (connected to Port0) does 8K output, the display of HDMI1 (connected to Port1) will be abnormal; only when Port0 outputs less than or equal to 4K@60Hz can Port1 output 4K@60Hz normally.

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

The AIO-3588L provides 2 RJ45 Gigabit network ports, corresponding to the `eth0` and `eth1` devices in the system.

### Viewing IP addresses and connectivity test

After the network ports are connected to the network, you can view the IP addresses through the debug serial port or adb:

```
ifconfig eth0
ifconfig eth1
```

Connectivity test:

```
ping -I eth0 -c 10 www.baidu.com
ping -I eth1 -c 10 168.168.4.168
```

Please modify the target IP according to the actual network environment. Under Linux, you can use `ip addr` or `ifconfig` to view the status and IP address of the network ports; under Android, the primary/secondary relationship of the dual Ethernet ports (internal/external network) is subject to the actual system.

## USB

The USB interfaces of the AIO-3588L are as follows:

* 1 x USB3.0 OTG (Type-C): used for firmware upgrade, adb debugging, etc.
* 4 x USB3.0, 3 x USB2.0: Host interfaces, which can be connected to USB keyboards, mice, U disks and other devices, plug and play.

After the USB device is inserted, you can use `lsusb` to check whether it is recognized successfully.

## TF Card

The AIO-3588L provides 1 TF Card slot. After inserting a TF card, the system will automatically recognize and mount it (the mount path is subject to the actual system).

## SIM Card

The SIM card slot of the AIO-3588L is used together with a 4G/5G module to provide mobile network connectivity. **The 4G/5G module is optional**, and the SIM card function is available only after the module has been installed inside the chassis. The SIM card insertion direction is shown in the figure below. Please power off the device before inserting or removing the SIM card.

<center>

<img alt="" src="../../../rk3588_img/EC-A3588L/sim_insert_direction.png" width="400">
</center>
