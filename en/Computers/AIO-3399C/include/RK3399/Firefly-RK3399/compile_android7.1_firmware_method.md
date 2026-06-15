### HDMI display compilation

1. Compile HDMI+DP by default
```
./FFTools/make.sh -j8 -d rk3399-firefly -l rk3399_firefly_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_box-userdebug
```

2. EDP7.85 compilation
```
./FFTools/make.sh -j8 -d rk3399-firefly-edp -l rk3399_firefly_edp_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_edp_box-userdebug
```

3. MIPI7.85 compilation
```
./FFTools/make.sh -j8 -d rk3399-firefly-mipi -l rk3399_firefly_mipi_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_mipi_box-userdebug
```

## Manual compilation AIO-3399C

Before compiling, execute the following command to configure environment variables:

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* Compile the kernel:


```
cd ~/proj/AIO-3399C/kernel/
make ARCH=arm64 firefly_defconfig
make ARCH=arm64 firefly_defconfig
make -j8 ARCH=arm64 rk3399-firefly.img
```

* Compile U-boot:

```
cd ~/proj/AIO-3399C/u-boot/
make rk3399_box_defconfig
make ARCHV=aarch64 -j8
```

* Compile Android:

```
cd ~/proj/AIO-3399C/
source build/envsetup.sh
lunch rk3399_firefly_box-userdebug
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
## Package into a unified firmware 

After compiling, execute the following command first, then use Firefly official script to package into a unified SD card firmware, execute the following command:

```
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_box-userdebug
```

According to different -l XXX-userdebug parameters, the packaged unified firmware will be stored in different directories (rockdev/image-XXX/): product name XXX_XXX_date XXX.img

Packing the unified firmware `update.img` under Windows is also very simple. Copy the compiled files to the `rockdev/Image` directory of AndroidTool, and then run the `mkupdate.bat` batch file under the `rockdev` directory. Create `update.img` and store it in the `rockdev/Image` directory.


## Generate firmware for tf card startup

After compiling, execute the following command first, then use Firefly official script to package into a unified SD card firmware, execute the following command:
```
./mkimage sdboot
./FFTools/mkupdate/mkupdate.sh update
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