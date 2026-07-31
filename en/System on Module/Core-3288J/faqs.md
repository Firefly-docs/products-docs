# FAQ

## Abnormal boot and reboot cycle


It probably not enough supply current, please use the voltage of 12V, current is 2.5A ~ 3A power.


## username and password of Ubuntu

```bash
Username: root  Password: firefly
Username: firefly  Password: firefly
```

## Git link address

[https://bitbucket.org/T-Firefly/firefly-rk3288](https://bitbucket.org/T-Firefly/firefly-rk3288)

## MAC address burn

Users may change the MAC address of the AIO-3288J. Please use the dynamic library tool `RKTools/windows/UpgradeDllTool_v1.3.tar.gz` under the SDK to write the MAC address.

## Open Root permissions

The Android system has a lot of powerful functions that require root permissions. Developers often encounter problems with permissions when using it.

How to open the root function of the system on the Firefly platform? Firefly has added the function of starting root authority in the system. The specific steps are as follows:

1. Find About device in Settgins apk and click in it
2. Click Build number for 7 times and you are now a developer will be prompted.
3. Then, after clicking Developer options option on the previous level, click Enable ROOT to open the ROOT permissions function

![](../../../rk3288_img/faqs_android_root.png)



## AIO-3288J HDMI output and HDMI input

AIO-3288J has two HDMI port,one of which is HDMI output(under the USB port) and one is HDMI input.See the follow picture:

![](../../../rk3288_img/faqs_hdmiin-out.png)



## OTG port cannot use USB device

AIO-3288J has a double-layer USB interface, the bottom of the USB interface is an OTG interface. When using ADB, you need to check the settings in the Settings->USB->Connect PC. If you use other USB devices, you need to remove the check, otherwise the USB device can not be used.

