# UART Use

## Introduction

There are 5 uarts on Firefly-RK3288 development board, which are uart0, uart1, uart2, uart3 and uart4.

* uart0 is uart_bt, used for Bluetooth data transmission.
* uart2 is uart_dbg, used for debugging.
* uart1, uart3 and uart4 are used for external uart data communication. You can connect them via J10. Note that uart4 and SPI0 use the same pins.
* All uarts have 64 bytes FIFO respectively, and support 5 bit, 6 bit,7 bit,8 bit serial data communication, in DMA-based or interrupt-based operation mode.

## Configuration Steps

Here we use uart3 as an example.

### Create DTS Node

The DTS node have already been created in file `kernel/arch/arm/boot/dts/rk3288.dtsi`, shown as following:

```dts
uart_gps: serial@ff1b0000 {
    compatible = "rockchip,serial";
    reg = <0xff1b0000 0x100>;
    interrupts = <GIC_SPI 58 IRQ_TYPE_LEVEL_HIGH>;
    clock-frequency = <24000000>;
    clocks = <&clk_uart3>, <&clk_gates6 11>;
    clock-names = "sclk_uart", "pclk_uart";
    current-speed = <115200>;
    reg-shift = <2>;
    reg-io-width = <4>;
    dmas = <&pdma1 7>, <&pdma1 8>;
    #dma-cells = <2>;
    pinctrl-names = "default";
    pinctrl-0 = <&uart3_xfer &uart3_cts &uart3_rts>;
    status = "disabled";
};
```

Note: uart_gps is defined in "aliases" node as: serial3 = &uart_gps;

The only thing you need to do is adding following code in file `kernel/arch/arm/boot/dts/firefly-rk3288.dts`:

```dts
&uart_gps {
    status = "okay";
    dma-names = "!tx", "!rx";
    pinctrl-0 = <&uart3_xfer &uart3_cts>;
};
```

## Compile and Flash Kernel

Turn on CONFIG_SERIAL_ROCKCHIP in kernel configuration, which will add corresponding  `drivers/tty/serial/rk_serial.c` to kernel. Then compile the kernel as followed:

```bash
make firefly-rk3288.img
```

After successful compilation, flash kernel.img and resource.img under kernel directory to the development board.

## Data Commuincation

You can now communicate with the uart3 via a USB-to-serial adapter in your host PC. Follow the steps below:

### Connect the uart port

Connect the TX, RX, GND pins of uart3 to the serial adapter's RX, TX, GND pins respectively.

### Open a serial terminal in host PC

Run kermit in a shell window, and set baud rate:

```bash
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 115200
C-Kermit> set flow-control none
C-Kermit> connect
```

* `/dev/ttyUSB0` is the device file of USB-to-serial adapter.
* baud is the "current-speed" attribute in the DTS node.

### Transmit data

The device file for uart3 is /dev/ttyS3. Run the following command in device:

```bash
echo firefly uart3 test... > /dev/ttyS3
```

The serial terminal in the host PC will receive string "firefly uart3 test...".

### Receive data

First, run the following command in device:

```bash
cat /dev/ttyS3
```

Then input string "Firefly uart3 test..." in the serial terminal. You can see the same string received in the device.