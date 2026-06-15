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

The configuration file will be connected to `device/rockchip/.BoardConfig.mk`, check the file to verify whether the configuration is successful.

Configruation content:

```bash
# Target arch
export RK_ARCH=arm64 # 64-bit ARM
# Uboot defconfig
export RK_UBOOT_DEFCONFIG= # u-boot configuration
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly_linux_defconfig # kernel configuration
# Kernel dts
export RK_KERNEL_DTS= # dts file
# parameter for GPT table
export RK_PARAMETER=parameter-ubuntu-fit.txt # partition table
# rootfs image path
export RK_ROOTFS_IMG=ubuntu_rootfs/rk356x_ubuntu_rootfs.img # filesystem path
```
#### Select Accessories to Compile

Under `device/rockchip/rk356x/` , in addition to , there are other configuration files that come with different accessories

The name of the configuration file indicates the screen and camera used.
If the screen is not specified, the default HDMI display is used.
If the camera is not specified, the default single camera is used

```bash
# For example:
xxxx-ubuntu.mk                      # Use HDMI + single-camera
xxxx-2cam-ubuntu.mk                 # Use HDMI + dual-camera
xxxx-mipi-ubuntu.mk                 # Use mipi + single-camera
xxxx-mipi-2cam-ubuntu.mk            # Use mipi + dual-camera
```

Chose the configuration file you like, execute `build.sh` to make it effective:

```bash
# For example:
./build.sh xxxx-mipi-2cam-ubuntu.mk
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

#### Automatic compilation

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

Notice：Firefly kernel does not come with all kernel features, need extra kernel features please refer to [Kernel](kernel_introduction.md)

```bash
./build.sh kernel
```
Notice：**Linux SDK v1.2.4a** and later using extboot, please use `./build.sh extboot` to build kernel

The output file is `SDK/kernel/extboot.img`, use it instead of boot.img

How to check SDK version:
1. The version format is vx.x.xx, eg: v1.2.4a
1. Firmware filename has SDK version(..._vx.x.xx_date.img)
1. Buildroot use`cat /etc/version`to get version(rk356x_linux_release_date_vx.x.xx.xml)
1. Ubuntu use`ffgo version`to get(rk356x_linux_release_date_vx.x.xx.xml)
1. In SDK check the link:`ls -l .repo/manifests/rk356x_linux_release.xml`
1. If you can't get version by methods above, that means you are using old version, no support for extboot

**Do not burn extboot.img into old version firmware!**

Besides, extboot ubuntu support update kernel by deb package, please see [Ubuntu Manual](https://wiki.t-firefly.com/en/Firefly-Linux-Guide/manual_ubuntu.html#linux-headers-and-linux-image)

* recovery

```bash
./build.sh recovery
```

#### Update links

Update each part of the `.img` link to the directory `rockdev/`:

```bash
./mkfirmware.sh
```

#### Pack the firmware

Pack the firmware, the firmware will be saved to the directory `rockdev/pack/`.

##### RK firmware

RK firmware is the firmware packaged in Rockchip's proprietary format, and can be flashed to eMMC or SD card with the tools provided by Rockchip (**Note**: If there is no special instruction, the firmware mentioned on WIKI defaults to RK firmware) .

```bash
# Pack RK firmware
./build.sh updateimg
```

##### RAW firmware

RAW firmware is a kind of firmware that can be flashed to the storage device in a bit-by-bit copy mode, and is the original image of the storage device. Different from the RK firmware, currently it only supports flashing to SD card to boot with Etcher tool.

[Etcher official download link](https://www.balena.io/etcher/)

```bash
# Package RAW firmware
./build.sh rawimg
```

### Partition table

#### parameter

The parameter.txt file contains the partition information of the firmware. Take parameter-ubuntu-fit.txt as an example:

path: `device/rockchip/rk356x/parameter-ubuntu-fit.txt`

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
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00010000@0x00008000(boot),0x00010000@0x00018000(recovery),0x00010000@0x00028000(backup),0x00c00000@0x00038000(rootfs),-@0x00c38000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is where we are concerned. Take uboot as an example. In 0x00002000@0x00004000(uboot), 0x00004000 is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

#### package-file

The package-file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt.

path: `tools/linux/Linux_Pack_Firmware/rockdev/rk356x-ubuntu-package-file`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
uboot           Image/uboot.img
misc            Image/misc.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
userdata        RESERVED
backup          RESERVED
```