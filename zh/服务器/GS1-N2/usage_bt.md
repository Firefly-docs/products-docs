# 蓝牙

GS1-N2 支持无线蓝牙，可以通过 `hciconfig -a` 命令显示蓝牙设备信息：

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

（5）此时在智能手机就能发现 GS1-N2 蓝牙设备，手机点击后就能配对上去：

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

## 蓝牙音频

蓝牙音频可以使用 bluez-alsa 工具，这是一个蓝牙音频 ALSA 后端实用程序。

### 安装 bluez-alsa 工具

GS1-N2 默认没有安装该工具，用户需要自行编译安装，这里演示 1.3.0 版本的步骤，如下：

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

### 音频测试

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


## 文件收发

蓝牙文件收发可以使用 OBEX 协议，它以对象模型封装信息数据，以会话协议规范传输应用。

在 Linux 中需要用到 Obex 服务，首先 GS1-N2 根据前面的步骤连接好蓝牙设备，然后开启 Obex 守护进程，并设置接收的目录为 `/home/linaro/`：

```shell
/usr/lib/bluetooth/obexd -r /home/linaro -a -d &
```

### 使用 obex push 服务

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

### 使用 obexctl 交互命令行

以下演示 GS1-N2 收文件的步骤：

（1）设备端（从设备）开启 obex 服务：

```shell
root@firefly:~# systemctl --user start obex
```

（2）进入交互命令行：

```shell
root@firefly:~# obexctl 
[NEW] Client /org/bluez/obex 
```

（3）连接 GS1-N2 （主设备）：

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

（5）查看 GS1-N2 的 `/home/linaro/`目录有 `test.txt` 文件：

```shell
linaro@sophon:~$ ls -l test.txt
-rw------- 1 linaro linaro 0 Nov 25 15:49 test.txt
```