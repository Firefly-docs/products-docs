# Serial port debugging

## Caution

In Firefly-RK3128 development board, <font color=#ff0000 size=3>pins of the debug uart2 port are multiplexed with TF card interface, making them mutual exclusive.</font> That is to say, if uart2 port is in used, TF card must be plugin out, and if TF card is in use, the uart2 port cannot be connected.

## Purchase the adapter

There are many USB-to-serial adapters on the online store, and they have the following types according to the chips:

* [CP2104](https://www.firefly.store/products/usb-to-uart-module-cp2104)
* PL2303
* CH340

In general, the adapter with CH340 has stable performance and is more expensive.

## Hardware Connection

There are four different color of connection line for the serial-to-USB adapter:

* Red: 3.3V Power, no need to connect.
* Black: GND，Ground, connect to GND pin of the board.
* White: TXD，Transmit，connect to TX pin of the board.
* Green: RXD，Receive, connect to RX pin of the board.

## Connection parameters

The following serial port parameters are used by the Firefly-RK3128:

* Baud rate: 115200
* Data bit: 8
* Stop bit: 1
* Parity check: none
* Flow control: none

## Serial port debugging is used on Windows

### Install the driver

Download the driver and install it:

* [CP2104](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers)
* CH340 [[1]](https://sparks.gogo.co.nz/ch340.html)
* PL2303 [[2]](http://www.prolific.com.tw/US/ShowProduct.aspx?pcid=41)

If PL2303 does not works under Win8, please find drivers with version 3.3.5.122 or before.  
Once the adapter is plugged in, you are prompted to discover the new hardware and initialize it, then you can find the corresponding COM port in the device manager: 
![](../../../rk3128_img/Core-3128J/win_com.png)

### Install software

Under Windows, the common softwares with serial port support are putty and SecureCRT. Putty is open source software. We give a brief introduction here. The use of SecureCRT is similar.  
Download putty from [[3]](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) .We recommend to download putty.zip, which contains other useful utilities.  
Extract and run <font color=#ff0000 size=3>`PUTTY.exe`</font>. 

* Select "Connection type" to "Serial".
* Modify "Serial line" to the COM port found in the device manager.
* Set "Speed" to 115200.
* Click "Open" button.

![](../../../rk3128_img/Core-3128J/win_putty.png)

## Serial port debugging is used on Ubuntu

There are several options available on Ubuntu:

* picocom
* minicom
* kermit

It is relatively simple to use the picocom, and next we will introduce the picocom, and other software is similar. 

Install：

```
sudo apt-get install picocom
```

If the serial line is connected, you can see what the serial device file is, and the following example is <font color=#ff0000 size=2>/dev/ttyUSB0</font>

```
$ ls /dev/ttyUSB*
/dev/ttyUSB0
```

Run:

```
$ picocom -b 115200 /dev/ttyUSB0
picocom v1.7

port is        : /dev/ttyUSB0
flowcontrol    : none
baudrate is    : 115200
parity is      : none
databits are   : 8
escape is      : C-a
local echo is  : no
noinit is      : no
noreset is     : no
nolock is      : no
send_cmd is    : sz -vv
receive_cmd is : rz -vv
imap is        : 
omap is        : 
emap is        : crcrlf,delbs,

Terminal ready
```

Messages above say that <font color=#ff0000 size=3>`Ctrl-a`</font> is the escape key. Pressing <font color=#ff0000 size=3>`Ctrl-a Ctrl-q`</font> can quit the terminal. Besides <font color=#ff0000 size=3>`Ctrl-q`</font>, there are several control commands frequently used:

* Ctrl-u : increase baud rate
* Ctrl-d : decrease baud rate
* Ctrl-f : cycle through flow controls (RTS/CTS, XON/XOFF,  and none).
* Ctrl-y : cycle through parity check (even, odd and none).
* Ctrl-b : cycle throught data bit (5, 6, 7, 8).
* Ctrl-c : toggle local echo.
* Ctrl-v : show program options (like baud rate, data bits, etc).