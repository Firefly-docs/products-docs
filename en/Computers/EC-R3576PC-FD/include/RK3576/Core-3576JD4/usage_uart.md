# UART Usage

## Description

ROC-RK3576-PC use UART3 for RS485, which is /dev/ttyS3 in system. And UART8 for RS232, which is /dev/ttyS8 in system.

And GPIO3_A3 is used for direction control of RS485, pull it up will change RS485 to send mode, pull it down will change RS485 to receive mode.

Users can use /sys/class/gpio subsystem to control this GPIO.

* GPIO3_A3 config output

```
echo 99 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio99/direction
```

* GPIO3_A3 set high, RS485 send: `echo 1 > /sys/class/gpio/gpio99/value`
* GPIO3_A3 set low, RS485 receive: `echo 0 > /sys/class/gpio/gpio99/value`

Interfaces:

![](../../../rk3576_img/EC-R3576PC-FD/usage_uart_interface.jpg)