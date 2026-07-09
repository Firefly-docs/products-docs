# Serial Usage

AIBOX-9075 has 2 RS485 serial.

The device node of RS485_1 is "/dev/ttyHS1"; and the device node of RS485_2 is "/dev/ttyHS3".

## Transmission Test

RS485 needs additional gpio to control send and receive.
```bash
# export gpio no.695
echo 695 > /sys/class/gpio/export

# set direction as output
echo out > /sys/class/gpio/gpio695/direction

# echo 1 means gpio output high level, set RS485_1 to send mode
echo 1 > /sys/class/gpio/gpio695/value

# send the data
echo "RS485_TEST" > /dev/ttyHS1

# echo 0 means gpio output low level, set RS485_1 to receive mode
echo 0 > /sys/class/gpio/gpio695/value

# disable echo and start receiving
stty -echo -F /dev/ttyHS1
cat /dev/ttyHS1

# operate RS485_2 with same method, but the control of RS485_2 is gpio no.693.
```