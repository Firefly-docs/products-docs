# 介绍

## 前言

CT36L/CT36B出厂默认安装 Linux 操作系统。

CT36L/CT36B有灵活的启动方式。一般情况下，除非硬件损坏，CT36L/CT36B 开发板是不会变砖的。

如果在升级过程中出现意外，bootloader 损坏，导致无法重新升级，此时仍可以进入 `MaskRom` 模式来修复。


## 固件获取
*	CT36L[下载链接](https://www.t-firefly.com/doc/download/238.html)
*	CT36B[下载链接](https://www.t-firefly.com/doc/download/238.html)
## 升级方式
CT36L/CT36B 支持通过以下两种方式升级固件：

* [使用USB线缆升级固件](upgrade_firmware.md)<br><br>使用CT36L/CT36B 将主板连接到电脑上，通过升级工具将固件烧写到主板上。<br><br>
* [使用SD卡升级固件](upgrade_firmware_sd.md)<br><br>
* 注1：需要 1.7 或更高版本的 `SocToolKit` 工具才能支持 SD 卡升级启动功能。
* 注2：此功能需要用户以管理员身份运行 `SocToolKit.exe`（开启工具时会默认询问）。
* 将制作成功的 SD 卡插入设备后重启，设备将优先进入 SD 卡中的 U-Boot 终端。
* 若 SD 卡有升级功能，则会自动升级设备。
* 升级完成后，需要拔掉 SD 卡，再重启设备，方能进入设备系统中。


## 启动存储器

CT36L 从以下的存储器中加载系统：

* SPI FLASH 接口
* SDMMC 接口

CT36B  从以下的存储器中加载系统：

* EMMC 接口
* SDMMC 接口

## 启动模式

CT36L/CT36B有两种启动模式：

* Normal 模式
* MaskRom 模式

### Normal 模式

Normal 模式就是正常的启动过程，各个组件依次加载，正常进入系统。

### MaskRom 模式

MaskRom 模式用于固件烧写。

***要强行进入 `MaskRom 模式`，请参阅[《MaskRom模式》](upgrade_maskrom_mode.md)一章。***