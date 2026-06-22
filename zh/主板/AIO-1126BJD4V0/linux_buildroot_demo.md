# Buildroot 使用手册
## 桌面应用

官方发布的 Buildroot 固件，默认已支持 Wayland 桌面环境以及一些 Qt 应用，如下图：

![](../../../rv1126b_img/linux_buildroot_desktop.png)

这些 Qt 应用提供了一些基础功能，如，摄像头预览，文件管理器，多媒体播放器，WiFi 连接等。

### multivideoplayer

![](../../../rv1126b_img/linux_buildroot_multi_video.png)

多路视频播放器用于测试设备的多路视频播放能力、显示能力以及硬件解码能力，点击启动应用后会自动循环播放 9 路视频 demo。

双击画面可以全屏播放。

### qfm

qfm 是一个文件浏览应用

![](../../../rv1126b_img/linux_buildroot_file_manage.png)

通过 qfm 可以浏览设备文件，点击目录名进入下一级目录，点击左上角返回上一级。

点击文件可以调用系统对应的默认应用打开文件，如点击 mp4 文件将调用 qplayer 应用打开视频。

### qplayer

qplayer 是一个多功能播放器，可以播放视频、音频和浏览图片。点击启动后将自动播放视频 demo。

![](../../../rv1126b_img/linux_buildroot_qplayer.png)

也可以通过在 qfm 中点击指定文件来使用 qplyer 打开。

双击画面可以全屏播放。

### qcamera

qcamera 是一款相机应用，可以进行拍摄和录像。

![](../../../rv1126b_img/linux_buildroot_qcamera.png)

设备连接摄像头的情况下启动 qcamera 将自动显示摄像头画面，右侧按钮：

* Image Mode: 照相模式，点击可切换为 Video Mode 视频录制模式。
* Capture: 捕捉图像，在 Video Mode 下会变为 Record 录制按钮。
* Exit: 退出。

### qsetting

![](../../../rv1126b_img/linux_buildroot_setting.png)

qsetting 是系统设置工具，其中可以设置 WiFi 连接，蓝牙连接，实现恢复出厂设置以及固件升级。

具体内容将在后面相应部分展示。

## 用户和密码

* 用户：root
* 密码：rockchip

## WiFi 连接

* 桌面操作

可以通过 qsetting 应用来连接 WiFi 。

![](../../../rv1126b_img/linux_buildroot_wifi_connect.png)

进入 qsetting 点击 WiFi 后，点击右上角开关打开 WiFi ，设备将自动搜索周围WiFi。

点击想要连接的WiFi名，输入密码完成连接。

* 命令操作

也可以通过命令实现WiFi连接：
```bash
# wifi_start.sh wifi-ssid password
wifi_start.sh wifi-test 12345678
```

## 蓝牙

### 扫描与配对

* 桌面操作

可以通过 qsetting 应用来进行蓝牙扫描与配对。

进入 qsetting 之后点击 BT ，然后点击右上角按钮打开蓝牙，设备会自动搜索附近蓝牙设备等待配对。

此时打开手机蓝牙，在可用设备中找到 Rockchip Linux BT 点击配对。配对成功后 qsetting 相应设备前会显示 Paired 。

![](../../../rv1126b_img/linux_buildroot_bt.png)

* 命令操作

也可以通过执行脚本`/usr/bin/bt_init.sh`来初始化蓝牙。直接输入`bt_init.sh`来执行。

执行脚本之后使用`hciconfig hci0 up`命令打开蓝牙。

使用`hcitool scan`进行周围蓝牙设备扫描：
```bash
[root@rk356x:~]# hcitool scan
Scanning ...
        01:02:03:04:05:06       device1
        0A:0B:0C:0D:0E:0F       device2
```

### 连接与使用

蓝牙的连接以及具体功能的实现需要用到 Buildroot 已经预装的 broadcom_bsa 脚本或 deviceio 工具。

#### Broadcom_bsa

