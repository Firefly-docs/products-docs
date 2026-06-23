# Bluetooth

The AIBOX-186 supports wireless Bluetooth and can display Bluetooth device information via the `hciconfig -a` command:

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

Note that if the following error message is printed:

```shell
Can't open HCI socket.: Address family not supported by protocol
```

Then there may be a problem with the dependencies of the loadable module, at this point reload the dependencies and soft reboot:

```shell
sudo -i
depmod -a
reboot
```

Check whether the device is currently a master or a slave:

```shell
linaro@sophon:~$ hciconfig hci0 lm
hci0:   Type: Primary  Bus: USB
        BD Address: F0:35:75:A7:E2:88  ACL MTU: 1021:6  SCO MTU: 255:12
        Link mode: SLAVE ACCEPT # 从设备
```

Start the PulseAudio service as a media device:

```shell
linaro@sophon:~$ pulseaudio --start --log-target=syslog
```

To connect a Bluetooth device, proceed as follows.

（1）Launch the Bluetooth management tool:

```shell
linaro@sophon:~$ bluetoothctl
Agent registered
[CHG] Controller F0:35:75:A7:E2:88 Pairable: yes
```

（2）Power up the Bluetooth controller:

```shell
[bluetooth]# power on
Changing power on succeeded
```

（3）Sets the Bluetooth proxy to its default value:

```shell
[bluetooth]# agent on
Agent is already registered
[bluetooth]# default-agent
Default agent request successful
```

（4）Turn on to be discoverable by other Bluetooth devices:

```shell
[bluetooth]# discoverable on
Changing discoverable on succeeded
[CHG] Controller F0:35:75:A7:E2:88 Discoverable: yesyes
```

（5）At this point the AIBOX-186 Bluetooth device can be found on the smartphone, and the phone can be paired up by clicking on it:

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

（6）Connect the mobile device:

```shell
[bluetooth]# connect 34:1C:F0:40:59:55
Attempting to connect to 34:1C:F0:40:59:55
[CHG] Device 34:1C:F0:40:59:55 Connected: yes
```

（7）Set your mobile device to trust:

```shell
[小黎的 Redmi K30 Ultr]# trust 34:1C:F0:40:59:55
[CHG] Device 34:1C:F0:40:59:55 Trusted: yes
Changing 34:1C:F0:40:59:55 trust succeeded
```

## Bluetooth audio

Bluetooth audio can be used with the bluez-alsa utility, which is a Bluetooth audio ALSA backend utility.

### Installing the bluez-alsa tool

AIBOX-186 does not have this tool installed by default, users need to compile and install it by themselves, here is a demo of the steps for version 1.3.0, as follows:

（1）Install the dependency:

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
（2）Download `bluez-alsa` and compile and install it:

```shell
wget https://github.com/arkq/bluez-alsa/archive/refs/tags/v1.3.0.tar.gz
tar -xf v1.3.0.tar.gz
cd bluez-alsa-1.3.0
autoreconf --install
mkdir build && cd build
../configure --enable-aac --enable-debug
make && make install
```

### Audio Testing

After the installation is complete you can connect Bluetooth headphones or speakers for music playback, first configure Bluetooth to master device mode:

```shell
sudo hciconfig hci0 lm master
```

Enable the bluez-alsa service, note that the PulseAudio service is mutually exclusive with bluez-alsa:

```shell
# 去掉 PulseAudio 进程
killall pulseaudio
# 开启 bluez-alsa 服务用于连接蓝牙耳机或音响
bluealsa -p a2dp-source -p hsp-ag &
```

Connect a Bluetooth headset:

```shell
[bluetooth]# connect 0C:AE:BD:9B:BB:5C
Attempting to connect to 0C:AE:BD:9B:BB:5C
[CHG] Device 0C:AE:BD:9B:BB:5C Connected: yes
Connection successful
[CHG] Device 0C:AE:BD:9B:BB:5C ServicesResolved: yes
[EDIFIER LolliPods 2022版]# 
```

Open another login and execute the following command to play the audio:

```shell
aplay -D bluealsa:HCI=hci0,DEV=0C:AE:BD:9B:BB:5C,PROFILE=a2dp example.wav
```

For more on the use of bluez-alsa see the source repository: [https://github.com/Arkq/bluez-alsa/tree/v1.3.0](https://github.com/Arkq/bluez-alsa/tree/v1.3.0)。


## Sending and receiving of documents

Bluetooth files can be sent and received using the OBEX protocol, which encapsulates the information data in an object model and specifies the transfer application in a session protocol.

The Obex service is required in Linux. First, AIBOX-186 connects the Bluetooth device according to the previous steps, then starts the Obex daemon and sets the receiving directory to `/home/linaro/`:

``shell
/usr/lib/bluetooth/obexd -r /home/linaro -a -d &
``

### Using the obex push service

Start by configuring Bluetooth to master device mode:

```shell
sudo hciconfig hci0 lm master
```

Find the channel for the phone's Obex Push service:

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

You can see that the channel is 12, and right after that you can push the file to your phone:

```shell
linaro@sophon:~$ obexftp --nopath --noconn --uuid none --bluetooth A4:90:CE:DF:64:4F --channel 12 --put sn.txt
Suppressing FBS.
Connecting..\done
Sending "sn.txt".../done
Disconnecting..-done
```

At this point, the mobile phone can see the pop-up window of whether to receive the file or not.

### Using the obexctl interactive command line

The following demonstrates the steps for the AIBOX-186 to receive documents:

（1）The obex service is enabled on the device side (slave device):

```shell
root@firefly:~# systemctl --user start obex
```

（2）Go to the interactive command line:

```shell
root@firefly:~# obexctl 
[NEW] Client /org/bluez/obex 
```

（3）Connect the AIBOX-186 (master):

```shell
[obex]# connect 20:57:9E:BA:7C:EC
Attempting to connect to 20:57:9E:BA:7C:EC
...
[NEW] Session /org/bluez/obex/client/session2 [default]
[NEW] ObjectPush /org/bluez/obex/client/session2 
```

（4）Send the file:

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

（5）Checking the `/home/linaro/` directory of AIBOX-186 has the file `test.txt`:

```shell
linaro@sophon:~$ ls -l test.txt
-rw------- 1 linaro linaro 0 Nov 25 15:49 test.txt
```