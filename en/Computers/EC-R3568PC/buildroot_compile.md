## Compile Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware. It is recommended to develop in the Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

### Compile SDK

#### Configuration before compilation

In the `device/rockchip/rk356x/` directory, there are configuration files of different board types.

Return to SDK root directory to select the configuration file:

```bash
./build.sh 
```

The configuration file will be linked to `device/rockchip/.BoardConfig.mk`, check the file to verify whether the configuration is successful.

Related configuration introduction:

```bash
# Target arch
export RK_ARCH=arm64 # 64-bit ARM architecture
# Uboot defconfig
export RK_UBOOT_DEFCONFIG= # u-boot configuration file
# Kernel defconfig
export RK_KERNEL_DEFCONFIG= # kernel configuration file
# Kernel dts
export RK_KERNEL_DTS= # dts file
# Buildroot config
export RK_CFG_BUILDROOT= # Buildroot configuration
# Recovery config
export RK_CFG_RECOVERY=rockchip_rk356x_recovery # recovery configuration
# parameter for GPT table
export RK_PARAMETER=parameter-buildroot-fit.txt # Partition table
# rootfs image path
export RK_ROOTFS_IMG=rockdev/rootfs.${RK_ROOTFS_TYPE} # root filesystem path
```

#### Select Accessories to Compile

Under `device/rockchip/rk356x/` , in addition to , there are other configuration files that come with different accessories

The name of the configuration file indicates the screen and camera used.
If the screen is not specified, the default HDMI display is used.
If the camera is not specified, the default single camera is used

```bash
# For example:
xxxx-buildroot.mk              # Use HDMI + single-camera
xxxx-2cam-buildroot.mk         # Use HDMI + dual-camera
xxxx-mipi-buildroot.mk         # Use mipi + single-camera
xxxx-mipi-2cam-buildroot.mk    # Use mipi + dual-camera
```

Chose the configuration file you like, execute `build.sh` to make it effective:

```bash
# For example:
./build.sh xxxx-mipi-2cam-buildroot.mk
```

#### Automatic compilation

Fully automatic compilation will perform all compilation and packaging operations to generate complete firmware.

```bash
./build.sh
```

#### Partial compilation

* Compile u-boot

```bash
./build.sh uboot
```

* Compile kernel

Notice：Firefly kernel does not come with all kernel features, need extra kernel features please refer to [Kernel](kernel_introduction.md)

```bash
./build.sh kernel
```

* Compile recovery

```bash
./build.sh recovery
```

* Compile Buildroot root filesystem

Compiling the Buildroot root filesystem will generate a compilation output directory in `buildroot/output`:

```bash
./build.sh buildroot

# Note: Make sure to compile the Buildroot root filesystem as a normal user to avoid unnecessary errors.
```

#### Package the firmware

Update each part of the mirror link to the `rockdev/` directory:

```bash
./mkfirmware.sh
```

Pack the firmware, the generated complete firmware will be saved to the `rockdev/pack/` directory.

```bash
./build.sh updateimg
```

### Partition description

#### parameter partition table

The parameter.txt file contains the partition information of the firmware. Take `parameter-buildroot-fit.txt` as an example:

Path: `device/rockchip/rk356x/parameter-buildroot-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3568
MACHINE_ID: 007
MANUFACTURER: RK3568
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00010000@0x00008000(boot),0x00010000@0x00018000(recovery),0x00010000@0x00028000(backup),0x00040000@0x00038000(oem),0x00c00000@0x00078000(rootfs),-@0x00c78000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is our focus. Taking uboot as an example, 0x00004000 in 0x00002000@0x00004000(uboot) is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

#### package-file

The package-file file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt file.

Path: `tools/linux/Linux_Pack_Firmware/rockdev/rk356x-package-file`

```bash
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
uboot           Image/uboot.img
misc            Image/misc.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
oem             Image/oem.img
userdata        Image/userdata.img
backup          RESERVED
```