# 启动模式说明

## 前言 	

AIO-3128C 有灵活的启动方式。一般情况下，除非硬件损坏，AIO-3128C 开发板是不会变砖的。  
如果在升级过程中出现意外，bootloader 损坏，导致无法重新升级，此时仍可以进入 MaskRom 模式来修复。  

## 加载方式  

AIO-3128C 有 20KB 的 BootRom 和 100KB 的内部 SRAM，支持从以下设备加载系统：

* 8位 Async Nand Flash  
* 8位 toggle Nand Flash  
* SPI 接口  
* eMMC 接口  
* SDMMC 接口  

也就是说，除了支持从 Nand Flash、SPI Flash、eMMC Flash 启动外，还支持 SD 卡启动。  
另外 AIO-3128C 支持从 USB OTG 接口下载系统代码。  

## 启动次序
启动的次序是这样的：

* 主控上电初始化  
* BootRom 代码在 SRAM 上运行，校验存储设备里的 bootloader  
* 校验通过，加载并运行 bootloader 引导代码  
* bootloader 引导代码负责初始化 DDR 内存，加载 bootloader 完整代码到 DDR 内存中并运行  
* bootloader 加载存储设备上的 Linux 内核，并将执行权交给 Linux 内核

## 启动模式  

AIO-3128C 有三种启动模式： 

* Normal 模式  
* Loader 模式  
* MaskRom 模式  

### Normal 模式  

Normal 模式就是正常的启动过程，各个组件依次加载，正常进入系统。
### Loader 模式

在 Loader 模式下，bootloader 会进入升级状态，等待主机命令，用于固件升级等。
要进入 Loader 模式，必须让 bootloader 在启动时检测到 RECOVERY（恢复）键按下，且 USB 处于连接状态：  

+ 一种方法是设备先断开电源适配器和双公头USB数据线的连接：  
    - 双公头USB数据线的一端连接主机。  
    - 按住设备上的 RECOVERY （恢复）键并保持。  
    - 双公头USB数据线的另一端连接设备。  
    - 大约两秒钟后，松开 RECOVERY 键。  
+ 另一种方法，无需断开电源适配器和双公头USB数据线的连接：  
    - 用双公头USB数据线连接好设备和主机。  
    - 按住设备上的 RECOVERY （恢复）键并保持。  
    - 短按一下 RESET（复位）键。  
    - 大约两秒钟后，松开 RECOVERY 键。  

### MaskRom 模式  
MaskRom 模式用于 bootloader 损坏时的系统修复。  
一般情况下是不用进入 MaskRom 模式的，只有在 bootloader 校验失败（读取不了 IDR 块，或 bootloader 损坏） 的情况下，BootRom 代码 就会进入 MaskRom 模式。此时 BootRom 代码等待主机通过 USB 接口传送 bootloader 代码，加载并运行之。  
要强行进入 MaskRom 模式，请参阅[《MaskRom》](maskrom_mode.md)一章。  