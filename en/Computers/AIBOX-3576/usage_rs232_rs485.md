# RS485/RS232 use

## RS485

AIBOX-3576 has a RS485 interface, the device name is `/dev/ttyS2`, supports full duplex, and the default baud rate is `9600`.

The test method is as follows:

(1) Connect hardware

Connect the A, B, and GND pins of the development board RS485 to the A, B, and GND pins of the PC serial port adapter (USB to 485 to serial port module) respectively.

(2) Open the PC serial port terminal

Open kermit in terminal and set the baud rate:

```shell
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

Among them, `/dev/ttyUSB0` is the device file of the USB-to-serial adapter, and it depends on the actual PC recognition.

(3) Send data

The device file for RS485 is `/dev/ttyS2`. Run the following commands on the device:

```shell
sudo -s
stty -F /dev/ttyS2 9600 -echo
echo firefly RS485 test... > /dev/ttyS2
```
The serial port terminal in the PC can receive the string “firefly RS485 test…”

(4) Receive data

First run the following command on the device:

```shell
sudo -s
cat /dev/ttyS2
```

Then enter the string "Firefly RS485 test..." on the serial terminal of the PC, and you can see the same string on the device side.

## RS232

In addition to RS485, AIBOX-3576 also has an RS232 interface, the device name is `/dev/ttyS1`, supports full duplex, and the default baud rate is `115200`.

It should be noted that RS232 is used for login by default, so if users want to use it, they should enter the following command to restore it to the normal communication serial port function:

```shell
sudo systemctl disable --now serial-getty@ttyS1.service
```

The test method is similar to the steps of RS485, only need to pay attention to the device name and baud rate.

