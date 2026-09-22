# Hardware Function Usage
## Debug Serial

The EC-A3576JD4 does not expose a debug serial port on the enclosure. You need to open the host and connect an external USB to TTL serial module to the debug serial pin row on the mainboard for debugging.

For the connection and usage of the debug serial port, see: [Debug Serial](usb_to_ttl.md).

## CAN

The AIO-3576JD4 provides 1 CAN interface (Phoenix terminal). When wiring, connect CAN_H to CAN_H and CAN_L to CAN_L.

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

The `candump` and `cansend` tools are included in the SDK and can also be obtained from can-utils (on Ubuntu, install them with `apt update && apt install can-utils`).

Debugging and verification notes:

* The bitrates of both communication ends must be configured consistently, otherwise no frame can be received; you can use `ip -details link show can0` to check the detailed configuration and status of the CAN device.
* If no frame is received after sending, check whether CAN_H and CAN_L on the bus are loose or reversed.

## UART (RS232 / RS485)

The AIO-3576JD4 provides 1 RS232 interface and 1 RS485 interface (both Phoenix terminals).

The serial devices appear as `/dev/ttyS*` nodes in the system (the specific numbers depend on the actual system, which can be checked with `ls /dev/ttyS*`). Taking RS485 as an example, connect the board with a USB to serial adapter (the node of the adapter on the host depends on the actual situation, such as `/dev/ttyUSB0`):

The board sends, the host receives:

```
# Run this on the host terminal first
cat /dev/ttyUSB0
# Run this on the debug serial terminal of the board
echo "firefly uart test..." > /dev/ttySx
```

The host sends, the board receives:

```
# Run this on the debug serial terminal of the board first
busybox stty -echo -F /dev/ttySx       # disable echo
cat /dev/ttySx
# Run this on the host terminal
echo "firefly uart test..." > /dev/ttyUSB0
```

Replace `/dev/ttySx` with the corresponding serial node. The RS232 can be verified in the same way.

## SIM Card

The SIM card slot of the AIO-3576JD4 is used to work with a 4G/5G module for mobile network connection. **The 4G/5G module is optional**. The SIM card function can only work normally after a 4G/5G module is installed inside the chassis. The SIM card insertion direction is shown in the figure below. Please power off the device before inserting or removing the SIM card.

**Note**: The Mini PCIe (4G module) and the PCIe M.2 (5G module) share one USB path and cannot be used at the same time. The default configuration is the 4G module. To use 5G, the position of the resistor on the carrier board needs to be adjusted.

<center>

<img alt="" src="../../../rk3576_img/EC-A3576JD4/sim_insert_direction.png" width="400">
</center>

## Display Interface

The AIO-3576JD4 provides 1 HDMI display output interface, which supports the HDMI2.1 protocol with a maximum resolution of 4K@120Hz.

If the HDMI display fails, first run the following commands to check whether the connection status, edid and resolution are correct:

```
cat /sys/class/drm/card0-HDMI-A-1/status
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
cat /sys/class/drm/card0-HDMI-A-1/modes
```

## Ethernet

The AIO-3576JD4 provides 2 RJ45 Gigabit ports, corresponding to the `eth0` and `eth1` devices in the system.

After the network ports are connected to the network, you can check the IP address through the debug serial port or adb and perform a connectivity test:

```
ifconfig eth0
ping -I eth0 -c 10 www.baidu.com
```
