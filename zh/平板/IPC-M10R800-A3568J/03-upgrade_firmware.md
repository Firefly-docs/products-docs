# 使用USB线缆升级固件

## 前言

本文介绍了如何将主机上的固件，通过双公头USB数据线烧录到 IPC-M10R800-A3568J 开发板的存储器中。升级时，需要根据主机操作系统和固件类型来选择合适的升级方式。

## 准备工具
* AIO-3568J 开发板
* 主机
* 良好的双公头USB数据线

## 准备固件
固件可以通过编译SDK获得，也可以通过[资源下载](https://www.t-firefly.com/doc/download/124.html)处下载公版固件（统一固件）。固件文件一般有两种：

* 单个统一固件

    统一固件是由分区表、bootloader、uboot、kernel、system等所有文件打包合并成的单个文件。Firefly正式发布的固件都是采用统一固件格式，升级统一固件将会更新主板上所有分区的数据和分区表，并且擦除主板上所有数据。

* 多个分区镜像

    即各个功能独立的文件，如分区表、bootloader、kernel等，在开发阶段生成。独立分区镜像可以只更新指定的分区，而保持其它分区数据不被破坏，在开发过程中会很方便调试。

>    通过统一固件解包/打包工具，可以把统一固件解包为多个分区镜像，也可以将多个分区镜像合并为一个统一固件。



## 安装烧写工具
### Windows操作系统
* 安装RK USB驱动

下载 [Release_DriverAssistant.zip]()，解压，然后运行里面的 DriverInstall.exe 。为了所有设备都使用更新的驱动，请先选择`驱动卸载`，然后再选择`驱动安装`。
<center>

![](../../../rk356x_img/upgrade_firmware_install_RK_USB.png)
</center>


* 运行AndroidTool的RKDevTool.exe


为避免由下载工具版本引起的烧写问题，推荐使用公版固件压缩包内部打包好的工具进行烧写，例如安卓公版固件压缩包解压后如下：
```shell
XXXX_Android11_HDMI_XXXX
├── XXXX_Android11_HDMI_XXXX.img
├── linux
│   └── Linux_Upgrade_Tool_v1.59.zip
└── windows
    ├── DriverAssitant_v5.1.1.zip
    └── RKDevTool_Release_v2.81.zip
```
也可以单独下载 [AndroidTool]()，解压，运行 `RKDevTool_Release_v2.xx` 目录里面的 `RKDevTool.exe`（注意，如果是 Windows 7/8,需要按鼠标右键，选择以管理员身份运行），如下图：

![](../../../rk356x_img/upgrade_firmware_androidtool_zh.png)

### Linux操作系统
Linux 下无须安装设备驱动
* [Linux_Upgrade_Tool]()工具

下载 [Linux_Upgrade_Tool](), 并按以下方法安装到系统中，方便调用：

```
unzip Linux_Upgrade_Tool_xxxx.zip
cd Linux_UpgradeTool_xxxx
sudo mv upgrade_tool /usr/local/bin
sudo chown root:root /usr/local/bin/upgrade_tool
sudo chmod a+x /usr/local/bin/upgrade_tool
```


* [Linux_adb_fastboot](https://www.t-firefly.com/doc/download/124.html#other_615)工具

下载 [Linux_adb_fastboot](https://www.t-firefly.com/doc/download/124.html#other_615), 并按以下方法安装到系统中，方便调用：

```
sudo mv adb /usr/local/bin
sudo chown root:root /usr/local/bin/adb
sudo chmod a+x /usr/local/bin/adb
```
```
sudo mv fastboot /usr/local/bin
sudo chown root:root /usr/local/bin/fastboot
sudo chmod a+x /usr/local/bin/fastboot
```


## 进入升级模式
通常我们升级固件的模式有两种，分别是Loader模式和MaskRom模式。烧写固件前，我们需要连接好设备，并让板子进入到可升级模式。

### Loader模式
#### 硬件方式进入Loader模式
连接设备并通过**RECOVERY**按键进入Loader升级模式步骤如下：


#### 软件方式进入Loader模式
双公头USB数据线接好后在串口调试终端或adb shell给板子运行以下命令

```shell
reboot loader
```

#### 查看Loader模式
如何确定板子是否进入Loader模式，我们可以通过工具去查看

**Windows操作系统**

通过AndroidTool工具可以看到下方提示`Found One LOADER Device`
![](../../../rk356x_img/upgrade_firmware_androidtool_zh.png)

如果有进行"进入Loader模式"的操作，仍旧没有看到烧写工具提示LOADER，此时可以可以看一下Windows主机是否有提示发现新硬件并配置驱动。打开设备管理器，会见到新设备 `Rockusb Device` 出现，如下图。如果没有，可返回上一步重新[安装驱动](03-upgrade_firmware.html#windows-cao-zuo-xi-tong)。

![](../../../rk356x_img/upgrade_firmware_new_equipment.png)

**Linux操作系统**

运行upgrade_tool后可以看到连接设备中有个`Loader`的提示

```shell
firefly@T-chip:~/severdir/down_firmware$ sudo upgrade_tool
List of rockusb connected
DevNo=1 Vid=0x2207,Pid=0x330c,LocationID=106    Loader
Found 1 rockusb,Select input DevNo,Rescan press <R>,Quit press <Q>:q
```

### MaskRom模式
进入MaskRom模式的方法，请参考[《MaskRom模式》](04-maskrom_mode.md)


## 烧写固件

注意：**Linux SDK v1.2.4a** 及之后版本采用 extboot，烧写内核请使用 extboot.img 取代后文中所有的 boot.img（仅限 Linux，Android 请无视）

如何查看版本：
1. 版本格式为 vx.x.xx，例如 v1.2.4a
1. 固件文件名称中存在版本号(..._vx.x.xx_日期.img)
1. Buildroot 使用`cat /etc/version`获取版本(rk356x_linux_release_日期_vx.x.xx.xml)
1. Ubuntu 使用`ffgo version`获取版本(rk356x_linux_release_日期_vx.x.xx.xml)
1. SDK 中可以在 SDK 根目录通过命令查看：`ls -l .repo/manifests/rk356x_linux_release.xml`
1. 如果上述方法找不到格式为 vx.x.xx 的版本，说明是旧版本，不支持 extboot

**不要将 extboot.img 烧录进旧版本固件!**

除此之外，extboot ubuntu 还支持以安装包的形式更新内核，详情查看[Ubuntu 使用手册](https://wiki.t-firefly.com/zh_CN/Firefly-Linux-Guide/manual_ubuntu.html#linux-headers-he-linux-image)

### windows操作系统

#### 烧写统一固件 update.img

烧写统一固件 update.img 的步骤如下:

1. 切换至`Upgrade Firmware`页。
2. 按`Firmware`按钮，打开要升级的固件文件。升级工具会显示详细的固件信息。
3. 按`Upgrade`按钮开始升级。
4. 如果升级失败，可以尝试使用[切换升级存储器](03-upgrade_firmware_with_flash.html#gu-jian-xia-zai-dao-emmc)里面的方法

#### 烧写分区映像
烧写分区映像的步骤如下：

1. 切换至`Upgrade Firmware`页。
2. 点击设备分区表按钮（Dev Partition）
3. 勾选需要烧录的分区，可以多选。
4. 确保映像文件的路径正确，需要的话，点路径右边的空白表格单元格来重新选择。
5. 点击`Run`按钮开始升级，升级结束后设备会自动重启。

![](../../../rk356x_img/upgrade_firmware_androidtool_zh.png)

### Linux操作系统

#### 烧写统一固件 update.img

```
sudo upgrade_tool uf update.img
```

如果升级失败，可以尝试使用[切换升级存储器](03-upgrade_firmware_with_flash.html#gu-jian-xia-zai-dao-emmc)里面的方法

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


安卓 fastboot 烧写动态分区

```
adb reboot fastboot # 进入bootloader
sudo fastboot flash vendor vendor.img
sudo fastboot flash system system.img
sudo fastboot reboot # 烧写成功后,重启
```



## 常见问题
### 1. 如何强行进入 MaskRom 模式

如果板子进入不了 Loader 模式，此时可以尝试强行进入 MaskRom 模式。操作方法见[《MaskRom模式》](04-maskrom_mode.md)。


### 2. 烧写失败分析

如果烧写过程中出现Download Boot Fail, 或者烧写过程中出错，如下图所示，通常是由于使用的USB线连接不良、劣质线材，或者电脑USB口驱动能力不足导致的，请更换USB线或者电脑USB端口排查。
![](../../../rk356x_img/upgrade_downloadfail.png)

### 3. 贴有Spi Flash(Nor Flash)，进入 MaskRom 后，烧录异常
如果板子同时贴有 Spi Flash(Nor Flash) 和 eMMC时，当进入 MaskRom 后，需要切换存储设备，操作方法见[《切换升级设备》](03-upgrade_firmware_with_flash.md)。


[Androidtool_xxx(版本号)]: http://www.t-firefly.com/share/index/index/id/2ea171f2235fe841e89734ca5189da8b.
[AndroidTool]: http://www.t-firefly.com/share/index/index/id/2ea171f2235fe841e89734ca5189da8b.html
[Release_DriverAssistant.zip]: http://www.t-firefly.com/share/index/index/id/1f98ebd663ed09a32e9ebf3fa893dfc0.html
[Linux_Upgrade_Tool]: http://www.t-firefly.com/share/index/index/id/f756718dd2bbf82eb405926549e75ef3.html
[Linux_adb_fastboot]: http://www.t-firefly.com/share/index/index/id/c64b7d743d9368de521a6ced87813dc5.html
