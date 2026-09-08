# 硬件功能使用

## 登录

AIBOX-PRO-KIT 登录方式有两种，一种是通过 Console 串口进行终端登录，一种是通过 HDMI 登录。

### Console 登录
Type-C 线接入 Console 口，登录账号为`root`，默认没有设置`root`密码。<br>
接口请参考[硬件接口介绍](interface_definition.md)。
使用以下串口参数：
* 波特率：115200
* 数据位：8
* 停止位：1
* 奇偶校验：无
* 流控：无

### HDMI 登录
在界面登录的时候，自动登录`firefly`用户，`firefly` 用户密码也为`firefly`。

## 看门狗
### 主模组 Core-3588JD4
#### 核心模组上的看门狗
看门狗的设备名称是`/dev/wdt_core`，使用方法如下:
```shell
# 写入不同字段来开启看门狗并设置时间
# 数字 0，1，2，3 分别表示 0.64s，2.56s，10.24s，40.96s
# 字母 e 表示开启看门狗，字母 d 表示关闭看门狗

# 开启并定时 10.24 秒，每 10.24 秒之内要写入一次，也可随时写入不同数字更改时间
echo e >/dev/wdt_core //开启
echo 2 >/dev/wdt_core //设置超时时间为10秒
```
#### 主板上的看门狗
看门狗的设备名称是`/dev/wdt_base`，使用方法如下:
```shell
# 写入不同字段来开启看门狗并设置时间
# 数字 0，1，2，3 分别表示 0.64s，2.56s，10.24s，40.96s
# 字母 e 表示开启看门狗，字母 d 表示关闭看门狗

# 开启并定时 10.24 秒，每 10.24 秒之内要写入一次，也可随时写入不同数字更改时间
echo e >/dev/wdt_base //开启
echo 2 >/dev/wdt_base //设置超时时间为10秒
```

## RTC

### 简介

AIBOX-PRO-KIT 有一路外部的 RTC，由外部的底板电容进行供电，掉电后短时间内保证RTC运行。在 kernel 中表示为 `rtc0`。

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

## 蓝牙

AIBOX-PRO-KIT 支持无线蓝牙，可以通过 `hciconfig -a` 命令显示蓝牙设备信息：

```shell
linaro@sophon:~$ hciconfig -a
hci0:   Type: Primary  Bus: USB
        BD Address: F0:35:75:A7:E2:88  ACL MTU: 1021:6  SCO MTU: 255:12
        UP RUNNING PSCAN ISCAN
        RX bytes:1050 acl:0 sco:0 events:61 errors:0
        TX bytes:2735 acl:0 sco:0 commands:61 errors:0
        Features: 0xff 0xff 0xff 0xfa 0xdb 0xbf 0x7b 0x87
        Packet type: DM1 DM3 DM5 DH1 DH3 DH5 HV1 HV2 HV3
        Link policy: RSWITCH HOLD SNIFF PARK
        Link mode: SLAVE ACCEPT
        Name: 'sophon'
        Class: 0x0c0000
        Service Classes: Rendering, Capturing
        Device Class: Miscellaneous,
        HCI Version: 5.1 (0xa)  Revision: 0x1ac7
        LMP Version: 5.1 (0xa)  Subversion: 0x2999
        Manufacturer: Realtek Semiconductor Corporation (93)

```

注意的是，如果打印如下错误信息：

```shell
Can't open HCI socket.: Address family not supported by protocol
```

则可能是可加载模块的依赖性出现问题，此时重新加载依赖性，并软重启即可：

```shell
sudo -i
depmod -a
reboot
```

查看当前是主设备还是从设备：

```shell
linaro@sophon:~$ hciconfig hci0 lm
hci0:   Type: Primary  Bus: USB
        BD Address: F0:35:75:A7:E2:88  ACL MTU: 1021:6  SCO MTU: 255:12
        Link mode: SLAVE ACCEPT # 从设备
```

启动 PulseAudio 服务作为用于媒体设备：

```shell
linaro@sophon:~$ pulseaudio --start --log-target=syslog
```

连接蓝牙设备步骤如下:

（1）启动蓝牙管理工具：

```shell
linaro@sophon:~$ bluetoothctl
Agent registered
[CHG] Controller F0:35:75:A7:E2:88 Pairable: yes
```

