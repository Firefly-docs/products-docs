# UART Use

## Introduction

AIO-3288J support SPI bridge/expansion of four enhanced functions of the serial  port (UART), respectively UART2, RS232 (above), RS485, UART3 and three  master comes with serial ports, respectively, UART1, RS232(below) Debug serial port. Each UART has a 256-byte FIFO buffer for data reception and transmission. among them:

* UART1 ~ UART3 are TTL level interface, RS232 is RS232 level interface, RS485 is RS485 level interface
* Each sub-channel UART baud rate, word length, parity format can be set independently, up to 2Mbps communication rate
* Each sub-channel  has a receive/send independent 256 BYTE FIFO, FIFO interrupt can be  programmed according to user needs Trigger point
* With sub-serial port receive FIFO timeout interrupt
* Support for start bit error detection
 detailed locations .

AIO-3288J development board serial interface interface as follows:

![](../../../rk3288_img/AIO-3288J/uart_interface.png)

## Configuration Steps

The spi to uart node have already been created in file `kernel/arch/arm/boot/dts/firefly-rk3288-aio-3288j.dts`, Enable this node to use it:

```dts
 &spi0 {
     status = "okay";
     max-freq = <48000000>;
     spidev@00 {
         status = "disabled";
         compatible = "linux,spidev";
         reg = <0x00>;
         spi-max-frequency = <48000000>;
     };
     spi_wk2xxx: spi_wk2xxx@00{
         status = "okay";
         compatible = "firefly,spi-wk2xxx";
         reg = <0x00>;
         spi-max-frequency = <10000000>;
         reset-gpio = <&gpio0 GPIO_C2 GPIO_ACTIVE_HIGH>;
         irq-gpio = <&gpio0 GPIO_A7 IRQ_TYPE_EDGE_FALLING>;
         cs-gpio = <&gpio5 GPIO_B5 GPIO_ACTIVE_HIGH>;
         pwr-en-gpio = <&gpio3 GPIO_B6 GPIO_ACTIVE_HIGH>;
     };
 };
```

## Debug method

The kernel has enabled these serial ports in default, the software nodes corresponding to hardware interface are:

```c
RS485：/dev/ttysWK3
RS232（above）：/dev/ttysWK1
RS232（below）：/dev/ttyS3
UART1：/dev/ttyS1
UART2：/dev/ttysWK0
UART3：/dev/ttysWK2
```

The user can use different USB to serial port adapters of the host computer to transmit/receive data to/from the serial port of development board according to different ports, for example, the debugging steps of RS485 are as follows:

### Connect hardware

Connect the A, B and GND pins of RS485 on development board to the corresponding pins of serial port adapter of host computer (USB to 485 to serial port module) respectively.

### Turn on the serial port terminal of host computer

Enable Kermit on the terminal and set Baud rate:

```bash
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

* `/dev/ttyUSB0` is the device file of USB to serial port adapter

### Send data

The device file of RS485 is `/dev/ttysWK3`. Run the following command on the device:

```bash
echo firefly RS485 test… > /dev/ttysWK3
```

The serial terminal in host computer can receive the character string “firefly RS485 test…”

### Receive data

Firstly run the following command on the device:

```bash
cat /dev/ttysWK3
```

Then, input the character string “Firefly RS485 test…” at the serial terminal in host computer. You can see the same character string on the device.