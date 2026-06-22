# Buildroot Manual
## Desktop application

The officially released Buildroot firmware supports the Wayland desktop environment and some Qt applications by default, as shown below:

![](../../../rv1126b_img/linux_buildroot_desktop.png)

These Qt applications provide some basic functions, such as camera preview, file manager, multimedia player, WiFi connection, etc.

### multivideoplayer

![](../../../rv1126b_img/linux_buildroot_multi_video.png)

The multivideoplayer is used to test the device's multi-channel video playback ability, display ability and hardware decoding ability. After clicking to start the application, it will automatically play 9-channel video demo in a loop.

Double-click the video image to play in full screen.

### qfm

The qfm is a file browser.

![](../../../rv1126b_img/linux_buildroot_file_manage.png)

Via qfm you can browse device's files, click directory name to enter the next level of directory, click the upper left corner button to return to the previous level.

Click a file can call the system default application to open it, for example, click a mp4 file will call qplayer to play it.

### qplayer

The qplayer is a multi-function player, it can play videos, audios and display pictures. Click to launch this application and it will play video demo automatically.

![](../../../rv1126b_img/linux_buildroot_qplayer.png)

You can also specify a file in qfm to open it with qplyer.

Double-click the video image to play in full screen.

### qcamera

The qcamera is a camera application, can take photos and record videos.

![](../../../rv1126b_img/linux_buildroot_qcamera.png)

Launch the qcamera when device is connected to a camera, qcamera will automatically display the camera pictures. And the buttons on the right side are:

* Image Mode: Now is image mode, click to change to the video record mode.
* Capture: capture image，it will change to Record button when in video record mode.
* Exit: close the application.

### qsetting

![](../../../rv1126b_img/linux_buildroot_setting.png)

The qsetting is a system setting tool, you can config WiFi connection, bluetooth connection and perform factory reset, firmware update in it.

The details will be shown in the later sections.

## User and password

* User: root
* Password: rockchip

## WiFi connection

* Desktop operation

You can connect to WiFi via qsetting tool.

![](../../../rv1126b_img/linux_buildroot_wifi_connect.png)

Enter qsetting and click WiFi, click upper right corner button to open WiFi. The device will automatically search for nearby WiFi.

Click the name of the WiFi that you want to connect to and enter the password to complete the connection.

* Command operation

You can also connect to WiFi with commands:

```bash
# wifi_start.sh wifi-ssid password
wifi_start.sh wifi-test 12345678
```

## Bluetooth

### Scan and pairing

* Desktop operation

You can perform bluetooth scanning and pairing via qsetting application.

Enter the qsetting and click BT, then click the upper right corner button to open bluetooth. The device will automatically search for nearby devices and waiting for pairing.

Now turn on your phone bluetooth, find `Rockchip Linux BT` in available devices and click it to start pairing. After the pairing is completed, `Paired` will be displayed in front of the corresponding device in the qsetting.

![](../../../rv1126b_img/linux_buildroot_bt.png)

* Command operation

You can also initialize the bluetooth with `/usr/bin/bt_init.sh`. Directly input `bt_init.sh` and execute.

After running the script, execute command `hciconfig hci0 up` to open the bluetooth.

Use command `hcitool scan` to scan nearby available bluetooth devices:
```bash
[root@rk356x:~]# hcitool scan
Scanning ...
        01:02:03:04:05:06       device1
        0A:0B:0C:0D:0E:0F       device2
```

### Connection and usage

To achieve bluetooth connection and specific functions, we have to use broadcom_bsa scripts or deviceio tools that are already installed in Buildroot.

#### Broadcom_bsa

Broadcom_bsa package provides some scripts for function implementaion. First execute command `bsa_server.sh start` to open services.
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

* Bluetooth music

The script `bsa_bt_sink.sh` is used for bluetooth music function, execute command `bsa_bt_sink.sh start` to open sink service.
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

The device prints these important logs above means the bluetooth music service is normally enabled. Too many other logs, omit not to show.

