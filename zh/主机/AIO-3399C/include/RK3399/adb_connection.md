一般情况下`adb`是默认开启的，如果没有开启请执行以下步骤：
* AIO-3399C用Type-C数据线连接设备和主机;
* 根据当前Android版本勾选对应路径下的Connect to PC
   * Android7.1、Android8.1 选择 Setting -> USB，然后勾选 Connect to PC
   * Android10.0 选择 Setting -> Connected devices 然后勾选 Connect to PC

* 基于你的系统安装 adb 驱动和命令。

当设备端状态栏提示 `USB debugging connected` 时，便可进行调试：

```
adb devices
adb shell
```
