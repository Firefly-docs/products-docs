# 升级固件

## 前言

本文介绍了如何将主机上的固件，通过双公头 USB 数据线烧录到 AIO-3288J 开发板的闪存中。升级时，需要根据主机操作系统和固件类型来选择合适的升级方式。

## 准备工作

* AIO-3288J 开发板
* 固件
* 主机
* 良好的双公头 USB 数据线

## 固件文件

固件文件一般有两种：

* 单个统一固件

    统一固件是由分区表、bootloader、uboot、kernel、system 等所有文件打包合并成的单个文件。Firefly 正式发布的固件都是采用统一固件格式，升级统一固件将会更新主板上所有分区的数据和分区表，并且擦除主板上所有数据。

* 多个分区镜像

    即各个功能独立的文件，如分区表、bootloader、kernel 等，在开发阶段生成。独立分区镜像可以只更新指定的分区，而保持其它分区数据不被破坏，在开发过程中会很方便调试。

> 通过统一固件解包/打包工具，可以把统一固件解包为多个分区镜像，也可以将多个分区镜像合并为一个统一固件。

## Windows

* 工具: [Androidtool_xxx(版本号)](http://www.t-firefly.com/doc/download/16.html#windows_22)

**<font color=#ff0000 >注意</font>**：不同固件使用的工具版本可能不同,请根据[《烧写须知》]下载对应的版本。

### 安装 RK USB 驱动

下载 [Release_DriverAssistant.zip](http://www.t-firefly.com/doc/download/16.html#other_23)，解压，然后运行里面的 DriverInstall.exe。为了所有设备都使用更新的驱动，请先选择 `驱动卸载`，然后再选择 `驱动安装`。

![](../../../rk3288_img/upgrade_firmware_install_RK_USB.jpg)

### 连接设备

设备进入升级模式的方式如下：

!INCLUDE "upgrade_firmware_enter_loader.mdpp"

AIO-3288J：

![](../../../rk3288_img/AIO-3288J/download_otg.png)


![](../../../rk3288_img/AIO-3288J/download_otg_1.png)



主机应该会提示发现新硬件并配置驱动。打开设备管理器，会见到新设备 `Rockusb Device` 出现，如下图。如果没有，则需要返回上一步重新安装驱动。

![](../../../rk3288_img/upgrade_firmware_new_equipment.jpg)

## 烧写固件

下载 [AndroidTool](http://www.t-firefly.com/doc/download/16.html#windows_22)，解压，运行 `AndroidTool_Release_vxx` 目录里面的 `AndroidTool.exe`（注意，如果是 Windows 7/8,需要按鼠标右键，选择以管理员身份运行），如下图：

![](../../../rk3288_img/upgrade_firmware_androidtool.jpg)

### 烧写统一固件 update.img

烧写统一固件 update.img 的步骤如下:

1. 切换至 `升级固件` 页。
2. 按 `固件` 按钮，打开要升级的固件文件。升级工具会显示详细的固件信息。
3. 按 `升级` 按钮开始升级。
4. <font color=#ff0000 >如果升级失败，可以尝试先按 `擦除 Flash` 按钮来擦除 Flash，然后再升级。一定要根据[《烧写须知》]进行擦除烧写</font>。

**注意：如果你烧写的固件 loader 版本与原来的机器的不一致，请在升级固件前先执行`擦除 Flash`。**

![](../../../rk3288_img/upgrade_firmware_erase_flash.jpg)

### 烧写分区映像

每个固件的分区可能不相同，请确保分区的地址信息和 `parameter` 文件保持一致。

烧写分区映像的步骤如下：

1. 切换至 `下载镜像` 页。
2. 勾选需要烧录的分区，可以多选。
3. 确保映像文件的路径正确，需要的话，点路径右边的空白表格单元格来重新选择。
4. 点击 `执行` 按钮开始升级，升级结束后设备会自动重启。

![](../../../rk3288_img/upgrade_firmware_androidtool.jpg)

## Linux

Linux 下无须安装设备驱动，参照 Windows 章节连接设备则可。

* 工具：[upgrade_tool_xxx(版本号)](http://www.t-firefly.com/doc/download/16.html#linux_22)

**<font color=#ff0000 >注意</font>**:不同固件使用的工具版本可能不同，请根据[《烧写须知》]下载对应的版本。

### upgrade_tool

下载 Linux_Upgrade_Tool，并按以下方法安装到系统中，方便调用：

```bash
unzip Linux_Upgrade_Tool_xxxx.zip
cd Linux_UpgradeTool_xxxx
sudo mv upgrade_tool /usr/local/bin
sudo chown root:root /usr/local/bin/upgrade_tool
sudo chmod a+x /usr/local/bin/upgrade_tool
```

烧写统一固件 update.img：

```bash
sudo upgrade_tool uf update.img
```

<font color=#ff0000 >如果升级失败，可以尝试先擦除后再升级。一定要根据[《烧写须知》]的表格进行擦除烧写</font>。

```bash
# 擦除 flash 使用 ef 参数需要指定 loader 文件或者对应的 update.img
sudo upgrade_tool ef update.img   #update.img：你需要烧写的固件
# 重新烧写
sudo upgrade_tool uf update.img
```

烧写分区镜像：

Linux(MBR)、Android5.1，使用以下方式：

```bash
sudo upgrade_tool di -b /path/to/boot.img
sudo upgrade_tool di -k /path/to/kernel.img
sudo upgrade_tool di -s /path/to/system.img
sudo upgrade_tool di -r /path/to/recovery.img
sudo upgrade_tool di -m /path/to/misc.img
sudo upgrade_tool di -re /path/to/resource.img
sudo upgrade_tool di -p paramater   #烧写 parameter
sudo upgrade_tool ul bootloader.bin # 烧写 bootloader
```

Linux(GPT)，使用以下方式：

```bash
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

```bash
sudo upgrade_tool lf update.img # 低级格式化
sudo upgrade_tool ef update.img # 擦除
```

## 常见问题

### 1. 如何强行进入 MaskRom 模式

如果板子进入不了 Loader 模式，此时可以尝试强行进入 MaskRom 模式。操作方法见[《MaskRom 模式》](maskrom_mode.md)。

### 2. 烧写失败分析

如果烧写过程中出现 Download Boot Fail, 或者烧写过程中出错，如下图所示，通常是由于使用的 USB 线连接不良、劣质线材，或者电脑 USB 口驱动能力不足导致的，请更换 USB 线或者电脑 USB 端口排查。

![](../../../rk3288_img/upgrade_downloadfail.jpg)

[《烧写须知》]: upgrade_table.md