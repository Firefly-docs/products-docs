# Compile Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware. It is recommended to develop in the Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

## Preparatory work

### Build a compilation environment

```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler python-pip ncurses-dev python-pyelftools
```

### Download Firefly_Linux_SDK

* Method One

Download via repo:

```bash
mkdir ~/proj/rk356x_linux_release_20211019/
cd ~/proj/rk356x_linux_release_20211019/
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_release.xml
```

* Method Two

Download Firefly_Linux_SDK sub-volume compressed package: [Firefly_Linux_SDK Source]()

After downloading, verify the MD5 code:

```bash
$ md5sum rk356x_linux_release_v1.2.0_20211019_split_dir/*firefly_split*
b633414d69240faa0c3bd755c255ede8  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file0
5cb1e2b63bbb4e3595c731038f6723fd  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file1
e06c0f29fd5a870e0942139a1877a470  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file2
4e14fbf72ccbb9b87f81d83a256205b0  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file3
be8b003703b51e7220e52bb36439a357  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file4
```

After confirming that it is correct, you can unzip:

<font color=red>**Attention: To avoid unnecessary errors, please do not unzip the SDK in shared folders, mounted folders or non-english directories.**</font>

```bash
mkdir ~/proj/
cd ~/proj/
cat path/to/rk356x_linux_release_v1.2.0_20211019_split_dir/*firefly_split* | tar -xzv
```

### Sync Code

Execute the following command to synchronize the code:

```bash
# Enter the SDK root directory
cd ~/proj/rk356x_linux_release_20211019/

# Sync
.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

You can use the following command to update the SDK later:

```bash
.repo/repo/repo sync -c --no-tags
```
### Directory Structure

```bash
$ tree -L 1
.
├── app
├── buildroot # Buildroot root filesystem build directory
├── build.sh -> device/rockchip/common/build.sh # Compile script
├── debian # Debian root filesystem compilation directory
├── device # Compile related configuration files
├── distro
├── docs # Documentation
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel
├── Makefile -> buildroot/build/Makefile
├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh # Link script
├── prebuilts # Cross compilation toolchain
├── rkbin
├── rkflash.sh -> device/rockchip/common/rkflash.sh # Flash script
├── tools # Tools directory
├── u-boot
└── yocto
```

## Compile SDK

### Configuration before compilation

In the `device/rockchip/rk356x/` directory, there are configuration files of different board types, select the configuration file:

```bash
./build.sh 
```

The configuration file will be linked to `device/rockchip/.BoardConfig.mk`, check the file to verify whether the configuration is successful.

Related configuration introduction:

```bash
# Target arch
export RK_ARCH=arm64 # 64-bit ARM architecture
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=rv1126b_defconfig # u-boot configuration file
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=rv1126b_defconfig # kernel configuration file
# Kernel dts
export RK_KERNEL_DTS=rv1126b-firefly-aio-1126bq38.dts # dts file
# Buildroot config
export RK_CFG_BUILDROOT= # Buildroot configuration
# Recovery config
export RK_CFG_RECOVERY=rockchip_rk356x_recovery # recovery configuration
# parameter for GPT table
export RK_PARAMETER=parameter-buildroot-fit.txt # Partition table
# rootfs image path
export RK_ROOTFS_IMG=rockdev/rootfs.${RK_ROOTFS_TYPE} # root filesystem path
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

* Compile Buildroot root filesystem

Compiling the Buildroot root filesystem will generate a compilation output directory in `buildroot/output`:

```bash
./build.sh buildroot

# Note: Make sure to compile the Buildroot root filesystem as a normal user to avoid unnecessary errors.
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

The parameter.txt file contains the partition information of the firmware. Take `parameter-buildroot-fit.txt` as an example:

Path: `device/rockchip/rk356x/parameter-buildroot-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3588
MACHINE_ID: 007
MANUFACTURER: RK3588
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

### package-file

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