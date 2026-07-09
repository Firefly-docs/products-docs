# Compile Linux Firmware (kernel-5.10)

## Download Firefly_Linux_SDK

First prepare an empty folder to place SDK, better under home, here we use `~/proj` as example.

<font color=red>

**Attention:**

**1. SDK uses cross-compile, so download SDK to your X86_64 PC, do not download SDK to arm64 board.**

**2. Build environment needs to be Ubuntu18.04 (real PC or docker container), other versions may cause build failure.**

**3. Do not put SDK under shared directory of VM or non-English path.**

**4. Please get/build SDK as a normal user, root privilege are neither allowed nor required (except installing sth. with apt)**
</font>

### Install Tools
```
sudo apt update
sudo apt install -y repo git python
```

### Init Code Repository

* Method One

Download via repo, you can choose to get full SDK or BSP:

```bash
mkdir ~/proj/rk3588_sdk/
cd ~/proj/rk3588_sdk/

## Full SDK
repo init --no-clone-bundle --repo-url https://github.com/Firefly-rk-linux-utils/git-repo.git --no-repo-verify -u https://github.com/Firefly-rk-linux/manifests.git -b master -m rk3588_linux_release.xml

## BSP (Only include some basic repositories and compile tools(Only for ubuntu & debian))
## BSP includes device/rockchip, docs, kernel, u-boot, rkbin, tools and cross-compile toolchian
repo init --no-clone-bundle --repo-url https://github.com/Firefly-rk-linux-utils/git-repo.git --no-repo-verify -u https://github.com/Firefly-rk-linux/manifests.git -b master -m rk3588_linux_bsp_release.xml
```

* Method Two

