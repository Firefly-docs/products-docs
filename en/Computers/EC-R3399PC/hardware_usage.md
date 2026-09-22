# Hardware Function Usage
## Debug Serial Port

The EC-R3399PC computer does not expose a debug serial port. You need to open the enclosure to access the debug serial port on the ROC-RK3399-PC motherboard, and connect a USB to TTL serial adapter for debugging. The serial port parameters are: baud rate 1500000, 8 data bits, 1 stop bit, no parity, no flow control.

For the connection and usage of the debug serial port, please refer to: [Debug Serial Port](usb_to_ttl.md).

## Display Interfaces

The EC-R3399PC provides two display output interfaces and supports dual-display:

* HDMI 2.0: 4K@60fps output.
* DP 1.2: output through the USB-C interface, up to 4K@60fps.

For dual-display, the combination of 1 x USB-C (2K output) + 1 x HDMI (4K output) is supported, which can be configured in the display settings of the system.

## Ethernet

The EC-R3399PC provides 1 RJ45 Gigabit Ethernet port, which is recognized as the `eth0` device in the system.

After the Ethernet port is connected to the network, you can check the IP address via the debug serial port or adb:

```
ifconfig eth0
```

Connectivity test:

```
ping -I eth0 -c 10 www.baidu.com
```

## USB Interfaces

The EC-R3399PC provides the following USB interfaces:

* 1 x USB3.0 (current limit 1A)
* 1 x USB2.0 (current limit 500mA)
* 1 x USB-C multi-function interface (USB3.0 OTG data transfer, DP1.2 video output). In OTG mode it is mainly used to connect to a host computer for firmware flashing

## Storage

The EC-R3399PC provides the following storage expansions:

* TF Card: the TF card slot on the panel supports TF card storage expansion. Please power off the device before inserting or removing the card.
* NVMe SSD: the motherboard inside provides 1 x PCIe2.1 (2 Lanes), expandable with a 2242 NVMe SSD, which needs to be installed after opening the enclosure. After installation, the SSD can be formatted and mounted in the system:

```
mkfs.ext4 /dev/nvme0n1
mount /dev/nvme0n1 /mnt/
df -h
```

## Audio

The EC-R3399PC provides a 3.5mm audio jack with MIC recording support. In addition, HDMI audio output and DP1.2 audio output (through the USB-C interface) are supported.

## IR and Buttons

The EC-R3399PC supports the IR remote control function. There are 1 Power button and 1 Recovery button (at the end of the audio jack) on the panel.