Then turn on the phone bluetooth, find `ROCKCHIP_AUDIO` in available devices, click it to pair and connect. When the connection succeeds, you can see these logs below:
```bash
BSA_trace 16@ 01/01 00h:54m:43s:938ms: bsa_sec_event_hdlr event:0
BSA_trace 17@ 01/01 00h:54m:44s:394ms: bsa_sec_event_hdlr event:6
BSA_trace 18@ 01/01 00h:54m:44s:395ms: BSA_SecSpCfmReplyInit
BSA_trace 19@ 01/01 00h:54m:44s:395ms: BSA_SecSpCfmReply
BSA_trace 20@ 01/01 00h:54m:47s:905ms: bsa_sec_event_hdlr event:3
Update name with device1 # Name of device that connected
Update link key
BSA_trace 21@ 01/01 00h:54m:49s:088ms: bsa_sec_event_hdlr event:5
BSA_trace 22@ 01/01 00h:54m:49s:089ms: BSA_SecAuthorizeReplyInit
BSA_trace 23@ 01/01 00h:54m:49s:089ms: BSA_SecAuthorizeReply
Added trusted services
Update name with device1
BSA_trace 17@ 01/01 00h:54m:49s:102ms: BSA_DmSetConfigInit
BSA_trace 18@ 01/01 00h:54m:49s:102ms: BSA_DmSetConfig
AVK connected to device 01:02:03:04:05:06 # Mac address of device that connected
BSA_trace 19@ 01/01 00h:54m:52s:195ms: BSA_AvkRegNotifRspInit
BSA_trace 20@ 01/01 00h:54m:52s:195ms: BSA_AvkRegNotifRsp
amixer: Unable to find simple control 'Master',0

BSA_trace 21@ 01/01 00h:54m:52s:247ms: BSA_AvkSetAbsVolumeRspInit
BSA_trace 22@ 01/01 00h:54m:52s:247ms: BSA_AvkSetAbsVolumeRsp
```

Now the device acts as a bluetooth speaker and can play music in your phone.

* Bluetooth phone

The script `bsa_bt_hfp.sh` is used for Hands-free Profile, execute `bsa_bt_hfp.sh start` to open the service.
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

The device prints these important logs above means HFP service is normally enabled.

Then turn on the phone  bluetooth, find `ROCKCHIP_AUDIO` in available devices, click it to pair and connect. When the connection succeeds, you can see these logs below:
```bash
BSA_trace 16@ 01/01 01h:11m:36s:770ms: bsa_sec_event_hdlr event:0
BSA_trace 17@ 01/01 01h:11m:37s:150ms: bsa_sec_event_hdlr event:6
BSA_trace 18@ 01/01 01h:11m:37s:151ms: BSA_SecSpCfmReplyInit
BSA_trace 19@ 01/01 01h:11m:37s:151ms: BSA_SecSpCfmReply
BSA_trace 20@ 01/01 01h:11m:40s:147ms: bsa_sec_event_hdlr event:3
Update name with device1 # Name of device that connected
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
    - Remote bdaddr: 01:02:03:04:05:06 # Mac address of device that connected
    - Service: Handsfree
```

Now device acts as a bluetooth headphone and you can pick up phone calls with it.

* BLE Config

The script `bsa_ble_wifi_introducer.sh` is used for BLE config. You need to download any BLE config application in your phone. Execute `bsa_ble_wifi_introducer.sh start` to start.
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

The device prints these logs above means BLE config function is normally enabled.

Open the BLE config application in your phone, find `WiFiInt` and click it to pair and connect. Find any service that supports WRITE and send any message to it.

![](../../../rv1126b_img/linux_buildroot_ble.png)

Then you can find messages from your phone in `/data/bsa/app_ble_wifi_introducer.log` :
```bash
^[[30;47mBSA_trace 46@ 01/01 01h:36m:19s:150ms: Write value:^[[30;47m^M
^[[30;47mBSA_trace 47@ 01/01 01h:36m:19s:150ms:     0000: 48 65 6c 6c 6f
# 48 65 6c 6c 6f is the HEX code of "Hello"
```

