# Ubuntu 上使用串口调试

在 Ubuntu 上可以有多种选择：

* minicom
* picocom
* kermit

篇幅关系，以下就介绍 minicom 的使用。

## 安装 minicom

```
sudo apt-get install minicom
```

连接好串口线的，看一下串口设备文件是什么，下面示例是 `/dev/ttyUSB0`

```
$ ls /dev/ttyUSB*
/dev/ttyUSB0
```

运行：

```
$ sudo minicom
Welcome to minicom 2.7
OPTIONS: I18n
Compiled on Jan  1 2014, 17:13:19.
Port /dev/ttyUSB0, 15:57:00
Press CTRL-A Z for help on special keys
```

以上提示 CTRL-A Z 是转义键，按 Ctrl-a 然后再按 Z 就可以调出帮助菜单。

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