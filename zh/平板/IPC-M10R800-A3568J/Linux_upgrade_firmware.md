[Linux_adb_fastboot]: https://www.t-firefly.com/doc/download/124.html#other_615
[RKDevTool]: https://www.t-firefly.com/doc/download/124.html#other_431
[Androidtool_xxx(版本号)]: https://www.t-firefly.com/doc/download/124.html#other_431
[Release_DriverAssistant.zip]: https://www.t-firefly.com/doc/download/124.html#other_432
[Linux_Upgrade_Tool]: https://www.t-firefly.com/doc/download/124.html#other_433# Linux 上升级固件

## 前言

本文介绍了如何将主机上的固件文，通过双公头USB数据线烧录到 IPC-M10R800-A3568J 开发板的闪存中。升级时，需要根据主机操作系统和固件类型来选择合适的升级方式。

## 准备工作

* IPC-M10R800-A3568J 开发板
* 固件
* 主机
* 良好的双公头USB数据线数据线


## 固件文件
固件文件一般有两种：

* 单个统一固件

    统一固件是由分区表、bootloader、uboot、kernel、system等所有文件打包合并成的单个文件。Firefly正式发布的固件都是采用统一固件格式，升级统一固件将会更新主板上所有分区的数据和分区表，并且擦除主板上所有数据。

* 多个分区镜像

    即各个功能独立的文件，如分区表、bootloader、kernel等，在开发阶段生成。独立分区镜像可以只更新指定的分区，而保持其它分区数据不被破坏，在开发过程中会很方便调试。

>    通过统一固件解包/打包工具，可以把统一固件解包为多个分区镜像，也可以将多个分区镜像合并为一个统一固件。


## Linux 主机烧录固件

 Linux 下无须安装设备驱动，参照 Windows 章节连接设备则可。

* 工具:[Linux_Upgrade_Tool]

### 安装烧写工具
* upgrade_tool

    下载 [Linux_Upgrade_Tool], 并按以下方法安装到系统中，方便调用：

    ```
    unzip Linux_Upgrade_Tool_xxxx.zip
    cd Linux_UpgradeTool_xxxx
    sudo mv upgrade_tool /usr/local/bin
    sudo chown root:root /usr/local/bin/upgrade_tool
    sudo chmod a+x /usr/local/bin/upgrade_tool
    ```

* [Linux_adb_fastboot]工具

    下载 [Linux_adb_fastboot], 并按以下方法安装到系统中，方便调用：

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


烧写统一固件 update.img：

```
sudo upgrade_tool uf update.img
```

<font color=#ff0000 >如果升级失败，可以尝试先擦除后再升级。</font>

```
# 擦除 flash 使用 ef 参数需要指定 loader 文件或者对应的 update.img
sudo upgrade_tool ef update.img   #update.img :你需要烧写的 Ubuntu 固件
# 重新烧写
sudo upgrade_tool uf update.img
```

**烧写分区镜像**

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

安卓 fastboot 烧写动态分区:
```
adb reboot fastboot # 进入bootloader
sudo fastboot flash vendor vendor.img
sudo fastboot flash system system.img
sudo fastboot reboot # 烧写成功后,重启
```

如果因 flash 问题导致升级时出错，可以尝试低级格式化、擦除 emmc：
```
sudo upgrade_tool lf update.img	# 低级格式化
sudo upgrade_tool ef update.img	# 擦除
```
## 常见问题

### 1. 如何强行进入 MaskRom 升级模式

如果板子进入不了 Loader 模式，此时可以尝试强行进入 MaskRom 升级模式。操作方法见[《MaskRom 升级模式》](04-maskrom_mode.md)。


### 2. 烧写失败分析

如果烧写过程中出现Download Boot Fail, 或者烧写过程中出错，如下图所示，通常是由于使用的USB线连接不良、劣质线材，或者电脑USB口驱动能力不足导致的，请更换USB线或者电脑USB端口排查。