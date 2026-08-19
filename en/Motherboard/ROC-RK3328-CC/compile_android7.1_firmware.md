# Compiling Android 7.1

## Preparation

### Hardware Requiremnts

Recommended hardware requirement of development workstation compiling Android 7.1:

- 64 bit CPU
- 16GB  Physical memory + Swap memory
- 30GB  Free disk space is used for building, and the source tree takes about 8GB

See also the hardware and software configuration stated in Google official document:

- [https://source.android.com/setup/build/requirements](https://source.android.com/setup/build/requirements)
- [https://source.android.com/setup/initializing](https://source.android.com/setup/initializing)

### Software Requiements

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

Due to the huge size of the Android SDK, please download SDK`firefly_rk3328_android7.1_git_20211216`:

* [Download](https://community.t-firefly.com/en/doc/download/34)

After the download completes, verify the MD5 checksum before extraction:

``` shell
$md5sum ~/firefly_rk3328_android7.1_git_20211216.7z.001
$md5sum ~/firefly_rk3328_android7.1_git_20211216.7z.002
$md5sum ~/firefly_rk3328_android7.1_git_20211216.7z.003

f4d62bfbb401f596328435e61cb1c68d  firefly_rk3328_android7.1_git_20211216.7z.001
0bb35911c8740119f789926d6d21c550  firefly_rk3328_android7.1_git_20211216.7z.002
19eebf52e1e033cea682cdd306d8f75a  firefly_rk3328_android7.1_git_20211216.7z.003
```

Then extract it:

``` shell
mkdir -p  ~/proj/firefly_rk3328_android7.1
cd ~/proj/firefly_rk3328_android7.1
7z x ~/firefly_rk3328_android7.1_git_20211216.7z.001 -r -o.
git reset --hard
```

Synchronize source code from gitlab:

``` shell
git pull gitlab roc-rk3328-cc:roc-rk3328-cc

git checkout roc-rk3328-cc
```

## Compiling with Firefly Scripts

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

**Compiling Everying**

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
make ARCH=arm64 firefly_defconfig
make -j8 ARCH=arm64 rk3328-roc-cc.img
```

**Compiling U-Boot**

``` shell
make rk3328_box_defconfig
make ARCHV=aarch64 -j8
```

**Compiling Android**

``` shell
source build/envsetup.sh
lunch roc_rk3328_cc_box-userdebug
make installclean
make -j8
./mkimage.sh
```

## Packing Rockchp Firmware

**Packing Fimware in Linux**

After compiling you can use Firefly official script to pack all partition image files into the one true Rockchip firmware or the one true original firmware, by executing the following command:

``` shell
rk firmware:
./FFTools/mkupdate/mkupdate.sh update

original firmware:
./FFTools/mkupdate/sd_mkupdate.sh update
```

The resulting file is `rockdev/Image-rk3328_firefly_box/update.img`.

The RK firmware needs to use the `SD_Firmware_Tool` tool and the function mode to select `SD Startup` to create the boot card,
and original firmware can use the `SDCard-installer` to make the boot card.

**Packing Fimware in Windows**

It is also very simple in packaging Rockchip firmware `update.img` under Windows:

1. Copy all the compiled files in `rockdev/Image-rk3328_firefly_box/` to the `rockdev\Image` directory of AndroidTool
2. Run the `mkupdate.bat` batch file in the `rockdev` directory of AndroidTool.
3. `update.img` will be created in `rockdev\Image` directory.

## Partition Images

`update.img` is the firmware released to end users, which is convenient to upgrade the system of the deveopment board.

During development cycle, it is a great time saving to only flash modified partition images.

Here's a table summarising the partition image in various stage:

```text
|------------------|---------------------|-----------|
| Stage            | Product             | Partition |
|------------------|---------------------|-----------|
| Compiling Kernel | kernel/kernel.img   | kernel    |
|                  | kernel/resource.img | resource  |
|------------------|---------------------|-----------|
| Compiling U-Boot | u-boot/uboot.img    | uboot     |
|------------------|---------------------|-----------|
| ./mkimage.sh     | boot.img            | boot      |
|                  | system.img          | system    |
|------------------|---------------------|-----------|
```

Note that by excuting `./mkimage.sh`, `boot.img` and `system.img` will be repacked wth the compiled results of Android in `out/target/product/rk3328_firefly_box/` directory, and all related image files will be copied to the directory `rockdev/Image-rk3328_firefly_box/`.

The following is a list of the image files:

- `boot.img`: Android initramfs image, contains base filesystem of Android root directory, whcih is responsible for initializing and loading the system partition.
- `system.img`: Android system partition image in ext4 filesystem format.
- `kernel.img`: kernel image.
- `resource.img`: Resource image, containing boot logo and kernel device tree blob.
- `misc.img`: misc partition image, responsible for starting the mode switch and first aid mode parameter transfer.
- `recovery.img`: Recovery mode image.
- `rk3328_loader_v1.08.244.bin`: Loader files.
- `uboot.img`: U-Boot image file.
- `trust.img`: Arm trusted file (ATF) image file.
- `parameter.txt`: Partition layout and kernel command line.

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