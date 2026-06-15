## Compile Ubuntu firmware

This chapter introduces the compilation process of Ubuntu firmware. It is recommended to develop under Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

### A brief introduction to Ubuntu firmware
[What is Ubuntu Minimal firmware?](ubuntu_minimal_support.html)

[What is Ubuntu Desktop firmware?](ubuntu_desktop_support.html)

### Build Linux-SDK

#### Precompile Configuration

There are configuration files for different board in `device/rockchip/rk356x/`.

Return to SDK root directory to select the configuration file:

```bash
./build.sh 
```

#### Download Ubuntu filesystem

* Download: [Ubuntu rootfs(64-bit)]()，put in SDK path

* Unzip

```bash
7z x ubuntu-aarch64-rootfs.7z
```

* Move filesystem to `ubuntu_rootfs/`

```bash
mkdir ubuntu_rootfs
mv ubuntu-aarch64-rootfs.img ubuntu_rootfs/rk356x_ubuntu_rootfs.img
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

The automatic compilation will perform all compilation and packaging operations to generate RK firmware.

```bash
./build.sh
```

#### Partial compilation

* u-boot

```bash
./build.sh uboot
```

* kernel

Notice: The kernel device tree is located in the directory kernel/arch/arm64/boot/dts/rockchip/. The device tree  used by it is only compatible with 2GB of memory.
If you need to apply it to devices with other memory specifications, you will need to modify its memory description label memory yourself.

**The hardware resources used by AMP need to be disabled in the kernel device tree and declared under the rockchip_amp label in the rk3568-amp.dtsi device tree.**

```bash
./build.sh kernel
```

* recovery

```bash
./build.sh recovery
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

#### Pack the firmware

* Update links

Update each part of the `.img` link to the directory `rockdev/`:

```bash
./mkfirmware.sh
```

* Pack the firmware

Pack the firmware, the firmware will be saved to the directory `rockdev/pack/`.

```bash
# Pack RK firmware
./build.sh updateimg
```

### Partition table

#### parameter

The parameter.txt file contains the partition information of the firmware. Take parameter-amp-ubuntu-fit.txt as an example:

path: `device/rockchip/rk356x/parameter-amp-ubuntu-fit.txt`

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
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00001000@0x00006000(misc),0x00001000@0x00007000(amp),0x00040000@0x00008000(boot:bootable),0x00020000@0x00048000(recovery),0x00010000@0x00068000(backup),0x49dc00@0x00078000(rootfs),-@0x515c00(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is where we are concerned. Take uboot as an example. In 0x00002000@0x00004000(uboot), 0x00004000 is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

#### package-file

The package-file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt.

path: `tools/linux/Linux_Pack_Firmware/rockdev/rk356x-amp-ubuntu-package-file`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
uboot           Image/uboot.img
misc            Image/misc.img
amp             Image/amp.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
userdata        RESERVED
backup          RESERVED
```