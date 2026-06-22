# Compile the Android5.1 firmware

## Download the Android SDK  

**The source code package of the Android SDK is large (approximately 6.3G), and the source code package can be obtained by:**

* [[Download Link]](http://en.t-firefly.com/doc/download/47.html#other_35)

verify the MD5 code when the download is complete:  

```bash
$md5sum ~/firefly_rk3288_rk3128_android5.1_git_20211216.7z.001
$md5sum ~/firefly_rk3288_rk3128_android5.1_git_20211216.7z.002

3d7a59a84e059cd55a68304d47a5462b  firefly_rk3288_rk3128_android5.1_git_20211216.7z.001
1979bd263b51bec27ec10eceb500731b  firefly_rk3288_rk3128_android5.1_git_20211216.7z.002
```

Once confirmed, you will be able to extract:

```bash
mkdir -p ~/proj/firefly_rk3288_android5.1
cd ~/proj/firefly_rk3288_android5.1
7z x ~/firefly_rk3288_rk3128_android5.1_git_20211216.7z.001 -r -o.
git reset --hard
```

You can update directly from `gitlab` in the future

```bash
git pull gitlab Firefly-RK3288:Firefly-RK3288
```

## Complie

### Overall Compilation
#### Public Compile
##### HDMI


```bash
./FFTools/make.sh -d firefly-rk3288-aio-3288c -j8 -l rk3288_aio_3288c_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3288_aio_3288c_box-userdebug
```



#### Display DM-M10R800 Compile
##### LVDS+HDMI

```
./FFTools/make.sh -d firefly-rk3288-aio-3288c_lvds -j8 -l rk3288_aio_3288c_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3288_aio_3288c_box-userdebug
```


### Compile step by step

* Compile uboot

```bash
cd ~/proj/firefly_rk3288_android5.1/u-boot
make rk3288_defconfig
make -j8
```

* Compile kernel

```bash
cd ~/proj/firefly_rk3288_android5.1/kernel
make firefly_defconfig
make -j8 firefly-rk3288-aio-3288c.img
```


* Compile Android

```bash
cd ~/proj/firefly_rk3288_android5.1/
source build.sh
lunch rk3288_aio_3288c_box-userdebug
make -j8
./mkimage.sh
```

### Build variation

The default target build variation (TARGET_BUILD_VARIANT) is userdebug. There are three common variations, user, userdebug and eng, their differences are:  

* user  
    + Only install the modules whose label is user
    + Set the property ro.secure=1, and turn on the security check function
    + Set the property ro.debuggable=0, and turn off the application debugging function
    + Turn off the adb function by default
    + Turn on the Proguard obfuscator
    + Turn on the DEXPREOPT pre-compilation optimization

* userdebug
    + Install the modules whose label is user or debug
    + Set the property ro.secure=1, and turn on the security check function
    + Set the property ro.debuggable=1, and turn on the application debugging function
    + Turn on the ADB function by default
    + Turn on the Proguard obfuscator
    + Turn on the DEXPREOPT pre-compilation optimization

* eng
    + Install the modules whose label is user, debug or eng
    + Set the property ro.secure=0, and turn off the security check function
    + Set the property ro.debuggable=1, and turn on the application debugging function
    + Set the property ro.kernel.android.checkjni=1, and enable the JNI call checking
    + Turn on the adb function by default
    + Turn off the Proguard obfuscator
    + Turn off the DEXPREOPT pre-compilation optimization  

If the target build variation is user, the adb cannot get the root privileges.  
If you want to select a target build variation, you can add the parameters at the make command line, for example:  

```bash
make -j8 PRODUCT-rk3288_aio_3288c_box-user
make -j8 PRODUCT-rk3288_aio_3288c_box-userdebug
make -j8 PRODUCT-rk3288_aio_3288c_box-eng
```

## Flash the partition image

The `./mkimage.sh` from the previous step will repack the `boot.img` and `system.img` and copy other related image files to the `rockdev/Image-rk3288_aio_3288c_box/` directory. The following lists the image files used by the general firmware:

* boot.img ：the initial file image for Android is responsible for initializing and loading the system partition.
* kernel.img ：kernel image.
* misc.img ：misc partition image is responsible for initiating the mode switching and passing the parameter in the emergency mode.
* recovery.img ：image in the emergency mode.
* resource.img ：resource image, which contains the boot image and the device tree blob information of the kernel.
* system.img ：system partition image for Android, it is ext4 file system format.

Please refer to [upgrade_firmware](upgrade_firmware.md) to flash the partition image file.  
If the Windows system is used, you can copy the image file above to the `rockdev\Image` directory of the AndroidTool (firmware upgrade tool under Windows), and then refer to the upgrade document to flash the partition image, the benefit is to use the default configuration without modifying the path of the file.

## Pack into a unified firmware update.img

After compiling, you can use the script of Firefly to pack into the unified firmware, the command is:

```bash
./FFTools/mkupdate/mkupdate.sh -l rk3288_aio_3288c_box-userdebug
```

The firmware is generated in the `rockdev/Image-rk3288_aio_3288c_box/` directory after packing.

It is easy to pack the unified firmware update.img under Windows, you can copy the file to the `rockdev\Image` directory of the AndroidTool by following the previous steps, and then run the `mkupdate.bat` batch file in the `rockdev` directory to create the `update.img` and save it to the `rockdev\Image` directory.  

The `update.img` is easy to release the firmware and is used for upgrading the system by the end user. It is convenient to use a partition image for general development.