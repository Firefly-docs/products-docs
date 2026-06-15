# UART Usage

## Description

AIO-3576JD4 use `UART10` for `RS485`, which is `/dev/ttyS10` in system. And `UART6` for `RS232`, which is `/dev/ttyS6` in system.

Interfaces:

![](../../../rk3576_img/EC-A3576JD4/usage_uart_interface.jpg)


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
