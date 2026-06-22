# 外设使用

## WiFi
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
## [4G 模组](https://store.t-firefly.com/goods.php?id=49)
查看 EC20 4G 模组套件是否已经加载
```
#加载成功会出现以下 4 个 USB 接口
ls /dev/ttyUSB*
/dev/ttyUSB0  /dev/ttyUSB1  /dev/ttyUSB2  /dev/ttyUSB3
```
如果没有加载成功，请检查内核是否打开以下的配置
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

如果4G模块上不了网，请检查内核是否打开以下配置
```
CONFIG_USB_USBNET
CONFIG_USB_NET_QMI_WWAN
```
查看EC20 4G网卡是否存在
```
ifconfig wwan0
```
EC20 4G 模组插 SIM 卡上网方法
```
# 从资料下载界面下载 ec20-armhf.tar.gz 压缩包并拷贝到开发板上解压
# 需要将文件夹的/ec20-armhf/usr/share/udhcpc/default.script替换掉系统的/usr/share/udhcpc/default.script
# 结果如下：
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
## 声卡

### EarPhone && Speak
EarPhone 和 Speak 均采用双声道接口

使用 aplay 命令播放 wav 格式音频
```
#path-to 表示存放音频的绝对路径
aplay /path-to/audio-name.wav
```
### Mic
* Mic 默认是关闭的，使用时需要将其打开，通过以下操作进行
```
#命令设置默认录制声卡通道
amixer cset numid=2,iface=MIXER,name='Capture MIC Path' 1

#图形界面设置默认录制声卡通道
alsamixer
#将 Capture MIC Path 设置为 Main Mic
```
* Mic录制音频
```
arecord -l											#查看所有可用的MIC设备
arecord -Dhw:0,0 -f cd -d 10 /path-to/audio.wav	#选择声卡并录制音频
#注：Mic 只支持单声道录音
```
## MIPI-CSI
* MIPI-CSI摄像头需要购买转接板
* v4l2接口操作MIPI-CSI摄像头
```
#判断驱动 probe 状态 RKISP 如果 probe 成功,会有 video 及 media 设备存在于 /dev/ 目录下。
#系统中可能存在多个 /dev/video 设备,通过 /sys 可以查询到RKISP注册的 video 节点。
#与之对应的是 rkispp_scale0 节点，如下打印也就是 /dev/video19
[root@RV1126_RV1109:/]# grep '' /sys/class/video4linux/video*/name
/sys/class/video4linux/video0/name:stream_cif_mipi_id0
/sys/class/video4linux/video1/name:stream_cif_mipi_id1
/sys/class/video4linux/video10/name:rkisp_rawwr3
/sys/class/video4linux/video11/name:rkisp_rawrd0_m
/sys/class/video4linux/video12/name:rkisp_rawrd2_s
/sys/class/video4linux/video13/name:rkisp_rawrd1_l
/sys/class/video4linux/video14/name:rkisp-statistics
/sys/class/video4linux/video15/name:rkisp-input-params
/sys/class/video4linux/video16/name:rkisp-mipi-luma
/sys/class/video4linux/video17/name:rkispp_input_image
/sys/class/video4linux/video18/name:rkispp_m_bypass
/sys/class/video4linux/video19/name:rkispp_scale0
/sys/class/video4linux/video2/name:stream_cif_mipi_id2
/sys/class/video4linux/video20/name:rkispp_scale1
/sys/class/video4linux/video21/name:rkispp_scale2
/sys/class/video4linux/video22/name:rkispp_iqtool
/sys/class/video4linux/video23/name:rkispp_input_params
/sys/class/video4linux/video24/name:rkispp-stats
/sys/class/video4linux/video3/name:stream_cif_mipi_id3
/sys/class/video4linux/video4/name:rkcif-mipi-luma
/sys/class/video4linux/video5/name:rkisp_mainpath
/sys/class/video4linux/video6/name:rkisp_selfpath
/sys/class/video4linux/video7/name:rkisp_rawwr0
/sys/class/video4linux/video8/name:rkisp_rawwr1
/sys/class/video4linux/video9/name:rkisp_rawwr2

