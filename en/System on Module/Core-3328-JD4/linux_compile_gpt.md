# Compile Ubuntu firmware(GPT)

In order to facilitate the use and development of users, the official Linux development kit SDK is provided. This chapter explains the specific use of the SDK in detail.

## Preparatory work

### Set up compilation environment

**Please confirm the installation of the following files!**

Here is use Ubuntu 18.04 for testing(It is recommended to use Ubuntu16.04 adnd above):
```
sudo apt-get update

sudo apt-get install repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler \
gcc-aarch64-linux-gnu mtools parted libudev-dev libusb-1.0-0-dev python-linaro-image-tools \
linaro-image-tools gcc-arm-linux-gnueabihf libssl-dev liblz4-tool genext2fs lib32stdc++6 \
gcc-aarch64-linux-gnu g+conf autotools-dev libsigsegv2 m4 intltool libdrm-dev curl sed make \
binutils build-essential gcc g++ bash patch gzip bzip2 perl tar cpio python unzip rsync file bc wget \
libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev libglade2-dev cvs git mercurial rsync openssh-client \
subversion asciidoc w3m dblatex graphviz python-matplotlib libc6:i386 libssl-dev texinfo fakeroot \
libparse-yapp-perl default-jre patchutils
```

**Note:** Ubuntu 17.04 or higher systems also need the following dependency packages:

```
sudo apt-get install lib32gcc-7-dev g++-7 libstdc++-7-dev
```

### Download Firefly_Linux_SDK sub-volume compressed package

**Since the Firefly_Linux_SDK source code package is relatively large, some users' computers do not support files above 4G or the network transmission of a single file is slow, so we use the method of sub-volume compression to package the SDK. Users can obtain the Firefly_Linux_SDK source code package in the following ways:**[Firefly_Linux_SDK Source]

After the download is complete, verify the MD5 code:

```
$ md5sum rk3328_linux_release_v2.5.1_20210301_firefly_split.file0*
0069f84a604b5f774f08206214f4d0fb  rk3328_linux_release_v2.5.1_20210301_firefly_split.file00
860ee04da6191e0fa0f20d285dfb6097  rk3328_linux_release_v2.5.1_20210301_firefly_split.file01
aa606808eeaf8e044f58af94b61acd7c  rk3328_linux_release_v2.5.1_20210301_firefly_split.file02
601e1c9ef2b17c7ba45cec5dfbf99928  rk3328_linux_release_v2.5.1_20210301_firefly_split.file03
0d362a5b6f8ba910f0f7d918724fbec1  rk3328_linux_release_v2.5.1_20210301_firefly_split.file04
b71e825a0bb3dd9b865beff03cff6d52  rk3328_linux_release_v2.5.1_20210301_firefly_split.file05
0df97b3c7c71c3bde49f55d83496d78a  rk3328_linux_release_v2.5.1_20210301_firefly_split.file06
278cef9b70531b5f7e5e8a79b8a7bfc4  rk3328_linux_release_v2.5.1_20210301_firefly_split.file07
```

### Unpack Firefly_Linux_SDK sub-volume compressed package

After confirming that it is correct, you can unpack:
```
cat rk3328_linux_release_v2.5.1_20210301_split_dir/*firefly_split* | tar -xzv

#This SDK folder contains a .repo directory, after decompression, perform the following operations in the current directory
cd rk3328_linux_release_v2.5.1_20210301
ls -al

.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

### Update Firefly_Linux_SDK

You can use the following command to update the SDK later
```
.repo/repo/repo sync -c --no-tags
```

[Firefly_Linux_SDK Source]: http://en.t-firefly.com/doc/download/34.html#other_414



### Linux_SDK catalog

catalog:

```
├── linux_sdk
│   ├── app
│   ├── buildroot                                               compile buildroot rootfs directory
│   ├── build.sh -> device/rockchip/common/build.sh             automatic compile script
│   ├── device                                                  config file
│   ├── distro                                                  debian_root.img directory
│   ├── docs                                                    document
│   ├── envsetup.sh -> buildroot/build/envsetup.sh
│   ├── external
│   ├── kernel                                                  kernel directory
│   ├── Makefile -> buildroot/build/Makefile
│   ├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh   update rockdev script
│   ├── prebuilts
│   ├── rkbin
│   ├── rkflash.sh -> device/rockchip/common/rkflash.sh         flashing firmware script
│   ├── rootfs                                                  compile debian rootfs directory
│   ├── tools                                                   flashing and package tools
│   └── u-boot                                                  uboot directory
```

## Compile SDK
<a id="configuration"></a>
### Configuration before compilation

The configuration file `firefly-rk3328-ubuntu.mk`:

```
./build.sh firefly-rk3328-ubuntu.mk


#The file path: `device/rockchip/rk3328/firefly-rk3328-ubuntu.mk`

