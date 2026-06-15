### The overall compilation

#### HDMI Firmware Compilation
```
./FFTools/make.sh -d roc-rk3588s-pc -j8 -l roc_rk3588s_pc-userdebug
./FFTools/mkupdate/mkupdate.sh -l roc_rk3588s_pc-userdebug
```

#### 10.1‘ MIPI DSI0 Firmware Compilation：
```
./FFTools/make.sh -d roc-rk3588s-pc-mipi101-M101014-BE45-A1 -j8 -l roc_rk3588s_pc_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l roc_rk3588s_pc_mipi-userdebug
```

#### Dual Camera CAM-2MS2MF Compile
##### HDMI+CAM-2MS2MF
* modify dts
```
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dts
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dts
@@ -9,8 +9,9 @@
 #include "aio-3588sjd4.dtsi"
-#include "aio-3588sjd4-cam-8ms1m.dtsi"
#include "aio-3588sjd4-cam-dcphy-8ms1m.dtsi"
-//#include "aio-3588sjd4-cam-2ms2mf.dtsi"
+//#include "aio-3588sjd4-cam-8ms1m.dtsi"
+#include "aio-3588sjd4-cam-2ms2mf.dtsi"
```

* Compile
```
./FFTools/make.sh -d roc-rk3588s-pc -j8 -l roc_rk3588s_pc-userdebug
./FFTools/mkupdate/mkupdate.sh -l roc_rk3588s_pc-userdebug
```
#### HDMI TO MIPI_CSI(RK628D) Compile
##### HDMI+CAM-2MS2MF
* modify dts
```
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dts
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dts
@@ -7,9 +7,9 @@
 /dts-v1/;

 #include "aio-3588sjd4.dtsi"
-#include "aio-3588sjd4-cam-8ms1m.dtsi"
+//#include "aio-3588sjd4-cam-8ms1m.dtsi"
 //#include "aio-3588sjd4-cam-2ms2mf.dtsi"
-//#include "aio-3588sjd4-tf-hdmi-mipi-rk628.dtsi"
+#include "aio-3588sjd4-tf-hdmi-mipi-rk628.dtsi"
 #include "aio-3588sjd4-cam-dcphy-8ms1m.dtsi"

```
* Compile
```
./FFTools/make.sh -d roc-rk3588s-pc -j8 -l roc_rk3588s_pc-userdebug
./FFTools/mkupdate/mkupdate.sh -l roc_rk3588s_pc-userdebug
```

### Step by step to compile

* Compile the kernel:

```
cd ~/proj/RK3588_Android12.0/kernel-5.10
export PATH=../prebuilts/clang/host/linux-x86/clang-r416183b/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64  firefly_defconfig android-11.config pcie_wifi.config
msk ARCH=arm64   BOOT_IMG=../rockdev/Image-roc_rk3588s_pc/boot.img roc-rk3588s-pc.img -j8
```

* Compile uboot:

```
cd ~/proj/RK3588_Android12.0/u-boot/
./make.sh rk3588
```

* Compile Android：

```
cd ~/proj/RK3588_Android12.0/
source build/envsetup.sh
lunch roc_rk3588s_pc-userdebug
make installclean
make -j8
./mkimage.sh
```

### Packaged into unified firmware update.img

After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command：
```
./FFTools/mkupdate/mkupdate.sh -l roc_rk3588s_pc-userdebug
```
After packaging, it will be in rockdev/Image-roc_rk3588s_pc/ Generate unified firmware under the directory: (product name )XXX_XXX_date XXX.img

It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create up
date.img and store it in rockdev \ Image directory.