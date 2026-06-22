## Compile Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware. It is recommended to develop in the Ubuntu 22.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

### Compile SDK

#### Pre-compilation configuration

Configuration files for different board types are stored in the `/path/to/sdk/device/rockchip/rk3506/` directory

Go back to the SDK root directory and execute `build.sh` to select the configuration file:

```bash
./build.sh firefly_rk3506_roc-rk3506b-cc-amp-emmc_buildroot_defconfig
```

#### Unified compilation and packaging of SDK

RK3506 supports the AMP hybrid architecture design of Linux, HAL, and RT-Thread, so that different CPUs can run different systems to meet flexible product design requirements. The SDK currently uses the Linux + RT-Thread hybrid structure model by default, where the CPU running Linux is the main core and the CPU running rtt is the slave core; cores 0~1 run Linux, and core 2 runs RT-Thread.

##### Compilation configuration

The unified compilation configuration script of the SDK is located in the device/rockchip/rk3506/ directory. The compilation configuration script includes the configuration of U-Boot, Kernel, HAL, RT-Thread, and AMP-related CPU allocation, memory allocation and other configurations. Users can add or modify the configuration script file as needed to meet their own compilation needs. The main AMP configuration files are as follows:

```bash
amp_linux.its								# Linux + rtt AMP package ITS configuration file
firefly_rk3506_roc-rk3506b-cc-amp-emmc_buildroot_defconfig		# Linux + rtt corresponding configuration file
```

The unified compilation script tool supports one-click compilation and packaging of U-Boot, Kernel, HAL, RT-Thread, ROOTFS, etc., and generates corresponding Image images.

```bash
./build.sh
```

#### Partial compilation

* Compile u-boot

```bash
./build.sh uboot
```

* Compile kernel

**The hardware resources used by AMP need to be disabled in the kernel device tree and declared under the rockchip_amp tag in the rk3506-amp.dtsi device tree**

```bash
./build.sh kernel
```

* Compile recovery

```bash
./build.sh recovery
```

* Compile the Buildroot root file system

Compile the Buildroot root file system, and the compilation output directory will be generated in `buildroot/output`:

```bash
./build.sh buildroot

# Note: Make sure to compile the Buildroot root file system as a normal user to avoid unnecessary errors.
```

##### Compiling AMP

Compile RT-Thread to create amp image

###### Compile and package amp firmware

* Compile and package amp firmware

```bash
./build.sh amp
```

Use /path/to/sdk/device/rockchip/rk3506/amp_linux.its configuration file to package output/firmware/amp.img

###### Other

By default, hal or RT-Thread uses uart5 as the debugging serial port with a baud rate of 1.5M.

#### Packaging firmware

Package the firmware, and the generated complete firmware will be saved in the `output/update/` directory.

```bash
./build.sh updateimg
```

### Partition Description

#### parameter partition table

The parameter.txt file contains the partition information of the firmware. Take `parameter-buildroot-amp-emmc-fit.txt` as an example:

Path: `device/rockchip/rk3506/parameter-buildroot-amp-emmc-fit.txt`

```bash
FIRMWARE_VER:8.1
MACHINE_MODEL:RK3506
MACHINE_ID:007
MANUFACTURER: RK3506
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3506
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
GROW_ALIGN: 0
CMDLINE:mtdparts=:0x00002000@0x00002000(uboot),0x00000800@0x00004000(misc),0x0000A000@0x00004800(recovery),0x00010000@0x0000E800(boot),0x00010000@0x0001E800(backup),0x00008000@0x0002E800(oem),0x00002000@0x00036800(amp),0x00c00000@0x00038800(rootfs),-@0x00c36800(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
uuid:boot=7A3F0000-0000-446A-8000-702F00006273
```

The CMDLINE attribute is what we are concerned about. Taking uboot as an example, 0x00002000@0x00002000(uboot) is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

#### Package-file

The package-file file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt file.

Path: `output/firmware/package-file`

```bash
# NAME  PATH
package-file    package-file
parameter       parameter.txt
bootloader      MiniLoaderAll.bin
uboot   uboot.img
misc    misc.img
recovery        recovery.img
boot    boot.img
backup  RESERVED
oem     oem.img
amp     amp.img
rootfs  rootfs.img
userdata        userdata.img
```