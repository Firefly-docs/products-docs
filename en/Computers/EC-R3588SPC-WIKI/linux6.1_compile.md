# Compile Linux Firmware (kernel-6.1)

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
├── kernel -> kernel-6.1
├── kernel-6.1                                              # linux kernel
├── Makefile -> device/rockchip/common/Makefile
├── prebuilt_rootfs                                         # used to place pre-built rootfs
├── prebuilts                                               # used to place cross-build toolchain
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
├── tools                                                   # some tools
└── u-boot                                                  # uboot
```

### About Configurations

There are many config files (xxxx_defconfig) under `device/rockchip/rk3588/`, these files control the building system.

And their names will be like:
```
<vendor>_<chip>_<model>-<extra>_<OS>_defconfig

vendor: vendor of the product
chip: soc the product used
model: product model, means this config file is for that specific product.
extra: some extra features, could be empty
OS: the operating system that will be running in product
```

About config file contents, take firefly_rk3588_itx-3588j_debian_defconfig as an example:
```bash
RK_KERNEL_DTS_NAME="rk3588-firefly-itx-3588j"                    # set the dts to be used during kernel compilation
RK_KERNEL_CFG_FRAGMENTS="firefly-linux.config"                   # set the config fragments to be used during kernel compilation
RK_UBOOT_CFG_FRAGMENTS="firefly-linux"                           # set the config fragments to be used during uboot compilation
RK_USE_FIT_IMG=y
RK_BOOT_FIT_ITS_NAME="bootramdisk.its"
RK_PARAMETER="parameter-debian-fit.txt"                          # set the partition parameter
RK_PRODUCT_MODEL="ITX-3588J"                                     # product name/model
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk3588_debian_rootfs.img"   # set the prebuilt rootfs to be used
RK_MISC_RECOVERY=y
RK_RECOVERY_RAMDISK="rk3588-recovery-arm64.cpio.gz"
RK_RAMDISK_IMG="kernel/ramdisk.img"
USE_EXTBOOT=y                                                    # use exlinux style boot.img
RK_EXTRA_PARTITION_NUM=1
```

### Partitions

Parameter file records the firmware partitions, for example `device/rockchip/rk3588/parameter-xxxxxx-fit.txt`

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
GROW_ALIGN: 0
CMDLINE: mtdparts=:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00040000@0x00048000(recovery),0x00010000@0x00088000(backup),0x01c00000@0x00098000(rootfs),-@0x01c98000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
uuid:boot=7A3F0000-0000-446A-8000-702F00006273
```

CMDLINE is what we need to focus on, in "0x00002000@0x00004000(uboot)", 0x00004000 is the start position of uboot, and 0x00002000 is the size of uboot partition. Start position + Partition size = Start position of the next partition. All partitions follow the same rule.

And the unit of these hex numbers is "block", each block is 512 Byte, which means uboot partition size 0x00002000 is 8192 block = 8192 x 512 / 1024 / 1024 = 4 MiB
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

Download rootfs here [Ubuntu rootfs(64-bit) Kernel6.1](https://en.t-firefly.com/doc/download/301.html), please use rootfs under kernel-6.1 folder.

After download, decompress and move the rootfs image to SDK/prebuilt_rootfs/, then create a symbolic link.
```bash
# Decompress
7z x Ubuntu22.04-xxxx.7z

mkdir ./SDK/prebuilt_rootfs/
mv Ubuntu22.04-xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf Ubuntu22.04-xxxx.img rk3588_ubuntu_rootfs.img
cd ..
```


### Configurations

Run `./build.sh lunch` to list all available configs, enter a number to select. Please choose one with "ubuntu" keyword:
```bash
./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rk3588_linux6.1_release_20250114_v1.1.1c

Log colors: message notice warning error fatal

Log saved at /home2/daijh/p/rk3588/sdk_6.1/output/sessions/2025-01-14_14-39-08
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3588_aio-3588jd4_buildroot_defconfig
3. firefly_rk3588_aio-3588jd4_debian_defconfig
4. firefly_rk3588_aio-3588jd4_ubuntu_defconfig
5. firefly_rk3588_aio-3588l_debian_defconfig
6. firefly_rk3588_aio-3588l_ubuntu_defconfig
7. firefly_rk3588_aio-3588q_debian_defconfig
8. firefly_rk3588_aio-3588q_ubuntu_defconfig
9. firefly_rk3588_aio-3588sjd4_debian_defconfig
10. firefly_rk3588_aio-3588sjd4_ubuntu_defconfig
11. firefly_rk3588_itx-3588j_buildroot_defconfig
12. firefly_rk3588_itx-3588j_debian_defconfig
13. firefly_rk3588_itx-3588j_ubuntu_defconfig
14. firefly_rk3588_roc-rk3588-rt-ext-10ge_debian_defconfig
15. firefly_rk3588_roc-rk3588-rt-ext-10ge_ubuntu_defconfig
16. firefly_rk3588_roc-rk3588-rt-ext-2g5e_debian_defconfig
17. firefly_rk3588_roc-rk3588-rt-ext-2g5e_ubuntu_defconfig
18. firefly_rk3588_roc-rk3588-rt_debian_defconfig
19. firefly_rk3588_roc-rk3588-rt_ubuntu_defconfig
20. firefly_rk3588_roc-rk3588s-pc-ext_debian_defconfig
21. firefly_rk3588_roc-rk3588s-pc-ext_ubuntu_defconfig
22. firefly_rk3588_roc-rk3588s-pc_debian_defconfig
23. firefly_rk3588_roc-rk3588s-pc_ubuntu_defconfig
24. rockchip_rk3588_evb1_lp4_v10_defconfig
25. rockchip_rk3588_evb7_v11_defconfig
26. rockchip_rk3588_ipc_evb1_v10_defconfig
27. rockchip_rk3588_ipc_evb7_lp4_v11_defconfig
28. rockchip_rk3588_multi_ipc_evb1_v10_defconfig
29. rockchip_rk3588s_evb1_lp4x_v10_defconfig
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

