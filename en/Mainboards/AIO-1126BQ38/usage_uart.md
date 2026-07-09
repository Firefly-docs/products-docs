# UART

## Introduction

AIO-1126BQ38 supports UART3 and UART5 interfaces. UART3 is used as RS485, and UART5 is used as RS232.
## DTS Configuration

File path: `kernel-6.1/arch/arm64/boot/dts/rockchip/rv1126b-firefly-aio-1126bq38.dts`

```dts
#ifdef FF_UART3
&uart3 {
        pinctrl-0 = <&uart3m1_xfer_pins>;
        status = "okay";
};
#endif

#ifdef FF_UART5
&uart5 {
        pinctrl-0 = <&uart5m1_xfer_pins>;
        status = "okay";
};
#endif
```

After configuring the serial port, the nodes corresponding to the hardware interface on the software are:

```
UART3:  /dev/ttyS3	RS485
UART5:  /dev/ttyS5	RS232
```

## Send And Receive Verification

Users can use different host USB-to-serial adapters to send and receive data to and from the development board's serial port, depending on the interface. For example, the debugging steps for UART5 are as follows:

(1) Development board sends, host receives

```bash
# Host terminal executes first:
# /dev/ttyUSB0 is the node for the host serial port adapter; modify it according to your actual setup. Set the baud rate to 115200.
stty -F /dev/ttyUSB0 115200 cs8 -cstopb -parenb -icanon -echo
cat /dev/ttyUSB0

# Development board debug serial port terminal executes:
# /dev/ttyS5 is the UART5 node. The baud rate is set to 115200.
stty -F /dev/ttyS5 115200 cs8 -cstopb -parenb -icanon -echo
echo "firefly UART5 test..." > /dev/ttyS5
```

The host terminal can receive the string "firefly UART5 test..."

(2) Host sends, development board receives

```
# Development board debug serial port terminal executes first:
# /dev/ttyS5 is the UART5 node. The baud rate is set to 115200.
stty -F /dev/ttyS5 115200 cs8 -cstopb -parenb -icanon -echo
cat /dev/ttyS5

# Host terminal execution:
# /dev/ttyUSB0 is the node for the host serial port adapter; modify it according to your actual setup. Set the baud rate to 115200.
stty -F /dev/ttyUSB0 115200 cs8 -cstopb -parenb -icanon -echo
echo "firefly UART5 test..." > /dev/ttyUSB0
```

The development board debugging serial port terminal can receive the string "firefly UART5 test..."

## RS485

GPIO0_B0 is used as an RS485 control pin for transmitting and receiving. Pulling this pin high enables transmitting, and pulling it low enables receiving. Users can operate it through the `/sys/class/gpio` subsystem.

* Below is the configuration of GPIO0_B0 for output:

```bash
echo 8 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio8/direction

```

* Pulling GPIO0_B0 high, RS485 transmitting: echo 1 > /sys/class/gpio/gpio8/value
* Pulling GPIO0_B0 low, RS485 receiving: echo 0 > /sys/class/gpio/gpio8/value
