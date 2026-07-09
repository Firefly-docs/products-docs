# FAQ

## Abnormal boot and reboot cycle


It probably not enough supply current, please use the voltage of 5V, current is 2.5A ~ 3A power.


## username and password of Ubuntu

```bash
Username: root  Password: firefly
Username: firefly  Password: firefly
```

## Git link address

[https://bitbucket.org/T-Firefly/firefly-rk3288](https://bitbucket.org/T-Firefly/firefly-rk3288)

## MAC address burn

Users may change the MAC address of the Firefly-RK3288. Please use the dynamic library tool `RKTools/windows/UpgradeDllTool_v1.3.tar.gz` under the SDK to write the MAC address.

## Open Root permissions

The Android system has a lot of powerful functions that require root permissions. Developers often encounter problems with permissions when using it.

How to open the root function of the system on the Firefly platform? Firefly has added the function of starting root authority in the system. The specific steps are as follows:

1. Find About device in Settgins apk and click in it
2. Click Build number for 7 times and you are now a developer will be prompted.
3. Then, after clicking Developer options option on the previous level, click Enable ROOT to open the ROOT permissions function

![](../../../rk3288_img/faqs_android_root.png)


## How to Switch Microphones

In Firefly-RK3288, there are two sources of audio input. One is the little microphone on board, the other is the microphone in your headset. Android switches between them automatically by default.

You can modify the mode by writing different values to the mic_state node.Methods as below:

```bash
shell@firefly:/ # echo 1 > sys/class/es8323/mic_state/mic_state  //use board mic
shell@firefly:/ # echo 2 > sys/class/es8323/mic_state/mic_state  //use headphone mic
```

## About VGA Auto Probe

Firefly-RK3288 can identify the configuration of the VGA display. However, if failing to read EDID (Extended Display Identification Data) from the display, it will use 1080P resolution by default.You can enter the [Settings] -> [Display] -> [VGA Mode] to  select the switching mode to manually adjust the VGA resolution. <font color=#ff0000 size=2>The Firefly-RK3288-Reload does not support VGA output, but supports dual HDMI output.</font>

![](../../../rk3288_img/faqs_vga.png)

## Firefly-RK3288-Reload dual HDMI output and HDMI input

There are three HDMI ports on the Firefly-RK3288-Reload chassis, two of which are HDMI outputs and one is HDMI input Which HDMI1 RGB signal to HDMI, the main display device. HDMI2 RK3288 chip built-in HDMI, from the display device. HDMI1 default output resolution of 1080P, will not read the peripheral EDID information. HDMI2 can read peripheral EDID information, automatically select the matching output resolution. In the Android system, each HDMI output resolution can be manually adjusted in the system settings.

## Bluetooth voice calls and VoIP

Firefly-RK3288 and Firefly-RK3288-Reload does not support Bluetooth voice calls or VoIP.

## About the fan

The fan of Firefly-RK3288 operating voltage is 5V, the black power line of fan connect to FAN- of Firefly-RK3288, and the red power line of fan connect to FAN+ of Firefly-RK3288. the FAN- and FAN+ directly connected with the Firefly-RK3288's power supply module，so the fan can not be controlled by software.

![](../../../rk3288_img/faqs_fan.jpg)



