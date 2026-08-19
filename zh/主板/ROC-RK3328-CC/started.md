# 简介

恭喜你入手 [ROC-RK3328-CC] 开发板！

对刚接触的人来说，[《上手指南》] 提供了烧写固件并让开发板跑起来的一切指引。

如果仔细阅读 [《上手指南》]后仍有疑问，请先查看[《常见问题解答》]。如果问题仍未解决，请参照[《串口调试》]一章，获取系统日志信息，并联系我们（[联系方式]），以便共同完善这份文档。

## 产品规格

[ROC-RK3328-CC]，是 Firefly 荣誉出品的第一块信用卡大小、性能强、功耗小的开源主板。

![](../../../rk3328_img/ROC-RK3328-CC/hw_board_interface_top.png)

主要功能：

- 核心
    + Quad-Core ARM® Cortex-A53 64-bit 处理器，高达 1.5GHz 的主频
    + ARM Mali-450 MP2 Quad-Core GPU，支持 OpenGL ES1.1/2.0，OpenVG1.1
    + DDR4 RAM (1GB/2GB/4GB)
- 连接
    + 2 x USB 2.0 Host 口，1 x USB 3.0 Host 口
    + 10/100/1000Mb 网口
    + 1 x IR 红外模组，支持自定义 IR 远程控制
    + 40 引脚拓展接口，包含 GPIO，I2S，SPI，I2C，UART，PWM SPDIF，DEBUG
- 显示
    + HDMI 2.0 ( Type-A )，支持最大 4K@60Hz 显示
    + TV out，CVBS 显示，参照 480i，576i 标准
- 音频
    + I2S，支持 8 通道
    + SPDIF，音频输出
- 视频
    + 4K VP9 和 4K 10位 H265 / H264 视频解码，可达 60fps
    + 1080P 多格式视频解码(WMV，MPEG-1/2/4，VP9，H.264，H.265)
    + 1080P 视频解码，支持 H.264/H.265
    + 视频后处理器：去隔行，去噪，边缘/细节/颜色优化
- 存储
    + 高速 eMMC 扩展接口
    + MicroSD (TF) 卡槽

(请参考完整的[《完整规格说明》](http://www.t-firefly.com/product/rocrk3328cc.html#spec))

这款令人难以置信的超小型主板，凭借其低功耗和高性能，能够安静流畅地运行 Android 7.1 或 Ubuntu 16.04 系统。

## 包装配件

[ROC-RK3328-CC] 标准套件的包装清单：

- ROC-RK3328-CC 主板
- Micro USB 线

对开发者来说，建议购买以下配件：

- [5V2A 电源适配器]，一个好的电源是必须的
- [USB 转串口适配器]，用于串口调试
- [eMMC 闪存]，提供更高的系统性能和可靠性

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
[下载页面]: https://community.t-firefly.com/doc/download/34
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
 <a id="firmware-format"></a>