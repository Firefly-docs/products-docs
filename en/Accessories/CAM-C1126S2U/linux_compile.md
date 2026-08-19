# Compile Linux firmware

## Download Firefly_Linux_SDK

First prepare an empty folder to place SDK, better under home, here we use `~/proj` as example.

<font color=red>

**Attention:**

**1. SDK uses cross-compile, so download SDK to your X86_64 PC, do not download SDK to arm64 board.**

**2. Build environment needs to be Ubuntu18.04 (real PC or docker container), other versions may cause build failure.**

**3. Do not put SDK under shared directory of VM or non-English path.**

**4. Please get/build SDK as a normal user, root privilege are neither allowed nor required (except installing sth. with apt)**
</font>


<font color=red>**Very important:**</font>

Before development, please update the SDK to the latest version according to the WIKI, especially for users who use the CS-R1 cluster server (if the code version of the CS-R1 user is too old and the software level configuration does not match the hardware, it may cause <font color=red>**hardware damage**</font>).

For `CORE-1126-JD4/CORE-1109-JD4` series, please download `rv1126_rv1109_linux_release` Linux SDK package.

If you want to develop UVC applications for the `CORE-1126-JD4/CORE-1109-JD4` series, please download the `rv1126_rv1109_linux_ai_camera_release` Linux SDK package.

### SDK Development Kit Description

`rv1126_rv1109_linux_release` SDK Description:
1. `CORE-1126-JD4/CORE-1109-JD4` series products can develop IPC firmware (IPC is the network camera)
2. `CAM-C1126S2U/CAM-C1109S2U` series products can develop Facial_gate firmware (Facial_gate is the face recognition gate)

`rv1126_rv1109_linux_ai_camera_release` SDK description:
1. `CORE-1126-JD4/CORE-1109-JD4` series products can develop AI_UVC firmware (UVC means driver-free camera)
2. `CAM-C1126S2U/CAM-C1109S2U` series products can develop AI_UVC firmware (UVC means driver-free camera)

Full SDK and BSP SDK Description:

`rv1126_rv1109_linux_release` and `rv1126_rv1109_linux_ai_camera_release` are both FULL SDKs.

`rv1126_rv1109_linux_bsp_release` and `rv1126_rv1109_linux_ai_camera_bsp_release` are both BSP SDKs.

The difference between the FULL SDK and the BSP SDK:
1. The FULL SDK can compile the buildroot system. The file size is relatively large. It contains the hardware acceleration file directory provided by rockchip. It is recommended to pull the FULL SDK for the first use.
2. The BSP SDK cannot compile the buildroot system. But the file size is relatively small. It is provided to customers who only need to update the partition firmware such as u-boot and kernel based on the public version of the firmware.

The similarities between the FULL SDK and the BSP SDK:
1. Both can compile and package Debian10 firmware.


### Install Tools
```
sudo apt update
sudo apt install -y repo git python
```

### Init Code Repository

* Method One

Download via repo, you can choose to get FULL SDK or BSP SDK:

* `rv1126_rv1109_linux_release` FULL SDK pull method
```bash
mkdir ~/proj/rv1126_sdk/
cd ~/proj/rv1126_sdk/

repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_release.xml

repo sync -c
# You need to repeatedly confirm whether the code download is successful
```

* `rv1126_rv1109_linux_bsp_release` BSP SDK pull method
```bash
mkdir ~/proj/rv1126_sdk/
cd ~/proj/rv1126_sdk/

repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_bsp_release.xml

repo sync -c
# You need to repeatedly confirm whether the code download is successful
```



* `rv1126_rv1109_linux_ai_camera_release` FULL SDK pull method

```bash
mkdir ~/proj/rv1126_sdk_ai/
cd ~/proj/rv1126_sdk_ai/

repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_ai_camera_release.xml

repo sync -c
# You need to repeatedly confirm whether the code download is successful
```



* `rv1126_rv1109_linux_ai_camera_bsp_release` BSP SDK pull method

```bash
mkdir ~/proj/rv1126_sdk_ai/
cd ~/proj/rv1126_sdk_ai/

repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_ai_camera_bsp_release.xml

repo sync -c
# You need to repeatedly confirm whether the code download is successful
```

