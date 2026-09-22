# 硬件功能使用
## 调试串口

EC-A3288C 的调试串口为 UART2（TTL 电平），对应系统设备节点 `/dev/ttyS2`，串口参数为：波特率 115200、数据位 8、停止位 1、无奇偶校验、无流控。

调试串口为 TTL 电平接口，使用时需外接 USB 转 TTL 串口模块进行调试。

调试串口的连接方式与使用方法详见：[调试串口](usb_to_ttl.md)。

## UART（RS485 / RS232）

EC-A3288C 通过 SPI 桥接扩展了增强功能串口，其中 RS485 为 RS485 电平接口，RS232 为 RS232 电平接口。内核已默认打开上述串口，各接口对应的软件节点如下：

```
RS485：/dev/ttyS1
RS232：/dev/ttyS3
UART2（调试串口）：/dev/ttyS2
```

**注意：RS232 需要使用 RS232 交叉线才能使用，不然 rx 和 tx 会出现收发不成功的问题。**

以 RS485 为例，调试步骤如下：

### 连接硬件

将 RS485 的 A、B、GND 引脚分别和主机串口适配器（USB 转 485 转串口模块）的 A、B、GND 引脚相连。

### 打开主机的串口终端

在终端打开 kermit，并设置波特率：

```bash
$ sudo kermit
C-Kermit> set line /dev/ttyUSB*
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

`/dev/ttyUSB*` 为主机识别到的 USB 转串口适配器的设备文件，以实际系统为准。

### 收发验证

设备发送，主机接收。RS485 的设备文件为 `/dev/ttyS1`，在设备上运行下列命令：

```bash
echo firefly RS485 test… > /dev/ttyS1
```

主机中的串口终端即可接收到字符串 "firefly RS485 test…"。

主机发送，设备接收。首先在设备上运行下列命令：

```bash
cat /dev/ttyS1
```

然后在主机的串口终端输入字符串 "firefly RS485 test…"，设备端即可见到相同的字符串。RS232 的验证方法同理，将节点换成 `/dev/ttyS3` 即可。

**注意：** 调试串口 UART2 可以将 rx/tx 短接进行回环通信测试，RS232 由于硬件上不支持回环收发数据，所以只能跟其他主机进行通信测试。

## 显示接口

EC-A3288C 支持双屏同显/双屏异显。SDK 提供 LVDS 显示输出的固件配置（`aio-3288c-lvds-ubuntu.mk`、`aio-3288c-lvds-buildroot.mk`），可输出的分辨率与接口形态以实际系统为准。

## 其他接口

EC-A3288C 其余对外接口（以太网、USB 等）的使用方法待补充，具体以实际系统为准。
