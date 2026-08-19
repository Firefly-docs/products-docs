# 启动模式说明

## 前言

AIO-RK3328-JD4 有灵活的启动方式。一般情况下，除非硬件损坏，AIO-RK3328-JD4开发板是不会变砖的。

如果在升级过程中出现意外，bootloader 损坏，导致无法重新升级，此时仍可以进入 `MaskRom` 模式来修复。

## 加载方式

AIO-RK3328-JD4 有 20KB 的 BootRom 和 36KB 的内部 SRAM，支持从以下设备加载系统：

* SPI 接口
* eMMC 接口
* SDMMC 接口

AIO-RK3328-JD4 从Type-C接口下载系统代码。

## 启动次序

启动的次序是这样的：

1. 主控上电初始化
2. BootRom 代码在 SRAM 上运行，校验存储设备里的 bootloader
3. 校验通过，加载并运行 bootloader 引导代码
4. bootloader 引导代码负责初始化 DDR 内存，加载 bootloader 完整代码到 DDR 内存中并运行
5. bootloader 加载存储设备上的 Linux 内核，并将执行权交给 Linux 内核

## 启动模式

AIO-RK3328-JD4有三种启动模式：

* Normal 模式
* Loader 模式
* MaskRom 模式

### Normal 模式

Normal 模式就是正常的启动过程，各个组件依次加载，正常进入系统。

<a id="loader_mode"></a>

### Loader 模式

在 Loader 模式下，bootloader 会进入升级状态，等待主机命令，用于固件升级等。   
要进入 Loader 模式，必须让 bootloader 在启动时检测到 `RECOVERY`（恢复）键按下，且 USB 处于连接状态。有两种方法可以使设备进入升级模式:

一种方式是断开电源适配器

* Type-C 线连接好设备和主机。
* 按住设备上的 RECOVERY （恢复）键并保持。
* 插上电源
* 大约两秒钟后，松开 RECOVERY 键。

另一种方式是接上电源适配器

* Type-C数据线连接好设备和主机。
* 按住设备上的 RECOVERY （恢复）键并保持。
* 短按一下 RESET（复位）键。
* 大约两秒钟后，松开 RECOVERY 键。

### MaskRom 模式

MaskRom 模式用于 bootloader 损坏时的系统修复。

一般情况下是不用进入 MaskRom 模式的，只有在 bootloader 校验失败（读取不了 IDR 块，或 bootloader 损坏） 的情况下，BootRom 代码 就会进入 MaskRom 模式。此时 BootRom 代码等待主机通过 USB 接口传送 bootloader 代码，加载并运行之。   

***要强行进入 MaskRom 模式，请参阅[《MaskRom》]一章。***

[《上手指南》]: started.md
[《常见问题解答》]: faqs.md
[《串口调试》]: debug.md
[《编译 Linux 根文件系统》]: linux_build_ubuntu_rootfs.md
[《编译 Linux 固件》]:linux_compile_gpt.md
[《编译 Android 固件》]:compile_android8.1_firmware.md
[《MaskRom》]:04-maskrom_mode.md
[《编译Linux固件(GPT)》]:linux_compile_gpt.md
[《烧写须知》]: 02-upgrade_table.md
[《ADB 介绍》]: adb_use.md
[Loader模式]:01-bootmode.md#loader_mode
[联系方式]: resource.md#社区
[原始固件]: started.md#raw-firmware-format
[RK 固件]: 02-upgrade_table.md#rk-firmware-format
[分区映像]: started.md#partition-image
[upgrade_tool]:03-upgrade_firmware.md#upgrade-tool
[AndroidTool]:03-upgrade_firmware.md#androidtool
[CORE-RK3328-JD4]:http://www.t-firefly.com/product/coreboard/core_3328_jd4.html?theme=pc
[下载页面]: https://community.t-firefly.com/doc/download/34
[论坛]: http://bbs.t-firefly.com
[脸书]: https://www.facebook.com/TeeFirefly
[Google+]: https://plus.google.com/u/0/communities/115232561394327947761
[油管]: https://www.youtube.com/channel/UCk7odZvUrTG0on8HXnBT7gA
[推特]: https://twitter.com/TeeFirefly
[在线商城]: http://store.t-firefly.com
[USB 转串口适配器]: https://store.t-firefly.com/goods.php?id=24
[5V2A 电源适配器]: https://store.t-firefly.com/goods.php?id=69
[《存储映射》]: http://opensource.rock-chips.com/wiki_Partitions#Default_storage_map