#### deviceio

Deviceio is an application development library that shields the complex WiFi/BT operations at the bottom and provides a friendly API up front.

Execute `deviceio_test bluetooth` to open deviceio, and then you will see the menu:
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

First input `1` and press enter to select `bt_server_open`.
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

The device prints these important logs above means bsa_server is normally opened.

* Bluetooth music

Select `bt_test_start_discovery` to search other bluetooth devices, that is, input `8 input [time]`, wait for a while and you can see:
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

Select `bt_test_sink_open` to open the sink service, that is, input `31`.

Select `bt_test_sink_connect_by_addr` to connect to the phone that you have just found, that is, input `45 input [bt addr]`, and you will see these logs below if connection succeeds:
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

After connection, input `39` to select `bt_test_sink_music_play`. Now device acts as a bluetooth speaker and can play music in your phone.
```bash
deviceio_test_bluetooth: szBuf =  39
deviceio_test_bluetooth: selset 39
BSA_trace 55@ 01/01 02h:19m:05s:098ms: BSA_AvkRemoteCmdInit
BSA_trace 56@ 01/01 02h:19m:05s:098ms: BSA_AvkRemoteCmd
++++++++++++ BT A2DP SINK STATE: started ++++++++++
BSA_trace 57@ 01/01 02h:19m:05s:400ms: BSA_AvkRemoteCmdInit
BSA_trace 58@ 01/01 02h:19m:05s:400ms: BSA_AvkRemoteCmd
```

**Note:** Serial number of functions may be different, select functions according to the actual situation.

* Other funtions

The usage of other functions, like HFP and BLE, is basically similar with bluetooth music, just select the correct function number. The device name will all start with `ROCKCHIP` despites different services are opened, notice that when connect to the device with your phone.

After successful connection with BLE, please refer to the BLE part in Broadcom_bsa section to perform data transmission, deviceio will automatically display the message it received, no need to check any log files.

