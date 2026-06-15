### The overall compilation

#### Public Firmware Compilation
```
./FFTools/make.sh -d aio-3588l -j8 -l aio_3588l-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588l-userdebug
```

### Step by step to compile

* Compile the kernel:

```
cd ~/proj/RK3588_Android12.0/kernel-5.10
export PATH=../prebuilts/clang/host/linux-x86/clang-r416183b/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64  firefly_defconfig android-11.config pcie_wifi.config
msk ARCH=arm64   BOOT_IMG=../rockdev/Image-aio_3588l/boot.img aio-3588l.img -j8
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