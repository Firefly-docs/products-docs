## Build Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware.

### Build Linux-SDK

#### Precompile Configuration

There are configuration files for different board in `device/rockchip/rk3288/`, select the configuration file:

##### HDMI

```bash
./build.sh aio-3288c-buildroot.mk
```


##### LVDS DM-M10R800

```bash
./build.sh aio-3288c-lvds-buildroot.mk
```


The configuration file will be connected to `device/rockchip/.BoardConfig.mk`, check the file to verify whether the configuration is successful.

Configruation content:

```bash
# Target arch
export RK_ARCH=arm                                              # 32-bit ARM
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly-rk3288                        # u-boot configuration
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly_linux_defconfig              # kernel configuration
# Kernel dts
export RK_KERNEL_DTS=rk3288-firefly-aioc                              # dts file
# Buildroot config
export RK_CFG_BUILDROOT=rockchip_rk3288                         # Buildroot configuration
# Recovery config
export RK_CFG_RECOVERY=rockchip_rk3288_recovery                 # recovery configuration
# parameter for GPT table
export RK_PARAMETER=parameter-buildroot.txt                     # partition table
# rootfs image path
export RK_ROOTFS_IMG=buildroot/output/$RK_CFG_BUILDROOT/images/rootfs.$RK_ROOTFS_TYPE   # filesystem path
```

#### Partial compilation

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

# Note: Be sure to compile the Buildroot filesystem as a normal user to avoid unnecessary errors.
```

#### Pack the firmware

Update each part of the `.img` link to the directory `rockdev/`:

```bash
./mkfirmware.sh
```

Pack the firmware, the firmware will be saved to the directory `rockdev/pack/`.

```bash
./build.sh updateimg
```

#### Automatic compilation

The automatic compilation will perform the above compilation and packaging operations to generate complete firmware.

```bash
./build.sh
```

### Partition table

#### parameter

The parameter.txt file contains the partition information of the firmware. Take parameter-buildroot.txt as an example:

path: `device/rockchip/rk3288/parameter-buildroot.txt`

```bash
FIRMWARE_VER: 8.1
MACHINE_MODEL:RK3288
MACHINE_ID:007
MANUFACTURER:RK3288
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3288
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(trust),0x00002000@0x00008000(misc),0x00010000@0x0000a000(boot),0x00010000@0x0001a000(recovery),0x00010000@0x0002a000(backup),0x00020000@0x0003a000(oem),0x00100000@0x0005a000(rootfs),-@0x0015a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is where we are concerned. Take uboot as an example. In 0x00002000@0x00004000(uboot), 0x00004000 is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

#### package-file

The package-file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt.

path: `tools/linux/Linux_Pack_Firmware/rockdev/rk3288-package-file`

```bash
# name          relative path
#
#hwdef          hwdef
package-file    package-file
bootloader      image/miniloaderall.bin
parameter       image/parameter.txt
trust           image/trust.img
uboot           image/uboot.img
misc            image/misc.img
boot            image/boot.img
recovery        image/recovery.img
rootfs          image/rootfs.img
oem             image/oem.img
userdata:grow   image/userdata.img
backup          RESERVED
```