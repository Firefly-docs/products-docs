# UART 使用

## 硬件

EC-R3576PC-FD 硬件版本的串口接口图如下：

![](../../../rk3576_img/EC-R3576PC-FD/usage_uart_interface.jpg)

## DTS配置

文件路径`kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc.dtsi`
```
/* uart6 */
&uart6 {
	pinctrl-names = "default";
	pinctrl-0 = <&uart6m3_xfer>;
	status = "okay";
};
```

配置好串口后，硬件接口对应软件上的节点为：
```
UART6:   /dev/ttyS6
```

## UART 收发验证

最简单的方式短接UART6 TX RX 引脚, 然后使用命令在调试串口或ADB执行命令

```
busybox  stty -echo -F /dev/ttyS6         # 关闭回显
cat /dev/ttyS6&                           # 后台获取/dev/ttyS7输入字符串
echo "firefly uart test..." > /dev/ttyS6  # 输入字符串
```

最终调试串口终端即可接收到字符串 "firefly uart test..."
