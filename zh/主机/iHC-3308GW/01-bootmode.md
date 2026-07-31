# 介绍

## 前言

IHC-3308GW出厂默认安装Buildroot操作系统，如果用户要运行其他操作系统，需要使用对应的固件烧写到主板。

IHC-3308GW有灵活的启动方式。一般情况下，除非硬件损坏，IHC-3308GW 开发板是不会变砖的。

如果在升级过程中出现意外，bootloader 损坏，导致无法重新升级，此时仍可以进入 `MaskRom` 模式来修复。


## 固件获取
*	[下载链接](https://www.t-firefly.com/doc/download/102.html)

## 升级方式
IHC-3308GW 支持通过以下方式升级固件：

-   使用USB线缆升级固件

    使用USB线将主板连接到电脑上，通过升级工具将固件烧写到主板上。


## 启动存储器

IHC-3308GW 从以下的存储器中加载系统：

* eMMC 接口

## 启动模式

IHC-3308GW有三种启动模式：

* Normal 模式
* Loader 模式
* MaskRom 模式

### Normal 模式

Normal 模式就是正常的启动过程，各个组件依次加载，正常进入系统。


### Loader 模式

在 Loader 模式下，bootloader 会进入升级状态，等待主机命令，用于固件升级等。要进入 Loader 模式，必须让 bootloader 在启动时检测到 `RECOVERY`（恢复）键按下，且 USB 处于连接状态。


使设备进入升级模式的方法如下:



一种方式是断开电源适配器

* Type-C 数据线连接好设备和主机。
* 按住设备上的 RECOVERY （恢复）键并保持。
* 插上电源
* 大约两秒钟后，松开 RECOVERY 键。
另一种方式是接上电源适配器

* Type-C 数据线连接好设备和主机。
* 按住设备上的 RECOVERY （恢复）键并保持。
* 短按一下 RESET（复位）键。
* 大约两秒钟后，松开 RECOVERY 键。


### MaskRom 模式

MaskRom 模式用于 bootloader 损坏时的系统修复。

一般情况下是不用进入 MaskRom 模式的，只有在 bootloader 校验失败（读取不了 IDB 块，或 bootloader 损坏） 的情况下，BootRom 代码 就会进入 MaskRom 模式。此时 BootRom 代码等待主机通过 USB 接口传送 bootloader 代码，加载并运行之。

***要强行进入 MaskRom 模式，请参阅[《MaskRom模式》](04-maskrom_mode.md)一章。***