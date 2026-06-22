# FAQS

## RK3399 HDMI 4K Resolution

RK3399 firmware default support HDMI 4K resolution output, corresponding UI is 2K resolution stretch to 4K mode,
However, some users need point-to-point 4K UI. 4K UI issue as follows:

1.Is it necessary to use 4K UI?

If you just want 4K video or 4K images, there is no need to configure 
4K UI, as the system's default video player and image browser can support.

2.How to configure 4K UI?

Configure the FrameBuffer to 4K, and then make sure the HDMI resolution is set to 4K, as follows:

android7.1/android8.1 
```
persist.sys.framebuffer.main=3840x2160@60 
```
android9.0 and after
```
persist.vendor.framebuffer.main=3840x2160@60 
```  

3.Flicker appears after configuring the 4K UI

For details, see the [[documentation]](https://drive.google.com/drive/folders/1Q8xOMXW2OUiu641FFFpxlyxe2Q319LYA?usp=sharing)



## Switch recording input source
AIO-3399C The default recording input source is an onboard microphone `Builtin Mic`,You can manually switch to phone recording `Wired Headset`:

1. Find `sound` in `settgins APK` and click it;
2. Click `advanced` and the `audio input` option will appear;
3. Select `wired header`

![](../../../rk3399_img/faqs_android_audio_input.png)

## AIO-3399C(AI) Type-C interface OTG mode is not normal, connected to the PC can not find ADB device?

Because AIO-3399C(AI) has built-in NPU module, NPU module and Type-C interface use the same set of USB2.0 as RK3399, **official firmware config this set of USB2.0 to HOST mode for NPU module to use by default**, So Type-C interface connected to the PC can not find ADB device.

**Official firmware config USB3.0 interface to OTG mode by default**. ADB debugging needs to connect to USB3.0 interface, refer to [ADB usage](adb_use.html).

The difference between AIO-3399C(AI) and AIO-3399C defaults is compared in the following table:

|  | AIO-3399C(AI) | AIO-3399C |
|---|---|---|
| Type-C | HOST | OTG |
| USB3.0 | OTG | HOST |

Note: **both AIO-3399C(AI) and AIO-3399C require Type-C interface to upgrade firmware.**

## AIO-3399C(AI) I have purchased the version without NPU. How to configure the Type-C interface to OTG mode and the USB3.0 interface to HOST mode?

The source code of Android7.1 industry release is modified as follows, other platforms can refer to the modification:

* Modifying the usb Controller

```
diff --git a/device/rockchip/rk3399/rk3399_firefly_aioc_ai/init.rk30board.usb.rc_for_ai b/device/rockchip/rk3399/rk3399_firefly_aioc_ai/init.rk30board.usb.rc_for_ai
index 4833e78..1a0cca9 100644
--- a/device/rockchip/rk3399/rk3399_firefly_aioc_ai/init.rk30board.usb.rc_for_ai
+++ b/device/rockchip/rk3399/rk3399_firefly_aioc_ai/init.rk30board.usb.rc_for_ai
@@ -26,7 +26,7 @@ on boot
     symlink /config/usb_gadget/g1/configs/b.1 /config/usb_gadget/g1/os_desc/b.1
     mount functionfs adb /dev/usb-ffs/adb uid=2000,gid=2000
     setprop sys.usb.configfs 1
-    setprop sys.usb.controller "fe900000.dwc3"
+    setprop sys.usb.controller "fe800000.dwc3"

 on property:sys.usb.config=none && property:sys.usb.configfs=1
     write /config/usb_gadget/g1/os_desc/use 0
```

* Modifying usb Mode

```
diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-aioc-ai.dtsi b/kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-aioc-ai.dtsi
index 097da18..36097de 100644
--- a/kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-aioc-ai.dtsi
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-aioc-ai.dtsi
@@ -218,7 +218,7 @@
 };

 &usbdrd_dwc3_0 {
-    dr_mode = "host";
+    dr_mode = "otg";
 };

 &usbdrd3_1 {
@@ -227,7 +227,7 @@
 };

 &usbdrd_dwc3_1 {
-    dr_mode = "otg";
+    dr_mode = "host";
 };

 &vdd_pcie3v3{
```

Sample firmware download link：[AIO-3399C(AI) Android7.1 industry](https://drive.google.com/drive/folders/1lACcVxo_UGDBGGeTjcbBmcYwLlkvwvEq?usp=share_link)
### Open Root permissions

There are many powerful functions of the Android system that require root permissions. Developers often encounter permissions problems when using them. How to enable the root permissions of the system on the Firefly platform? Firefly has added the function of starting root privileges in the system. The specific steps are as follows:

1. Find `About device` in `Settgins apk` and click into it;
2. After clicking on `Build number` 5 times, it will prompt (you are now a developer);
3. Then return to the previous level and click the option `Developer options`, and click `ROOT access` in the options to open the root authority function.

![](../../../rk3399_img/Firefly-RK3399/faqs_android_root.png)
## What should I do if the boot is abnormal and restarts cyclically?

It may be that the power supply current is not enough. Please use a power supply with a voltage of 12V and a current of 2.5A~3A.

## What is the default username and password for Ubuntu?

* Username: `firefly`
* Password: `firefly`
* Switch super user `sudo -s`


## How to support RK3399K chip?
The highest frequency of the RK3399K chip can reach 2.016GHz. If the hardware device in your hand uses the RK3399K chip and needs to support RK3399K, manually add the following patch. Take the RK3399K chip supported on the AIO-3399J all-in-one machine as an example:

```
#Before patching, first confirm whether there is a kernel/arch/arm64/boot/dts/rockchip/rk3399k-opp.dtsi file in the SDK source code
#After confirming that the file exists, manually add the following patch to the final dts, as follows, manually add the following in rk3399-firefly-aio.dts:

git diff
diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-aio.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-aio.dts
index 060e88d..14f9fb3 100644
--- a/kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-aio.dts
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-aio.dts
@@ -43,6 +43,7 @@
 /dts-v1/;

 #include "rk3399-firefly-aio.dtsi"
+#include "rk3399k-opp.dtsi"

 / {
        model = "AIO-3399J HDMI (Android)";
```

Check the main frequency list after recompiling and burning the firmware:

```
#Small core highest frequency 1.512GHz
cat /sys/devices/system/cpu/cpufreq/policy0/scaling_available_frequencies
408000 600000 816000 1008000 1200000 1416000 1512000

# Large core maximum frequency 2.016GHz
cat /sys/devices/system/cpu/cpufreq/policy4/scaling_available_frequencies
408000 600000 816000 1008000 1200000 1416000 1608000 1800000 2016000
```

## Git link address?

[https://gitlab.com/TeeFirefly/FireNow-Nougat](https://gitlab.com/TeeFirefly/FireNow-Nougat)

## Where is the RK3399 chip technical manual?

RK3399 chip technical manual link: [Brief](http://www.t-firefly.com/download/Firefly-RK3399/docs/Chip%20Specifications/Rockchip_RK3399_Datasheet_V0.7_20160219.pdf) [Part1](http://www. t-firefly.com/download/Firefly-RK3399/docs/TRM/Rockchip%20RK3399TRM%20V1.3%20Part1.pdf) [Part2](http://www.t-firefly.com/download/Firefly-RK3399/ docs/TRM/Rockchip%20RK3399TRM%20V1.3%20Part2.pdf)


## Write number tool to write SN, MAC address

<font color=red>**Note:**</font> If the eMMC erase operation is performed on the development board, the previously written data will also be cleared.

### Windows way
* Install RKDevInfoWriteTool
    * [Download link](http://en.t-firefly.com/doc/download/52.html#other_297)
* Select "RPMB" in **Settings** of RKDevInfoWriteTool
* Configure "SN", "WIFI MAC", "LAN MAC", "BT MAC", etc. in the **Settings** of RKDevInfoWriteTool as needed
* The development board enters loader mode
* RKDevInfoWriteTool performs **write** or **read** operations

For specific operations, please refer to the PDF document "RKDevInfoWriteTool User Guide" under the RKDevInfoWriteTool installation directory.

### Linux way

How to write the number of the development board itself

* Buildroot enable `BR2_PACKAGE_VENDOR_STORAGE`
* Read and write operations through the vendor_storage command
    * [Download link](http://en.t-firefly.com/doc/download/52.html#other_297)
     * SN
     ```shell
     vendor_storage -w VENDOR_SN_ID -t string -i cad895bedb8ee15f
     vendor_storage -r VENDOR_SN_ID -t hex -i /dev/null
     ```

     * LAN MAC
     ```shell
     vendor_storage -w VENDOR_LAN_MAC_ID -t string -i AABBCCDDEEFF
     vendor_storage -r VENDOR_LAN_MAC_ID -t hex -i /dev/null
     ```

        Others can be operated according to the prompt of `vendor_storage -h`.

For how to read the application, please refer to the `vendor_storage_read` function in `buildroot/package/rockchip/vendor_storage/vendor_storage.c`.

## On Ubuntu system, if there is no sound after plugging in headphones, what should I do?

`Menu` -> `Multimedia` -> `PulseAudio Volume Control` -> `Configuration` -> Select the sound card that is working and turn off the other sound card.

## How to make the system crawl LOG under Android?

`Settings (settings)` -> `About phone (about phone)` -> Click 5 times `Build number (version number)` -> `Developer options (Developer options)` -> `Enable logging to save Save)`. After the function is turned on, the folder `.LOGSAVE` will be generated under the root directory of the system `storage`, which includes the system logcat and kernel kmsg.