# 固件和工具

- [固件下载页面][下载页面]

固件烧写工具：

- 烧写 SD 卡
    + 图形界面烧写工具：
        * [SDCard Installer] (Linux/Windows/Mac)
        * [Etcher] (Linux/Windows/Mac)
		* [SD Firmware Tool] (Windows)
    + 命令行烧写工具
        * [dd] (Linux)
- 烧写 eMMC
    + 图形界面烧写工具：
        * [AndroidTool] (Windows)
    + 命令行烧写工具：
        * [upgrade_tool] (Linux)

# 文档和参考

- [Partition and Storage Map](http://opensource.rock-chips.com/wiki_Partitions)

# 硬件规格书和接口

规格书：

V1.0
- [ROC-RK3328-CC 原理图](http://www.t-firefly.com/download/ROC-RK3288-CC/ROC-RK3328-CC-V1.0-A_yl.pdf)
- [ROC-RK3328-CC 贴片图](http://www.t-firefly.com/download/ROC-RK3288-CC/ROC-RK3328-CC-V1.0-A_tp.pdf)

V1.3
- [ROC-RK3328-CC 原理图](https://download.t-firefly.com/product/RK3328/Docs/ROC-RK3328-CC/roc-rk3328-cc-v13-b.pdf)
- [ROC-RK3328-CC 贴片图](https://download.t-firefly.com/product/RK3328/Docs/ROC-RK3328-CC/ROC-RK3328-CC-V1.3-B%E8%B4%B4%E7%89%87%E5%9B%BE.pdf)

- [Rockchip RK3328 规格书](http://www.t-firefly.com/download/ROC-RK3288-CC/Rockchip%20RK3328%20Datasheet%20V1.0-20170117.pdf)
- [Rockchip RK805 规格书 V1.1](http://files.pine64.org/doc/rock64/Rockchip_RK805_Datasheet_V1.1%C2%A020160921.pdf)

扩展口：

![](../../../rk3328_img/ROC-RK3328-CC/hw_expansion_interface.png)

开发板正面：

![](../../../rk3328_img/ROC-RK3328-CC/hw_board_interface_top.png)

开发板背面：

![](../../../rk3328_img/ROC-RK3328-CC/hw_board_interface_bottom.png)

# 社区

Firefly：

- [论坛]
- [脸书]
- [Google+]
- [油管]
- [推特]
- [在线商城]

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