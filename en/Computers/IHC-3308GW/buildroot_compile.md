## Compile Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware. It is recommended to develop in the Ubuntu 18.04 system environment. If other system versions are used, the compilation environment may need to be adjusted accordingly.

### Compile SDK

#### Configuration before compilation

The configuration files of different board types are stored in the `device/rockchip/rk3308/` directory

Go back to the SDK root directory and execute `build.sh` to select the configuration file:

```bash
./build.sh ihc-3308gw-buildroot.mk
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
# Buildroot config
export RK_CFG_BUILDROOT=firefly_rk3308_release # Buildroot configuration
# Recovery config
export RK_CFG_RECOVERY=firefly_rk3308_recovery # recovery configuration
# parameter for GPT table
export RK_PARAMETER=parameter-64bit-emmc.txt # Partition table
# rootfs image path
export RK_ROOTFS_IMG=rockdev/rootfs.${RK_ROOTFS_TYPE} # root file system path
````


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