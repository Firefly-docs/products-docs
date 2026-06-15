# UART 使用

## 硬件

AIO-3576C 使用了 `UART3` 做 `RS485`，在系统中对应 `/dev/ttyS3` 设备；`UART8` 做 `RS232`，在系统中对应 `/dev/ttyS8` 设备。

接口图如下：

![](../../../rk3576_img/AIO-3576C/usage_uart_interface.jpg)

![](../../../rk3576_img/AIO-3576C/usage_uart_interface2.jpg)

## RS485 节点使用
```
# 设置收发控制引脚，高电平发送，低电平接收
sudo echo 22 > /sys/class/gpio/export
sudo echo out > /sys/class/gpio/gpio22/direction
# 关闭回显
sudo stty -F /dev/ttyS3 -echo
# 发送
sudo echo 1 > /sys/class/gpio/gpio22/value
sudo echo "firefly uart test..." > /dev/ttyS3   # 输入字符串
# 接收
sudo echo 0 > /sys/class/gpio/gpio22/value
sudo cat /dev/ttyS3
```