Download Firefly_Linux_SDK sub-volume compressed package: [Linux SDK](https://en.t-firefly.com/doc/download/337.html), follow the README document:

```bash
└── rk3588_linux_release_xxx
    ├── linux_sdk_tar
    │   ├── rk3588_linux_release_xxx.sdk.split00
    │   ├── rk3588_linux_release_xxx.sdk.split01
    │   ├── rk3588_linux_release_xxx.sdk.split02
    │   ├── rk3588_linux_release_xxx.sdk.split03
    ├── md5sum.txt
    ├── README_EN.txt
    ├── README_ZH.txt
    └── sdk_tools.sh
```

### Sync Code

Execute the following command to synchronize the code:

```bash
# Enter the SDK root directory
cd ~/proj/rk3588_sdk

# Sync
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

You can use the following command to update the SDK later:

```bash
.repo/repo/repo sync -c --no-tags
```
## Linux SDK Configuration introduction

### Directory

```bash
$ tree -L 1
.
├── app
├── buildroot # Buildroot root filesystem build directory
├── build.sh -> device/rockchip/common/build.sh # Compile script
├── device # Compile related configuration files
├── docs # Documentation
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel
├── Makefile -> buildroot/build/Makefile
├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh # Link script
├── prebuilts # Cross compilation toolchain
├── rkbin
├── rkflash.sh -> device/rockchip/common/rkflash.sh # Flash script
├── tools # Tools directory
├── u-boot
```

### Introduction to configuration files

In the `device/rockchip/rk3588/` directory, there are configuration files (xxxx.mk) for different board types, which are used to manage the compilation configuration of each project of the SDK. The relevant configuration introduction:

```bash
# Target arch
export RK_ARCH=arm64
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=xxxx_defconfig
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=xxxx_defconfig
# Kernel defconfig fragment
export RK_KERNEL_DEFCONFIG_FRAGMENT=xxxx.config
# Kernel dts
export RK_KERNEL_DTS=rk3588-firefly-aio-3588sjd4-ai.dts
# parameter for GPT table
export RK_PARAMETER=parameter-xxxx.txt
# rootfs image path
export RK_ROOTFS_IMG=ubuntu_rootfs/rootfs.img
```

### Partition table

#### parameter

The parameter.txt file contains the partition information of the firmware. Take parameter-ubuntu-fit.txt as an example:

path: `device/rockchip/rk3588/parameter-ubuntu-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3588
MACHINE_ID: 007
MANUFACTURER: RK3588
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00020000@0x00008000(boot:bootable),0x00040000@0x00028000(recovery),0x00010000@0x00068000(backup),0x00c00000@0x00078000(rootfs),0x00040000@0x00c78000(oem),-@0x00cb8000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is where we are concerned. Take uboot as an example. In 0x00002000@0x00004000(uboot), 0x00004000 is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

#### package-file

The package-file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt.

path: `tools/linux/Linux_Pack_Firmware/rockdev/rk3588-package-file`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
uboot           Image/uboot.img
misc            Image/misc.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
userdata        RESERVED
backup          RESERVED
```
## Compile Ubuntu firmware

This chapter introduces the compilation process of Ubuntu firmware. It is recommended to develop under Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

<font color=red>**The compilation portion of this tutorial works with SDK versions above v1.0.6e**</font>

```
$ readlink -f .repo/manifest.xml
/home/daijh/p/rk3588/.repo/manifests/rk3588/rk3588_linux_release_20230301_v1.0.6e.xml
```
### Preparatory work

#### Set up compilation environment

```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip \
device-tree-compiler ncurses-dev \
```

### Compile SDK

#### Precompile Configuration

There are configuration files for different board in `device/rockchip/rk3588/`, select the configuration file:

```bash
./build.sh firefly_rk3588_aio-3588sjd4-ai_ubuntu_defconfig
```

#### Build

##### Automatic compilation

* Download: [Ubuntu rootfs(64-bit)]()，put in SDK path

```bash
7z x ubuntu-aarch64-rootfs.7z
mkdir ubuntu_rootfs
mv ubuntu-aarch64-rootfs.img ubuntu_rootfs/rootfs.img
```

* start compiling
```bash
./build.sh
```
the firmware will be saved to the directory `rockdev/pack/`.

##### Partial compilation

* u-boot

```bash
./build.sh uboot
```

* kernel

```bash
./build.sh extboot
```

* recovery

```bash
./build.sh recovery
```

* Download: [Ubuntu rootfs(64-bit)]()，put in SDK path

```bash
7z x ubuntu-aarch64-rootfs.7z
mkdir ubuntu_rootfs
mv ubuntu-aarch64-rootfs.img ubuntu_rootfs/rootfs.img
```

* Update each part of the `.img` link to the directory `rockdev/`:

```bash
./mkfirmware.sh
```

* Pack the firmware, the firmware will be saved to the directory `rockdev/pack/`.

```bash
./build.sh updateimg
```

## Compile Debian firmware

This chapter introduces the compilation process of Debian firmware. It is recommended to develop under Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

<font color=red>**The compilation portion of this tutorial works with SDK versions above v1.0.6e**</font>

```
$ readlink -f .repo/manifest.xml
/home/daijh/p/rk3588/.repo/manifests/rk3588/rk3588_linux_release_20230301_v1.0.6e.xml
```
### Preparatory work

#### Set up compilation environment

```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip \
device-tree-compiler ncurses-dev \

### Compile SDK

#### Precompile Configuration

There are configuration files for different board in `device/rockchip/rk3588/`, select the configuration file:

```bash
./build.sh firefly_rk3588_aio-3588sjd4-ai_debian_defconfig
```

#### Build

##### Automatic compilation

* Download: [Debian rootfs(64-bit)](https://en.t-firefly.com/doc/download/337.html)，put in SDK path

```bash
7z x debian_rk3588_rootfs_xxx.7z
mkdir debian
mv debianxx-rootfs.img debian/debian-rootfs.img
```

* start compiling
```bash
./build.sh
```
the firmware will be saved to the directory `rockdev/pack/`.

##### Partial compilation

* u-boot

```bash
./build.sh uboot
```

* kernel

```bash
./build.sh extboot
```

* recovery

```bash
./build.sh recovery
```

* Download: [Debian rootfs(64-bit)]()，put in SDK path

```bash
7z x debian_rk3588_rootfs_xxx.7z
mkdir debian
mv debianxx-rootfs.img debian/rootfs.img
```

* Update each part of the `.img` link to the directory `rockdev/`:

```bash
./mkfirmware.sh
```

* Pack the firmware, the firmware will be saved to the directory `rockdev/pack/`.

```bash
./build.sh updateimg
```
## Compile Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware. It is recommended to develop under Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

<font color=red>**The compilation portion of this tutorial works with SDK versions above v1.0.6e**</font>

```
$ readlink -f .repo/manifest.xml
/home/daijh/p/rk3588/.repo/manifests/rk3588/rk3588_linux_release_20230301_v1.0.6e.xml
```
### Preparatory work

#### Set up compilation environment

```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip \
device-tree-compiler ncurses-dev \
```

### Compile SDK

#### Precompile Configuration

There are configuration files for different board in `device/rockchip/rk3588/`, select the configuration file:

```bash
./build.sh firefly_rk3588_aio-3588sjd4-ai_buildroot_defconfig
```

#### Build

##### Automatic compilation

* start compiling
```bash
./build.sh
```
the firmware will be saved to the directory `rockdev/pack/`.

##### Partial compilation

* u-boot

```bash
./build.sh uboot
```

* kernel

```bash
./build.sh extboot
```

* recovery

```bash
./build.sh recovery
```

* buildroot

```bash
./build.sh rootfs
```

* Update each part of the `.img` link to the directory `rockdev/`:

```bash
./mkfirmware.sh
```

* Pack the firmware, the firmware will be saved to the directory `rockdev/pack/`.

```bash
./build.sh updateimg
```