**Download compilation instructions:**
At present, the Android7.1 version does not do major maintenance. Please compile and select the Android7.1 industry version<font color="#dd0000">(Main maintenance)</font>. This version is more widely used in the fields of industry and tablets and boxes. It has a stable performance and has been verified by mass production. This version As the main maintenance version of our company, it is applicable to all models of our RK3399 system.


# Compile Android7.1 firmware

**The Android SDK source package is relatively large, and the Android7.1 source package can be obtained as follows:**[Android7.1 SDK]

After downloading, verify the MD5 code:

```
$md5sum ~/firefly_rk3399_android7.1_git_20211220.7z.001
$md5sum ~/firefly_rk3399_android7.1_git_20211220.7z.002
$md5sum ~/firefly_rk3399_android7.1_git_20211220.7z.003

90145747b33ad6834cd065b594f6687f  firefly_rk3399_android7.1_git_20211220.7z.001
6f28a1d66b1d168a89bfd3a7923775f8  firefly_rk3399_android7.1_git_20211220.7z.002
dc4d87ba8574900eed84f60f9cbe3646  firefly_rk3399_android7.1_git_20211220.7z.003
```

After confirming that it is correct, you can unzip:

```
mkdir -p ~/proj/ROC-RK3399-PC-PLUS
cd ~/proj/ROC-RK3399-PC-PLUS
7z x ~/firefly_rk3399_android7.1_git_20211220.7z.001 -r -o.
git reset --hard
```

* Update

```
#1.Enter root dir
cd ~/proj/ROC-RK3399-PC-PLUS

#2. pull bundle.git first
git clone https://gitlab.com/TeeFirefly/rk3399-nougat-bundle.git .bundle

# 3.If clone fail, get bundle package from [Android7.1 SDK]
# The decompression instructions are as follows:

7z x rk3399-nougat-bundle-20190612.7z -r -o. && mv rk3399-nougat-bundle/ .bundle/

# 4.Update the SDK, and subsequent updates do not need to pull the remote warehouse again, just execute the following command

.bundle/update

# 5.Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD
```

## ROC-RK3399-PC-PLUS Product compilation method

### HDMI display compilation

```
./FFTools/make.sh -d rk3399-roc-pc -j8 -l ROC_RK3399_PC-userdebug
./FFTools/mkupdate/mkupdate.sh -l ROC_RK3399_PC-userdebug
```


## Manual compilation ROC-RK3399-PC-PLUS

Before compiling, execute the following command to configure environment variables:

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* Compile the kernel:


```
cd ~/proj/ROC-RK3399-PC-PLUS/kernel/
make ARCH=arm64 firefly_defconfig
make -j8 ARCH=arm64 rk3399-roc-pc.img
```

* Compile U-boot:

```
cd ~/proj/ROC-RK3399-PC-PLUS/u-boot/
make rk3399_box_defconfig
make ARCHV=aarch64 -j8
```

* Compile Android:

```
cd ~/proj/ROC-RK3399-PC-PLUS/
source build/envsetup.sh
lunch rk3399_roc_pc_plus-userdebug
make -j8
./mkimage.sh
```

* Compile the kernel separately：

```
cd ~/proj/ROC-RK3399-PC-PLUS/
./FFTools/make.sh -k -j8
```

* Compile the uboot separately：

```
cd ~/proj/ROC-RK3399-PC-PLUS/
./FFTools/make.sh -u -j8
```

* Compile Android upper layer separately：

```
cd ~/proj/ROC-RK3399-PC-PLUS/
./FFTools/make.sh -a -j8
```

* Compile ubooot, kernel, Android at the same time：

```
cd ~/proj/ROC-RK3399-PC-PLUS/
./FFTools/make.sh -j8
```
## Package into a unified firmware 

After compiling, execute the following command first, then use Firefly official script to package into a unified SD card firmware, execute the following command:

```
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus-userdebug
```

According to different -l XXX-userdebug parameters, the packaged unified firmware will be stored in different directories (rockdev/image-XXX/): product name XXX_XXX_date XXX.img

Packing the unified firmware `update.img` under Windows is also very simple. Copy the compiled files to the `rockdev/Image` directory of AndroidTool, and then run the `mkupdate.bat` batch file under the `rockdev` directory. Create `update.img` and store it in the `rockdev/Image` directory.


## Generate firmware for tf card startup

After compiling, execute the following command first, then use Firefly official script to package into a unified SD card firmware, execute the following command:
```
./mkimage sdboot
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus-userdebug
```

## Flash partition image

When compiling, executing ./mkimage.sh will repackage boot.img and system.img, and copy other related image files to the directory rockdev/Image-roc_3399_pc/. The following lists the image files used by general firmware:

* boot.img: Android's initial file image, responsible for initializing and loading the system partition.
* kernel.img: kernel image.
* misc.img: misc partition image, responsible for starting mode switching and parameter transfer in emergency mode.
* parameter.txt: the partition information of emmc
* recovery.img: emergency mode image.
* resource.img: resource image, including boot image and kernel device tree information.
* system.img: Android system partition image, ext4 file system format.
* trust.img: files related to wake-up from sleep
* rk3399_loader_v1.08.106.bin: Loader file
* uboot.img: uboot file

Please refer to ["Upgrade Firmware"](03-upgrade_firmware.md) to burn the partition image file.

If you are using a Windows system, copy the above image file to the `rockdev/Image` directory of AndroidTool (firmware upgrade tool under Windows), and then refer to the upgrade document to burn the partition image. This has the advantage of using the default Just configure it without modifying the path of the file.

update.img facilitates the release of firmware for end users to upgrade the system. It is more convenient to use partition image during development.

[Android7.1 SDK]: http://en.t-firefly.com/doc/download/78.html#other_18

