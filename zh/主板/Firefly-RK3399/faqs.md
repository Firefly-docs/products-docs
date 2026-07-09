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
Firefly-RK3399 默认录音输入源采用的是板载麦克风 `Builtin Mic`,可手动切换为phone录音`Wired Headset`:

1. 在 `Settgins apk` 里面找到 `Sound` 然后点击进去;
2. 点击 `Advanced` 后会出现 `Audio input`选项;
3. 选择`Wired Headset`

![](../../../rk3399_img/faqs_android_audio_input.png)

## 如何强行进入 MaskRom 模式

如果板子进入不了 Loader 模式，此时可以尝试强行进入 MaskRom 模式。操作方法见[《MaskRom模式》](04-maskrom_mode.md)。

## PCIE
* 开发板上的两个 PCIE 的区别?
  * 正面是 PCIe 1.0, M.2，背面的是 USB 转的 Mini PCIe。SSD、SATA 接 M.2，LTE/3G 接 Mini PCIe。

* M2 接口类型：
  * B-key

* 如何接 SSD：
  * 市面上的 SSD 基本是 M-key，如果要接 SSD，需要到[商城](https://store.t-firefly.com/goods.php?id=51)购买 B-key 转 M-key 的转接板

* SSD 支持 NVME 吗：
  * 支持，测试过英特尔（Intel）600P 系列，三星 EVO 系列

* 如何接 SATA 硬盘：
  * 如果要接 SATA，需要到[商城](https://store.t-firefly.com/goods.php?id=52)购买 PCIE 转 SATA 的转接板


## TYPE-C

* 怎样接显示器？
  * 需要转接头，目前我们这边测试过有转 DP 和 HDMI 的转接头：
  * Type-C 转 DP：[购买参考链接](https://detail.tmall.com/item.htm?id=531442057703&areaId=442000&user_id=1127317597&cat_id=2&is%20_b=1&rn=4eff2fc1aac30e8c67ff74ef5fe76b56)
  * Type-C 转 HDMI: [购买参考链接](https://item.taobao.com/item.htm?spm=a1z0d.6639537.1997196601.291.150IpW&id=540645282055&qq-pf-to=pcqq.temporaryc2c)
  * 类似的转 VGA 和 DVI 也可以，我们没有买过类似的转接头，客户可以自行验证

* 显示的最大分辨率
  * 默认最大为2K，如果需要4K，需要修改软件

* 显示 Linux 下可以吗
  * 目前只有 Android 支持，Linux 驱动原厂还在调试中

* 可以跟板载的 HDMI 同时显示吗
  * 可以同时显示

* 支持 OTG（UFP/DFP）吗
  * 支持，需要购买 3.0 的转接线,购买[参考链接](https://detail.tmall.com/item.htm?spm=a220o.1000855.w5003-14913680624.1.W2eSKK&id=528676463455&scene=taobao_shop&skuId=3173247769269)

## BT (蓝牙)

* 开发板支持蓝牙耳机吗？
  * 支持

## LTE/4G

* 目前支持的型号：
  * EC20

## Camera

* 支持双摄像头吗？
  * 支持，目前我们调试验证过的有 OV13850

* 支持 USB 摄像头吗？
  * 支持

* 最多可以支持多少个 USB 摄像头
  * 理论上板子上的 USB 口都支持接摄像头，但考虑到一个 hub 上只能挂一个 YUV 和编码的限制，两个是没有问题

* 支持两个 MIPI 摄像头，一个 DVP 摄像头同时用吗？
  * 不支持，由于软件架构问题，目前只支持最多两个同时用

## 风扇

* 支持速度控制吗？
  * 目前的硬件不支持，只支持检测运行状态

## RTC

* 开发板上电后时间不同步
  * 检查一下 RTC 电池是否正确接入。

* 支持定时开机吗
  * 支持，详细请看[wiki](http://wiki.t-firefly.com/zh_CN/Firefly-RK3399/driver_rtc.html)



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
    * [下载地址](http://www.t-firefly.com/doc/download/page/id/3.html#other_379)
* RKDevInfoWriteTool的**设置**里选中"RPMB"
* 根据需要在RKDevInfoWriteTool的**设置**里配置"SN"，"WIFI MAC"，"LAN MAC"，"BT MAC"等
* 开发板进入loader模式
* RKDevInfoWriteTool进行**写入**或者**读取**操作

具体操作可以参考RKDevInfoWriteTool安装目录下的《RKDevInfoWriteTool使用指南》PDF文档。
### Linux方式
开发板自身写号方式
* buildroot使能`BR2_PACKAGE_VENDOR_STORAGE`
* 通过vendor_storage命令进行读写操作
    * [下载地址](http://www.t-firefly.com/doc/download/page/id/3.html#other_379)
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