Note: The default repo tool uses python2 to pull code. If the Linux PC system is relatively new, the default version may be python3. If the code cannot be pulled, please manually set the python version of the Linux PC system to python2. And use the `.repo/repo/repo` tool in the current directory to pull the code. View the default python version of the Linux PC system:

```shell
$ ls -l /usr/bin/python
lrwxrwxrwx 1 root root 9 4月  16  2018 /usr/bin/python -> python2.7*
```



* Method Two

Download Firefly_Linux_SDK sub-volume compressed package: [Linux SDK](https://en.t-firefly.com/doc/download/86.html)

Note: This method can only obtain the complete SDK. The BSP SDK needs to be obtained using method 1.

* `rv1126_rv1109_linux_release` SDK download

After downloading, verify the MD5 code:

```bash
md5sum rv1126_rv1109_linux_release_20211022.tgz
596c6bc6bb3095aea97d54c9df4cf333
```

After confirming that it is correct, you can unzip:
```bash
mkdir -p ~/proj/rv1126_sdk
cd ~/proj/rv1126_sdk
mv /path/to/rv1126_rv1109_linux_release_20211022.tgz ./
tar xvf rv1126_rv1109_linux_release_20211022.tgz

# This compressed package contains a .repo directory. After decompression, perform the following operations in the current directory
.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all

# You can use the following command to update the SDK later
.repo/repo/repo sync -c --no-tags

# Due to network environment and other reasons, the `.repo/repo/repo sync -c --no-tags` command may fail to update the code. You can execute it repeatedly.
```

* `rv1126_rv1109_linux_ai_camera_release` SDK download

After downloading, verify the MD5 code:

```bash
md5sum rv1126_rv1109_linux_ai_camera_release_20210306.tgz
26fcb17ddd82a43d396eb3eac4b7479f
```

After confirming that it is correct, you can unzip:
```bash
mkdir -p ~/proj/rv1126_sdk_ai
cd ~/proj/rv1126_sdk_ai
mv /path/to/rv1126_rv1109_linux_ai_camera_release_20210306.tgz ./
tar xvf rv1126_rv1109_linux_ai_camera_release_20210306.tgz

# This compressed package contains a .repo directory. After decompression, perform the following operations in the current directory
.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all

# You can use the following command to update the SDK later
.repo/repo/repo sync -c --no-tags

# Due to network environment and other reasons, the `.repo/repo/repo sync -c --no-tags` command may fail to update the code. You can execute it repeatedly.
```

### Synchronize Code

Execute the following command to synchronize code:

* Enter the root directory of rv1126_rv1109_linux_release SDK

```bash
cd ~/proj/rv1126_sdk
```

* Enter the root directory of rv1126_rv1109_linux_ai_camera_release SDK

```bash
cd ~/proj/rv1126_sdk_ai
```

* Synchronize command

```bash
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

You can use the following command to update the SDK later:

```bash
.repo/repo/repo sync -c --no-tags
```

Due to network environment and other reasons, `.repo/repo/repo sync -c --no-tags` The command to update the code may fail and can be executed repeatedly.

## Linux SDK Configuration Introduction

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

In the `device/rockchip/rv1126_rv1109/` directory, there are configuration files (xxxx.mk) for different board types, which are used to manage the compilation configuration of each project of the SDK. The relevant configuration introduction:

```bash
# Target arch
export RK_ARCH=arm # 32-bit ARM architecture
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=xxxx_defconfig # u-boot configuration file
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=xxxx_defconfig # kernel configuration file
# Kernel defconfig fragment
export RK_KERNEL_DEFCONFIG_FRAGMENT=xxxx.config # kernel configuration file (fragment)
# Kernel dts
export RK_KERNEL_DTS=xxxx.dts # dts file
# parameter for GPT table
export RK_PARAMETER=parameter-xxxx.txt # partition table
# rootfs image path
export RK_ROOTFS_IMG=ubuntu_rootfs/rootfs.img # root file system path
```

### Partition table

#### parameter

The parameter.txt file contains the partition information of the firmware. Take parameter-ubuntu-fit.txt as an example:

path: `device/rockchip/rv1126_rv1109/parameter-xxxxxx-fit.txt`

```bash
FIRMWARE_VER: 8.1
MACHINE_MODEL: RV1126
MACHINE_ID: 007
MANUFACTURER: RV1126
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00010000@0x00008000(boot),0x00010000@0x00018000(recovery),0x00010000@0x00028000(backup),0x00300000@0x00038000(rootfs),0x00060000@0x00338000(oem),0x00400000@0x00398000(userdata),-@0x00798000(media:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is where we are concerned. Take uboot as an example. In 0x00002000@0x00004000(uboot), 0x00004000 is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

#### package-file

The package-file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt.

path: `tools/linux/Linux_Pack_Firmware/rockdev/rv1126_rv1109-package-file`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
#
# if uboot.img is fit, uboot.img had include uboot and trust,
# so ignore trust.img
# file Image/uboot.img
# Image/uboot.img: Device Tree Blob version 17
#
# trust         Image/trust.img
#
uboot           Image/uboot.img
misc            Image/misc.img
#resource       Image/resource.img
#kernel         Image/kernel.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
oem             Image/oem.img
userdata        Image/userdata.img
# 要写入backup分区的文件就是自身（update.img）
# SELF 是关键字，表示升级文件（update.img）自身
# 在生成升级文件时，不加入SELF文件的内容，但在头部信息中有记录
# 在解包升级文件时，不解包SELF文件的内容。
backup          RESERVED
#update-script  update-script
#recover-script recover-script
```


## Compile Debian firmware

This chapter introduces the compilation process of Debian firmware. It is recommended to develop under Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

<font color=red>**The compilation portion of this tutorial works with SDK versions above v2.2.5f**</font>

```
$ realpath .repo/manifest.xml
/home/lvsx/project/rv1126_rv1109/.repo/manifests/rv1126_rv1109/rv1126_rv1109_linux_release_20240117_v2.2.5f.xml
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

#### Pre-compilation configuration

In the `device/rockchip/rv1126_rv1109/` directory, there are configuration files for different board types. Select the configuration file:

#### Compile Debian firmware
* Pre-compilation configuration for V2 mipi screen (old version screen, which has been discontinued)
```
./build.sh cam-crv1126s2u-facial_gate-BE-45-debian.mk
```
* Compile V3 mipi screen configuration

```bash
./build.sh cam-crv1126s2u-facial_gate-mipi101-BSD1218-A101KL68-debian.mk
```

#### Build

##### Automatic compilation

* Download: [Debian rootfs(32-bit)](https://en.t-firefly.com/doc/download/86.html)，put in SDK path

```bash
7z x debian10_rootfs_2022_12_22.7z
mkdir ubuntu_rootfs
mv rootfs.img ubuntu_rootfs/rootfs.img
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
./build.sh kernel
```

* recovery

```bash
./build.sh recovery
```

* Download: [Debian rootfs(32-bit)]()，put in SDK path

```bash
7z x debian10_rootfs_2022_12_22.7z
mkdir ubuntu_rootfs
mv rootfs.img ubuntu_rootfs/rootfs.img
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

<font color=red>**The compilation portion of this tutorial works with SDK versions above v2.2.5f**</font>

```
$ realpath .repo/manifest.xml
/home/lvsx/project/rv1126_rv1109/.repo/manifests/rv1126_rv1109/rv1126_rv1109_linux_release_20240117_v2.2.5f.xml
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

There are configuration files for different board in `device/rockchip/rv1126_rv1109/`, select the configuration file:

`rv1126_rv1109_linux_release` SDK and `rv1126_rv1109_linux_bsp_release` SDK Select configuration file:
```bash



# V2 MIPI screen (old version screen, this screen has been discontinued)
./build.sh cam-crv1126s2u-facial_gate-BE-45.mk 

# V3 MIPI screen
./build.sh cam-crv1126s2u-facial_gate-mipi101-BSD1218-A101KL68.mk


```

`rv1126_rv1109_linux_ai_camera_release` SDK and `rv1126_rv1109_linux_ai_camera_bsp_release` SDK select configuration file:


Since uvc firmware generally does not need to use mipi screen for display . If you need a mipi display, you can select the corresponding screen to compile the firmware.

```bash
# V2 MIPI screen (old version screen, this screen has been discontinued)
./build.sh cam-crv1126s2u-uvcc-BE-45.mk 

# V3 MIPI screen
./build.sh cam-crv1126s2u-uvcc-mipi101-BSD1218-A101KL68.mk 
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
./build.sh kernel
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
