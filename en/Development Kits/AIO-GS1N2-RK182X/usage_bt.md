# Bluetooth

RK182X开发套件 supports wireless bluetooth, you can display the bluetooth device information through `hciconfig -a` command:

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

Note that if the following error message is printed:

```shell
Can't open HCI socket.: Address family not supported by protocol
```

Then there may be a problem with the dependencies of the loadable modules. At this time, reload the dependencies and soft restart:

```shell
sudo -i
depmod -a
reboot
```

Check whether it is currently the master device or the slave device:

```shell
linaro@bm1684:~$ hciconfig hci0 lm
hci0:	Type: Primary  Bus: USB
	BD Address: 20:57:9E:BA:7C:EC  ACL MTU: 1021:8  SCO MTU: 255:16
	Link mode: SLAVE ACCEPT # slave device
```

Start the PulseAudio service as a media device:

```shell
linaro@bm1684:~$ pulseaudio --start --log-target=syslog
```

The steps to connect a Bluetooth device are as follows:

(1) Start the Bluetooth controller tool:

```shell
linaro@bm1684:~$ bluetoothctl
[NEW] Controller 20:57:9E:BA:7C:EC bm1684 [default]
```

(2) Power on the Bluetooth controller:

```shell
[bluetooth]# power on
Changing power on succeeded
```

(3) Set the bluetooth agent as default:

```shell
[bluetooth]# agent on
Agent registered
[bluetooth]# default-agent
Default agent request successful
```

(4) Open to be discovered by other Bluetooth devices:

```shell
[bluetooth]# discoverable on
Changing discoverable on succeeded
[CHG] Controller 20:57:9E:BA:7C:EC Discoverable: yes
```

(5) At this point, the RK182X开发套件 Bluetooth device can be found on the smartphone, and the phone can be paired after clicking:

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

(6) Connect mobile device:

```shell
[bluetooth]# connect A4:90:CE:DF:64:4F
Attempting to connect to A4:90:CE:DF:64:4F
[CHG] Device A4:90:CE:DF:64:4F Connected: yes
Connection successful
[CHG] Device A4:90:CE:DF:64:4F ServicesResolved: yes
[iQOO Neo6 SE]# 
```

(7) Set the mobile device to trust:

```shell
[iQOO Neo6 SE]# trust A4:90:CE:DF:64:4F
[CHG] Device A4:90:CE:DF:64:4F Trusted: yes
Changing A4:90:CE:DF:64:4F trust succeeded
```

## Bluetooth Audio

Bluetooth audio is available using the bluez-alsa tool, a Bluetooth audio ALSA backend utility.

### Install the bluez-alsa tool

RK182X开发套件 does not have this tool installed by default, and users need to compile and install it by themselves. Here are the steps for version 1.3.0, as follows:

(1) Download the source package: [https://github.com/Arkq/bluez-alsa/releases/tag/v1.3.0](https://github.com/Arkq/bluez-alsa/releases/tag/v1.3.0)

(2) Installation dependencies:

```shell
sudo apt install -y libasound2-dev libbluetooth-dev libglib2.0-dev libsbc-dev libfdk-aac-dev pkgconf
```

(3) Extract:

```shell
tar xzvf bluez-alsa-1.3.0.tar.gz
cd bluez-alsa-1.3.0
```

(4) Compile and install:

```shell
autoreconf --install
mkdir build && cd build
../configure --enable-aac --enable-debug
make && make install
```

### Audio Test

After the installation is complete, you can connect a Bluetooth headset or speaker to play music. First, configure Bluetooth as the master device mode:

```shell
sudo hciconfig hci0 lm master
```

Enable the bluez-alsa service, note that the PulseAudio service and bluez-alsa are mutually exclusive:

```shell
# Remove the PulseAudio process
killall pulseaudio
# Open the bluez-alsa service for connecting to bluetooth 
bluealsa -p a2dp-source -p hsp-ag &
```

To connect a Bluetooth headset:

```shell
[bluetooth]# connect 0C:AE:BD:9B:BB:5C
Attempting to connect to 0C:AE:BD:9B:BB:5C
[CHG] Device 0C:AE:BD:9B:BB:5C Connected: yes
Connection successful
[CHG] Device 0C:AE:BD:9B:BB:5C ServicesResolved: yes
[EDIFIER LolliPods 2022版]# 
```

In addition, open a login and execute the following command to play the audio:

```shell
aplay -D bluealsa:HCI=hci0,DEV=0C:AE:BD:9B:BB:5C,PROFILE=a2dp example.wav
```

For more information about bluez-alsa, please refer to the source code repository: [https://github.com/Arkq/bluez-alsa/tree/v1.3.0](https://github.com/Arkq/bluez-alsa/tree/v1.3.0)。


## Send and receive files

Bluetooth file sending and receiving can use the OBEX protocol, which encapsulates information data with an object model and transmits applications with a session protocol specification.

The Obex service is needed in Linux. Firstly, connect the RK182X开发套件 to the Bluetooth device according to the previous steps, then start the Obex daemon, and set the receiving directory to `/home/linaro/`:

```shell
/usr/lib/bluetooth/obexd -r /home/linaro -a -d &
```

### Use obex push service

First configure Bluetooth as master device mode:

```shell
sudo hciconfig hci0 lm master
```

Find the channel for the Obex Push service on your mobile phone:

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

You can see that the channel is 12, and then you can push the file to the phone:

```shell
linaro@bm1684:~$ obexftp --nopath --noconn --uuid none --bluetooth A4:90:CE:DF:64:4F --channel 12 --put sn.txt
Suppressing FBS.
Connecting..\done
Sending "sn.txt".../done
Disconnecting..-done
```

At this time, you can see the pop-up window whether to receive the file on the mobile phone.

### Use obexctl interactive command line

The following demonstrates the steps for RK182X开发套件 to receive files:

(1) Start the obex service on the device side (slave device):

```shell
root@firefly:~# systemctl --user start obex
```

(2) Enter the interactive command line:

```shell
root@firefly:~# obexctl 
[NEW] Client /org/bluez/obex 
```

(3) Connect RK182X开发套件 (master device):

```shell
[obex]# connect 20:57:9E:BA:7C:EC
Attempting to connect to 20:57:9E:BA:7C:EC
...
[NEW] Session /org/bluez/obex/client/session2 [default]
[NEW] ObjectPush /org/bluez/obex/client/session2 
```

(4) Send files:

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

(5) Check the `test.txt` file in the `/home/linaro/` directory of RK182X开发套件:

```shell
linaro@bm1684:~$ ls -l test.txt
-rw------- 1 linaro linaro 0 Nov 25 15:49 test.txt
```