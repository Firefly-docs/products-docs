### 整体编译
**注意：由于 Core-3399-JD4 的硬件迭代版本是 [ROC-RK3399-PC-Pro](https://wiki.t-firefly.com/zh_CN/ROC-RK3399-PC-Pro/started.html),所以软件的编译方法是一致。最终生成的固件如：ROC-RK3399-PC-Pro_xxx.img 对 Core-3399-JD4 也是兼容的。**

#### 公版编译
##### HDMI+DP
```
./FFTools/make.sh -d rk3399-roc-pc-plus -j8 -l rk3399_roc_pc_plus-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus-userdebug
```

#### 显示屏 DM-M10R800 V2 编译
##### MIPI_DSI0+HDMI
```
./FFTools/make.sh -j8 -d rk3399-roc-pc-plus-mipi101-JDM101014_BC45_A1 -l rk3399_roc_pc_plus_mipi101-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus_mipi101-userdebug
```

#### 显示屏 DM-M10R800 V3 编译
##### MIPI_DSI0+HDMI
```
./FFTools/make.sh -j8 -d rk3399-roc-pc-plus-mipi101-BSD1218-A101KL68 -l rk3399_roc_pc_plus_mipi101-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus_mipi101-userdebug
```

#### 双目摄像头 SV-TAYSH-TQ 编译
##### HDMI+SV-TAYSH-TQ

* 修改 `device/rockchip/rk3399/rk3399_roc_pc_plus.mk`
```
 BOARD_NFC_SUPPORT := false
 BOARD_HAS_GPS := false
+BOARD_XC7022_XC6130_SUPPORT := true

 #for 3G/4G modem dongle support
 BOARD_HAVE_DONGLE := false
```

* 编译
```
./FFTools/make.sh -d rk3399-roc-pc-plus -j8 -l rk3399_roc_pc_plus-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus-userdebug
```

### 分步编译 

编译前执行如下命令配置环境变量：
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

*    编译kernel：
```
cd ~/proj/firefly-rk3399-industry/kernel/
make ARCH=arm64 firefly_defconfig
make -j8 ARCH=arm64 rk3399-roc-pc-plus.img
```

*    编译uboot：
```
cd ~/proj/firefly-rk3399-industry/u-boot/
make rk3399_defconfig
make ARCHV=aarch64 -j8
```

* 编译android：
```
cd ~/proj/firefly-rk3399-Industry/
source build/envsetup.sh
lunch  
make -j8
./mkimage.sh
```

### 打包成统一固件 

编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：
```
./FFTools/mkupdate/mkupdate.sh -l 
```
根据不同的-l XXX-userdebug参数，打包生成统一固件会存放在不同目录下（rockdev/Image-XXX/）： product名XXX_XXX_日期XXX.img

在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。

## 烧写分区映像

编译的时候执行 ./mkimage.sh 会重新打包 boot.img 和 system.img, 并将其它相关的映像文件拷贝到目录 rockdev/Image-rk3399_firefly/ 中。以下列出一般固件用到的映像文件：

*    boot.img ：Android 的初始文件映像，负责初始化并加载 system 分区。
*    kernel.img ：内核映像。
*    misc.img ：misc 分区映像，负责启动模式切换和急救模式的参数传递。
*    parameter.txt ：emmc的分区信息
*    recovery.img ：急救模式映像。
*    resource.img ：资源映像，内含开机图片和内核的设备树信息。
*    system.img ：Android 的 system 分区映像，ext4 文件系统格式。
*    trust.img ：休眠唤醒相关的文件
*    rk3399_loader_v1.08.106.bin ：Loader文件
*    uboot.img ：uboot文件

请参照 [《升级固件》](03-upgrade_firmware.md) 一文来烧写分区映像文件。

如果使用的是 Windows 系统，将上述映像文件拷贝到 AndroidTool （Windows 下的固件升级工具）的 rockdev\Image 目录中，之后参照升级文档烧写分区映像即可，这样的好处是使用默认配置即可，不用修改文件的路径。<br />
update.img 方便固件的发布，供终端用户升级系统使用。一般开发时使用分区映像比较方便。

<font color=#ff0000 size=4>**注意：**</font> <br />
<font color=#ff0000>使用升级工具进行industry版固件升级时，如果原有烧录的是tvbox版固件，需要先短路emmc或进行flash擦除。且注意使用相适配升级工具</font><br />
请参照 [烧写须知](upgrade_table.html#shao-xie-xu-zhi-zhong-yao)