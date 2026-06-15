### USB ADB

* AIO-3399C connects device Type-C port and host with Type-C data cable.
* AIO-3399C(AI) use a dual male USB date cable connect host and USB3.0 of Mianboard, and also need to check `Settings` -> `USB` -> `Connect to PC`.

![](../../img/AIO-3399C/adb_connection.jpg)

When the device-side status bar prompts `USB debugging connection', debugging can be carried out:

```
adb devices
adb shell
```