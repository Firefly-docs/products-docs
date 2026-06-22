# 使用SD卡更新固件

本文主要介绍了如何将实现使用MicroSD卡，更新主板上的固件。

使用MicroSD更新固件，需要在电脑上，通过做卡工具，将统一固件写入MicroSD卡，目前此操作只支持在Windows操作系统上完成。

## 准备工作

* 主板
* 电脑
* SD卡
* USB读卡器

## 操作步骤

* **SD_Firmware_Tool**
SD_Firmware_Tool 1.xx  为瑞芯微官方发布的SD卡烧录工具。


### 操作步骤如下：
* 下载SD卡制作工具：[SD_Firmware_Tool](http://www.t-firefly.com/doc/download/51.html#other_394)
* 下载需要升级到主板上的统一固件。
* 将MicroSD插入USB读卡器，再插入到电脑USB口上。
* 打开SD_Firmware_Tool，勾选“固件升级”框，选择正确的可移动磁盘设备和升级固件。
* 点击开始创建之后，等待创建结束。
* 取出MicroSD卡，插入主板的MicroSD卡插槽，对主板上电开机，主板自动开始升级。
* 升级完成后，取出MicroSD卡，主板自动重启，完成整个更新固件的流程。

![](../../../rk3288_img/SD_Firmware_Tool.png)