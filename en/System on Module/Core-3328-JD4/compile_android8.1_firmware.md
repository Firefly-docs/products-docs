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

Please download `RK3328_Android8.1_git_20190719.7z` first:

- [Google Drive](https://community.t-firefly.com/en/doc/download/62)

After the download completes, verify the MD5 checksum before extraction:

``` shell
$ md5sum /path/to/RK3328_Android8.1_git_20190719.7z.001
d2198123c17af2140a9336b2377e0e93  RK3328_Android8.1_git_20190719.7z.001

$ md5sum /path/to/RK3328_Android8.1_git_20190719.7z.002
9df675c96997b5e246549acf0789fa39  RK3328_Android8.1_git_20190719.7z.002

$ md5sum /path/to/RK3328_Android8.1_git_20190719.7z.003
56d54d78aa98a6737d171f631dc66960  RK3328_Android8.1_git_20190719.7z.003
```

Then extract it:

``` shell
mkdir -p ~/proj/rk3328
cd ~/proj/rk3328

# Extract the 7z archives to get full .git dir
7z x /path/to/RK3328_Android8.1_git_20190719.7z.001(Just unzip the first one and you'll get the full SDK)

# Reset the working directory to get full source codes
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
./build.sh rk3328-core-jd4
```
After executing the above command, it will compile the U-Boot, kernel and upper layer of Android, sort out the partition Image and generate unified firmware `update.img`, and put it in the directory of `rockdev/image-rk3328_core_jd4/`.

### Modular compilation
**Compiling configuration file**

<font color=#ff0000 size=3>Note</font>:"Compiling configuration file" is a prerequisite. You must complete this step before proceeding:

``` shell
./FFTools/make.sh rk3328-core-jd4
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
make -j8 ARCH=arm64 rk3328-core-jd4.img
```

**Compiling U-Boot**

``` shell
make rk3328_box_defconfig
make ARCHV=aarch64 -j8
```

**Compiling Android**

``` shell
source build/envsetup.sh
lunch rk3328_core_jd4-userdebug
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

The resulting file is `rockdev/Image-rk3328_core_jd4/update.img`.

The RK firmware needs to use the `SD_Firmware_Tool` tool and the function mode to select `SD Startup` to create the boot card

**Packing Fimware in Windows**

It is also very simple in packaging Rockchip firmware `update.img` under Windows:

1. Copy all the compiled files in `rockdev/Image-rk3328_core_jd4/` to the `rockdev\Image` directory of AndroidTool
2. Run the `mkupdate.bat` batch file in the `rockdev` directory of AndroidTool.
3. `update.img` will be created in `rockdev\Image` directory.

## Flash Image 

Reference: [《Flash Image》](03-upgrade_firmware.md)

[Getting Started]: started.md
[FAQ]: faqs.md
[Serial Debug]: debug.md
[Building Linux Root Filesystem]: linux_build_ubuntu_rootfs.md
[ADB introduction]: adb_use.md
[Contact]: resource.md#community
[RK Firmware]: 02-upgrade_table.md#rk-firmware-format
[Compile Linux Firmware]:linux_compile_gpt.md
[Compile Android Firmware]:compile_android8.1_firmware.md
[MaskRom]:04-maskrom_mode.md
[Flashing Notes]:02-upgrade_table.md
[Boot Mode]:01-bootmode.md
[ROC-RK3328-PC]: http://en.t-firefly.com/product/rocrk3328pc.html "ROC-RK3328-PC Official Website"
[Download Page]: https://community.t-firefly.com/en/doc/download/34
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