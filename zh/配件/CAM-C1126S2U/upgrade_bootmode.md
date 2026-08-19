# 介绍

## 前言

CAM-C1126S2U出厂默认安装Linux操作系统，如果用户要运行其他操作系统，需要使用对应的固件烧写到主板。

CAM-C1126S2U有灵活的启动方式。一般情况下，除非硬件损坏，CAM-C1126S2U 开发板是不会变砖的。

如果在升级过程中出现意外，bootloader 损坏，导致无法重新升级，此时仍可以进入 `MaskRom` 模式来修复。


## 固件获取
*	[下载链接](https://www.t-firefly.com/doc/download/99.html)

## 升级方式
CAM-C1126S2U 支持通过以下两种方式升级固件：

* [使用USB线缆升级固件](upgrade_firmware.md)<br><br>使用CAM-C1126S2U 将主板连接到电脑上，通过升级工具将固件烧写到主板上。<br><br>


## 启动存储器

CAM-C1126S2U 从以下的存储器中加载系统：

* eMMC 接口

## 启动模式

CAM-C1126S2U有三种启动模式：

* Normal 模式
* Loader 模式
* MaskRom 模式

### Normal 模式

Normal 模式就是正常的启动过程，各个组件依次加载，正常进入系统。


### Loader 模式

在 Loader 模式下，bootloader 会进入升级状态，等待主机命令，用于固件升级等。

### MaskRom 模式

MaskRom 模式用于 bootloader 损坏时的系统修复。

一般情况下是不用进入 `MaskRom 模式`的，只有在 bootloader 校验失败（读取不了 IDB 块，或 bootloader 损坏） 的情况下，BootRom 代码 就会进入 `MaskRom 模式`。此时 BootRom 代码等待主机通过 USB 接口传送 bootloader 代码，加载并运行之。

***要强行进入 `MaskRom 模式`，请参阅[《MaskRom模式》](upgrade_maskrom_mode.md)一章。***