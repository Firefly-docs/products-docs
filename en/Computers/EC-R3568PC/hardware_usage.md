# Hardware Function Usage
## Debug Serial

The EC-R3568PC does not expose a debug serial port on the enclosure. You need to open the host and connect a USB to TTL serial module to the debug serial port on the mainboard for debugging.

For the connection and usage of the debug serial port, see: [Debug Serial](debug.md).

## UART (RS232 / RS485)

The ROC-RK3568-PCSE panel provides 1 RJ45 Control Port, which expands RS232 and RS485 signals. See the interface figure above for the signal definition. The serial device nodes depend on the actual system and can be listed with `ls /dev/ttyS*`.

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

The debug serial terminal of the computer will receive the string `firefly RS485 test...`. The RS232 signals can be verified in the same way, just replace the node with the corresponding RS232 device node.

## SIM Card

The SIM card slot of the ROC-RK3568-PCSE is used together with a 4G module for mobile network connection. **The 4G module is optional**. The SIM card function can only work after a 4G module is installed inside the chassis. The SIM card insertion direction is shown in the figure below. Please power off the device before inserting or removing the SIM card.

<center>

<img alt="" src="../../../rk356x_img/EC-R3568PC/sim_insert_direction.png" width="400">
</center>

## Display Interface

The ROC-RK3568-PCSE provides 1 HDMI 2.0 output interface, supporting up to 4K@60Hz output. On Android, the resolution and display mode can be adjusted in `Settings -> Display`.

Get the edid, supported modes and connection status of HDMI:

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
cat /sys/class/drm/card0-HDMI-A-1/modes
cat /sys/class/drm/card0-HDMI-A-1/status
```

Generally, if the HDMI display fails, first run the commands above to check whether the connection status, edid and resolution are correct.

## Ethernet

The ROC-RK3568-PCSE provides 2 RJ45 Gigabit Ethernet ports, corresponding to the `eth0` and `eth1` devices in the system.

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

The ROC-RK3568-PCSE provides 1 USB 3.0 interface (Max 1A) and 2 USB 2.0 interfaces (Max 500mA) for USB flash drives, USB keyboards/mice and other devices. After a device is plugged in, you can check whether it is recognized with `lsusb`:

```
lsusb
```

In addition, the USB-C interface on the panel is an OTG interface, which can be used to connect to a host for adb debugging or firmware upgrade.
