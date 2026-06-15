# UART

## Introduction

ROC-RK3588S-PC has one RS232 interfaces and one RS485 interface

The serial interface diagram of the ROC-RK3588S-PC development board is as follows:

![](../../../rk3588_img/Station-M3/usage_uart_interface.jpg)

## DTS configuration
The RS232 interface of the development board is extended by the main control UART1, and the RS485 interface is extended by the main control UART6.

File path `kernel/arch/arm64/boot/dts/rockchip/aio-3588jd4.dtsi`
```
/* RS485 */
&uart6{
        pinctrl-0 = <&uart6m1_xfer>;
        status = "okay";
};

/* RS232 */
&uart1{
        pinctrl-0 = <&uart1m1_xfer>;
        status = "okay";
};

```
After configuring the serial port, the nodes on the software corresponding to the hardware interface are:

```
RS232:   /dev/ttyS1
RS485：  /dev/ttyS6
```

## Send and receive verification

Users can use different host's USB-to-serial adapters to send and receive data to the serial port of the development board according to different interfaces. For example, the debugging steps of RS485 are as follows(ROC-RK3588S-PC RS485 is half-duplex and requires gpio to control transmission and reception):

(1) Development board sends, host receives

```
# The host terminal executes first:
# /dev/ttyUSB0 is the node of the host serial adapter, modify it according to the actual situation
cat /dev/ttyUSB0

# The development board debugs the serial port terminal and executes:
# /dev/ttyS1 is the RS485 node
echo "firefly RS485 test..." > /dev/ttyS1
```

The host terminal will receive the string "firefly RS485 test..."

(2) Host sends, development board receives

```
echo 34 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio34/direction


# To debug the serial terminal on the development board, execute first:
# /dev/ttyS1 is the RS485 node
echo 0 > /sys/class/gpio/gpio34/value
busybox  stty -echo -F /dev/ttyS1       # turn off echo
cat /dev/ttyS1

# The host terminal executes:
# /dev/ttyUSB0 is the node of the host serial adapter, modify it according to the actual situation
echo 1 > /sys/class/gpio/gpio34/value
echo "firefly RS485 test..." > /dev/ttyUSB0
```

The development board debugs the serial terminal to receive the string "firefly RS485 test..."