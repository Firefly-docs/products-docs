# Compile the Android firmware

The configuration requirements for compiling Android on the machine are high:

To compile Android, requirement of PC is:

* 64 bit CPU
* 16GB physical memory + swap memory
* 30GB of free disk space is used for building. In addition, about 25GB of disk space is used for the source code tree

Ubuntu 14.04 is officially recommended. However, Ubuntu 12.04 is confirmed to be supported,  providing the requirements of software and hardware in [http://source.android.com/source/building.html](http://source.android.com/source/building.html ) are met.
To initialize compiling environment, please reference [http://source.android.com/source/initializing.html](http://source.android.com/source/initializing.html ) .

* Install OpenJDK 7:

```
sudo apt-get install openjdk-7-jdk
```   

Tips: After installing openjdk-7-jdk, you might need to fix JDK's default link:

```
$ sudo update-alternatives --config java
$ sudo update-alternatives --config javac
```

to switch JDK version. SDK will switch to internal JDK path once the default one cannot be found. Therefore, in order to compile Android 5.1 and before, removing the JDK7 link is more preferable:

```
$ sudo /var/lib/dpkg/info/openjdk-7-jdk:amd64.prerm remove
```

* Ubuntu 12.04 packages install：

```
sudo apt-get install git gnupg flex bison gperf build-essential \
zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \
libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \
g++-multilib mingw32 tofrodos gcc-multilib ia32-libs \
python-markdown libxml2-utils xsltproc zlib1g-dev:i386 \
lzop libssl1.0.0 libssl-dev
```
 
* Ubuntu 14.04 packages install：

```
sudo apt-get install git-core gnupg flex bison gperf libsdl1.2-dev \
libesd0-dev libwxgtk2.8-dev squashfs-tools build-essential zip curl \
libncurses5-dev zlib1g-dev pngcrush schedtool libxml2 libxml2-utils \
xsltproc lzop libc6-dev schedtool g++-multilib lib32z1-dev lib32ncurses5-dev \
lib32readline-gplv2-dev gcc-multilib libswitch-perl \
libssl1.0.0 libssl-dev
```

## Download the Android SDK

The size of SDK is huge. Please download folder firefly_rk3288_rk3128_android5.1_git_20211216 from one of the following cloud storage:

* [Download](https://community.t-firefly.com/en/doc/download/37)

Please check the md5 checksum before proceeding：

```
$md5sum ~/firefly_rk3288_rk3128_android5.1_git_20211216.7z.001
$md5sum ~/firefly_rk3288_rk3128_android5.1_git_20211216.7z.002

3d7a59a84e059cd55a68304d47a5462b  firefly_rk3288_rk3128_android5.1_git_20211216.7z.001
1979bd263b51bec27ec10eceb500731b  firefly_rk3288_rk3128_android5.1_git_20211216.7z.002
```

If it is correct, uncompress it:

```  
mkdir -p ~/pro/firefly_rk3128_android5.1
cd ~/pro/firefly_rk3128_android5.1
7z x ~/firefly_rk3288_rk3128_android5.1_git_20211216.7z.001 -r -o.
git reset --hard 
git checkout fireprime
```

From now on, you can pull from gitlab:

```
git pull gitlab fireprime:fireprime
```

## Compile Android SDK
### Overall Compilation

#### Public Compile
##### HDMI

```
./FFTools/make.sh -d aio-3128c -j8 -l aio_3128c-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3128c-userdebug
```

#### Display DM-M10R800 Compile
##### LVDS+HDMI

```
./FFTools/make.sh -d aio-3128c-lvds -j8 -l aio_3128c-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3128c-userdebug
```

### Compile step by step

* Compile Uboot:
```
cd ~/pro/firefly_rk3128_android5.1/u-boot/
make rk3128_defconfig
make -j8
```

* Compile Kernel
```
cd ~/pro/firefly_rk3128_android5.1/kernel
make firefly_defconfig
make aio-3128c.img -j4
```

* Compile Android
```
cd ~/pro/firefly_rk3128_android5.1/
source build.sh
lunch aio_3128c-userdebug
make -j8
./mkimage.sh
```

TARGET_BUILD_VARIANT defaults to userdebug. There are three variants: user, userdebug and eng. The main differences are:

- user  
    + Only install modules with user tag
    + Set property ro.secure=1, to turn on security check.
    + Set property ro.debuggable=0, to close application debugging function.
    + Turn off adb by default.
    + Enable Proguard.
    + Enable DEXPREOPT to optimize applcation.

- userdebug  
    + Only install modules with userdebug tag
    + Set property ro.secure=1, to turn on security check.
    + Set property ro.debuggable=1, to turn on application debugging function.
    + Turn on adb by default.
    + Enable Proguard.
    + Enable DEXPREOPT to optimize applcation.

- eng  
    + Install modules with user, debug or eng tag.
    + Set property ro.secure=0, to turn off security check.
    + Set property ro.debuggable=1, to turn on  application debugging fucntion.
    + Turn on adb by default.
    + Disable Proguard.
    + Disable DEXPREOPT, not optimizing applcation.

If TARGET_BUILD_VARIANT is user, adb can not gain root permission.  
To switch target build variant, add parameter to the make command line:

```
make -j8 PRODUCT-rk312x-user
make -j8 PRODUCT-rk312x-userdebug
make -j8 PRODUCT-rk312x-eng
```

### Packaged into unified firmware

After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command:

```
./FFTools/mkupdate/mkupdate.sh -l aio_3128c-userdebug
```

According to different `-l XXX-userdebug` parameters, the packaged unified firmware will be stored in different directories (rockdev/image-XXX/): `(product_name)XXX_XXX_(date)XXX.img`

It is also very simple to package the unified firmware `update.img` under Windows. Copy the generated files to the `rockdev\Image` directory of AndroidTool, and then run the `mkupdate.bat` batch file under the `rockdev` directory to create `update.img` and store it in `rockdev\Image` directory.
 
## Flash the partition image

`./mkimage.sh` at previous step will repack `boot.img` and `system.img`, and copy other related image files to the `rockdev/Image-rk312x/` directory. The common image files are listed below:

* boot.img : Android's initramfs, to initialize and mount system partition.
* kernel.img : Kernel image.
* misc.img : Misc partition image, to switch boot mode and pass parameter in recovery mode.
* recovery.img : Recovery mode image.
* resource.img : Resource image, containing boot logo and kernel's device tree info.
* system.img : System partition image with ext4 filesystem format.

Please flash the image according to [Flash image](upgrade_firmware.md).  
If you are using Windows system to flash the images, please copy the image files mentioned above to `rockdev\Image` directory in AndroidTool (Image flash tool in Windows). Then follow instructions in the flash image guide. This is the easy way, using default configuration, no need to modify image file's path.