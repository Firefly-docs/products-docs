# Compiling Android 8.1

## Prerequisite

### Hardware requiremnts

Recommended hardware requirement of development workstation compiling Android 8.1:

- 64 bit CPU
- 16GB  Physical memory + Swap memory
- 30GB  Free disk space is used for building, and the source tree takes about 8GB

See also the hardware and software configuration stated in Google official document:

- [https://source.android.com/setup/build/requirements](https://source.android.com/setup/build/requirements)
- [https://source.android.com/setup/initializing](https://source.android.com/setup/initializing)

### Software requiements

**Installing JDK 8**

``` shell
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```

**Installing required packages**

``` shell
sudo apt-get install git-core gnupg flex bison gperf libsdl1.2-dev \
  libesd0-dev libwxgtk2.8-dev squashfs-tools build-essential zip curl \
  libncurses5-dev zlib1g-dev pngcrush schedtool libxml2 libxml2-utils \
  xsltproc lzop libc6-dev schedtool g++-multilib lib32z1-dev lib32ncurses5-dev \
  lib32readline-gplv2-dev gcc-multilib libswitch-perl

sudo apt-get install gcc-arm-linux-gnueabihf \
  libssl1.0.0 libssl-dev \
  p7zip-full
```

## Downloading Android SDK

Due to the huge size of the Android SDK, it is not possible to directly host it in Gitlab. Therefore the `.git` directory is compressed as a `7z` archive file, uploaded to cloud storage server. The increment update of the SDK is saved in `git bundle` and hosted in Gitlab.

Please download `firefly_rk3328_android8.1_git_20211221` first:

* [Download](https://community.t-firefly.com/en/doc/download/34.html#other_197)

After the download completes, verify the MD5 checksum before extraction:

``` shell
$md5sum ~/firefly_rk3328_android8.1_git_20211221.7z.001
$md5sum ~/firefly_rk3328_android8.1_git_20211221.7z.001
$md5sum ~/firefly_rk3328_android8.1_git_20211221.7z.001

fda8c753df5ba4f20cdc9006f316d028  firefly_rk3328_android8.1_git_20211221.7z.001
e78f2bb73fcb2afdcbeece2cf6bc581b  firefly_rk3328_android8.1_git_20211221.7z.002
79b889bb5df551c2f69337b8a8cf3bbb  firefly_rk3328_android8.1_git_20211221.7z.003
```

Then extract it:

``` shell
mkdir -p  ~/proj/firefly_rk3328_android8.1
cd ~/proj/firefly_rk3328_android8.1
7z x ~/firefly_rk3328_android8.1_git_20211221.7z.001 -r -o.
git reset --hard
```

For first and subsequent SDK update, run the following commands:

```shell
# Update and apply the git bundle
test -d .bundle || \
  git clone \
    https://gitlab.com/TeeFirefly/rk3328-android8.1-oreo-bundle.git \
    .bundle

.bundle/update

# If the command above prompts "[Info]Already up to date!", it indicates that
# no SDK update is available and nothing need to do here.

# Otherwise, the latest commit of the SDK update is saved to the symbolic-ref
# FETCH_HEAD. Show the relation between the local branch and SDK update first.
git show-branch HEAD FETCH_HEAD
# If there're local commits, choose either rebase or merge to incorporate
# the SDK change:
# To rebase
git rebase FETCH_HEAD
# To merge
git merge FETCH_HEAD
```

## Compiling with Firefly Scripts

### Automatic compilation
``` shell
./build.sh rk3328-roc-cc
```
After executing the above command, it will compile the U-Boot, kernel, and upper layer of Android, sort out the partition Image and generate unified firmware `update.img`, and put it in the directory of `rockdev/image-rk3328_box/`

### Modular compilation
**Compiling configuration file**

<font color=#ff0000 size=3>Note</font>:"Compiling configuration file" is a prerequisite. You must complete this step before proceeding:

``` shell
./FFTools/make.sh rk3328-roc-cc 
```

**Compiling Kernel**

``` shell
./FFTools/make.sh -k -j8
```

**Compiling U-Boot**

``` shell
./FFTools/make.sh -u -j8
```

**Compiling Android**

``` shell
./FFTools/make.sh -a -j8
```

**Compiling all Image**

This will compile kernel, U-Boot and Android with a single command:

``` shell
./FFTools/make.sh -j8
```

## Compiling Without Script

Before compilation, execute the following commands to configure environment variables:

``` shell
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

**Compiling Kernel**

``` shell
make ARCH=arm64 rockchip_defconfig
make -j8 ARCH=arm64 roc-rk3328-cc.img
```

**Compiling U-Boot**

``` shell
make rk3328_box_defconfig
make ARCHV=aarch64 -j8
```

**Compiling Android**

``` shell
source build/envsetup.sh
lunch rk3328_box-userdebug
make installclean
make -j8
./mkimage.sh
```

## Packing Rockchp Firmware

**Packing Fimware in Linux**

After compiling you can use Firefly official script to pack all partition image files into the one true Rockchip firmware, by executing the following command:

``` shell
./FFTools/mkupdate/mkupdate.sh update
```

The resulting file is `rockdev/Image-rk3328_box/update.img`.

The RK firmware needs to use the `SD_Firmware_Tool` tool and the function mode to select `SD Startup` to create the boot card

**Packing Fimware in Windows**

It is also very simple in packaging Rockchip firmware `update.img` under Windows:

1. Copy all the compiled files in `rockdev/Image-rk3328_box/` to the `rockdev\Image` directory of AndroidTool
2. Run the `mkupdate.bat` batch file in the `rockdev` directory of AndroidTool.
3. `update.img` will be created in `rockdev\Image` directory.

## Flash Image 

Reference: [《Flashing to the eMMC》](flash_emmc.md)

[Getting Started]: getting_started.md
[FAQ]: faqs.md
[Serial Debug]: debug.md
[Building Linux Root Filesystem]: linux_build_ubuntu_rootfs.md
[Contact]: resource.md#community
[Raw Firmware]: getting_started.md#raw-firmware-format
[RK Firmware]: getting_started.md#rk-firmware-format
[Partition Image]: getting_started.md#partition-image
[SDCard Installer]: flash_sd.md#sdcard-installer
[Etcher]: flash_sd.md#etcher
[dd]: flash_sd.md#dd
[SD Firmware Tool]: flash_sd.md#sd-firmware-tool
[AndroidTool]: flash_emmc.md#androidtool
[upgrade_tool]: flash_emmc.md#upgrade-tool
[rkdeveloptool]: flash_emmc.md#rkdeveloptool
[Rockusb Mode]: flash_emmc.md#rockusb-mode
[Maskrom Mode]: flash_emmc.md#maskrom-mode
[Rockusb Driver]: flash_emmc.md#rockusb-driver
[ROC-RK3328-CC]: http://en.t-firefly.com/product/rocrk3328cc.html "ROC-RK3328-CC Official Website"
[Download Page]: http://en.t-firefly.com/doc/download/34.html
[Forum]: http://bbs.t-firefly.com/
[Facebook]: https://www.facebook.com/TeeFirefly
[Google+]: https://plus.google.com/u/0/communities/115232561394327947761
[Youtube]: https://www.youtube.com/channel/UCk7odZvUrTG0on8HXnBT7gA
[Twitter]: https://twitter.com/TeeFirefly
[Shop]: http://shop.t-firefly.com/
[USB Serial Adapter]: https://www.firefly.store/products/usb-to-uart-module-cp2104 
[5V2A US Adapter]: https://www.firefly.store/products/5v2a-us-adapter-3c-fcc-ce 
[eMMC Flash]: https://www.firefly.store/products
[Storage Map]: http://opensource.rock-chips.com/wiki_Partitions#Default_storage_map