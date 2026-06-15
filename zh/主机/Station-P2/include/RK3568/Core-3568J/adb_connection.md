### USB 数据线 ADB

* AIO-3399C 用 Type-C 数据线连接设备 Type-C 口和主机。
* AIO-3399C(AI) 用双端公头 USB 数据线连接设备蓝色 USB 3.0 口和主机,还需要在 AI0-3399C(AI) 勾上 ***设置 -> USB -> Connect to PC*** 。

![](../../img/AIO-3399C/adb_connection.jpg)

当设备端状态栏提示 `USB debugging connected` 时，便可进行调试：

```
adb devices
adb shell
```