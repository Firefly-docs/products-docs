
# FAQ

<Accordions>

<Accordion title="Q：K3 支持哪些烧录方式？" type="single" id="aibox_upgrade_faq_1">

**A：** AIBOX-K3 支持两种烧录方式：

* **USB 线缆烧录**：通过 Type-C 线缆将主机上的固件烧录到板子的存储器中，支持烧录统一固件与分区映像，需先让板子进入烧录模式，再使用 titanflasher 烧录工具完成刷机，详见[《使用USB线缆升级固件》](upgrade_firmware.md)。
* **MaskRom（硬件烧录模式）烧录**：当板子无法进入 Loader 模式时的最后防线，通过 boot 按键配合上电进入，详见[《使用USB线缆升级固件》](upgrade_firmware.md)。

刷机前请详细阅读[《刷机工具使用手册》](https://spacemit.com/community/document/info?lang=zh&nodepath=tools/user_guide/flasher_user_guide.md)。

titanflasher 烧录工具界面如下：

<center>

<img alt="" src="../../../../../aibox_img/AIBOX-K3/upgrade_firmware_titanflasher_zh.png" width="800">
</center>

</Accordion>

<Accordion title="Q：如何进入烧录模式（Loader）？" type="single" id="aibox_upgrade_faq_2">

**A：** 软件方式进入 Loader 模式需要操作 uboot 终端（不能使用 adb 工具），步骤如下：

1. 用 Type-C 线缆连接板子与主机，并接好串口调试终端。
2. 在串口调试终端执行 `reboot`，重启板子。
3. 板子重启到 uboot 阶段时按住 `s` 键进入 uboot 调试终端，然后删除多余的 `s` 字符输入信息。
4. 在 uboot 调试终端执行 `fastboot 0` 进入烧写模式。
5. 在 PC 端 titanflasher 上点击：研发工具 --> 单机烧录 --> 扫描设备 --> 选择刷机文件 --> 开始刷机。

> 注意：此时 titanflasher 工具会烧写一部分必要的固件进板子，但不会烧入全部固件，还需在板子的 uboot 终端再次按下 Enter 按键进行后续的固件烧录。

如果板子进入不了 Loader 模式，可以强行进入硬件烧录模式，方法见下文「常见错误有哪些？」。

</Accordion>

<Accordion title="Q：是否支持分区烧写？" type="single" id="aibox_upgrade_faq_3">

**A：** 支持。在 titanflasher 上除了烧录统一固件，还可以通过配置分区文件烧写分区映像，步骤如下：

研发工具 --> 单机烧录 --> 扫描设备 --> 本地文件 --> 选择刷机文件 --> 刷完自启动 --> 配置分区文件 --> 开始刷机。

配置分区文件时需要选择分区文件：

1. `partition_4M.json`：更新核心板上的 Nor Flash。
2. `partition_universal.json`：更新核心板上的 UFS 分区映像。

<center>

<img alt="" src="../../../../../aibox_img/AIBOX-K3/upgrade_firmware_part_zh.png" width="800">
</center>

</Accordion>

<Accordion title="Q：烧写失败怎么办？" type="single" id="aibox_upgrade_faq_4">

**A：** 如果烧写过程中出错，通常是由于使用的 USB 线连接不良、劣质线材，或者电脑 USB 口驱动能力不足导致的，可按以下步骤排查：

1. 更换一条状态良好的 Type-C 线缆。
2. 更换电脑的 USB 端口，重新扫描设备后再试。
3. 仍失败时，可尝试强行进入硬件烧录模式后再烧写（见下文「常见错误有哪些？」）。

</Accordion>

<Accordion title="Q：常见错误有哪些？" type="single" id="aibox_upgrade_faq_5">

**A：**

* **无法进入 Loader 模式**：可以强行进入硬件烧录模式：PC 通过 Type-C 线接到板子的 Type-C USB 3.0 口（注意不要接错到 USB 串口上），按住 boot 按键后上电，设备即进入硬件烧录模式，再点击烧写工具的《扫描设备》识别设备。

<center>

<img alt="" src="../../../../../aibox_img/AIBOX-K3/upgrade_maskrom_zh.png" width="800">
</center>

* **烧写工具识别不到设备**：确认 Type-C 线缆连接良好且没有接错端口（应接 Type-C USB 3.0 口），必要时更换 USB 线或电脑 USB 端口，重新点击《扫描设备》。
* **Nor Flash 相关**：`partition_4M.json` 用于更新核心板上的 Nor Flash，烧写前请先确认核心板上是否贴有 Nor Flash 存储器，避免选错分区文件。

</Accordion>

<Accordion title="Q：设备有哪些启动模式？" type="single" id="aibox_upgrade_faq_6">

**A：** AIBOX-K3 有以下启动模式：

* Normal 模式
* Loader 模式
* 硬件烧录模式

**Normal 模式**

Normal 模式就是正常的启动过程，各个组件依次加载，正常进入系统。

**Loader 模式**

在 Loader 模式下，bootloader 会进入升级状态，等待主机命令，用于固件升级等。

**硬件烧录模式**

硬件烧录模式用于 bootloader 损坏时的系统修复。

</Accordion>

<Accordion title="Q：板子会不会变砖？" type="single" id="aibox_upgrade_faq_7">

**A：** AIBOX-K3 有灵活的启动方式。一般情况下，除非硬件损坏，开发板是不会变砖的。如果在升级过程中出现意外，bootloader 损坏，导致无法重新升级，此时仍可以强制进入硬件烧录模式来修复。

***要强行进入硬件烧录模式，请参阅[《使用USB线缆升级固件》](upgrade_firmware.md)一章。***

</Accordion>

<Accordion title="Q：设备从哪些存储器中加载系统？" type="single" id="aibox_upgrade_faq_8">

**A：** AIBOX-K3 从以下的存储器中加载系统：

* UFS 接口
* PCIE 接口

</Accordion>

</Accordions>


