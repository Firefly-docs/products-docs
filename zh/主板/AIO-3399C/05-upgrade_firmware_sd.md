# 使用 SD 卡升级固件

本文主要介绍了如何将实现使用MicroSD卡，更新主板上的固件。<font color = "red">但也仅限于固件小于4G大小的情况下</font>，否则请查看[《若固件大于4G的SD升级卡制作教程》](05-upgrade_firmware_sd_morethan_4g.md)。

使用MicroSD更新固件，需要在电脑上，通过做卡工具，将统一固件写入MicroSD卡，目前此操作只支持在Windows操作系统上完成。

## 准备工作

* AIO-3399C
* 电脑
* MicroSD卡(即TF 卡)
* USB读卡器
* [SD_Firmware_Tool](http://www.t-firefly.com/doc/download/page/id/54.html#other_541)

## 操作步骤
1. 运行`SD_Firmware_Tool.exe`
2. 可修改**config.ini**文件，设置`Selected=1`则语言为中文，设置`Selected=2`则语言为英文。
3. 下载需要升级到主板上的统一固件。
4. 将MicroSD插入USB读卡器，再插入到电脑USB口上。
5. 打开SD_Firmware_Tool，选择正确的可移动磁盘设备，勾选`固件升级`框，点击`选择固件`选择所要升级的固件。

    ![](../../../rk3399_img/sdfirmwaretool.zh_CN.jpg)
6. 点击`开始创建`之后，等待创建结束。

    ![](../../../rk3399_img/sdfirmwaretool_done.zh_CN.jpg)
7. 取出MicroSD卡，插入主板的MicroSD卡插槽，对主板上电开机，主板自动开始升级。
8. 升级完成后，取出MicroSD卡，主板自动重启，完成整个更新固件的流程。