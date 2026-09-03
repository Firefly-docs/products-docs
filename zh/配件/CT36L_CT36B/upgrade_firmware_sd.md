# 使用SD卡升级固件

## 介绍

本文主要介绍如何通过MicroSD卡，升级主板上的固件。

使用 MicroSD 升级固件，需要在电脑上，通过做卡工具，将统一固件写入 MicroSD 卡，目前此操作只支持在 Windows 操作系统上完成。

## 准备工具

* 主板
* 电脑
* SD卡
* USB读卡器
* [**SocToolKit**](https://community.t-firefly.com/doc/download/238)

## 操作步骤

1. 下载需要升级到主板上的固件。
2. 打开 SocToolKit，点击 SDTool 选择 MicroSD 卡工具。
3. 选择要操作的 MicroSD 卡设备。
4. 选择 MicroSD 卡的作用。支持 MicroSD 卡升级固件和 MicroSD 卡系统启动。这里选择 MicroSD 卡升级固件。
6. 当前 MicroSD 卡是作为升级卡，所以需要添加所有的升级固件。
7. 点击 Create SD 按钮进行 MicroSD 升级卡的制作。
8. 将制作成功的 MicroSD 卡插入设备后重启，设备将优先进入 MicroSD 卡中的 U-Boot 终端。
9. 若 MicroSD 卡有升级功能，则会自动升级设备。
10. 升级完成后，需要拔掉 MicroSD 卡，再重启设备，方能进入设备系统中。


* 注1：需要 1.7 或更高版本的 SocToolKit 工具才能支持 SD 卡升级启动功能。
* 注2：此功能需要用户以管理员身份运行 SocToolKit.exe（开启工具时会默认询问）。

![](../../../rv1106_img/CT36L/upgrade_firmware_sd_tool_zh-1.png)
![](../../../rv1106_img/CT36L/upgrade_firmware_sd_tool_zh-2.png)