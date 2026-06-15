# UART

## Introduction

ROC-RK3568-PCSE supports UART、RS232、RS485 interfaces
* UARTx1
* RS485x1
* RS232x1

The RS232 of the board is extended from RK3568 uart3, RS485 is extended from uart0 and UART is uart9.

## DTS configuration

File `kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-itx-3568q.dtsi` has the definition of spi to uart related nodes:

```
&uart3 {
	pinctrl-names = "default";
	pinctrl-0 = <&uart3m0_xfer>;
    status = "okay";
};

&uart0 {
    status = "okay";
};

&uart9 {
	pinctrl-names = "default";
	pinctrl-0 = <&uart9m1_xfer>;
    status = "okay";
};
```

After the serial port is configured, the nodes on the hardware interface corresponding to the software are:

```
RS485:   /dev/ttyS0
UART9:   /dev/ttyS9
RS232:   /dev/ttyS3
```

## PINs Definition
* RS232 & RK485
	![](../../img/EC-R3568PC/iCore-3568JQ_RS232-RS485_pins.jpg)

* UART9
	![](../../img/EC-R3568PC/iCore-3568JQ_uart_pins.jpg)

## RS232/RS485 Instruction

The following commands can be used to switch RS485/RS232:

- Android

```
echo 255 > /sys/class/leds/firefly\:rs232_485_switch\:user/brightness  //enabled RS485
echo 0 > /sys/class/leds/firefly\:rs232_485_switch\:user/brightness  //enabled RS232
```

- Linux

```
uart-switch.sh on  //enable RS485
uart-switch.sh off  //enable RS232
uart-switch.sh status  //current status
```

## Debug method

Users can send and receive data to the serial port of the development board by using the USB to serial port adapter of different host according to different interfaces. For example, the debugging steps of RS485 are as follows:

(1) Connected hardware

Connect A, B and GND pins of development board RS485 with A, B and GND pins of host serial port adapter (USB to 485 to serial port module) respectively.

(2) Open the serial terminal of the host

Input the install Kermit command `sudo apt install Kermit` on the terminal. After the installation, open Kermit, set the baud rate and connect:

```
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

* `/dev/ttyUSB0` is the device file for USB to serial port adapter.

(3) Send data to host serial port adapter

The device file for RS485 is `/dev/ttyS0`. Run the following commands on the device:

```
echo firefly RS485 test... > /dev/ttyS0
```

The serial terminal in the host can receive a string: "firefly RS485 test... "

(4) Receive data from host serial port adapter

First run the following commands on the device:

```
cat /dev/ttyS0
```

Then enter a string at the host's serial terminal: `Firefly RS485 test...`, the device side can see the same string.

(5) Exit kermit connection

The host hit keyboard `ctrl+\` and then hit key c, input "exit" and hit enter

```
C-Kermit>exit
OK to exit? ok
```

