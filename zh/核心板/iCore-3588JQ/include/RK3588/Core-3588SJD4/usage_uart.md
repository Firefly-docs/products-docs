# UART 使用

## 简介

AIO-3588JQ  支持 RS232、RS485接口

AIO-3588JQ  开发板的串口接口图如下：

![](../../../rk3588_img/iCore-3588JQ/usage_uart_interface.jpg)

## DTS配置
开发板的 RS232 接口由主控的 UART0 扩展出来，而 RS485 接口由主控 UART3 扩展出来。

文件路径`kernel-5.10/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dtsi
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
```
配置好串口后，硬件接口对应软件上的节点分别为：

```
RS232 :   /dev/ttyS0
RS485 ：  /dev/ttyS3
```

## 收发验证

用户可以根据不同的接口使用不同的主机的 USB 转串口适配器向开发板的串口收发数据，例如 RS485 的调试步骤如下：

(1) 连接硬件

`RS485` 连接 `USB 转 RS485 串口线`;

`主机串口适配器（USB 转 485 转串口模块）` 连接 主机

(2) 开发板发送, 主机接收

```
# 主机终端先执行:
# /dev/ttyUSB0 为 主机串口适配器 的节点，根据实际修改
cat /dev/ttyUSB0

# 开发板调试串口终端执行：
# /dev/ttyS3 为 RS485 节点
echo "firefly RS485 test..." > /dev/ttyS3
```

主机终端即可接收到字符串 "firefly RS485 test..."

(3) 主机发送，开发板接收

```
# 开发板调试串口终端先执行:
# /dev/ttyS3 为 RS485 节点
busybox  stty -echo -F /dev/ttyS3       # 关闭回显
cat /dev/ttyS3

# 主机终端执行：
# /dev/ttyUSB0 为 主机串口适配器 的节点，根据实际修改
echo "firefly RS485 test..." > /dev/ttyUSB0
```

开发板调试串口终端即可接收到字符串 "firefly RS485 test..."