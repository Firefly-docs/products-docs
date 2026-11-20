# RS485/RS232

* RS485: Half Duplex, device name: `/dev/ttyTHS2`, default baud rate `9600`
    * Before sending data: `echo 1 > /sys/class/leds/RS485_H_SEND_L_RECV/brightness`
    * Before receiving data: `echo 0 > /sys/class/leds/RS485_H_SEND_L_RECV/brightness`
* RS232: Full Duplex, device name: `/dev/ttyTHS1`, default baud rate `115200`

## Debugging Method [step]

Users can use different USB to serial adapters on the host to send and receive data through the board's serial port depending on the interface. Take the RS485 debugging steps as an example:

(1) Connect the hardware

Connect the RS485B-A, RS485B-B and GND pins of the board to the A, B and GND pins of the host serial port adapter (USB to 485 serial port module) respectively.

(2) Open the serial terminal on the host

Open kermit in the terminal and set the baud rate:

```
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

* `/dev/ttyUSB0` is the device file of the USB to serial port adapter

(3) Send data

The device file of RS485 is `/dev/ttyTHS2`. Run the following commands on the device:

```
sudo -s
stty -F /dev/ttyTHS2 9600 -echo
echo firefly RS485 test... > /dev/ttyTHS2
```

The serial terminal on the host will receive the string "firefly RS485 test..."

(4) Receive data

First run the following commands on the device:

```
sudo -s
cat /dev/ttyTHS2
```

Then enter the string "Firefly RS485 test..." in the serial terminal of the host, and the same string can be seen on the device side.