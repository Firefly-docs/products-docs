### HDMI display compilation


```
./FFTools/make.sh  -d rk3399-roc-pc -j8 -l rk3399_roc_pc_plus-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus-userdebug
```

* HDMI + EDP display compilation

* Single LVDS
```
./FFTools/make.sh  -d rk3399-roc-pc-edp -j8 -l rk3399_roc_pc_plus-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus-userdebug
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
cd ~/proj/ROC-RK3399-PC-PLUS/kernel/
make ARCH=arm64 firefly_defconfig
make -j8 ARCH=arm64 rk3399-roc-pc.img
```

* Compile uboot：

```
cd ~/proj/ROC-RK3399-PC-PLUS/u-boot/
make rk3399_defconfig
make ARCHV=aarch64 -j8
```

* Compile Android：

```
cd ~/proj/ROC-RK3399-PC-PLUS/
source build/envsetup.sh
lunch rk3399_roc_pc_plus-userdebug
make installclean
make -j8
./mkimage.sh
```


## Use Firefly official script to compile
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
## Packaged into unified firmware 

After compiling, you can use Firefly's official script to package into unified firmware, execute the following command：

```
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus-userdebug
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