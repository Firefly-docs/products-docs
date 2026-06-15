# 切换升级存储器

## 前言

本文介绍当主机同时存在 eMMC(默认) 和 NOR Flash 这两种存储器的时候，在烧录固件的时候，需根据以下的启动模式和情况来升级固件。

如下图，则设备有贴 Nor Falsh 存储器。

![](../../img/ROC-RK3566-PC/nor_flash-position.jpg)

## Loader 模式 

如果是通过 软件执行 `reboot loader` 或者 通过硬件Recovery 按键进入的Loader 下载模式的话，可不需要看接下来的内容，直接跳到 [《使用USB线缆升级固件》](03-upgrade_firmware.md) 操作。



## Maskrom 模式
如果是通过 [《MaskRom模式》](04-maskrom_mode.md)  章节进入了Maskrom 模式（或设备异常，经过擦除设备进入Maskrom），而板子**同时**存在NOR flash和 eMMC 两种存储器，这个时候想要烧录固件，需要区分存储器：

### 固件下载到NOR flash
系统在Maskrom模式下**默认**将固件下载到NOR flash中，不过由于NOR flash 比较小，不足以装载系统镜像，所以一般只会烧写比较小的文件，如MiniLoaderAll.bin。如果不小心将整个固件下载到NOR flash中，会出现`下载固件失败`的现象, 此时大家可以参考[FAQ](03-upgrade_firmware_with_flash.html#maskrom-mo-shi-shao-xie-shi-bai)去处理



### 固件下载到eMMC
有两种方法可以将固件烧写进eMMC中，一种是RK原厂提供的烧写方法，需要烧写`MiniLoaderAll.bin`并切换存储器；另一种是Firefly为了方便大家烧写，提供的一种参考烧写方法，该方法操作上无需要切换存储器，也可以使用Linux端的烧写工具upgrade_tool进行烧写
#### 方法一（Firefly）

<font color="red">此方法仅在官方最新SDK编译或者最新的官方固件中可用。</font>如果在Maskrom模式下不慎将固件下载到NOR flash中，导致重启后无法正常启动， 那么可以进入[《MaskRom模式》](04-maskrom_mode.md)，烧录最新SDK编译或者官方提供的固件；不管烧录成功还是失败，当机器重启后，假如NOR flash 存在数据则自动擦除，擦除时间大概30~60s，擦除成功后，自动进入Loader模式，之后直接[烧录固件](03-upgrade_firmware.html#shao-xie-gu-jian)即可。

#### 方法二（原厂）

若要将固件下载到eMMC中，需要先下载`MiniLoaderAll.bin`, 然后选择将存储器切换到eMMC。具体操作步骤如下：

1.先准备好 `MiniLoaderAll.bin`，一般存放在以下路径（需提前编译固件）：
```shell
#Android SDK
path/to/SDK/rockdev/Image-rk3566_roc_pc/MiniLoaderAll.bin

#Linux SDK
path/to/SDK/rockdev/MiniLoaderAll.bin
```

 
如果还没有编译SDK，可以从以下链接下载 一个对应CPU型号的[MiniLoaderAll.bin](https://www.t-firefly.com/share/index/index/id/4e86cc234c3b26e2b12084e383074ada.html)。


2.点击下载工具的`Advanced Function`，然后将准备好的`MiniLoaderAll.bin`下载到存储器中，见下图

![](../../img/Core-3568J/load_emmc_with_flash01.png)

3.点击`List Storage`读取存储器，此时存储列表选中的是`SPINOR`，为确保Nor flash为空，我们选择`EraseAll`擦除

![](../../img/Core-3568J/maskrom_erease_spinor_flash.png)

* `X` 表示设备不存在该存储器
* `0` 表示存在该存储器，但未选中
* `√` 表示存在该存储器并处于被选中状态


4.在存储列表中选中`Emmc`后点击`Switch Storage`

 ![](../../img/Core-3568J/load_emmc_with_flash02.png)

可以看到存储列表中的`Emmc`状态就会从`0`切到`√`,表示选择将固件下载到eMMC

![](../../img/Core-3568J/load_emmc_with_flash03.png)

5.点击`EraseAll`擦除

![](../../img/Core-3568J/load_emmc_with_flash04.png)

6.点击下载工具的`Upgrade Firmware`，然后选择我们想要下载进eMMC中的固件进行升级

![](../../img/Core-3568J/load_emmc_with_flash05.png)



## FAQ
### Maskrom模式烧写失败

![](../../img/Core-3566JD4/maskrom_download_failed_with_flash.png)

该现象是因为固件直接烧写进NOR flash中引起的，出现该现象时大家可以根据当前的情况进行处理：

**情况一**：*下载失败时没有断电或重启*

可以直接点击`EraseFlash`擦除NOR flash

![](../../img/Maskrom_EraseFlash.png)

之后按照章节[固件下载到eMMC](03-upgrade_firmware_with_flash.html#gu-jian-xia-zai-dao-emmc)的步骤进行固件升级即可


**情况二**：*有进行过重启或断电的操作*

通过Recovery按键进入Loader模式, 然后点击AndroidTool的`Go maskrom`进入Maskrom模式

![](../../img/Go_maskrom.png)

之后按照章节[固件下载到eMMC](03-upgrade_firmware_with_flash.html#gu-jian-xia-zai-dao-emmc)的步骤进行固件升级即可。若不清楚如何通过Recovery按键进入Loader模式请参考[使用USB线缆升级固件](03-upgrade_firmware.html#lian-jie-she-bei)章节


如果大家有接串口，可以查看一下log信息，其中`Bootdev(atags): mtd 2`说明系统启动到NOR flash存储器的uboot中

```shell
Bootdev(atags): mtd 2
GUID Partition Table Header signature is wrong: 0xA9425BF5A94153F3 != 0x5452415020494645
*** ERROR: Can't write GPT header ***
Backup GPT repair fail!
PartType: EFI
...
Device 0: unknown device
No ethernet found.
missing environment variable: pxeuuid
...
Retrieving file: pxelinux.cfg/default
No ethernet found.
Config file not found
No ethernet found.
No ethernet found.
=>
```

我们可以直接在这个模式擦除NOR flash
```shell
=> mtd erase nor0
Erasing 0x00000000 ... 0x01ffffff (8192 eraseblock(s))
=> reboot
```

也可以通过rbrom回到Maskrom模式
```shell
=> rbrom
```

进入到Maskrom模式后，固件升级都需要按照[固件下载到eMMC](03-upgrade_firmware_with_flash.html#gu-jian-xia-zai-dao-emmc)的步骤进行操作

### Go Maskrom无效
点击AndroidTool的`Go Maskrom`无法进入Maskrom模式，rbrom命令同样也不行，原因是部分旧固件不支持从NOR flash的bootloader回到Maskrom模式，此时可通过Recovery按键重新进入Loader模式，点击AndroidTool的`EraseFlash`进行擦除

![](../../img/Loader_EraseFlash.png)

无论是否提示`擦除IDB失败`都重启一下板子，重启后可进入Maskrom模式

![](../../img/EraseFlash_IDB_failed.png)

### 其它
烧写异常问题可参考帖子[rk3566/rk3568 烧写异常问题](https://dev.t-firefly.com/forum.php?mod=viewthread&tid=104417&page=1&extra=#pid265132)，若遇到其它问题也可以到社区论坛对应的板块[发帖](https://dev.t-firefly.com/forum.php)