Broadcom_bsa 包提供了一些脚本用于功能实现，首先执行`bsa_server.sh start`开启服务。
```bash
[root@rk356x:~]# bsa_server.sh start
hcd_file = /vendor/etc/firmware/BCM4362A2.hcd
ttys_dev = /dev/ttyS8
killall: bsa_server: no process killed
/usr/bin/bsa_server.sh: line 41: check_not_exist.sh: not found
[ 2275.880487] [BT_RFKILL]: bt shut off power
[ 2275.886378] Bluetooth: hci0: sending frame failed (-49)
[ 2276.918325] [BT_RFKILL]: rfkill_rk_set_power: set bt wake_host high!
[ 2276.969598] [BT_RFKILL]: ENABLE UART_RTS
[ 2277.076347] [BT_RFKILL]: DISABLE UART_RTS
[ 2277.076483] [BT_RFKILL]: bt turn on power
[ 2277.076595] [BT_RFKILL]: Request irq for bt wakeup host
[ 2277.076738] [BT_RFKILL]: ** disable irq
start broadcom bluetooth server bsa_sever
|----- bluetooth bsa server is open ------|
```

* 蓝牙音乐

脚本 bsa_bt_sink.sh 用于蓝牙音乐功能，执行`bsa_bt_sink.sh start`开启。
```bash
[root@rk356x:/]# bsa_bt_sink.sh start
hcd_file = /vendor/etc/firmware/BCM4362A2.hcd
......
......
start broadcom bluetooth server bsa_sever
......
......
start broadcom bluetooth app_manager
start broadcom bluetooth app_avk
......
......
BSA_trace 6@ 01/01 00h:48m:18s:632ms: BSA_SecSetSecurityInit
BSA_trace 7@ 01/01 00h:48m:18s:632ms: BSA_SecSetSecurity
Server is ready for client connect...
......
......
#########act as a bluetooth music player#########
send msg:2 to app_avk
client connted
msg = 2, choice :2
......
......
|----- bluetooth music player ------|
```

出现以上关键信息即可说明蓝牙音乐服务正常开启，其他打印过多，省略不予展示。

此时打开手机蓝牙，在可用设备中找到 ROCKCHIP_AUDIO ，点击配对并连接，连接成功后出现如下打印：
```bash
BSA_trace 16@ 01/01 00h:54m:43s:938ms: bsa_sec_event_hdlr event:0
BSA_trace 17@ 01/01 00h:54m:44s:394ms: bsa_sec_event_hdlr event:6
BSA_trace 18@ 01/01 00h:54m:44s:395ms: BSA_SecSpCfmReplyInit
BSA_trace 19@ 01/01 00h:54m:44s:395ms: BSA_SecSpCfmReply
BSA_trace 20@ 01/01 00h:54m:47s:905ms: bsa_sec_event_hdlr event:3
Update name with device1 # 已连接的设备名
Update link key
BSA_trace 21@ 01/01 00h:54m:49s:088ms: bsa_sec_event_hdlr event:5
BSA_trace 22@ 01/01 00h:54m:49s:089ms: BSA_SecAuthorizeReplyInit
BSA_trace 23@ 01/01 00h:54m:49s:089ms: BSA_SecAuthorizeReply
Added trusted services
Update name with device1
BSA_trace 17@ 01/01 00h:54m:49s:102ms: BSA_DmSetConfigInit
BSA_trace 18@ 01/01 00h:54m:49s:102ms: BSA_DmSetConfig
AVK connected to device 01:02:03:04:05:06 # 已连接的设备地址
BSA_trace 19@ 01/01 00h:54m:52s:195ms: BSA_AvkRegNotifRspInit
BSA_trace 20@ 01/01 00h:54m:52s:195ms: BSA_AvkRegNotifRsp
amixer: Unable to find simple control 'Master',0

BSA_trace 21@ 01/01 00h:54m:52s:247ms: BSA_AvkSetAbsVolumeRspInit
BSA_trace 22@ 01/01 00h:54m:52s:247ms: BSA_AvkSetAbsVolumeRsp
```

此时设备可以作为蓝牙音响播放手机中的音乐了。

* 蓝牙通话

