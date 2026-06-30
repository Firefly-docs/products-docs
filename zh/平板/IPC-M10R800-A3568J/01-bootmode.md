# 更新固件介绍


## 前言

AIO-3568J 有2种工作模式。一般情况下，开机直接进入`Normal 模式`正常启动系统。如需对板子系统进行升级，可以根据情况选择合适的`升级模式`进行固件升级。

* 注意：AIO-3568J出厂默认安装 Android 操作系统，如果用户要运行其他操作系统，需要使用对应的固件烧写到主板。可以前往[固件下载页面](https://www.t-firefly.com/doc/download/124.html)下载更多固件。


![](../../../rk356x_img/working_mode.png)

**AIO-3568J按工作模式进行分类，可以分为二种工作模式：**

## Normal 模式

| 工作模式  | Normal 模式 | 升级模式 |
| :--------: | :-------: | :------- |
| 启动介质 | eMMC 接口/SDMMC 接口| | √ |
| 描述 | Normal 模式就是正常的启动过程，<br />各个组件依次加载，正常进入系统。 | 目前支持3种升级模式，各有优缺点：<br />1. [MaskRom 升级模式](04-maskrom_mode.html)<br />2. [Loader 升级模式](loader_mode.html)<br />3. [SD 升级模式](05-upgrade_firmware_sd.html)|

## 升级模式



**其中升级模式中，不同升级模式之间的对比：**

| 升级模式  | [MaskRom 升级模式](04-maskrom_mode.html) | [Loader 升级模式](loader_mode.html) | [SD 升级模式](05-upgrade_firmware_sd.html) |
| :--------: | :------- | :------- | :------- |
| 简单描述 | 1. 使用USB线将主板连接到电脑上；<br />2. 硬件操作使板子进入升级模式；<br />3. 在PC上使用USB升级单板固件。  |  1. 使用USB线将主板连接到电脑上；<br />2. 软件或按键操作使板子进入升级模式；<br />3. 在PC上使用USB升级单板固件。 | 1.通过升级卡制作工具，将MicroSD卡制作为升级卡；<br />2. 将升级卡插入主板，上电开机，机器自动执行升级。|
| 连接方式 | USB | USB | TF卡（少数为SD卡槽） |
| 升级工具<br />Windows PC<br />Linux PC | <br />[Windows 上升级固件](Windows_upgrade_firmware.html)<br />[Linux 上升级固件](Linux_upgrade_firmware.html) | <br />[Windows 上升级固件](Windows_upgrade_firmware.html)<br />[Linux 上升级固件](Linux_upgrade_firmware.html) | <br />[Windows 上制作升级卡](05-upgrade_firmware_sd.html)<br />不支持 |
| 进入方法 | 需要硬件操作 |  按键或软件进入| 上电直接进入|
| 使用条件 | 硬件操作进入 |  能正常使用uboot| 无|
| 使用场景推荐 | 1. 当板子无法正常启动时候；<br />2. 在切换烧写Linux和Android固件的情况下。 |  1. 有完整uboot或能正常进入系统；<br />2. 需要单独烧写分区(uboot或boot分区等)。| 1. 工人操作方便，适合产品批量生产时候；<br />2. 产品定型后升级，方便最终客户操作。|
| 优点 | <font color = "red">1. 最基本的烧写方式；</font><br />2. 非固件和硬件问题，一般都能成功烧写；<br />3. 不需要uboot支持，拯救变砖的单板；<br />4. 支持跨系统升级(Linux和安卓等)。 | 1. 烧写效果MaskRom 升级模式差不多；<br />2. 能单独烧写分区；<br />3. 进入loader模式方便。 | 1. 操作方便，只需插卡启动；<br />2.集合了MaskRom 升级模式的优点。 |
| 缺点 | 1. 进入方式麻烦，不适难拆除外壳的产品；<br />2. 烧写分区表麻烦，较难单独烧写分区；<br />3. 需要完全擦除设备再烧写。| 1. 需要完整的loader(通常指uboot)；<br />2. 跨系统升级需要完全擦除设备再烧写。 | 1. 需要合成完整固件。 |



#### MaskRom 升级模式

一般情况下是不用进入 `MaskRom 升级模式`的，只有在 bootloader 校验失败（读取不了 IDB 块，或 bootloader 损坏） 的情况下，BootRom 代码 就会进入此模式。此时 BootRom 代码等待主机通过 USB 接口传送 bootloader 代码，加载并运行之。当板子变砖无法正常启动或升级程序时，也可以手动进入`MaskRom 升级模式`.

***要强行进入 `MaskRom 升级模式`，请参阅[《MaskRom 升级模式》](04-maskrom_mode.md)一章。***


#### Loader 升级模式

在 `Loader 升级模式`下，bootloader 会进入升级状态，等待主机命令，用于固件升级等。要进入此模式，必须让 bootloader 在启动时检测到 `RECOVERY`（恢复）键按下，且 USB 处于连接状态。

***要强行进入 `Loader 升级模式`，请参阅[《Loader 升级模式》](loader_mode.md)一章。***

#### SD 升级模式

使用SD升级，本质上是制作一个可启动的SD启动的升级固件，让板子SD启动，擦除和烧写EMMC。

***要强行进入 `SD 升级模式`，请参阅[《使用SD卡更新固件》](05-upgrade_firmware_sd.md)一章。***
