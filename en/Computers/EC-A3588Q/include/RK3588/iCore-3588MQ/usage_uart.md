# UART

## Introduction

AIO-3588Q has two RS232 interfaces (RS232_0, RS232_1) and one RS485 interface

The serial interface diagram of the AIO-3588Q development board is as follows:

![](../../../rk3588_img/EC-A3588Q/usage_uart_interface.jpg)

RS232 and RS485 are recommended to use <font color=#ff00>official FC10 to DP9 serial port cable</font>. The serial port cable sequence of different manufacturers may be different, which will cause the serial port to fail to communicate.

![](../../../rk3588_img/EC-A3588Q/usage_sata_fc10_to_db9.png)

## DTS configuration
The RS232 interface of the development board is extended by the main control UART0, and the RS485 interface is extended by the main control UART1.

File path `kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q.dtsi`
```
/* RS232_0 */
&uart0{
	pinctrl-0 = <&uart0m2_xfer>;
	status = "okay";
};

/* RS232_1 */
&uart5{
	pinctrl-0 = <&uart5m1_xfer>;
	status = "okay";
};

/* RS485 */
&uart1{
	pinctrl-0 = <&uart1m1_xfer>;
	status = "okay";
};

```
After configuring the serial port, the nodes on the software corresponding to the hardware interface are:

```
RS232_0:   /dev/ttyS0
RS232_1:   /dev/ttyS5
RS485：  /dev/ttyS1
```

## Send and receive verification

Users can use different host's USB-to-serial adapters to send and receive data to the serial port of the development board according to different interfaces. For example, the debugging steps of RS485 are as follows:

(1) Connect the hardware

`RS485` to connect `FC10 to DP9 serial cable`;

`FC10 to DP9 serial cable` to connect `host serial adapter (USB to 485 serial port module)`;

`Host serial adapter (USB to 485 to serial module)` connect to the host

(2) Development board sends, host receives

```
# The host terminal executes first:
# /dev/ttyUSB0 is the node of the host serial adapter, modify it according to the actual situation
cat /dev/ttyUSB0

# The development board debugs the serial port terminal and executes:
# /dev/ttyS1 is the RS485 node
echo "firefly RS485 test..." > /dev/ttyS1
```

The host terminal will receive the string "firefly RS485 test..."

(3) Host sends, development board receives

```
# To debug the serial terminal on the development board, execute first:
# /dev/ttyS1 is the RS485 node
busybox  stty -echo -F /dev/ttyS1       # turn off echo
cat /dev/ttyS1

# The host terminal executes:
# /dev/ttyUSB0 is the node of the host serial adapter, modify it according to the actual situation
echo "firefly RS485 test..." > /dev/ttyUSB0
```

The development board debugs the serial terminal to receive the string "firefly RS485 test..."