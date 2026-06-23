# 蓝牙

AIBOX-1684X 支持无线蓝牙，可以通过 `hciconfig -a` 命令显示蓝牙设备信息：

```shell
linaro@bm1684:~$ hciconfig -a
hci0:	Type: Primary  Bus: USB
	BD Address: 20:57:9E:BA:7C:EC  ACL MTU: 1021:8  SCO MTU: 255:16
	UP RUNNING 
	RX bytes:650 acl:0 sco:0 events:41 errors:0
	TX bytes:2170 acl:0 sco:0 commands:41 errors:0
	Features: 0xff 0xff 0xff 0xfa 0xdb 0xbd 0x7b 0x87
	Packet type: DM1 DM3 DM5 DH1 DH3 DH5 HV1 HV2 HV3 
	Link policy: RSWITCH HOLD SNIFF PARK 
	Link mode: SLAVE ACCEPT 
	Name: 'bm1684'
	Class: 0x000000
	Service Classes: Unspecified
	Device Class: Miscellaneous, 
	HCI Version: 4.2 (0x8)  Revision: 0xaba8
	LMP Version: 4.2 (0x8)  Subversion: 0xa0cd
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
linaro@bm1684:~$ hciconfig hci0 lm
hci0:	Type: Primary  Bus: USB
	BD Address: 20:57:9E:BA:7C:EC  ACL MTU: 1021:8  SCO MTU: 255:16
	Link mode: SLAVE ACCEPT # 从设备
```

启动 PulseAudio 服务作为用于媒体设备：

```shell
linaro@bm1684:~$ pulseaudio --start --log-target=syslog
```

连接蓝牙设备步骤如下:

（1）启动蓝牙管理工具：

```shell
linaro@bm1684:~$ bluetoothctl
[NEW] Controller 20:57:9E:BA:7C:EC bm1684 [default]
```

（2）上电蓝牙控制器：

```shell
[bluetooth]# power on
Changing power on succeeded
```

（3）将蓝牙代理设置为默认值：

```shell
[bluetooth]# agent on
Agent registered
[bluetooth]# default-agent
Default agent request successful
```

（4）打开可被其他蓝牙设备发现：

```shell
[bluetooth]# discoverable on
Changing discoverable on succeeded
[CHG] Controller 20:57:9E:BA:7C:EC Discoverable: yes
```

（5）此时在智能手机就能发现 AIBOX-1684X 蓝牙设备，手机点击后就能配对上去：

```shell
[NEW] Device A4:90:CE:DF:64:4F iQOO Neo6 SE
[CHG] Device A4:90:CE:DF:64:4F Modalias: bluetooth:v001Dp1200d1436
[CHG] Device A4:90:CE:DF:64:4F UUIDs: 00001105-0000-1000-8000-00805f9b34fb
[CHG] Device A4:90:CE:DF:64:4F UUIDs: 0000110a-0000-1000-8000-00805f9b34fb
[CHG] Device A4:90:CE:DF:64:4F UUIDs: 0000110c-0000-1000-8000-00805f9b34fb
[CHG] Device A4:90:CE:DF:64:4F UUIDs: 0000110e-0000-1000-8000-00805f9b34fb
[CHG] Device A4:90:CE:DF:64:4F UUIDs: 00001112-0000-1000-8000-00805f9b34fb
[CHG] Device A4:90:CE:DF:64:4F UUIDs: 00001115-0000-1000-8000-00805f9b34fb
[CHG] Device A4:90:CE:DF:64:4F UUIDs: 00001116-0000-1000-8000-00805f9b34fb
[CHG] Device A4:90:CE:DF:64:4F UUIDs: 0000111f-0000-1000-8000-00805f9b34fb
[CHG] Device A4:90:CE:DF:64:4F UUIDs: 0000112d-0000-1000-8000-00805f9b34fb
[CHG] Device A4:90:CE:DF:64:4F UUIDs: 0000112f-0000-1000-8000-00805f9b34fb
[CHG] Device A4:90:CE:DF:64:4F UUIDs: 00001132-0000-1000-8000-00805f9b34fb
[CHG] Device A4:90:CE:DF:64:4F UUIDs: 00001200-0000-1000-8000-00805f9b34fb
[CHG] Device A4:90:CE:DF:64:4F UUIDs: 00001800-0000-1000-8000-00805f9b34fb
[CHG] Device A4:90:CE:DF:64:4F UUIDs: 00001801-0000-1000-8000-00805f9b34fb
[CHG] Device A4:90:CE:DF:64:4F UUIDs: 2c042b0a-7f57-4c0a-afcf-1762af70257c
[CHG] Device A4:90:CE:DF:64:4F UUIDs: 8fa9c715-bd1f-596c-a1b0-13162b15c892
[CHG] Device A4:90:CE:DF:64:4F ServicesResolved: yes
[CHG] Device A4:90:CE:DF:64:4F Paired: yes
[CHG] Device A4:90:CE:DF:64:4F ServicesResolved: no
[CHG] Device A4:90:CE:DF:64:4F Connected: no
[CHG] Controller 20:57:9E:BA:7C:EC Discoverable: no
```

