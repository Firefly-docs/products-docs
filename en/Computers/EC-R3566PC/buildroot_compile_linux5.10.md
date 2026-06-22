## Compile Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware. It is recommended to develop in the Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

### Compile SDK

#### Configuration before compilation

In the `device/rockchip/rk3566_rk3568/` directory, there are configuration files of different board types.

Return to SDK root directory to select the configuration file:

```bash
./build.sh 
```

The configuration file will be linked to `output/defconfig`, check the file to verify whether the configuration is successful.

Related configuration introduction:

```bash
# device tree name
RK_KERNEL_DTS_NAME=""
# kernel defconfig
RK_KERNEL_CFG=""
# use FIT image
RK_USE_FIT_IMG=y
# UART for bluetooth
RK_WIFIBT_TTY="ttyS8"
# parameter
RK_PARAMETER="parameter-buildroot-fit.txt"
# image tree source of boot partition image
RK_BOOT_FIT_ITS="bootramdisk.its"
# ramdisk image
RK_RAMDISK_IMG="ramdisk.img"
# use extlinux way to load kernel
USE_EXTBOOT=y
```

#### Select Accessories to Compile

Under `device/rockchip/rk3566_rk3568/` , in addition to , there are other configuration files that come with different accessories

The name of the configuration file indicates the screen and camera used.
If the screen is not specified, the default HDMI display is used.
If the camera is not specified, the default single camera is used

```bash
# For example:
xxxx-buildroot_defconfig               # Use HDMI + single-camera
xxxx-2cam-buildroot_defconfig          # Use HDMI + dual-camera
xxxx-mipi-buildroot_defconfig          # Use mipi + single-camera
xxxx-mipi-2cam-buildroot_defconfig     # Use mipi + dual-camera
```

Chose the configuration file you like, execute `build.sh` to make it effective:

```bash
# For example:
./build.sh xxxx-mipi_buildroot_defconfig
```

#### Automatic compilation

Fully automatic compilation will perform all compilation and packaging operations to generate complete firmware.

```bash
./build.sh all
```
Firmware will be saved to `output/update/` directory

#### Partial compilation

* Compile u-boot

```bash
./build.sh uboot
```
Output image is u-boot/uboot.img

* Compile kernel

Notice：Firefly kernel does not come with all kernel features, need extra kernel features please refer to [Kernel](kernel_introduction.md)

```bash
./build.sh extboot
```
Output image is kernel/extboot.img

* Compile recovery

```bash
./build.sh recovery
```
Outputs are under output/recovery

* Compile Buildroot root filesystem

```bash
./build.sh buildroot

# Note: Make sure to compile the Buildroot root filesystem as a normal user to avoid unnecessary errors.
```
Compiling the Buildroot root filesystem will generate a compilation output directory in `buildroot/output`

#### Package the firmware


```bash
./build.sh updateimg
```
Pack the firmware, the generated complete firmware will be saved to the `output/update/` directory.

### Partition description

#### parameter partition table

The parameter.txt file contains the partition information of the firmware. Take `parameter-buildroot-fit.txt` as an example:

Path: `device/rockchip/rk3566_rk3568/parameter-buildroot-fit.txt`

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
GROW_ALIGN: 0
CMDLINE: mtdparts=:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00020000@0x00048000(recovery),0x00010000@0x00068000(backup),0x00040000@0x00078000(oem),0x00c00000@0x000b8000(rootfs),-@0x00cb8000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is our focus. Taking uboot as an example, 0x00004000 in 0x00002000@0x00004000(uboot) is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on. The unit is block, each block is 512 Byte.