脚本 bsa_bt_hfp.sh 用于蓝牙通话功能，执行`bsa_bt_hfp.sh start`开启。
```bash
[root@rk356x:/]# bsa_bt_hfp.sh start
hcd_file = /vendor/etc/firmware/BCM4362A2.hcd
......
......
start broadcom bluetooth bsa_sever
......
......
start broadcom bluetooth app_manager
start broadcom bluetooth app_hs
......
......
Created thread with thread_id=  b07781e0
......
......
Server is ready for client connect...
|----- bluetooth hfp is start------|
```

出现以上关键信息即可说明蓝牙通话服务正常开启。

此时打开手机蓝牙，在可用设备中找到 ROCKCHIP_AUDIO ，点击配对并连接，连接成功后出现如下打印：
```bash
BSA_trace 16@ 01/01 01h:11m:36s:770ms: bsa_sec_event_hdlr event:0
BSA_trace 17@ 01/01 01h:11m:37s:150ms: bsa_sec_event_hdlr event:6
BSA_trace 18@ 01/01 01h:11m:37s:151ms: BSA_SecSpCfmReplyInit
BSA_trace 19@ 01/01 01h:11m:37s:151ms: BSA_SecSpCfmReply
BSA_trace 20@ 01/01 01h:11m:40s:147ms: bsa_sec_event_hdlr event:3
Update name with device1 # 已连接的设备名
......
......
Call Indicator  ("call",(0,1)),("callsetup",(0-3)),("service",(0-1)),("signal",(0-5)),("roam",(0,1)),("battchg",(0-5)),("callheld",(0-2))
......
......
BSA_HS_CIND_EVT
Call Indicator  0,0,1,5,0,4,0
BSA_trace 21@ 01/01 01h:11m:41s:137ms: Service: SERVICE AVAILABLE,1
......
......
Call Ind Status  4,20
......
......
BSA_HS_CONN_EVT:
    - Remote bdaddr: 01:02:03:04:05:06 # 已连接的设备地址
    - Service: Handsfree
```

此时设备可作为蓝牙耳机接听手机电话。

* BLE 调试

脚本 bsa_ble_wifi_introducer.sh 用于蓝牙 BLE 调试，另外手机上需要在应用商店下载任意 BLE 调试工具。执行`bsa_ble_wifi_introducer.sh start`启动。
```bash
[root@rk356x:/]# bsa_ble_wifi_introducer.sh start
hcd_file = /vendor/etc/firmware/BCM4362A2.hcd
[ 5021.786246] [BT_RFKILL]: bt shut off power
[ 5024.823746] [BT_RFKILL]: rfkill_rk_set_power: set bt wake_host high!
[ 5024.875698] [BT_RFKILL]: ENABLE UART_RTS
[ 5024.982482] [BT_RFKILL]: DISABLE UART_RTS
[ 5024.982597] [BT_RFKILL]: bt turn on power
[ 5024.982693] [BT_RFKILL]: Request irq for bt wakeup host
[ 5024.982807] [BT_RFKILL]: ** disable irq
start broadcom bluetooth server bsa_sever
killall: bsa_server: no process killed
BSA_trace 1@ 01/01 01h:23m:48s:014ms: bsa_sv_dm_set_3d_type type=chip_id (not safe)
BSA_trace 2@ 01/01 01h:23m:48s:016ms: Command [all] with arg [0]
BSA_trace 3@ 01/01 01h:23m:48s:016ms: [ all ] trace level = 0
BSA_trace 4@ 01/01 01h:23m:48s:017ms: BTM_SetTraceLevel
start broadcom bluetooth app_manager
start broadcom bluetooth wifi introducer
Created thread with thread_id=  aff4a1f0
Created thread with thread_id=  8195b1f0
|----- bluetooth ble wifi config ------|
```

出现以上打印即可说明蓝牙 BLE 调试开启。

打开手机的 BLE 调试工具，找到 WiFiInt 配对并连接，找到任意一个支持 WRITE 的服务向其写入数据。

![](../../../rv1126b_img/linux_buildroot_ble.png)