For details of other functions in deviceio, please refer to [Gitlab](https://gitlab.com/firefly-linux/external/deviceio_release) .

## Audio/Video playback

```bash
# Play wav
aplay test.wav
gstwavplay.sh test.wav

# Play mp3
mp3play.sh test.mp3
gstmp3play.sh test.mp3

# Play mp4
gstmp4play.sh test.mp4
gstvideoplay.sh test.mp4
```

## SSH

Buildroot already supports SSH service by default.

* ssh login device

Let the device be online, execute the login command on the PC:

```bash
# User: root; Password: rockchip
# IP: 192.168.27.129
~$ ssh root@192.168.27.129
The authenticity of host '192.168.27.129 (192.168.27.129)' can't be established.
ECDSA key fingerprint is SHA256:+NwFawDiU0EwLHRFrIA/7snmlMc9ZfN6Nxa5vUSC7Pg.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.27.129' (ECDSA) to the list of known hosts.
root@192.168.27.129's password:
[root@rk356x:~]#
```

* scp

Execute the command on the PC to copy the file to the device:

```bash
~$ scp ./file root@192.168.27.129:/
root@192.168.27.129's password:
file                            100% 154 0.2KB/s 00:00
```

## External storage device

Buildroot supports automatic mounting of external storage devices:

U disk mount path: `/udisk`

TF card mounting path: `/sdcard`

## Reset

Buildroot supports restoring factory settings. 

**Note: This factory setting means that it is restored to the initial state after the last firmware upgrade of the device.**

* Desktop operation

Factory Settings can be restored in qsetting.

![](../../../rv1126b_img/linux_buildroot_factory_reset.png)

Enter qsetting and click Factory Reset, then click OK button to execute.

* Command operation

You can also run `update` directly or add the `factory/reset` parameter to enter recovery to restore factory settings.

```bash
update
# Or
update factory / update reset
```

## Upgrade firmware

Buildroot supports upgrading firmware from external storage devices. The following is the upgrade process description.

### Make upgrade firmware

Follow the normal firmware compilation process to make the firmware for upgrade. [Build Buildroot Firmware](linux_buildroot_compile.md)

It is not necessary to upgrade the firmware in all partitions. You can modify the `package-file` file to remove the partitions that are not to be upgraded, which can reduce the size of the upgrade package.

For example, modify the file `tools/linux/Linux_Pack_Firmware/rockdev/rk356x-package-file`, change the relative path of `rootfs` to `RESERVED`, so that the root filesystem will not be packaged, that is, the root filesystem partition will not be upgraded.

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

Copy the prepared upgrade firmware to the U disk, TF card, or the `/userdata/` directory of the device, and rename it to `update.img`.

**Note:** If the upgrade firmware is placed in the `/userdata/` directory of the device, do not package `userdata.img`, and change `image/userdata.img` to `RESERVED`.

### Upgrade

* Desktop operation

Upgrade can be performed via qsetting.

![](../../../rv1126b_img/linux_buildroot_update.png)

After putting update firmware into the right place, enter qsetting and click Update then click OK button to execute.

* Command operation

You can also update the device via commands. Start the device, execute the upgrade command on the command line, `update ota path/to/update.img`, the device will enter recovery to upgrade.

```bash
# U Disk
update ota /udisk/update.img
# TF card
update ota /sdcard/update.img
# /userdata/
update ota /userdata/udpate.img
```

Wait for the upgrade to complete. After the upgrade is successful, the device will reboot into the system.

## FirmwareOta

Firmwareota is a Qt desktop application that supports downloading firmware from the Internet for upgrades. The following are instructions for use.

### Add desktop application

* Download

[Download link](https://drive.google.com/drive/folders/1Lc9azHpsBLWH82sIcdLgLMIdWGofaCkr?usp=sharing)

* Unzip

```bash
tar -zxvf firmwareota.tar.gz
```

* According to the directory, copy the files to the device

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

* Restart desktop

```bash
/etc/init.d/S50launcher stop
/etc/init.d/S50launcher start
```

* Start up

After the setting is complete, you can start the application on the desktop.

![](../../../rv1126b_img/linux_buildroot_firmwareota.png)

### Set download address

The program will download the list file `ota_list` through the default Http address, and parse the list file to obtain the firmware download link.

List file format, for example:

```bash
firmware_name_1
http://192.168.2.11:8000/firmware_name_1
firmware_name_2
http://192.168.2.11:8000/firmware_name_2
...
```

We can add the file `/userdata/list_url` to set the download address of the specified list file, for example:

```bash
http://192.168.2.11:8000/ota_list
```

## Weston

We can customize the display by configuring Weston, some of the settings are explained below.

### Status Bar

Weston supports setting the background color and position of the status bar in the shell section of the weston.ini configuration file, as well as setting quick launch programs in the launcher section, such as:

```ini
# /etc/xdg/weston/weston.ini

[shell]
# The color format is ARGB8888
panel-color=0xff002244
# top|bottom|left|right|none
panel-position=bottom

[launcher]
icon=/usr/share/weston/terminal.png
path=/usr/bin/weston-terminal

[launcher]
# Icon path
icon=/usr/share/weston/icon_flower.png
# Quick start command
path=/usr/bin/qsetting
```

### Background

Weston supports setting background patterns and colors in the shell section of the weston.ini configuration file, such as:

```ini
# /etc/xdg/weston/weston.ini

[shell]
# Background pattern (wallpaper) absolute path
background-image=/usr/share/weston/background.png
# scale|scale-crop|tile
background-type=scale
# The color format is ARGB8888, which will take effect when the background pattern is not set
background-color=0xff002244
```

### Standby and lock screen

Weston's timeout standby time can be configured in the startup parameters or in the core section of weston.ini, such as:

```bash
# /etc/init.d/S50launcher
     start)
         ...
         # 0 is to prohibit standby, the unit is second
         weston --tty=2 -B=drm-backend.so --idle-time=0&
```

or:

```ini
# /etc/xdg/weston/weston.ini

[core]
# Set to enter the standby state after 5 seconds of inactivity
idle-time=5
```

### Display color format

The current default display format of Weston in Buildroot SDK is ARGB8888. For some low-performance platforms, it can be configured as RGB565 in the core section of weston.ini, such as:

```ini
# /etc/xdg/weston/weston.ini

[core]
# xrgb8888|rgb565|xrgb2101010
gbm-format=rgb565
```

You can also configure the display format of each screen separately in the output section of weston.ini, such as:

```ini
# /etc/xdg/weston/weston.ini

[output]
# The name of output can be viewed /sys/class/drm/card0-name
name=LVDS-1
# xrgb8888|rgb565|xrgb2101010
gbm-format=rgb565
```

### Screen orientation

Weston's screen display direction can be configured in the output section of weston.ini, such as:

```ini
# /etc/xdg/weston/weston.ini

[output]
name=LVDS-1
# normal|90|180|270|flipped|flipped-90|flipped-180|flipped-270
transform=180
```

If you need to dynamically configure the screen orientation, you can use a dynamic configuration file, such as:

```bash
echo "output:all:rotate90"> /tmp/.weston_drm.conf # All screens are rotated 90 degrees
echo "output:eDP-1:rotate180"> /tmp/.weston_drm.conf # eDP-1 rotate 180 degrees
```

### Resolution and scaling

The screen resolution and scaling of Weston can be configured in the output section of weston.ini, such as:

```ini
# /etc/xdg/weston/weston.ini

[output]
name=HDMI-A-1
# Need to be an effective resolution supported by the screen
mode=1920x1080
# Must be an integer multiple
scale=2
```

If you need to dynamically configure resolution and scaling, you can use dynamic configuration files, such as:

```bash
echo "output:HDMI-A-1:mode=800x600"> /tmp/.weston_drm.conf # Modify the resolution of HDMI-A-1 to 800x600
```

You need to rely on RGA acceleration when scaling in this way.

### Freeze screen

When Weston was started, there was a black screen for a short period switching between the boot logo and the UI display. If you need to prevent a black screen, you can temporarily freeze the Weston screen content through the following dynamic configuration file methods:

```bash
# /etc/init.d/S50launcher
     start)
         ...
         export WESTON_FREEZE_DISPLAY=/tmp/.weston_freeze # Set the path of the special configuration file
         touch /tmp/.weston_freeze # Freeze display
         weston --tty=2 -B=drm-backend.so --idle-time=0&
         ...
         sleep 1 && rm /tmp/.weston_freeze& # Thaw in 1 second
```

### Multi-screen

The Weston of the Buildroot SDK supports functions such as multi-screen same-different display and hot-plugging. The distinction between different display screens is based on the drm name (obtained via /sys/class/drm/card0-name), and the relevant configuration is set through environment variables, such as:

```bash
# /etc/init.d/S50launcher

    start)
        ...
        export WESTON_DRM_PRIMARY=HDMI-A-1 # Specify the main display as HDMI-A-1
        export WESTON_DRM_MIRROR=1 # Use mirror mode (multiple screens at the same display), if you don’t set this environment variable, it will be a different display
        export WESTON_DRM_KEEP_RATIO=1 # The zoom maintains the aspect ratio in the mirror mode. If this variable is not set, the full screen is forced
        export WESTON_DRM_PREFER_EXTERNAL=1 # The built-in display is automatically turned off when the external display is connected
        export WESTON_DRM_PREFER_EXTERNAL_DUAL=1 # When the external display is connected, the first external display is the main display by default
        weston --tty=2 -B=drm-backend.so --idle-time=0&
```

When zooming the display content in the mirror mode, you need to rely on RGA acceleration.

At the same time, it is also supported to individually disable specific screens in the output section of weston.ini:

```ini
# /etc/xdg/weston/weston.ini

[output]
name=LVDS-1
mode=off
# off|current|preferred|<WIDTHxHEIGHT@RATE>
```

### Input device related

Weston service requires at least one input device by default. If there is no input device, special settings in the core section of weston.ini are required:

```ini
# /etc/xdg/weston/weston.ini

[core]
require-input=false
```