# Serial debug

If you are doing U-Boot or kernel development, the USB serial adapter (short for USB to serial TTL adapter) is very useful for checking the system startup log, especially when there is no graphical desktop display.

## Buy Adapter

There are many USB adapter to serial port on the shop, divided by chip, there are the following:

* [CP2104](https://item.taobao.com/item.htm?spm=a1z10.5-c.w4002-12605442688.14.aa5e1e8srwECg&id=546045713700)
* PL2303
* CH340

## Hardware Connection

Serial port to USB adapter, there are four pins:

* 3.3V Power, no need to connect.
* GND，Ground, connect to GND pin of the board.
* TXD，Transmit，connect to TX pin of the board.
* RXD，Receive, connect to RX pin of the board.

**Note:** If you encounter the problem that TX and RX cannot input and output when using other serial adapters, you can try to exchange TX and RX connections.

AIO-3288J serial port connection diagram:

![](../../../rk3288_img/AIO-3288J/debug_connection.jpg)


## Parameter Setting

AIO-3288J use the following serial parameters:

* Baud rate: 115200
* Data bit: 8
* Stop bit: 1
* Parity check: none
* Flow control: none

## Use serial on Windows

### Install Driver

Download driver and install:

* [CH340](https://sparks.gogo.co.nz/ch340.html)
* [PL2303](http://www.prolific.com.tw/US/ShowProduct.aspx?pcid=41)
* [CP210X](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers)

If you can’t use PL2303 normally on Win8, use 3.3.5.122 or older version of the old driver, please refer to [This article](https://blog.csdn.net/ropai/article/details/19619951). Please find drivers with version 3.3.5.122 or before.

After the adapter is inserted, the system will prompt for the discovery of new hardware and initialization, and then the corresponding COM port can be found in the device manager:

![](../../../rk3288_img/debug_find_com.jpg)

### Install Software

Putty or SecureCRT is generally used on Windows. Among them, we recommend using the free version of MobaXterm. This is a powerful terminal software. Here we will introduce how to use MobaXterm.

[Download MobaXterm](https://mobaxterm.mobatek.net/):

* Select `session` to `Serial`.
* Modify `Serial port` to the COM port found in the device manager.
* Set `Speed (bsp)` to 115200.
* Click `OK` button.

![](../../../rk3288_img/debug_set_MobaXterm1.PNG)
![](../../../rk3288_img/debug_set_MobaXterm2.PNG)

## Use serial debug on Ubuntu

There are many options available on Ubuntu:

* minicom
* picocom
* kermit

The following shows how to use minicom.

### Install minicom

```
sudo apt-get install minicom
```

To connect the serial port line, see what the serial port device file is. The following example is `/dev/ttyUSB0`:

```
$ ls /dev/ttyUSB*
/dev/ttyUSB0
```

Run:

```
$ sudo minicom
Welcome to minicom 2.7
OPTIONS: I18n
Compiled on Jan  1 2014, 17:13:19.
Port /dev/ttyUSB0, 15:57:00
Press CTRL-A Z for help on special keys
```

Based on the above tips: Press `Ctrl-a` and then press `Z` again to bring up the help menu.
```
+-------------------------------------------------------------------+
|                      Minicom Command Summary                      |
|                                                                   |
|              Commands can be called by CTRL-A                     |
|                                                                   |
|               Main Functions                  Other Functions     |
|                                                                   |
| Dialing directory..D  run script (Go)....G | Clear Screen.......C |
| Send files.........S  Receive files......R | cOnfigure Minicom..O |
| comm Parameters....P  Add linefeed.......A | Suspend minicom....J |
| Capture on/off.....L  Hangup.............H | eXit and reset.....X |
| send break.........F  initialize Modem...M | Quit with no reset.Q |
| Terminal settings..T  run Kermit.........K | Cursor key mode....I |
| lineWrap on/off....W  local Echo on/off..E | Help screen........Z |
| Paste file.........Y  Timestamp toggle...N | scroll Back........B |
| Add Carriage Ret...U                                              |
|                                                                   |
|             Select function or press Enter for none.              |
+-------------------------------------------------------------------+
```

Press O to enter the setting interface, as follows:

```
           +-----[configuration]------+
           | Filenames and paths      |
           | File transfer protocols  |
           | Serial port setup        |
           | Modem and dialing        |
           | Screen and keyboard      |
           | Save setup as dfl        |
           | Save setup as..          |
           | Exit                     |
           +--------------------------+
```

Move the cursor to `Serial port setup`, press enter to enter the Serial port setup interface, then enter the letter prompted earlier, select the corresponding option, and set it as follows:

```
  +-----------------------------------------------------------------------+
   | A -    Serial Device      : /dev/ttyUSB0                              |
   | B - Lockfile Location     : /var/lock                                 |
   | C -   Callin Program      :                                           |
   | D -  Callout Program      :                                           |
   | E -    Bps/Par/Bits       : 115200 8N1                                |
   | F - Hardware Flow Control : No                                        |
   | G - Software Flow Control : No                                        |
   |                                                                       |
   |    Change which setting?                                              |
   +-----------------------------------------------------------------------+
```

<font color=#ff0000>**Note:** `Hardware Flow Control` and `Software Flow Control` should be set to `No`, otherwise it may not be impossible to input.</font>

After finishing the setting, go back to the previous menu and select `Save setup as dfl` to save as the default configuration, which will be used by default later.