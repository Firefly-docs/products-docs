# FAQs
## RK3399 HDMI 4K 分辨率
RK3399 出厂固件默认支持HDMI 4K分辨率输出，对应的UI是2K分辨率拉伸到4K方式，
而部分用户需要的是点对点的4K UI方式，关于4K UI相关问题如下：

1.确认是否一定要4K UI？

如果只是想要4K视频或是4K图片，那可以不需要配置 4K UI，系统默认的视频播放器和图片浏览器可以支持。 

2.如何配置4K UI？

把FrameBuffer 配置成4K，然后 HDMI分辨率确保设置成4K，修改方式如下：

android7.1/android8.1 
```
persist.sys.framebuffer.main=3840x2160@60 
```
android9.0 and after
```
persist.vendor.framebuffer.main=3840x2160@60 
```  

3.配置成 4K UI 之后出现闪屏？

修改方式可以参考[[文档]](https://pan.baidu.com/s/1CHEq05zUydKWtlDHz23-5A) 提取码：1234 



## 切换录音输入源
AIO-3399C 默认录音输入源采用的是板载麦克风 `Builtin Mic`,可手动切换为phone录音`Wired Headset`:

1. 在 `Settgins apk` 里面找到 `Sound` 然后点击进去;
2. 点击 `Advanced` 后会出现 `Audio input`选项;
3. 选择`Wired Headset`

![](../../../rk3399_img/faqs_android_audio_input.png)

## AIO-3399C(AI) 的 Type-C 接口 OTG 模式不正常，接到 PC 端不能发现 ADB 设备？

因为 AIO-3399C(AI) 内置 NPU 模块，NPU 模块和 Type-C 接口使用 RK3399 同一组 USB2.0，**官方固件默认将这组 USB2.0 配置为 HOST 模式供 NPU 模块使用**，所以 Type-C 接口接到 PC 端不能发现 ADB 设备。

**官方固件默认将 USB3.0 接口配置为 OTG 模式**，ADB 调试需要接到 USB3.0 接口，参考 [ADB 使用](adb_use.html)。

AIO-3399C(AI) 和 AIO-3399C 默认的配置差异对比如下表：

|  | AIO-3399C(AI) | AIO-3399C |
|---|---|---|
| Type-C 接口 | HOST | OTG |
| USB3.0 接口 | OTG | HOST |

注意：**AIO-3399C(AI) 和 AIO-3399C 升级固件都需要接到 Type-C 接口。**

## AIO-3399C(AI) 选购了不带 NPU 的版本，如何将 Type-C 接口配置为 OTG 模式， USB3.0 接口配置为 HOST 模式？

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

## 打开 Root 权限
Android 系统有很多很强大的功能都需要用到 root 权限，开发者经常在使用的时候遇到权限的问题，那如何在 Firefly 平台上开启系统的 root 权限功能呢？Firefly 已在系统添加启动 root 权限的功能，具体的步骤如下：

1. 在 `Settgins apk` 里面找到 `About device` 然后点击进去;
2. 点击 `Build number` 5次后会提示 (you are now a developer);
3. 然后返回上一级点击 `Developer options` 选项后，在选项中点击 `ROOT access` 就打开 root 权限功能。

![](../../../rk3399_img/Firefly-RK3399/faqs_android_root.png)

## 开机异常并循环重启怎么办？

有可能是电源电流不够，请使用电压为 12V，电流为 2.5A~3A 的电源。

## Ubuntu 默认的用户名和密码是什么？

* 用户名：`firefly`
* 密码：`firefly`
* 切换超级用户 `sudo -s`


## 如何支持RK3399K芯片？
RK3399K芯片最高主频可达2.016GHz,如果手上的硬件设备使用的是RK3399K芯片，需要支持RK3399K则手动加入下面补丁,以AIO-3399J一体机上支持RK3399K芯片为例：
```
#打补丁前首先需要确认SDK源码中是否存在 kernel/arch/arm64/boot/dts/rockchip/rk3399k-opp.dtsi 文件
#确认文件存在后在最终使用的dts中手动加入下面补丁，如下在 rk3399-firefly-aio.dts 中手动加入:
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

重新编译烧录固件后查看主频列表:

```
#小核最高频率1.512GHz
cat /sys/devices/system/cpu/cpufreq/policy0/scaling_available_frequencies
408000 600000 816000 1008000 1200000 1416000 1512000

#大核最高频率2.016GHz
cat /sys/devices/system/cpu/cpufreq/policy4/scaling_available_frequencies
408000 600000 816000 1008000 1200000 1416000 1608000 1800000 2016000
```

## Git链接地址？

[https://gitlab.com/TeeFirefly/FireNow-Nougat](https://gitlab.com/TeeFirefly/FireNow-Nougat)

## 哪里有 RK3399 芯片技术手册？

RK3399 芯片技术手册链接：[Brief](http://www.t-firefly.com/download/Firefly-RK3399/docs/Chip%20Specifications/Rockchip_RK3399_Datasheet_V0.7_20160219.pdf) [Part1](http://www.t-firefly.com/download/Firefly-RK3399/docs/TRM/Rockchip%20RK3399TRM%20V1.3%20Part1.pdf) [Part2](http://www.t-firefly.com/download/Firefly-RK3399/docs/TRM/Rockchip%20RK3399TRM%20V1.3%20Part2.pdf)


## 写号工具写入SN，MAC地址
<font color=red>**注意:**</font>如果开发板进行了eMMC擦除操作，之前写入的数据也会被清除。
### Windows方式
* 安装RKDevInfoWriteTool
    * [下载地址](http://www.t-firefly.com/doc/download/page/id/54.html#other_379)
* RKDevInfoWriteTool的**设置**里选中"RPMB"
* 根据需要在RKDevInfoWriteTool的**设置**里配置"SN"，"WIFI MAC"，"LAN MAC"，"BT MAC"等
* 开发板进入loader模式
* RKDevInfoWriteTool进行**写入**或者**读取**操作

具体操作可以参考RKDevInfoWriteTool安装目录下的《RKDevInfoWriteTool使用指南》PDF文档。
### Linux方式
开发板自身写号方式
* buildroot使能`BR2_PACKAGE_VENDOR_STORAGE`
* 通过vendor_storage命令进行读写操作
    * [下载地址](http://www.t-firefly.com/doc/download/page/id/54.html#other_379)
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

        其他可以根据`vendor_storage -h`提示进行操作。

应用程序如何读取，可以参考`buildroot/package/rockchip/vendor_storage/vendor_storage.c`里的`vendor_storage_read`函数。

## Ubuntu系统，如果插入耳机后，没有声音，该如何处理？

`Menu` -> `Multimedia` -> `PulseAudio Volume Control` -> `Configuration` -> 选择正在工作的声卡，关闭另一个声卡。

## Android 下如何让系统抓取 LOG？

`Settings(设置)` -> `About phone(关于手机)` -> 点击5下 `Build number(版本号)` -> `Developer options(开发者选项)` -> `Enable logging to save(启用日志保存)`。打开功能后，系统的 `storage` 根目录下就会生成 `.LOGSAVE` 文件夹，里面包括系统 logcat 和内核 kmsg。