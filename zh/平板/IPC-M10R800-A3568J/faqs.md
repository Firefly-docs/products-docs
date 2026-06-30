# FAQs

## kernel 编译弹出 “IO-Domain Checklist” 对话框
kernel 新建了 dts 文件 `arch/arm64/boot/dts/rockchip/-DEMO.dts`，编译 kernel 时弹出如下对话框
![](../../../rk356x_img/faq_rk356x_io-domain_checklist.png)

<br>

**先进行如下操作，再编译 kernel 。**       
```
cp arch/arm64/boot/dts/rockchip/..dtb.dts.tmp.domain arch/arm64/boot/dts/rockchip/.-DEMO.dtb.dts.tmp.domain
```

<font color=red>**切记：不要轻易修改主控电源域的IO电平，否则，最坏的情况可能会导致IO的损坏。**</font>



## 修改 kernel/buildroot 等配置后编译烧录发现不生效
通过`make menuconfig`修改配置之后，使用`build.sh`编译，烧录发现修改没有生效。然后检查 config 发现之前的修改消失了。

这是因为只将修改保存到了临时文件`.config`，编译时`build.sh`会用配置文件覆盖掉`.config`

所以修改后要保存到配置文件：
```bash
make ARCH=arm64 savedefconfig
mv defconfig arch/arm64/configs/firefly_linux_defconfig
```
之后再使用`build.sh`编译。


## 开机异常并循环重启怎么办？


有可能是电源电流不够，请使用电压为 12V，电流为 2.5A~3A 的电源。



## 写号工具写入SN，MAC地址
<font color=red>**注意:**</font>如果开发板进行了eMMC擦除操作，之前写入的数据也会被清除。
### Windows方式
* 安装RKDevInfoWriteTool
    * [下载地址](https://www.t-firefly.com/doc/download/124.html#other_379)
* RKDevInfoWriteTool的**设置**里选中"RPMB"
* 根据需要在RKDevInfoWriteTool的**设置**里配置"SN"，"WIFI MAC"，"LAN MAC"，"BT MAC"等
* 开发板进入loader模式
* RKDevInfoWriteTool进行**写入**或者**读取**操作

具体操作可以参考RKDevInfoWriteTool安装目录下的《RKDevInfoWriteTool使用指南》PDF文档。
### Linux方式
开发板自身写号方式
* buildroot使能`BR2_PACKAGE_VENDOR_STORAGE`
* 通过vendor_storage命令进行读写操作
    * [下载地址](https://www.t-firefly.com/doc/download/124.html#other_379)
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


## Android 下如何让系统抓取 LOG？

1、`Settings(设置)` -> `About tablet(关于平板电脑)` -> 点击7下 `Build number(版本号)` 

2、`Settings(设置)`->`System(系统)`->`Advanced(高级)`->`Developer options(开发者选项)` -> `Android LogSave(日志采集器)`。打开功能后，系统的 `/data/vendor/logs` 目录下就会生成 对应日期的日志 文件夹，里面包括系统 logcat 和内核 kmsg。

## Android SDK单独编译生成的boot.img遇到问题
目前，Android单独编译kernel生成的boot.img的方法：

* 编译时指定`BOOT_IMG` 参数 ，如`BOOT_IMG=../rockdev//boot.img`

	`BOOT_IMG`指定的前置boot.img在编译完后会更新`path/to/sdk/kernel/boot.img`，烧写该boot.img即可更新kernel的修改
	```
	make ARCH=arm64 BOOT_IMG=../rockdev//boot.img .img -j8
	```
	
如果不指定`BOOT_IMG`，那么就会导致在下载之后，系统会跑进了`Recovery`模式(或者引起其他启动错误)，而不是进行正常的启动流程。
如果没有`rockdev`这个目录，请先完整编译一次SDK，编译步骤参考*公版编译*。

