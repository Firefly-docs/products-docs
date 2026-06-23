
# UART 


## TTL
Full Duplex 

device name: `/dev/ttyTHS1`

## RS232
Full Duplex 

device name: `/dev/ttyTHS3`

## RS485
Half Duplex 

device name: `/dev/ttyTHS4`

before send: `echo 1 > /sys/class/leds/rs485_re_de/brightness`

before receive: `echo 0 > /sys/class/leds/rs485_re_de/brightness`




