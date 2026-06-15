### HDMI display compilation

1.AIOC Standard Edition
```
./FFTools/make.sh -d rk3399-firefly-aioc -j8 -l rk3399_firefly_aioc_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_box-userdebug
```

2.AIOC AI version
```
./FFTools/make.sh -d rk3399-firefly-aioc-ai -j8 -l rk3399_firefly_aioc_ai_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_box-userdebug
```

### HDMI+lvds compilation
1.AIOC Standard Edition

* Dual LVDS
```
./FFTools/make.sh -d rk3399-firefly-aioc-lvds -j8 -l rk3399_firefly_aioc_lvds_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_lvds_box-userdebug
```

* Single LVDS
```
./FFTools/make.sh -d rk3399-firefly-aioc-lvds-HSX101H40C -j8 -l rk3399_firefly_aioc_lvds_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_lvds_box-userdebug
```

2.AIOC AI version
* Dual LVDS
```
./FFTools/make.sh -d rk3399-firefly-aioc-ai-lvds -j8 -l rk3399_firefly_aioc_ai_lvds_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_lvds_box-userdebug
```

* Single LVDS
```
./FFTools/make.sh -d rk3399-firefly-aioc-ai-lvds-HSX101H40C -j8 -l rk3399_firefly_aioc_ai_lvds_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_lvds_box-userdebug
```


## Manual compilation ROC-RK3399-PC-Pro

Before compiling, execute the following command to configure environment variables:

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* Compile the kernel:


```
cd ~/proj/ROC-RK3399-PC-Pro/kernel/
make ARCH=arm64 firefly_defconfig
Standard version: make -j8 ARCH=arm64 rk3399-firefly-aioc.img
AI version: make -j8 ARCH=arm64 rk3399-firefly-aioc-ai.img
```

* Compile U-boot:

```
cd ~/proj/ROC-RK3399-PC-Pro/u-boot/
make rk3399_box_defconfig
make ARCHV=aarch64 -j8
```

* Compile Android:

```
cd ~/proj/ROC-RK3399-PC-Pro/
source build/envsetup.sh
Standard Edition: lunch rk3399_firefly_aioc_box-userdebug
AI version: lunch rk3399_firefly_aioc_ai_box-userdebug
make -j8
./mkimage.sh
```

## Use Firefly official script to compile
* Compile the kernel separately：

```
cd ~/proj/ROC-RK3399-PC-Pro/
./FFTools/make.sh -k -j8
```

* Compile the uboot separately：

```
cd ~/proj/ROC-RK3399-PC-Pro/
./FFTools/make.sh -u -j8
```

* Compile Android upper layer separately：

```
cd ~/proj/ROC-RK3399-PC-Pro/
./FFTools/make.sh -a -j8
```

* Compile ubooot, kernel, Android at the same time：

```
cd ~/proj/ROC-RK3399-PC-Pro/
./FFTools/make.sh -j8
```
## Package into a unified firmware 

After compiling, execute the following command first, then use Firefly official script to package into a unified SD card firmware, execute the following command:

```
Standard Edition:./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_box-userdebug
AI version: ./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_box-userdebug
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