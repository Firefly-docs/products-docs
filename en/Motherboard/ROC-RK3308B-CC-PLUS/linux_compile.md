# Compile Linux firmware

## Build the build environment

This chapter introduces the compilation process of the Linux SDK

<font color=red>**Note: (1) It is recommended to develop in the Ubuntu 18.04 system environment. If other system versions are used, the compilation environment may need to be adjusted accordingly.** </font>

<font color=red>**(2) Compile with ordinary user, do not compile with root user authority.** </font>

### Install dependencies

```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler python-pip ncurses-dev python-pyelftools
````

### Download Firefly_Linux_SDK

* Method 1 (recommended)

**Because the Firefly_Linux_SDK source code package is relatively large, some users' computers do not support files above 4G or the network transmission of a single file is slow, so we use the method of volume compression to package the SDK. Users can obtain the Firefly_Linux_SDK source package in the following ways: **[Firefly_Linux_SDK source package](https://www.t-firefly.com/doc/download/97.html#other_300)

After the download is complete, verify the MD5 code first:

```bash
$ md5sum rk3308_linux_release_v1.5.0a_20221212_split_dir/*firefly_split*
a97f218aed44be1b9a423a2868e09335  rk3308_linux_release_v1.5.0a_20221212_split_dir/rk3308_linux_release_v1.5.0a_20221212_firefly_split.file0
3f3936a72635b75613b89a4bf8d9de39  rk3308_linux_release_v1.5.0a_20221212_split_dir/rk3308_linux_release_v1.5.0a_20221212_firefly_split.file1
794ea504543a797d972c2112dfef2f3f  rk3308_linux_release_v1.5.0a_20221212_split_dir/rk3308_linux_release_v1.5.0a_20221212_firefly_split.file2
0de1858d73e2841790006e487ff70a19  rk3308_linux_release_v1.5.0a_20221212_split_dir/rk3308_linux_release_v1.5.0a_20221212_firefly_split.file3
````

After confirming that it is correct, you can unzip it:

<font color=red>**Note: Do not decompress the SDK in shared folders, mount folders and non-English directories to avoid unnecessary errors**</font>

```bash
# unzip
mkdir ~/proj/
cd ~/proj/
cat path/to/rk3308_linux_release_v1.5.0a_20221212_split_dir/*firefly_split* | tar -xzv

# export data
.repo/repo/repo sync -l
````

* Method Two

Pulling the code through repo, this method has high network requirements and can be used under conditions

You can choose to get the full SDK or BSP:

```bash
mkdir ~/proj/rk3308_linux_release_v1.5.0a_20221212/
cd ~/proj/rk3308_linux_release_v1.5.0a_20221212/

## Full SDK
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3308_linux_release.xml

## BSP (only the base repository and compilation tools are included)
## BSP includes device/rockchip , docs , kernel , u-boot , rkbin , tools and cross compile chain
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3308_linux_bsp_release.xml
````

### Sync code

Execute the following command to synchronize the code:

```bash
# Enter the SDK root directory
cd ~/proj/rk3308_linux_release_v1.5.0a_20221212/

# Synchronize
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
````

The SDK can then be updated with the following command:

```bash
.repo/repo/repo sync -c --no-tags
````

Due to the network environment and other reasons, the `.repo/repo/repo sync -c --no-tags` command may fail to update the code, and it can be executed repeatedly.
## Compile Ubuntu firmware

This chapter introduces the compilation process of Ubuntu firmware. It is recommended to develop in the Ubuntu 18.04 system environment. If other system versions are used, the compilation environment may need to be adjusted accordingly.

### Ubuntu firmware brief introduction

[What is Ubuntu Minimal firmware? ](ubuntu_minimal_support.md)


### Compile SDK

#### Configuration before compilation

The configuration files of different board types are stored in the `device/rockchip/rk3308/` directory

Go back to the SDK root directory and execute `build.sh` to select the configuration file:

```bash
./build.sh roc-rk3308b-cc-plus-ubuntu.mk
````

The configuration file will be linked to `device/rockchip/.BoardConfig.mk`, check this file to verify that the configuration was successful.

Related configuration introduction:

```bash
# Target arch
export RK_ARCH=arm64 # 64-bit ARM architecture
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly-rk3308-debug-uart4 # u-boot configuration file
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly-rk3308b_linux_defconfig # kernel configuration file
# Kernel dts
export RK_KERNEL_DTS=rk3308b-roc-cc-plus-amic_emmc # dts file
# parameter for GPT table
export RK_PARAMETER=parameter-64bit-ubuntu.txt # partition table
# rootfs image path
export RK_ROOTFS_IMG=ubuntu_rootfs/rk3308-ubuntu_rootfs.img # root file system path
````

#### Download Ubuntu root filesystem

* Download the root file system: [Ubuntu root file system (64-bit)](https://en.t-firefly.com/doc/download/84.html#other_528), put it under the SDK path

* Put the root file system in the `ubuntu_rootfs/` directory of the SDK

```bash
mkdir ubuntu_rootfs
mv ubuntu-aarch64-rootfs.img ubuntu_rootfs/rk3308-ubuntu_rootfs.img
````

#### Fully automatic compilation

Fully automatic compilation will perform all compilation and packaging operations to generate RK firmware directly.

```bash
./build.sh
````

#### Partial compilation

* Compile u-boot

```bash
./build.sh uboot
````

* compile kernel

```bash
./build.sh kernel
````

* compile recovery

```bash
./build.sh recovery
````

#### Update link

Update each part of the mirror link to the `rockdev/` directory:

```bash
./mkfirmware.sh
````

#### Package firmware

Pack the firmware, and the generated complete firmware will be saved to the `rockdev/pack/` directory.

```bash
# Package RK firmware
./build.sh updateimg
````

### partition description

#### parameter partition table

The parameter.txt file contains the partition information of the firmware. Take parameter-ubuntu-fit.txt as an example:

Path: `device/rockchip/rk3308/parameter-64bit-ubuntu.txt`

```bash
FIRMWARE_VER:8.1
MACHINE_MODEL:RK3308
MACHINE_ID:007
MANUFACTURER: RK3308
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3308
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00001000@0x00002000(uboot),0x00001000@0x00003000(trust),0x00000800@0x00004000(misc),0x0000A000@0x00004800(recovery),0x0000A000@0x0000E800(boot),0x00100000@0x00018800(rootfs),-@ 0x118800 (userdata: grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
````

The CMDLINE attribute is what we focus on. Taking uboot as an example, 0x00001000 in 0x00001000@0x00002000(uboot) is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

#### package-file

The package-file file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt file.

Path: `tools/linux/Linux_Pack_Firmware/rockdev/rk3308-package-file-ubuntu`

```bash
# NAME Relative path
#
#HWDEF HWDEF
package-file package-file
bootloader Image/MiniLoaderAll.bin
parameter Image/parameter.txt
trust Image/trust.img
uboot Image/uboot.img
boot Image/boot.img
rootfs Image/rootfs.img
recovery Image/recovery.img
#oem Image/oem.img
userdata:grow RESERVED
misc Image/misc.img
backup RESERVED
#update-script update-script
#recover-script recover-script
````

### more content

["Firefly Ubuntu User Manual"](https://wiki.t-firefly.com/en/Firefly-Linux-Guide/manual_ubuntu.html)
## Compile Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware. It is recommended to develop in the Ubuntu 18.04 system environment. If other system versions are used, the compilation environment may need to be adjusted accordingly.

### Compile SDK

#### Configuration before compilation

The configuration files of different board types are stored in the `device/rockchip/rk3308/` directory

Go back to the SDK root directory and execute `build.sh` to select the configuration file:

```bash
./build.sh roc-rk3308b-cc-plus-buildroot.mk
````

The configuration file will be linked to `device/rockchip/.BoardConfig.mk`, check this file to verify that the configuration was successful.

Related configuration introduction:

```bash
# Target arch
export RK_ARCH=arm64 # 64-bit ARM architecture
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly-rk3308-debug-uart4 # u-boot configuration file
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly-rk3308b_linux_defconfig # kernel configuration file
# Kernel dts
export RK_KERNEL_DTS=rk3308b-roc-cc-plus-amic_emmc # dts file
# Buildroot config
export RK_CFG_BUILDROOT=firefly_rk3308_release # Buildroot configuration
# Recovery config
export RK_CFG_RECOVERY=firefly_rk3308_recovery # recovery configuration
# parameter for GPT table
export RK_PARAMETER=parameter-64bit-emmc.txt # Partition table
# rootfs image path
export RK_ROOTFS_IMG=rockdev/rootfs.${RK_ROOTFS_TYPE} # root file system path
````


Introduction to the configuration file of buildroot:

- roc-rk3308b-cc-plus-buildroot.mk

   - Screen mod: not supported

   - UI interface: none

- roc-rk3308b-cc-plus-rgb-4.0inch-qt.mk

   - Screen module: 4.0 inch RGB LCD screen module
   - UI interface: Qt demo

- roc-rk3308b-cc-plus-rgb-7.0inch-qt.mk

   - Screen module: 7.0 inch RGB LCD screen module
   - UI interface: Qt demo


#### Fully automatic compilation

Fully automated compilation performs all compilation and packaging operations to generate the complete firmware.

```bash
./build.sh
````

#### Partial compilation

* Compile u-boot

```bash
./build.sh uboot
````

* compile kernel

```bash
./build.sh kernel
````

* compile recovery

```bash
./build.sh recovery
````

* Compile the Buildroot root filesystem

Compiling the Buildroot root filesystem will generate the build output directory in `buildroot/output`:

```bash
./build.sh buildroot

# Note: Make sure to compile the Buildroot root filesystem as a normal user to avoid unnecessary errors.
````

#### Package firmware

Update each part of the mirror link to the `rockdev/` directory:

```bash
./mkfirmware.sh
````

Pack the firmware, and the generated complete firmware will be saved to the `rockdev/pack/` directory.

```bash
./build.sh updateimg
````

### partition description

#### parameter partition table

The parameter.txt file contains the partition information of the firmware. Take `parameter-buildroot-fit.txt` as an example:

Path: `device/rockchip/rk356x/parameter-buildroot-fit.txt`

```bash
FIRMWARE_VER:8.1
MACHINE_MODEL:RK3308
MACHINE_ID:007
MANUFACTURER: RK3308
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3308
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00001000@0x00002000(uboot),0x00001000@0x00003000(trust),0x00000800@0x00004000(misc),0x0000A000@0x00004800(recovery),0x0000A000@0x0000E800(boot),0x00080000@0x00018800(rootfs),0x00020000@ 0x00098800(oem),-@0x00B8800(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
````

The CMDLINE attribute is where we focus. Taking uboot as an example, 0x00001000 in 0x00001000@0x00004000(uboot) is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

#### package-file

The package-file file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt file.

Path: `tools/linux/Linux_Pack_Firmware/rockdev/rk3308-package-file`

```bash
# NAME Relative path
#
#HWDEF HWDEF
package-file package-file
bootloader Image/MiniLoaderAll.bin
parameter Image/parameter.txt
trust Image/trust.img
uboot Image/uboot.img
boot Image/boot.img
rootfs Image/rootfs.img
recovery Image/recovery.img
oem Image/oem.img
userdata:grow Image/userdata.img
misc Image/misc.img
backup RESERVED
#update-script update-script
#recover-script recover-script
````

### more content

["Firefly Buildroot User Manual"](https://wiki.t-firefly.com/en/Firefly-Linux-Guide/manual_buildroot.html)