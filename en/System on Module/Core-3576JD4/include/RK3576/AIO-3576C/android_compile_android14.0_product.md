### The overall compilation

#### HDMI Firmware Compilation
```
./FFTools/make.sh -j8 -l rk3576_firefly_aio_3576_jd4-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_aio_3576_jd4-userdebug
```

#### 10.1‘ MIPI V3S Firmware Compilation：
```
./FFTools/make.sh -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```

#### 10.1‘ LVDS DM-M10R800 Firmware Compilation：
##### HDMI+LVDS
* modify dts
```
--- a/kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576c.dts
+++ b/kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576c.dts
@@ -21,11 +21,11 @@
 #define RS232          1
 #define RS485          1
 #define DP             1
 #define SATA           0
 #define PCIE           1
 #define EDP            0
-#define DSI            1
-#define LVDS           0
+#define DSI            0
+#define LVDS           1
```

```
./FFTools/make.sh -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```

### Step by step to compile

* Compile the kernel:

```
cd ~/proj/RK3576_Android14.0/kernel-6.1
export PATH=../prebuilts/clang/host/linux-x86/clang-r487747c/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64 firefly_defconfig android-14.config rk3576.config
msk ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3576_firefly_aio_3576c/boot.img rk3576-firefly-aio-3576c.img -j8
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
lunch rk3576_firefly_aio_3576_jd4-userdebug
make installclean
make -j8
./mkimage.sh
```

### Packaged into unified firmware update.img

After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command：
```
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_aio_3576_jd4-userdebug
```
After packaging, it will be in rockdev/Image-rk3576_firefly_aio_3576_jd4/ Generate unified firmware under the directory: (product name )XXX_XXX_date XXX.img

It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create up
date.img and store it in rockdev \ Image directory.