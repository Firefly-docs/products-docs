# 固件升级

Firefly 提供` Linux `和` Windows ` 升级工具，升级步骤非常简单。

## 获取固件

请到官方[资源下载](http://www.t-firefly.com/doc/download/101.html)页面下载对应设备的固件。

## Windows

### 安装驱动

* [下载驱动(RK 驱动助手)](https://www.t-firefly.com/doc/download/101.html)
* 以管理员的方式运行` exe `文件
* 完成安装

### 升级 

* 以管理员的方式打开 Windows 升级工具 [RKDevTool](http://www.t-firefly.com/doc/download/101.html)。

* 按住设备` recovery `按键后使用` 双公头 USB `线连接设备`OTG 升级口`和电脑然后上电，` RKDevTool `显示识别到设备后松开` recovery `按键（需要` 3 `秒左右）。

![](../../../rv1126b_img/rkdevtool_loader.png)

* 导入升级使用的配置文件` core-rv1126-jd4.cfg`。

![](../../../rv1126b_img/rkdevtool_setting.png)

* 选择好要烧写分区固件的路径。

![](../../../rv1126b_img/rkdevtool_path.png)

* 点击`执行`等待升级完成。

![](../../../rv1126b_img/rkdevtool_start.png)

* 修改` config.ini `文件可以设置语言。

![](../../../rv1126b_img/language.png)

## Linux

### 安装工具

* 下载 Linux 升级工具 [upgrade_tool](http://www.t-firefly.com/doc/download/101.html)。
* 安装
```
unzip Linux_Upgrade_Tool_xxxx.zip
cd Linux_UpgradeTool_xxxx
sudo mv upgrade_tool /usr/local/bin
sudo chown root:root /usr/local/bin/upgrade_tool
sudo chmod a+x /usr/local/bin/upgrade_tool
```

### 升级

* 按住设备` recovery `按键后使用` Typec `线连接设备和电脑上电, 等待` 3 `秒松开` recovery `按键。

* 进入下载固件的路径下依次运行以下命令完成升级。
```
sudo upgrade_tool ul MiniLoaderAll.bin
sudo upgrade_tool di -p parameter.txt
sudo upgrade_tool di -uboot uboot.img
sudo upgrade_tool di -b boot.img
sudo upgrade_tool di -r recovery.img
sudo upgrade_tool di -m misc.img
sudo upgrade_tool di -oem oem.img
sudo upgrade_tool di -userdata userdata.img
sudo upgrade_tool di -rootfs rootfs.img
sudo upgrade_tool rd

# 如果升级的固件不在当前目录，请输入完整路径。
```
## 使用SD卡升级固件
本文主要介绍如何通过MicroSD卡，升级主板上的固件。
使用MicroSD升级固件，需要在电脑上，通过做卡工具，将统一固件写入MicroSD卡，目前此操作只支持在Windows操作系统上完成。
### 准备工具
* 主板
* 电脑
* SD卡
* USB读卡器
* [SD_Firmware_Tool](https://www.t-firefly.com/doc/download/101.html#other_622)
### 操作步骤
* 下载需要升级到主板上的统一固件。
* 打开SD_Firmware_Tool，勾选固件升级框，点击选择固件选择正确升级固件。
* 将MicroSD插入USB读卡器，再插入到电脑USB口上,在列表式组合框中选择正确的USB设备。
* 点击开始创建之后，等待创建结束。
* 取出MicroSD卡，插入主板的MicroSD卡插槽，对主板上电开机，主板自动开始升级。
* 升级完成后，取出MicroSD卡，主板自动重启，完成整个升级固件的流程。
![](../../../rv1126b_img/SD_Firmware_Tool_zh.png)

## 注意事项

* 如果需要单独烧写某个分区，`Windows `只需要在` RKDevTool `勾选需要烧写的分区，`Linux `只需要执行对应烧写分区命令即可。

* Wiki 中所有固件、工具、驱动下载都可以在官网[资源下载](http://www.t-firefly.com/doc/download/101.html)页面中找到。