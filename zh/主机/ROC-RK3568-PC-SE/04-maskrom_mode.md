# MaskRom模式

***有关启动模式的介绍，请参阅[《升级固件介绍》](01-bootmode.md)一章***

## 简介

`MaskRom` 模式是设备变砖的最后一条防线。强行进入 `MaskRom` 涉及硬件操作，有一定风险，因此仅在设备进入不了 `Loader` 模式的情况下，方可尝试 `MaskRom` 模式。进入 `MaskRom` 的原理是人为的把 EMMC 的数据脚与地线短接，系统会认为 EMMC 数据出错，从而清除 EMMC 数据。

**请小心阅读，并谨慎操作！**

操作步骤如下：


* 设备断开电源
* 设备插入 Type-C 数据线
* 按住设备上的 Maskrom 按键
* 设备插入电源
* 稍候片刻，之后松开按键

![](../../img/ROC-RK3568-PC-SE/maskrom_test_points.jpg)

Markrom 按键(短接 EMMC）位置可以查看[《接口定义》](interface_definition.md)

板子同时贴有NOR flash，若EMMC为空，而NOR flash中有烧录过文件，则需要短接NOR flash附近的D0和GND测试点进入Maskrom模式,下图为短接点。此时升级固件需要参考章节[《切换升级存储器》](03-upgrade_firmware_with_flash)

![](../../img/ROC-RK3568-PC-SE/maskrom_test_points_flash.jpg)



此时设备就会进入 MaskRom 模式。

![](../../img/maskrom_zh.png)