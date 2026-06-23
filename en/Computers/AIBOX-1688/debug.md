# Serial port debugging

## Using Type-C for serial port debugging

The AIO-1688JD4 can be connected to a PC using a Type-C cable for serial port debugging:

![](../../../bm1688_img/AIBOX-1688/type-c_connection.png)

Please note that if you need to enable the Type-C debug port function, you must set the corresponding dip switch to the specified position:

![](../../../bm1688_img/AIBOX-1688/debug_switch.png)

* ON: Indicates that the Type-C serial port is enabled
* 1: Indicates that it is used as a standard USB2.0 (Host only) interface

### Serial port parameter configuration

The AIBOX-1688 uses the following serial port parameters:

* Baud rate: 115200
* Data bits: 8
* Stop bits: 1
* Parity: None
* Flow control: None

### Using serial port debugging on Windows

On Windows, PuTTY or SecureCRT software is generally used. We recommend using the free version of MobaXterm. This is a powerful terminal software, and here we introduce it; the usage methods of other software are similar.

To here[Download MobaXterm](https://mobaxterm.mobatek.net/)：

1. Select `session` For `Serial`。
2. Will `Serial port` Change to the COM port found in the Device Manager.
3. Set `Speed (bsp)` For `115200`。
4. Click `OK` Button.

![](../../../bm1688_img/debug_set_MobaXterm1.PNG)
![](../../../bm1688_img/debug_set_MobaXterm2.PNG)

### Using Serial Port Debugging on Linux

There are various options on Linux:

* minicom
* picocom
* kermit

Due to space constraints, we will introduce the use of minicom.

Install minicom:

```
sudo apt-get install minicom
```

After connecting the serial cable, check what the serial device file is (**Note, if using a Type-C interface, the device file in the Linux system is: /dev/ttyACMx**), the following example is `/dev/ttyUSB0`：

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

The above prompt `CTRL-A Z` is an escape key, press `Ctrl-a` then press again `z` to bring up the menu:

```
   +-------------------------------------------------------------------+
                          Minicom Command Summary                      |
  |                                                                    |
  |              Commands can be called by CTRL-A <key>                |
  |                                                                    |
  |               Main Functions                  Other Functions      |
  |                                                                    |
  | Dialing directory..D  run script (Go)....G | Clear Screen.......C  |
  | Send files.........S  Receive files......R | cOnfigure Minicom..O  |
  | comm Parameters....P  Add linefeed.......A | Suspend minicom....J  |
  | Capture on/off.....L  Hangup.............H | eXit and reset.....X  |
  | send break.........F  initialize Modem...M | Quit with no reset.Q  |
  | Terminal settings..T  run Kermit.........K | Cursor key mode....I  |
  | lineWrap on/off....W  local Echo on/off..E | Help screen........Z  |
  | Paste file.........Y  Timestamp toggle...N | scroll Back........B  |
  | Add Carriage Ret...U                                               |
  |                                                                    |
  |             Select function or press Enter for none.               |
  +--------------------------------------------------------------------+
```

Press according to the prompt `O` to enter the settings interface, as follows:

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

Move the cursor to 'Serial port setup', press enter to enter the serial port settings interface, then input the letters prompted earlier, select the corresponding options, and set them as follows:

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

**Note:**`Hardware Flow Control` and `Software Flow Control` must both be set to No, otherwise it may lead to input issues.

After completing the settings, return to the previous menu and select `Save setup as dfl` to save as the default configuration, which will be used by default in the future.

## Using the DEBUG port for debugging

If you want to perform U-Boot or kernel development, you need to use the DEBUG port for debugging. The operation is consistent with Type-C debugging, but attention should be paid to the adapter and its driver issues.

### Choosing an adapter

There are many USB to serial port adapters available online, categorized by chip type as follows:

|                                                  Serial port                                                 | Maximum baud rate |   Recommended?  |                                                     Review                                                     |                                                      Purchase link                                                      |
| :----------------------------------------------------------------------------------------------------------: | :---------------: | :-------------: | :------------------------------------------------------------------------------------------------------------: | :---------------------------------------------------------------------------------------------------------------------: |
| [CP2104](https://item.taobao.com/item.htm?spm=a1z10.5-c.w4002-12605442688.14.aa5e1e8srwECg\&id=546045713700) |       2Mbps       |  Recommendation |                      Supports high baud rate communication, good stability and durability                      | [Click to purchase](https://item.taobao.com/item.htm?spm=a1z10.5-c.w4002-12605442688.14.aa5e1e8srwECg\&id=546045713700) |
|                                                     CH340                                                    |       2Mbps       | Not recommended | In actual use, it has been found that many CH340 adapters on the market do not achieve a baud rate of 1.5 Mbps |                                                                                                                         |
|                                                    PL2303                                                    |      1.2Mbps      | Not recommended |                                    Maximum baud rate does not reach 1.5 Mbps                                   |                                                                                                                         |

Generally speaking, adapters using the CH340 chip have relatively stable performance, but are a bit more expensive.

**Note:** The default baud rate for AIBOX-1688 is 115200.

### Hardware connection

USB to Serial Adapter, with four pins.

* 3.3V Power (NC), no connection needed
* GND, the ground line of the serial port, connect to the GND pin of the development board
* TXD, the output line of the serial port, connect to the TX pin of the serial port of the development board
* RXD, the input line of the serial port, connect to the RX pin of the serial port of the development board

**Note:** If you encounter the problem that TX and RX cannot be input and output when using other serial adapters, you can try to swap the TX and RX connections.

AIO-1688JD4 DEBUG port:

![](../../../bm1688_img/AIBOX-1688/debug_connection.png)

### Driver Installation

Windows systems need to install the adapter driver (Linux does not):

Download the driver and install it.

* [CH340](https://sparks.gogo.co.nz/ch340.html)
* [PL2303](http://www.prolific.com.tw/US/ShowProduct.aspx?pcid=41)
* [CP210X](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers)

If you install the CP210X driver from the official website on Windows system, and set the baud rate of serial port to 115200 by using tools such as PUTTY or SecureCRT, and if there is a problem of not being able to set it or it is invalid, you can download the old version of [Driver](http://www.t-firefly.com/share/index/). index/id/a2e8f25f3d53992bf3e04f45b0e6c8e8.html).

After plugging in the adapter, the system will prompt that new hardware is found and initialised, after which you can find the corresponding COM port in Device Manager:

![](../../../bm1688_img/debug_find_com.jpg)