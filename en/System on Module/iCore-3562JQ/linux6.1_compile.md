# Compile Linux Firmware (kernel-6.1)

## Download the SDK

Please contact sales@t-firefly.com to get SDK download link.

<font color=red>

**Notice:**

**1. SDK use cross-compilation, so use SDK in x86_64 PC, do not download SDK to the device**

**2. We suggest to use Ubuntu20.04 (real PC or docker) to build, other OS may cause building failure**

**3. Do not place or decompress the SDK archive in Virtual Machine share folder or non-english folder**

**4. Please use the regular user to get/compile the SDK, use root privilege may cause building failure**
</font>

Prepare a empty folder to place the SDK, we suggest a folder under your home dir. For example `~/proj`

### Verify and Decompress

Verify the MD5 of SDK archives after download.
```bash
$ md5sum rk3562_6.1.tgz.split*
```

Compare the results with the md5sum.txt, if MD5s are the same, you can decompress now:
```bash
# decompress
cd ~/proj/
cat rk3562_6.1.tgz.split0* | tar -zx

# you will get
~/proj/rk3562_6.1

# install python2
sudo apt update && sudo apt install python2

# check out files from repo
cd ~/proj/rk3562_6.1
python2 .repo/repo/repo sync -l --no-tags
```
## SDK Introduction

### SDK Structure

```bash
$ tree -L 1
.
├── app
├── buildroot                                               # buildroot
├── build.sh -> device/rockchip/common/scripts/build.sh     # building script
├── device                                                  # building system, includes building configs
├── docs                                                    # development documents
├── external                                                # some components
├── kernel                                                  # linux kernel
├── Makefile -> device/rockchip/common/Makefile
├── prebuilt_rootfs                                         # used to place pre-built rootfs
├── prebuilts                                               # used to place cross-build toolchain
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
├── tools                                                   # some tools
└── u-boot                                                  # uboot
```

### About Configurations

There are many config files (xxxx_defconfig) under `device/rockchip/rk3562/`, these files control the building system.

And their names will be like:
```
<vendor>_<chip>_<model>-<extra>_<OS>_defconfig

vendor: vendor of the product
chip: soc the product used
model: product model, means this config file is for that specific product.
extra: some extra features, could be empty
OS: the operating system that will be running in product
```

About config file contents, take firefly_rk3562_aio-3562jq_debian_defconfig as an example:
```bash
RK_WIFIBT_CHIP="AP6256"
RK_UBOOT_SPL=y
RK_KERNEL_DTS_NAME="rk3562-firefly-aio-3562jq"                   # set the dts to be used during kernel compilation
RK_KERNEL_CFG_FRAGMENTS="firefly_linux.config"                   # set the config fragments to be used during kernel compilation
RK_UBOOT_CFG_FRAGMENTS="firefly-linux"                           # set the config fragments to be used during uboot compilation
RK_PARAMETER="parameter-debian-fit.txt"                          # set the partition parameter
RK_USE_FIT_IMG=y
RK_RAMDISK_IMG="kernel/ramdisk.img"
USE_EXTBOOT=y                                                    # use exlinux style boot.img
RK_RECOVERY_RAMDISK="rk3562-recovery-arm64.cpio.gz"
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk3562_debian_rootfs.img"   # set the prebuilt rootfs to be used
RK_MISC_RECOVERY=y
RK_EXTRA_PARTITION_NUM=1
```

### Partitions

Parameter file records the firmware partitions, for example `device/rockchip/rk3562/parameter-xxxxxx-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3562
MACHINE_ID: 007
MANUFACTURER: RK3562
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
GROW_ALIGN: 0
CMDLINE: mtdparts=:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00040000@0x00048000(recovery),0x00010000@0x00088000(backup),0x00c00000@0x00098000(rootfs),-@0x00c98000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE is what we need to focus on, in "0x00002000@0x00004000(uboot)", 0x00004000 is the start position of uboot, and 0x00002000 is the size of uboot partition. Start position + Partition size = Start position of the next partition. All partitions follow the same rule.

And the unit of these hex numbers is "block", each block is 512 Byte, which means uboot partition size 0x00002000 is 8192 block = 8192 x 512 / 1024 / 1024 = 4 MiB
## Build Debian Firmware

### Preparations

Run these two commands to install necessary tools
```bash
sudo apt update

sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev p7zip-full
```

Download rootfs here [Debian Rootfs](https://en.t-firefly.com/doc/download/219.html#other_739), please use rootfs under kernel-6.1 folder.

After download, decompress and move the rootfs image to SDK/prebuilt_rootfs/, then create a symbolic link.
```bash
# Decompress
7z x debian12_xxxx_rootfs_xxxx.7z

# Move the rootfs image to sdk and then create a symbolic link.
mv debian12_xxxx_rootfs_xxxx.img ./rk3562_6.1/prebuilt_rootfs/
cd ./rk3562_6.1/prebuilt_rootfs/
ln -sf debian12_xxxx_rtoofs_xxxx.img rk3562_debian_rootfs.img
cd ..
```

### Configurations

Run `./build.sh lunch` to list all available configs, enter a number to select. Please choose one with "debian" keyword:
```bash
./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rk3562_linux6.1_release_20241220_v1.0.0a.xml

Log colors: message notice warning error fatal

