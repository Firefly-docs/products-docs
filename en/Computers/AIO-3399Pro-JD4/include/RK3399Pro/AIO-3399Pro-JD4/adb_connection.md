## Use USB
When using `adb`, you need:
1. Go to Options->Developer Options on the development board and check the "USB Debugging" option
1. AIO-3399Pro-JD4Use Double male USB data cable to connect the device usb3.0 interface and the PC host;
1. host/device mode switch
    - Settings -> Connected devices -> Connect to PC

When the device status bar prompts `USB debugging connected`, you can debug:

`` `
adb devices
adb shell
`` `