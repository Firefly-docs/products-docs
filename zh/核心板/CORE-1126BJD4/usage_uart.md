# UART 使用

## 简介

AIO-1126BJD4 支持UART6,UART7做RS485。UART3,UART4做RS232，还有UART5单独做串口。

## DTS配置

文件路径`kernel-6.1/arch/arm64/boot/dts/rockchip/rv1126b-firefly-aio-1126bjd4.dtsi`

```dts
#ifdef FF_RS485_1
&uart6{
	pinctrl-names = "default";
	pinctrl-0 = <&uart6m0_xfer_pins>;
	status = "okay";
};
#endif

#ifdef FF_RS485_2
&uart7{
	pinctrl-names = "default";
	pinctrl-0 = <&uart7m0_xfer_pins>;
	status = "okay";
};
#endif

#ifdef FF_RS232_1
&uart3 {
	pinctrl-0 = <&uart3m1_xfer_pins>;
	status = "okay";
};
#endif

#ifdef FF_RS232_2
&uart4 {
	pinctrl-0 = <&uart4m1_xfer_pins>;
	status = "okay";
};
#endif

#ifdef FF_UART5
&uart5{
	status = "okay";
	pinctrl-0 = <&uart5m1_xfer_pins>;
};
#endif
```
配置好串口后，硬件接口对应软件上的节点分别为：

```
UART6:  /dev/ttyS6        RS485_1
UART7:  /dev/ttyS7        RS485_2
UART3:  /dev/ttyS3        RS232_1
UART4:  /dev/ttyS4        RS232_2
UART5:  /dev/ttyS5       
```

## 收发验证

用户可以根据不同的接口使用不同的主机的 USB 转串口适配器向开发板的串口收发数据，例如 UART3 的调试步骤如下：

(1) 开发板发送, 主机接收

```bash
# 主机终端先执行:
# /dev/ttyUSB0 为 主机串口适配器 的节点，根据实际修改
cat /dev/ttyUSB0

# 开发板调试串口终端执行：
# /dev/ttyS3 为 UART3 节点
echo "firefly UART3 test..." > /dev/ttyS3
```

主机终端即可接收到字符串 "firefly UART3 test..."

(2) 主机发送，开发板接收

```bash
# 开发板调试串口终端先执行:
# /dev/ttyS3 为 UART3 节点
busybox  stty -echo -F /dev/ttyS3       # 关闭回显
cat /dev/ttyS3

# 主机终端执行：
# /dev/ttyUSB0 为 主机串口适配器 的节点，根据实际修改
echo "firefly UART3 test..." > /dev/ttyUSB0
```

开发板调试串口终端即可接收到字符串 "firefly UART3 test..."

## RS485
其中 GPIO5_B3和GPIO5_B6分别用作RS485_1和RS485_2控制收发引脚，该引脚拉高是发送，拉低为接收。用户可以通过 /sys/class/gpio 子系统去操作。
* 下面是GPIO5_B3(RS485_1)配置成输出
```bash
echo 171 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio171/direction
```
* GPIO5_B3 拉高，RS485 发送： echo 1 > /sys/class/gpio/gpio171/value
* GPIO5_B3 拉低，RS485 接收： echo 0 > /sys/class/gpio/gpio171/value
* 下面是GPIO5_B6(RS485_2)配置成输出
```bash
echo 174 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio174/direction
```
* GPIO5_B6 拉高，RS485 发送： echo 1 > /sys/class/gpio/gpio174/value
* GPIO5_B6 拉低，RS485 接收： echo 0 > /sys/class/gpio/gpio174/value