# FAQ

## Q：设备有哪些启动模式？

**A：** ROC-RK3568-PCSE有三种启动模式：

* Normal 模式
* Loader 模式
* MaskRom 模式

### Normal 模式

Normal 模式就是正常的启动过程，各个组件依次加载，正常进入系统。

### Loader 模式

在 Loader 模式下，bootloader 会进入升级状态，等待主机命令，用于固件升级等。

### MaskRom 模式

MaskRom 模式用于 bootloader 损坏时的系统修复。

一般情况下是不用进入 `MaskRom 模式`的，只有在 bootloader 校验失败（读取不了 IDB 块，或 bootloader 损坏） 的情况下，BootRom 代码 就会进入 `MaskRom 模式`。此时 BootRom 代码等待主机通过 USB 接口传送 bootloader 代码，加载并运行之。

## Q：如何判断当前处于哪种升级模式？

**A：** 如何确定板子当前处于哪种升级模式，我们可以通过工具去查看。

**Windows操作系统**

通过AndroidTool工具可以看到下方提示`Found One LOADER Device`
<center>

<img alt="" src="../../../rk356x_img/upgrade_firmware_androidtool_zh.png" width="800">
</center>

如果有进行"进入Loader模式"的操作，仍旧没有看到烧写工具提示LOADER，此时可以可以看一下Windows主机是否有提示发现新硬件并配置驱动。打开设备管理器，会见到新设备 `Rockusb Device` 出现，如下图。如果没有，可返回上一步重新[安装驱动](03-upgrade_firmware.md)。

<center>

<img alt="" src="../../../rk356x_img/upgrade_firmware_new_equipment.png" width="800">
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

如何让设备进入可升级模式，请参阅[进入升级模式的操作](03-upgrade_firmware.md)。

## Q：如何进行分区刷写？

**A：** 以下内容面向需要单独烧写分区（boot/kernel/rootfs 等）的进阶用户；仅需完整固件升级请参阅[《使用USB线缆升级固件》](03-upgrade_firmware.md)。

注意：**Linux SDK v1.2.4a** 及之后版本采用 extboot，烧写内核请使用 extboot.img 取代下文中所有的 boot.img（仅限 Linux，Android 请无视）

如何查看版本：
1. 版本格式为 vx.x.xx，例如 v1.2.4a
1. 固件文件名称中存在版本号(..._vx.x.xx_日期.img)
1. Buildroot 使用`cat /etc/version`获取版本(rk356x_linux_release_日期_vx.x.xx.xml)
1. Ubuntu 使用`ffgo version`获取版本(rk356x_linux_release_日期_vx.x.xx.xml)
1. SDK 中可以在 SDK 根目录通过命令查看：`ls -l .repo/manifests/rk356x_linux_release.xml`
1. 如果上述方法找不到格式为 vx.x.xx 的版本，说明是旧版本，不支持 extboot

**不要将 extboot.img 烧录进旧版本固件!**

除此之外，extboot ubuntu 还支持以安装包的形式更新内核，详情查看[Ubuntu 使用手册](/docs/software/os-guide/Ubuntu-Debian/ubuntu-debian)

### Windows 操作系统

#### 烧写分区映像

烧写分区映像的步骤如下：

1. 切换至`Upgrade Firmware`页。
2. 点击设备分区表按钮（Dev Partition），勾选需要烧录的分区，可以多选。
3. 确保映像文件的路径正确，需要的话，点路径右边的空白表格单元格来重新选择。
4. 点击`Run`按钮开始升级，升级结束后设备会自动重启。

<center>

![AndroidTool 烧写工具界面](../../../rk356x_img/upgrade_firmware_androidtool_zh.png)
</center>

### Linux 操作系统

#### 烧写统一固件

```shell
sudo upgrade_tool uf update.img
```

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

安卓 fastboot 烧写动态分区：

```shell
adb reboot fastboot # 进入bootloader
sudo fastboot flash vendor vendor.img
sudo fastboot flash system system.img
sudo fastboot reboot # 烧写成功后,重启
```

## Q：烧写时出现异常怎么办？

**A：**

### 烧写失败分析

如果烧写过程中出现Download Boot Fail, 或者烧写过程中出错，如下图所示，通常是由于使用的USB线连接不良、劣质线材，或者电脑USB口驱动能力不足导致的，请更换USB线或者电脑USB端口排查。
<center>

<img alt="" src="../../../rk356x_img/upgrade_downloadfail.png" width="800">
</center>

### 贴有 Spi Flash(Nor Flash) 的设备烧录异常

如果板子同时贴有 Spi Flash(Nor Flash) 和 eMMC时，当进入 MaskRom 后，需要先切换升级存储器再烧写，操作方法参阅[《切换升级设备》](03-upgrade_firmware_with_flash.md)。

### 无法识别设备 / 提示发现新设备

如果有进行"进入Loader模式"的操作，仍旧没有看到烧写工具提示LOADER，通常是主机尚未识别设备或未装好驱动，处理方法已在上文《如何判断当前处于哪种升级模式》一节中说明（含发现新设备 `Rockusb Device` 及驱动安装的截图），此处不再重复。

## Q：如何进入 MaskRom 模式？

**A：** `MaskRom` 模式是设备变砖的最后一条防线。强行进入 `MaskRom` 涉及硬件操作，有一定风险，因此仅在设备进入不了 `Loader` 模式的情况下，方可尝试 `MaskRom` 模式。进入 `MaskRom` 的原理是人为的把 EMMC 的数据脚与地线短接，系统会认为 EMMC 数据出错，从而清除 EMMC 数据。

**请小心阅读，并谨慎操作！**