之后在`/data/bsa/app_ble_wifi_introducer.log`中可以看到手机发送来的信息：
```bash
^[[30;47mBSA_trace 46@ 01/01 01h:36m:19s:150ms: Write value:^[[30;47m^M
^[[30;47mBSA_trace 47@ 01/01 01h:36m:19s:150ms:     0000: 48 65 6c 6c 6f
# 48 65 6c 6c 6f 即 Hello 的 HEX 码
```

#### deviceio

Deviceio 是一个应用开发库，它屏蔽了底层复杂的Wi-Fi/BT操作，向上提供友好的应用开发接口。

执行`deviceio_test bluetooth`开启，之后会进入选择菜单：
```bash
[root@rk356x:/]# deviceio_test bluetooth
version:V1.3.5
#### Please Input Your Test Command Index ####
01.  bt_server_open
02.  bt_test_set_class
03.  bt_test_get_device_name
04.  bt_test_get_device_addr
05.  bt_test_set_device_name
......
......
Which would you like:
```

首先输入“1”回车，即选择 bt_server_open 开启服务。
```bash
bt_test_bluetooth_init_thread: BT BLUETOOTH INIT
++++++++++ RK_BT_STATE_TURNING_ON ++++++++++
hcd_file = /vendor/etc/firmware/BCM4362A2.hcd
ttys_dev = /dev/ttyS8
......
......
start broadcom bluetooth server bsa_sever
|----- bluetooth bsa server is open ------|
......
......
Created thread with thread_id=  aab541c0
......
......
++++++++++ RK_BT_STATE_ON ++++++++++
```

出现以上关键信息说明开启成功。

* 蓝牙音乐

选择 bt_test_start_discovery 搜索蓝牙，即输入“8 input [搜索时间]”，等待一小段时间后出现：
```bash
++++++++++++ Device is found ++++++++++++
    address: 01:02:03:04:05:06
    name: device1
    class: 0x5a020c
    rssi: -60
+++++++++++++++++++++++++++++++++++++++++
++++++++++++ Device is found ++++++++++++
    address: 0A:0B:0C:0D:0E:0F
    name: device2
    class: 0x5a020c
    rssi: -67
+++++++++++++++++++++++++++++++++++++++++
......
......
```

选择 bt_test_sink_open 打开 sink 服务，即输入“31”

选择 bt_test_sink_connect_by_addr 连接刚才搜索到的手机蓝牙地址，即输入“45 input [bt addr]”，出现如下信息说明连接成功：
```bash
BSA_trace 41@ 01/01 02h:16m:26s:529ms: bsa_sec_event_hdlr event:6
BSA_trace 42@ 01/01 02h:16m:26s:530ms: BSA_SecSpCfmReplyInit
BSA_trace 43@ 01/01 02h:16m:26s:531ms: BSA_SecSpCfmReply
++++++++++ BT BOND BONDING: device1, 01:02:03:04:05:06 ++++++++++
BSA_trace 44@ 01/01 02h:16m:27s:696ms: bsa_sec_event_hdlr event:3
BSA_trace 45@ 01/01 02h:16m:27s:715ms: BSA_DiscStartInit
BSA_trace 46@ 01/01 02h:16m:27s:715ms: BSA_DiscStart
++++++++++ BT BONDED: device1, 01:02:03:04:05:06 ++++++++++
BSA_trace 47@ 01/01 02h:16m:27s:898ms: BSA_DmSetConfigInit
BSA_trace 48@ 01/01 02h:16m:27s:898ms: BSA_DmSetConfig
++++++++++++ BT SINK EVENT: connect sucess ++++++++++
BSA_trace 49@ 01/01 02h:16m:27s:926ms: BSA_AvkGetElementAttrCmdInit
BSA_trace 50@ 01/01 02h:16m:27s:926ms: BSA_AvkGetElementAttrCmd
BSA_trace 51@ 01/01 02h:16m:27s:928ms: BSA_AvkRegNotifRspInit
BSA_trace 52@ 01/01 02h:16m:27s:928ms: BSA_AvkRegNotifRsp
```

