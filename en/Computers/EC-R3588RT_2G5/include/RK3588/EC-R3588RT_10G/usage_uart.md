# UART

<font color=#FF0000>This chapter contains descriptions of RS232 and RS485 </font>

## Hardware interface

EC-R3588RT_2G5 The following figure shows the serial port of the hardware version：

![](../../../rk3588_img/EC-R3588RT_2G5/usage_uart_interface.jpg)

## DTS config

File path `kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi`
```
/* uart7 */
&uart7{
    pinctrl-0 = <&uart7m2_xfer>;
    status = "okay";
};
```
File path `kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc-ext.dtsi`
```
&spi1 {
    status = "okay";
    max-freq = <48000000>;
    dev-port = <0>;
    pinctrl-0 = <&spi1m2_pins>;
    num-cs = <1>;

    spi_wk2xxx: spi_wk2xxx@00{
        status = "okay";
        compatible = "firefly,spi-wk2xxx";
        reg = <0x00>;
        spi-max-frequency = <10000000>;
        reset-gpio = <&gpio3 RK_PA6 GPIO_ACTIVE_HIGH>;
        irq-gpio = <&gpio3 RK_PC6 IRQ_TYPE_EDGE_FALLING>;
        cs-gpio = <&gpio1 RK_PD3 GPIO_ACTIVE_HIGH>;
    };
};
```

After the serial port is configured, the node corresponding to the hardware interface
```
RS485 :   /dev/ttysWK0(Silk screen：A1 B1)    /dev/ttysWK1(Silk screen：A2 B2)
RS232 :   /dev/ttysWK3(Silk screen：T1 R1)    /dev/ttysWK2(Silk screen：T2 R2)   /dev/ttyS7(Silk screen：T3 R3)
```

## RS232 send and receive

The easiest way to do this is to stub the RS232 TX RX pin and then use the command to execute the command in the debug serial port or ADB

/dev/ttyS7 test example is as follows (please select /dev/ttysWK2,/dev/ttysWK3,/dev/ttyS7 according to the actual stub)
```
busybox  stty -echo -F /dev/ttyS7          # Close the echo
cat /dev/ttyS7 &                           # Get /dev/ttyS7 
echo "firefly uart test..." > /dev/ttyS7   # Input string
```

The final debugging serial port terminal can receive the string "firefly uart test..."

## RS485 send and receive

The easiest way to do this is to stub the two RS485, A1 to A2, B1 to B2 and then use the command to execute the command in the debug serial port or ADB

```
busybox  stty -echo -F /dev/ttysWK0          # Close the echo
cat /dev/ttysWK1 &                           # Get /dev/ttysWK1 
echo "firefly uart test..." > /dev/ttysWK0   # Input string
```

The final debugging serial port terminal can receive the string "firefly uart test..."