```

Effective configuration file will be connected to the `device/rockchip/.BoardConfig.mk`, check the file to verify that the configuration was successful.

**Note :** `firefly-rk3328-ubuntu.mk` is configuration file after compiled buildroot firmware. At the same time, users can also generate new configuration files by referring to this configuration to adapt the firmware they need.

Important configuration information :(if you need diy firmware, you may need to modify the following configuration information)

```
#Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly-rk3328              #Compile the uboot configuration file

#Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly_linux_defconfig            #Compile the kernel configuration file

#Kernel dts
export RK_KERNEL_DTS=rk3328-firefly                                  #Compile the DTS used by kernel

#parameter for GPT table
export RK_PARAMETER=parameter-ubuntu.txt                      #Partitioning information (very important)

#packagefile for make update image
export RK_PACKAGE_FILE=rk3328-ubuntu-package-file             #Packaging configuration file

#rootfs image path
export RK_ROOTFS_IMG=xxxx/xxxx.img                            #The root file system image path
```

**<font color=#ff0000 >Attention, very important ! !</font>**

Buildroot firmware is compiled by default. If you want to compile other firmware (such as ubuntu firmware), please do the following:

*  [Download the Ubuntu root filesystem image]

*  Put the resulting image in the specified directory in SDK:

```
#Unpack
tar -xvf rk3328_ubuntu18.04_LXDE.img.tgz

#The root directory of sdk
mkdir ubunturootfs
mv rk3328_ubuntu18.04_LXDE.img ubunturootfs/

#Modify firefly-rk3328-ubuntu.mk :
vim device/rockchip/RK3328/firefly-rk3328-ubuntu.mk

#Change the RK_ROOTFS_IMG property to the ubuntu file system image path(such as rk3328_ubuntu18.04_LXDE.img)
RK_ROOTFS_IMG=ubunturootfs/rk3328_ubuntu18.04_LXDE.img
```

**<font color=#ff0000 >NOTE</font>**: Ubuntu root file system image storage path can not be wrong !!

### Automatic compilation

Under the premise that the configuration and setting up of the environment are completed:

```
./build.sh
```

Automatic compilation of firmware default compiler `buildroot` firmware. Generate the firmware directory `rockdev/`, at the same time will backup in the IMAGE.

### Partial compilation

#### kernel

```
./build.sh kernel
```

#### u-boot

```
./build.sh uboot
```

#### recovery

recovery partition can be omitted, if necessary, compile it:

```
./build.sh recovery
```

#### rootfs

* buildroot

```
./build.sh rootfs
```

* debian:

```
cd rootfs/

