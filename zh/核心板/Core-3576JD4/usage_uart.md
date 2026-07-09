# UART 使用

## 硬件

Core-3576JD4 使用了 UART3 做 RS485，在系统中对应 /dev/ttyS3 设备；UART8 做 RS232，在系统中对应 /dev/ttyS8 设备。没有引出普通 UART。

其中 GPIO3_A3 用作了 RS485 的收发控制，该引脚拉高是发送，拉低为接收。用户可以通过 /sys/class/gpio 子系统去操作。

* GPIO3_A3 配置成输出

```
echo 99 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio99/direction
```

* GPIO3_A3 拉高，RS485 发送： `echo 1 > /sys/class/gpio/gpio99/value`
* GPIO3_A3 拉低，RS485 接收： `echo 0 > /sys/class/gpio/gpio99/value`

接口图如下：

![](../../../rk3576_img/Core-3576JD4/usage_uart_interface.jpg)