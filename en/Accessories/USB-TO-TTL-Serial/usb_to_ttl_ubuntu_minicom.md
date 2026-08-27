# Use serial debug on Ubuntu

There are many options available on Ubuntu:

* minicom
* picocom
* kermit

The following shows how to use minicom.

## Install minicom

```
sudo apt-get install minicom
```

To connect the serial port line, see what the serial port device file is. The following example is `/dev/ttyusb0`:

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
+--------------------------------------------------------------------+
                        Minicom Command Summary                      |
|                                                                    |
|              Commands can be called by CTRL-A <key>                |
|                                                                    |
|               Main Functions                  Other Functions      |
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