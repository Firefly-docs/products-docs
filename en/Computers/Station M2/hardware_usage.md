# Hardware Function Usage
## Debug Serial

The Station M2 does not expose a debug serial port on the enclosure. You need to open the host and connect a USB to TTL serial module to the debug serial port on the mainboard for debugging.

For the connection and usage of the debug serial port, see: [Debug Serial](debug.md).

## Display Interface

The Station M2 provides 1 HDMI 2.0 output interface, supporting up to 4K@60Hz output. On Android, the resolution and display mode can be adjusted in `Settings -> Display`.

Get the edid, supported modes and connection status of HDMI:

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
cat /sys/class/drm/card0-HDMI-A-1/modes
cat /sys/class/drm/card0-HDMI-A-1/status
```

Generally, if the HDMI display fails, first run the commands above to check whether the connection status, edid and resolution are correct.

## Ethernet

The Station M2 provides 1 RJ45 Gigabit Ethernet port, corresponding to the Ethernet device in the system (such as `eth0`, depending on the actual system).

After the port is connected to the network, you can check the IP address via the debug serial port or adb:

```
ifconfig eth0
```

Connectivity test:

```
ping -I eth0 -c 10 www.baidu.com
```

When testing, please modify the target domain/IP according to the actual network environment. On Linux, you can use `ip addr` or `ifconfig` to check the port status and IP address.

## USB Interfaces

The Station M2 provides 1 USB 3.0 interface and 1 USB 2.0 interface for USB flash drives, USB keyboards/mice and other devices. After a device is plugged in, you can check whether it is recognized with `lsusb`:

```
lsusb
```

In addition, the Type-C interface on the panel is a DC 5V power supply / OTG shared interface: it is used as the power input by default, and can be used as a USB OTG interface for adb/OTG scenarios (the multiplexing method depends on the actual hardware design).

## M.2 SSD Expansion

The Station M2 provides an M.2 slot inside the chassis for large-capacity SSD expansion. Open the enclosure, insert the SSD in the direction of the M.2 slot silkscreen and fix it. You can check the recognized drives in the system with `lsblk`:

```
lsblk
```
