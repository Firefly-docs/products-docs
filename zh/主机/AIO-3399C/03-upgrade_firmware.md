# 升级固件

## 前言

本文介绍了如何将主机上的固件文，通过Type-C数据线烧录到 AIO-3399C 开发板的闪存中。升级时，需要根据主机操作系统和固件类型来选择合适的升级方式。

## 准备工作

* AIO-3399C 开发板
* 固件
* 主机
* 良好的Type-C数据线数据线


## 固件文件
固件文件一般有两种：

* 单个统一固件

    统一固件是由分区表、bootloader、uboot、kernel、system等所有文件打包合并成的单个文件。Firefly正式发布的固件都是采用统一固件格式，升级统一固件将会更新主板上所有分区的数据和分区表，并且擦除主板上所有数据。

* 多个分区镜像

    即各个功能独立的文件，如分区表、bootloader、kernel等，在开发阶段生成。独立分区镜像可以只更新指定的分区，而保持其它分区数据不被破坏，在开发过程中会很方便调试。

>    通过统一固件解包/打包工具，可以把统一固件解包为多个分区镜像，也可以将多个分区镜像合并为一个统一固件。


## Windows 主机烧录固件

* 工具: [Androidtool_xxx(版本号)]

**<font color=#ff0000 >注意</font>**:不同固件使用的工具版本可能不同,请根据[《使用USB线烧写须知(重要)》]下载对应的版本

### 安装 RK USB 驱动

下载 [Release_DriverAssistant.zip]，解压，然后运行里面的 DriverInstall.exe 。为了所有设备都使用更新的驱动，请先选择`驱动卸载`，然后再选择`驱动安装`。

![](../../../rk3399_img/upgrade_firmware_install_RK_USB.jpg)

### 连接设备

设备进入升级模式的方式如下：

* 先断开电源适配器连接：

   * USB 一端连接主机，Type-C 一端连接开发板 Type-C 母口
   * 按住设备上的 RECOVERY （恢复）键并保持
   * 接上电源
   * 大约两秒钟后，松开 RECOVERY 键

主机应该会提示发现新硬件并配置驱动。打开设备管理器，会见到新设备 `Rockusb Device` 出现，如下图。如果没有，则需要返回上一步重新安装驱动。

![](../../../rk3399_img/upgrade_firmware_new_equipment.jpg)

## 烧写固件

下载 [RKDevTool](**若系统是 Android8.1 则需要 2.54 以上版本**)，解压，运行 `RKDevTool_Release_v2.38` 目录里面的 `RKDevTool.exe`（注意，如果是 Windows 7/8,需要按鼠标右键，选择以管理员身份运行），如下图：

![](../../../rk3399_img/upgrade_firmware_androidtool.jpg)

### 烧写统一固件 update.img

烧写统一固件 update.img 的步骤如下:

1. 切换至`升级固件`页。
2. 按`固件`按钮，打开要升级的固件文件。升级工具会显示详细的固件信息。
3. 按`升级`按钮开始升级。
4. <font color=#ff0000 >如果升级失败，可以尝试先按`擦除 Flash `按钮来擦除 Flash，然后再升级。一定要根据[《使用USB线烧写须知(重要)》]进行擦除烧写</font>

**注意：如果你烧写的固件 laoder 版本与原来的机器的不一致，请在升级固件前先执行`擦除 Flash `。**

![](../../../rk3399_img/upgrade_firmware_erase_flash.jpg)

### 烧写分区映像

每个固件的分区可能不相同,请注意以下两点:
1. 使用 `Androidtool_2.38` 烧写 `ubuntu(MBR)` 和 `Android7.1` 固件时使用默认配置即可;
2. 使用 `Androidtool_2.58` 烧写 `ubuntu(GPT)` 使用默认配置即可，烧写 `Android8.1` 固件请先执行以下操作:
<font color=#ff0000 >切换至`下载镜像页面`; 右键点击表格，选择`导入配置`; 选择rk3399-Android81.cfg</font>
3. 使用`Androidtool_2.71`烧写`Android10`或`Android9`固件时使用默认配置即可;

烧写分区映像的步骤如下：

1. 切换至`下载镜像`页。
2. 勾选需要烧录的分区，可以多选。
3. 确保映像文件的路径正确，需要的话，点路径右边的空白表格单元格来重新选择。
4. 点击`执行`按钮开始升级，升级结束后设备会自动重启。

