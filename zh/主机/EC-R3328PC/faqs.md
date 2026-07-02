# FAQs



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


## 写号工具写入SN，MAC地址
<font color=red>**注意:**</font>如果开发板进行了eMMC擦除操作，之前写入的数据也会被清除。
### Windows方式
* 安装RKDevInfoWriteTool
    * [下载地址](https://www.t-firefly.com/doc/download/84.html#other_379)
* RKDevInfoWriteTool的**设置**里选中"RPMB"
* 根据需要在RKDevInfoWriteTool的**设置**里配置"SN"，"WIFI MAC"，"LAN MAC"，"BT MAC"等
* 开发板进入loader模式
* RKDevInfoWriteTool进行**写入**或者**读取**操作

具体操作可以参考RKDevInfoWriteTool安装目录下的《RKDevInfoWriteTool使用指南》PDF文档。
### Linux方式
开发板自身写号方式
* buildroot使能`BR2_PACKAGE_VENDOR_STORAGE`
* 通过vendor_storage命令进行读写操作
    * [下载地址](https://www.t-firefly.com/doc/download/84.html#other_379)
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