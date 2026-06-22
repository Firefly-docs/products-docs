  产品的开发主要有两部分：**固件开发**以及**固件升级**。 

固件开发即通过官方渠道获取固件的 SDK ，对系统固件进行二次开发

固件升级是指在对系统固件进行二次开发或者是想更新官方发布的固件的时候进行的操作。

## 固件升级

获取[固件](https://www.t-firefly.com/doc/download/183.html)后，可以通过 Type-C 数据线或者 TF-card 两种方式进行固件升级。

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

## More
可参考链接以获取更多开发以及玩法内容：
* [ROC-RK3588-PC](https://wiki.t-firefly.com/zh_CN/ROC-RK3588-PC/started.html)
* [StationPC 维基百科](https://wiki.stationpc.cn/docs/stationpc)
* [技术案例](https://www.t-firefly.com/doc/case.html)
