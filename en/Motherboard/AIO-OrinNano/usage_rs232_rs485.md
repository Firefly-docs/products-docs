# RS485/RS232

* RS485: Half Duplex, device name: `/dev/ttyTHS2`
    * Before Send Data: `echo 1 > /sys/class/leds/RS485_H_SEND_L_RECV/brightness`
    * Before Receive Data: `echo 0 > /sys/class/leds/RS485_H_SEND_L_RECV/brightness`
* RS232: Full Duplex, device name: `/dev/ttyTHS1`

