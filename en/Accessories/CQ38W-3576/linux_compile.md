# Compile Linux firmware

## Download the SDK

Please contact sales@t-firefly.com to get SDK download link and read the readme file

<font color=red>

**Notice:**

**1. SDK use cross-compilation, so use SDK in x86_64 PC, do not download SDK to the device**

**2. We suggest to use Ubuntu20.04 (real PC or docker) to build, other OS may cause building failure**

**3. Do not place or decompress the SDK archive in Virtual Machine share folder or non-english folder**

**4. Please use the regular user to get/compile the SDK, use root privilege may cause building failure**
</font>## SDK Introduction

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

There are many config files (xxxx_defconfig) under `device/rockchip/rk3576/`, these files control the building system.

And their names will be like:
```
<vendor>_<chip>_<model>-<extra>_<OS>_defconfig

vendor: vendor of the product
chip: soc the product used
model: product model, means this config file is for that specific product.
extra: some extra features, could be empty
OS: the operating system that will be running in product
```

About config file contents, take firefly_rk3576_aio-3576jd4_debian_defconfig as an example:

```bash
#include "firefly.config"                                             # includes firefly.config
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk3576_debian_rootfs.img"        # set the prebuilt rootfs to be used
RK_PARAMETER="parameter-ubuntu-fit.txt"                               # set the partition parameter
RK_KERNEL_DTS_NAME="rk3576-firefly-aio-3576jd4"                       # set the dts to be used during kernel compilation
RK_PRODUCT_MODEL="AIO-3576JD4"                                        # product name/model
UBOOT_DEBUG_BAUDRATE="115200"                                         # set debug serial baudrate as 115200
```

And firefly.config mainly has:

```bash
RK_KERNEL_CFG_FRAGMENTS="firefly-linux.config firefly-docker.config"  # set the config fragments to be used during kernel compilation
RK_UBOOT_CFG_FRAGMENTS="firefly-linux"                                # set the config fragments to be used during uboot compilation
USE_EXTBOOT=y                                                         # use exlinux style boot.img
```

### Partitions

Parameter file records the firmware partitions, for example `device/rockchip/rk3576/parameter-xxxxxx-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK357
MACHINE_ID: 007
MANUFACTURER: RK3576
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
GROW_ALIGN: 0
CMDLINE: mtdparts=:0x00004000@0x00004000(uboot),0x00002000@0x00008000(misc),0x00040000@0x0000a000(boot:bootable),0x00040000@0x0004a000(recovery),0x00010000@0x0008a000(backup),0x00c00000@0x0009a000(rootfs),-@0x00c9a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
uuid:boot=7A3F0000-0000-446A-8000-702F000062736
```

CMDLINE is what we need to focus on, in "0x00040000@0x0000a000(boot:bootable)", 0x0000a000 is the start position of boot, and 0x00040000 is the size of boot partition. Start position + Partition size = Start position of the next partition. All partitions follow the same rule.

And the unit of these hex numbers is "block", each block is 512 Byte, which means boot partition size 0x00040000 is 262144 block = 262144 x 512 / 1024 / 1024 = 128 MiB
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

Download rootfs here [Ubuntu rootfs](https://www.t-firefly.com/doc/download/327.html#other_791), in general, choose the filesystem with latest date.

After download, decompress and move the rootfs image to SDK/prebuilt_rootfs/, then create a symbolic link.
```bash
# Decompress
7z x Ubuntu22.04-xxxx.7z

mkdir ./SDK/prebuilt_rootfs/
mv Ubuntu22.04-xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf Ubuntu22.04-xxxx.img rk3576_ubuntu_rootfs.img
cd ..
```

### Configurations

Run `./build.sh lunch` to list all available configs, enter a number to select. Please choose one with "ubuntu" keyword:
```bash
./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rk3576_linux_release_20250520_v1.0.1f.xml

Log colors: message notice warning error fatal

Log saved at /home2/liuth/project/linux/rk3576/output/sessions/2025-08-06_15-27-20
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3576_aio-3576c_buildroot_defconfig
3. firefly_rk3576_aio-3576c_debian_defconfig
4. firefly_rk3576_aio-3576c_ubuntu_defconfig
5. firefly_rk3576_aio-3576jd4_buildroot_defconfig
6. firefly_rk3576_aio-3576jd4_debian_defconfig
7. firefly_rk3576_aio-3576jd4_ubuntu_defconfig
......
......
24. firefly_rk3576_roc-rk3576-pc_buildroot_defconfig
25. firefly_rk3576_roc-rk3576-pc_debian_defconfig
26. firefly_rk3576_roc-rk3576-pc_ubuntu_defconfig
27. rockchip_rk3576_evb1_v10_defconfig
28. rockchip_rk3576_industry_evb_v10_defconfig
29. rockchip_rk3576_iotest_v10_defconfig
30. rockchip_rk3576_ipc_evb1_v10_defconfig
31. rockchip_rk3576_multi_ipc_evb1_v10_defconfig
32. rockchip_rk3576_test1_v10_defconfig
33. rockchip_rk3576_test2_v10_defconfig
34. rockchip_rk3576_vehicle_evb_v10_defconfig
35. rockchip_rk3576_vehicle_evb_v20_mos_defconfig
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

Download rootfs here [Debian rootfs](https://www.t-firefly.com/doc/download/327.html#other_791), in general, choose the filesystem with latest date.

After download, decompress and move the rootfs image to SDK/prebuilt_rootfs/, then create a symbolic link.
```bash
# Decompress
7z x Debian12-xxxx.7z

mkdir ./SDK/prebuilt_rootfs/
mv Debian12-xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf Debian12-xxxx.img rk3576_debian_rootfs.img
cd ..
```

### Configurations

Run `./build.sh lunch` to list all available configs, enter a number to select. Please choose one with "debian" keyword:
```bash
./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rk3576_linux_release_20250520_v1.0.1f.xml

Log colors: message notice warning error fatal

Log saved at /home2/liuth/project/linux/rk3576/output/sessions/2025-08-06_15-27-20
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3576_aio-3576c_buildroot_defconfig
3. firefly_rk3576_aio-3576c_debian_defconfig
4. firefly_rk3576_aio-3576c_ubuntu_defconfig
5. firefly_rk3576_aio-3576jd4_buildroot_defconfig
6. firefly_rk3576_aio-3576jd4_debian_defconfig
7. firefly_rk3576_aio-3576jd4_ubuntu_defconfig
......
......
24. firefly_rk3576_roc-rk3576-pc_buildroot_defconfig
25. firefly_rk3576_roc-rk3576-pc_debian_defconfig
26. firefly_rk3576_roc-rk3576-pc_ubuntu_defconfig
27. rockchip_rk3576_evb1_v10_defconfig
28. rockchip_rk3576_industry_evb_v10_defconfig
29. rockchip_rk3576_iotest_v10_defconfig
30. rockchip_rk3576_ipc_evb1_v10_defconfig
31. rockchip_rk3576_multi_ipc_evb1_v10_defconfig
32. rockchip_rk3576_test1_v10_defconfig
33. rockchip_rk3576_test2_v10_defconfig
34. rockchip_rk3576_vehicle_evb_v10_defconfig
35. rockchip_rk3576_vehicle_evb_v20_mos_defconfig
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
