## Compile Ubuntu firmware

This chapter introduces the compilation process of Ubuntu firmware. It is recommended to develop under Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

### A brief introduction to Ubuntu firmware
[What is Ubuntu Minimal ?](ubuntu_minimal_support.html)

[What is Ubuntu Desktop ?](ubuntu_desktop_support.html)

### Build Linux-SDK

#### Precompile Configuration

There are configuration files for different board in `device/rockchip/rk3566_rk3568/`.

Return to SDK root directory to select the configuration file:

```bash
./build.sh 
```

The configuration file will be connected to `output/defconfig`, check the file to verify whether the configuration is successful.

Configruation content:

```bash
# prebuilt rootfs
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk356x_ubuntu_rootfs.img"
# UART for bluetooth
RK_WIFIBT_TTY="ttyS8"
# kernel defconfig
RK_KERNEL_CFG=""
# device tree name
RK_KERNEL_DTS_NAME=""
# ramdisk image
RK_RAMDISK_IMG="ramdisk.img"
# image tree source for boot partition image
RK_BOOT_FIT_ITS="bootramdisk.its"
# prebuilt recovery image
RK_RECOVERY_RAMDISK="rk356x-recovery-arm64.cpio.gz"
# parameter
RK_PARAMETER="parameter-ubuntu-fit.txt"
# use FIT image
RK_USE_FIT_IMG=y
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
xxxx-ubuntu_defconfig                       # Use HDMI + single-camera
xxxx-2cam_ubuntu_defconfig                  # Use HDMI + dual-camera
xxxx-mipi_ubuntu_defconfig                  # Use mipi + single-camera
xxxx-mipi_ubuntu_defconfig                  # Use mipi + dual-camera
```

Chose the configuration file you like, execute `build.sh` to make it effective:

```bash
# For example:
./build.sh xxxx-mipi_ubuntu_defconfig 
```

#### Download Ubuntu filesystem

* Download: [Ubuntu rootfs(64-bit)]()，put in SDK path

* Unzip

```bash
# For example, the archive you download is Ubuntu20.04-xxx_RK3568_KERNEL-5.10_xxx.7z
7z x Ubuntu20.04-xxx_RK3568_KERNEL-5.10_xxx.7z
```

* Move filesystem to `prebuilt_rootfs/`

```bash
mkdir prebuilt_rootfs
# For example, after unzip, the filesystem image is Ubuntu20.04-xxx_RK3568_KERNEL-5.10_xxx.img
mv Ubuntu20.04-xxx_RK3568_KERNEL-5.10_xxx.img prebuilt_rootfs/
ln -sf Ubuntu20.04-xxx_RK3568_KERNEL-5.10_xxx.img rk356x_ubuntu_rootfs.img
```

#### Automatic compilation

The automatic compilation will perform all compilation and packaging operations to generate RK firmware.

```bash
./build.sh all
```
Firmware will be saved to the directory `output/update/`.

#### Partial compilation

* u-boot

```bash
./build.sh uboot
```
Output image is u-boot/uboot.img

* kernel

Notice：Firefly kernel does not come with all kernel features, need extra kernel features please refer to [Kernel](kernel_introduction.md)

```bash
./build.sh extboot
```
Output image is kernel/extboot.img

#### Pack the firmware

```bash
./build.sh updateimg
```
Pack the firmware, the firmware will be saved to the directory `rockdev/pack/`.

### Partition table

#### parameter

The parameter.txt file contains the partition information of the firmware. Take parameter-ubuntu-fit.txt as an example:

path: `device/rockchip/rk3566_rk3568/parameter-ubuntu-fit.txt`

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
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00020000@0x00048000(recovery),0x00010000@0x00068000(backup),0x00c00000@0x00078000(rootfs),-@0x00c78000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is where we are concerned. Take uboot as an example. In 0x00002000@0x00004000(uboot), 0x00004000 is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on. The unit is block, each block is 512 Byte.