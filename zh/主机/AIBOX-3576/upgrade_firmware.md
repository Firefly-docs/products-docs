# 使用USB线缆升级固件

## 前言

本文介绍了如何将主机上的固件，通过Type-C 数据线烧录到 AIBOX-3576 的存储器中。升级时，需要根据主机操作系统和固件类型来选择合适的升级方式。

## 准备工具
* AIBOX-3576
* [固件](https://www.t-firefly.com/doc/download/268.html)
* 主机
* 良好的Type-C 数据线

## 准备固件
固件可以通过编译SDK获得，也可以通过[资源下载](https://www.t-firefly.com/doc/download/268.html)处下载公版固件（统一固件）。固件文件一般有两种：

* 单个统一固件

    统一固件是由分区表、bootloader、uboot、kernel、system等所有文件打包合并成的单个文件。Firefly正式发布的固件都是采用统一固件格式，升级统一固件将会更新主板上所有分区的数据和分区表，并且擦除主板上所有数据。

* 多个分区镜像

    即各个功能独立的文件，如分区表、bootloader、kernel等，在开发阶段生成。独立分区镜像可以只更新指定的分区，而保持其它分区数据不被破坏，在开发过程中会很方便调试。

>    通过统一固件解包/打包工具，可以把统一固件解包为多个分区镜像，也可以将多个分区镜像合并为一个统一固件。



## 安装烧写工具
### Windows操作系统
* 安装RK USB驱动

下载 [Release_DriverAssistant.zip](https://www.t-firefly.com/doc/download/268.html#other_572)，解压，然后运行里面的 DriverInstall.exe 。为了所有设备都使用更新的驱动，请先选择`驱动卸载`，然后再选择`驱动安装`。
<center>

![](../../../aibox_img/AIBOX-3576/upgrade_firmware_install_rk_usb.jpg)
</center>


* 运行AndroidTool的RKDevTool.exe


![](../../../aibox_img/AIBOX-3576/upgrade_firmware_androidtool_zh.png)

### Linux操作系统
Linux 下无须安装设备驱动
* [Linux_Upgrade_Tool](https://www.t-firefly.com/doc/download/268.html#other_571)工具

下载 [Linux_Upgrade_Tool](https://www.t-firefly.com/doc/download/268.html#other_571), 并按以下方法安装到系统中，方便调用：

```
unzip Linux_Upgrade_Tool_xxxx.zip
cd Linux_UpgradeTool_xxxx
sudo mv upgrade_tool /usr/local/bin
sudo chown root:root /usr/local/bin/upgrade_tool
sudo chmod a+x /usr/local/bin/upgrade_tool
```

## 进入升级模式
通常我们升级固件的模式有两种，分别是Loader模式和MaskRom模式。烧写固件前，我们需要连接好设备，并让板子进入到可升级模式。

### Loader模式
AIBOX-3576只支持软件方式进入Loader模式

#### 软件方式进入Loader模式
Type-C 数据线接好后在串口调试终端给板子运行以下命令

```shell
reboot loader
```

#### 查看Loader模式
如何确定板子是否进入Loader模式，我们可以通过工具去查看

**Windows操作系统**

通过AndroidTool工具可以看到下方提示`Found One LOADER Device`
![](../../../aibox_img/AIBOX-3576/upgrade_firmware_androidtool_zh.png)

如果有进行"进入Loader模式"的操作，仍旧没有看到烧写工具提示LOADER，此时可以可以看一下Windows主机是否有提示发现新硬件并配置驱动。打开设备管理器，会见到新设备 `Rockusb Device` 出现，如下图。如果没有，可返回上一步重新[安装驱动](upgrade_firmware.html#windows-cao-zuo-xi-tong)。

![](../../../aibox_img/AIBOX-3576/upgrade_firmware_new_equipment.jpg)

**Linux操作系统**

运行upgrade_tool后可以看到连接设备中有个`Loader`的提示

```shell
firefly@T-chip:~/severdir/down_firmware$ sudo upgrade_tool
List of rockusb connected
DevNo=1 Vid=0x2207,Pid=0x330c,LocationID=106    Loader
Found 1 rockusb,Select input DevNo,Rescan press <R>,Quit press <Q>:q
```

### MaskRom模式
进入MaskRom模式的方法，请参考[《MaskRom模式》](upgrade_maskrom_mode_rockchip.md)


## 烧写固件
### windows操作系统

#### 烧写统一固件 update.img

烧写统一固件 update.img 的步骤如下:

1. 切换至`Upgrade Firmware`页。
2. 按`Firmware`按钮，打开要升级的固件文件。升级工具会显示详细的固件信息。
3. 按`Upgrade`按钮开始升级。
4. <font color=#ff0000 >如果升级失败，可以尝试先按`EraseFlash `按钮来擦除 Flash，然后再升级。</font>

![](../../../aibox_img/AIBOX-3576/upgrade_firmware_erase_flash_zh.png)

#### 烧写分区映像
烧写分区映像的步骤如下：

1. 切换至`Upgrade Firmware`页。
2. 勾选需要烧录的分区，可以多选。
3. 确保映像文件的路径正确，需要的话，点路径右边的空白表格单元格来重新选择。
4. 点击`Run`按钮开始升级，升级结束后设备会自动重启。

![](../../../aibox_img/AIBOX-3576/upgrade_firmware_androidtool_zh.png)

### Linux操作系统

#### 烧写统一固件 update.img

```
sudo upgrade_tool uf update.img
```

<font color=#ff0000 >如果升级失败，可以尝试先擦除后再升级。 </font>

```
# 擦除 flash 使用 ef 参数需要指定 loader 文件或者对应的 update.img
sudo upgrade_tool ef update.img   #update.img :你需要烧写的 Ubuntu 固件
# 重新烧写
sudo upgrade_tool uf update.img
```

#### 烧写分区镜像

```
sudo upgrade_tool di -b /path/to/boot.img
sudo upgrade_tool di -r /path/to/recovery.img
sudo upgrade_tool di -m /path/to/misc.img
sudo upgrade_tool di -u /path/to/uboot.img
sudo upgrade_tool di -dtbo /path/to/dtbo.img
sudo upgrade_tool di -p paramater   #烧写 parameter
sudo upgrade_tool ul bootloader.bin # 烧写 bootloader
```


如果因 flash 问题导致升级时出错，可以尝试低级格式化、擦除 emmc：
```
sudo upgrade_tool lf update.img	# 低级格式化
sudo upgrade_tool ef update.img	# 擦除
```



## 常见问题
### 1. 如何强行进入 MaskRom 模式

如果板子进入不了 Loader 模式，此时可以尝试强行进入 MaskRom 模式。操作方法见[《MaskRom模式》](upgrade_maskrom_mode_rockchip.md)。


### 2. 烧写失败分析

如果烧写过程中出现Download Boot Fail, 或者烧写过程中出错，如下图所示，通常是由于使用的USB线连接不良、劣质线材，或者电脑USB口驱动能力不足导致的，请更换USB线或者电脑USB端口排查。
![](../../../aibox_img/AIBOX-3576/upgrade_firmware_download_fail.png)


[Androidtool_xxx(版本号)]: http://www.t-firefly.com/share/index/index/id/2ea171f2235fe841e89734ca5189da8b.
[AndroidTool]: http://www.t-firefly.com/share/index/index/id/2ea171f2235fe841e89734ca5189da8b.html
[Release_DriverAssistant.zip]: http://www.t-firefly.com/share/index/index/id/1f98ebd663ed09a32e9ebf3fa893dfc0.html
[Linux_Upgrade_Tool]: http://www.t-firefly.com/share/index/index/id/f756718dd2bbf82eb405926549e75ef3.html