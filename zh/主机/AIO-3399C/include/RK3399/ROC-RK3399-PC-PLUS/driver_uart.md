
# UART 使用


## 简介

为了方便用户开发，ROC-3399-PC-PLUS 在 板子的双排扩展接口上引出了UART0,以及UART4（与SPI1复用）等两个通道。 

* UART0，UART4为TTL电平接口。

* UART0 UART4最高支持波特率691200。

* 每个子通道具备收/发独立的256 BYTE FIFO,FIFO的中断可按用户需求进行编程触发点

* 具备子串口接收FIFO超时中断

* 支持起始位错误检测



## DTS配置
以UART0 配置为例，设备树默认打开了uart0的节点。 
```
&uart0 {
    status = "okay";
};
```

## 调试方法

配置好串口后，硬件接口对应软件上的节点分别为：
```
UART0：/dev/ttyS0
```

(1) 连接硬件

将UART0的TX与RX口与USB转串口适配器的RX,TX相连。

(2) 打开主机的串口终端

在终端打开kermit,并设置波特率：
```
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 115200
C-Kermit> set flow-control none
C-Kermit> connect
```

*  /dev/ttyUSB0 为 USB 转串口适配器的设备文件

(3) 发送数据

UART0 的设备文件为 /dev/ttyS0。在设备上运行下列命令：
```
echo firefly test... > /dev/ttyS0
```
主机中的串口终端即可接收到字符串“firefly test...”

(4) 接收数据

首先在设备上运行下列命令：
```
cat /dev/ttyS0
```
然后在主机的串口终端输入字符串 “firefly test...”，设备端即可见到相同的字符串。

## 注意事项

GPS[无线模块](module_wireless.md)会占用uart0,使用uart0作普通串口时，需要将GPS功能禁止。