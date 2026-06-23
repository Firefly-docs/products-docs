### 使用SD卡升级固件
本文主要介绍如何通过MicroSD卡，升级主板上的固件。

使用MicroSD升级固件(建议用32G及以下容量的卡)，需要在电脑上，通过做卡工具，将统一固件写入MicroSD卡，目前此操作只支持在Windows操作系统上完成。

#### 准备工具

* 主板
* 电脑
* SD卡
* USB读卡器
* [**SD_Firmware_Tool**](https://www.t-firefly.com/doc/download/358.html#other_573)

#### 操作步骤
* 下载需要升级到主板上的统一固件。
* 打开SD_Firmware_Tool，勾选`固件升级`框，点击`选择固件`选择正确升级固件。
* 将MicroSD插入USB读卡器，再插入到电脑USB口上,在列表式组合框中选择正确的USB设备。
* 点击`开始创建`之后，等待创建结束。
* 取出MicroSD卡，插入主板的MicroSD卡插槽，对主板上电开机，主板自动开始升级。
* 升级完成后，取出MicroSD卡，主板自动重启，完成整个升级固件的流程。


<center>

![](../../../gs1-n2_img/RK182X开发套件/rockchip_upgrade_firmware_sd_tool.png)
</center>