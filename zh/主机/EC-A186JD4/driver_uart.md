
# UART 使用

AIO-186JD4 串口[位置如图所示](interface_definition.html#zheng-ji-jie-kou-ding-yi)  

## 简介
AIO-186JD4 支持 3 个 UART 设备:

| 设备名称 | 接口功能 | 波特率 | 是否全双工 | 电平 |
|----------|----------|--------|:----------:|:----:|
| /dev/ttyS0 | 调试串口(Type-C / 4 Pin) | 115200 | 是 | TTL |
| /dev/ttyS1 | RS232 | 115200 | 是 | RS232 |
| /dev/ttyS4 | RS485 | 9600 | 否 | RS485 |

##  RS485 调试方法

用户可以根据不同的接口使用不同的主机的 USB 转串口适配器向开发板的串口收发数据，例如 RS485 的调试步骤如下：

(1) 连接硬件
将开发板RS485 的A、B、GND 引脚分别和主机串口适配器（USB 转 485 转串口模块）的 A、B、GND 引脚相连。

(2) 打开主机的串口终端

在终端打开 kermit，并设置波特率：

```
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

* `/dev/ttyUSB0` 为 USB 转串口适配器的设备文件

(3) 发送数据

RS485 的设备文件为 `/dev/ttyS4`。在设备上运行下列命令(**请注意，在运行命令之前，需要打开 RS485 的发送功能，因为 AIO-186JD4 有一个 gpio 切换开关，可以切换 RS485 为发送状态还是接受状态。**)：

```
sudo -s
echo 1 > /sys/class/leds/RS485_H_SEND_L_RECV/brightness # 1 -> TX ， 0 -> RX
stty -F /dev/ttyS4 9600 -echo
echo firefly RS485 test... > /dev/ttyS4
```

主机中的串口终端即可接收到字符串 "firefly RS485 test..."

(4) 接收数据

首先在设备上运行下列命令：

```
sudo -s
cat /dev/ttyS4
```

然后在主机的串口终端输入字符串 "Firefly RS485 test..."，设备端即可见到相同的字符串。

##  RS232 调试方法

测试方法与 RS485 的步骤是类似的，只需要注意设备名称与波特率即可。唯一不同的是， RS232 **不需要**去做发送和接收的开关切换。

