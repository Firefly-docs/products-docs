# 硬件功能使用

## 调试串口

AIBOX-Orin Nano 板载 Type-C Console 调试串口，无需外接串口模块，使用 Type-C 数据线连接主机与 PC 即可进行串口调试。

## 登录

AIBOX-Orin Nano 登录方式有两种，一种是通过 HDMI 登录，一种是通过 Console（调试串口）进行终端登录。


### HDMI 登录
在界面登录的时候，Ubuntu 的会话以及密码均选择 `nvidia` 进行登录。

### Console 登录（调试串口）
Type-C 接入 Console 口，登录账号与密码均为 `nvidia`。<br>

<center>

<img alt="" src="../../../aibox_img/AIBOX-3588S/AIBOX-3588S-console.png" width="400">
</center>

#### 串口参数配置

AIBOX-Orin Nano 使用以下串口参数：

* 波特率：115200
* 数据位：8
* 停止位：1
* 奇偶校验：无
* 流控：无

#### Windows 上使用串口调试

Windows 上一般用 putty 或 SecureCRT 软件。其中我们推荐使用 MobaXterm 免费版本。这是一款功能强大的终端软件，在这里介绍一下，其他软件的使用方法与之类似。

到这里[下载 MobaXterm](https://mobaxterm.mobatek.net/)：

1. 选择 `session` 为 `Serial`。
2. 将 `Serial port` 修改为在设备管理器中找到的 COM 端口。
3. 设置 `Speed (bsp)` 为 `115200`。
4. 点击 `OK` 按钮。

<center>

<img alt="" src="../../../aibox_img/debug_set_MobaXterm1.PNG" width="800">
</center>
<center>

<img alt="" src="../../../aibox_img/debug_set_MobaXterm2.PNG" width="800">
</center>

#### Linux 上使用串口调试

在 Linux 上可以有多种选择：

* minicom
* picocom
* kermit

篇幅关系，以下就介绍 minicom 的使用。

安装 minicom：

```
sudo apt-get install minicom
```

连接好串口线的，看一下串口设备文件是什么（**注意，如果 Type-C 接口，在 Linux 系统的设备文件为：/dev/ttyACMx**），下面示例是 `/dev/ttyUSB0`：

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

以上提示 `CTRL-A Z` 是转义键，按 `Ctrl-a` 然后再按 `z` 就可以调出菜单：

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

根据提示按 `O` 进入设置界面，如下：

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
   | E -    Bps/Par/Bits       : 115200 8N1                                |
   | F - Hardware Flow Control : No                                        |
   | G - Software Flow Control : No                                        |
   |                                                                       |
   |    Change which setting?                                              |
   +-----------------------------------------------------------------------+
```

**注意：**`Hardware Flow Control` 和 `Software Flow Control` 都要设成 No，否则可能导致无法输入。

设置完成后回到上一菜单，选择 `Save setup as dfl` 即可保存为默认配置，以后将默认使用该配置。

## 看门狗

AIBOX-Orin Nano 的外部看门狗设备名称是`/dev/wdt_crl`，使用方法如下:

```shell
# 写入不同字段来开启看门狗并设置时间
# 数字 0，1，2，3 分别表示 0.64s，2.56s，10.24s，40.96s
# 字母 e 表示开启看门狗，字母 d 表示关闭看门狗

# 开启并定时 10.24 秒，每 10.24 秒之内要写入一次，也可随时写入不同数字更改时间
echo e >/dev/wdt_crl #开启
echo 2 >/dev/wdt_crl #设置超时时间为10秒
```

## RTC

### 简介

AIBOX-Orin Nano 有一路外部的 RTC，由外部的底板电容进行供电，掉电后短时间内保证RTC运行。在 kernel 中表示为 `rtc0`。

### 接口使用

Linux 提供了三种用户空间调用接口。路径为：

*    SYSFS接口：/sys/class/rtc/rtc0/
*    PROCFS接口： /proc/driver/rtc
*    IOCTL接口： /dev/rtc0

同步最新的系统时间到 RTC 内：
```
hwclock -w
```

#### SYSFS接口

可以直接使用 `cat` 和 `echo` 操作 `/sys/class/rtc/rtc0/` 下面的接口。

比如查看当前 RTC 的日期和时间：

```
# cat /sys/class/rtc/rtc0/date
2024-07-10
# cat /sys/class/rtc/rtc0/time
02:36:30

```

#### PROCFS 接口

打印 RTC 相关的信息：

```
# cat /proc/driver/rtc
rtc_time        : 02:37:00
rtc_date        : 2024-07-10
alrm_time       : 00:00:00
alrm_date       : 1970-01-01
alarm_IRQ       : no
alrm_pending    : no
update IRQ enabled      : no
periodic IRQ enabled    : no
periodic IRQ frequency  : 1
max user IRQ frequency  : 64
24hr            : yes
```

#### IOCTL接口

可以使用 `ioctl` 控制 `/dev/rtc0`。

详细使用说明请参考文档 `kernel-jammy-src/Documentation/admin-guide/rtc.rst` 。

# jtop 安装
```
sudo apt update
sudo apt install python3-pip
sudo pip3 install -U jetson-stats
```

## 重要!重要!重要
<font color=red>不要执行 `sudo apt-get upgrade` 和 `sudo apt-get dist-upgrade` 。</font>
<br>
<font color=red>不要执行 `sudo apt upgrade` 和 `sudo apt dist-upgrade` 。</font>


为了防止被自动 upgrade : 
* 修改 `/etc/apt/apt.conf.d/20auto-upgrades` (如果文件不存在，就创建它)

```
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Unattended-Upgrade "0";
```

* 关闭和停止 apt-daily timers 服务

```
sudo systemctl disable apt-daily.timer
sudo systemctl disable apt-daily-upgrade.timer
sudo systemctl stop apt-daily.timer
sudo systemctl stop apt-daily-upgrade.timer
```
# nvpmodel 

||7W|15W|25W|MAXN_SUPER|
|----|----|----|----|----|
|Jetson Orin Nano 8GB|Mode ID 0|Mode ID 1|N/A|N/A|
|Jetson Orin Nano 8GB SUPER|N/A|Mode ID 0|Mode ID 1|Mode ID 2|

||MAXN_SUPER|MAXN|10W|15W|20W|40W|
|----|----|----|----|----|----|----|
|Jetson Orin NX 8GB|N/A|Mode ID 0|Mode ID 1|Mode ID 2|Mode ID 3|N/A|
|Jetson Orin NX 8GB SUPER|Mode ID 0|N/A|Mode ID 1|Mode ID 2|Mode ID 3|Mode ID 4|

||MAXN_SUPER|MAXN|10W|15W|25W|40W|
|----|----|----|----|----|----|----|
|Jetson Orin NX 16GB|N/A|Mode ID 0|Mode ID 1|Mode ID 2|Mode ID 3|N/A|
|Jetson Orin NX 16GB SUPER|Mode ID 0|N/A|Mode ID 1|Mode ID 2|Mode ID 3|Mode ID 4|

## 命令
* 读取: `sudo nvpmodel -q`
* 设置: `sudo nvpmodel -m <x>`
    * `<x>` 是 `Mode ID` (比如, 0, 1, 2, 3 or 4)

## GUI
<center>

![](../../../aibox_img/nvpmodel_gui.png)
</center>

* 点击英伟达的图标
* 点击"Power mode", 选择所需的功率

# Browser
```
sudo apt update
sudo apt install chromium-browser
```

如果安装浏览器后，打开失败，进行如下操作:

```
snap download snapd --revision=24724
sudo snap ack snapd_24724.assert
sudo snap install snapd_24724.snap
```


# HDMI Audio
`Settings` --> `Sound` --> `Output Device` --> `HDMI/ DisplayPort-Build-in Audio`

