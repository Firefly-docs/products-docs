## USB方式
使用 `adb`时，你需要：
1. 在开发板上进入选项->开发人员选项，勾上 “USB 调试” 选项
1. ROC-RK3328-PC用Type-C数据线连接设备和主机;
1. host/device模式切换
    - 设置->Connected devices(已关联的设备)->连接到PC

当设备端状态栏提示 `USB debugging connected` 时，便可进行调试：

```
adb devices
adb shell
```