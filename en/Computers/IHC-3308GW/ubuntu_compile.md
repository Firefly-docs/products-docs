## Compile Ubuntu firmware

This chapter introduces the compilation process of Ubuntu firmware. It is recommended to develop in the Ubuntu 18.04 system environment. If other system versions are used, the compilation environment may need to be adjusted accordingly.

### Ubuntu firmware brief introduction

[What is Ubuntu Minimal firmware? ](ubuntu_minimal_support.md)


### Compile SDK

#### Configuration before compilation

The configuration files of different board types are stored in the `device/rockchip/rk3308/` directory

Go back to the SDK root directory and execute `build.sh` to select the configuration file:

```bash
./build.sh ihc-3308gw-ubuntu.mk
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
export RK_KERNEL_DTS=rk3308b-roc-cc-plus-amic-ext_emmc # dts file
# parameter for GPT table
export RK_PARAMETER=parameter-64bit-ubuntu.txt # partition table
# rootfs image path
export RK_ROOTFS_IMG=ubuntu_rootfs/rk3308-ubuntu_rootfs.img # root file system path
````

#### Download Ubuntu root filesystem

* Download the root file system: [Ubuntu root file system (64-bit)](https://en.t-firefly.com/doc/download/92.html#other_528), put it under the SDK path

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