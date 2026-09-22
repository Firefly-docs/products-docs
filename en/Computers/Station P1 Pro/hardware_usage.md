# Hardware Function Usage
## Debug Serial Port

The Station P1 Pro computer does not expose a debug serial port. You need to open the enclosure to access the debug serial port on the ROC-RK3399-PC Pro motherboard, and connect a USB to TTL serial adapter for debugging. The serial port parameters are: baud rate 1500000, 8 data bits, 1 stop bit, no parity, no flow control.

For the connection and usage of the debug serial port, please refer to: [Debug Serial Port](usb_to_ttl.md).

## Display Interfaces

The Station P1 Pro provides two display output interfaces and supports dual-display:

* HDMI 2.0: 4K@60Hz output.
* DP 1.2: output through the USB-C interface, up to 4K@60Hz.

For dual-display, the combination of DP1.2 (2K output) + HDMI (4K output) is supported, which can be configured in the display settings of the system.

## Ethernet

The Station P1 Pro provides 1 RJ45 Gigabit Ethernet port (1000Mbps), which is recognized as the `eth0` device in the system.

After the Ethernet port is connected to the network, you can check the IP address via the debug serial port or adb:

```
ifconfig eth0
```

Connectivity test:

```
ping -I eth0 -c 10 www.baidu.com
```

## USB Interfaces

The Station P1 Pro provides the following USB interfaces:

* 1 x USB3.0 (current limit 1000mA)
* 1 x USB2.0 (current limit 500mA)
* 1 x USB-C (USB3.0 / OTG / DP1.2). In OTG mode it is mainly used to connect to a host computer for firmware flashing

## Storage

The Station P1 Pro provides the following storage expansions:

* TF Card: the TF card slot on the panel supports TF card storage expansion. Please power off the device before inserting or removing the card.
* NVMe SSD: the motherboard inside provides 1 x PCIe2.1, expandable with a 2242 NVMe SSD, which needs to be installed after opening the enclosure. After installation, the SSD can be formatted and mounted in the system:

```
mkfs.ext4 /dev/nvme0n1
mount /dev/nvme0n1 /mnt/
df -h
```

## Audio

The Station P1 Pro provides a 3.5mm Audio Jack with Mic recording support; the Recovery button is inside the audio jack. In addition, HDMI audio output and DP1.2 audio output (through the USB-C interface) are supported.

## IR and Buttons

The Station P1 Pro supports the IR remote control function. There is 1 Power button on the panel.
