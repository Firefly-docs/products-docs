### The overall compilation

#### Public Firmware Compilation
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