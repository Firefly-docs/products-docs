
# UART

## Introduction

ROC-RK3568-PCSE supports  UART3, UART5(disabled by default, Serial port and SPI multiplexing), RS232(UART9), and RS485(UART0). Each UART has a 256-byte FIFO buffer for data receiving and sending. Among them:

* UART3, UART5 are TTL level interfaces, RS232 are RS232 level interfaces, and RS485 are RS485 level interfaces.
* each sub-channel has 256 BYTE FIFO, which can be programmed according to user requirements.
* has a subserial port to receive FIFO timeout interrupts.
* supports start bit error detection.

The serial interface diagram of the ROC-RK3568-PCSE development board is as follows:
* UART
![](../../img/EC-R3568PC/uart1_interface.jpg)
* RS232
![](../../img/EC-R3568PC/rs232_interface.jpg)
* RS485
![](../../img/EC-R3568PC/rs485_interface.jpg)

## DTS configuration

File `kernel/arch/arm64/boot/dts/rockchip/rk3566-firefly-aiojd4.dtsi` has the definition of spi to uart related nodes:

```
&uart0 {
    status = "okay";
};

&uart3 {
    pinctrl-0 = <&uart3m0_xfer>;
    status = "okay";
};

&uart9 {
    pinctrl-0 = <&uart9m1_xfer>;
    status = "okay";
};
```

## Debug method

After the serial port is configured, the nodes on the hardware interface corresponding to the software are:

```
RS485：/dev/ttyS0
RS232：/dev/ttyS9
UART3：/dev/ttyS3
```

Users can send and receive data to the serial port of the development board by using the USB to serial port adapter of different host according to different interfaces. For example, the debugging steps of RS485 are as follows:

(1) Connected hardware

Connect A, B and GND pins of development board RS485 with A, B and GND pins of host serial port adapter (USB to 485 to serial port module) respectively.

(2) Open the serial terminal of the host

Open `kermit` at the terminal and set the baud rate:

```
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

* `/dev/ttyUSB0` is the device file for USB to serial port adapter.

(3) Send data

The device file for RS485 is `/dev/ttyS0`. Run the following commands on the device:

```
echo firefly RS485 test... > /dev/ttyS0
```

The serial terminal in the host can receive a string: "firefly RS485 test... "

(4) Receive data

First run the following commands on the device:

```
cat /dev/ttyS0
```

Then enter a string at the host's serial terminal: `Firefly RS485 test...`, the device side can see the same string.

