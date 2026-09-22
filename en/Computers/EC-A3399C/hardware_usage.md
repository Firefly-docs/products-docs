# Hardware Function Usage
## Debug Serial Port

The EC-A3399C computer does not expose a debug serial port. You need to open the enclosure to access the debug serial port on the AIO-3399C motherboard, and connect a USB to TTL serial adapter for debugging. The serial port parameters are: baud rate 1500000, 8 data bits, 1 stop bit, no parity, no flow control.

For the connection and usage of the debug serial port, please refer to: [Debug Serial Port](usb_to_ttl.md).

## Display Interfaces

The AIO-3399C supports two display output interfaces:

* HDMI 2.0: supports up to 4K@60Hz output.
* DisplayPort 1.2: supports up to 4K@60Hz output.

Mirrored or extended dual-display configurations are supported, which can be configured in the display settings of the system.

## Ethernet

The AIO-3399C provides 1 RJ45 Gigabit Ethernet port, which is recognized as the `eth0` device in the system.

After the Ethernet port is connected to the network, you can check the IP address via the debug serial port or adb:

```
ifconfig eth0
```

Connectivity test:

```
ping -I eth0 -c 10 www.baidu.com
```

## USB Interfaces

The AIO-3399C provides the following USB interfaces:

* 1 x USB3.0 (rear panel)
* 2 x USB2.0 (dual-stacked USB recepts on the front panel), for USB devices such as mice and keyboards
* 1 x Type-C (OTG), mainly used to connect to a host computer for firmware flashing

## Serial Ports (RS232 / RS485)

The front panel of the AIO-3399C provides 1 RS232 interface (DB9) and 1 RS485 interface (terminal block), which are commonly used to connect industrial equipment:

* RS232: connect RXD, TXD and GND according to the silkscreen of the DB9 connector.
* RS485: when wiring, connect A to A and B to B. If communication fails, check whether the A/B wires are loose or reversed.

For a simple test, you can short-circuit the transmit and receive pins of RS232 (short-circuit A and B for RS485), then run `cat`/`echo` on the corresponding serial device node in the system to send and receive data (the device node name depends on the actual system). The motherboard also has 2 on-board TTL UARTs, which are available after opening the enclosure.

## TF Card

The rear panel of the AIO-3399C provides 1 TF card slot for storage expansion. Please power off the device before inserting or removing the TF card.

## SIM Card

The SIM card slot of the AIO-3399C is used together with a Mini PCIe 3G/4G module to provide mobile network connectivity. **The 3G/4G module is optional**, and the SIM card function is available only after the module has been installed inside the chassis. The SIM card insertion direction is shown in the figure below. Please power off the device before inserting or removing the SIM card.

<center>

<img alt="" src="../../../rk3399_img/EC-A3399C/sim_insert_direction.png" width="400">
</center>

## Audio

The front panel of the AIO-3399C provides a 3.5mm Phone jack for audio output. The motherboard also provides LINE-IN, LINE-OUT, and SPEAKER audio interfaces (available after opening the enclosure).

## IR

The front panel of the AIO-3399C has a built-in IR receiver and supports the IR remote control function.
