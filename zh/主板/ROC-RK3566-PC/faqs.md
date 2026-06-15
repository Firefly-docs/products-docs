# FAQs

## kernel 编译弹出 “IO-Domain Checklist” 对话框
kernel 新建了 dts 文件 `arch/arm64/boot/dts/rockchip/rk3566-roc-pc-DEMO.dts`，编译 kernel 时弹出如下对话框
![](../../img/faq_rk356x_io-domain_checklist.png)

<br>

**先进行如下操作，再编译 kernel 。**       
```
cp arch/arm64/boot/dts/rockchip/.rk3566-roc-pc.dtb.dts.tmp.domain arch/arm64/boot/dts/rockchip/.rk3566-roc-pc-DEMO.dtb.dts.tmp.domain
```

<font color=red>**切记：不要轻易修改主控电源域的IO电平，否则，最坏的情况可能会导致IO的损坏。**</font>


## ROC-RK3566-PC SPK和MIC的使用问题
如果需要使用ROC-RK3566-PC来调试 SPK 和 MIC功能，可以使用外部扩展出来的IO口进行调试，软件默认支持，由于硬件信号是直接从codec引出来的，所以SPK只需接上喇叭即可，MIC这部分需要增加偏置电路才可以录音，如下

![](../../img/ROC-RK3566-PC/roc-rk3566-pc-spk_mic.png)

<br>

**注意：** 硬件版本 <font color=red>ROC-RK3566-PC V1.2 2022.01.22</font> 的 MIC这部分已经有了偏置电路。
## RK3566 双屏显示问题

由于RK3566双屏幕显示是使用同一个内部输入源，即`VOP`是`Same Source, Dual Display`,所以如果是使用MIPI屏幕作为主屏，会导致副屏HDMI画面会严重拉伸（如下图)，所以想避免
这个问题，主副屏幕都应该使用同分辨率同方向（横竖屏一致）的屏幕。

![](../../img/ROC-RK3566-PC/3566_dualscreen.png)


## 修改 kernel/buildroot 等配置后编译烧录发现不生效
通过`make menuconfig`修改配置之后，使用`build.sh`编译，烧录发现修改没有生效。然后检查 config 发现之前的修改消失了。

这是因为只将修改保存到了临时文件`.config`，编译时`build.sh`会用配置文件覆盖掉`.config`

所以修改后要保存到配置文件：
```bash
make ARCH=arm64 savedefconfig
mv defconfig arch/arm64/configs/firefly_linux_defconfig
```
之后再使用`build.sh`编译。

## V1.0版本的设备和V1.1版本的设备的区别
### V1.0版本设备
在V1.0版本设备中，I2C2_M0和I2C2_M1被硬件同时引出，造成了复用，所以使用I2C2_M1的MIPI CSI和使用I2C2_M0的RTC和MIPI DSI触摸部分是不能同时使用，V1.0版本设备出厂固件默认是使能I2C2_M0（即支持RTC和MIPI DSI关闭MIPI CSI），如需要使能MIPI CSI，则SDK需要进行以下修改即可
```
&i2c2{
	pinctrl-0 = <&i2c2m1_xfer>;
};

```

这样I2C2就使用了M1通道，而RTC和MIPI DSI的触摸部分暂时用不了。

### V1.1及以上版本设备
在V1.1及以上版本的设备中，MIPI CSI单独使用的I2C4_M0和RTC,MIPI DSI所使用的I2C2_M0是不冲突的，共用的，不需要使用上一步的配置来进行区分使用。

## HDMI 无法 4K 显示？

暂时无法支持自动切换 4K, 默认 1080P 显示, 后续会更新支持。可以使用以下命令手动设置 4K 分辨率后,重新插拔 HDMI 线：

```
# 设置4k60：
setprop persist.sys.resolution.main 3840x2160@60
# 每次设置完更新sys.display.timeline(每次加1)使分辨率生效
setprop sys.display.timeline 1
```

## 3.5mm耳机座子接国标耳机有异常？

目前ROC-RK3566-PC仅支持美标类型(CTIA)的耳机,对于国标类型(OMTP)的耳机硬件不兼容,会出现左右声道同时存在双声道的现象。





## 开机异常并循环重启怎么办？


有可能是电源电流不够，请使用PD 电源来供电。

## Ubuntu 默认的用户名和密码是什么？

* 用户名：`firefly`
* 密码：`firefly`
* 切换超级用户 `sudo -s`


## 写号工具写入SN，MAC地址
<font color=red>**注意:**</font>如果开发板进行了eMMC擦除操作，之前写入的数据也会被清除。
### Windows方式
* 安装RKDevInfoWriteTool
    * [下载地址](https://www.t-firefly.com/doc/download/106.html#other_379)
* RKDevInfoWriteTool的**设置**里选中"RPMB"
* 根据需要在RKDevInfoWriteTool的**设置**里配置"SN"，"WIFI MAC"，"LAN MAC"，"BT MAC"等
* 开发板进入loader模式
* RKDevInfoWriteTool进行**写入**或者**读取**操作

具体操作可以参考RKDevInfoWriteTool安装目录下的《RKDevInfoWriteTool使用指南》PDF文档。
### Linux方式
开发板自身写号方式
* buildroot使能`BR2_PACKAGE_VENDOR_STORAGE`
* 通过vendor_storage命令进行读写操作
    * [下载地址](https://www.t-firefly.com/doc/download/106.html#other_379)
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

1、`Settings(设置)` -> `About tablet(关于平板电脑)` -> 点击7下 `Build number(版本号)` 

2、`Settings(设置)`->`System(系统)`->`Advanced(高级)`->`Developer options(开发者选项)` -> `Android LogSave(日志采集器)`。打开功能后，系统的 `/data/vendor/logs` 目录下就会生成 对应日期的日志 文件夹，里面包括系统 logcat 和内核 kmsg。

## Android SDK单独编译生成的boot.img遇到问题
目前，Android单独编译kernel生成的boot.img的方法：

* 编译时指定`BOOT_IMG` 参数 ，如`BOOT_IMG=../rockdev/Image-rk3566_roc_pc/boot.img`

	`BOOT_IMG`指定的前置boot.img在编译完后会更新`path/to/sdk/kernel/boot.img`，烧写该boot.img即可更新kernel的修改
	```
	make ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3566_roc_pc/boot.img rk3566-roc-pc.img -j8
	```
	
如果不指定`BOOT_IMG`，那么就会导致在下载之后，系统会跑进了`Recovery`模式(或者引起其他启动错误)，而不是进行正常的启动流程。
如果没有`rockdev`这个目录，请先完整编译一次SDK，编译步骤参考*公版编译*。

