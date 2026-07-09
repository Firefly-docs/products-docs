# UART 使用

## 简介

AIO-3288C 开发板支持 SPI 桥接/扩展 4个增强功能串口(UART)的功能，分别为 RS232，RS485 和 1个调试串口 UART2。

其中：UART2 为 TTL 电平接口，RS232 为 RS232 电平接口，RS485 为 RS485 电平接口。

**注意:RS232 需要使用RS232交叉线才能使用，不然rx和tx会出现收发不成功的问题**

## 调试方法

内核已默认打开这个几个串口，硬件接口对应软件上的节点分别为：

```c
RS485：/dev/ttyS1
RS232：/dev/ttyS3
UART2：/dev/ttyS2
```

用户可以根据不同的接口使用不同的主机的 USB 转串口适配器向开发板的串口收发数据，例如 RS485 的调试步骤如下：

### 连接硬件

将开发板 RS485 的 A、B、GND 引脚分别和主机串口适配器（USB 转 485 转串口模块）的 A、B、GND 引脚相连。

### 打开主机的串口终端

在终端打开 kermit,并设置波特率：

```bash
$ sudo kermit
C-Kermit> set line /dev/ttyUSB*
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

* `/dev/ttyUSB*` 为 USB 转串口适配器的设备文件

### 发送数据

RS485 的设备文件为 `/dev/ttyS1`。在设备上运行下列命令：

```bash
echo firefly RS485 test… > /dev/ttyS1
```

主机中的串口终端即可接收到字符串 “firefly RS485 test…”

### 接收数据

首先在设备上运行下列命令：

```bash
cat /dev/ttyUSB*
```

然后在主机的串口终端输入字符串 “Firefly RS485 test…”，设备端即可见到相同的字符串。

#### 注意

UART2 可以将 rx/tx 短接进行回环通信测试，RS232 由于硬件上不支持回环收发数据，所以只能跟其他主机进行通信测试。