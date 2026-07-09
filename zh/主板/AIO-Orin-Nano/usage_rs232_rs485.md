# RS485/RS232

* RS485 接口，设备名称为 `/dev/ttyTHS2`，支持半双工，默认波特率为 `9600` 。
    * 发送数据前先执行： `echo 1 > /sys/class/leds/RS485_H_SEND_L_RECV/brightness`
    * 接收数据前先执行： `echo 0 > /sys/class/leds/RS485_H_SEND_L_RECV/brightness`
* RS232 接口，设备名称为 `/dev/ttyTHS1`，支持全双工，默认波特率为 `115200` 。

## 调试方法

用户可以根据不同的接口使用不同的主机的 USB 转串口适配器向开发板的串口收发数据，例如 RS485 的调试步骤如下：

(1) 连接硬件

将开发板的 RS485B-A、RS485B-B 以及 GND 引脚分别和主机串口适配器（USB 转 485 转串口模块）的 A、B、GND 引脚相连。

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

RS485 的设备文件为 `/dev/ttyTHS2`。在设备上运行下列命令：

```
sudo -s
stty -F /dev/ttyTHS2 9600 -echo
echo firefly RS485 test... > /dev/ttyTHS2
```

主机中的串口终端即可接收到字符串 "firefly RS485 test..."

(4) 接收数据

首先在设备上运行下列命令：

```
sudo -s
cat /dev/ttyTHS2
```

然后在主机的串口终端输入字符串 "Firefly RS485 test..."，设备端即可见到相同的字符串。

