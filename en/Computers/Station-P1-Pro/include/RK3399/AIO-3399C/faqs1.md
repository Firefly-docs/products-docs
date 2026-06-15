
## Switch recording input source
Station-P1-Pro The default recording input source is an onboard microphone `Builtin Mic`,You can manually switch to phone recording `Wired Headset`:

1. Find `sound` in `settgins APK` and click it;
2. Click `advanced` and the `audio input` option will appear;
3. Select `wired header`

![](../../../rk3399_img/faqs_android_audio_input.png)

## Station-P1-Pro(AI) Type-C interface OTG mode is not normal, connected to the PC can not find ADB device?

Because Station-P1-Pro(AI) has built-in NPU module, NPU module and Type-C interface use the same set of USB2.0 as RK3399, **official firmware config this set of USB2.0 to HOST mode for NPU module to use by default**, So Type-C interface connected to the PC can not find ADB device.

**Official firmware config USB3.0 interface to OTG mode by default**. ADB debugging needs to connect to USB3.0 interface, refer to [ADB usage](adb_use.html).

The difference between Station-P1-Pro(AI) and Station-P1-Pro defaults is compared in the following table:

|  | Station-P1-Pro(AI) | Station-P1-Pro |
|---|---|---|
| Type-C | HOST | OTG |
| USB3.0 | OTG | HOST |

Note: **both Station-P1-Pro(AI) and Station-P1-Pro require Type-C interface to upgrade firmware.**

## Station-P1-Pro(AI) I have purchased the version without NPU. How to configure the Type-C interface to OTG mode and the USB3.0 interface to HOST mode?

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