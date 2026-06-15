### Overall Compilation
#### Public Compile
##### HDMI

###### AIOC Standard Edition
```
./FFTools/make.sh  -d rk3399-firefly-aioc -j8 -l rk3399_firefly_aioc-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc-userdebug
```

###### AIOC AI version
```
./FFTools/make.sh  -d rk3399-firefly-aioc-ai -j8 -l rk3399_firefly_aioc_ai-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai-userdebug
```

#### Display DM-M10R800 Compile
##### LVDS+HDMI

###### AIOC Standard Edition
```
./FFTools/make.sh  -d rk3399-firefly-aioc-lvds-HSX101H40C -j8 -l rk3399_firefly_aioc_lvds-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_lvds-userdebug
```

###### AIOC AI version
```
./FFTools/make.sh  -d rk3399-firefly-aioc-ai-lvds-HSX101H40C -j8 -l rk3399_firefly_aioc_ai_lvds-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_lvds-userdebug
```

#### Display DM-M10R800 V2 Compile
##### MIPI_DSI1+HDMI
###### AIOC AI version
```
./FFTools/make.sh  -d rk3399-firefly-aioc-ai-mipi101-JDM101014_BC45_A1 -j8 -l rk3399_firefly_aioc_ai_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_mipi-userdebug
```

#### Display DM-M10R800 V3 Compile
##### MIPI_DSI1+HDMI
###### AIOC AI version
```
./FFTools/make.sh  -d rk3399-firefly-aioc-ai-mipi101-BSD1218-A101KL68 -j8 -l rk3399_firefly_aioc_ai_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_mipi-userdebug
```



### Compile step by step ROC-RK3328-PC

Before compiling, execute the following command to configure environment variables:
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

*    Compile kernel:
```
cd ~/proj/ROC-RK3328-PC//kernel/
make ARCH=arm64 firefly_defconfig
Standard Edition: 	make -j8 ARCH=arm64 rk3399-firefly-aioc.img
AI version: 		make -j8 ARCH=arm64 rk3399-firefly-aioc-ai.img
```

*    Compile uboot:
```
cd ~/proj/ROC-RK3328-PC/u-boot/
make rk3399_defconfig
make ARCHV=aarch64 -j8
```

* Compile Android:
```
cd ~/proj/ROC-RK3328-PC/
source build/envsetup.sh
Standard Edition: lunch rk3399_firefly_aioc-userdebug
AI version: lunch rk3399_firefly_aioc_ai-userdebug
make -j8
./mkimage.sh
```

### Packaged into unified firmware 

After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command：
```
Standard Edition:./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc-userdebug
AI version:./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai-userdebug
```

According to different -l XXX-userdebug parameters, the packaged unified firmware will be stored in different directories (rockdev/image-XXX/): product name XXX_XXX_date XXX.img

It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create update.img and store it in rockdev \ Image directory.



## Flash partition image

When compiling, executing ./mkimage.sh will repackage boot.img and system.img, and copy other related image files to the directory rockdev / Image-rk3399_firefly /. The following lists the image files used by general firmware:

*    boot.img ：The initial file image of Android is responsible for initializing and loading the system partition.
*    kernel.img ：Kernel image.
*    misc.img ：misc partition image, responsible for starting mode switching and parameter transfer of emergency mode.
*    parameter.txt ：emmc partition information
*    recovery.img ：First aid mode image。
*    resource.img ：The resource image contains the boot tree and the device tree information of the kernel.
*    system.img ：Android system partition image, ext4 file system format.
*    trust.img ：Hibernation and wake-up related files
*    rk3399_loader_v1.08.106.bin ：Loader file
*    uboot.img ：uboot file



Please refer to [Upgrade the firmware](03-upgrade_firmware.md) article to program partition image file.

If you are using a Windows system, copy the above image file to the rockdev \ Image directory of the AndroidTool (firmware upgrade tool under Windows), and then refer to the upgrade document to burn the partition image. This has the advantage of using the default configuration. No need to modify the file path.<br />

update.img facilitates the release of firmware for end users to upgrade the system. It is more convenient to use partition images in general development.

<font color=#ff0000 size=4>**note:**</font> <br />
<font color=#ff0000>When using the upgrade tool to upgrade the industry version of the firmware, if the original tvbox version of the firmware is burned, you need to short-circuit emmc or flash erase first. And pay attention to use the matching upgrade tool</font><br />

Please refer to [Instructions for writing with USB cable (important)](02-upgrade_table.md)