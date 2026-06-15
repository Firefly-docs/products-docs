# UART

## Hardware interface

AIO-3576Q38 The following figure shows the serial port of the hardware version：

![](../../../rk3576_img/AIO-3576Q38/usage_uart_interface.jpg)

## DTS config

File path `kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc.dtsi`
```
/* uart6 */
&uart6 {
	pinctrl-names = "default";
	pinctrl-0 = <&uart6m3_xfer>;
	status = "okay";
};
```

After the serial port is configured, the node corresponding to the hardware interface
```
UART6:   /dev/ttyS6
```

## UART send and receive

The easiest way to do this is to stub the UART7 TX RX pin and then use the command to execute the command in the debug serial port or ADB

```
busybox  stty -echo -F /dev/ttyS6          # Close the echo
cat /dev/ttyS6 &                           # Get /dev/ttyS6
echo "firefly uart test..." > /dev/ttyS6   # Input string
```

The final debugging serial port terminal can receive the string "firefly uart test..."
