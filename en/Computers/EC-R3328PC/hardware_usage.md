# Hardware Function Usage
## Debug Serial Port

The EC-R3328PC computer does not expose a debug serial port. You need to open the enclosure to access the debug serial port on the ROC-RK3328-PC motherboard, and connect a USB to TTL serial adapter for debugging. The serial port parameters are: baud rate 1500000, 8 data bits, 1 stop bit, no parity, no flow control.

For the connection and usage of the debug serial port, please refer to: [Debug Serial Port](usb_to_ttl.md).

## Display Interfaces

The EC-R3328PC provides two display output interfaces:

* HDMI 2.0: supports 4K@60Hz output, HDCP 1.4/2.2 supported.
* AV interface: CVBS output complying with the 480i and 576i standards, which can be used to connect legacy display devices.

Audio output supports both HDMI audio and AV audio.

## Ethernet

The EC-R3328PC provides 1 RJ45 Gigabit Ethernet port (1000Mbps), which is recognized as the `eth0` device in the system.

After the Ethernet port is connected to the network, you can check the IP address via the debug serial port or adb:

```
ifconfig eth0
```

Connectivity test:

```
ping -I eth0 -c 10 www.baidu.com
```

## USB Interfaces

The EC-R3328PC provides the following USB interfaces:

* 1 x USB3.0 (current limit 1000mA)
* 1 x USB2.0 (current limit 500mA)
* 1 x Type-C (USB3.0 OTG / DC IN 5V/2A). In OTG mode it is mainly used to connect to a host computer for firmware flashing; in power supply mode, use a 5V/2A Type-C power adapter

## TF Card

The EC-R3328PC provides 1 TF card slot for storage expansion. Please power off the device before inserting or removing the TF card.

## IR and Buttons

The EC-R3328PC has a built-in IR receiver and supports the IR remote control function. There is 1 power button on the panel.
