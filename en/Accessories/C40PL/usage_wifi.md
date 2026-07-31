## WiFi
* ` RV1109/RV1126` uses `connman` to manage WiFi by default and `wpa_supplicant` is the key process for WiFi:

```
ps -ef
#You can see the following two processes
connmand			 #It uses dbus to communicate with wpa_supplicant
wpa_supplicant -u  #Turns on support for dbus communication
```
* Standard usage method: WiFi operation is carried out through the `RV1126/RV1109` web interface, refer 
to the relevant documents of `RV1109/RV1126` platform:
* Terminal simple test method is as follows:
```
killall ipc-daemon netserver
connmanctl
connmanctl> enable wifi
connmanctl> scan wifi #Allows multiple scans
connmanctl> scan wifi #Allows multiple scans
connmanctl> agent on
connmanctl> services
#List the scanned WiFi list
connmanctl>
	*AO yyz123
	NETGEAR75-5G	wifi_c0847daf6f42_4e45544745415237352d3547_managed_psk
	aaabbb			wifi_c0847daf6f42_616161626262_managed_psk
	HiWiFi-Free		wifi_c0847daf6f42_204869576946692d46726565_managed_none
	Fang-HiWiFi		wifi_c0847daf6f42_46616e672d486957694669_managed_psk
	yyz123				wifi_c0847daf6f42_79797a313233_managed_psk

connmanctl> connect wifi_c0847daf6f42_4e45544745415237352d3547_managed_psk
#If you want to connect to NETGEAR75-5G, then the connect parameter is wifixxx_psk
connmanctl>
Connected wifi_c0847daf6f42_4e45544745415237352d3547_managed_psk #This print will appear if the connection is successful
connmanctl> quit #Exit connection mode

ifconfig wlan0
#See the IP address of wlan0
```
* If you don't want to use connman and use the traditional `wpa_supplicant/wpa_cli` approach, do the following
```
#Close the default camera application
/oem/RkLunch-stop.sh
#Unlock peripheral devices
rfkill unblock all
killall wpa_supplicant
#Modify the wpa_supplicant.conf configuration file
vi /etc/wpa_supplicant.conf
#Replace SSID with WIFI name
#Replace PASSWORD with WiFi PASSWORD
wpa_supplicant -i wlan0 -c /etc/wpa_supplicant.conf -B & #WiFi WPA_Supplicant Background Connection
udhcpc -i wlan0 #Obtains IP address after successful WiFi connection
```

* ping test whether wlan0 is connected to the external network
```
#The following results indicate a successful connection
[root@RV1126_RV1109:/]# ifconfig wlan0
wlan0     Link encap:Ethernet  HWaddr 10:2C:6B:7E:18:90
          inet addr:172.20.10.10  Bcast:172.20.10.15  Mask:255.255.255.240
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:54 errors:0 dropped:3 overruns:0 frame:0
          TX packets:128 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:4803 (4.6 KiB)  TX bytes:16388 (16.0 KiB)
[root@RV1126_RV1109:/]# route add default gw 172.20.10.1 wlan0
[root@RV1126_RV1109:/]# ping -I wlan0 www.baidu.com
PING www.baidu.com (14.215.177.39) from 172.20.10.10 wlan0: 56(84) bytes of data.
64 bytes from 14.215.177.39 (14.215.177.39): icmp_seq=1 ttl=49 time=265 ms
64 bytes from 14.215.177.39 (14.215.177.39): icmp_seq=2 ttl=49 time=47.7 ms
64 bytes from 14.215.177.39 (14.215.177.39): icmp_seq=3 ttl=49 time=61.4 ms
64 bytes from 14.215.177.39 (14.215.177.39): icmp_seq=4 ttl=49 time=51.5 ms
64 bytes from 14.215.177.39 (14.215.177.39): icmp_seq=5 ttl=49 time=62.7 ms
```