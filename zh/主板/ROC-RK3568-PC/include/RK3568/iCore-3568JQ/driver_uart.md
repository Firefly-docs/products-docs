# UART 使用

## 简介

ROC-RK3568-PC支持UART、RS232、RS485接口
* UARTx1
* RS485x1
* RS232x1

其中开发板的 RS232 接口由主控的 uart3 扩展出来，RS485 是由 uart0 扩展出来，UART 则是 uart9

## DTS配置

文件路径`kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-itx-3568q.dtsi`

```
&uart3 {
	pinctrl-names = "default";
	pinctrl-0 = <&uart3m0_xfer>;
    status = "okay";
};

&uart0 {
    status = "okay";
};

&uart9 {
	pinctrl-names = "default";
	pinctrl-0 = <&uart9m1_xfer>;
    status = "okay";
};
```

配置好串口后，硬件接口对应软件上的节点分别为：

```
RS485:   /dev/ttyS0
UART9:   /dev/ttyS9
RS232:   /dev/ttyS3
```

## PIN 脚定义
* RS232 & RS485
	![](../../img/ROC-RK3568-PC/iCore-3568JQ_RS232-RS485_pins.jpg)

* UART9
	![](../../img/ROC-RK3568-PC/iCore-3568JQ_uart_pins.jpg)

## RS232/RS485 使用说明

使用如下指令切换RS485/RS232：

- Android

```
echo 255 > /sys/class/leds/firefly\:rs232_485_switch\:user/brightness  //enabled RS485
echo 0 > /sys/class/leds/firefly\:rs232_485_switch\:user/brightness  //enabled RS232
```

- Linux

```
uart-switch.sh on  //enable RS485
uart-switch.sh off  //enable RS232
uart-switch.sh status  //current status
```

## 调试方法

用户可以根据不同的接口使用不同的主机的 USB 转串口适配器向开发板的串口收发数据，例如 RS485 的调试步骤如下：

(1) 连接硬件

将开发板RS485 的A、B、GND 引脚分别和主机串口适配器（USB 转 485 转串口模块）的 A、B、GND 引脚相连。

(2) 打开主机的串口终端

在终端输入安装kermit命令`sudo apt install ckermit`，安装完成后打开kermit，设置波特率并连接：

```
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

* `/dev/ttyUSB0` 为 主机USB 转串口适配器的设备文件

(3) 开发板发送数据

开发板的RS485 设备文件为 /dev/ttyS0。在开发板设备上运行下列命令：

```
echo "firefly RS485 test..." > /dev/ttyS0
```

主机中的串口终端即可接收到字符串 "firefly RS485 test..."

(4) 开发板接收数据

首先在开发板设备上运行下列命令：

```
cat /dev/ttyS0
```

然后在主机的串口终端输入字符串 "Firefly RS485 test..."，设备端即可见到相同的字符串。

(5) 主机退出kermit串口连接

`ctrl+\`后按c，退回终端可输入exit

```
C-Kermit>exit
OK to exit? ok
```

