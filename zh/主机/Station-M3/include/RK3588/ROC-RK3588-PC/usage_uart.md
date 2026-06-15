# UART 使用

## 硬件

Station-M3 硬件版本的串口接口图如下：

![](../../../rk3588_img/Station-M3/usage_uart_interface.jpg)

## DTS配置

文件路径`kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588-pc.dtsi`
```
/* uart7 */
&uart7 {
	status = "okay";
	pinctrl-0 = <&uart7m0_xfer>;
};
```

配置好串口后，硬件接口对应软件上的节点为：
```
UART7:   /dev/ttyS7
```

## UART 收发验证

最简单的方式短接UART7 TX RX 引脚, 然后使用命令在调试串口或ADB执行命令

```
busybox  stty -echo -F /dev/ttyS7          # 关闭回显
cat /dev/ttyS7 &                           # 后台获取/dev/ttyS7输入字符串
echo "firefly uart test..." > /dev/ttyS7   # 输入字符串
```

最终调试串口终端即可接收到字符串 "firefly uart test..."
