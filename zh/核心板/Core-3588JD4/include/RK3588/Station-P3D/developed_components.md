产品的开发主要有两部分：**固件开发**以及**固件升级**。 

固件开发即通过官方渠道获取固件的 SDK ，对系统固件进行二次开发。

固件升级是指在对系统固件进行二次开发或者是想更新官方发布的固件的时候进行的操作。

## 固件升级

获取[固件](ZH_DOW_LINK)后，可以通过 Type-C 数据线或者 TF-card 两种方式进行固件升级。

首先推荐的是使用 TF-card 进行升级，因为如果使用  Type-C 数据线进行升级就需要使用螺丝刀把主板卸下来后才可以进行升级。

升级的步骤可以参考 [ROC-RK3588-PC](https://wiki.t-firefly.com/zh_CN/ROC-RK3588-PC/upgrade_bootmode.html) 的固件升级章节：

* [TF-card 升级](https://wiki.t-firefly.com/zh_CN/ROC-RK3588-PC/upgrade_firmware_sd.html)
* [Type-C 数据线升级](https://wiki.t-firefly.com/zh_CN/ROC-RK3588-PC/upgrade_firmware.html)

 的 Type-C 数据线接口以及 TF-card 卡槽位置位于：

xxx 图片


xxx 图片

## 固件开发

进行固件开发主要有 4 个步骤：
* 获取 SDK 源码（Android/Linux）
* 修改 SDK 源码
* 编译 SDK 源码
* 进行编译后的固件升级

以上关于开发的步骤都可参考 ROC-RK3588-PC 的固件开发章节，两者的内容是一致的：
* [Android](https://wiki.t-firefly.com/zh_CN/ROC-RK3588-PC/android_adb_use.html)
* [Linux](https://wiki.t-firefly.com/zh_CN/ROC-RK3588-PC/linux_compile.html)
* [设备接口开发](https://wiki.t-firefly.com/zh_CN/ROC-RK3588-PC/usage_adc.html)

## 模块扩展

 支持多种不同模块（扩展板）的接入，以便于扩展多种不同的功能，适配各种应用场景。官方的[固件](https://www.t-firefly.com/doc/download/290.html)是**可以同时兼容**不同的模块扩展场景的。

下列为官方已支持的模块扩展版本。（如需自定义客制化的模块，可联系商务：**sales@t-firefly.com**）

### 默认版本

默认版本的模块可扩展出以下接口：
* 1 x VGA 
* 3 x USB3.1 
* 1 x SPDIF（音频光纤）
* 1 x 千兆网口
* 1 x 4G LTE（带 SIM 卡座）
* 1 x SATA 2.0 


XXXX 接口图片

## More
可参考链接以获取更多开发以及玩法内容：
* [ROC-RK3588-PC](https://wiki.t-firefly.com/zh_CN/ROC-RK3588-PC/started.html)
* [StationPC 维基百科](https://wiki.stationpc.cn/docs/stationpc)
* [技术案例](https://www.t-firefly.com/doc/case.html)
