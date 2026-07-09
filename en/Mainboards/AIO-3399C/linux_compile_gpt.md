# Compile Linux firmware

In order to facilitate the use and development of users, the official Linux development kit SDK is provided. This chapter explains the specific use of the SDK in detail.

## Preparatory work

### Set up compilation environment

**Please confirm the installation of the following files!**

Here is use Ubuntu 18.04 for testing(It is recommended to use the ubuntu18.04 system for development, or use docker to deploy the Ubuntu18.04 container, otherwise the environment package cannot be installed):
```
sudo apt-get update

sudo apt-get install repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler \
gcc-aarch64-linux-gnu mtools parted libudev-dev libusb-1.0-0-dev python-linaro-image-tools \
linaro-image-tools gcc-arm-linux-gnueabihf libssl-dev liblz4-tool genext2fs lib32stdc++6 \
gcc-aarch64-linux-gnu g+conf autotools-dev libsigsegv2 m4 intltool libdrm-dev curl sed make \
binutils build-essential gcc g++ bash patch gzip bzip2 perl tar cpio python unzip rsync file bc wget \
libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev libglade2-dev cvs git mercurial rsync openssh-client \
subversion asciidoc w3m dblatex graphviz python-matplotlib libssl-dev texinfo fakeroot \
libparse-yapp-perl default-jre patchutils swig chrpath diffstat gawk
```

**Note:** Ubuntu 17.04 or higher systems also need the following dependency packages:

```
sudo apt-get install lib32gcc-7-dev g++-7 libstdc++-7-dev
```

### Download Firefly_Linux_SDK sub-volume compressed package

**Since the Firefly_Linux_SDK source code package is relatively large, some users' computers do not support files above 4G or the network transmission of a single file is slow, so we use the method of sub-volume compression to package the SDK. Users can obtain the Firefly_Linux_SDK source code package in the following ways:**[Firefly_Linux_SDK Source]

After downloading, follow the README document:

```
└── rk3399_linux_release_xxx
    ├── linux_sdk_tar
    │   ├── rk3399_linux_release_xxx.sdk.split00
    │   ├── rk3399_linux_release_xxx.sdk.split01
    ├── md5sum.txt
    ├── README_EN.txt
    ├── README_ZH.txt
    └── sdk_tools.sh

```

### Update Firefly_Linux_SDK

You can use the following command to update the SDK later
```
.repo/repo/repo sync -c --no-tags
```

[Firefly_Linux_SDK Source]: http://en.t-firefly.com/doc/download/52.html#other_144


### Linux_SDK catalog

catalog:

```
$ tree -L 1
.
├── app
├── buildroot                                               # Buildroot root file system build directory
├── build.sh -> device/rockchip/common/build.sh             # compile script
├── device                                                  # Compile related configuration files
├── docs                                                    # Documentation
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel
├── Makefile -> buildroot/build/Makefile
├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh   # link script
├── prebuilts                                               # Cross compilation tool chain
├── rkbin
├── rkflash.sh -> device/rockchip/common/rkflash.sh         # Flash script
├── tools                                                   # Tools directory
├── u-boot
└── yocto
```

## Compile SDK
<a id="configuration"></a>
### Configuration before compilation

The configuration file `aio-3399c-ubuntu.mk`:

```
./build.sh aio-3399c-ubuntu.mk

#The file path: `device/rockchip/rk3399/aio-3399c-ubuntu.mk`
```

Effective configuration file will be connected to the `device/rockchip/.BoardConfig.mk`, check the file to verify that the configuration was successful.

**Note :** `aio-3399c-ubuntu.mk` is configuration file after compiled ubuntu firmware. At the same time, users can also generate new configuration files by referring to this configuration to adapt the firmware they need.

Important configuration information :(if you need diy firmware, you may need to modify the following configuration information)

```bash
#Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly-rk3399    # Compile the uboot configuration file

#Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly_linux_defconfig   # Compile the kernel configuration file

#Kernel dts
export RK_KERNEL_DTS=rk3399-firefly-aioc                     # Compile the DTS used by kernel

#parameter for GPT table
export RK_PARAMETER=parameter-ubuntu.txt               # Partitioning information (very important)

#packagefile for make update image
export RK_PACKAGE_FILE=rk3399-ubuntu-package-file      # Packaging configuration file

#rootfs image path
export RK_ROOTFS_IMG=xxxx/xxxx.img                     # The root file system image path
```

**<font color=#ff0000 >Attention, the following steps are very important! !</font>**

*  [Download the Ubuntu root filesystem image]
*  Because RK3399 belongs to a 64-bit processor, so select the required file system under the corresponding `Linux Rootfs/arm64` folder. This article uses `rk3399_ubuntu18.04.img.7z` as an example.
Place the resulting image at the root of the SDK:
*  Put the resulting image in the specified directory in SDK:

```
#Unpack
7z x rk3399_ubuntu18.04.img.7z

#The root directory of sdk
mkdir ubuntu_rootfs
mv rk3399_ubuntu18.04.img ubuntu_rootfs/

#Modify aio-3399c-ubuntu.mk :
vim device/rockchip/rk3399/aio-3399c-ubuntu.mk

#Change the RK_ROOTFS_IMG property to the ubuntu file system image path(such as rk3399_ubuntu18.04.img)
RK_ROOTFS_IMG=ubuntu_rootfs/rk3399_ubuntu18.04.img
```

**<font color=#ff0000 >NOTE</font>**: Ubuntu root file system image storage path can not be wrong !!

Run `./mkfirmware.sh` will automatically update `rockdev/rootfs.img` links.

### Automatic compilation

Under the premise that the configuration and setting up of the environment are completed:

```
./build.sh
```

### Partial compilation

#### kernel

```
./build.sh kernel
```

Notice：**Linux SDK v2.5.1c** and later using extboot, please use `./build.sh extboot` to build kernel

The output file is `SDK/kernel/extboot.img`, use it instead of boot.img

How to check SDK version:
1. The version format is vx.x.xx, eg: v2.5.1c
2. Firmware filename has SDK version(..._vx.x.xx_date.img)
3. Buildroot use`cat /etc/version`to get version(rk3399_linux_release_date_vx.x.xx.xml)
4. Ubuntu use`ffgo version`to get(rk3399_linux_release_date_vx.x.xx.xml)
5. In SDK check the link:`ls -l .repo/manifests/rk3399_linux_release.xml`
6. If you can't get version by methods above, that means you are using old version, no support for extboot

**Do not flash extboot.img into old version firmware!**

Besides, extboot ubuntu support update kernel by deb package, please see [Ubuntu Manual](https://wiki.t-firefly.com/en/Firefly-Linux-Guide/manual_ubuntu.html#linux-headers-and-linux-image)

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

Run `./mkfirmware.sh` will automatically update `rockdev/rootfs.img` links.

## Pack the firmware

### Update the part images synchronously

Before each package firmware, ensure `rockdev/` directory file link is correct:

```
ls -l

├── boot.img -> ~/project/linux_sdk/kernel/boot.img
├── idbloader.img -> ~/project/linux_sdk/u-boot/idbloader.img
├── linaro-rootfs.img
├── MiniLoaderAll.bin -> ~/project/linux_sdk/u-boot/rk3399_loader_v1.14.115.bin
├── misc.img -> ~/project/linux_sdk/device/rockchip/rockimg/wipe_all-misc.img
├── oem.img
├── parameter.txt -> ~/project/linux_sdk/device/rockchip/RK3399/parameter-ubuntu.txt
├── recovery.img -> ~/project/linux_sdk/buildroot/output/rockchip_rk3399_recovery/images/recovery.img
├── rootfs.img -> ~/project/linux_sdk/ubuntu_rootfs/rk3399_ubuntu18.04.img
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
error: /home/ljh/proj/linux-sdk/buildroot/output/rockchip_rk3399_recovery/images/recovery.img not found!
#Represents that the recovery partition has not compiled an image, and other situations are similar, such as oem.img, userdata.img. As mentioned above, these are elliptical partition images and can be ignored.
```

### Packaged into a unified firmware

**Note:** Please make sure `tools/linux/Linux_Pack_Firmware/rockdev/package-file` is correct before packing. The packaging is partitioned based on this file. This file link is updated when the `./build.sh aio-rk3399c.mk` command is executed. If the configuration is not correct, go back to the `Configuration before compilation` section and configure it again.

Pack the firmware, the firmware will be saved to the directory `rockdev/pack/`.

#### RK firmware

RK firmware is the firmware packaged in Rockchip's proprietary format, and can be flashed to eMMC or SD card with the tools provided by Rockchip (**Note**: If there is no special instruction, the firmware mentioned on WIKI defaults to RK firmware) .

```bash
# Pack RK firmware
./build.sh updateimg
```

#### RAW firmware

RAW firmware is a kind of firmware that can be flashed to the storage device in a bit-by-bit copy mode, and is the original image of the storage device. Different from the RK firmware, currently it only supports flashing to SD card to boot with Etcher tool.

[Etcher official download link](https://www.balena.io/etcher/)

```bash
# Pack RAW firmware
./build.sh rawimg
```

## Partition introduction

### parameter

`parameter.txt` contains firmware partition information is very important. You can find some `parameter.txt` files in `device/rockchip/RK3399` directory. The following is introduced with `parameter.txt` as an example:

```
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3399
MACHINE_ID: 007
MANUFACTURER: RK3399
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(trust),0x00002000@0x00008000(misc),0x00040000@0x0000a000(boot:bootable),0x00040000@0x0004a000(recovery),0x00010000@0x0008a000(backup),0x00c00000@0x0009a000(rootfs),0x00040000@0x00c9a000(oem),-@0x00d1a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

`CMDLINE` property is what we care about. Take uboot as an example, `0x00004000` in `0x00002000@0x00004000(uboot)` is the starting position of the uboot partition, and `0x00002000` is the size of the partition. The following partition rules are the same. Users can add or subtract or modify partition information according to their needs, but please keep at least the `uboot`, `trust`, `boot`, `rootfs` partition, which is a prerequisite for the machine to start normally. The simplest partition scheme is used in `parameter-ubuntu.txt`.

Partition introduction:

**uboot:**     Upgrade the *uboot.img* compiled by uboot.

**trust:**     Upgrade the *trust.img* compiled by uboot.

**misc:**      Upgrade the *misc.img*. Turn on and Enter *recovery* mode.(omitted)

**boot:**      Upgrade the *boot.img* compiled by kernel. Contains kernel and device tree information.

**recovery:**  Upgrade the *recovery.img*.(omitted)

**backup:**    Reserved. Not for the time being. In the future, it will be used as *backup* of *recovery* just like Android.(omitted)

**oem:**       For manufacturers to use, used to store the manufacturer's app or data. Read-only. Replace the data partition of the original speaker. Mount in `/oem`.(omitted)

**rootfs:**    Store the rootfs.img compiled by buildroot,  Read-only.

**userdata:**  Save the files generated by the app or is for the end user. Read and Write, Mount in `/userdata`.(omitted)

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

[18.04 FS]: http://en.t-firefly.com/doc/download/52.html#other_117
[Download the Ubuntu root filesystem image]: http://en.t-firefly.com/doc/download/52.html#other_117