1:
#Building base debian system by ubuntu-build-service from linaro
sudo apt-get install binfmt-support qemu-user-static live-build
sudo dpkg -i ubuntu-build-service/packages/*
sudo apt-get install -f

2:
#Compile 32-bit debian:
RELEASE=stretch TARGET=desktop ARCH=armhf ./mk-base-debian.sh
#Or compile 64-bit debian:
RELEASE=stretch TARGET=desktop ARCH=arm64 ./mk-base-debian.sh

#If the above compilation encounters the following problems:
noexec or nodev issue /usr/share/debootstrap/functions: line 1450: ..../rootfs/ubuntu-build-
service/stretch-desktop-armhf/chroot/test-dev-null: Permission denied E: Cannot install into target
'/home/foxluo/work3/rockchip/rk_linux/rk3328_linux/rootfs/ubuntu-build-service/stretch-
desktop-armhf/chroot' mounted with noexec or nodev

# Solutions:
mount -o remount,exec,dev xxx (xxx is the mount place), then rebuild it.

3:
#Compile 32-bit stretch:
VERSION=debug ARCH=armhf ./mk-rootfs-stretch.sh

#It is recommend to use that rear band debug in the development phase
#Compile 64-bit stretch:
VERSION=debug ARCH=arm64 ./mk-rootfs-stretch-arm64.sh

4:
./mk-image.sh
mv linaro-rootfs.img ../distro/

5:
Modify firefly-rk3328-ubuntu.mk
vim device/rockchip/RK3328/firefly-rk3328-ubuntu.mk

#Change the RK_ROOTFS_IMG property to the ubuntu file system image path(such as linaro-rootfs.img)
RK_ROOTFS_IMG=distro/linaro-rootfs.img
```

* ubuntu18.04 can be downloaded via cloud disk:[18.04 FS]




Place the resulting image at the root of the SDK

```
#Unpack
tar -xvf rk3328_ubuntu18.04_LXDE.img.tgz

#The root directory of sdk
mkdir ubunturootfs
mv rk3328_ubuntu18.04_LXDE.img ubunturootfs/

#Modify firefly-rk3328-ubuntu.mk :
vim device/rockchip/RK3328/firefly-rk3328-ubuntu.mk

#Change the RK_ROOTFS_IMG property to the ubuntu file system image path(such asrk3328_ubuntu18.04_LXDE.img)
RK_ROOTFS_IMG=ubunturootfs/rk3328_ubuntu18.04_LXDE.img
```

**NOTE**:buntu root file system image storage path can not be wrong!!

Run `./mkfirmware.sh` will automatically update `rockdev/rootfs.img` links.

## Pack the firmware

### Update the part images synchronously

Before each package firmware, ensure `rockdev/` directory file link is correct:

```
ls -l

├── boot.img -> ~/project/linux_sdk/kernel/boot.img
├── idbloader.img -> ~/project/linux_sdk/u-boot/idbloader.img
├── linaro-rootfs.img
├── MiniLoaderAll.bin -> ~/project/linux_sdk/u-boot/rk3328_loader_v1.14.115.bin
├── misc.img -> ~/project/linux_sdk/device/rockchip/rockimg/wipe_all-misc.img
├── oem.img
├── parameter.txt -> ~/project/linux_sdk/device/rockchip/RK3328/parameter-ubuntu.txt
├── recovery.img -> ~/project/linux_sdk/buildroot/output/rockchip_rk3328_recovery/images/recovery.img
├── rootfs.img -> ~/project/linux_sdk/ubunturootfs/rk3328_ubuntu18.04_LXDE.img
├── trust.img -> ~/project/linux_sdk/u-boot/trust.img
├── uboot.img -> ~/project/linux_sdk/u-boot:/uboot.img
└── userdata.img
```

Runing `./mkfirmware.sh` to update links

```
./mkfirmware.sh
```

**Tip:** If you do not compile all the partition images, you will encounter the following situation when running `./mkfirmware`:

```
error: /home/ljh/proj/linux-sdk/buildroot/output/rockchip_rk3328_recovery/images/recovery.img not found!
#Represents that the recovery partition has not compiled an image, and other situations are similar, such as oem.img, userdata.img. As mentioned above, these are elliptical partition images and can be ignored.
```

### Packaged into a unified firmware

**Note:** Please make sure `tools/linux/Linux_Pack_Firmware/rockdev/package-file` is correct before packing. The packaging is partitioned based on this file. This file link is updated when the `./build.sh aio-rk3328c.mk` command is executed. If the configuration is not correct, go back to the `Configuration before compilation` section and configure it again.

Integrated into unified firmware:

```
./build.sh updateimg
```

## Partition introduction

### parameter

`parameter.txt` contains firmware partition information is very important. You can find some `parameter.txt` files in `device/rockchip/RK3328` directory. The following is introduced with `parameter-debian.txt` as an example:

```
FIRMWARE_VER: 8.1
MACHINE_MODEL: RK3328
MACHINE_ID: 007
MANUFACTURER: RK3328
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3328
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(trust),0x00002000@0x00008000(misc),0x00010000@0x0000a000(boot),0x00010000@0x0001a000(recovery),0x00010000@0x0002a000(backup),0x00020000@0x0003a000(oem),0x00700000@0x0005a000(rootfs),-@0x0075a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

`CMDLINE` property is what we care about. Take uboot as an example, `0x00004000` in `0x00002000@0x00004000(uboot)` is the starting position of the uboot partition, and `0x00002000` is the size of the partition. The following partition rules are the same. Users can add or subtract or modify partition information according to their needs, but please keep at least the `uboot`, `trust`, `boot`, `rootfs` partition, which is a prerequisite for the machine to start normally. The simplest partition scheme is used in `parameter-ubuntu.txt`.

Partition introduction:

**uboot:** Upgrade the *uboot.img* compiled by uboot.

**trust:** Upgrade the *trust.img* compiled by uboot.

**misc:** Upgrade the *misc.img*. Turn on and Enter *recovery* mode.(omitted)

**boot:** Upgrade the *boot.img* compiled by kernel. Contains kernel and device tree information.

**recovery:**  Upgrade the *recovery.img*.(omitted)

**backup:** Reserved. Not for the time being. In the future, it will be used as *backup* of *recovery* just like Android.(omitted)

**oem:** For manufacturers to use, used to store the manufacturer's app or data. Read-only. Replace the data partition of the original speaker. Mount in `/oem`.(omitted)

**rootfs:** Store the rootfs.img compiled by buildroot or debian,  Read-only.

**userdata:** Save the files generated by the app or is for the end user. Read and Write, Mount in `/userdata`.(omitted)

### package-file

This file should be consistent with the `parameter` and used for firmware packaging. Relevant documents can be found under `tools/linux/Linux_Pack_Firmware/rockdev`.

```
# NAME          Relative path
#
# HWDEF         HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
trust           Image/trust.img
uboot           Image/uboot.img
boot            Image/boot.img
rootfs:grow     Image/rootfs.img
backup          RESERVED
```

The above is the mirror file generated after SDK compilation. Package only the img files you use according to `parameter.txt`.

## FAQs

### How to enter upgrade mode ?

See operation method in [Upgrade firmware](03-upgrade_firmware.md)

[18.04 FS]: http://en.t-firefly.com/doc/download/34.html#other_117
[Download the Ubuntu root filesystem image]: http://en.t-firefly.com/doc/download/34.html#other_117