
## 切换录音输入源
ROC-RK3399-PC 默认录音输入源采用的是板载麦克风 `Builtin Mic`,可手动切换为phone录音`Wired Headset`:

1. 在 `Settgins apk` 里面找到 `Sound` 然后点击进去;
2. 点击 `Advanced` 后会出现 `Audio input`选项;
3. 选择`Wired Headset`

![](../../../rk3399_img/faqs_android_audio_input.png)

## ROC-RK3399-PC(AI) 的 Type-C 接口 OTG 模式不正常，接到 PC 端不能发现 ADB 设备？

因为 ROC-RK3399-PC(AI) 内置 NPU 模块，NPU 模块和 Type-C 接口使用 RK3399 同一组 USB2.0，**官方固件默认将这组 USB2.0 配置为 HOST 模式供 NPU 模块使用**，所以 Type-C 接口接到 PC 端不能发现 ADB 设备。

**官方固件默认将 USB3.0 接口配置为 OTG 模式**，ADB 调试需要接到 USB3.0 接口，参考 [ADB 使用](adb_use.html)。

ROC-RK3399-PC(AI) 和 ROC-RK3399-PC 默认的配置差异对比如下表：

|  | ROC-RK3399-PC(AI) | ROC-RK3399-PC |
|---|---|---|
| Type-C 接口 | HOST | OTG |
| USB3.0 接口 | OTG | HOST |

注意：**ROC-RK3399-PC(AI) 和 ROC-RK3399-PC 升级固件都需要接到 Type-C 接口。**

## ROC-RK3399-PC(AI) 选购了不带 NPU 的版本，如何将 Type-C 接口配置为 OTG 模式， USB3.0 接口配置为 HOST 模式？

在Android7.1 industry版本的源代码修改如下，其他平台可参考修改：

* 修改 usb 控制器

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

* 修改 usb 模式

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

示例固件下载链接：[AIO-3399C(AI) Android7.1 industry版本](https://pan.baidu.com/s/1O_eddDrb4NZpczEJUrw68Q)(提取码:1234)