（2）上电蓝牙控制器：

```shell
[bluetooth]# power on
Changing power on succeeded
```

（3）将蓝牙代理设置为默认值：

```shell
[bluetooth]# agent on
Agent is already registered
[bluetooth]# default-agent
Default agent request successful
```

（4）打开可被其他蓝牙设备发现：

```shell
[bluetooth]# discoverable on
Changing discoverable on succeeded
[CHG] Controller F0:35:75:A7:E2:88 Discoverable: yesyes
```

（5）此时在智能手机就能发现 AIBOX-PRO-KIT 蓝牙设备，手机点击后就能配对上去：

```shell
[NEW] Device 34:1C:F0:40:59:55 小黎的 Redmi K30 Ultr
Request confirmation
[agent] Confirm passkey 062794 (yes/no): yes
[CHG] Device 34:1C:F0:40:59:55 Modalias: bluetooth:v0046p1200d1436
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 00001105-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 0000110a-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 0000110c-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 00001112-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 00001115-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 00001116-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 0000111f-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 0000112f-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 00001132-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 00001200-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 00001800-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 00001801-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 ServicesResolved: yes
[CHG] Device 34:1C:F0:40:59:55 Paired: yes
Authorize service
[agent] Authorize service 0000110d-0000-1000-8000-00805f9b34fb (yes/no): yes
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 00001105-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 0000110a-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 0000110c-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 0000110d-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 00001112-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 00001115-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 00001116-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 0000111f-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 0000112f-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 00001132-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 00001200-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 00001800-0000-1000-8000-00805f9b34fb
[CHG] Device 34:1C:F0:40:59:55 UUIDs: 00001801-0000-1000-8000-00805f9b34fb
```

（6）连接手机设备：

```shell
[bluetooth]# connect 34:1C:F0:40:59:55
Attempting to connect to 34:1C:F0:40:59:55
[CHG] Device 34:1C:F0:40:59:55 Connected: yes
```

（7）把手机设备设置成信任：

```shell
[小黎的 Redmi K30 Ultr]# trust 34:1C:F0:40:59:55
[CHG] Device 34:1C:F0:40:59:55 Trusted: yes
Changing 34:1C:F0:40:59:55 trust succeeded
```

### 蓝牙音频

蓝牙音频可以使用 bluez-alsa 工具，这是一个蓝牙音频 ALSA 后端实用程序。

#### 安装 bluez-alsa 工具

AIBOX-PRO-KIT 默认没有安装该工具，用户需要自行编译安装，这里演示 1.3.0 版本的步骤，如下：

（1）安装依赖：

```shell
sudo apt-get install git automake build-essential libtool pkg-config python3-docutils
sudo apt-get install libasound2-dev libbluetooth-dev libdbus-1-dev libglib2.0-dev  libsndfile1-dev  libsbc-dev
sudo apt-get install libudev-dev libical-dev libreadline-dev
```  

```shell
# 安装 fdk-aac
wget https://downloads.sourceforge.net/opencore-amr/fdk-aac-2.0.1.tar.gz  
tar -zxvf fdk-aac-2.0.1.tar.gz
cd fdk-aac-2.0.1/
autoreconf -fiv
sudo ./configure --prefix=/usr --disable-shared
sudo make
sudo make install
```
（2）下载 `bluez-alsa` 并编译安装：

```shell
wget https://github.com/arkq/bluez-alsa/archive/refs/tags/v1.3.0.tar.gz
tar -xf v1.3.0.tar.gz
cd bluez-alsa-1.3.0
autoreconf --install
mkdir build && cd build
../configure --enable-aac --enable-debug
make && make install
```

#### 音频测试

安装完成后可以连接蓝牙耳机或者音箱进行音乐播放，首先将蓝牙配置成主设备模式：

```shell
sudo hciconfig hci0 lm master
```

开启 bluez-alsa 服务，注意 PulseAudio 服务与 bluez-alsa 是互斥的：

```shell
# 去掉 PulseAudio 进程
killall pulseaudio
# 开启 bluez-alsa 服务用于连接蓝牙耳机或音响
bluealsa -p a2dp-source -p hsp-ag &
```

连接蓝牙耳机：

