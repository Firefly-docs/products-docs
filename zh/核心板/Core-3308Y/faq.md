# FAQ

## 开机异常卡死或重启

可能是电源电流不够，板子的工作电压为 5V，工作电流为 500mA 以上，视具体挂载外设而定。

## 查看内置Codec增益所有状态

* 方法一

```shell
amixer contents
```

* 方法二

```shell
tinymix contents
```

## 音频输出声音太小

### 耳机

查看codec当前左右声道输出增益：

```shell
amixer cget name='DAC HPOUT Left Volume'
amixer cget name='DAC HPOUT Right Volume'
```

根据所需调节前级增益：

```shell
amixer cset name='DAC HPMIX Left Volume' 1
amixer cset name='DAC HPMIX Right Volume' 1
```

根据所需调节后级增益：

```shell
amixer cset name='DAC HPOUT Left Volume' 18
amixer cset name='DAC HPOUT Right Volume' 18
```

调节音量(百分比)：

```shell
amixer cset name='Master Playback Volume' 40
```

### 喇叭

查看codec当前左右声道输出增益：

```shell
amixer cget name='DAC LINEOUT Left Volume'
amixer cget name='DAC LINEOUT Right Volume'
```

根据所需调节前级增益：

```shell
amixer cset name='DAC HPMIX Left Volume' 1
amixer cset name='DAC HPMIX Right Volume' 1
```

根据所需调节后级增益：

```shell
amixer cset name='DAC LINEOUT Left Volume' 3
amixer cset name='DAC LINEOUT Right Volume' 3
```

调节音量(百分比)：

```shell
amixer cset name='Master Playback Volume' 40
```

## 录音

### 查看声卡

```shell
cat /proc/asound/cards
0 [rockchiprk3308b]: rockchip_rk3308 - rockchip,rk3308b-acodec
                     rockchip,rk3308b-acodec
1 [Audio          ]: USB-Audio - AC108 USB Audio
                     XPowers AND ST AC108 USB Audio at usb-ff440000.usb-1.1, full speed
7 [Loopback       ]: Loopback - Loopback
                     Loopback 1
```

* card 0: rk3308b 内置声卡
* card 1: usb 声卡
* card 7: aloop 生成的虚拟声卡

### 内置codec的mic增益调整    

* Group n
  * Group 0: mic1和mic2
  * Group 1: mic3和mic4
  * Group 2: mic5和mic6
  * Group 3: mic7和mic8
* "ADC MIC"前缀：表示调节前级MIC PGA线性放大增益
* "ADC ALC"前缀：表示调节后级ALC线性放大增益

```shell
amixer cset name='ADC MIC Group 0 Left Volume'  3   # mic1,range 0->3
amixer cset name='ADC MIC Group 0 Right Volume' 3   # mic2,range 0->3
amixer cset name='ADC ALC Group 0 Left Volume'  13  # mic1,range 0->31
amixer cset name='ADC ALC Group 0 Right Volume' 13  # mic2,range 0->31
```

采集8通道**声卡0**音频数据：

```shell
arecord -D hw:0,0 -c 8 -r 16000 -f S16_LE test.wav
```

采样率大于16000hz时，录音命令要加上`--period-size=1024 --buffer-size=4096`参数，例如：

```
arecord -D hw:0,0 -c 8 -r 44100 -f S16_LE --period-size=1024 --buffer-size=4096 test.wav
```

### 内置codec的mic录音没声音

* 使用有源MIC
* 使用无源MIC
  * 检查是否有接偏置电压？（偏置电压可使用MICBIAS1或者MICBIAS2） 
  * 参考电路
    <br></br>
    ![](../../../rk3308_img/Core-3308Y/micbias.png)

## SoX - Sound eXchange

提取双声道音频文件中单个声道的数据并作为单声道音频输出

```shell
sox stereo.wav left.wav  remix 1 	#提取左声道音频
sox stereo.wav right.wav remix 2 	#提取右声道音频
```

## 语音识别开发

关于ROC-RK3308B-CC-PLUS(或者ROC-RK3308B-CC)开源主板支持的语音套件，客户如需更深入的业务定制合作，需要与我司或者对应的语音公司进行商务沟通。

## 显示屏

* 支持RGB接口
  * RGB888
  * RGB666
  * RGB565
* 支持 MCU i80接口
* 最大分辨率是720p

## AndroidTool

### 单独烧写分区镜像，分区地址错误

AndroidTool在加载parameter.txt时，会自动根据parameter分配分区地址，所以每次单独烧写分区镜像时，顺便加载parameter.txt，就不需要手动修改分区地址了。  

## parameter.txt

### 单个分区说明

例如：`0x00004800@0x0000A800(boot)`，@符号之前的数值是分区的大小，@符号之后的数值是分区的起始位置，括号里面的字符是分区的名字。所有数值的单位是sector，1个sector为512Bytes。
</br>
</br>
为了性能，每个分区起始位置需要32KB(64 sectors)对齐，大小也需要32KB的整数倍。

## I2C

### i2cdetect检测不到i2c3所挂载的设备  