（6）连接手机设备：

```shell
[bluetooth]# connect A4:90:CE:DF:64:4F
Attempting to connect to A4:90:CE:DF:64:4F
[CHG] Device A4:90:CE:DF:64:4F Connected: yes
Connection successful
[CHG] Device A4:90:CE:DF:64:4F ServicesResolved: yes
[iQOO Neo6 SE]# 
```

（7）把手机设备设置成信任：

```shell
[iQOO Neo6 SE]# trust A4:90:CE:DF:64:4F
[CHG] Device A4:90:CE:DF:64:4F Trusted: yes
Changing A4:90:CE:DF:64:4F trust succeeded
```

## 蓝牙音频

### 更新源

往 `/etc/apt/sources.list` 添加新的源：

```
deb-src https://mirrors.huaweicloud.com/ubuntu-ports/ bionic main restricted universe multiverse
deb-src https://mirrors.huaweicloud.com/ubuntu-ports/ bionic-updates main restricted universe multiverse
deb-src https://mirrors.huaweicloud.com/ubuntu-ports/ bionic-backports main restricted universe multiverse
deb-src https://mirrors.huaweicloud.com/ubuntu-ports/ bionic-security main restricted universe multiverse
```
更新源：
```
apt-get update
```

### 安装 sbc 1.5-3
```shell
# clone and checkout
git clone https://salsa.debian.org/bluetooth-team/sbc.git
cd sbc
git checkout debian/1.5-3

# downgrade debhelper-compat
sed -i -e 's/debhelper-compat (= 13)/debhelper-compat (= 12)/g' debian/control

# install build dependency
sudo apt-get build-dep sbc

# build deb
dpkg-buildpackage -b -us -uc

# install deb
cd ..
sudo apt-get install ./*sbc*_1.5-3_*.deb
```
### 安装 fdk-aac 2.0.2

```shell
# clone and checkout
git clone https://salsa.debian.org/multimedia-team/fdk-aac
cd fdk-aac
git checkout debian/2.0.2-1

# downgrade debhelper-compat
sed -i -e 's/debhelper-compat (= 13)/debhelper-compat (= 12)/g' debian/control

# install build dependency
sudo apt-get build-dep libfdk-aac-dev

# build deb
dpkg-buildpackage -b -us -uc

# install deb
cd ..
sudo apt-get install ./*libfdk-aac-dev*.deb
```
### 安装最新版本的 bluez-alsa

```shell
# clone and checkout
# 笔者测试的时候，该仓库的 HEAD 是：f0f9427da7d747cfe48b5c83b3076841f48d8703
git clone https://github.com/arkq/bluez-alsa
cd bluez-alsa

# install build dependency
sudo apt-get install automake build-essential libtool pkg-config python3-docutils
sudo apt-get install libasound2-dev libbluetooth-dev libdbus-1-dev libglib2.0-dev libsbc-dev libspandsp-dev

# build and install
autoreconf --install --force
mkdir build
cd build
../configure --enable-aac --enable-msbc --enable-systemd
make -j4
sudo make install

# update systemd service
sudo sed -i -e 's,^ExecStart=.*$,ExecStart=/usr/bin/bluealsad -S -p a2dp-source -p a2dp-sink -p hfp-ag -p hsp-ag,g' /lib/systemd/system/bluealsa.service
# disable pulseaudio, MUST BE RUN AS USER 'linaro'
systemctl --user disable --now pulseaudio pulseaudio.socket
sudo systemctl daemon-reload
sudo systemctl restart bluealsa

```
### 音频测试
* 打开第一个终端: 使用 bluetoothctl 连接蓝牙耳机
```shell
bluetoothctl
Agent registered
[bluetooth]# scan on
# ^- turn on scan, lots of result follow
...
[bluetooth]# scan off
# find the handset mac shown above
# and do the pairing (replace the MAC with actual device MAC)
[bluetooth]# pair MAC
[bluetooth]# trust MAC
[bluetooth]# connect MAC

# show the device info, for example:
[DEV]# info
info
Device 38:F5:FA:A2:70:A0 (public)
        Name: W25
        Alias: W25
        Class: 0x00240404
        Icon: audio-card
        Paired: yes
        Trusted: yes
        Blocked: no
        Connected: yes
        LegacyPairing: no
        UUID: Audio Sink                (0000110b-0000-1000-8000-00805f9b34fb)
        UUID: A/V Remote Control Target (0000110c-0000-1000-8000-00805f9b34fb)
        UUID: A/V Remote Control        (0000110e-0000-1000-8000-00805f9b34fb)
        UUID: Handsfree                 (0000111e-0000-1000-8000-00805f9b34fb)
        UUID: PnP Information           (00001200-0000-1000-8000-00805f9b34fb)
        Modalias: bluetooth:v05D6p000Ad0240
        ManufacturerData Key: 0x7262
        ManufacturerData Value:
  32 32 78 78 11 22 33 44 55 66 aa bb 00 00        22xx."3DUf....
```

