### 整体编译

#### HDMI 固件编译 
```
./FFTools/make.sh -j8 -l rk3576_firefly_ext_icore_3576q38-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_ext_icore_3576q38-userdebug
```
#### 显示屏 DM-M10R800 V3S 固件编译：
```
./FFTools/make.sh -j8 -l firefly_ext_icore_3576q38_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l firefly_ext_icore_3576q38_mipi-userdebug
```
<!--
#### 双目摄像头 CAM-2MS2MF 编译：
##### HDMI+CAM-2MS2MF
* 修改 dts
```
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dts
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dts
@@ -9,8 +9,8 @@
 #include "roc-rk3588s-pc.dtsi"
 //#include "roc-rk3588s-pc-ext.dtsi"
 
-#include "roc-rk3588s-pc-cam-8ms1m.dtsi"
-//#include "roc-rk3588s-pc-cam-2ms2mf.dtsi"
+//#include "roc-rk3588s-pc-cam-8ms1m.dtsi"
+#include "roc-rk3588s-pc-cam-2ms2mf.dtsi"
```

* 编译
```
./FFTools/make.sh -d  -j8 -l rk3576_firefly_ext_icore_3576q38-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_ext_icore_3576q38-userdebug
```
*/
#### HDMI TO MIPI_CSI(RK628D) 编译
* 修改 dts
```
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dts
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dts
@@ -9,9 +9,9 @@
 #include "roc-rk3588s-pc.dtsi"
 //#include "roc-rk3588s-pc-ext.dtsi"
 
-#include "roc-rk3588s-pc-cam-8ms1m.dtsi"
+//#include "roc-rk3588s-pc-cam-8ms1m.dtsi"
 //#include "roc-rk3588s-pc-cam-2ms2mf.dtsi"
-//#include "roc-rk3588s-pc-tf-hdmi-mipi-rk628.dtsi"
+#include "roc-rk3588s-pc-tf-hdmi-mipi-rk628.dtsi"
```

* 编译
```
./FFTools/make.sh -d  -j8 -l rk3576_firefly_ext_icore_3576q38-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_ext_icore_3576q38-userdebug
```
-->
### 分步编译

* 编译 kernel：

```
cd ~/proj/RK3576_Android14.0/kernel-6.1
export PATH=../prebuilts/clang/host/linux-x86/clang-r487747c/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64 firefly_defconfig android-14.config rk3576.config
msk ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3576_firefly_aio_3576q/boot.img rk3576-firefly-aio-3576q.img -j8
```

* 编译 uboot：

```
cd ~/proj/RK3576_Android14.0/u-boot/
make rk3576_defconfig
./make.sh --spl-new
```

* 编译 Android：

```
cd ~/proj/RK3576_Android14.0/
source build/envsetup.sh
lunch rk3576_firefly_ext_icore_3576q38-userdebug
make installclean
make -j8
./mkimage.sh
```

### 打包成统一固件 update.img

编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：
```
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_ext_icore_3576q38-userdebug
```
打包完成后将在rockdev/rk3576_firefly_ext_icore_3576q38/ 目录下生成统一固件： product名XXX_XXX_日期XXX.img

在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。