# 介绍

## 前言

AIBOX-K3出厂默认安装Linux操作系统，如果用户要运行其他操作系统，需要使用对应的固件烧写到主板。

AIBOX-K3有灵活的启动方式。一般情况下，除非硬件损坏，AIBOX-K3 开发板是不会变砖的。

如果在升级过程中出现意外，bootloader 损坏，导致无法重新升级，此时仍可以强制进入烧录模式来修复。


## 固件获取

*	[下载链接](https://www.t-firefly.com/doc/download/376.html)

## 升级方式

AIBOX-K3 支持通过以下方式升级固件：

* [使用USB线缆升级固件](upgrade_firmware.md)<br><br>使用Type-C 数据线将主板连接到电脑上，通过升级工具将固件烧写到主板上。<br><br>

## 启动存储器

AIBOX-K3 从以下的存储器中加载系统：

* UFS 接口
* PCIE 接口

## 启动模式

AIBOX-K3有以下启动模式：

* Normal 模式
* Loader 模式
* 硬件烧录模式

### Normal 模式

Normal 模式就是正常的启动过程，各个组件依次加载，正常进入系统。

### Loader 模式

在 Loader 模式下，bootloader 会进入升级状态，等待主机命令，用于固件升级等。

### 硬件烧录模式

硬件烧录模式用于 bootloader 损坏时的系统修复。

***要强行进入 `硬件烧录模式`，请参阅[《硬件烧录模式》](upgrade_boot_mode_spacemit.md)一章。***