# 上手指南

[ROC-RK3328-CC] 支持从以下存储设备启动：

- SD 卡
- eMMC

<a id="firmware-format"></a>

## 固件格式

固件有两种格式：

- 原始固件(raw firmware)
- RK固件(Rockchip firmware)

<a id="raw-firmware-format"></a>

[原始固件]，是一种能以逐位复制的方式烧写到存储设备的固件，是存储设备的原始映像。[原始固件]一般烧写到 SD 卡中，但也可以烧写到 eMMC 中。
烧写[原始固件]有许多工具可以选用：

- 烧写 SD 卡
    + 图形界面烧写工具：
        * [SDCard Installer] (Linux/Windows/Mac)
        * [Etcher] (Linux/Windows/Mac)
    + 命令行烧写工具
        * [dd] (Linux)
- 烧写 eMMC
    + 图形界面烧写工具：
        * [AndroidTool] (Windows)
    + 命令行烧写工具：
        * [upgrade_tool] (Linux)
        * [rkdeveloptool] (Linux)

<a id="rk-firmware-format"></a>

[RK 固件]，是以 Rockchip专有格式打包的固件，使用 Rockchip 提供的工具可以烧写到 eMMC 或者 SD 卡中。
烧写[RK 固件]有以下工具可以选用：

- 烧写 SD 卡
    + 图形界面烧写工具：
        * [SD Firmware Tool] (Windows)
- 烧写 eMMC
    + 图形界面烧写工具：
        * [AndroidTool] (Windows)
    + 命令行烧写工具：
        * [upgrade_tool] (Linux)


<a id="partition-image"></a>

[分区映像]，是分区的映像数据，用于存储设备对应分区的烧写。例如，编译 Android SDK会构建出 `boot.img`、`kernel.img`和`system.img`等[分区映像]文件，`kernel.img` 会被写到eMMC 或 SD 卡的 "kernel" 分区。

## 下载和烧写固件

以下是支持的系统列表：

- Android 7.1.2
- Android 8.1.0
- Ubuntu 18.04
- Ubuntu 16.04
- Debian 9
- LibreELEC 9.0

根据所使用的存储设备，所需的固件以及主机系统，来选择合适的工具去烧写固件：

- 烧写 SD 卡 : 参考[《烧写SD卡》](flash_sd.md)

- 烧写 eMMC : 参考[《烧写eMMC》](flash_emmc.md)


## 开发板上电启动

在开发板上电启动前，确认以下事项：

- 可启动的 SD 卡或eMMC
- 5V2A 电源适配器
- Micro USB 线

然后按照以下步骤操作：

1. 将电源适配器拔出电源插座。
2. 使用 micro USB 线连接电源适配器和主板。
3. 插入可启动的 SD 卡或eMMC 之一（不能同时插入）。
4. 插入 HDMI 线、USB 鼠标或键盘（可选）。
5. 检查一切连接正常后，电源适配器上电。

[《上手指南》]: getting_started.md
[《常见问题解答》]: faqs.md
[《串口调试》]: debug.md
[《编译 Linux 根文件系统》]: linux_build_ubuntu_rootfs.md
[联系方式]: resource.md#社区
[原始固件]: getting_started.md#raw-firmware-format
[RK 固件]: getting_started.md#rk-firmware-format
[分区映像]: getting_started.md#partition-image
[SDCard Installer]: flash_sd.md#sdcard-installer
[Etcher]: flash_sd.md#etcher
[dd]: flash_sd.md#dd
[SD Firmware Tool]: flash_sd.md#sd-firmware-tool
[AndroidTool]: flash_emmc.md#androidtool
[upgrade_tool]: flash_emmc.md#upgrade-tool
[rkdeveloptool]: flash_emmc.md#rkdeveloptool
[Rockusb 模式]: flash_emmc.md#rockusb-mode
[Maskrom 模式]: flash_emmc.md#maskrom-mode
[Rockusb 驱动]: flash_emmc.md#rockusb-driver
[ROC-RK3328-CC]: http://www.t-firefly.com/product/rocrk3328cc.html "ROC-RK3328-CC 官网"
[下载页面]: http://www.t-firefly.com/doc/download/page/id/34.html
[论坛]: http://bbs.t-firefly.com
[脸书]: https://www.facebook.com/TeeFirefly
[Google+]: https://plus.google.com/u/0/communities/115232561394327947761
[油管]: https://www.youtube.com/channel/UCk7odZvUrTG0on8HXnBT7gA
[推特]: https://twitter.com/TeeFirefly
[在线商城]: http://store.t-firefly.com
[USB 转串口适配器]: https://store.t-firefly.com/goods.php?id=24
[5V2A 电源适配器]: https://store.t-firefly.com/goods.php?id=69
[eMMC 闪存]: https://store.t-firefly.com/goods.php?id=71
[《存储映射》]: http://opensource.rock-chips.com/wiki_Partitions#Default_storage_map