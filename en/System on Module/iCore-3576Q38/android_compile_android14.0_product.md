### The overall compilation

#### HDMI Firmware Compilation
```
./FFTools/make.sh -j8 -l rk3576_firefly_ext_icore_3576q38-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_ext_icore_3576q38-userdebug
```

#### 10.1‘ MIPI V3S Firmware Compilation：
```
./FFTools/make.sh -j8 -l firefly_ext_icore_3576q38_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l firefly_ext_icore_3576q38_mipi-userdebug
```
<!--
#### Dual Camera CAM-2MS2MF Compile
##### HDMI+CAM-2MS2MF
* modify dts
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

* Compile
```
./FFTools/make.sh -d  -j8 -l rk3576_firefly_ext_icore_3576q38-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_ext_icore_3576q38-userdebug
```

#### HDMI TO MIPI_CSI(RK628D) Compile
* modify dts
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

* Compile
```
./FFTools/make.sh -d  -j8 -l rk3576_firefly_ext_icore_3576q38-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_ext_icore_3576q38-userdebug
```
-->
### Step by step to compile

* Compile the kernel:

```
cd ~/proj/RK3576_Android14.0/kernel-6.1
export PATH=../prebuilts/clang/host/linux-x86/clang-r487747c/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64 firefly_defconfig android-14.config rk3576.config
msk ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3576_firefly_aio_3576q/boot.img rk3576-firefly-aio-3576q.img -j8
```

* Compile uboot:

```
cd ~/proj/RK3588_Android14.0/u-boot/
make rk3576_defconfig
./make.sh --spl-new
```

* Compile Android：

```
cd ~/proj/RK3588_Android14.0/
source build/envsetup.sh
lunch rk3576_firefly_ext_icore_3576q38-userdebug
make installclean
make -j8
./mkimage.sh
```

### Packaged into unified firmware update.img

After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command：
```
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_ext_icore_3576q38-userdebug
```
After packaging, it will be in rockdev/rk3576_firefly_ext_icore_3576q38/ Generate unified firmware under the directory: (product name )XXX_XXX_date XXX.img

It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create up
date.img and store it in rockdev \ Image directory.