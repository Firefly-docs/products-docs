# Hardware Function Usage
## Debug Serial

The Station P2 does not expose a debug serial port on the enclosure. You need to open the host and connect a USB to TTL serial module to the debug serial port on the mainboard for debugging.

For the connection and usage of the debug serial port, see: [Debug Serial](debug.md).

## UART (RS232 / RS485)

The Station P2 panel provides 1 Control Port, which expands RS232 x 2 and RS485 x 1 signals. The signal definition is as follows:

```
1. RS232_TX3    2. RS232_RX3
3. RS232_TX2    4. GND
5. GND          6. RS232_RX2
7. RS485_A      8. RS485_B
```

The serial device nodes depend on the actual system and can be listed with `ls /dev/ttyS*`.

### Send/Receive Verification

Take RS485 (`RS485_A`/`RS485_B`) as an example. Connect the host and the computer with a USB to RS485 adapter (the adapter node on the host depends on the actual device, e.g. `/dev/ttyUSB0`; the RS485 node on the computer depends on the actual system, `/dev/ttyS1` is used as an example below).

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

The debug serial terminal of the computer will receive the string `firefly RS485 test...`. The RS232 signals can be verified in the same way, just replace the node with the corresponding RS232 device node.

## Display Interface

The Station P2 provides 1 HDMI 2.0 output interface, supporting up to 4K@60Hz output. On Android, the resolution and display mode can be adjusted in `Settings -> Display`.

Get the edid, supported modes and connection status of HDMI:

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
cat /sys/class/drm/card0-HDMI-A-1/modes
cat /sys/class/drm/card0-HDMI-A-1/status
```

Generally, if the HDMI display fails, first run the commands above to check whether the connection status, edid and resolution are correct.

## Ethernet

The Station P2 provides 2 RJ45 Gigabit Ethernet ports, corresponding to the `eth0` and `eth1` devices in the system.

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

## USB Interfaces

The Station P2 provides 1 USB 3.0 interface and 2 USB 2.0 interfaces for USB flash drives, USB keyboards/mice and other devices. After a device is plugged in, you can check whether it is recognized with `lsusb`:

```
lsusb
```

In addition, the USB-C interface on the panel is an OTG interface, which can be used to connect to a host for adb debugging or firmware upgrade.

## SATA Drive Installation

The Station P2 provides 1 SATA 3.0 interface inside the chassis, supporting 2.5-inch, 7mm thick HDD/SSD. The installation is shown below:

<center>

<img alt="" src="../../../rk356x_img/Station-P2/station_p2_zh_Install.png" width="700">
</center>
