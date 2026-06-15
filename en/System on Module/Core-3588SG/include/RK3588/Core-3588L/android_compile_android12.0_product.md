### The overall compilation

#### HDMI Firmware Compilation
```
./FFTools/make.sh -d rk3588s-firefly-aio-3588sg -j8 -l rk3588s_firefly_aio_3588sg-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3588s_firefly_aio_3588sg-userdebug
```

#### 10.1‘ MIPI DSI0 Firmware Compilation：
```
./FFTools/make.sh -d  -j8 -l rk3588s_firefly_aio_3588sg_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3588s_firefly_aio_3588sg_mipi-userdebug
```
#### Dual Camera CAM-2MS2MF Compile
##### HDMI+CAM-2MS2MF
* modify dts
```
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dts
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dts
@@ -8,8 +8,8 @@

 #include "rk3588-firefly-itx-3588j.dtsi"
-#include "rk3588-firefly-itx-cam-8ms1m.dtsi"
-//#include "rk3588-firefly-itx-cam-2ms2mf.dtsi"
+//#include "rk3588-firefly-itx-cam-8ms1m.dtsi"
+#include "rk3588-firefly-itx-cam-2ms2mf.dtsi"
 #include "rk3588-firefly-demo.dtsi"
```

* Compile
```
./FFTools/make.sh -d rk3588s-firefly-aio-3588sg -j8 -l rk3588s_firefly_aio_3588sg-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3588s_firefly_aio_3588sg-userdebug
```
#### HDMI TO MIPI_CSI(RK628D) Compile
* modify dts
```
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dts
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dts
@@ -7,9 +7,9 @@
 /dts-v1/;

 #include "rk3588-firefly-itx-3588j.dtsi"
-#include "rk3588-firefly-itx-cam-8ms1m.dtsi"
+//#include "rk3588-firefly-itx-cam-8ms1m.dtsi"
 //#include "rk3588-firefly-itx-cam-2ms2mf.dtsi"
-//#include "rk3588-firefly-itx-3588j-tf-hdmi-mipi-rk628.dtsi"
+#include "rk3588-firefly-itx-3588j-tf-hdmi-mipi-rk628.dtsi"
 #include "rk3588-firefly-demo.dtsi"
```
* Compile
```
./FFTools/make.sh -d rk3588s-firefly-aio-3588sg -j8 -l rk3588s_firefly_aio_3588sg-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3588s_firefly_aio_3588sg-userdebug
```

### Step by step to compile

* Compile the kernel:

```
cd ~/proj/RK3588_Android12.0/kernel-5.10
export PATH=../prebuilts/clang/host/linux-x86/clang-r416183b/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64  firefly_defconfig android-11.config pcie_wifi.config
msk ARCH=arm64   BOOT_IMG=../rockdev/Image-rk3588s_firefly_aio_3588sg/boot.img rk3588s-firefly-aio-3588sg.img -j8
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
lunch rk3588s_firefly_aio_3588sg-userdebug
make installclean
make -j8
./mkimage.sh
```

### Packaged into unified firmware update.img

After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command：
```
./FFTools/mkupdate/mkupdate.sh -l rk3588s_firefly_aio_3588sg-userdebug
```
After packaging, it will be in rockdev/Image-rk3588s_firefly_aio_3588sg/ Generate unified firmware under the directory: (product name )XXX_XXX_date XXX.img

It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create up
date.img and store it in rockdev \ Image directory.