![](../../../rk3399_img/upgrade_firmware_androidtool.jpg)

## Linux 主机烧录固件

 Linux 下无须安装设备驱动，参照 Windows 章节连接设备则可。

* 工具:[Linux_Upgrade_Tool]

**<font color=#ff0000 >注意</font>**:不同固件使用的工具版本可能不同,请根据[烧写须知]下载对应的版本

### upgrade_tool

下载 [Linux_Upgrade_Tool] (**系统是 Android8.1则需要 Linux_Upgrde_Tool_for_android8.1**), 并按以下方法安装到系统中，方便调用：

```
unzip Linux_Upgrade_Tool_xxxx.zip
cd Linux_UpgradeTool_xxxx
sudo mv upgrade_tool /usr/local/bin
sudo chown root:root /usr/local/bin/upgrade_tool
sudo chmod a+x /usr/local/bin/upgrade_tool
```

烧写统一固件 update.img：

```
sudo upgrade_tool uf update.img
```

<font color=#ff0000 >如果升级失败，可以尝试先擦除后再升级。一定要根据[《使用USB线烧写须知(重要)》]的表格进行擦除烧写</font>

```
# 擦除 flash 使用 ef 参数需要指定 loader 文件或者对应的 update.img
sudo upgrade_tool ef update.img   #update.img :你需要烧写的 Ubuntu 固件
# 重新烧写
sudo upgrade_tool uf update.img
```

**烧写分区镜像**

Android7.1、Android8.1使用以下方式:
```
sudo upgrade_tool di -b boot.img
sudo upgrade_tool di -k kernel.img
sudo upgrade_tool di -s system.img
sudo upgrade_tool di -r recovery.img
sudo upgrade_tool di -m misc.img
sudo upgrade_tool di -re resource.img
sudo upgrade_tool di -p paramater
sudo upgrade_tool ul bootloader.bin
```

Android9.0、Android10.0使用以下方式:
```
sudo upgrade_tool di -b boot.img
sudo upgrade_tool di -dtbo dtbo.img
sudo upgrade_tool di -misc misc.img
sudo upgrade_tool di -parameter parameter.txt
sudo upgrade_tool di -r recovery.img
sudo upgrade_tool di -super super.img
sudo upgrade_tool di -trust trust.img
sudo upgrade_tool di -uboot uboot.img
sudo upgrade_tool di -vbmeta vbmeta.img
```

Ubuntu(GPT)使用以下方式:
```
sudo upgrade_tool ul $LOADER
sudo upgrade_tool di -p $PARAMETER
sudo upgrade_tool di -uboot $UBOOT
sudo upgrade_tool di -trust $TRUST
sudo upgrade_tool di -boot $BOOT
sudo upgrade_tool di -recovery $RECOVERY
sudo upgrade_tool di -misc $MISC
sudo upgrade_tool di -oem $OEM
sudo upgrade_tool di -userdata $USERDATA
sudo upgrade_tool di -rootfs $ROOTFS
```


如果因 flash 问题导致升级时出错，可以尝试低级格式化、擦除 nand flash：
```
sudo upgrade_tool lf update.img	# 低级格式化
sudo upgrade_tool ef update.img	# 擦除
```
## 常见问题

### 1. 如何强行进入 MaskRom 升级模式

如果板子进入不了 Loader 模式，此时可以尝试强行进入 MaskRom 升级模式。操作方法见[《MaskRom 升级模式》](04-maskrom_mode.md)。


### 2. 烧写失败分析

如果烧写过程中出现Download Boot Fail, 或者烧写过程中出错，如下图所示，通常是由于使用的USB线连接不良、劣质线材，或者电脑USB口驱动能力不足导致的，请更换USB线或者电脑USB端口排查。
![](../../../rk3399_img/upgrade_downloadfail.jpg)

[《使用USB线烧写须知(重要)》]: 02-upgrade_table.md


[RKDevTool]: http://www.t-firefly.com/doc/download/page/id/54.html#other_248
[Androidtool_xxx(版本号)]: http://www.t-firefly.com/doc/download/page/id/54.html#other_248
[Release_DriverAssistant.zip]: http://www.t-firefly.com/doc/download/page/id/54.html#windows_11
[Linux_Upgrade_Tool]: http://www.t-firefly.com/doc/download/page/id/54.html#windows_375