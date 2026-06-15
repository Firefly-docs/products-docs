### 整体编译

#### 默认固件编译 
```
./FFTools/make.sh -d roc-rk3588s-pc -j8 -l roc_rk3588s_pc-userdebug
./FFTools/mkupdate/mkupdate.sh -l roc_rk3588s_pc-userdebug
```
#### 显示屏 DM-M10R800 V3S 固件编译：
```
./FFTools/make.sh -d roc-rk3588-pc-mipi101-BSD1218-A101KL68 -j8 -l roc_rk3588s_pc_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l roc_rk3588s_pc_mipi-userdebug
```
<!--
#### 双目摄像头 CAM-2MS2MF 编译：
##### HDMI+CAM-2MS2MF
* 修改 dts
```
--- a/kernel-6.1/arch/arm64/boot/dts/rockchip/rk3588s-firefly-aio-3588sg.dts
+++ b/kernel-6.1/arch/arm64/boot/dts/rockchip/rk3588s-firefly-aio-3588sg.dts
@@ -9,8 +9,8 @@
 #include "rk3588s-firefly-aio-3588sg.dtsi"
 //#include "rk3588s-firefly-aio-3588sg-ext.dtsi"
 
-#include "rk3588s-firefly-aio-3588sg-cam-8ms1m.dtsi"
-//#include "rk3588s-firefly-aio-3588sg-cam-2ms2mf.dtsi"
+//#include "rk3588s-firefly-aio-3588sg-cam-8ms1m.dtsi"
+#include "rk3588s-firefly-aio-3588sg-cam-2ms2mf.dtsi"
```

* 编译
```
./FFTools/make.sh -d roc-rk3588s-pc -j8 -l roc_rk3588s_pc-userdebug
./FFTools/mkupdate/mkupdate.sh -l roc_rk3588s_pc-userdebug
```

#### HDMI TO MIPI_CSI(RK628D) 编译
* 修改 dts
```
--- a/kernel-6.1/arch/arm64/boot/dts/rockchip/rk3588s-firefly-aio-3588sg.dts
+++ b/kernel-6.1/arch/arm64/boot/dts/rockchip/rk3588s-firefly-aio-3588sg.dts
@@ -9,9 +9,9 @@
 #include "rk3588s-firefly-aio-3588sg.dtsi"
 //#include "rk3588s-firefly-aio-3588sg-ext.dtsi"
 
-#include "rk3588s-firefly-aio-3588sg-cam-8ms1m.dtsi"
+//#include "rk3588s-firefly-aio-3588sg-cam-8ms1m.dtsi"
 //#include "rk3588s-firefly-aio-3588sg-cam-2ms2mf.dtsi"
-//#include "rk3588s-firefly-aio-3588sg-tf-hdmi-mipi-rk628.dtsi"
+#include "rk3588s-firefly-aio-3588sg-tf-hdmi-mipi-rk628.dtsi"
```

* 编译
```
./FFTools/make.sh -d roc-rk3588s-pc -j8 -l roc_rk3588s_pc-userdebug
./FFTools/mkupdate/mkupdate.sh -l roc_rk3588s_pc-userdebug
```
-->
### 分步编译

* 编译 kernel：

```
cd ~/proj/RK3588_Android14.0/kernel-6.1
export PATH=../prebuilts/clang/host/linux-x86/clang-r487747c/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64 firefly_defconfig android-14.config pcie_wifi.config
msk ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3588s_firefly_aio_3588sg/boot.img rk3588s-firefly-aio-3588sg.img -j8

```

* 编译 uboot：

```
cd ~/proj/RK3588_Android14.0/u-boot/
./make.sh rk3588
```

* 编译 Android：

```
cd ~/proj/RK3588_Android14.0/
source build/envsetup.sh
lunch roc_rk3588s_pc-userdebug
make installclean
make -j8
./mkimage.sh
```

### 打包成统一固件 update.img

编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：
```
./FFTools/mkupdate/mkupdate.sh -l roc_rk3588s_pc-userdebug
```
打包完成后将在rockdev/Image-roc_rk3588s_pc/ 目录下生成统一固件： product名XXX_XXX_日期XXX.img

在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。