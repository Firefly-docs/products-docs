# UART 使用

## 简介

EC-A3588Q 有一个 RS232 和一个 RS485 接口

EC-A3588Q 开发板的串口接口图如下：

![](../../../rk3588_img/EC-A3588Q/usage_uart_interface.jpg)

## DTS配置
开发板的 RS232 接口由主控的 UART1 扩展出来，而 RS485 接口由主控 UART6 扩展出来。

文件路径`kernel/arch/arm64/boot/dts/rockchip/aio-3588jd4.dtsi`
```
/* RS485 */
&uart6{
        pinctrl-0 = <&uart6m1_xfer>;
        status = "okay";
};

/* RS232 */
&uart1{
        pinctrl-0 = <&uart1m1_xfer>;
        status = "okay";
};

```
配置好串口后，硬件接口对应软件上的节点分别为：

```
RS232:   /dev/ttyS1
RS485：  /dev/ttyS6
```

## 收发验证

用户可以根据不同的接口使用不同的主机的 USB 转串口适配器向开发板的串口收发数据，例如 RS485 的调试步骤如下(EC-A3588Q RS485 是半双工，需要 gpio 控制收发)：

(1) 开发板发送, 主机接收

```
# 主机终端先执行:
# /dev/ttyUSB0 为 主机串口适配器 的节点，根据实际修改
cat /dev/ttyUSB0

# 开发板调试串口终端执行：
# /dev/ttyS1 为 RS485 节点
echo "firefly RS485 test..." > /dev/ttyS1
```

主机终端即可接收到字符串 "firefly RS485 test..."

(2) 主机发送，开发板接收

```
echo 34 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio34/direction

# 开发板调试串口终端先执行:
# /dev/ttyS1 为 RS485 节点
busybox  stty -echo -F /dev/ttyS6       # 关闭回显
echo 0 > /sys/class/gpio/gpio34/value
cat /dev/ttyS1

# 主机终端执行：
# /dev/ttyUSB0 为 主机串口适配器 的节点，根据实际修改
echo 1 > /sys/class/gpio/gpio34/value
echo "firefly RS485 test..." > /dev/ttyUSB0
```

开发板调试串口终端即可接收到字符串 "firefly RS485 test..."