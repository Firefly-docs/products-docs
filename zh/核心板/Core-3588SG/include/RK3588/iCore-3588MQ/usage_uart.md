# UART 使用

## 简介

AIO-3588Q 有两个 RS232 接口(RS232_0 、RS232_1)和一个 RS485 接口

AIO-3588Q 开发板的串口接口图如下：

![](../../../rk3588_img/Core-3588SG/usage_uart_interface.jpg)

RS232、RS485 推荐使用<font color=#ff00>官方的 FC10 转 DP9 串口线</font>，不同厂商的串口线线序可能不同，会导致串口无法通信。

![](../../../rk3588_img/Core-3588SG/usage_sata_fc10_to_db9.png)

## DTS配置
开发板的 RS232_0 接口由主控的 UART0 扩展出来，RS232_1 接口由主控的 UART5 扩展出来，而 RS485 接口由主控 UART1 扩展出来。

文件路径`kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q.dtsi`
```
/* RS232_0 */
&uart0{
	pinctrl-0 = <&uart0m2_xfer>;
	status = "okay";
};

/* RS232_1 */
&uart5{
	pinctrl-0 = <&uart5m1_xfer>;
	status = "okay";
};

/* RS485 */
&uart1{
	pinctrl-0 = <&uart1m1_xfer>;
	status = "okay";
};

```
配置好串口后，硬件接口对应软件上的节点分别为：

```
RS232_0:   /dev/ttyS0
RS232_1:   /dev/ttyS5
RS485：  /dev/ttyS1
```

## 收发验证

用户可以根据不同的接口使用不同的主机的 USB 转串口适配器向开发板的串口收发数据，例如 RS485 的调试步骤如下：

(1) 连接硬件

`RS485` 连接 `FC10 转 DP9 串口线`;

`FC10 转 DP9 串口线` 连接 `主机串口适配器（USB 转 485 串口模块）`;

`主机串口适配器（USB 转 485 转串口模块）` 连接 主机

(2) 开发板发送, 主机接收

```
# 主机终端先执行:
# /dev/ttyUSB0 为 主机串口适配器 的节点，根据实际修改
cat /dev/ttyUSB0

# 开发板调试串口终端执行：
# /dev/ttyS1 为 RS485 节点
echo "firefly RS485 test..." > /dev/ttyS1
```

主机终端即可接收到字符串 "firefly RS485 test..."

(3) 主机发送，开发板接收

```
# 开发板调试串口终端先执行:
# /dev/ttyS1 为 RS485 节点
busybox  stty -echo -F /dev/ttyS1       # 关闭回显
cat /dev/ttyS1

# 主机终端执行：
# /dev/ttyUSB0 为 主机串口适配器 的节点，根据实际修改
echo "firefly RS485 test..." > /dev/ttyUSB0
```

开发板调试串口终端即可接收到字符串 "firefly RS485 test..."