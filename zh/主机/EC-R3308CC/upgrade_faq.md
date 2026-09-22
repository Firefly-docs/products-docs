# FAQ

## Q：设备有哪些启动模式？

**A：** EC-R3308CC 有三种启动模式：

* Normal 模式
* Loader 模式
* MaskRom 模式

### Normal 模式

Normal 模式就是正常的启动过程，各个组件依次加载，正常进入系统。

### Loader 模式

在 Loader 模式下，bootloader 会进入升级状态，等待主机命令，用于固件升级等。要进入 Loader 模式，必须让 bootloader 在启动时检测到 `RECOVERY`（恢复）键按下，且 USB 处于连接状态。

### MaskRom 模式

MaskRom 模式用于 bootloader 损坏时的系统修复。

一般情况下是不用进入 `MaskRom 模式`的，只有在 bootloader 校验失败（读取不了 IDB 块，或 bootloader 损坏） 的情况下，BootRom 代码 就会进入 `MaskRom 模式`。此时 BootRom 代码等待主机通过 USB 接口传送 bootloader 代码，加载并运行之。

## Q：如何让设备进入可升级模式（Loader 模式）？

**A：** 用 Type-C 数据线 连接好设备和主机，然后选择以下任一方式：

* 一种方式是断开电源适配器

    * 按住设备上的 RECOVERY （恢复）键并保持。
    * 插上电源。
    * 大约两秒钟后，松开 RECOVERY 键。

* 另一种方式是接上电源适配器

    * 按住设备上的 RECOVERY （恢复）键并保持。
    * 短按一下 RESET（复位）键。
    * 大约两秒钟后，松开 RECOVERY 键。

也可以在串口调试终端或 adb shell 中执行如下命令进入 Loader 模式：

```shell
reboot loader
```

## Q：如何判断当前处于哪种升级模式？

**A：** 如何确定板子当前处于哪种升级模式，我们可以通过工具去查看。

**Windows操作系统**

通过AndroidTool/RKDevTool工具可以看到下方提示`Found One LOADER Device`

<center>

<img alt="" src="../../../rk3308_img/upgrade_firmware_androidtool_zh.png" width="800">
</center>

如果有进行"进入Loader模式"的操作，仍旧没有看到烧写工具提示LOADER，此时可以看一下Windows主机是否有提示发现新硬件并配置驱动。打开设备管理器，会见到新设备 `Rockusb Device` 出现，如下图。如果没有，可返回上一步重新[安装驱动](03-upgrade_firmware.md)。

<center>

<img alt="" src="../../../rk3308_img/upgrade_firmware_new_equipment.png" width="800">
</center>

若烧写工具提示 `Found One MASKROM Device`，则说明设备当前处于 MaskRom 模式。

**Linux操作系统**

运行upgrade_tool后可以看到连接设备中有个`Loader`的提示

```shell
firefly@T-chip:~/severdir/down_firmware$ sudo upgrade_tool
List of rockusb connected
DevNo=1 Vid=0x2207,Pid=0x330c,LocationID=106    Loader
Found 1 rockusb,Select input DevNo,Rescan press <R>,Quit press <Q>:q
```

若列表中显示的是 `MaskRom`，则说明设备当前处于 MaskRom 模式。

**调试串口辅助判断**

如果设备已连接调试串口，也可以通过串口终端观察 U-Boot/内核启动日志，辅助判断设备所处的模式。以 Windows 下的 MobaXterm 为例，串口配置如下（波特率 1500000，具体连接方式请参阅[调试串口](usb_to_ttl.md)）：

<center>

<img alt="" src="../../../rk3308_img/debug_set_MobaXterm1.png" width="800">
</center>

<center>

<img alt="" src="../../../rk3308_img/debug_set_MobaXterm2.PNG" width="800">
</center>

如何让设备进入可升级模式，请参阅[《使用USB线缆升级固件》](03-upgrade_firmware.md)。

## Q：如何进行分区刷写？

**A：** 以下内容面向需要单独烧写分区（boot/kernel/rootfs 等）的进阶用户；仅需完整固件升级请参阅[《使用USB线缆升级固件》](03-upgrade_firmware.md)。

### Windows 操作系统

#### 烧写分区映像

烧写分区映像的步骤如下：

1. 切换至`Upgrade Firmware`页。
2. 勾选需要烧录的分区，可以多选。
3. 确保映像文件的路径正确，需要的话，点路径右边的空白表格单元格来重新选择。
4. 点击`Run`按钮开始升级，升级结束后设备会自动重启。

<center>

![AndroidTool 烧写工具界面](../../../rk3308_img/upgrade_firmware_androidtool_zh.png)
</center>

### Linux 操作系统

#### 烧写分区镜像

```shell
sudo upgrade_tool di -b /path/to/boot.img
sudo upgrade_tool di -r /path/to/recovery.img
sudo upgrade_tool di -m /path/to/misc.img
sudo upgrade_tool di -u /path/to/uboot.img
sudo upgrade_tool di -dtbo /path/to/dtbo.img
sudo upgrade_tool di -p paramater   # 烧写 parameter
sudo upgrade_tool ul bootloader.bin # 烧写 bootloader
```

如果因 flash 问题导致升级时出错，可以尝试低级格式化、擦除 emmc：

```shell
sudo upgrade_tool lf update.img	# 低级格式化
sudo upgrade_tool ef update.img	# 擦除
```

## Q：烧写时出现异常怎么办？

**A：**

### 烧写失败分析

如果烧写过程中出现Download Boot Fail, 或者烧写过程中出错，如下图所示，通常是由于使用的USB线连接不良、劣质线材，或者电脑USB口驱动能力不足导致的，请更换USB线或者电脑USB端口排查。

<center>

<img alt="" src="../../../rk3308_img/upgrade_downloadfail.png" width="800">
</center>


### 无法识别设备 / 提示发现新设备

如果有进行"进入Loader模式"的操作，仍旧没有看到烧写工具提示LOADER，通常是主机尚未识别设备或未装好驱动，处理方法已在上文《如何判断当前处于哪种升级模式》一节中说明（含发现新设备 `Rockusb Device` 及驱动安装的截图），此处不再重复。

如果升级失败，可以尝试先擦除 Flash 后再升级，方法见上文《如何进行分区刷写》一节。

## Q：如何进入 MaskRom 模式？

**A：** `MaskRom` 模式是设备变砖的最后一条防线。强行进入 `MaskRom` 涉及硬件操作，有一定风险，因此仅在设备进入不了 `Loader` 模式的情况下，方可尝试 `MaskRom` 模式。进入 `MaskRom` 的原理是人为的把 EMMC 的数据脚与地线短接，系统会认为 EMMC 数据出错，从而清除 EMMC 数据。

**请小心阅读，并谨慎操作！**

整机如需进入 MaskRom 模式，需拆开主机后在主板上找到对应的测试点进行短接，测试点位置以实际主板为准。

