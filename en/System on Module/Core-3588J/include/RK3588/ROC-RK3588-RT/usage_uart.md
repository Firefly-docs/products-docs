# UART

## Hardware interface

Core-3588J The following figure shows the serial port of the hardware version：

![](../../../rk3588_img/Core-3588J/usage_uart_interface.jpg)

## DTS config

File path `kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi`
```
/* uart7 */
&uart7{
    pinctrl-0 = <&uart7m2_xfer>;
    status = "okay";
};
```

After the serial port is configured, the node corresponding to the hardware interface
```
UART7:   /dev/ttyS7
```

## UART send and receive

The easiest way to do this is to stub the UART7 TX RX pin and then use the command to execute the command in the debug serial port or ADB

```
busybox  stty -echo -F /dev/ttyS7          # Close the echo
cat /dev/ttyS7 &                           # Get /dev/ttyS7 
echo "firefly uart test..." > /dev/ttyS7   # Input string
```

The final debugging serial port terminal can receive the string "firefly uart test..."
