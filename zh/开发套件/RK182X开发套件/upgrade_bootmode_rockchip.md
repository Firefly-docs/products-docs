# 启动模式

RK182X开发套件 出厂默认安装操作系统。如果需要运行其他操作系统，请从[固件下载页面](https://community.t-firefly.com/doc/download/369)获取对应固件。

如果升级过程中 bootloader 损坏，导致无法正常升级，仍可以进入 `MaskRom` 模式修复主板。
完整的升级操作请参阅[使用 USB 线升级固件](upgrade_firmware_rockchip.md)或[使用 SD 卡升级固件](upgrade_firmware_sd_rockchip.md)。

## 启动存储器

RK182X开发套件 从以下存储器加载系统：

* eMMC 接口
* SDMMC 接口

## 启动模式

RK182X开发套件 有两种启动模式：

* Normal 模式
* MaskRom 模式

### Normal 模式

Normal 模式就是正常的启动过程，各个组件依次加载，系统正常启动。


### MaskRom 模式

MaskRom 模式用于烧写固件，或在 bootloader 损坏时修复系统。对于本开发套件，USB 升级始终使用主板上的 `MaskRom` 按键进入该模式。

硬件操作方法请参阅 [MaskRom 模式](upgrade_maskrom_mode_rockchip.md)。