* 内核dts有没有使能i2c3
* 检查i2c3所使用的pin脚有没有被复用成其他功能    
* 检查i2c3有没有接上拉电压，可参考原理图i2c1
* 检测i2c3所挂载的设备有没有正常供电       

其他i2c同理。

## VAD

### 使用内置codec的mic

* `rockchip,adc-grps-route = <1 0 3 2>;`：其对应的内置codec的mic的0到7通道顺序是`MIC3，MIC4`，`MIC1，MIC2`，`MIC7，MIC8`，`MIC5，MIC6`。
* `rockchip,det-channel = <0>;`:vad检测的通道是0。

检查所使用的dts里`rockchip,det-channel`的通道与`rockchip,adc-grps-route`所对应内置codec的mic通道是否一致？

### 寄存器配置

如果想要比较理想的唤醒效果，需要配置VAD的寄存器，具体可以参考`SDK/docs/soc_internal/RK3308/`目录下的《Rockchip_RK3308_Developer_Guide_Linux_VAD_Register_Configuration_CN.pdf》文档。

## watchdog

### 使用步骤

* kernel所使用的dts里使能wdt    

```shell
&wdt {
	status = "okay";
};
```

* 可执行文件demo    
  * open: 打开设备节点`/dev/watchdog`，从而启动看门狗 
  * ioctl: WDIOC_SETTIMEOUT设置超时时间    
  * write: 喂狗    

具体程序（可以参考互联网上的文章）需要自己编写并编译。   

### 测试方法

* 板子启动后，运行demo文件，会启动看门狗并进行喂狗。
* 把该进程结束掉，就不进行喂狗了，超时后机器就会重启。

## SSH

### 使用方法

* 使能SSH相关选项

  * openssh

  ```shell
  BR2_PACKAGE_OPENSSH=y
  ```

  * 配置登录的账户root和密码

  ```shell
  BR2_TARGET_ENABLE_ROOT_LOGIN=y
  BR2_TARGET_GENERIC_ROOT_PASSWD="123"
  ```

