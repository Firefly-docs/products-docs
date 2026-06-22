# 启动模式说明

## 前言

AIO-3288J 出厂默认安装 Android 操作系统，如果用户要运行其他操作系统，需要使用对应的固件烧写到主板。

AIO-3288J 有灵活的启动方式。一般情况下，除非硬件损坏，AIO-3288J 开发板是不会变砖的。

如果在升级过程中出现意外，bootloader 损坏，导致无法重新升级，此时仍可以进入 `MaskRom` 模式来修复。

## 固件信息

* [固件更新说明](https://github.com/T-Firefly/firmware_doc)
* [固件下载链接](http://www.t-firefly.com/doc/download/page/id/16.html)

## 升级方式

AIO-3288J 支持通过以下两种方式更新固件：

* 使用 USB 线缆更新固件

    使用 USB 线将主板连接到电脑上，通过升级工具将固件烧写到主板上。

* 使用 MicroSD 卡

    通过升级卡制作工具，将 MicroSD 卡制作为升级卡，将升级卡插入主板，上电开机，机器自动执行升级。

## 启动介质

AIO-3288J 从以下设备加载系统：

* eMMC 接口
* SDMMC 接口

## 启动模式

AIO-3288J 有三种启动模式：

* Normal 模式
* Loader 模式
* MaskRom 模式

### Normal 模式

Normal 模式就是正常的启动过程，各个组件依次加载，正常进入系统。

### Loader 模式

在 Loader 模式下，bootloader 会进入升级状态，等待主机命令，用于固件升级等。要进入 Loader 模式，必须让 bootloader 在启动时检测到 `RECOVERY`（恢复）键按下，且 USB 处于连接状态。

使设备进入升级模式的方法如下:

!INCLUDE "upgrade_firmware_enter_loader.mdpp"

### MaskRom 模式

MaskRom 模式用于 bootloader 损坏时的系统修复。

一般情况下是不用进入 MaskRom 模式的，只有在 bootloader 校验失败（读取不了 IDR 块，或 bootloader 损坏） 的情况下，BootRom 代码 就会进入 MaskRom 模式。此时 BootRom 代码等待主机通过 USB 接口传送 bootloader 代码，加载并运行之。

***要强行进入 MaskRom 模式，请参阅[《MaskRom模式》](maskrom_mode.md)一章。***