Log saved at /home4/lth/project/linux/rk3562_6.1/output/sessions/2024-12-25_18-03-39
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3562_aio-3562jq_buildroot_defconfig
3. firefly_rk3562_aio-3562jq_debian_defconfig
4. firefly_rk3562_aio-3562jq_ubuntu_defconfig
5. rockchip_rk3562_dictpen_test3_v20_defconfig
6. rockchp_rk3562_evb1_lp4x_v10_amp_defconfig
7. rockchip_rk3562_evb1_lp4x_v10_defconfig
8. rockchip_rk3562_evb2_ddr4_v10_amp_defconfig
9. rockchip_rk3562_evb2_ddr4_v10_defconfig
10. rockchip_rk3562_robot_evb1_lp4x_v10_defconfig
11. rockchip_rk3562_robot_evb2_ddr4_v10_defconfig
Which would you like? [1]:
```

### Compilation

#### Full Compilation

Run
```bash
./build.sh all
```

The output firmware will be `output/update/update.img`

#### Partial Compilation

After a full compilation, if you modified kernel or uboot, you can do partial compilation to save time.

* Only build u-boot, will generate u-boot/uboot.img

```bash
./build.sh uboot
```

* Only build kernel, will generate kernel/extboot.img

```bash
./build.sh extboot
```

* Pack all parts to update.img

```bash
./build.sh updateimg
```
## Build Ubuntu Firmware

### Preparations

Run these two commands to install necessary tools
```bash
sudo apt update

sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev p7zip-full
```

Download rootfs here [Ubuntu Rootfs](https://en.t-firefly.com/doc/download/222.html#other_669), please use rootfs under kernel-6.1 folder.

After download, decompress and move the rootfs image to SDK/prebuilt_rootfs/, then create a symbolic link.
```bash
# Decompress
7z x Ubuntu22.04-Xfce_xxxx_xxxx.7z

# Move the rootfs image to sdk and then create a symbolic link.
mv Ubuntu22.04-Xfce_xxxx_xxxx.img ./rk3562_6.1/prebuilt_rootfs/
cd ./rk3562_6.1/prebuilt_rootfs/
ln -sf Ubuntu22.04-Xfce_xxxx_xxxx.img rk3562_ubuntu_rootfs.img
cd ..
```

### Configurations

Run `./build.sh lunch` to list all available configs, enter a number to select. Please choose one with "ubuntu" keyword:
```bash
./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rk3562_linux6.1_release_20241220_v1.0.0a.xml

Log colors: message notice warning error fatal

Log saved at /home4/lth/project/linux/rk3562_6.1/output/sessions/2024-12-25_18-03-39
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3562_aio-3562jq_buildroot_defconfig
3. firefly_rk3562_aio-3562jq_debian_defconfig
4. firefly_rk3562_aio-3562jq_ubuntu_defconfig
5. rockchip_rk3562_dictpen_test3_v20_defconfig
6. rockchp_rk3562_evb1_lp4x_v10_amp_defconfig
7. rockchip_rk3562_evb1_lp4x_v10_defconfig
8. rockchip_rk3562_evb2_ddr4_v10_amp_defconfig
9. rockchip_rk3562_evb2_ddr4_v10_defconfig
10. rockchip_rk3562_robot_evb1_lp4x_v10_defconfig
11. rockchip_rk3562_robot_evb2_ddr4_v10_defconfig
Which would you like? [1]:
```

### Compilation

#### Full Compilation

Run
```bash
./build.sh all
```

The output firmware will be `output/update/update.img`

#### Partial Compilation

After a full compilation, if you modified kernel or uboot, you can do partial compilation to save time.

* Only build u-boot, will generate u-boot/uboot.img

```bash
./build.sh uboot
```

* Only build kernel, will generate kernel/extboot.img

```bash
./build.sh extboot
```

* Pack all parts to update.img

```bash
./build.sh updateimg
```
## Build Buildroot Firmware

### Preparations

Run these two commands to install necessary tools
```bash
sudo apt update

sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev
```

### Configurations

Run `./build.sh lunch` to list all available configs, enter a number to select. Please choose one with "buildroot" keyword:
```bash
./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rk3562_linux6.1_release_20241220_v1.0.0a.xml

Log colors: message notice warning error fatal

Log saved at /home4/lth/project/linux/rk3562_6.1/output/sessions/2024-12-25_18-03-39
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3562_aio-3562jq_buildroot_defconfig
3. firefly_rk3562_aio-3562jq_debian_defconfig
4. firefly_rk3562_aio-3562jq_ubuntu_defconfig
5. rockchip_rk3562_dictpen_test3_v20_defconfig
6. rockchp_rk3562_evb1_lp4x_v10_amp_defconfig
7. rockchip_rk3562_evb1_lp4x_v10_defconfig
8. rockchip_rk3562_evb2_ddr4_v10_amp_defconfig
9. rockchip_rk3562_evb2_ddr4_v10_defconfig
10. rockchip_rk3562_robot_evb1_lp4x_v10_defconfig
11. rockchip_rk3562_robot_evb2_ddr4_v10_defconfig
Which would you like? [1]:
```

### Compilation

#### Full Compilation

Run
```bash
./build.sh all
```

The output firmware will be `output/update/update.img`

#### Partial Compilation

After a full compilation, if you modified kernel or uboot, you can do partial compilation to save time.

* Only build u-boot, will generate u-boot/uboot.img

```bash
./build.sh uboot
```

* Only build kernel, will generate kernel/extboot.img

```bash
./build.sh extboot
```

* Pack all parts to update.img

```bash
./build.sh updateimg
```

But if you modified buildroot, you need to do full compilation.