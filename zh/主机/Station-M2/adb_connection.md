使用 `adb`时，你需要：
1. ROC-RK3566-PC用Type-C 数据线连接设备和主机;
1. 基于你的主机系统安装 adb 驱动和命令。(后面会介绍到)

安卓系统还需要：
1. 在开发板上系统中打开设置(`Settings`) -> 关于平板电脑(`About tablet`) -> 连续点击7下版本号(`Build number`)
1. 在开发板上进入设置(`Settings`) -> 系统(`System`) -> 高级(`Advance`) -> 开发者选项(`Developer options`)，勾上 “USB 调试(`USB debugging`)” 选项(默认已勾选)，Root 授权(`Root access`)勾选带有ADB的选项;

当设备端状态栏提示 `USB debugging connected` 时，便可进行调试：

```
adb devices
adb shell
```
![](../../../rk356x_img/Station-M2/otg_interface.jpg)