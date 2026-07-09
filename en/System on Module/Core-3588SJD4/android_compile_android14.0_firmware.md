# Compile Android14.0 firmware

## Download Android14.0 SDK

SDK source code and bundle compression package are stored in the Google Drive.

### Download Android SDK
* The SDK can be obtained by email. Send the order number to <font color=red>sales@t-firefly.com</font> and indicate the required SDK name [firefly_rk3588_android14_git_20240822](https://en.t-firefly.com/doc/download/149.html)

* After downloading, verify the MD5 code:

    ```
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.001
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.002
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.003
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.004
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.005
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.006
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.007
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.008
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.009
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.010
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.011

    c9cc4863a3ad3b48f3c74d1f061d0438  firefly_rk3588_android14_git_240822.7z.001
    e81083e1b9b435e4e666f7d08dcd1bd6  firefly_rk3588_android14_git_240822.7z.002
    49b7aace84ab01b9a014b410fa26f42f  firefly_rk3588_android14_git_240822.7z.003
    1632fc98d881d5f4a8ed953afc9c73b1  firefly_rk3588_android14_git_240822.7z.004
    f581633cefdef6103de584501612e5dd  firefly_rk3588_android14_git_240822.7z.005
    f0e591f54a8969755775642567dabb17  firefly_rk3588_android14_git_240822.7z.006
    ee3d00d688ac0ad2bb43f8fe73f354bd  firefly_rk3588_android14_git_240822.7z.007
    09dc458646cce9f53a792d68e0ece3ec  firefly_rk3588_android14_git_240822.7z.008
    add00c4da466d50782ea5dba279c72d1  firefly_rk3588_android14_git_240822.7z.009
    51438ba104f0a67fd4fa09b5fee0d301  firefly_rk3588_android14_git_240822.7z.010
    b733f1d13c070862b5af6ce71052c020  firefly_rk3588_android14_git_240822.7z.011

    ```

* After confirming that it is correct, we can unzip:

    ```
    cd ~/proj/
    7z x ./firefly_rk3588_android14.0_git_20240822.7z.001 -oRK3588_Android14.0
    cd ./RK3588_Android14.0
    git reset --hard
    ```

   <font color=red>**Attention: To avoid unnecessary errors, please do not unzip the SDK in shared folders, mounted folders or non-english directories.**</font>

#### Second, Update SDK
Note: Be sure to update the remote warehouse after decompression. The following is how to update from gitlab:
```
1. Enter the SDK root directory
cd ~/proj/RK3588_Android14.0

2. Download remote bundle repository
git clone https://gitlab.com/T-Firefly/rk3588-android14.0-bundle.git .bundle

3. If the download warehouse fails, it may be stuck or failed problems during synchronization. 
we can download and unzip it from the cloud disk link below to the SDK root directory.

7z x rk3588-android14.0-bundle.7z  -r -o. && mv rk3588-android14.0-bundle .bundle

4. Update the SDK, and subsequent updates do not need to pull the remote warehouse again, just execute the following command
.bundle/update

5. Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD

```

Google Driver [Android14.0 Bundle](https://en.t-firefly.com/doc/download/149.html#other_768)。


## Core-3588SJD4 product compilation method

### The overall compilation

#### HDMI Firmware Compilation
```
./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
```

#### 10.1‘ V3S MIPI DSI0 Firmware Compilation：
```
./FFTools/make.sh -d aio-3588sjd4-mipi101-BSD1218-A101KL68 -j8 -l aio_3588sjd4_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4_mipi-userdebug
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
./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
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
./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
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
## Some Introduction about Compiling
### Android14.0 can’t be upgraded directly kernel.img and resource.img

`kernel.img` and `resource.img` for Android14.0 are included in `boot.img`, After compiling the kernel, you need to run the `./mkimage.sh` command in the android root directory to repackage `boot.img`, and then upgrade `boot.img` of the rockdev/Image-aio_3588sjd4/ directory.

### Compiling kernel generation separately boot.img

Principle of compilation: in the kernel directory, the generated `kernel.img` And `resource.img` Replace with old `boot.img`, so you need to use BOOT_IMG=XXX parameter specification `boot.img` when compiling. The command is as follows:

```
cd ~/proj/RK3588_Android14.0/kernel-6.1
export PATH=../prebuilts/clang/host/linux-x86/clang-r487747c/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64 firefly_defconfig android-14.config rk3576.config
msk ARCH=arm64 BOOT_IMG=../rockdev/Image-aio_3588sjd4/boot.img aio-3588sjd4.img -j8
```

After compiling, `boot.img` in the kernel directory can be directly upgrade.

## Partition image

When compiling, executing `./mkimage.sh` will repackage `boot.img` and `super.img`, and copy other related image files to the directory rockdev/Image-aio_3588sjd4/. The following lists the image files used by general firmware:

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

