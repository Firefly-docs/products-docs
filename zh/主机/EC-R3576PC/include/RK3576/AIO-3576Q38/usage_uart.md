# UART 使用

## 硬件

EC-R3576PC 使用了 `UART10` 做 `RS485`，在系统中对应 `/dev/ttyS10` 设备；`UART6` 做 `RS232`，在系统中对应 `/dev/ttyS6` 设备。

接口图如下：

![](../../../rk3576_img/EC-R3576PC/usage_uart_interface.jpg)


## RS485 节点使用
```
# 设置收发控制引脚，高电平发送，低电平接收
sudo echo 24 > /sys/class/gpio/export
sudo echo out > /sys/class/gpio/gpio24/direction
# 关闭回显
sudo stty -F /dev/ttyS10 -echo
# 发送
sudo echo 1 > /sys/class/gpio/gpio24/value
sudo echo "firefly uart test..." > /dev/ttyS10   # 输入字符串
# 接收
sudo echo 0 > /sys/class/gpio/gpio24/value
sudo cat /dev/ttyS10
```