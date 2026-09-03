# 使用 SD 卡升级固件
本文主要介绍如何通过 MicroSD 卡升级主板上的固件。

使用 MicroSD 卡升级固件时，建议使用 32G 及以下容量的卡。需要在电脑上通过制卡工具将统一固件写入 MicroSD 卡，目前此操作只支持在 Windows 操作系统上完成。

## 准备工具

* RK182X开发套件
* 电脑
* MicroSD 卡
* USB 读卡器
* [**SD_Firmware_Tool**](https://community.t-firefly.com/doc/download/358)

## 操作步骤
1. 下载需要升级到主板上的统一固件。
2. 打开 SD_Firmware_Tool，勾选“固件升级”，点击“选择固件”并选择正确的固件文件。
3. 将 MicroSD 卡插入 USB 读卡器，再插入电脑 USB 接口，在设备列表中选择正确的 USB 设备。
4. 点击“开始创建”，等待制卡完成。
5. 取出 MicroSD 卡，插入主板的 MicroSD 卡插槽并上电，主板会自动开始升级。
6. 升级完成后取出 MicroSD 卡，主板会自动重启。


<center>

![](../../../gs1-n2_img/common/upgrade_firmware_sd_tool_zh.png)
</center>