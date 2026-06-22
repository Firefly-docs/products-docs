# UART

## Introduction

The ICORE-1126BQ38 motherboard provides an RS485 interface.

## DTS Configuration

File path: `kernel-6.1/arch/arm64/boot/dts/rockchip/rv1126b-firefly-cam-1126bq38.dtsi`

```dts
#ifdef FF_RS485
&uart3 {
        pinctrl-names = "default";
        pinctrl-0 = <&uart3m1_xfer_pins>;
        status = "okay";
};
#endif
```
After configuring the serial port, the nodes corresponding to the hardware interface on the software are:
```
UART3:  /dev/ttyS3        RS485
```

## Send And Receive Verification

Users can use different USB to RS485 adapters from the host computer to send and receive data to and from the development board's RS485 interface, depending on the interface used. For example, the debugging steps for UART3 are as follows:

(1) Development board sends, host receives

```
# Host terminal executes first:
# /dev/ttyUSB0 is the node for the host's serial port adapter; modify this according to your actual setup. The baud rate is set to 115200.
stty -F /dev/ttyUSB0 115200 cs8 -cstopb -parenb -icanon -echo
cat /dev/ttyUSB0

# Development board debug serial port terminal executes:
# /dev/ttyS3 is the UART3 RS485 node. The baud rate is set to 115200.
stty -F /dev/ttyS3 115200 cs8 -cstopb -parenb -icanon -echo
echo "firefly UART3 test..." > /dev/ttyS3
```

The host terminal can receive the string "firefly UART3 test..."

(2) Host sends, development board receives

```
# Development board debug serial port terminal executes first:
# /dev/ttyS3 is the UART3 RS485 node. The baud rate is set to 115200.
stty -F /dev/ttyS3 115200 cs8 -cstopb -parenb -icanon -echo
cat /dev/ttyS3

# Host terminal execution:
# /dev/ttyUSB0 is the node for the host's serial port adapter; modify this according to your actual setup. The baud rate is set to 115200.
stty -F /dev/ttyUSB0 115200 cs8 -cstopb -parenb -icanon -echo
echo "firefly UART3 test..." > /dev/ttyUSB0
```

The development board debugging serial port terminal can receive the string "firefly UART3 test..."
