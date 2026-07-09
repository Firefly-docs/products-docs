# Compile the Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware. It is recommended to develop in the Ubuntu 16.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

### Download Firefly_Linux_SDK sub-volume compressed package

**Since the Firefly_Linux_SDK source code package is relatively large, some users' computers do not support files above 4G or the network transmission of a single file is slow, so we use the method of sub-volume compression to package the SDK. Users can obtain the Firefly_Linux_SDK source code package in the following ways:**[Firefly_Linux_SDK Source]

After downloading, follow the README document:

```
└── rk3399_linux_release_xxx
    ├── linux_sdk_tar
    │   ├── rk3399_linux_release_xxx.sdk.split00
    │   ├── rk3399_linux_release_xxx.sdk.split01
    ├── md5sum.txt
    ├── README_EN.txt
    ├── README_ZH.txt
    └── sdk_tools.sh

```

### Update Firefly_Linux_SDK

You can use the following command to update the SDK later
```
.repo/repo/repo sync -c --no-tags
```

[Firefly_Linux_SDK Source]: http://en.t-firefly.com/doc/download/52.html#other_144
### Directory Structure

```bash
$ tree -L 1
.
├── app
├── buildroot # Buildroot root file system build directory
├── build.sh -> device/rockchip/common/build.sh # compile script
├── debian # Debian root file system compilation directory
├── device # Compile related configuration files
├── distro
├── docs # Documentation
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel
├── Makefile -> buildroot/build/Makefile
├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh # link script
├── prebuilts # Cross compilation tool chain
├── rkbin
├── rkflash.sh -> device/rockchip/common/rkflash.sh # Flash script
├── tools # Tools directory
├── u-boot
└── yocto
```

### Build a compilation environment

```bash
sudo apt-get install expect-dev repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler \
gcc-aarch64-linux-gnu mtools parted libudev-dev libusb-1.0-0-dev python-linaro-image-tools \
linaro-image-tools autoconf autotools-dev libsigsegv2 m4 intltool libdrm-dev curl sed make \
binutils build-essential gcc g++ bash patch gzip bzip2 perl tar cpio python unzip rsync file bc wget \
libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev libglade2-dev cvs git mercurial rsync openssh-client \
subversion asciidoc w3m dblatex graphviz python-matplotlib libc6:i386 libssl-dev texinfo \
liblz4-tool genext2fs lib32stdc++6 expect
```

## Compile SDK

### Configuration before compilation

In the `device/rockchip/rk3399/` directory, there are configuration files of different board types, select the configuration file:

```bash
./build.sh aio-3399c-buildroot.mk
```

The configuration file will be linked to `device/rockchip/.BoardConfig.mk`, check the file to verify whether the configuration is successful.

Related configuration introduction:

```bash
# Target arch
export RK_ARCH=arm64 # 64-bit ARM architecture
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly-rk3399 # u-boot configuration file
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly_linux_defconfig # kernel configuration file
# Kernel dts
export RK_KERNEL_DTS=rk3399-firefly-aioc # dts file
# Buildroot config
export RK_CFG_BUILDROOT=rockchip_rk3399 # Buildroot configuration
# Recovery config
export RK_CFG_RECOVERY=rockchip_rk3399_recovery # recovery configuration
# parameter for GPT table
export RK_PARAMETER=parameter-buildroot.txt # Partition table
# rootfs image path
export RK_ROOTFS_IMG=buildroot/output/$RK_CFG_BUILDROOT/images/rootfs.$RK_ROOTFS_TYPE # root file system path
```

### Partial compilation

* Compile u-boot

```bash
./build.sh uboot
```

* Compile kernel

```bash
./build.sh kernel
```

* Compile recovery

```bash
./build.sh recovery
```

* Compile Buildroot root file system

Compiling the Buildroot root file system will generate a compilation output directory in `buildroot/output`:

```bash
./build.sh buildroot

# Note: Make sure to compile the Buildroot root file system as a normal user to avoid unnecessary errors.
```

### Package the firmware

Update each part of the mirror link to the `rockdev/` directory:

```bash
./mkfirmware.sh
```

Pack the firmware, the generated complete firmware will be saved to the `rockdev/pack/` directory.

```bash
./build.sh updateimg
```

### Automatic compilation

Fully automatic compilation will perform the above compilation and packaging operations to generate complete firmware.

```bash
./build.sh
```

## Partition description

### parameter partition table

The parameter.txt file contains the partition information of the firmware. Take `parameter.txt` as an example:

Path: `device/rockchip/rk3288/parameter.txt`

```bash
FIRMWARE_VER: 8.1
MACHINE_MODEL: RK3399
MACHINE_ID: 007
MANUFACTURER: RK3399
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3399
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(trust),0x00002000@0x00008000(misc),0x00010000@0x0000a000(boot),0x00010000@0x0001a000(recovery),0x00010000x@0x0000@2a000) 0x0003a000(oem),0x00c00000@0x0005a000(rootfs),-@0x00c5a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is our focus. Taking uboot as an example, 0x00004000 in 0x00002000@0x00004000(uboot) is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

### package-file

The package-file file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt file.

Path: `tools/linux/Linux_Pack_Firmware/rockdev/rk3399-package-file`

```bash
# NAME Relative path
#
#HWDEF HWDEF
package-file package-file
bootloader Image/MiniLoaderAll.bin
parameter Image/parameter.txt
trust Image/trust.img
uboot Image/uboot.img
misc Image/misc.img
#resource Image/resource.img
#kernel Image/kernel.img
boot Image/boot.img
recovery Image/recovery.img
rootfs Image/rootfs.img
oem Image/oem.img
userdata:grow Image/userdata.img
backup RESERVED
#update-script update-script
#recover-script recover-script
```