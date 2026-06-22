## WIFI 使用
*` RV1109/RV1126` 平台默认使用 `connman` 管理 WiFi，而且 WiFi 的核心进程 `wpa_supplicant` 的启动方法由它启动：
```
ps -ef
#可以看到下面两个进程
connmand		   #它使用dbus跟wpa_supplicant进行通信
wpa_supplicant -u  #打开支持dbus通信
```
* 标准使用方法：通过 `RV1126/RV1109` 的 web 界面进行 WiFi 操作，参考 `RV1109/RV1126` 平台的相关文档：
* 终端简单测试方法如下：
```
killall ipc-daemon netserver
connmanctl
connmanctl> enable wifi
connmanctl> scan wifi #可以多次扫描
connmanctl> scan wifi #可以多次扫描
connmanctl> agent on
connmanctl> services
#列出扫描到的 wifi 列表
connmanctl>
	*AO yyz123
	NETGEAR75-5G	wifi_c0847daf6f42_4e45544745415237352d3547_managed_psk
	aaabbb			wifi_c0847daf6f42_616161626262_managed_psk
	HiWiFi-Free		wifi_c0847daf6f42_204869576946692d46726565_managed_none
	Fang-HiWiFi		wifi_c0847daf6f42_46616e672d486957694669_managed_psk
	yyz123				wifi_c0847daf6f42_79797a313233_managed_psk

connmanctl> connect wifi_c0847daf6f42_4e45544745415237352d3547_managed_psk
#假如要连接上面 NETGEAR75-5G，则 connect 的参数为后面的 wifixxx_psk
connmanctl>
Connected wifi_c0847daf6f42_4e45544745415237352d3547_managed_psk #如果连接成功则会有这个打印
connmanctl> quit #退出连接模式

ifconfig wlan0
#可以看到wlan0的IP地址
```
* 如果不想用connman，而使用传统的 `wpa_supplicant/wpa_cli` 的方式，则进行如下操作
```
#关闭默认摄像头应用
/oem/RkLunch-stop.sh
#解锁外设设备
rfkill unblock all
killall wpa_supplicant
#修改 wpa_supplicant.conf 配置文件
vi /etc/wpa_supplicant.conf
#将 SSID 替换成WiFi名称
#将 PASSWORD 替换成 WiFi 密码
wpa_supplicant -i wlan0 -c /etc/wpa_supplicant.conf -B & #WiFi wpa_supplicant 后台连接
udhcpc -i wlan0 #WiFi 连接成功后获取 IP 地址
```

* ping 测试 wlan0 是否已经连接外网
```
#下列结果表示连接成功
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