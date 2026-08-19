# 烧写 SD 卡

下面我们将介绍如何烧写固件到 SD 卡。关于固件的类型说明可以看[这里](getting_started.md#firmware-format)。

以下是支持的系统列表：

- Android 7.1.2
- Android 8.1.0
- Ubuntu 18.04
- Ubuntu 16.04
- Debian 9
- LibreELEC 9.0

## 准备 SD 卡

作为启动盘，一张**优质可靠、高速读写**的 SD 卡，对于系统的稳定性来说是非常关键。现将 [《如何准备一张 SD 卡(英文)》](https://docs.armbian.com/User-Guide_Getting-Started/#how-to-prepare-a-sd-card)一文的要点摘录如下：

- 遇到启动或稳定性问题，有超过 95% 的可能是电源供应不足或 SD 卡问题（坏卡，坏读卡器，烧写SD卡时出错，卡读写太慢）。
- 推荐使用 Class 10 以上的 SD 卡，并使用专业工具去测试是否真卡。
- 尽量使用带校验的烧写工具。
- 如需将 SD 卡恢复至出厂设置，使用 [SD Formatter](https://www.sdcard.org/downloads/formatter_4/) 工具做格式化。
- 选择以下优质 SD 卡：

    ![](../../../rk3328_img/ROC-RK3328-CC/sdcard-samsung-1.png) ![](../../../rk3328_img/ROC-RK3328-CC/sdcard-sandisk-1.png) ![](../../../rk3328_img/ROC-RK3328-CC/sdcard-transcend-1.png)

## 下载固件

- [固件下载页面](https://community.t-firefly.com/doc/download/34)

**固件说明：固件分为[原始固件]与[RK 固件]，已经分类到不同的文件夹中，固件命名的日期最新则为最新的固件，其稳定性更好，请根据你所需要的固件类型选择正确的烧写工具。**

## 烧写工具

请根据所用主机的操作系统选择相应的烧写 SD 卡工具：

- 烧写原始固件
    + 图形界面烧写工具：
        * [SDCard Installer] (Linux/Windows/Mac)
        * [Etcher] (Linux/Windows/Mac)
    + 命令行烧写工具：
        * [dd] (Linux)

- 烧写RK固件
    + 图形界面烧写工具：
        * [SD Firmware Tool] (Windows)

<a id="sdcard-installer"></a>
### SDCard Installer

烧写[原始固件]，最轻松的方式就是使用官方的 [SDCard Installer]，它基于 Etcher / Rock64 Installer 定制，实现了一站式的固件选择和烧录操作，让烧写工作变成轻松简单。

[SDCard Installer] 节省了搜索开发板可用固件的时间。你只需要选择开发板、操作系统，插入 SD 卡，点击烧写按钮即可完成整个写卡工作，实在简单方便。

**安装使用说明**：

1. 到 [下载页面]去下载[SDCard Installer]。
2. 安装运行：
    + Windows： 解压后运行安装程序，按照提示安装到系统，之后在开始菜单里找到 [SDCard Installer]，并**以管理员身份**打开。
    + Linux： 解压后运行其中的 `.AppImage` 文件即可。
    + Mac： 直接双击 `.dwg` 文件，拖动安装到系统或直接运行。
3. 点击 "Choose an OS" 按钮， 在 "Please select your device" 组合框中选择 "ROC-RK3328-CC"。
4. 可用的固件列表将从网络更新，如下图所示：
    ![](../../../rk3328_img/ROC-RK3328-CC/started_sdcard-installer.png)
5. 选择所需的操作系统和版本，并点击 "OK" 按钮确认。另外也可以从文件管理器中选择本地的一个固件文件，拖放到 `SDCard Installer`。
6. 插入 SD 卡，工具应该会自动选中该卡；如果插有多张 SD 卡，可以点击 "Change" 按钮进行选择。
7. 点击 "Flash!" 按钮，开始固件下载、烧写和校验，请耐心等待。
    ![](../../../rk3328_img/ROC-RK3328-CC/started_sdcard-installer_flashing.png)

**注意事项**：

- [SDCard Installer] 在 Windows 下运行需要管理员权限，请用鼠标右键点击应用图标，在弹出菜单中选择 **以管理员身份运行**。
- 有时，当进度达到 99％ 或 100％ 时，可能会出现卸载 SD 卡的错误，这可以忽略，并且不会损坏烧写到 SD 卡的数据：
    ![](../../../rk3328_img/ROC-RK3328-CC/started_sdcard-installer_umount_fail.png)
- 在线下载的固件会缓存到本地目录，下次烧写时不用重新下载。缓存目录可以点击左下角的设置按钮，在 "Download Location:" 处设置。

<a id="etcher"></a>
### Etcher

[Etcher] 与 [SDCard Installer] 相比，少了固件选择的集成，但代码比较新。如果 [SDCard Installer] 在烧写 SD 卡中出错，或有什么问题，可以尝试使用 [Etcher] 去烧写，此时直接使用 [SDCard Installer] 缓存目录里的固件即可。

[Etcher] 可以到[Etcher 官网](https://etcher.io)去下载，安装和使用过程与 [SDCard Installer] 比较类似，这里就不再重复。

<a id="dd"></a>
### dd

[dd] 是 Linux 下常用的命令行工具，它适用于烧写[原始固件]。

首先，插入SD卡，如果被文件管理器自动挂载，则先将其卸载。

然后通过检查内核的日志查找 SD 卡的设备文件：
> dmesg | tail

如果设备文件为 `/dev/mmcblk0`，使用 `dd` 命令去烧录：
> sudo dd if=/path/to/your/raw/firmware of=/dev/mmcblk0 conv=notrunc

烧写一般所需时间较长，但上面的命令不会显示烧写进度，只能一直等待命令的完成。此时，我们可以使用另一个工具 `pv` 去实现进度条的显示。

安装 `pv`：
> sudo apt-get install pv

然后利用管道操作显示烧写进度：
> pv -tpreb /path/to/your/raw/firmware | sudo dd of=/dev/mmcblk0 conv=notrunc

<a id="sd-firmware-tool"></a>
### SD Firmware Tool

**注意**：以下介绍的是如何将[RK 固件]烧写到 SD 卡。

首先，到[SD Firmware Tool 下载](http://download.t-firefly.com/product/RK3328/Tools/SD_Firmware_Tool/SD_Firmware_Tool1.56.zip)去下载 `SD_Firmware_Tool`，并解压。

运行 `SD_Firmware_Tool.exe`:

![](../../../rk3328_img/ROC-RK3328-CC/sdfirmwaretool.zh_CN.png)

1. 插入 SD 卡。
2. 从组合框中选择 SD 卡对应的设备。
3. 勾选 "SD启动" 选项。
4. 点击 "选择固件" 按钮，在文件对话框中选择[RK 固件]。
5. 点击 "开始创建" 按钮。
6. 然后会显示警告对话框，选择 "是" 来确保选择了正确的SD卡设备。
7. 等待操作完成，直到提示成功对话框出现：

    ![](../../../rk3328_img/ROC-RK3328-CC/sdfirmwaretool_done.zh_CN.png)

8. 拔出 SD 卡。

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