* 确定rootfs是否为可读写

  * 参考[Rootfs 切换为 ext2](buildroot_development.html#rootfs-qie-huan-wei-ext2)章节。

* 修改配置文件

  * 修改板卡里`/etc/ssh/sshd_config`文件

  ```shell
  PermitRootLogin yes
  ```

## USB Camera

### 使能配置

kernel使能config如下：    

```shell
CONFIG_MEDIA_CAMERA_SUPPORT=y
CONFIG_MEDIA_USB_SUPPORT=y
CONFIG_USB_VIDEO_CLASS=y
CONFIG_USB_VIDEO_CLASS_INPUT_EVDEV=y
```

## 写号工具

<font color=red>**注意:**</font>如果开发板进行了eMMC擦除操作，之前写入的数据也会被清除。

### Windows方式

* 安装RKDevInfoWriteTool
  * [下载地址](http://www.t-firefly.com/doc/download/73.html)
* RKDevInfoWriteTool的**设置**里选中"RPMB"
* 根据需要在RKDevInfoWriteTool的**设置**里配置"SN"，"WIFI MAC"，"LAN MAC"，"BT MAC"等
* 开发板进入loader模式
* RKDevInfoWriteTool进行**写入**或者**读取**操作

具体操作可以参考RKDevInfoWriteTool安装目录下的《RKDevInfoWriteTool使用指南》PDF文档。

### Linux方式

开发板自身写号方式

* buildroot使能`BR2_PACKAGE_VENDOR_STORAGE`

* 通过vendor_storage命令进行读写操作    

  * SN

  ```shell
  vendor_storage -w VENDOR_SN_ID -t string -i cad895bedb8ee15f
   vendor_storage -r VENDOR_SN_ID
  ```

  * LAN MAC

  ```shell
  vendor_storage -w VENDOR_LAN_MAC_ID -t string -i AABBCCDDEEFF
   vendor_storage -r VENDOR_LAN_MAC_ID
  ```

 	其他可以根据`vendor_storage -h`提示进行操作。

应用程序如何读取，可以参考`buildroot/package/rockchip/vendor_storage/vendor_storage.c`里的`vendor_storage_read`函数。

### 说明

#### WIFI MAC

##### RTL8188EU

VENDOR_WIFI_MAC_ID的值是给了p2p0的MAC地址，而wlan0的MAC地址是VENDOR_WIFI_MAC_ID的值的第一个字节`&= ~0x2`所得。

##### AP6236

需要做如下修改：

```shell
diff --git a/kernel/drivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd/Makefile b/kernel/drivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd/Makefile
index 2550ff6..ecd4028 100644
--- a/kernel/drivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd/Makefile
+++ b/kernel/drivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd/Makefile
@@ -23,7 +23,7 @@ DHDCFLAGS = -Wall -Wstrict-prototypes -Dlinux -DBCMDRIVER                 \
        -DKEEP_ALIVE -DPKT_FILTER_SUPPORT -DPNO_SUPPORT -DDHDTCPACK_SUPPRESS  \
        -DDHD_DONOT_FORWARD_BCMEVENT_AS_NETWORK_PKT                           \
        -DMULTIPLE_SUPPLICANT -DTSQ_MULTIPLIER -DMFP                          \
-       -DWL_EXT_IAPSTA -DSUPPORT_P2P_GO_PS                                   \
+       -DWL_EXT_IAPSTA -DSUPPORT_P2P_GO_PS -DGET_CUSTOM_MAC_ENABLE           \
        -DENABLE_INSMOD_NO_FW_LOAD -DDHD_UNSUPPORT_IF_CNTS                    \
        -Idrivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd \
        -Idrivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd/include
```

## UAC

### 使用方法

* kernel配置文件使能相关选项

```shell
CONFIG_USB_CONFIGFS_F_UAC1=y
```

or

```shell
CONFIG_USB_CONFIGFS_F_UAC2=y
```

* 确定rootfs是否为可读写
  * 参考[Rootfs 切换为 ext2](buildroot_development.html#rootfs-qie-huan-wei-ext2)章节。

* .usb_config配置

```shell
echo usb_uac1_en > /etc/init.d/.usb_config
```

or

```shell
echo usb_uac2_en > /etc/init.d/.usb_config
```

* 开发板重启

### 测试方法

参考`SDK/docs/driver/USB/`目录下的《Rockchip-Developer-Guide-Linux4.4-USB-Gadget-UAC-CN.pdf》文档。

## 改写MAC地址

ROC-RK3308B-CC-PLUS以太网芯片RTL8152B的MAC地址可通过`/oem/Rtl8152b_mac_tool`下的工具进行改写

* 将`EF8152B.cfg`中的`NODEID`修改成要烧写的MAC地址，`NODEID`的地址要满足在`STARTID`和`ENDID`所构成的范围之间, 这个范围也是可以改的

  ```shell
  NODEID = 00 E0 4C 36 00 01
  STARTID = 00 E0 4C 36 00 01
  ENDID = 00 E0 4C 36 FF FF
  ```

* `/oem/Rtl8152b_mac_tool`目录下执行`./rtunicpg-aarch64 /s `搜索设备, 当前的MAC地址`C2:20:D1:D3:00:9A`是随机生成的，未烧写过MAC地址时，系统每一次重启都不一样

  ```shell
  ./rtunicpg-aarch64 /s
  
  *************************************************************************
  *   RTUNicPG  - EFUSE/EEPROM/FLASH Programming Utility for
  *         RTL8152/RTL8153 Family USB FE/GBE Network Adapter
  *   Version : v2.0.6.9
  * Copyright (C) Realtek Semiconductor Corp. 2017. All Rights Reserved.
  *************************************************************************
  - RTL8152B
  
  0      eth1    VID:0BDA   PID:8152   bcdDevice:2000   C2:20:D1:D3:00:9A   path:1.4
  ```

* 由于只有一个设备，直接运行`./rtunicpg-aarch64 /efuse `进行初次烧写

烧写成功后执行`ifconfig eth1`可以看出MAC地址已被改写成`00:E0:4C:36:00:01`，重启系统再次查看MAC地址依旧不变

后续需要烧写时可采用`./rtunicpg-aarch64  /efuse /nodeid 00E04C360002 `的方式，**因为efuse空间有限，所以烧写MAC地址的次数尽量不要超过10次** 

## 串口波特率9600改为115200

设备树使能了串口后可以通过`stty [-F DEVICE | --file=DEVICE] [-a|--all]`去查看串口的波特率，默认是9600，以下两种方式都可以将波特率改为115200

* 在应用层中修改, 如ttyS3

  ```shell
  stty -F /dev/ttyS3 speed 115200
  ```

* 修改驱动

  ```shell
  diff --git a/kernel/drivers/tty/serial/serial_core.c b/kernel/drivers/tty/serial/serial_core.c
  index 87625fa..39e7d0c 100644
  --- a/kernel/drivers/tty/serial/serial_core.c
  +++ b/kernel/drivers/tty/serial/serial_core.c
  @@ -2394,8 +2394,8 @@ int uart_register_driver(struct uart_driver *drv)
          normal->type            = TTY_DRIVER_TYPE_SERIAL;
          normal->subtype         = SERIAL_TYPE_NORMAL;
          normal->init_termios    = tty_std_termios;
  -       normal->init_termios.c_cflag = B9600 | CS8 | CREAD | HUPCL | CLOCAL;
  -       normal->init_termios.c_ispeed = normal->init_termios.c_ospeed = 9600;
  +       normal->init_termios.c_cflag = B115200 | CS8 | CREAD | HUPCL | CLOCAL;
  +       normal->init_termios.c_ispeed = normal->init_termios.c_ospeed = 115200;
          normal->flags           = TTY_DRIVER_REAL_RAW | TTY_DRIVER_DYNAMIC_DEV;
          normal->driver_state    = drv;
          tty_set_operations(normal, &uart_ops);
  
  ```

应用层修改的方式在系统重启后需要重新设置，而修改驱动的方式则会将所有串口的默认波特率改成115200