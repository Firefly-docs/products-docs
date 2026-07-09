
# UART 




## RS232
Full Duplex 

device name: `/dev/ttyAMA9`

## RS485
Half Duplex 

device name: `/dev/ttyAMA10`

before send: `echo 1 > /sys/class/leds/rs485_re_de/brightness`

before receive: `echo 0 > /sys/class/leds/rs485_re_de/brightness`


