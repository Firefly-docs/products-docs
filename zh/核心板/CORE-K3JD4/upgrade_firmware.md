# 使用USB线缆升级固件

## 前言

本文介绍了如何将主机上的固件，通过Type-C 数据线烧录到 AIBOX-K3 的存储器中。升级时，需要根据主机操作系统和固件类型来选择合适的升级方式。

刷机前请详细阅读《刷机工具使用手册》。如果没有显示，请刷新下网页。

* [刷机工具使用手册](https://spacemit.com/community/document/info?lang=zh&nodepath=tools/user_guide/flasher_user_guide.md)

## 准备工具
* AIBOX-K3
* [固件](https://www.t-firefly.com/doc/download/376.html)
* 主机
* 良好的Type-C 数据线

## 准备固件

固件可以通过编译SDK获得，也可以通过[资源下载](https://www.t-firefly.com/doc/download/376.html)处下载公版固件（统一固件）。固件文件一般有两种：

* 单个统一固件

    统一固件是由分区表、bootloader、uboot、kernel、system等所有文件打包合并成的单个文件。Firefly正式发布的固件都是采用统一固件格式，升级统一固件将会更新主板上所有分区的数据和分区表，并且擦除主板上所有数据。

* 多个分区镜像

    即各个功能独立的文件，如分区表、bootloader、kernel等，在开发阶段生成。独立分区镜像可以只更新指定的分区，而保持其它分区数据不被破坏，在开发过程中会很方便调试。

>    通过解压统一固件，可以把统一固件解包为多个分区镜像，也可以将多个分区镜像打包合并为一个统一固件。


## 安装烧写工具

### Windows操作系统

1. 下载最新版本的刷机工具 titantools_for_windows.exe
2. 双击已经下载好的刷机工具 titantools_for_windows_last 开始安装；
3. 安装好后运行 titanflasher.exe 


### Linux操作系统

1. 下载最新版本的刷机工具 titantools_for_linux.AppImage
2. 双击已经下载好的刷机工具 titantools_for_linux_last 开始安装；
3. 安装好后运行 titanflasher


![](../../../aibox_img/AIBOX-K3/upgrade_firmware_titanflasher_zh.png)


* [Linux_adb_fastboot](https://www.t-firefly.com/doc/download/324.html)工具

下载 [Linux_adb_fastboot](https://www.t-firefly.com/doc/download/324.html), 并按以下方法安装到系统中，方便调用：

```
sudo mv fastboot /usr/local/bin
sudo chown root:root /usr/local/bin/fastboot
sudo chmod a+x /usr/local/bin/fastboot
```

## 进入升级模式

通常我们升级固件的模式有两种，分别是 Loader 模式和硬件烧录模式。烧写固件前，我们需要连接好设备，并让板子进入到可升级模式。

### Loader模式

#### 软件方式进入Loader模式

Type-C 数据线接好后在串口调试终端给板子运行以下命令。不能使用 adb 工具进入到 loader 模式，因为软件进入 loader 模式需要操作 uboot 终端。

```shell
reboot
```

板子系统重启到 uboot 阶段后按住 s 键进入到 uboot 调试终端。进入到 uboot 调试终端后，删除多余的 s 字符输入信息。执行 fastboot 命令进入到烧写模式
```bash
fastboot 0
```

在 PC 端的 titanflasher 烧录工具上点击：研发工具 --> 单机烧录 --> 扫描设备 --> 选择刷机文件 --> 开始刷机。

注意：此时 titanflasher 工具会烧写一部分必要的固件进板子，但不会烧入全部固件。还需要在板子的 uboot 终端再次按下 Enter 按键进行后续的固件烧录。

### 硬件烧录模式

进入硬件烧录模式的方法，请参考[《硬件烧录模式》](upgrade_boot_mode_spacemit.md)


## 烧写固件

#### 烧写统一固件

烧写压缩包统一固件的步骤如下:

点击步骤：研发工具 --> 单机烧录 --> 扫描设备 --> 本地文件 --> 选择刷机文件 --> 刷完自启动 --> 开始刷机。

![](../../../aibox_img/AIBOX-K3/upgrade_firmware_zh.png)

#### 烧写分区映像

点击步骤：研发工具 --> 单机烧录 --> 扫描设备 --> 本地文件 --> 选择刷机文件 --> 刷完自启动 --> 配置分区文件 --> 开始刷机。

配置分区文件需要选择分区文件。

1. partition_4M.json 是更新核心板上的 nor flash 。
2. partition_universal.json 是更新核心板上的 UFS 分区映像。

![](../../../aibox_img/AIBOX-K3/upgrade_firmware_part_zh.png)

## 常见问题

### 1. 如何强行进入硬件烧录模式

如果板子进入不了 Loader 模式，此时可以尝试强行进入硬件烧录模式。操作方法见[《硬件烧录模式》](upgrade_boot_mode_spacemit.md)。

### 2. 烧写失败分析

如果烧写过程中出错，通常是由于使用的USB线连接不良、劣质线材，或者电脑USB口驱动能力不足导致的，请更换USB线或者电脑USB端口排查。

[Androidtool_xxx(版本号)]: http://www.t-firefly.com/share/index/index/id/2ea171f2235fe841e89734ca5189da8b.
[AndroidTool]: http://www.t-firefly.com/share/index/index/id/2ea171f2235fe841e89734ca5189da8b.html
[Release_DriverAssistant.zip]: http://www.t-firefly.com/share/index/index/id/1f98ebd663ed09a32e9ebf3fa893dfc0.html
[Linux_Upgrade_Tool]: http://www.t-firefly.com/share/index/index/id/f756718dd2bbf82eb405926549e75ef3.html
[Linux_adb_fastboot]: http://www.t-firefly.com/share/index/index/id/c64b7d743d9368de521a6ced87813dc5.html
