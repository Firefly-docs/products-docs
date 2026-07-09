# Compile Buildroot firmware

In order to facilitate the use and development of users, the official Linux development kit SDK is provided. This chapter explains the specific use of the SDK in detail.

## Preparatory work

### Download Firefly_Linux_SDK

#### Method 1
```
repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk1808_linux_release.xml

repo sync -c
# It may be necessary to execute it repeatedly until the code is successfully downloaded
```

#### Method 2

* Download the `REPO_SDK` package from the [download page](http://en.t-firefly.com/doc/download/73.html)</br>
* Compare the MD5 code of the REPO_SDK package to verify the integrity, and then decompress it.</br>
```
md5sum rk1808_linux_release_20210306.tgz
be79c2e877970e4e0fc2bc0be44a8d02  rk1808_linux_release_20210306.tgz
tar xvf rk1808_linux_release_20210306.tgz
```
* After decompression, enter the folder to complete the synchronization work.</br>
```
# This compressed package contains a ".repo" directory, after decompression, perform the following operations in the current directory
.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all

# You can use the following command to update the SDK later
.repo/repo/repo sync -c --no-tags
# It may be necessary to execute it repeatedly until the code is successfully downloaded
```

### Linux_SDK catalog

catalog:
```
firefly-sdk/
├── app
├── buildroot                                              The compile directory for the buildroot
├── build.sh -> device/rockchip/common/build.sh            Fully automated compiled scripts
├── debian
├── device                                                 Compile-related configuration files
├── distro
├── docs                                                   Document
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel                                                 Kernel
├── Makefile -> buildroot/build/Makefile
├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh  Rockdev links to the update script
├── prebuilts
├── rkbin
├── rkflash.sh -> device/rockchip/common/rkflash.sh        The upgrade script
├── tools                                                  Upgrade and package tools
├── u-boot                                                 Uboot
└── yocto
```

### Set up the compilation environment of SDK

* Compile buildroot :

```
sudo apt-get install expect-dev repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler \
gcc-aarch64-linux-gnu mtools parted libudev-dev libusb-1.0-0-dev python-linaro-image-tools \
linaro-image-tools autoconf autotools-dev libsigsegv2 m4 intltool libdrm-dev curl sed make \
binutils build-essential gcc g++ bash patch gzip bzip2 perl tar cpio python unzip rsync file bc wget \
libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev libglade2-dev cvs git mercurial rsync openssh-client \
subversion asciidoc w3m dblatex graphviz python-matplotlib libc6:i386 libssl-dev texinfo \
liblz4-tool genext2fs lib32stdc++6
```

**WARNNING**:Ubuntu 17.04 or higher systems also need the following dependency packages:
```
sudo apt-get install lib32gcc-7-dev g++-7 libstdc++-7-dev
```
## Compile SDK

### Configuration before compilation

The configuration file : *aio-rk1808-jd4-buildroot.mk*
```
./build.sh aio-rk1808-jd4-buildroot.mk
# The file path:`device/rockchip/rk1808/aio-rk1808-jd4-buildroot.mk`
```
Effective configuration file will be connected to the `device/rockchip/.BoardConfig.mk`, check the file to verify that the configuration was successful.

**Note:** `aio-rk1808-jd4-buildroot.mk` is configuration file after compiled ubuntu firmware. At the same time, users can also generate new configuration files by referring to this configuration to adapt the firmware they need.

Important configuration information :

```
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly_rk1808                            Compile the uboot configuration file

# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly_rk1808_defconfig                 Compile the kernel configuration file

# Kernel dts
export RK_KERNEL_DTS=rk1808-firefly-aiojd4                          Compile the DTS used by kernel

# parameter for GPT table
export RK_PARAMETER=parameter-buildroot.txt                         Partitioning information (very important)

# packagefile for make update image 
export RK_PACKAGE_FILE=rk1808-package-file                          Packaging configuration file

# rootfs image path
export RK_ROOTFS_IMG=rockdev/rootfs.${RK_ROOTFS_TYPE}               The root file system image path
```

### Automatic compilation

Under the premise that the configuration and setting up of the environment are completed :

```
./build.sh aio-rk1808-jd4-buildroot.mk
./build.sh
```
Generate the firmware directory `rockdev/pack`, at the same time will backup in the IMAGE.

### Partial compilation

#### Configuration
```
./build.sh aio-rk1808-jd4-buildroot.mk
```
#### kernel
```
./build.sh kernel
```
#### u-boot
```
./build.sh uboot
```
#### recovery
```
./build.sh recovery
```
#### rootfs
```
./build.sh rootfs
```

### Update the part images synchronously

Before each package firmware, ensure `rockdev/` directory file link is correct:

```
rockdev/
├── boot.img -> ../kernel/boot.img
├── MiniLoaderAll.bin -> ../u-boot/rk1808_loader_v1.04.105.bin
├── misc.img -> ../device/rockchip/rockimg/wipe_all-misc.img
├── oem.img
├── parameter.txt -> ../device/rockchip/rk1808/parameter-buildroot.txt
├── recovery.img -> ../buildroot/output/rockchip_rk1808_recovery/images/recovery.img
├── rootfs.ext4 -> ../buildroot/output/rockchip_rk1808/images/rootfs.ext2
├── rootfs.img -> ../buildroot/output/rockchip_rk1808/images/rootfs.ext2
├── trust.img -> ../u-boot/trust.img
├── uboot.img -> ../u-boot/uboot.img
└── userdata.img

```
Runing `./mkfirmware.sh` to update links

```
./mkfirmware.sh
```

### Packaged into a unified firmware

**Note:** please make sure `tools/linux/Linux_Pack_Firmware/rockdev/package-file` is correct before packing. The packaging is partitioned based on this file. This file link is updated when the `./build.sh aio-rk1808-jd4-buildroot.mk` command is executed. If the configuration is not correct, go back to the Configuration section and configure it again.

To integrate unified firmware, you need to run `./mkfirmware.sh` to update the link each time before packaging.
```
./build.sh updateimg
```
After entering this command, it will remind you whether you need to rename, enter `y` to rename the firmware, enter `n` to use the default firmware name, and generate the firmware in the `rockdev/pack` directory.

## Partition introduction

### parameter

`parameter.txt` contains firmware partition information is very important. You can find some `parameter.txt` files in `device/rockchip/rk1808` directory. The following is introduced with `parameter-buildroot.txt` as an example:

```
FIRMWARE_VER: 8.1
MACHINE_MODEL: RK1808
MACHINE_ID: 007
MANUFACTURER: RK1808
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 1808
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(trust),0x00002000@0x00008000(misc),0x00010000@0x0000a000(boot),0x00010000@0x0001a000(recovery),0x00010000@0x0002a000(backup),0x00020000@0x0003a000(oem),0x00300000@0x0005a000(rootfs),-@0x0035a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

`CMDLINE` property is what we care about. Take uboot as an example,  `0x00004000` in `0x00002000@0x00004000(uboot)` is the starting position of the uboot partition, and `0x00002000` is the size of the partition. The following partition rules are the same. Users can add or subtract or modify partition information according to their needs, but please keep at least the `uboot`, `trust`, `boot`, `rootfs` partition, which is a prerequisite for the machine to start normally. 

Partition introduction:
```
uboot: Upgrade the uboot.img compiled by uboot.
trust: Upgrade the trust.img compiled by uboot.
misc: Upgrade the misc.img. Turn on and Enter recovery mode.
boot: Upgrade the boot.img compiled by kernel. Contains kernel and device tree information
recovery:  Upgrade the recovery.img.
backup: Reserved. Not for the time being. In the future, it will be used as backup of recovery just like Android.
rootfs: Store the rootfs.img compiled by buildroot or ubuntu,  Read-only.
```

### package-file

This file should be consistent with the parameter and used for firmware packaging. Relevant documents can be found under `tools/linux/Linux_Pack_Firmware/rockdev`. 
```
# NAME      Relative path
#
#HWDEF      HWDEF
package-file    package-file
bootloader  Image/MiniLoaderAll.bin
parameter   Image/parameter.txt
trust       Image/trust.img
uboot       Image/uboot.img
misc        Image/misc.img
boot        Image/boot.img
recovery    Image/recovery.img
rootfs      Image/rootfs.img
userdata:grow RESERVED
backup      RESERVED
```
The above is the mirror file generated after SDK compilation. Package only the img files you use according to `parameter.txt`.

## FAQs

### How to enter upgrade mode ?

See operation method in ["Upgrade firmware"](upgrade_firmware.html)