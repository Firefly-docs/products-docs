# Compile Android8.1

### Download Android SDK

Due to the larger SDK, you can choose the cloud disk to download Firefly-RK3399_Android8.1_xxx.7za from the download page (you can directly use the Firefly-RK3399 source package):

[Android8.1 SDK Source]

After the download is complete, verify the MD5 code before decompression:
```
$md5sum ~/firefly_rk3399_android8.1_git_20211217.7z.001
$md5sum ~/firefly_rk3399_android8.1_git_20211217.7z.002
$md5sum ~/firefly_rk3399_android8.1_git_20211217.7z.003

48b4cfcb538cdc39ea53349989691617  firefly_rk3399_android8.1_git_20211217.7z.001
05776d4966d92efdcb88170355d8fbd4  firefly_rk3399_android8.1_git_20211217.7z.002
c83f54d89b4cde6eb620b92cd2e99a05  firefly_rk3399_android8.1_git_20211217.7z.003
```

Then unzip:
```
mkdir -p ~/proj/roc-rk3399-pc
cd ~/proj/roc-rk3399-pc
7z x ~/firefly_rk3399_android8.1_git_20211217.7z.001 -r -o.
git reset --hard
```

Update the remote warehouse:
```
git remote rm origin
git remote add gitlab git@gitlab.com:TeeFirefly/firenow-oreo-rk3399.git
```

Synchronize the source code from gitlab:
```
git pull gitlab firefly-rk3399:firefly-rk3399
```

You can also view the source code online at the following address:
[[https://gitlab.com/TeeFirefly/firenow-oreo-rk3399#]](https://gitlab.com/TeeFirefly/firenow-oreo-rk3399#)

[Android8.1 SDK Source]: http://en.t-firefly.com/doc/download/52.html#other_100
## AIO-3399C product compilation method

### HDMI display compilation


```
./FFTools/make.sh  -d rk3399-firefly-aioc -j8 -l rk3399_firefly_aioc_mid-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_mid-userdebug
```

* HDMI + lvds display compilation

* Single LVDS
```
./FFTools/make.sh  -d rk3399-firefly-aioc-lvds-HSX101H40C -j8 -l rk3399_firefly_aioc_lvds_mid-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_lvds_mid-userdebug
```

## Manually compile Android 8.1

Before compiling, execute the following command to configure environment variables:
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* Compile kernel：

```
cd ~/proj/AIO-3399C/kernel/
make ARCH=arm64 firefly_defconfig
make -j8 ARCH=arm64 rk3399-firefly-aioc.img
```

* Compile uboot：

```
cd ~/proj/AIO-3399C/u-boot/
make rk3399_defconfig
make ARCHV=aarch64 -j8
```

* Compile Android：

```
cd ~/proj/AIO-3399C/
source build/envsetup.sh
lunch rk3399_firefly_aioc_mid-userdebug
make -j8
./mkimage.sh
```


## Use Firefly official script to compile
* Compile the kernel separately：

```
cd ~/proj/AIO-3399C/
./FFTools/make.sh -k -j8
```

* Compile the uboot separately：

```
cd ~/proj/AIO-3399C/
./FFTools/make.sh -u -j8
```

* Compile Android upper layer separately：

```
cd ~/proj/AIO-3399C/
./FFTools/make.sh -a -j8
```

* Compile ubooot, kernel, Android at the same time：

```
cd ~/proj/AIO-3399C/
./FFTools/make.sh -j8
```
## Packaged into unified firmware 

After compiling, you can use Firefly's official script to package into unified firmware, execute the following command：

```
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_mid-userdebug
```

According to different -l XXX-userdebug parameters, the packaged unified firmware will be stored in different directories (rockdev/image-XXX/): product name XXX_XXX_date XXX.img

It is also very simple to package the unified firmware `update.img` under Windows, copy the compiled files to the  `rockdev/Image` directory of AndroidTool, and then run the `mkupdate.bat` batch file under the `rockdev` directory Create `update.img` and store it in the `rockdev/Image` directory.

## Partition mirror

* boot.img：The Android initramfs image contains the basic file system of the Android root directory, which is responsible for initializing and loading the system partition.
* system.img： Android file system partition image in ext4 file system format.
* kernel.img： Kernel image.
* resource.img： Resource image, including boot pictures and kernel device tree.
* misc.img： The misc partition image is responsible for switching the startup mode and passing the parameters of the emergency mode.
* recovery.img： Recovery mode image.
* rk3399_loader_v1.12.112.bin：Loader file.
* uboot.img： U-Boot image file.
* trust.img： Arm trusted file (ATF) image file.
* parameter.txt： Partition layout and kernel command line.
* vendor.img： TODO
* oem.img： TODO
* baseparameter.img： TODO
## Other Android versions
* <font color=#ff0000 size=3>Main maintenance:</font>

   ["Compile Android7.1 Industry firmware"](compile_android7.1_industry_firmware.md)     ["Compile Android10.0"](compile_android10.0_firmware.md)  
* Support but not maintain:

   ["Compile Android7.1 firmware"](compile_android7.1_firmware.md)     ["Compile Android8.1"](compile_android8.1_firmware.md)  