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


