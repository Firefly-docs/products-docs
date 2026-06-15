# UART使用

<font color=#FF0000> 本章包含RS232节点和RS485节点的说明</font>

## 硬件

EC-R3588RT的串口接口图如下：

![](../../../rk3588_img/EC-R3588RT/usage_uart_interface.jpg)

## DTS配置

文件路径`kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi`
```
/* uart7 */
&uart7{
    pinctrl-0 = <&uart7m2_xfer>;
    status = "okay";
};
```
文件路径`kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc-ext.dtsi`
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

配置好串口后，硬件接口对应软件上的节点为：
```
RS485 节点:   /dev/ttysWK0(丝印：A1 B1)    /dev/ttysWK1(丝印：A2 B2)
RS232 节点:   /dev/ttysWK3(丝印：T1 R1)    /dev/ttysWK2(丝印：T2 R2)   /dev/ttyS7(丝印：T3 R3)
```

## 232 节点 收发验证

最简单的方式短接RS232 TX RX 引脚, 然后使用命令在调试串口或ADB执行命令

节点/dev/ttyS7测试示例如下（请根据实际短接脚位选择 /dev/ttysWK2， /dev/ttysWK3， /dev/ttyS7）
```
busybox  stty -echo -F /dev/ttyS7          # 关闭回显，
cat /dev/ttyS7 &                           # 后台获取/dev/ttyS7输入字符串
echo "firefly uart test..." > /dev/ttyS7   # 输入字符串
```
最终调试串口终端即可接收到字符串 "firefly uart test..."

## 485 节点 收发验证

最简单的方式两个RS485互相短接， A1接A2,B1接B2, 然后使用命令在调试串口或ADB执行命令

```
busybox  stty -echo -F /dev/ttysWK0          # 关闭回显
cat /dev/ttysWK1 &                           # 后台获取/dev/ttysWK1输入字符串
echo "firefly uart test..." > /dev/ttysWK0   # 输入字符串
```

最终调试串口终端即可接收到字符串 "firefly uart test..."