手机点击配对后输入“39”选择 bt_test_sink_music_play 就可以播放手机的音乐了。
```bash
deviceio_test_bluetooth: szBuf =  39
deviceio_test_bluetooth: selset 39
BSA_trace 55@ 01/01 02h:19m:05s:098ms: BSA_AvkRemoteCmdInit
BSA_trace 56@ 01/01 02h:19m:05s:098ms: BSA_AvkRemoteCmd
++++++++++++ BT A2DP SINK STATE: started ++++++++++
BSA_trace 57@ 01/01 02h:19m:05s:400ms: BSA_AvkRemoteCmdInit
BSA_trace 58@ 01/01 02h:19m:05s:400ms: BSA_AvkRemoteCmd
```

**注意：** 功能编号可能不同，请按实际情况选择。

* 其他

其他功能如 hfp 和 BLE 连接与使用方法与蓝牙音乐基本相同，选择对用的功能编号即可。开启不同服务后，设备名称均以 ROCKCHIP 开头，手机连接时加以注意。

BLE 连接成功后可参考 Broadcom_bsa 中 BLE 部分进行数据发送，deviceio 会自动显示收到的信息，无需在 log 中查看。

其他有关 deviceio 的功能参考 [Gitlab](https://gitlab.com/firefly-linux/external/deviceio_release) 。

## 音/视频播放

```bash
# 播放 wav
aplay test.wav
gstwavplay.sh test.wav

# 播放 mp3
mp3play.sh test.mp3
gstmp3play.sh test.mp3

# 播放 mp4
gstmp4play.sh test.mp4
gstvideoplay.sh test.mp4
```

## SSH

Buildroot 默认已支持 SSH 服务。

* ssh 登录设备

让设备处于联网状态，在 PC 端执行登录命令：

```bash
# 用户：root；密码：rockchip
# IP：192.168.27.129
~$ ssh root@192.168.27.129
The authenticity of host '192.168.27.129 (192.168.27.129)' can't be established.
ECDSA key fingerprint is SHA256:+NwFawDiU0EwLHRFrIA/7snmlMc9ZfN6Nxa5vUSC7Pg.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.27.129' (ECDSA) to the list of known hosts.
root@192.168.27.129's password:
[root@rk356x:~]#
```

* scp

在 PC 端执行命令，拷贝文件到设备：

```bash
~$ scp ./file root@192.168.27.129:/
root@192.168.27.129's password:
file                           100%  154     0.2KB/s   00:00
```


## 外部存储设备

Buildroot 支持自动挂载外部存储设备：

U 盘挂载路径：`/udisk`

TF 卡挂载路径：`/sdcard`

## 恢复出厂设置

Buildroot 支持恢复出厂设置。

**注意：此出厂设置表示恢复为设备最后一次升级固件之后的初始状态。**

* 桌面操作

可以在 qsetting 中进行恢复出厂设置。

![](../../../rv1126b_img/linux_buildroot_factory_reset.png)

进入 qsetting 后点击 Factory Reset，点击 OK 按钮执行操作。

* 命令操作

也可以命令行直接运行 `update` 或者加上 `factory/reset` 参数均可进入 recovery 恢复出厂设置。

```bash
update
# 或者
update factory / update reset
```

## 升级固件

Buildroot 支持从外部存储设备升级固件，以下是升级流程说明。

### 制作升级固件

按照正常的固件编译流程，制作用于升级的固件。[《编译 Buildroot 固件》](linux_buildroot_compile.md)

升级固件不一定要全分区升级，可修改 `package-file` 文件，将不要升级的分区去掉，这样可以减少升级包的大小。

例如，修改文件 `tools/linux/Linux_Pack_Firmware/rockdev/rk356x-package-file`，将 `rootfs` 的相对路径改为 `RESERVED`，这样就不会打包根文件系统，即不升级根文件系统分区。

```bash
# name          relative path
#
#hwdef          hwdef
package-file    package-file
bootloader      image/miniloaderall.bin
parameter       image/parameter.txt
trust           image/trust.img
uboot           image/uboot.img
misc            image/misc.img
boot            image/boot.img
recovery        image/recovery.img
rootfs          RESERVED
oem             image/oem.img
userdata:grow   image/userdata.img
backup          RESERVED
```

将制作好的升级固件拷贝到 U 盘、TF 卡或者设备的 `/userdata/` 目录下，重命名为 `update.img`。

**注意：** 若将升级固件放至设备的 `/userdata/` 目录，则不要打包 `userdata.img`，将 `image/userdata.img` 改为 `RESERVED`。

### 升级

* 桌面操作

可以通过 qsetting 进行升级

![](../../../rv1126b_img/linux_buildroot_update.png)

将升级固件放入正确位置后，进入 qsetting 点击 Update，点击 OK 开始升级。

* 命令操作

也可以在命令行执行升级命令，`update ota path/to/update.img`，设备会进入 recovery 进行升级。

```bash
# U 盘
update ota /udisk/update.img
# TF 卡
update ota /sdcard/update.img
# /userdata/
update ota /userdata/udpate.img
```

等待升级完成，升级成功后，设备会重新启动进入系统。

## FirmwareOta

firmwareota 是一个 Qt 桌面应用，支持从网络下载固件进行固件升级，下面是使用说明。

### 添加桌面应用

* 下载

[下载链接](https://pan.baidu.com/s/1cvuxjUlbgXHOnr-6gwakww)(提取码：1234)

* 解压

```bash
tar -zxvf firmwareota.tar.gz
```

* 按照目录结构，将文件拷贝到设备

```bash
└── usr
    ├── bin
    │   └── firmwareota
    └── share
        ├── applications
        │   └── firmwareota.desktop
        └── icon
            └── icon_firmwareota.png
```

* 重启桌面

```bash
/etc/init.d/S50launcher stop
/etc/init.d/S50launcher start
```

* 启动

设置完成后，则可以在桌面启动应用。

![](../../../rv1126b_img/linux_buildroot_firmwareota.png)

### 设置下载地址

程序会通过默认的 Http 地址下载列表文件 `ota_list`，解析列表文件获取固件下载链接。

列表文件格式，例：

```bash
firmware_name_1
http://192.168.2.11:8000/firmware_name_1
firmware_name_2
http://192.168.2.11:8000/firmware_name_2
...
```

我们可以通过添加 `/userdata/list_url` 文件，设置指定列表文件下载地址，例：

```bash
http://192.168.2.11:8000/ota_list
```
## Weston 配置

我们可以通过配置 Weston 对显示进行一些自定义设置，下文对部分设置进行说明。

### 状态栏设置

Weston 支持在 weston.ini 配置文件的 shell 段设置状态栏的背景色、位置，以及在 launcher 段设置快捷启动程序，如：

```ini
# /etc/xdg/weston/weston.ini

[shell]
# 颜色格式为 ARGB8888
panel-color=0xff002244
# top|bottom|left|right|none
panel-position=bottom

[launcher]
icon=/usr/share/weston/terminal.png
path=/usr/bin/weston-terminal

[launcher]
# 图标路径
icon=/usr/share/weston/icon_flower.png
# 快捷启动命令
path=/usr/bin/qsetting
```

### 背景设置

Weston 支持在 weston.ini 配置文件的 shell 段设置背景图案、颜色，如：

```ini
# /etc/xdg/weston/weston.ini

[shell]
# 背景图案(壁纸)绝对路径
background-image=/usr/share/weston/background.png
# scale|scale-crop|tile
background-type=scale
# 颜色格式为 ARGB8888，未设置背景图案时生效
background-color=0xff002244
```

### 待机及锁屏配置

Weston 的超时待机时长可以在启动参数中配置，也可以在 weston.ini 的 core 段配置，如：

```bash
# /etc/init.d/S50launcher
    start)
        ...
        # 0 为禁止待机，单位为秒
        weston --tty=2 -B=drm-backend.so --idle-time=0&
```

或者：

```ini
# /etc/xdg/weston/weston.ini

[core]
# 设置 5 秒未操作后进入待机状态
idle-time=5
```

### 显示颜色格式配置

Buildroot SDK 内 Weston 目前默认显示格式为 ARGB8888，对于某些低性能平台，可以在 weston.ini 的 core 段配置为 RGB565，如：

```ini
# /etc/xdg/weston/weston.ini

[core]
# xrgb8888|rgb565|xrgb2101010
gbm-format=rgb565
```

也可以在 weston.ini 的 output 段单独配置每个屏幕的显示格式，如：

```ini
# /etc/xdg/weston/weston.ini

[output]
# output 的 name 可以查看 /sys/class/drm/card0-name
name=LVDS-1
# xrgb8888|rgb565|xrgb2101010
gbm-format=rgb565
```

### 屏幕方向设置

Weston 的屏幕显示方向可以在 weston.ini 的 output 段配置，如：

```ini
# /etc/xdg/weston/weston.ini

[output]
name=LVDS-1
# normal|90|180|270|flipped|flipped-90|flipped-180|flipped-270
transform=180
```

如果需要动态配置屏幕方向，可以通过动态配置文件，如：

```bash
echo "output:all:rotate90" > /tmp/.weston_drm.conf # 所有屏幕旋转 90 度
echo "output:eDP-1:rotate180" > /tmp/.weston_drm.conf # eDP-1 旋转 180 度
```

### 分辨率及缩放配置

Weston 的屏幕分辨率及缩放可以在 weston.ini 的 output 段配置，如：

```ini
# /etc/xdg/weston/weston.ini

[output]
name=HDMI-A-1
# 需为屏幕支持的有效分辨率
mode=1920x1080
# 需为整数倍数
scale=2
```

如果需要动态配置分辨率及缩放，可以通过动态配置文件，如：

```bash
echo "output:HDMI-A-1:mode=800x600" > /tmp/.weston_drm.conf # 修改 HDMI-A-1 分辨率为800x600
```

这种方式缩放时需要依赖 RGA 加速。

### 冻结屏幕

在启动 Weston 时，开机 logo 到 UI 显示之间存在短暂切换黑屏。如需要防止黑屏，可以通过以下种动态配置文件方式短暂冻结 Weston 屏幕内容：

```bash
# /etc/init.d/S50launcher
    start)
        ...
        export WESTON_FREEZE_DISPLAY=/tmp/.weston_freeze # 设置特殊配置文件路径
        touch /tmp/.weston_freeze # 冻结显示
        weston --tty=2 -B=drm-backend.so --idle-time=0&
        ...
        sleep 1 && rm /tmp/.weston_freeze& # 1 秒后解冻
```

### 多屏配置

Buildroot SDK 的 Weston 支持多屏同异显及热拔插等功能，不同显示器屏幕的区分根据 drm 的 name (通过 /sys/class/drm/card0-name 获取)，相关配置通过环境变量设置，如：

```bash
# /etc/init.d/S50launcher

    start)
        ...
        export WESTON_DRM_PRIMARY=HDMI-A-1 # 指定主显为 HDMI-A-1
        export WESTON_DRM_MIRROR=1 # 使用镜像模式(多屏同显)，不设置此环境变量即为异显
        export WESTON_DRM_KEEP_RATIO=1 # 镜像模式下缩放保持纵横比，不设置此变量即为强制全屏
        export WESTON_DRM_PREFER_EXTERNAL=1 # 外置显示器连接时自动关闭内置显示器
        export WESTON_DRM_PREFER_EXTERNAL_DUAL=1 # 外置显示器连接时默认以第一个外显为主显
        weston --tty=2 -B=drm-backend.so --idle-time=0&
```

镜像模式缩放显示内容时需要依赖 RGA 加速。

同时也支持在 weston.ini 的 output 段单独禁用指定屏幕：

```ini
# /etc/xdg/weston/weston.ini

[output]
name=LVDS-1
mode=off
# off|current|preferred|<WIDTHxHEIGHT@RATE>
```

### 输入设备相关配置

Weston 服务默认需要至少一个输入设备，如无输入设备，则需要在 weston.ini 中的 core 段特殊设置：

```ini
# /etc/xdg/weston/weston.ini

[core]
require-input=false
```