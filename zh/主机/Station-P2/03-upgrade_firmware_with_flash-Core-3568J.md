## Maskrom 模式
如果是通过 [《MaskRom模式》](04-maskrom_mode.md)  章节进入了MaskRom 模式（或设备异常，经过擦除设备进入Maskrom），而板子**同时**存在NOR flash和 EMMC 两种存储器，这个时候想要烧录固件，需要区分存储器：

### 固件下载到NOR flash
系统在Maskrom模式下**默认**将固件下载到NOR flash中，不过由于NOR flash 比较小，不足以装载系统镜像，所以一般只会烧写比较小的文件，如MiniLoaderAll.bin。如果不小心将整个固件下载到NOR flash中，会出现`下载固件失败`导致板子变砖,此时可以参考 [《MaskRom模式》](04-maskrom_mode.md),短接NOR flash的D0 和 GND重新进入Maskrom

### 固件下载到EMMC
若要将固件下载到EMMC中，需要先下载Loader,然后选择将存储器切换到EMMC。具体操作步骤如下：

1.先准备好 `MiniLoaderAll.bin`，一般存放在以下路径（需提前编译固件）：
```shell
#Android SDK
path/to/SDK/rockdev//MiniLoaderAll.bin

#Linux SDK
path/to/SDK/rockdev/MiniLoaderAll.bin
```

2.点击下载工具的`Advanced Function`，然后将准备好的`MiniLoaderAll.bin`下载到存储器中，见下图

![](../../../rk356x_img/ROC-RK3568-PC/load_emmc_with_flash01.png)

3.点击`List Storage`读取存储器，然后选中存储列表中的`Emmc`

![](../../../rk356x_img/ROC-RK3568-PC/load_emmc_with_flash02.png)

* `X` 表示设备不存在该存储器
* `0` 表示存在该存储器，但未选中
* `√` 表示存在该存储器并处于被选中状态

4.点击`Switch Storage`，可以看到存储列表中的`Emmc`状态就会从`0`切到`√`,表示选择将固件下载到EMMC

![](../../../rk356x_img/ROC-RK3568-PC/load_emmc_with_flash03.png)

5.为确保整个固件从EMMC中启动，我们需要点击`EraseAll`清空存储器

![](../../../rk356x_img/ROC-RK3568-PC/load_emmc_with_flash04.png)

6.点击下载工具的`Upgrade Firmware`，然后选择我们想要下载进EMMC中的固件进行升级

![](../../../rk356x_img/ROC-RK3568-PC/load_emmc_with_flash05.png)


