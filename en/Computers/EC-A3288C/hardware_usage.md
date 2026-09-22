# Hardware Function Usage
## Debug Serial Port

The debug serial port of the EC-A3288C is UART2 (TTL level), corresponding to the system device node `/dev/ttyS2`. The serial port parameters are: baud rate 115200, 8 data bits, 1 stop bit, no parity, no flow control.

The debug serial port is a TTL-level interface. An external USB to TTL serial module is required for debugging.

For the connection and usage of the debug serial port, please refer to: [Debug Serial Port](usb_to_ttl.md).

## UART (RS485 / RS232)

The EC-A3288C expands the enhanced function serial ports through the SPI bridge, where RS485 is an RS485-level interface and RS232 is an RS232-level interface. The kernel enables the above serial ports by default. The software nodes corresponding to each interface are as follows:

```
RS485:              /dev/ttyS1
RS232:              /dev/ttyS3
UART2 (debug port): /dev/ttyS2
```

**Note: RS232 requires an RS232 crossover cable, otherwise the rx and tx cannot successfully send and receive data.**

Take RS485 as an example, the debugging steps are as follows:

### Connect the hardware

Connect the A, B, and GND pins of RS485 to the A, B, and GND pins of the host serial port adapter (USB to 485 serial module) respectively.

### Open the serial terminal of the host

Open kermit in the terminal and set the baud rate:

```bash
$ sudo kermit
C-Kermit> set line /dev/ttyUSB*
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

`/dev/ttyUSB*` is the device file of the USB to serial port adapter recognized by the host, subject to the actual system.

### Send and receive verification

The device sends and the host receives. The device file of RS485 is `/dev/ttyS1`. Run the following command on the device:

```bash
echo firefly RS485 test… > /dev/ttyS1
```

The serial terminal of the host will receive the string "firefly RS485 test…".

The host sends and the device receives. First run the following command on the device:

```bash
cat /dev/ttyS1
```

Then enter the string "firefly RS485 test…" in the serial terminal of the host, and the same string can be seen on the device side. The verification method of RS232 is the same, just replace the node with `/dev/ttyS3`.

**Note:** The debug serial port UART2 can short-circuit rx/tx for a loopback communication test. Since RS232 does not support loopback sending and receiving data in hardware, it can only communicate with other hosts.

## Display Interface

The EC-A3288C supports dual-screen identical/differential display. The SDK provides the firmware configuration for LVDS display output (`aio-3288c-lvds-ubuntu.mk`, `aio-3288c-lvds-buildroot.mk`). The supported resolutions and interface forms are subject to the actual system.

## Other Interfaces

The usage of other external interfaces (Ethernet, USB, etc.) of the EC-A3288C is to be supplemented, subject to the actual system.
