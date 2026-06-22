## [4G Module](https://store.t-firefly.com/goods.php?id=49)
Check if the EC20 4G module kit has been loaded.
```
#The following 4 USB interfaces will appear after successful loading
ls /dev/ttyUSB*
/dev/ttyUSB0  /dev/ttyUSB1  /dev/ttyUSB2  /dev/ttyUSB3
```
If the load is not successful, check that the kernel has the following configuration turned on.
```
Symbol: USB_SERIAL_OPTION [=y]
Type  : tristate
Prompt: USB driver for GSM and CDMA modems
Location:
	-> Device Drivers
		-> USB support (USB_SUPPORT [=y]) 
			-> USB Serial Converter support (USB_SERIAL [=y]) 
Defined at drivers/usb/serial/Kconfig:558
Depends on: USB_SUPPORT [=y] && USB [=y] && USB_SERIAL [=y]
Selects: USB_SERIAL_WWAN [=y]
```

If the 4G module cannot be connected to the network, please check whether the following configuration is turned on in the kernel.
```
CONFIG_USB_USBNET
CONFIG_USB_NET_QMI_WWAN
```
Check whether the EC20 4G network card exists.
```
ifconfig wwan0
```
How to use EC20 4G module to access the Internet with a SIM card
```
# Download the ec20-armhf.tar.gz compressed package from the data download interface and copy it to the development board to decompress
# Need to replace the folder /ec20-armhf/usr/share/udhcpc/default.script with the system's /usr/share/udhcpc/default.script
# The results are as follows:
[root@RV1126_RV1109:/userdata/ec20-armhf]# quectel-CM &
[root@RV1126_RV1109:/userdata/ec20-armhf]# [04-20_14:27:04:405] Quectel_Linux_ConnectManager_SR01A01V21
[04-20_14:27:04:406] quectel-CM profile[1] = (null)/(null)/(null)/0, pincode = (null)
[04-20_14:27:04:407] Find qmichannel = /dev/cdc-wdm0
[04-20_14:27:04:407] Find usbnet_adapter = wwan0
[04-20_14:27:04:422] cdc_wdm_fd = 7
[04-20_14:27:05:422] QmiThreadSendQMITimeout pthread_cond_timeout_np=110, errno: 2 (No such file or directory)
[04-20_14:27:06:520] Get clientWDS = 19
[04-20_14:27:06:552] Get clientDMS = 1
[04-20_14:27:06:584] Get clientNAS = 3
[04-20_14:27:06:617] Get clientUIM = 1
[04-20_14:27:06:648] Get clientWDA = 1
[04-20_14:27:06:681] requestBaseBandVersion EC20CFDR02A07M4G  1  [Nov 10 2016 17:00:00]
[04-20_14:27:06:743] requestGetSIMStatus SIMStatus: SIM_READY
[04-20_14:27:06:777] requestGetProfile[1] cmnet///0
[04-20_14:27:06:808] requestRegistrationState2 MCC: 460, MNC: 0, PS: Attached, DataCap: LTE
[04-20_14:27:06:839] requestQueryDataCall ConnectionStatus: DISCONNECTED
[04-20_14:27:06:904] requestRegistrationState2 MCC: 460, MNC: 0, PS: Attached, DataCap: LTE
[04-20_14:27:07:447] requestSetupDataCall WdsConnectionIPv4Handle: 0x87205120
[04-20_14:27:07:513] requestQueryDataCall ConnectionStatus: CONNECTED
udhcpc: started, v1.27.2
Failed to kill daemon: No such file or directory
udhcpc: sending discover
udhcpc: sending select for 10.154.101.71
udhcpc: lease of 10.154.101.71 obtained, lease time 7200
Failed to kill daemon: No such file or directory
[04-20_14:27:07:729] deleting routers
[04-20_14:27:07:772] adding dns 221.179.38.7
[04-20_14:27:07:772] adding dns 120.196.165.7
[root@RV1126_RV1109:/userdata/ec20-armhf]#ifconfig wwan0
wwan0     Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  
          inet addr:10.154.101.71  P-t-P:10.154.101.71  Mask:255.255.255.240
          UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1500  Metric:1
          RX packets:2 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:612 (612.0 B)  TX bytes:656 (656.0 B)
```