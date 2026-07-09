# UART Usage

## Description

AIO-3576C use `UART3` for `RS485`, which is `/dev/ttyS3` in system. And `UART8` for `RS232`, which is `/dev/ttyS8` in system.

Interfaces:

![](../../../rk3576_img/AIO-3576C/usage_uart_interface.jpg)

![](../../../rk3576_img/AIO-3576C/usage_uart_interface2.jpg)

## RS485 Usage
```
# set transceiver control pin, high level transmit, low level receive
sudo echo 22 > /sys/class/gpio/export
sudo echo out > /sys/class/gpio/gpio24/direction
# close echo
sudo stty -F /dev/ttyS3 -echo
# transmit
sudo echo 1 > /sys/class/gpio/gpio22/value
sudo echo "firefly uart test..." > /dev/ttyS3
# receive
sudo echo 0 > /sys/class/gpio/gpio22/value
sudo cat /dev/ttyS3
```
