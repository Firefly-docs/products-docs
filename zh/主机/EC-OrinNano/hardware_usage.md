# 硬件功能使用

整机也可通过 HDMI 登录桌面：在界面登录的时候，Ubuntu 的会话以及密码均选择 `nvidia` 进行登录。

## 重要注意事项

<font color=red>不要执行 `sudo apt-get upgrade` 和 `sudo apt-get dist-upgrade`。</font>

<font color=red>不要执行 `sudo apt upgrade` 和 `sudo apt dist-upgrade`。</font>

为了防止被自动 upgrade：

* 修改 `/etc/apt/apt.conf.d/20auto-upgrades`（如果文件不存在，就创建它）：

```
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Unattended-Upgrade "0";
```

* 关闭和停止 apt-daily timers 服务：

```
sudo systemctl disable apt-daily.timer
sudo systemctl disable apt-daily-upgrade.timer
sudo systemctl stop apt-daily.timer
sudo systemctl stop apt-daily-upgrade.timer
```

## 显示接口

EC-Orin Nano 提供 1 路 HDMI 输出接口。

## 以太网

EC-Orin Nano 提供 1 个千兆以太网口。

## USB 接口

EC-Orin Nano 提供 2 个 USB 3.0 接口。

## SIM 卡

EC-Orin Nano 的 SIM 卡槽用于配合 4G 模块实现移动网络连接。**4G 模块为选配**，需要在机箱内部安装 4G 模块后，SIM 卡功能才能正常使用。SIM 卡插入方向如下图所示，插拔前请先断电。

<center>

<img alt="" src="../../../nvidia_img/EC-Orin-Nano/sim_insert_direction.png" width="400">
</center>

## CAN

### CAN 简介

CAN(Controller Area Network)总线，即控制器局域网总线，是一种有效支持分布式控制或实时控制的串行通信网络。CAN总线是一种在汽车上广泛采用的总线协议，被设计作为汽车环境中的微控制器通讯。如果想了解更多的内容可以参考[CAN应用报告](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf)。

### 硬件连接

EC-Orin Nano 的 CAN [接口位置如图所示](started.md)。由于只有一个 CAN，所以默认在内核中，第一个创建的设备为 `can0`。

### CAN 通信测试

使用 candump 和 cansend 工具进行收发报文测试即可:

```
#在收发端关闭can0设备
ip link set can0 down
#在收发端设置比特率为250Kbps
ip link set can0 type can bitrate 250000
#在收发端打开can0设备
ip link set can0 up
#在接收端执行candump,阻塞等待报文
candump can0
#在发送端执行cansend，发送报文
cansend can0 123#1122334455667788
```

### FAQS

#### 报文发送后很久才接收到，或者接收不到。

检查总线 CAN_H 和 CAN_L，杜邦线是否松动或者接反。

## 串口（RS232 / RS485）

* RS485 接口，设备名称为 `/dev/ttyTHS2`，支持半双工，默认波特率为 `9600`。
    * 发送数据前先执行： `echo 1 > /sys/class/leds/RS485_H_SEND_L_RECV/brightness`
    * 接收数据前先执行： `echo 0 > /sys/class/leds/RS485_H_SEND_L_RECV/brightness`
* RS232 接口，设备名称为 `/dev/ttyTHS1`，支持全双工，默认波特率为 `115200`。

### 调试方法

以 RS485 的调试步骤为例：

(1) 连接硬件

将 EC-Orin Nano 的 RS485B-A、RS485B-B 以及 GND 引脚分别和主机串口适配器（USB 转 485 转串口模块）的 A、B、GND 引脚相连。

(2) 打开主机的串口终端

在终端打开 kermit，并设置波特率：

```
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

* `/dev/ttyUSB0` 为 USB 转串口适配器的设备文件

(3) 发送数据

RS485 的设备文件为 `/dev/ttyTHS2`。在设备上运行下列命令：

```
sudo -s
stty -F /dev/ttyTHS2 9600 -echo
echo firefly RS485 test... > /dev/ttyTHS2
```

主机中的串口终端即可接收到字符串 "firefly RS485 test..."

(4) 接收数据

首先在设备上运行下列命令：

```
sudo -s
cat /dev/ttyTHS2
```

然后在主机的串口终端输入字符串 "Firefly RS485 test..."，设备端即可见到相同的字符串。

## 音频

EC-Orin Nano 拥有两路音频输出以及一路音频输入。

### 音频输出

用户可以通过耳机口以及 HDMI 口去做音频的输出。

#### 界面切换

在系统设置中切换耳机以及 HDMI 接口，选择单独一路进行输出：

<center>

<img alt="" src="../../../nvidia_img/EC-Orin-Nano/Sound.png" width="700">
</center>

#### 命令行模式

在终端下，执行命令：

```shell
# cat /proc/asound/cards
 0 [HDA            ]: tegra-hda - NVIDIA Jetson Orin Nano HDA
                       NVIDIA Jetson Orin Nano HDA at 0x3518000 irq 120
 1 [APE            ]: tegra-ape - NVIDIA Jetson Orin Nano APE
                       NVIDIA-NVIDIAJetsonOrinNanoDeveloperKit-NotSpecified-Jetson
```

* `HDA` 代表的是 HDMI 的声卡，设备号为 `3`
* `APE` 代表的是耳机的声卡，设备号为 `0`

* HDMI

    ```shell
    aplay -D hw:HDA,3 hdmi.wav # 其中 hdmi.wav 需要双声道格式文件
    ```

* 耳机

    ```shell
    amixer -c APE cset name="I2S2 Mux" ADMAIF1
    aplay -D hw:APE,0 test.wav
    ```

### 音频输入

同样是 `APE` 声卡，接入耳机开始录音，命令行执行：

```shell
amixer -c APE cset name="ADMAIF1 Mux" I2S2
arecord -D hw:APE,0 -c 2 -r 16000  -f S32_LE test.wav
```

## RTC

### 简介

EC-Orin Nano 有一路内部的 RTC，由外部的底板电池进行供电，在 kernel 中表示为 `rtc0`。

### 接口使用

Linux 提供了三种用户空间调用接口。路径为：

*    SYSFS接口：/sys/class/rtc/rtc0/
*    PROCFS接口： /proc/driver/rtc
*    IOCTL接口： /dev/rtc0

同步最新的系统时间到 RTC 内：

```
hwclock -w
```

### SYSFS接口

可以直接使用 `cat` 和 `echo` 操作 `/sys/class/rtc/rtc0/` 下面的接口。

比如查看当前 RTC 的日期和时间：

```
# cat /sys/class/rtc/rtc0/date
2024-07-10
# cat /sys/class/rtc/rtc0/time
02:36:30
```

### PROCFS 接口

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
periodic IRQ enabled      : no
periodic IRQ frequency  : 1
max user IRQ frequency  : 64
24hr            : yes
```

### IOCTL接口

可以使用 `ioctl` 控制 `/dev/rtc0`。

详细使用说明请参考文档 `kernel-jammy-src/Documentation/admin-guide/rtc.rst`。

## 看门狗

外部看门狗的设备名称是 `/dev/wdt_crl`，使用方法如下:

```shell
# 写入不同字段来开启看门狗并设置时间
# 数字 0，1，2，3 分别表示 0.64s，2.56s，10.24s，40.96s
# 字母 e 表示开启看门狗，字母 d 表示关闭看门狗

# 开启并定时 10.24 秒，每 10.24 秒之内要写入一次，也可随时写入不同数字更改时间
echo e >/dev/wdt_crl //开启
echo 2 >/dev/wdt_crl //设置超时时间为10秒
```