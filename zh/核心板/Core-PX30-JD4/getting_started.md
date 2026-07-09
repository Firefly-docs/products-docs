# 上手教程

## 入手指南
### 配件

AIO-PX30-JD4 的标准套装包含以下配件：

* CORE-PX30-JD4 核心板一块
* MB-JD4-RK3328&PX30 底板一块
* 铜管天线×1  
* 12V-2A电源适配器一个
* Type-A转Type-C数据线×1

另外可以选购的配件有：

* Firefly串口模块

另外，在使用过程中，你可能需要以下配件：
*    显示设备
     * LVDS 转接模块，10.1寸LVDS屏幕
*    网络
     *   100M 以太网线缆，及有线路由器
     *   WiFi 路由器
*    输入设备
     *   USB 无线/有线的鼠标/键盘
     *   红外遥控器(需要接上红外接收器)
*    升级固件，调试
     *   Type A 转 Type C线
     *   串口转 USB 适配器 

 <a id="firmware-format"></a>
### 固件类型

 固件有两种格式：

 * 原始固件(raw firmware)
 * RK固件(Rockchip firmware)

<a id="raw-firmware-format"></a>
    [原始固件]是一种能以逐位复制的方式烧写到存储设备的固件，是存储设备的原始映像。原始固件一般烧写到SD卡中，但也可以烧写到eMMC中。烧写原始固件有许多工具可以选用：
		
<a id="rk-firmware-format"></a>
    [RK固件]是以Rockchip专有格式打包的固件，使用Rockchip提供的upgrade_tool(Linux)或AndroidTool(Windows)工具烧写到eMMC闪存中。RK固件是Rockchip的传统固件打包格式，常用于Android设备上。另外，Android的RK固件也可以使用SD Firmware Tool工具烧写到SD卡中。

<a id="partition-image"></a>
    [分区映像]是分区的映像数据，用于存储设备对应分区的烧写。例如，编译Android SDK会构建出`boot.img`、`kernel.img`和`system.img`等分区映像文件，`kernel.img`会被写到eMMC或SD卡的“kernel”分区。

### 下载和烧写固件


以下是支持的系统列表：

- Android 8.1
- Ubuntu 18.04

根据所使用的操作系统来选择合适的工具去烧写固件：

- 烧写 SD卡
  + 图形界面烧写工具：
	* [Etcher] (windows/linux/Mac)
  + 命令行烧写工具
	* [dd] (Linux)

- 烧写 eMMC
  + 图形界面烧写工具：
	* [AndroidTool] (Windows)
  + 命令行烧写工具：
	* [upgrade_tool] (Linux)

### 开机
确认主板配件连接无误后，将电源适配器插入带电的插座上，电源线接口插入开发板，开发板第一次加电会自动开机。 

开机时，蓝色的电源指示灯会亮起。如果板子接了lvds屏幕，可以看到Firefly 官方logo。

[RK固件]:getting_started.html#raw-firmware-format
[原始固件]:getting_started.html#rk-firmware-format
[分区映像]:getting_started.html#partition-image
[固件类型]:getting_started.html#firmware-format
[SD Firmware Tool]: upgrade_firmware_rk.html#SD_Firmware_Tool
[Etcher]: upgrade_firmware_sd.html#Etcher
[dd]: upgrade_firmware_sd.html#dd
[AndroidTool]: upgrade_firmware.html#Androidtool
[upgrade_tool]: upgrade_firmware.html#upgrade_and_upgrade_tool
[rkdeveloptool]: upgrade_firmware.html#upgrade_and_rkdeveloptoo

## 串口调试   
### 选购适配器

网店上有许多USB转串口的适配器，按芯片来分，有以下几种：

* [CP2104](https://item.taobao.com/item.htm?spm=a1z10.5-c.w4002-12605442688.14.aa5e1e8srwECg&id=546045713700)
* PL2303  
* CH340   

* 注意：PX30默认的波特率是1500000，有些USB转串口芯片波特率无法达到1500000，同一芯片的不同系列也可能会有差异，所以在选购之前一定要确认是否支持。

### 硬件连接

串口转 USB 适配器，有四个pin脚：

* 3.3V 电源（NC），不需要连接
* GND，串口的地线，接开发板串口的 GND 针
* TXD，串口的输出线，接开发板串口的 TX 针
* RXD，串口的输入线，接开发板串口的 RX 针

**注：如使用其它串口适配器遇到TX和RX不能输入和输出的问题，可以尝试对调TX和RX的连接。**

AIO-PX30-JD4串口连接图：

![](../../../px30_img/uart.jpg)
### 连接参数

AIO-PX30-JD4 使用以下串口参数：

* 波特率：1500000
* 数据位：8
* 停止位：1
* 奇偶校验：无
* 流控：无

### Windows 上使用串口调试

#### 安装驱动

下载驱动并安装:

* [CH340](https://sparks.gogo.co.nz/ch340.html)
* [PL2303](http://www.prolific.com.tw/US/ShowProduct.aspx?pcid=41)
* [CP210X](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers)

如果在 Win8 上不能正常使用 PL2303，参考[这篇文章](http://blog.csdn.net/ropai/article/details/19619951)， 采用 3.3.5.122 或更老版本的旧驱动即可。  

如果在 Windows 系统上安装官网的CP210X驱动，使用PUTTY或SecureCRT等工具设置串口波特率为1500000，如果出现设置不了或无效的问题，可以下载旧版本[驱动](http://t-firefly.oss-cn-hangzhou.aliyuncs.com/product/Tools/Driver/CP210X/CP210x_VCP_Windows.zip)

插入适配器后，系统会提示发现新硬件，并初始化，之后可以在设备管理器找到对应的 COM 口：   
![](../../../px30_img/debug2.png)

#### 安装软件

Windows 上一般用 putty 或 SecureCRT。其中 putty 是开源软件，在这里介绍一下，SecureCRT 的使用方法与之类似。
到这里[下载 putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)，建议下载 `putty.zip`，它包含了其它有用的工具。

解压后运行 PUTTY.exe，选择 Connection type（连接类型）为 Serial（串口），将 Serial line（串口线）设置成设备管理器所看到的  COM 口，并将 Speed（波特率）设置为 1500000，按 Open（打开）即可:

![](../../../px30_img/debug3.png)

### Ubuntu 上使用串口调试

在 Ubuntu 上可以有多种选择：

* minicom

以下就介绍 minicom的使用。

#### 安装
```
sudo apt-get install minicom
```
连接好串口线的，看一下串口设备文件是什么，下面示例是 /dev/ttyUSB0
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
* 注意：Hardware Flow Control和Software Flow Control都要设成No，否则可能导致无法输入。

设置完成后回到上一菜单，选择“Save setup as dfl”即可保存为默认配置，以后将默认使用该配置。
