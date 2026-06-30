[Linux_adb_fastboot]: https://www.t-firefly.com/doc/download/124.html#other_615
[RKDevTool]: https://www.t-firefly.com/doc/download/124.html#other_431
[Androidtool_xxx(版本号)]: https://www.t-firefly.com/doc/download/124.html#other_431
[Release_DriverAssistant.zip]: https://www.t-firefly.com/doc/download/124.html#other_432
[Linux_Upgrade_Tool]: https://www.t-firefly.com/doc/download/124.html#other_433
# Windows 上升级固件

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


## Windows 主机烧录固件

* 工具: [RKDevTool]

**<font color=#ff0000 >注意</font>**:不同固件使用的工具版本可能不同,请根据[《使用USB线烧写须知(重要)》]下载对应的版本

### 安装 RK USB 驱动

下载 [Release_DriverAssistant.zip]，解压，然后运行里面的 DriverInstall.exe 。为了所有设备都使用更新的驱动，请先选择`驱动卸载`，然后再选择`驱动安装`。

![](../../../rk356x_img/upgrade_firmware_install_RK_USB.jpg)

### 连接设备

设备进入升级模式的方式如下：



主机应该会提示发现新硬件并配置驱动。打开设备管理器，会见到新设备 `Rockusb Device` 出现，如下图。如果没有，则需要返回上一步重新安装驱动。

![](../../../rk356x_img/upgrade_firmware_new_equipment.jpg)

## 烧写固件

下载 [RKDevTool]后解压，运行 `RKDevTool_Release` 目录里面的 `RKDevTool.exe`（注意，如果是 Windows 7/8,需要按鼠标右键，选择以管理员身份运行），如下图：

![](../../../rk356x_img/upgrade_firmware_androidtool.jpg)

### 烧写统一固件 update.img

烧写统一固件 update.img 的步骤如下:

1. 切换至`升级固件`页。
2. 按`固件`按钮，打开要升级的固件文件。升级工具会显示详细的固件信息。
3. 按`升级`按钮开始升级。
4. <font color=#ff0000 >如果升级失败，可以尝试先按`擦除 Flash `按钮来擦除 Flash，然后再升级。</font>

**注意：如果你烧写的固件 laoder 版本与原来的机器的不一致，请在升级固件前先执行`擦除 Flash `。**

![](../../../rk356x_img/upgrade_firmware_erase_flash.jpg)

### 烧写分区映像

烧写分区映像的步骤如下：

1. 切换至`下载镜像`页。
2. 勾选需要烧录的分区，可以多选。
3. 确保映像文件的路径正确，需要的话，点路径右边的空白表格单元格来重新选择。
4. 点击`执行`按钮开始升级，升级结束后设备会自动重启。

![](../../../rk356x_img/upgrade_firmware_androidtool.jpg)


## 常见问题

### 1. 如何强行进入 MaskRom 升级模式

如果板子进入不了 Loader 模式，此时可以尝试强行进入 MaskRom 升级模式。操作方法见[《MaskRom模式》](04-maskrom_mode.md)。


### 2. 烧写失败分析

如果烧写过程中出现Download Boot Fail, 或者烧写过程中出错，如下图所示，通常是由于使用的USB线连接不良、劣质线材，或者电脑USB口驱动能力不足导致的，请更换USB线或者电脑USB端口排查。
![](../../../rk356x_img/upgrade_downloadfail.jpg)

