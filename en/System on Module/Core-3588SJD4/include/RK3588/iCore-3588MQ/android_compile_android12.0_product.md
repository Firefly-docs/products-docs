### The overall compilation

The current software version supports the compatibility between cpu of 3588q. Therefore, no additional software configuration is required.

#### HDMI Firmware Compilation
```
./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
```

#### 10.1‘ MIPI DSI0 Firmware Compilation：
```
./FFTools/make.sh -d aio-3588sjd4-mipi101-M101014-BE45-A1 -j8 -l aio_3588sjd4_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4_mipi-userdebug
```

#### NV156FHM-T06 edp Firmware compilation
```
./FFTools/make.sh -d  -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```
#### Dual Camera CAM-2MS2MF Compile
##### HDMI+CAM-2MS2MF
* modify dts
```
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q.dts
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q.dts
@@ -7,8 +7,8 @@
 #include "rk3588-firefly-aio-3588q.dtsi"
-#include "rk3588-firefly-aio-cam-8ms1m.dtsi"
-//#include "rk3588-firefly-aio-cam-2ms2mf.dtsi"
+//#include "rk3588-firefly-aio-cam-8ms1m.dtsi"
+#include "rk3588-firefly-aio-cam-2ms2mf.dtsi"
 #include "rk3588-firefly-demo.dtsi"
```

* Compile
```
./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
```

#### HDMI TO MIPI_CSI(RK628D) Compile
* modify dts
```
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q.dts
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q.dts
@@ -7,9 +7,9 @@
 /dts-v1/;

 #include "rk3588-firefly-aio-3588q.dtsi"
-#include "rk3588-firefly-aio-cam-8ms1m.dtsi"
+//#include "rk3588-firefly-aio-cam-8ms1m.dtsi"
 //#include "rk3588-firefly-aio-cam-2ms2mf.dtsi"
-//#include "rk3588-firefly-aio-3588q-tf-hdmi-mipi-rk628.dtsi"
+#include "rk3588-firefly-aio-3588q-tf-hdmi-mipi-rk628.dtsi"
 #include "rk3588-firefly-demo.dtsi"
```
* Compile
```
./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
```

### Step by step to compile

* Compile the kernel:

```
cd ~/path/to/sdk/kernel-5.10
export PATH=../prebuilts/clang/host/linux-x86/clang-r416183b/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64  firefly_defconfig android-11.config pcie_wifi.config
msk ARCH=arm64   BOOT_IMG=../rockdev/Image-aio_3588sjd4/boot.img aio-3588sjd4.img -j8
```

* Compile uboot:

```
cd ~/path/to/sdk/u-boot/
./make.sh rk3588
```

* Compile Android：

```
cd ~/path/to/sdk/
source build/envsetup.sh
lunch aio_3588sjd4-userdebug
make installclean
make -j8
./mkimage.sh
```

### Packaged into unified firmware update.img

After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command：
```
./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
```
After packaging, it will be in rockdev/Image-aio_3588sjd4/ Generate unified firmware under the directory: (product name )XXX_XXX_date XXX.img

It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create up
date.img and store it in rockdev \ Image directory.