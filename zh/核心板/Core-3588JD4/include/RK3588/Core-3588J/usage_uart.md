# UART 使用

## 简介

ITX-3588J 支持 RS232、RS485、UART0、UART1接口
* RS232 和 UART0 只能二选一使用
* RS485 和 UART1 只能二选一使用

ITX-3588J 开发板的串口接口图如下：

![](../../../rk3588_img/Core-3588JD4/usage_uart_interface.jpg)

如何使用跳帽选择 RS232 或 UART0, RS485 或 UART1：

![](../../../rk3588_img/Core-3588JD4/usage_uart_jump_cap_interface.jpg)

* RS232：`8` 和 `9` 短接，`11` 和 `12` 短接
* UART0：`8` 和 `7` 短接，`11` 和 `10` 短接
* RS485：`2` 和 `3` 短接，`5` 和 `6` 短接
* UART1：`2` 和 `1` 短接，`5` 和 `4` 短接

例如，选择使用 RS232、RS485，跳帽连接图如下：

![](../../../rk3588_img/Core-3588JD4/usage_uart_jump_cap.png)

RS232、RS485 推荐使用<font color=#ff00>官方的 FC10 转 DP9 串口线</font>，不同厂商的串口线线序可能不同，会导致串口无法通信。

![](../../../rk3588_img/Core-3588JD4/usage_sata_fc10_to_db9.png)

## DTS配置
开发板的 RS232 接口由主控的 UART0 扩展出来，而 RS485 接口由主控 UART1 扩展出来。

文件路径`kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dtsi`
```
/* uart/232/485 */
&uart0{
    pinctrl-0 = <&uart0m2_xfer>;
    status = "okay";
};

&uart1{
    pinctrl-0 = <&uart1m1_xfer>;
    status = "okay";
};

```
配置好串口后，硬件接口对应软件上的节点分别为：

```
RS232 或 UART0:   /dev/ttyS0
RS485 或 UART1：  /dev/ttyS1
```

## 收发验证

用户可以根据不同的接口使用不同的主机的 USB 转串口适配器向开发板的串口收发数据，例如 RS485 的调试步骤如下：

(1) 连接硬件
![](../../../rk3588_img/Core-3588JD4/usage_uart_rs485_connect.jpg)

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