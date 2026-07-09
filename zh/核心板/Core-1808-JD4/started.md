
# 入手指南
## 配件

AIO-1808-JD4 的标准套装包含以下配件：

* CORE-1808-JD4 核心板一块
* MB-JD4-3399&3399PRO 底板一块
* 铜管天线×1  
* 12V-2A电源适配器一个
* 公对公USB线一条

另外可以选购的配件有：

* Firefly串口模块

另外，在使用过程中，你可能需要以下配件：

*    显示设备
     *   10.1寸IPS高清屏幕模组，支持10点同时触摸，超清画质，兼容性强(**注意: 底板上的HDMI接口为了兼容其它的核心板，RK1808并不支持HDMI显示功能**)
*    网络
     *   100M/1000M 以太网线缆，及有线路由器
     *   WiFi 路由器
*    输入设备
     *   USB 无线/有线的鼠标/键盘
*    升级固件，调试
     *   公对公USB线
     *   串口转 USB 适配器 

 <a id="firmware-format"></a>
## 固件类型

 固件有两种格式：

 * 原始固件(raw firmware)
 * RK固件(Rockchip firmware)

<a id="raw-firmware-format"></a>
    [原始固件]是一种能以逐位复制的方式烧写到存储设备的固件，是存储设备的原始映像。原始固件一般烧写到SD卡中，但也可以烧写到eMMC中。烧写原始固件有许多工具可以选用：
		
<a id="rk-firmware-format"></a>
    [RK固件]是以Rockchip专有格式打包的固件，使用Rockchip提供的upgrade_tool(Linux)或AndroidTool(Windows)工具烧写到eMMC闪存中。RK固件是Rockchip的传统固件打包格式，常用于Android设备上。另外，Android的RK固件也可以使用SD Firmware Tool工具烧写到SD卡中。

<a id="partition-image"></a>
    [分区映像]是分区的映像数据，用于存储设备对应分区的烧写。例如，编译Android SDK会构建出`boot.img`、`kernel.img`和`system.img`等分区映像文件，`kernel.img`会被写到eMMC或SD卡的“kernel”分区。

## 下载和烧写固件


以下是支持的系统列表：

- Buildroot
- Ubuntu18.04

根据所使用的操作系统来选择合适的工具去烧写固件：

- 烧写 SD卡
  + 图形界面烧写工具：
	* [Etcher] (windows/linux/Mac)
  + 命令行烧写工具
	* [dd] (Linux)

- 烧写 eMMC
  + 图形界面烧写工具：
	* [AndroidTool] (Windows)
  + 命令行烧写工具：
	* [upgrade_tool] (Linux)

## 开机
确认主板配件连接无误后，将电源适配器插入带电的插座上，电源线接口插入开发板，开发板第一次加电会自动开机。 在系统选择关机后，维持开发板供电，此时 AIO-1808-JD4 方式如下：

*    长按电源键三秒(扩展按键)

开机时，蓝色的电源指示灯会亮起。

[RK固件]:started.html#rk-firmware-format
[原始固件]:started.html#raw-firmware-format
[分区映像]:started.html#partition-image
[固件类型]:started.html#firmware-format
[SD Firmware Tool]: upgrade_firmware_rk.html#SD_Firmware_Tool
[Etcher]: upgrade_firmware_sd.html#Etcher
[dd]: upgrade_firmware_sd.html#dd
[AndroidTool]: upgrade_firmware.html#Androidtool
[upgrade_tool]: upgrade_firmware.html#upgrade_and_upgrade_tool
[rkdeveloptool]: upgrade_firmware.html#upgrade_and_rkdeveloptoo