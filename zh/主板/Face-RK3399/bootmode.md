# 启动模式说明

## 前言

Face-RK3399 有灵活的启动方式。一般情况下，若非硬件损坏，Face-RK3399 开发板是不会变砖的。

如果在升级过程中出现意外，bootloader 损坏，导致无法重新升级，此时仍可以进入 `MaskRom` 模式来修复。

## 加载方式

RK3399 有 32KB 的 BootRom 和 200KB 的内部 SRAM，支持从以下设备加载系统：

* SPI 接口
* eMMC 接口
* SDMMC 接口

另外 Face-RK3399 支持从 USB OTG 接口下载系统代码。

## 启动次序

启动的次序是这样的：

1. 主控上电初始化
2. BootRom 代码在 SRAM 上运行，校验存储设备里的 bootloader
3. 校验通过，加载并运行 bootloader 引导代码
4. bootloader 引导代码负责初始化 DDR 内存，加载 bootloader 完整代码到 DDR 内存中并运行
5. bootloader 加载存储设备上的 Linux 内核，并将执行权交给 Linux 内核

## 启动模式

Face-RK3399 有三种启动模式：

* Normal 模式
* Loader 模式
* MaskRom 模式

### Normal 模式

Normal 模式就是正常的启动过程，各个组件依次加载，正常进入系统。

### Loader 模式

在 Loader 模式下，bootloader 会进入升级状态，等待主机命令，用于固件升级等。   
要进入 Loader 模式，必须让 bootloader 在启动时检测到 `RECOVERY`（恢复）键按下，且 USB 处于连接状态。使设备进入升级模式方法如下:

断开电源适配器

* USB数据线连接好设备的USB OTG接口和PC主机。
* 按住设备上的 RECOVERY （恢复）键并保持。
* 插上电源
* 大约两秒钟后，松开 RECOVERY 键。

### MaskRom 模式

MaskRom 模式用于 bootloader 损坏时的系统修复。

一般情况下是不用进入 MaskRom 模式的，只有在 bootloader 校验失败（读取不了 IDR 块，或 bootloader 损坏） 的情况下，BootRom 代码 就会进入 MaskRom 模式。此时 BootRom 代码等待主机通过 USB 接口传送 bootloader 代码，加载并运行之。   

***要强行进入 MaskRom 模式，请参阅[《MaskRom》](maskrom_mode.md)一章。***