# UART

## Hardware interface

iCore-3588JQ The following figure shows the serial port of the hardware version：

![](../../../rk3588_img/iCore-3588JQ/usage_uart_interface.png)

## DTS config

File path `kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-port.dtsi`
```
&uart0 {
    status = "okay";
    pinctrl-names = "default";
    pinctrl-0 = <&uart0m2_xfer>;
};

&uart3 {
    status = "okay";
    pinctrl-names = "default";
    pinctrl-0 = <&uart3m2_xfer>;
};

&uart4 {
	status = "okay";
	pinctrl-names = "default";
	pinctrl-0 = <&uart4m0_xfer>;
};

&uart5 {
	status = "okay";
	pinctrl-names = "default";
	pinctrl-0 = <&uart5m1_xfer>;
};
```

After the serial port is configured, the node corresponding to the hardware interface
```
UART0:   /dev/ttyS0
UART3:   /dev/ttyS3
UART4:   /dev/ttyS4
UART5:   /dev/ttyS5
```

## UART send and receive

The easiest way to do this is to stub the UART0 TX RX pin and then use the command to execute the command in the debug serial port or ADB

```
busybox  stty -echo -F /dev/ttyS0          # Close the echo
cat /dev/ttyS0 &                           # Get /dev/ttyS0 
echo "firefly uart test..." > /dev/ttyS0   # Input string
```

The final debugging serial port terminal can receive the string "firefly uart test..."
