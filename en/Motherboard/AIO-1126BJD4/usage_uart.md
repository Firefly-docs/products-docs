# UART

## Introduction

AIO-1126-JD4 supports  UART6,UART7 to RS485. UART3,UART4 interfaces to RS232.
## DTS Configuration

File path: `kernel-6.1/arch/arm64/boot/dts/rockchip/rv1126b-firefly-aio-1126bjd4.dtsi`

```dts
#ifdef FF_RS485_1
&uart6{
	pinctrl-names = "default";
	pinctrl-0 = <&uart6m0_xfer_pins>;
	status = "okay";
};
#endif

#ifdef FF_RS485_2
&uart7{
	pinctrl-names = "default";
	pinctrl-0 = <&uart7m0_xfer_pins>;
	status = "okay";
};
#endif

#ifdef FF_RS232_1
&uart3 {
	pinctrl-0 = <&uart3m1_xfer_pins>;
	status = "okay";
};
#endif

#ifdef FF_RS232_2
&uart4 {
	pinctrl-0 = <&uart4m1_xfer_pins>;
	status = "okay";
};
#endif

#ifdef FF_UART5
&uart5{
	status = "okay";
	pinctrl-0 = <&uart5m1_xfer_pins>;
};
#endif
```
After configuring the serial port, the nodes corresponding to the hardware interface on the software are:
```
UART6:  /dev/ttyS6        RS485_1
UART7:  /dev/ttyS7        RS485_2
UART3:  /dev/ttyS3        RS232_1
UART4:  /dev/ttyS4        RS232_2
UART5:  /dev/ttyS5
```

## Send And Receive Verification

Users can use different host USB to serial port adapters to send and receive data to the serial port of the development board according to different interfaces. For example, the debugging steps of UART3 are as follows:

(1) Development board sends, host receives

```
# Host terminal executes first:
# /dev/ttyUSB0 is the node of host serial port adapter, modify according to actual situation
cat /dev/ttyUSB0

# Development board debug serial port terminal executes:
# /dev/ttyS3 is UART3 node
echo "firefly UART3 test..." > /dev/ttyS3
```

The host terminal can receive the string "firefly UART3 test..."

(2) Host sends, development board receives

```
# Development board debug serial port terminal executes first:
# /dev/ttyS3 is UART3 node
busybox  stty -echo -F /dev/ttyS3       # Turn off echo
cat /dev/ttyS3

# Host terminal execution:
# /dev/ttyUSB0 is the node of the host serial port adapter, modify according to the actual situation
echo "firefly UART3 test..." > /dev/ttyUSB0
```

The development board debugging serial port terminal can receive the string "firefly UART3 test..."

## RS485
GPIO5_B3 and GPIO5_B6 are used as control transceiver pins for RS485_1 and RS485_2 respectively. The pin is set to transmit when pulled high and to receive when pulled low. Users can operate it through the /sys/class/gpiosubsystem.
* Configure GPIO5_B3(RS485_1) as output. 
```bash
echo 171 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio171/direction
```
* Pull GPIO5_B3 high for RS485 transmission: echo 1 > /sys/class/gpio/gpio171/value

* Pull GPIO5_B3 low for RS485 reception: echo 0 > /sys/class/gpio/gpio171/value
* Configure GPIO5_B6(RS485_2) as output. 
```bash
echo 174 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio174/direction
```
* Pull GPIO5_B6 high for RS485 transmission: echo 1 > /sys/class/gpio/gpio174/value

* Pull GPIO5_B6 low for RS485 reception: echo 0 > /sys/class/gpio/gpio174/value
