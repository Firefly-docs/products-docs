### USB 数据线 ADB

* AIO-3399C 用 Type-C 数据线连接设备 Type-C 口和主机
* AIO-3399C(AI) 用双端公头 USB 数据线连接设备蓝色 USB 3.0 口和主机，然后根据当前Android版本勾选对应路径下的Connect to PC
   * Android7.1、Android8.1 选择 Setting -> USB，然后勾选 Connect to PC
   * Android10.0 选择 Setting -> Connected devices 然后勾选 Connect to PC

![](../../../rk3399_img/AIO-3399C/adb_connection.jpg)

当设备端状态栏提示 `USB debugging connected` 时，便可进行调试：

```
adb devices
adb shell
```