# MaskRom模式

***有关启动模式的介绍，请参阅[《升级固件介绍》](upgrade_bootmode.md)一章***

## 简介

`MaskRom` 模式是设备变砖的最后一条防线。强行进入 `MaskRom` 涉及硬件操作，有一定风险，因此仅在设备进入不了 `Loader` 模式的情况下，方可尝试 `MaskRom` 模式。进入 `MaskRom` 的原理是人为的把 EMMC 的数据脚与地线短接，系统会认为 EMMC 数据出错，从而清除 EMMC 数据。

**请小心阅读，并谨慎操作！**

操作步骤如下：

1. 设备断开电源
2. 按住底板 MASKROM 按键
3. 设备插入电源上电

![](../../../rv1126b_img/CORE-1126BJD4/upgrade_maskrom_key.png)





此时设备就会进入 MaskRom 模式。

![](../../../rv1126b_img/common/upgrade_maskrom_zh.png)

注意：如果按照上述操作执行，最终 Windows PC 烧写工具还是没发现到 MASKROM 设备，这就要检查下 Windows PC 的软件驱动是否安装到最新版本。

点击：[驱动下载](https://www.t-firefly.com/doc/download/333.html#other_930)

先点 1 驱动卸载，再点 2 驱动安装。安装好驱动后，再按照上述步骤顺序执行操作，Windows PC 烧写工具就能识别到 MASKROM 设备了。

![](../../../rv1126b_img/common/DriverAssitant.png)