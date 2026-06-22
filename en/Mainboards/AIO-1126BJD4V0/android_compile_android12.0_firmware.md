# Compile Android12.0 firmware

## Download Android12.0 SDK

SDK source code and bundle compression package are stored in the Google Drive.

### Download Android SDK
* The SDK can be obtained by email. Send the order number to <font color=red>sales@t-firefly.com</font> and indicate the required SDK name [firefly_rk3588_android12_git_20240704]()

* After downloading, verify the MD5 code:
   ```
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.001
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.002
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.003
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.004
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.005
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.006
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.007

    1ce2a5ab4e3a9f0a4d6d8cd47ca4981d  firefly_rk3588_android12_git_20240704.7z.001
    553fb5ee2cc5623d97bfb19ba299605b  firefly_rk3588_android12_git_20240704.7z.002
    cb9cce227a79356fa57c0781a426fd89  firefly_rk3588_android12_git_20240704.7z.003
    1461a09bacbef0cb3c9a9008e8483e93  firefly_rk3588_android12_git_20240704.7z.004
    d154b39087bbf410a7973da97a6a9ecb  firefly_rk3588_android12_git_20240704.7z.005
    7cfc42443caa57cb4cf474bf3e78b5ce  firefly_rk3588_android12_git_20240704.7z.006
    9ad4374d89212a882dc00541769732f8  firefly_rk3588_android12_git_20240704.7z.007

    ```
* After confirming that it is correct, we can unzip:

   ```
    cd ~/proj/
    7z x ./firefly_rk3588_android12_git_20240704.7z.001 -oRK3588_Android12.0
    cd ./RK3588_Android12.0
    git reset --hard
   ```

   <font color=red>**Attention: To avoid unnecessary errors, please do not unzip the SDK in shared folders, mounted folders or non-english directories.**</font>

#### Second, Update SDK
Note: Be sure to update the remote warehouse after decompression. The following is how to update from gitlab:
```
1. Enter the SDK root directory
cd ~/proj/RK3588_Android12.0

2. Download remote bundle repository
git clone https://gitlab.com/T-Firefly/rk3588-android12.0-bundle.git .bundle

3. If the download warehouse fails, it may be stuck or failed problems during synchronization. 
we can download and unzip it from the cloud disk link below to the SDK root directory.

7z x rk3588-android12.0-bundle.7z  -r -o. && mv rk3588-android12.0-bundle .bundle

4. Update the SDK, and subsequent updates do not need to pull the remote warehouse again, just execute the following command
.bundle/update

5. Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD

```

Google Driver [Android12.0 Bundle]()。



## AIO-1126BJD4V0 product compilation method

### 整体编译

#### HDMI 固件编译 
```
./FFTools/make.sh -d  -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```
#### 显示屏 DM-M10R800 V2 固件编译：
```
./FFTools/make.sh -d  -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```

#### 双目摄像头 CAM-2MS2MF 编译：
##### HDMI+CAM-2MS2MF
* 修改 dts
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

* 编译
```
./FFTools/make.sh -d  -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```

#### HDMI TO MIPI_CSI(RK628D) 编译
* 修改 dts
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
* 编译
```
./FFTools/make.sh -d  -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```

### 分步编译

* 编译 kernel：

```
cd ~/proj/RK3588_Android12.0/kernel-5.10
export PATH=../prebuilts/clang/host/linux-x86/clang-r416183b/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64 
msk ARCH=arm64   BOOT_IMG=../rockdev//boot.img .img -j8

```

* 编译 uboot：

```
cd ~/proj/RK3588_Android12.0/u-boot/
./make.sh rv1126b
```

* 编译 Android：

```
cd ~/proj/RK3588_Android12.0/
source build/envsetup.sh
lunch 
make installclean
make -j8
./mkimage.sh
```

### 打包成统一固件 update.img

编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：
```
./FFTools/mkupdate/mkupdate.sh -l 
```
打包完成后将在rockdev// 目录下生成统一固件： product名XXX_XXX_日期XXX.img

在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。
## Some Introduction about Compiling
### Android12.0 can’t be upgraded directly kernel.img and resource.img

`kernel.img` and `resource.img` for Android12.0 are included in `boot.img`, After compiling the kernel, you need to run the `./mkimage.sh` command in the android root directory to repackage `boot.img`, and then upgrade `boot.img` of the rockdev// directory.

### Compiling kernel generation separately boot.img

Principle of compilation: in the kernel directory, the generated `kernel.img` And `resource.img` Replace with old `boot.img`, so you need to use BOOT_IMG=XXX parameter specification `boot.img` when compiling. The command is as follows:

```
cd ~/proj/RK3588_Android12.0/kernel-5.10
export PATH=../prebuilts/clang/host/linux-x86/clang-r416183b/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64  firefly_defconfig android-11.config pcie_wifi.config
msk ARCH=arm64   BOOT_IMG=../rockdev//boot.img .img -j8
```

After compiling, `boot.img` in the kernel directory can be directly upgrade.

## Partition image

When compiling, executing `./mkimage.sh` will repackage `boot.img` and `super.img`, and copy other related image files to the directory rockdev//. The following lists the image files used by general firmware:

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

Q: Error occurred in git clone remote bundle repository while [updating SDK](android_compile_android12.0_firmware.html#second-update-sdk):

```
$ git clone https://gitlab.com/T-Firefly/rk3588-android12.0-bundle.git .bundle
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

