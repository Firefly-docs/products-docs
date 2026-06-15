# FAQs

## HDMI 无法 4K 显示？

AIO-3399ProC 默认出厂固件是支持 LVDS+HDMI 1080P 的双屏显示，HDMI 分辨率最高只能到 1080P。HDMI 要支持到 4K 分辨率需要重新烧写[默认固件](http://www.t-firefly.com/doc/download/31.html#other_80)，或者重新编译内核 `make ARCH=arm64 rk3399pro-firefly-aioc.img` ，重新烧写 `resource.img` 即可。

## 如何确认固件是否支持 4K？

AIO-3399ProC 打开`设置`->`显示` ,显示设备处若出现 HDMI 和 DSI，代表固件是支持双屏显示的，不支持 4K。若只有 HDMI 则是默认固件，可以支持 4K。

## 如何调整HDMI输出分辨率？

AIO-3399ProC 的 HDMI 能自动识别显示的分辨率。假如无法读取显示器的 EDID (Extended Display Identification Data, 扩展显示器标识数据) ，HDMI 会默认设置为 1080P 的分辨率。你也可以进入系统设置对 HDMI 分辨率进行手动调整。

![](../../../rk3399_img/faqs_setting_resolution.jpg)

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


## 哪里有 RK3399Pro 芯片技术手册？

K3399Pro 芯片技术手册链接：[Part1](https://download.t-firefly.com/product/RK3399/Docs/Chip%20Specifications/RK3399Pro-TRM-V1.0/Rockchip%20RK3399Pro%20TRM%20V1.0%20Part1-20190122.pdf)  [Part2](https://download.t-firefly.com/product/RK3399/Docs/Chip%20Specifications/RK3399Pro-TRM-V1.0/Rockchip%20RK3399Pro%20TRM%20V1.0%20Part2-20190122.pdf)

## Android 系统如何关闭音频？

针对无 codec 模块的用户，若不关闭音频相关配置，内核是会一直报异常 log 信息。关闭音频方法有两种：

1、在源码中 disable audioserver：

```
--- a/vendor/rockchip/common/device-vendor.mk
+++ b/vendor/rockchip/common/device-vendor.mk
@@ -121,6 +121,8 @@ ifeq ($(strip $(TARGET_BOARD_PLATFORM)), rk3399pro)
 $(call inherit-product-if-exists, vendor/rockchip/common/npu/npu_transfer.mk)
  endif

   -$(call inherit-product-if-exists, vendor/rockchip/common/tinyalsa/tinyalsa.mk)

    $(call inherit-product-if-exists, vendor/rockchip/common/pppoe/pppoe.mk)
```

2、直接删除 `udio.primary.default.so`：

```
out/target/product/rk3399pro_firefly_aiojd4/vendor/lib64/hw/audio.primary.default.so
```

## 写号工具写入SN，MAC地址
<font color=red>**注意:**</font>如果开发板进行了eMMC擦除操作，之前写入的数据也会被清除。
### Windows方式
* 安装RKDevInfoWriteTool
    * [下载地址](http://www.t-firefly.com/doc/download/page/id/76.html#other_379)
* RKDevInfoWriteTool的**设置**里选中"RPMB"
* 根据需要在RKDevInfoWriteTool的**设置**里配置"SN"，"WIFI MAC"，"LAN MAC"，"BT MAC"等
* 开发板进入loader模式
* RKDevInfoWriteTool进行**写入**或者**读取**操作

具体操作可以参考RKDevInfoWriteTool安装目录下的《RKDevInfoWriteTool使用指南》PDF文档。
### Linux方式
开发板自身写号方式
* buildroot使能`BR2_PACKAGE_VENDOR_STORAGE`
* 通过vendor_storage命令进行读写操作
    * [下载地址](http://www.t-firefly.com/doc/download/page/id/76.html#other_379)
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