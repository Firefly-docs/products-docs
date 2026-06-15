# I2C 

## Introduction
There are nine I2C controllers on the RK3588S, and data on the I2C bus can be transferred at rates up to 100k bit/s in standard mode and 400k bit/s in fast mode.
AIO-3588SJD4-AI leads to two I2C for users to configure.

![](../../../rk3588_img/Core-3588SJD4-AI/usage_i2c_interface.png)

This article describes how to debug i2c

## i2cdetect command

View the list of I2C buses installed in the system
```
rk3588s_firefly_aio_3588sg:/ # i2cdetect -l
i2c-10  i2c             DP-AUX                                  I2C Adapter
i2c-8   i2c             rk3x-i2c                                I2C Adapter
i2c-6   i2c             rk3x-i2c                                I2C Adapter
i2c-4   i2c             rk3x-i2c                                I2C Adapter
i2c-2   i2c             rk3x-i2c                                I2C Adapter
i2c-0   i2c             rk3x-i2c                                I2C Adapter
i2c-9   i2c             fde50000.dp                             I2C Adapter
i2c-7   i2c             rk3x-i2c                                I2C Adapter
```

Look at the device drivers under i2c6
```
rk3588s_firefly_aio_3588sg:/ # i2cdetect -y 6
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- UU -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- UU -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```
You can see that two device drivers are loaded on the i2c6. The device addresses are 0x48 and 0x5d.  
UU indicates that the driver has been loaded to that address.  The real device is not necessarily connected.

## i2cdump command
View the value of the register of the 0x64 device 0x00-0xff under i2c4
```
rk3588s_firefly_aio_3588sg:/ # i2cdump -f  -y 4 0x63
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    0123456789abcdef
00: 0d 00 34 3a 63 26 81 00 00 00 40 8a aa 50 0f f5    ?.4:c&?...@??P??
10: 3c 00 00 00 00 00 00 00 b0 c4 bf b9 9b 97 e0 cf    <.......????????
20: c1 cd bb 9d 88 7c 65 56 52 50 4e 97 79 d2 de ff    ?????|eVRPN?y???
30: e5 b4 71 7c b0 c5 ae 93 9d b5 cf d5 c6 b0 99 89    ??q|????????????
40: 82 85 91 a8 c1 c9 b0 43 00 00 90 02 00 00 00 00    ???????C..??....
50: 00 00 64 00 00 00 00 00 00 00 00 00 00 00 00 a2    ..d............?
60: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
70: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
80: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
90: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
a0: 00 00 00 00 00 00 64 00 00 e6 00 00 00 00 00 00    ......d..?......
b0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
```
You can see the value of the register <br>
address: 0x00     value: 0x0d  <br>
address: 0x01     value: 0x00  <br>
address: 0x02     value: 0x34  <br>
address: 0x03     value: 0x3a  <br>
...  <br>


## i2cget command
Gets the value of the 0x0d register of the i2c4 0x63 device
```
rk3588s_firefly_aio_3588sg:/ # i2cget -f -y 4 0x63 0x0d
0x50
```

## i2cset command
Write data to the 0x0d register of the 0x63 device under i2c4, where 0x51 is data and b indicates that the data length is 8bit
```
rk3588s_firefly_aio_3588sg:/ # i2cset -fy 4 0x63 0x0d 0x51 b
rk3588s_firefly_aio_3588sg:/ #
rk3588s_firefly_aio_3588sg:/ # i2cget -f -y 4 0x63 0x0d
0x51
```

Write data to the 0x0c register of the 0x63 device under i2c4, where 0xff00 is the data and w indicates that the data length is 16bit
```
rk3588s_firefly_aio_3588sg:/ # i2cset -fy 4 0x63 0x0c 0xff00 w
rk3588s_firefly_aio_3588sg:/ #
rk3588s_firefly_aio_3588sg:/ # i2cget -f -y 4 0x63 0x0c
0x00
rk3588s_firefly_aio_3588sg:/ # i2cget -f -y 4 0x63 0x0d
0xff
```
 