```shell
[bluetooth]# connect 0C:AE:BD:9B:BB:5C
Attempting to connect to 0C:AE:BD:9B:BB:5C
[CHG] Device 0C:AE:BD:9B:BB:5C Connected: yes
Connection successful
[CHG] Device 0C:AE:BD:9B:BB:5C ServicesResolved: yes
[EDIFIER LolliPods 2022版]# 
```

另外开启一个登录，执行以下命令播放音频：

```shell
aplay -D bluealsa:HCI=hci0,DEV=0C:AE:BD:9B:BB:5C,PROFILE=a2dp example.wav
```

关于 bluez-alsa 更多使用可参考源代码仓库：[https://github.com/Arkq/bluez-alsa/tree/v1.3.0](https://github.com/Arkq/bluez-alsa/tree/v1.3.0)。


### 文件收发

蓝牙文件收发可以使用 OBEX 协议，它以对象模型封装信息数据，以会话协议规范传输应用。

在 Linux 中需要用到 Obex 服务，首先 AIBOX-PRO-KIT 根据前面的步骤连接好蓝牙设备，然后开启 Obex 守护进程，并设置接收的目录为 `/home/linaro/`：

```shell
/usr/lib/bluetooth/obexd -r /home/linaro -a -d &
```

#### 使用 obex push 服务

首先将蓝牙配置成主设备模式：

```shell
sudo hciconfig hci0 lm master
```

查找手机 Obex Push 服务的通道：

```shell
linaro@sophon:~$ sdptool search --bdaddr A4:90:CE:DF:64:4F OPUSH
Searching for OPUSH on A4:90:CE:DF:64:4F ...
Service Name: OBEX Object Push
Service RecHandle: 0x1000d
Service Class ID List:
  "OBEX Object Push" (0x1105)
Protocol Descriptor List:
  "L2CAP" (0x0100)
  "RFCOMM" (0x0003)
    Channel: 12
  "OBEX" (0x0008)
Profile Descriptor List:
  "OBEX Object Push" (0x1105)
    Version: 0x0102

Searching for OPUSH on A4:90:CE:DF:64:4F ...
Service Search failed: Invalid argument
```

可以看到通道为 12，紧接着就能把文件 push 到手机上：

```shell
linaro@sophon:~$ obexftp --nopath --noconn --uuid none --bluetooth A4:90:CE:DF:64:4F --channel 12 --put sn.txt
Suppressing FBS.
Connecting..\done
Sending "sn.txt".../done
Disconnecting..-done
```

此时手机端就可看到是否接收文件的弹窗。

#### 使用 obexctl 交互命令行

以下演示 AIBOX-PRO-KIT 收文件的步骤：

（1）设备端（从设备）开启 obex 服务：

```shell
root@firefly:~# systemctl --user start obex
```

（2）进入交互命令行：

```shell
root@firefly:~# obexctl 
[NEW] Client /org/bluez/obex 
```

（3）连接 AIBOX-PRO-KIT （主设备）：

```shell
[obex]# connect 20:57:9E:BA:7C:EC
Attempting to connect to 20:57:9E:BA:7C:EC
...
[NEW] Session /org/bluez/obex/client/session2 [default]
[NEW] ObjectPush /org/bluez/obex/client/session2 
```

（4）发送文件：

```shell
[20:57:9E:BA:7C:EC]# send /root/test.txt
Attempting to send /root/test.txt to /org/bluez/obex/client/session1
[NEW] Transfer /org/bluez/obex/client/session1/transfer1
Transfer /org/bluez/obex/client/session1/transfer1
        Status: queued
        Name: test.txt
        Size: 0
        Filename: /root/test.txt
        Session: /org/bluez/obex/client/session1
[CHG] Transfer /org/bluez/obex/client/session1/transfer1 Status: complete
```

（5）查看 AIBOX-PRO-KIT 的 `/home/linaro/`目录有 `test.txt` 文件：

```shell
linaro@sophon:~$ ls -l test.txt
-rw------- 1 linaro linaro 0 Nov 25 15:49 test.txt
```

## WIFI

AIBOX-PRO-KIT 支持无线 WIFI，在系统中网卡名默认为 `wlanx` 或者是 `wlPxp1s0`（根据不同的 wifi 模块，有所不同），以 `wlan0` 为例：


```shell
linaro@sophon:~# ifconfig wlan0
wlan0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 20:57:9e:ba:02:fc  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

#### WIFI 连接

（1）使能 WIFI：

```shell
nmcli radio wifi on
```

（2）查看 WIFI 是否使能成功：

```shell
# 打印出 enabled 表示成功
nmcli radio wifi
```

（3）查看 WIFI 接入点：

```shell
nmcli dev wifi
```

（4）连接到 WIFI 接入点：

```shell
sudo nmcli device wifi connect zouxftest1 password 12345678 name test
```
其中 `zouxftest1` 为 WIFI 接入点的名称，`12345678` 则是密码。

连接成功日志如下：

```shell
Device 'wlan0' successfully activated with 'fd6c634b-f517-4ae9-a8e9-292a9c19d25c'.
```

请注意，如果要禁用 WIFI 状态：

```shell
nmcli radio wifi off
```

#### WIFI 热点

使用 `nmcli` 命令可以创建无线 AP 热点：

```shell
sudo nmcli device wifi hotspot ifname wlan0 con-name my-hostapt ssid zouxftest7 band bg password 12345678 channel 5
```

说明如下：
- `con-name`：连接名称，这里定义为 `my-hostapt`
- `ssid`：创建的 AP 热点的名称，这里定义为 `zouxftest7`
- `band`：WIFI 的协议标准，这里选择 `bg`
- `password`：创建的 AP 热点的密码，这里定义为 `12345678`
- `channel`：创建的 AP 热点的通过，这里定义为 `5`

在创建了无线 AP 热点以后，如果要打开/关闭 WIFI 热点：

```shell
sudo nmcli connection up[down] my-hostapt
```

## RS485
AIBOX-PRO-KIT 有一个 RS485 接口，如果CPU是3588，则设备名称为 `/dev/ttyS6`， 如果CPU是3576，则为`/dev/ttyS3`，支持半双工，默认波特率为 `9600`。该接口为凤凰端子座，因此需要对应的端子接口接入，接入后，即可按照常规串口方法进行调试。

## 蜂窝网络

AIBOX-PRO-KIT 支持 4G LTE, 在系统设置处，有多种网络形式，可以在此打开数据流量开关：

<center>

![](../../../aibox_img/AIBOX-PRO-KIT/4G.png)
</center>

在命令行生成网卡：

```shell
$ ifconfig wwan0
wwan0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 10.176.252.100  netmask 255.255.255.248  destination 10.176.252.100
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 1000  (UNSPEC)
        RX packets 4  bytes 405 (405.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8  bytes 439 (439.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

## CAN 使用
### CAN 简介
CAN(Controller Area Network)总线，即控制器局域网总线，是一种有效支持分布式控制或实时控制的串行通信网络。CAN总线是一种在汽车上广泛采用的总线协议，被设计作为汽车环境中的微控制器通讯。
如果想了解更多的内容可以参考[CAN应用报告](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf)

### 硬件连接
AIBOX-PRO-KIT 开发板的 CAN [接口位置如图所示](interface_definition.md)



由于只有一个 CAN，所以默认在内核中，第一个创建的设备为 `can0`。


### CAN 通信测试    
使用 candump 和 cansend 工具进行收发报文测试即可，将工具push到/system/bin/目录下执行。工具包含在SDK中，也可以从 [GitHub](https://github.com/linux-can/can-utils) 下载。

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

#### 更多指令
```
1、 ip link set canX down 		//关闭can设备；
2、 ip link set canX up   		//开启can设备；
3、 ip -details link show canX 		//显示can设备详细信息；
4、 candump canX  			//接收can总线发来数据；
5、 ifconfig canX down 			//关闭can设备，以便配置;
6、 ip link set canX up type can bitrate 250000 //设置can波特率
7、 conconfig canX bitrate + 波特率；
8、 canconfig canX start 		//启动can设备；
9、 canconfig canX ctrlmode loopback on //回环测试；
10、canconfig canX restart 		// 重启can设备；
11、canconfig canX stop 		//停止can设备；
12、canecho canX 			//查看can设备总线状态；
13、cansend canX --identifier=ID+数据 	//发送数据；
14、candump canX --filter=ID：mask	//使用滤波器接收ID匹配的数据
```

### FAQS
总结调试过程中遇到的几个问题及解决方法：

#### 报文发送后很久才接收到，或者接收不到。

检查总线 CAN_H 和 CAN_L， 杜邦线是否松动或者接反。


