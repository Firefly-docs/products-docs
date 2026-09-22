# Hardware Function Usage
## Debug Serial Port

The debug serial port of the IHC-3308GW is UART4. The serial port parameters are: baud rate 1500000, 8 data bits, 1 stop bit, no parity, no flow control.

The debug serial port is a TTL-level interface. An external USB to TTL serial module is required for debugging.

For the connection and usage of the debug serial port, please refer to: [Debug Serial Port](usb_to_ttl.md).

## UART (RS485 / RS232)

The expansion board expands multiple serial ports, including 3 `RS485` ports and 1 `RS232` port.

The kernel supports the above serial port functions by default. The device files corresponding to each serial port are as follows:

```bash
RS485_1: /dev/ttysWK0
RS485_2: /dev/ttysWK1
RS485_3: /dev/ttysWK2
RS232  : /dev/ttysWK3
```

Take RS485_1 as an example:

* Connection

Connect the A and B pins of RS485_1 to the A and B pins of the host serial port adapter (USB to 485 serial module) respectively.

* Open the serial terminal of the host

Open kermit in the terminal and set the baud rate:

```bash
$ sudo kermit
C-Kermit> set line /dev/ttysWK0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

`/dev/ttyUSB0` is the device file of the USB to serial port adapter recognized by the host.

* Send data

Run the following command on the device:

```bash
echo "Firefly RS485 test..." > /dev/ttysWK0
```

The serial terminal of the host will receive the string "Firefly RS485 test...".

* Receive data

First run the following command on the device:

```bash
cat /dev/ttysWK0
```

Then enter the string "Firefly RS485 test..." in the serial terminal of the host, and the same string can be seen on the device side. The verification methods of RS232 (`/dev/ttysWK3`), RS485_2 and RS485_3 are the same.

## CAN

- Connection

Just connect the `CANH` and `CANL` of the device to the `CANH` and `CANL` of the communication peer correspondingly.

* Send data

```bash
ip link set can0 down
ip link set can0 type can bitrate 250000
ip link set can0 up
cansend can0 123#1122334455667788
```

* Receive data

```bash
ip link set can0 down
ip link set can0 type can bitrate 250000
ip link set can0 up
candump can0
```

* Loopback mode test

```bash
ip link set can0 down
ip link set can0 type can bitrate 50000 loopback on
ip link set can0 up
candump can0 &
cansend can0 123#11223344556677
```

## DIN

The IHC-3308GW supports an optocoupler-isolated interface, where `DI` corresponds to `INPUT1` in the hardware schematic, and `COM` corresponds to `INPUT_COM` in the hardware schematic.

- Circuit schematic

<center>

<img alt="" src="../../../rk3308_img/IHC-3308GW/gpio_input.png" width="700">
</center>

* Detection

When `INPUT1` and `INPUT_COM` are connected, `GPIO_INPUT1` will detect a low level; when `INPUT1` and `INPUT_COM` are disconnected, `GPIO_INPUT1` will detect a high level.

The corresponding `GPIO` is as follows:

```bash
GPIO_INPUT1: GPIO1_A6, 38
```

The detection method is as follows:

```bash
# Apply for the GPIO
echo 38 > /sys/class/gpio/export
# Set it as input
echo in > /sys/class/gpio/gpio38/direction
# Read the level value
cat /sys/class/gpio/gpio38/value
```

## DOUT

The IHC-3308GW supports a relay interface. `DO` corresponds to `OUTPUT1` in the hardware schematic, and `COM` corresponds to `RELAY_COM1` in the hardware schematic.

* Circuit schematic

<center>

<img alt="" src="../../../rk3308_img/IHC-3308GW/relay_ctl.png" width="700">
</center>

* Control

When `RELAY_CTL1` outputs a low level, `OUTPUT1` and `RELAY_COM1` are disconnected; when `RELAY_CTL1` outputs a high level, `OUTPUT1` and `RELAY_COM1` are connected.

The corresponding `GPIO` is as follows:

```
RELAY_CTL1: GPIO1_B2, 42
```

The control method is as follows:

```bash
# Apply for the GPIO
echo 42 > /sys/class/gpio/export
# Set it as output
echo out > /sys/class/gpio/gpio42/direction
# Set the level value, 1 / 0
echo 1 > /sys/class/gpio/gpio42/value
```

## LED

The IHC-3308GW supports 6 customizable LEDs, and the corresponding GPIOs are as follows:

| **L1** | GPIO2_A7 (gpio71)      |
| ------ | ---------------------- |
| **L2** | **GPIO2_A6 (gpio70)**  |
| **L3** | **GPIO2_B3 (gpio74)**  |
| **L4** | **GPIO2_B2 (gpio73)**  |
| **L5** | **GPIO2_B5 (gpio76)**  |
| **L6** | **GPIO2_B4 (gpio75)**  |

The control method is as follows, taking L1 as an example:

```bash
# On
echo 1 > /sys/class/leds/firefly\:green\:L1/brightness
# Off
echo 0 > /sys/class/leds/firefly\:green\:L1/brightness
```
