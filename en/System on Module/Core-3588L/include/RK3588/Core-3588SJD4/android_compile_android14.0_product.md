### The overall compilation

#### HDMI Firmware Compilation
```
./FFTools/make.sh -d aio-3588l -j8 -l aio_3588l-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588l-userdebug
```

#### 10.1‘ V3S MIPI DSI0 Firmware Compilation：
```
./FFTools/make.sh -d aio-3588l-mipi101-BSD1218-A101KL68 -j8 -l aio_3588l_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588l_mipi-userdebug
```
<!--
#### Dual Camera CAM-2MS2MF Compile
##### HDMI+CAM-2MS2MF
* modify dts
```
--- a/kernel-6.1/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dts
+++ b/kernel-6.1/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dts
@@ -9,8 +9,8 @@
 #include "aio-3588sjd4.dtsi"
 //#include "aio-3588sjd4-ext.dtsi"

-#include "aio-3588sjd4-cam-8ms1m.dtsi"
-//#include "aio-3588sjd4-cam-2ms2mf.dtsi"
+//#include "aio-3588sjd4-cam-8ms1m.dtsi"
+#include "aio-3588sjd4-cam-2ms2mf.dtsi"
```

* Compile
```
./FFTools/make.sh -d aio-3588l -j8 -l aio_3588l-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588l-userdebug
```

#### HDMI TO MIPI_CSI(RK628D) Compile
* modify dts
```
--- a/kernel-6.1/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dts
+++ b/kernel-6.1/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dts
@@ -9,9 +9,9 @@
 #include "aio-3588sjd4.dtsi"
 //#include "aio-3588sjd4-ext.dtsi"

-#include "aio-3588sjd4-cam-8ms1m.dtsi"
+//#include "aio-3588sjd4-cam-8ms1m.dtsi"
 //#include "aio-3588sjd4-cam-2ms2mf.dtsi"
-//#include "aio-3588sjd4-tf-hdmi-mipi-rk628.dtsi"
+#include "aio-3588sjd4-tf-hdmi-mipi-rk628.dtsi"
```

* Compile
```
./FFTools/make.sh -d aio-3588l -j8 -l aio_3588l-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588l-userdebug
```
-->
### Step by step to compile

* Compile the kernel:

```
cd ~/proj/RK3588_Android14.0/kernel-6.1
export PATH=../prebuilts/clang/host/linux-x86/clang-r487747c/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64 firefly_defconfig android-14.config pcie.config
msk ARCH=arm64 BOOT_IMG=../rockdev/Image-aio_3588sjd4/boot.img aio-3588sjd4.img -j8
```

* Compile uboot:

```
cd ~/proj/RK3588_Android14.0/u-boot/
./make.sh rk3588
```

* Compile Android：

```
cd ~/proj/RK3588_Android14.0/
source build/envsetup.sh
lunch aio_3588l-userdebug
make installclean
make -j8
./mkimage.sh
```

### Packaged into unified firmware update.img

After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command：
```
./FFTools/mkupdate/mkupdate.sh -l aio_3588l-userdebug
```
After packaging, it will be in rockdev/Image-aio_3588l/ Generate unified firmware under the directory: (product name )XXX_XXX_date XXX.img

It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create up
date.img and store it in rockdev \ Image directory.