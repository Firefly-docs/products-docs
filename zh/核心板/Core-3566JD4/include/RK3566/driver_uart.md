
# UART 使用

## 简介

Core-3566JD4支持UART、RS232、RS485接口
* UARTx2
* RS485x1
* RS232x1

其中开发板的RS232接口由主控的uart9扩展出来，而RS485则由主控的UART0扩展出来，UART 则由主控的UART3 / UART5(和SPI复用，默认不打开)直接引出来

AIO-3566JD4 开发板的串口接口图如下：
* UART
![](../../img/Core-3566JD4/uart1_interface.jpg)
* RS232
![](../../img/Core-3566JD4/rs232_interface.jpg)
* RS485
![](../../img/Core-3566JD4/rs485_interface.jpg)

## DTS配置

文件路径`kernel/arch/arm64/boot/dts/rockchip/rk3566-firefly-aiojd4.dtsi`
```
&uart0 {                                                                                                                                                                                                           
    status = "okay";
};

&uart3 {
    pinctrl-0 = <&uart3m0_xfer>;
    status = "okay";
};

&uart9 {
    pinctrl-0 = <&uart9m1_xfer>;
    status = "okay";
};
```

配置好串口后，硬件接口对应软件上的节点分别为：

```
RS485：  /dev/ttyS0
RS232:   /dev/ttyS9
UART3：  /dev/ttyS3
```

## 调试方法

用户可以根据不同的接口使用不同的主机的 USB 转串口适配器向开发板的串口收发数据，例如 RS485 的调试步骤如下：

(1) 连接硬件

将开发板RS485 的A、B、GND 引脚分别和主机串口适配器（USB 转 485 转串口模块）的 A、B、GND 引脚相连。

(2) 打开主机的串口终端

在终端打开 kermit，设置波特率并连接：

```
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

* `/dev/ttyUSB0` 为 主机USB 转串口适配器的设备文件
*  ubuntu安装kermit命令`sudo apt install ckermit`

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

