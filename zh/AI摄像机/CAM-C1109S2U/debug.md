# 串口调试

如果正在进行 U-Boot 或内核开发，USB 串口适配器（ USB 转串口 TTL 适配器的简称）对于检查系统启动日志非常有用，特别是在没有图形桌面显示的情况下。

## 选购适配器

网店上有许多 USB 转串口的适配器，按芯片来分，有以下几种：

* [CP2104](https://item.taobao.com/item.htm?spm=a1z10.5-c.w4002-12605442688.14.aa5e1e8srwECg&id=546045713700)
* PL2303
* CH340

一般来说，采用 CH340 芯片的适配器，性能比较稳定，价格上贵一些。

**注意：**  CAM-C1109S2U 默认的波特率是 1500000，有些USB转串口芯片波特率无法达到 1500000，同一芯片的不同系列也可能会有差异，所以在选购之前一定要确认是否支持。

## 硬件连接

串口转 USB 适配器，有四个引脚：

* 3.3V 电源（NC），不需要连接
* GND，串口的地线，接开发板串口的 GND 针
* TXD，串口的输出线，接开发板串口的 TX 针
* RXD，串口的输入线，接开发板串口的 RX 针

**注意：** 如使用其它串口适配器遇到 TX 和 RX 不能输入和输出的问题，可以尝试对调 TX 和 RX 的连接。

CAM-C1109S2U  串口连接图：

![](../../../rv1126_img/CAM-C1109S2U/debug_connection.jpg)

## 串口参数配置

CAM-C1109S2U 使用以下串口参数：

* 波特率：1500000
* 数据位：8
* 停止位：1
* 奇偶校验：无
* 流控：无

## Windows 上使用串口调试

### 安装驱动

下载驱动并安装:

* [CH340](https://sparks.gogo.co.nz/ch340.html)
* [PL2303](http://www.prolific.com.tw/US/ShowProduct.aspx?pcid=41)
* [CP210X](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers)

如果在 Win8 上不能正常使用 PL2303，参考[这篇文章](http://blog.csdn.net/ropai/article/details/19619951)， 采用 3.3.5.122 或更老版本的旧驱动即可。

如果在 Windows 系统上安装官网的 CP210X 驱动，使用 PUTTY 或 SecureCRT 等工具设置串口波特率为 1500000，如果出现设置不了或无效的问题，可以下载旧版本[驱动](http://www.t-firefly.com/share/index/index/id/a2e8f25f3d53992bf3e04f45b0e6c8e8.html)。

插入适配器后，系统会提示发现新硬件，并初始化，之后可以在设备管理器找到对应的 COM 口：

![](../../../rv1126_img/common/debug_find_com.jpg)

### 安装软件

Windows 上一般用 putty 或 SecureCRT。其中我们推荐使用 MobaXterm 免费版本。这是一款功能强大的终端软件，在这里介绍一下，MobaXterm 的使用方法与之类似。

到这里[下载 MobaXterm](https://mobaxterm.mobatek.net/)：

1. 选择	`session` 为 `Serial`。
2. 将 `Serial port` 修改为在设备管理器中找到的 COM 端口。
3. 设置 `Speed (bsp)` 为 1500000。
4. 点击 `OK` 按钮。

![](../../../rv1126_img/common/debug_set_moba_xterm1.png)
![](../../../rv1126_img/common/debug_set_moba_xterm2.png)

## Ubuntu 上使用串口调试

在 Ubuntu 上可以有多种选择：

* minicom
* picocom
* kermit

篇幅关系，以下就介绍 minicom 的使用。

### 安装 minicom

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

根据提示按O进入设置界面，如下：

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

把光标移动到“Serial port setup”，按enter进入串口设置界面，再输入前面提示的字母，选择对应的选项，设置成如下：

```
   +-----------------------------------------------------------------------+
   | A -    Serial Device      : /dev/ttyUSB0                              |
   | B - Lockfile Location     : /var/lock                                 |
   | C -   Callin Program      :                                           |
   | D -  Callout Program      :                                           |
   | E -    Bps/Par/Bits       : 1500000 8N1                               |
   | F - Hardware Flow Control : No                                        |
   | G - Software Flow Control : No                                        |
   |                                                                       |
   |    Change which setting?                                              |
   +-----------------------------------------------------------------------+
   ```

**注意：**`Hardware Flow Control` 和 `Software Flow Control` 都要设成 No，否则可能导致无法输入。

设置完成后回到上一菜单，选择 `Save setup as dfl` 即可保存为默认配置，以后将默认使用该配置。