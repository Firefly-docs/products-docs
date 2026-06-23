
# UART 使用


## TTL
全双工

设备名称为： `/dev/ttyTHS1`

## RS232
全双工

设备名称为： `/dev/ttyTHS3`

## RS485
半双工

设备名称为： `/dev/ttyTHS4`

发送前先执行： `echo 1 > /sys/class/leds/rs485_re_de/brightness`

接收前先执行： `echo 0 > /sys/class/leds/rs485_re_de/brightness`




