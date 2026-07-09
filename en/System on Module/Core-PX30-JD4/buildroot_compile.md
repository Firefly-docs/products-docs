# Build Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware.

## Build Linux-SDK

### Precompile Configuration

There are configuration files for different board in `device/rockchip/px30/`, select the configuration file:

```bash
./build.sh px30-lvds-buildroot.mk
```

The configuration file will be connected to `device/rockchip/.BoardConfig.mk`, check the file to verify whether the configuration is successful.

Configruation content:

```bash
# Target arch
export RK_ARCH=arm64                                            # 64-bit ARM
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=evb-px30                              # u-boot configuration
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=px30_linux_defconfig                 # kernel configuration
# Kernel dts
export RK_KERNEL_DTS=px30-firefly-lvds                          # dts file
# Buildroot config
export RK_CFG_BUILDROOT=rockchip_px30_64                        # Buildroot configuration
# Recovery config
export RK_CFG_RECOVERY=rockchip_px30_recovery                   # recovery configuration
# parameter for GPT table
export RK_PARAMETER=parameter.txt                               # partition table
# rootfs image path
export RK_ROOTFS_IMG=buildroot/output/$RK_CFG_BUILDROOT/images/rootfs.$RK_ROOTFS_TYP    # filesystem path
```

### Partial compilation

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

* Buildroot filesystem

Compile the Buildroot filesystem and generate the compiled output directory in `buildroot/output`:

```bash
./build.sh buildroot
```

### Pack the firmware

Update each part of the `.img` link to the directory `rockdev/`:

```bash
./mkfirmware.sh
```

Pack the firmware, the firmware will be saved to the directory `rockdev/pack/`.

```bash
./build.sh updateimg
```

### Automatic compilation

The automatic compilation will perform the above compilation and packaging operations to generate complete firmware.

```bash
./build.sh
```

## Partition table

### parameter

The parameter.txt file contains the partition information of the firmware. Take parameter.txt as an example:

path: `device/rockchip/px30/parameter.txt`

```bash
FIRMWARE_VER: 8.1
MACHINE_MODEL: PX30
MACHINE_ID: 007
MANUFACTURER: PX30
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: px30
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(trust),0x00002000@0x00008000(misc),0x00010000@0x0000a000(boot),0x00010000@0x0001a000(recovery),0x00010000@0x0002a000(backup),0x00020000@0x0003a000(oem),0x00c00000@0x0005a000(rootfs),-@0x00c5a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is where we are concerned. Take uboot as an example. In 0x00002000@0x00004000(uboot), 0x00004000 is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

### package-file

The package-file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt.

path: `tools/linux/Linux_Pack_Firmware/rockdev/px30-package-file`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
trust           Image/trust.img
uboot           Image/uboot.img
misc            Image/misc.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
oem             Image/oem.img
userdata:grow   Image/userdata.img
backup          RESERVED
```