# v4l2-ctl 抓取一帧图片并保存在 /tmp/cif.out
# /dev/video19 不是固定的，需要按照上面匹配得到相应的节点来配置
#抓取图片之前需要关闭默认的摄像头应用
/oem/RkLunch-stop.sh
/oem/usr/bin/dbserver &
/usr/bin/ispserver &

#开启 dbserver 和 ispserver 之后就可以进行抓图了
v4l2-ctl -d /dev/video19 \
--set-fmt-video=width=1920,height=1080,pixelformat=NV12 \
--stream-mmap=3 \
--stream-skip=3 \
--stream-to=/tmp/cif.out \
--stream-count=1 \
--stream-poll

# v4l2-ctl 抓取 10 帧连续图片作为一段视频并保存在 /tmp/test.yuv
v4l2-ctl -d /dev/video19  --try-fmt-video=width=1920,height=1080,pixelformat=NV12 --stream-mmap=3 --stream-to=/tmp/test.yuv --stream-count=10 --stream-poll

# Linux PC 使用 ffplay 播放抓取到的视频
ffplay -f rawvideo -video_size 1920x1080 -pix_fmt nv12 test.yuv
```

* 访问网络码流
使用支持 RTSP 或 RTMP 的播放器访问,例如( VLC 播放器)。
* RTSP访问地址:
```
rtsp://设备IP地址/live/mainstream
rtsp://设备IP地址/live/substream
rtsp://设备IP地址/live/thirdstream
```
* RTMP 访问地址:
```
rtmp://设备IP地址:1935/live/substream
```

例：VLC 播放器预览 RTSP 流
```
#注：需要安装 VLC 播放器
vlc rtsp://设备IP地址/live/mainstream
```
* 通过网页访问设备信息
打开 Web 浏览器(推荐 Chrome 浏览器)访问地址:
```
http://设备IP地址
#用户名和密码均是 admin
```
网页端详细的操作说明请参考 SDK 目录 `docs` 下的文档 `Rockchip_Instructions_Linux_Web_Configuration_CN.pdf`。

## MIPI-DSI

屏幕转向
```
#指定 QT 输出平台并使屏幕顺时针转向 90°
export QT_QPA_PLATFORM=linuxfb:rotation=90
```
* 如果屏幕出现花屏，或者不显示的情况。请检查LCD排线是否有问题，更换优质材质的短排线能解决花屏问题。

## LED

* 选择 GPIO 口，修改内核设备树，添加自定义 LED 灯设备节点。
```
vim sdk/kernel/arch/arm/boot/dts/rv1126-firefly-ai-cam.dts
# 参考如下配置添加 LED 节点。
leds {
    compatible = "gpio-leds";

	work {
        label = "firefly:blue:power";
        linux,default-trigger = "ir-power-click";
        gpios = <&gpio0 RK_PA4 GPIO_ACTIVE_HIGH>;
        pinctrl-names = "default";
        pinctrl-0 = <&led_power>;
        default-state = "on";
	};
    user {
        label = "firefly:yellow:user";
        linux,default-trigger = "ir-user-click";
        gpios = <&gpio0 RK_PC0 GPIO_ACTIVE_HIGH>;
        pinctrl-names = "default";
        pinctrl-0 = <&led_user>;
        default-state = "on";
	};
};

# pinctrl 需要添加 GPIO 复用控制
leds {
	led_power: led-power {
            rockchip,pins = <0 RK_PA4 RK_FUNC_GPIO &pcfg_pull_none>;
        };

        led_user: led-user {
            rockchip,pins = <0 RK_PC0 RK_FUNC_GPIO &pcfg_pull_none>;
        };
};
# 修改完成后编译升级内核
```

* 控制 LED
```
echo 1 > /sys/class/leds/firefly:yellow:user/brightness # 亮
echo 0 > /sys/class/leds/firefly:yellow:user/brightness # 灭
```
## POE
* 可添加 `POE` 功能的网口是 `1000M` 网口，如下图所示
![](../../../rv1126_img/Core-1126-JD4/POE-interface.png) 
需要将 `POE` 模组接到底板的 `POE` 接口处
```
#查看1000M POE网口是否存在
ifconfig eth1 
```