* 打开第二个终端: 播放和录音  

```shell
# List the available BT audio PCMs:
$ sudo bluealsa-aplay -L
bluealsa:DEV=38:F5:FA:A2:70:A0,PROFILE=sco,SRV=org.bluealsa
    W25, trusted audio-card, capture
    SCO (mSBC): S16_LE 1 channel 16000 Hz
bluealsa:DEV=38:F5:FA:A2:70:A0,PROFILE=sco,SRV=org.bluealsa
    W25, trusted audio-card, playback
    SCO (mSBC): S16_LE 1 channel 16000 Hz
bluealsa:DEV=38:F5:FA:A2:70:A0,PROFILE=a2dp,SRV=org.bluealsa
    W25, trusted audio-card, playback
    A2DP (SBC): S16_LE 2 channels 48000 Hz
```
播放:
```shell
sudo aplay -D bluealsa:DEV=38:F5:FA:A2:70:A0,PROFILE=a2dp,SRV=org.bluealsa /usr/share/sounds/alsa/Front_Center.wav
```
录制:
```shell
# -d 6: record 6 seconds
sudo arecord -f S16_LE -r 16000 -D bluealsa:DEV=38:F5:FA:A2:70:A0,PROFILE=sco,SRV=org.bluealsa -d 6 /tmp/a.wav
```

音量控制:

```shell
$ sudo bluealsactl list-pcms
/org/bluealsa/hci0/dev_38_F5_FA_A2_70_A0/a2dpsrc/sink
/org/bluealsa/hci0/dev_38_F5_FA_A2_70_A0/hfpag/sink
/org/bluealsa/hci0/dev_38_F5_FA_A2_70_A0/hfpag/source

# Get Volume
$ sudo bluealsactl volume /org/bluealsa/hci0/dev_38_F5_FA_A2_70_A0/a2dpsrc/sink
Volume: 127 127

# Set Volume
$ sudo bluealsactl volume /org/bluealsa/hci0/dev_38_F5_FA_A2_70_A0/a2dpsrc/sink 100
```

## 文件收发

蓝牙文件收发可以使用 OBEX 协议，它以对象模型封装信息数据，以会话协议规范传输应用。

在 Linux 中需要用到 Obex 服务，首先 AIBOX-1684X 根据前面的步骤连接好蓝牙设备，然后开启 Obex 守护进程，并设置接收的目录为 `/home/linaro/`：

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
linaro@bm1684:~$ sdptool search --bdaddr A4:90:CE:DF:64:4F OPUSH
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
linaro@bm1684:~$ obexftp --nopath --noconn --uuid none --bluetooth A4:90:CE:DF:64:4F --channel 12 --put sn.txt
Suppressing FBS.
Connecting..\done
Sending "sn.txt".../done
Disconnecting..-done
```

此时手机端就可看到是否接收文件的弹窗。

### 使用 obexctl 交互命令行

以下演示 AIBOX-1684X 收文件的步骤：

（1）设备端（从设备）开启 obex 服务：

```shell
root@firefly:~# systemctl --user start obex
```

（2）进入交互命令行：

```shell
root@firefly:~# obexctl 
[NEW] Client /org/bluez/obex 
```

（3）连接 AIBOX-1684X （主设备）：

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

（5）查看 AIBOX-1684X 的 `/home/linaro/`目录有 `test.txt` 文件：

```shell
linaro@bm1684:~$ ls -l test.txt
-rw------- 1 linaro linaro 0 Nov 25 15:49 test.txt
```