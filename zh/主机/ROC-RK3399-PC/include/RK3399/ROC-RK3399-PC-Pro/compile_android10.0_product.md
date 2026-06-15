### 整体编译

**注意**：由于 ROC-RK3399-PC 是对 [ROC-RK3399-PC-PLUS](https://wiki.t-firefly.com/zh_CN/ROC-RK3399-PC-PLUS/started.html) 硬件上的迭代，所以软件的编译方法是一致。最终生成的固件如：`ROC-RK3399-PC-Pro_xxx.img` 对 [ROC-RK3399-PC-PLUS](https://wiki.t-firefly.com/zh_CN/ROC-RK3399-PC-PLUS/started.html) 也是兼容的。

#### 公版编译
##### HDMI+DP
```
./FFTools/make.sh -d rk3399-roc-pc-plus -j8 -l rk3399_roc_pc_plus-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus-userdebug
```

#### 显示屏 DM-M10R800 V2 编译
##### MIPI_DSI0+HDMI
```
./FFTools/make.sh -j8 -d rk3399-roc-pc-plus-mipi101-JDM101014_BC45_A1 -l rk3399_roc_pc_plus_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus_mipi-userdebug
```

#### 摄像头 OV13850 编译
##### HDMI+OV13850

* 修改 `kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus.dtsi`
```
 &i2c1{

-    XC6130b@23{
+    ov13850b@10{
     status = "okay";
     };
-    XC7022b@1b{
+    ov13850f@10{
     status = "okay";
     };

-    /delete-node/ ov13850b@10;
-    /delete-node/ ov13850f@10;
+    /delete-node/ XC6130b@23;
+    /delete-node/ XC7022b@1b;

 };
```

* 编译
```
./FFTools/make.sh -d rk3399-roc-pc-plus -j8 -l rk3399_roc_pc_plus-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus-userdebug
```

#### 双目摄像头 SV-TAYSH-TQ 编译
##### HDMI+SV-TAYSH-TQ

* 修改 `kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus.dtsi`
```
    xc7160b@1b{
+    status = "disabled";
    };
    xc7160f@1b{
+    status = "disabled";
    };

    XC6130b@23{
+    status = "okay";
    };
    XC7022b@1b{
+    status = "okay";
    };
```

* 编译
```
./FFTools/make.sh -d rk3399-roc-pc-plus -j8 -l rk3399_roc_pc_plus-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus-userdebug
```

### 分步编译

以编译 HDMI 固件为例，编译前执行如下命令配置环境变量：

```
source ./FFTools/build.sh
```
* 编译 kernel

注意：若进行内核 debug，需要将 `resource.img` 和 `kernel.img` 编译打包成 `boot.img` 后对 `boot` 分区进行烧写才能生效。
```
cd ~/proj/rk3399_Android10.0/kernel/
make ARCH=arm64 firefly_defconfig android-10.config rk3399.config
make ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3399_roc_pc_plus/boot.img rk3399-roc-pc-plus.img -j8
```

* 编译 uboot

```
cd ~/proj/rk3399_Android10.0/u-boot/
./make.sh rk3399
```

* 编译 Android

```
cd ~/proj/rk3399_Android10.0/
lunch rk3399_roc_pc_plus-userdebug
make -j8
./mkimage.sh
```

* 打包成统一固件

	编译完可以用 **Firefly** 官方的脚本打包成统一固件，执行如下命令：
	```
	./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus-userdebug
	```
	根据不同的 `-l XXX-userdebug` 参数，打包生成统一固件会存放在不同目录下 `path/to/sdk/rockdev/Image-XXX/product名XXX_XXX_日期XXX.img`。
	在 Windows 下打包统一固件 `update.img` 也很简单，将编译生成的文件拷贝到 AndroidTool 的 `rockdev\rk3399_roc_pc_plus-userdebug` 目录中，然后运行 rockdev 目录下的 `mkupdate.bat` 批处理文件即可创建 `update.img` 并存放到 `rockdev\rk3399_roc_pc_plus-userdebug` 目录里。


### 分区镜像

编译的时候执行 ./mkimage.sh 会重新打包不同分区的 `image`文件到目录 `rockdev/rk3399_roc_pc_plus-userdebug/` 中。
以下列出一般固件用到的映像文件：

* boot.img：包含 ramdisk、kernel、dtb
* dtbo.img：Device Tree Overlays 
* kernel.img：includekernel（目前无法单独烧写，需要打包到boot.img内烧写）
* MiniLoaderAll.bin：包含一级loader
* misc.img：包含recovery-wipe开机标识信息，烧写后会进行recovery
* odm.img：包含android odm，包含在super.img分区内,单独烧写需要用fastboot烧写
* parameter.txt：包含分区信息
* pcba_small_misc.img：包含pcba开机标识信息，烧写后会进入简易版pcba模式
* pcba_whole_misc.img：包含pcba开机标识信息，烧写后会进入完整版pcba模式
* recovery.img：包含recovery-ramdis、kernel、dtb
* resource.img：包含dtb，kernel和uboot阶段的log及uboot充电logo，目前无法单独烧写，需要打包到boot.img内烧写
* super.img：包含odm、vendor、system分区内容
* system.img：包含android system，包含在super.img分区内,单独烧写需要同fastboot烧写
* trust.img：包含BL31、BL32
* uboot.img：包含uboot固件
* vbmeta.img：包含avb校验信息，用于AVB校验
* vendor.img：包含android vendor，包含在super.img分区内,单独烧写需要同fastboot烧写
* update.img：包含以上需要烧写的img文件，可以用于工具直接烧写整个固件包

### 烧写固件

请参考：[《升级固件》](03-upgrade_firmware.md) 