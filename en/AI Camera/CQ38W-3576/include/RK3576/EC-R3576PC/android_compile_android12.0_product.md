### The overall compilation

<font color=#FF0000>Note: Note: As the camera interface is occupied, the camera configuration needs to be disabled and enable the expansion board configuration before compilation </font>
```
diff --git a/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dts b/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dts
index 6d376df7e6e..289a8347c07 100644
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dts
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dts
@@ -7,9 +7,9 @@
 #include "roc-rk3588s-pc.dtsi"
-//#include "roc-rk3588s-pc-ext.dtsi"
+#include "roc-rk3588s-pc-ext.dtsi"

-#include "rk3588-roc-pc-cam-8ms1m.dtsi"
+//#include "rk3588-roc-pc-cam-8ms1m.dtsi"
```

#### HDMI Firmware Compilation
```
./FFTools/make.sh -d  -j8 -l rk3576_firefly_ext_icore_3576q38-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_ext_icore_3576q38-userdebug
```

#### 10.1‘ MIPI DSI0 Firmware Compilation：
```
./FFTools/make.sh -d  -j8 -l firefly_ext_icore_3576q38_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l firefly_ext_icore_3576q38_mipi-userdebug
```

### Step by step to compile

* Compile the kernel:

```
cd ~/path/to/sdk/kernel-5.10
export PATH=../prebuilts/clang/host/linux-x86/clang-r416183b/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64 
msk ARCH=arm64   BOOT_IMG=../rockdev/rk3576_firefly_ext_icore_3576q38/boot.img .img -j8
```

* Compile uboot:

```
cd ~/path/to/sdk/u-boot/
./make.sh rk3576
```

* Compile Android：

```
cd ~/path/to/sdk/
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