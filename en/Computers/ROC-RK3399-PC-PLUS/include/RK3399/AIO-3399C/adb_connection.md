### USB ADB

* AIO-3399C connects device Type-C port and host with Type-C data cable.
* AIO-3399C(AI) use a dual male USB date cable connect host and USB3.0 of Mianboard, then check connect to PC in the corresponding path according to the current Android version.
    * For Android 7.1 and Android 8.1, select setting > USB, and then check connect to PC
    * For Android 10.0, select setting > connected devices, and then check connect to PC
	
![](../../../rk3399_img/AIO-3399C/adb_connection.jpg)

When the device-side status bar prompts `USB debugging connection', debugging can be carried out:

```
adb devices
adb shell
```