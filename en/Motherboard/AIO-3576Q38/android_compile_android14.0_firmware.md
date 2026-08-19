# Compile Android14.0 firmware

## Download Android14.0 SDK

SDK source code and bundle compression package are stored in the Google Drive.

### Download Android SDK
* The SDK can be obtained by email. Send the order number to <font color=red>sales@t-firefly.com</font> and indicate the required SDK name [firefly_rk3576_android14.0_git_20241126]()

* After downloading, verify the MD5 code:

    ```
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.001
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.002
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.003
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.004
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.005
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.006

    e368c4274125e754eeea35a97306e47b  firefly_rk3576_android14.0_git_20241126.7z.001
    d4436c73b800523dbb1df2ae0d524b26  firefly_rk3576_android14.0_git_20241126.7z.002
    9f6a7ed4f01a8952c135cc5bf6fd4ecd  firefly_rk3576_android14.0_git_20241126.7z.003
    3d2babe7c2c9f737d23e8843c2df3117  firefly_rk3576_android14.0_git_20241126.7z.004
    1d9ae98cc3e0959de19bc1dd628ed112  firefly_rk3576_android14.0_git_20241126.7z.005
    09e35581108741bf4a564829edcd1a5f  firefly_rk3576_android14.0_git_20241126.7z.006

    ```

* After confirming that it is correct, we can unzip:

    ```
    cd ~/proj/
    7z x ./firefly_rk3576_android14.0_git_20241126.7z.001 -oRK3576_Android14.0
    cd ./RK3576_Android14.0
    git reset --hard
    ```

   <font color=red>**Attention: To avoid unnecessary errors, please do not unzip the SDK in shared folders, mounted folders or non-english directories.**</font>

#### Second, Update SDK
How to update code after downloading SDK:
```
1. Enter the SDK root directory
cd ~/proj/RK3576_Android14.0

2. we can download and unzip it from the cloud disk link below to the SDK root directory.

7z x rk3576-android14.0-bundle.7z  -r -o. && mv rk3576-android14.0-bundle .bundle

3. Update the SDK
.bundle/update

4. Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD

```

Google Driver [Android14.0 Bundle](https://en.t-firefly.com/doc/download/341.html#other_713)。

<!-- 2. Download remote bundle repository
git clone https://gitlab.com/T-Firefly/rk3576-android14.0-bundle.git .bundle

3. If the download warehouse fails, it may be stuck or failed problems during synchronization. 
we can download and unzip it from the cloud disk link below to the SDK root directory.
 -->


## AIO-3576Q38 product compilation method

### The overall compilation

#### HDMI Firmware Compilation
```
./FFTools/make.sh -j8 -l rk3576_firefly_aio_3576q38-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_aio_3576q38-userdebug
```

<!--
#### 10.1‘ MIPI V3S Firmware Compilation：
```
./FFTools/make.sh -j8 -l firefly_aio_3576q38_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l firefly_aio_3576q38_mipi-userdebug
```
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
./FFTools/make.sh -d rk3576-firefly-aio-3576q38 -j8 -l rk3576_firefly_aio_3576q38-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_aio_3576q38-userdebug
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
./FFTools/make.sh -d rk3576-firefly-aio-3576q38 -j8 -l rk3576_firefly_aio_3576q38-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_aio_3576q38-userdebug
```
-->
### Step by step to compile

* Compile the kernel:

```
cd ~/proj/RK3576_Android14.0/kernel-6.1
export PATH=../prebuilts/clang/host/linux-x86/clang-r487747c/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64 firefly_defconfig android-14.config rk3576.config
msk ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3576_firefly_aio_3576q38/boot.img rk3576-firefly-aio-3576q38.img -j8
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
lunch rk3576_firefly_aio_3576q38-userdebug
make installclean
make -j8
./mkimage.sh
```

### Packaged into unified firmware update.img

After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command：
```
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_aio_3576q38-userdebug
```
After packaging, it will be in rockdev/Image-rk3576_firefly_aio_3576q38/ Generate unified firmware under the directory: (product name )XXX_XXX_date XXX.img

It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create up
date.img and store it in rockdev \ Image directory.
## Some Introduction about Compiling
### Android14.0 can’t be upgraded directly kernel.img and resource.img

`kernel.img` and `resource.img` for Android14.0 are included in `boot.img`, After compiling the kernel, you need to run the `./mkimage.sh` command in the android root directory to repackage `boot.img`, and then upgrade `boot.img` of the rockdev/Image-rk3576_firefly_aio_3576q38/ directory.

### Compiling kernel generation separately boot.img

Principle of compilation: in the kernel directory, the generated `kernel.img` And `resource.img` Replace with old `boot.img`, so you need to use BOOT_IMG=XXX parameter specification `boot.img` when compiling. The command is as follows:

```
cd ~/proj/RK3576_Android14.0/kernel-6.1
export PATH=../prebuilts/clang/host/linux-x86/clang-r487747c/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64 firefly_defconfig android-14.config rk3576.config
msk ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3576_firefly_aio_3576q38/boot.img rk3576-firefly-aio-3576q38.img -j8
```

After compiling, `boot.img` in the kernel directory can be directly upgrade.

## Partition image

When compiling, executing `./mkimage.sh` will repackage `boot.img` and `super.img`, and copy other related image files to the directory rockdev/Image-rk3576_firefly_aio_3576q38/. The following lists the image files used by general firmware:

| Image | Instruction |
|-------|-------------|
| boot.img | including ramdis、kernel、dtb |
| dtbo.img | Device Tree Overlays |
| MiniLoaderAll.bin | including first level loader |
| misc.img | including recovery-wipe boot symbol information, after flashing it will enter recovery |
| parameter.txt | including partition information |
| recovery.img | including recovery-ramdis、kernel、dtb |
| super.img | including the contents of odm、vendor、system partitions |
| uboot.img | including uboot image |
| vbmeta.img | including avb verification information, used for AVB verification |
| update.img | including the above img files to be flashed, can be used for the tool to directly flash the whole image package |

For details about how to upgrade a partition image file, see [Upgrade the firmware via USB cable](upgrade_firmware.md).

## OTA Compilation 

Refer to the [OTA Compilation](android_compile_ota_package.md) section

## FAQs

### git clone remote bundle repository failed

Q: Error occurred in git clone remote bundle repository while [updating SDK](android_compile_Android14.0_firmware.html#second-update-sdk):

```
$ git clone https://gitlab.com/T-Firefly/rk3588-Android14.0-bundle.git .bundle
Cloning into '.bundle'...
remote: Enumerating objects: 20, done.
remote: Counting objects: 100% (20/20), done.
remote: Compressing objects: 100% (18/18), done.
error: RPC failed; curl 56 GnuTLS recv error (-9): A TLS packet with unexpected length was received.
fatal: the remote end hung up unexpectedly
fatal: early EOF
fatal: unpack-objects failed
```

A: The buffer size is insufficient and needs to be enlarged:

```
git config --global https.postBuffer 1048576000
```

