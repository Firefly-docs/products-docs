使用 `adb`时，你需要：
1. ROC-RK3308B-CC-PLUS用Type-C 数据线连接设备和主机;
1. 基于你的系统安装 adb 驱动和命令。

当设备端状态栏提示 `USB debugging connected` 时，便可进行调试：

```
adb devices
adb shell
```
