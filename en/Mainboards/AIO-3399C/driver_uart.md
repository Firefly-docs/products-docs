

# UART

## Introduction

AIO-3399C supports SPI bridge/extension functions of four enhanced serial ports (UART), namely UART1, UART2, RS232, and RS485. Each UART has a 256-byte FIFO buffer for data receiving and sending. Among them:

* UART1, UART2 are TTL level interfaces, RS232 are RS232 level interfaces, and RS485 are RS485 level interfaces.
* each sub-channel has 256 BYTE FIFO, which can be programmed according to user requirements.
* has a subserial port to receive FIFO timeout interrupts.
* supports start bit error detection.

The serial interface diagram of the AIO-3399C development board is as follows:

![](../../../rk3399_img/AIO-3399C/uart_interface.jpg)

**Note: RS232 needs to use RS232 crossover cable to use, otherwise rx and tx will have unsuccessful transmission and reception.**

## DTS configuration

File `kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-port.dtsi` has the definition of spi to uart related nodes:

```
&spi1 {
    spi_wk2xxx: spi_wk2xxx@00{
    status = "disabled";
    compatible = "firefly,spi-wk2xxx";
    reg = <0x00>;
    spi-max-frequency = <10000000>;
    power-gpio = <&gpio2 4 GPIO_ACTIVE_HIGH>;
    reset-gpio = <&gpio1 17 3 GPIO_ACTIVE_HIGH>;
    irq-gpio = <&gpio1 2 IRQ_TYPE_EDGE_FALLING>;
    cs-gpio = <&gpio1 10 GPIO_ACTIVE_HIGH>;
    /* rk3399 driver support SPI_CPOL | SPI_CPHA | 			SPI_CS_HIGH */
    //spi-cpha;     /* SPI mode: CPHA=1 */
    //spi-cpol;     /* SPI mode: CPOL=1 */
    //spi-cs-high;
     };
}
```

As you can see, enabling the node in the `kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-aioc.dts` file is available. In addition, SPI1 node needs to be enabled because SPI to UART serial port module used by our board is attached to SPI1. As follows:

```
&spi1 {
    status = "okay";
};

&spi_wk2xxx {
    status = "okay";
};
```

Note: since two pins of `spi1_rxd` and `spi1_txd` can be reused as `uart4_rx` and `uart4_tx`, please note to turn off the use of `uart4`, as follows:

```
&uart4 {
    status = "disabled";
};
```

## Debug method

After the serial port is configured, the nodes on the hardware interface corresponding to the software are:

```
RS485：/dev/ttysWK0
RS232：/dev/ttysWK1
UART2：/dev/ttysWK2
UART1：/dev/ttysWK3
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

The device file for RS485 is `/dev/ttyswk0`. Run the following commands on the device:

```
echo firefly RS485 test... > /dev/ttysWK0
```

The serial terminal in the host can receive a string: "firefly RS485 test... "

(4) Receive data

First run the following commands on the device:

```
cat /dev/ttysWK0
```

Then enter a string at the host's serial terminal: `Firefly RS485 test...`, the device side can see the same string.


