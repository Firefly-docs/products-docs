# 更新固件

## 升级前准备

PC 系统要求：Ubuntu22.04, 需要支持 NFS 服务，且在升级过程中可能会遇到一些命令缺失，如：`sshpass` 等命令，需用户自行安装。


## 让设备进入 Recovery 模式

设备断电，用细小的顶针或者是牙签，通过耳机口顶进去，会按压到 Recovery 按键，在按下 Recovery 按键的同时，设备上电，并保持 2s，此后松开即可。


用 type-c 线连接设备和电脑，假设设备成功进入 Recovery 模式，是可以在 Linux  看到设备的：

```
$ lsusb
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 003: ID 0b05:19af ASUSTek Computer, Inc. AURA LED Controller
Bus 001 Device 004: ID 05e3:0608 Genesys Logic, Inc. Hub
Bus 001 Device 022: ID 0403:6001 Future Technology Devices International, Ltd FT232 Serial (UART) IC
Bus 001 Device 023: ID 093a:2510 Pixart Imaging, Inc. Optical Mouse
Bus 001 Device 030: ID 0955:7523 NVIDIA Corp. APX
Bus 001 Device 040: ID 046d:c31c Logitech, Inc. Keyboard K120
Bus 001 Device 086: ID 067b:2731 Prolific Technology, Inc. USB SD Card Reader     
Bus 001 Device 119: ID 067b:23a3 Prolific Technology, Inc. ATEN Serial Bridge
Bus 001 Device 120: ID 2109:2817 VIA Labs, Inc. USB2.0 Hub             
Bus 001 Device 121: ID 10c4:ea60 Silicon Labs CP210x UART Bridge
Bus 001 Device 123: ID 10c4:ea60 Silicon Labs CP210x UART Bridge
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
```

其中`0955:7523 NVIDIA Corp. APX` 表示电脑检测到已经进入 Recovery 模式的 NVIDIA 设备。

注意：如果没有发现 `0955:7523 NVIDIA Corp. APX`，需要通过板端的拨码开关(TYPE C-SEL)去切换到OTG，整机需要拆机。


* `0955:7523 NVIDIA Corp. APX` : Jetson Orin Nano 8GB
* `0955:7423 NVIDIA Corp. APX` : Jetson Orin NX 8GB
* `0955:7323 NVIDIA Corp. APX` : Jetson Orin NX 16GB

## R36.3 (JetPack 6.0)
### 下载固件包

可直接在 Firefly [下载页面](https://www.t-firefly.com/doc/download/266.html)进行下载

下载后，执行 tar 解压：

解压后，进入固件包内进行升级



如果一切顺利，一般升级成功后，会出现以下字段，表示升级成功：
```
Copied 16896 bytes from /mnt/internal/gpt_secondary_3_0.bin to address 0x03ffbe00 in flash
[ 233]: l4t_flash_from_kernel: Successfully flash the qspi
[ 233]: l4t_flash_from_kernel: Flashing success
[ 233]: l4t_flash_from_kernel: The device size indicated in the partition layout xml is smaller than the actual size. This utility will try to fix the GPT.
Flash is successful
Reboot device
Cleaning up...
Log is saved to Linux_for_Tegra/initrdlog/flash_1-2_0_20250527-153418.log
```

其中  `Log is saved to Linux_for_Tegra/initrdlog/flash_1-2_0_20250527-153418.log` 表示升级时候的日志，如果你升级不成功，可以查看日志去追查原因,类似的日志路径名为：`mfi_firefly-1688jd4-orin-nano-nx-devkit-super/initrdlog/flash_1-2_0_20250527-153418.log`  


## R36.4 (JetPack 6.2)

* 支持 Super Mode
* Orin NX 电源适配器需要 **12V/5A**
* Orin NX 的底板硬件版本至少是 **V1.1**

### 下载 fireflyFlash.tbz2
[下载地址](https://www.t-firefly.com/doc/download/266.html)
<br>
`固件` --> `Jetson Linux`

### 解压 fireflyFlash.tbz2
```
mkdir fireflyFlash
tar xf fireflyFlash.tbz2 -C fireflyFlash
cd fireflyFlash
sudo ./l4t_flash_prerequisites.sh
```

### 烧录
在 `fireflyFlash` 目录下，执行命令：  `./firefly_flash.sh -d aio-orin`  

<font color=red>注意：烧录完，设备进入桌面后至少 5 分钟才能断电。</font>
