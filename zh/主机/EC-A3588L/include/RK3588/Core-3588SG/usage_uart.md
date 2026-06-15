# UART 使用

## 硬件

EC-A3588L 硬件版本的串口接口图如下：

![](../../../rk3588_img/EC-A3588L/usage_uart_interface.png)

## DTS配置

文件路径`kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-port.dtsi`
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

配置好串口后，硬件接口对应软件上的节点为：
```
UART0:   /dev/ttyS0
UART3:   /dev/ttyS3
UART4:   /dev/ttyS4
UART5:   /dev/ttyS5
```

## UART 收发验证

最简单的方式短接 UART0 TX RX 引脚，然后使用命令在调试串口或 ADB 执行命令

```
busybox  stty -echo -F /dev/ttyS0          # 关闭回显
cat /dev/ttyS0 &                           # 后台获取/dev/ttyS7输入字符串
echo "firefly uart test..." > /dev/ttyS0   # 输入字符串
```

最终调试串口终端即可接收到字符串 "firefly uart test..."
