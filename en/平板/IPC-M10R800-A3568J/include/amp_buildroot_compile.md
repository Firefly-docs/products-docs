## Compile Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware. It is recommended to develop in the Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

### Compile SDK

#### Configuration before compilation

In the `device/rockchip/rk356x/` directory, there are configuration files of different board types.

Return to SDK root directory to select the configuration file:

```bash
./build.sh 
```

#### SDK unified compilation and packaging

The RK3568 supports Linux, HAL, and RT-Thread's AMP hybrid architecture design, allowing different cpus to run different systems to meet flexible product design needs. The SDK currently uses the mixed structure model of Linux + RT-Thread by default, in which the CPU running Linux is the primary core, and the CPU running rtt is the secondary core. Cores 0 to 2 run Linux, cores 3 run RT-Thread.

#### Compile configuration

The SDK unified compilation configuration script is located in the device/rockchip/rk356x/ directory. The compilation configuration script includes U-Boot, Kernel, HAL, and RT-Thread configurations, as well as AMP-related CPU allocation and memory allocation configurations. You can add or modify configuration script files to meet your compilation requirements. The main AMP configuration files are as follows:

```bash
rk3568_amp_linux.its                    # linux+hal's AMP packs ITS configuration files
rk3568_amp_linux_rtt.its                # linux+rtt's AMP packs ITS configuration files
firefly-rk3568-amp.mk                   # The AMP configuration file corresponding to linux+hal is also the AMP base configuration file
firefly-rk3568-linux-rtt.mk             # The AMP configuration file for linux+rtt
                 # AIO-3568Jcorresponds to the board-level configuration script file
```

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

Notice: The kernel device tree is located in the directory kernel/arch/arm64/boot/dts/rockchip/. The device tree  used by it is only compatible with 2GB of memory.
If you need to apply it to devices with other memory specifications, you will need to modify its memory description label memory yourself.

**The hardware resources used by AMP need to be disabled in the kernel device tree and declared under the rockchip_amp label in the rk3568-amp.dtsi device tree.**

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

##### Compile AMP

Compile hal or rt-thread to make amp images

###### Use hal

* Compile hal
```bash
./build.sh hal 3
```
The generated image is hal/project/rk3568/GCC/hal3.bin

###### Use rt-thread

* Compile rt-thread

```bash
./build.sh rtthread 3
```
The generated image is rt-thread/bsp/rockchip/rk3568-32/Image/rtt3.bin

###### Pack the AMP firmware

* Pack the AMP images
```bash
./build.sh ampimg
```
Use the rockdev/amp/amp.its configuration file to package rockdev/amp/amp.img

###### Others

hal or rt-thread uses uart4 as the debug serial port by default with a baud rate of 115200.

* For more details, please refer to the amp documentation in SDK/docs/amp/manuals/rk3568.

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

The parameter.txt file contains the partition information of the firmware. Take `parameter-amp-buildroot-fit.txt` as an example:

Path: `device/rockchip/rk356x/parameter-amp-buildroot-fit.txt`

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

Path: `tools/linux/Linux_Pack_Firmware/rockdev/rk356x-amp-package-file`

```bash
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
uboot           Image/uboot.img
misc            Image/misc.img
amp             Image/amp.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
oem             Image/oem.img
userdata        Image/userdata.img
backup          RESERVED
```