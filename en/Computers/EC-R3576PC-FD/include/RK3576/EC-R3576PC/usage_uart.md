# UART

<font color=#FF0000>This chapter contains descriptions of RS232 and RS485 </font>

## Hardware interface

EC-R3576PC-FD The following figure shows the serial port of the hardware version：

![](../../../rk3576_img/EC-R3576PC-FD/usage_uart_interface.jpg)

## DTS config

File path `kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc.dtsi`
```
/* uart6 */
&uart6 {
        pinctrl-names = "default";
        pinctrl-0 = <&uart6m3_xfer>;
        status = "okay";
};
```
File path `kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc-ext.dtsi`
```
&spi3 {
        status = "okay";
        //max-freq = <48000000>;
        dev-port = <0>;
    pinctrl-0 = <&spi3m1_pins>;
        num-cs = <1>;

        spi_wk2xxx: spi_wk2xxx@00{
                status = "okay";
                compatible = "firefly,spi-wk2xxx";
                reg = <0x00>;
                spi-max-frequency = <10000000>;
                //power-gpio = <&gpio2 4 GPIO_ACTIVE_HIGH>;
                reset-gpio = <&gpio4 RK_PA0 GPIO_ACTIVE_HIGH>;
                irq-gpio = <&gpio3 RK_PA4 IRQ_TYPE_EDGE_FALLING>;
                cs-gpio = <&gpio3 RK_PD7 GPIO_ACTIVE_HIGH>;
        };
};
```

After the serial port is configured, the node corresponding to the hardware interface
```
RS485 :   /dev/ttysWK0(Silk screen：A1 B1)    /dev/ttysWK1(Silk screen：A2 B2)
RS232 :   /dev/ttysWK3(Silk screen：T1 R1)    /dev/ttysWK2(Silk screen：T2 R2)   /dev/ttyS6(Silk screen：T3 R3)
```

## RS232 send and receive

The easiest way to do this is to stub the RS232 TX RX pin and then use the command to execute the command in the debug serial port or ADB

/dev/ttyS6 test example is as follows (please select /dev/ttysWK2,/dev/ttysWK3,/dev/ttyS6 according to the actual stub)
```
busybox  stty -echo -F /dev/ttyS6          # Close the echo
cat /dev/ttyS6 &                           # Get /dev/ttyS7 
echo "firefly uart test..." > /dev/ttyS6   # Input string
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