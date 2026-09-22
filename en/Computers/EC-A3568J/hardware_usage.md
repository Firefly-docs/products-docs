# Hardware Function Usage
## Debug Serial

The EC-A3568J does not expose a debug serial port on the enclosure. You need to open the host and connect a USB to TTL serial module to the debug serial port on the mainboard for debugging.

For the connection and usage of the debug serial port, see: [Debug Serial](debug.md).

## CAN

The AIO-3568J provides 1 CAN interface (CAN/CANFD supported). When wiring, connect CAN_H to CAN_H and CAN_L to CAN_L.

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

The rear panel of the AIO-3568J provides 2 RS232 interfaces and 2 RS485 interfaces (terminal blocks). See the interface figures above for their positions. The serial device nodes depend on the actual system and can be listed with `ls /dev/ttyS*`.

### Send/Receive Verification

Take RS485 as an example. Connect the host and the computer with a USB to RS485 adapter (the adapter node on the host depends on the actual device, e.g. `/dev/ttyUSB0`; the RS485 node on the computer depends on the actual system, `/dev/ttyS1` is used as an example below).

The computer sends, the host receives:

```
# Run on the host terminal first
cat /dev/ttyUSB0
# Run on the debug serial terminal of the computer
echo "firefly RS485 test..." > /dev/ttyS1
```

The host terminal will receive the string `firefly RS485 test...`.

The host sends, the computer receives:

```
# Run on the debug serial terminal of the computer first
busybox stty -echo -F /dev/ttyS1       # disable echo
cat /dev/ttyS1
# Run on the host terminal
echo "firefly RS485 test..." > /dev/ttyUSB0
```

The debug serial terminal of the computer will receive the string `firefly RS485 test...`. The RS232 interfaces can be verified in the same way, just replace the node with the corresponding RS232 device node.

## SIM Card

The SIM card slot of the AIO-3568J is used together with a 4G module for mobile network connection. **The 4G module is optional**. The SIM card function can only work after a 4G module is installed inside the chassis. The SIM card insertion direction is shown in the figure below. Please power off the device before inserting or removing the SIM card.

<center>

<img alt="" src="../../../rk356x_img/EC-A3568J/sim_insert_direction.png" width="400">
</center>

## Display Interface

The AIO-3568J provides 1 HDMI 2.0 output interface, supporting up to 4K output. On Android, the resolution and display mode can be adjusted in `Settings -> Display`.

Get the edid, supported modes and connection status of HDMI:

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
cat /sys/class/drm/card0-HDMI-A-1/modes
cat /sys/class/drm/card0-HDMI-A-1/status
```

Generally, if the HDMI display fails, first run the commands above to check whether the connection status, edid and resolution are correct.

## Ethernet

The AIO-3568J provides 2 RJ45 Gigabit Ethernet ports (the WAN port supports PoE), corresponding to the Ethernet devices in the system (such as `eth0` and `eth1`, depending on the actual system).

After both ports are connected to the network, you can check the IP addresses via the debug serial port or adb:

```
ifconfig eth0
ifconfig eth1
```

Connectivity test:

```
ping -I eth0 -c 10 www.baidu.com
ping -I eth1 -c 10 168.168.4.168
```

When testing, please modify the target IP according to the actual network environment. On Linux, you can use `ip addr` or `ifconfig` to check the port status and IP address.

## SATA Drive Installation

The AIO-3568J provides 1 SATA 3.0 interface inside the chassis, supporting 2.5-inch SSD/HDD. The installation is shown below:

<center>

<img alt="" src="../../../rk356x_img/EC-A3568J/ec-a3568j_sata.png" width="700">
</center>
