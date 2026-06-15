Under normal circumstances, `adb` is enabled by default. If it is not enabled, please perform the following steps:
* AIO-3399C Connect the device and host with Type-C data cable;
* Check connect to PC in the corresponding path according to the current Android version
    * For Android 7.1 and Android 8.1, select setting > USB, and then check connect to PC
    * For Android 10.0, select setting > connected devices, and then check connect to PC
* Install the adb driver and commands based on your system.

When the device status bar prompts `USB debugging connected`, you can debug:

`` `
adb devices
adb shell
`` `
