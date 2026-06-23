# RS485/RS232 使用

## RS485

AIBOX-1684X 有一个 RS485 接口，设备名称为 `/dev/ttyS2`，支持全双工，默认波特率为 `9600`。

测试方法如下：

（1）连接硬件

将开发板 RS485 的A、B、GND 引脚分别和 PC 机串口适配器（USB 转 485 转串口模块）的 A、B、GND 引脚相连。

（2）打开 PC 机串口终端

在终端打开 kermit，并设置波特率：

```shell
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

其中 `/dev/ttyUSB0`  为 USB 转串口适配器的设备文件，注意要看实际 PC 机识别的为准。

（3）发送数据

RS485 的设备文件为 `/dev/ttyS2`。在设备上运行下列命令：

```shell
sudo -s
stty -F /dev/ttyS2 9600 -echo
echo firefly RS485 test... > /dev/ttyS2
```
PC 机中的串口终端即可接收到字符串 “firefly RS485 test…”

（4）接收数据

首先在设备上运行下列命令：

```shell
sudo -s
cat /dev/ttyS2
```

然后在 PC 机的串口终端输入字符串 “Firefly RS485 test…”，设备端即可见到相同的字符串。

## RS232

除了 RS485，AIBOX-1684X 还有一个 RS232 接口，设备名称为 `/dev/ttyS1`，支持全双工，默认波特率为 `115200`。

需要注意的是 RS232 默认是用作登录使用，因此用户若要使用，应当输入以下命令将其恢复成普通通讯串口功能：

```shell
sudo systemctl disable --now serial-getty@ttyS1.service
```

测试方法与 RS485 的步骤是类似的，只需要注意设备名称与波特率即可。