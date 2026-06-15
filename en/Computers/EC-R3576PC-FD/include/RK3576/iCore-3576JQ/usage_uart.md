# UART Usage

## Description

ROC-RK3576-PC use `UART10` for `RS485`, which is `/dev/ttyS10` in system. And `UART8` for `RS232`, which is `/dev/ttyS8` in system.Using `UART5` for common uart,which is `/dev/ttyS5` in system.

Interfaces:

![](../../../rk3576_img/EC-R3576PC-FD/usage_uart_interface.jpg)

![](../../../rk3576_img/EC-R3576PC-FD/usage_uart_interface2.jpg)

## RS485 Usage
```
# set transceiver control pin, high level transmit, low level receive
sudo echo 24 > /sys/class/gpio/export
sudo echo out > /sys/class/gpio/gpio24/direction
# close echo
sudo stty -F /dev/ttyS10 -echo
# transmit
sudo echo 1 > /sys/class/gpio/gpio24/value
sudo echo "firefly uart test..." > /dev/ttyS10
# receive
sudo echo 0 > /sys/class/gpio/gpio24/value
sudo cat /dev/ttyS10
```
