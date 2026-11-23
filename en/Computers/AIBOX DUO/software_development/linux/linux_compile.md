# Compile Linux Firmware

<CodeBlockTabs defaultValue="k612">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="k612">Kernel 6.12</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="k612">
## Download the SDK [step]

Please contact sales@t-firefly.com to get SDK download link and read the readme file

<Callout title="Compilation Notes" type="info">

**Notice:**

**1. SDK use cross-compilation, so use SDK in x86_64 PC, do not download SDK to the device**

**2. We suggest to use Ubuntu20.04 (real PC or docker) to build, other OS may cause building failure**

**3. Do not place or decompress the SDK archive in Virtual Machine share folder or non-english folder**

**4. Please use the regular user to get/compile the SDK, use root privilege may cause building failure**



</Callout>
**SDK Introduction**

<Accordions>

<Accordion title="SDK Structure" type="single" id="sdk612_dir">

```bash
$ tree -L 1
.
├── app                                                        		          # built-in application examples and tools
├── buildroot                                                  		          # buildroot rootfs building, the SDK uses externally pre-built images
├── build.sh -> device/rockchip/common/scripts/build.sh        		          # firmware building script
├── common -> device/rockchip/common                           		          # building configs
├── debian                                                     		          # debian rootfs building, the SDK uses externally pre-built images
├── device                                                     		          # building system, includes building configs
├── docs                                                       		          # development documents
├── external                                                                  # some components
├── firefly-build.sh -> device/rockchip/common/scripts/firefly-build.sh       # Firefly building script
├── kernel -> kernel-6.12                                                     # kernel 6.12 (symlink)
├── kernel-6.12								                               # kernel 6.12 (real directory)
├── Makefile -> device/rockchip/common/Makefile
├── output
├── prebuilt_rootfs                                                           # used to place externally pre-built rootfs images
├── prebuilts                                                                 # cross-build toolchain
├── README.md -> device/rockchip/common/README.md
├── rkbin                                                                     # Rockchip closed-source binaries
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh                   # flashing script
├── rockdev -> output/firmware                                                # firmware output path
├── tools                                                                     # some tools
├── u-boot                                                                    # u-boot 2025.04 source code
└── yocto                                                                     # only used when building with Yocto
```

</Accordion>

<Accordion title="About Configurations" type="single" id="sdk612_config">

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

About config file contents, take firefly_rk3588_lga-3588l-mini-duo_rk182x_ubuntu_defconfig as an example:
```bash
RK_KERNEL_CFG_FRAGMENTS+="firefly.config firefly-rk182x.config firefly-net.config"   # set the config fragments to be used during kernel compilation
RK_KERNEL_DTS_NAME="rk3588-firefly-lga-3588l-mini-duo-rk182x"                        # set the dts to be used during kernel compilation
RK_PRODUCT_MODEL="LGA-3588L-MINI-DUO-RK182X"                                         # product name/model
RK_ROOTFS_PREBUILT_PATH="prebuilt_rootfs/rk3588_ubuntu_rootfs.img"                   # set the prebuilt rootfs to be used
RK_EXTRA_PARTITION_NUM=1                                                             # number of extra partitions
RK_EXTRA_PARTITION_1_DEV="oem"                                                       # oem partition
RK_EXTRA_PARTITION_1_NAME="oem"                                                      # partition name
RK_EXTRA_PARTITION_1_FSTYPE="ext4"                                                   # partition filesystem type
RK_EXTRA_PARTITION_1_SRC="rk1828-policy"                                             # directory location
RK_EXTRA_PARTITION_1_SIZE="128M"                                                     # partition size
RK_PARAMETER="parameter-firefly.txt"                                                 # set the partition parameter
```

</Accordion>

<Accordion title="Partitions" type="single" id="sdk612_partition">

Parameter file records the firmware partitions, for example `device/rockchip/rk3588/parameter-firefly.txt`

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
CMDLINE: mtdparts=:0x00003000@0x00004000(uboot),\
		           0x00002000@0x00007000(misc),\
		           0x00080000@0x00009000(boot:bootable),\
		           0x00040000@0x00089000(recovery),\
		           0x00010000@0x000c9000(backup),\
		           0x01000000@0x000d9000(rootfs),\
		           0x00040000@0x010d9000(oem),\
		           -@0x01129000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
uuid:boot=7A3F0000-0000-446A-8000-702F00006273
```
<Callout title="Parameter Description" type="info">

CMDLINE is what we need to focus on, in "0x00003000@0x00004000(uboot)", 0x00004000 is the start position of uboot, and 0x00003000 is the size of uboot partition. Start position + Partition size = Start position of the next partition. All partitions follow the same rule.

And the unit of these hex numbers is "block", each block is 512 Byte, which means uboot partition size 0x00003000 is 12288 blocks = 12288 x 512 / 1024 / 1024 = 6 MiB

</Callout>

</Accordion>

</Accordions>
## Compile Firmware [step]

  <CodeBlockTabs defaultValue="ubuntu">
    <CodeBlockTabsList>
      <CodeBlockTabsTrigger value="ubuntu">ubuntu</CodeBlockTabsTrigger>
      <CodeBlockTabsTrigger value="debian">debian</CodeBlockTabsTrigger>
    </CodeBlockTabsList>

    <CodeBlockTab value="ubuntu">
Build Ubuntu Firmware

### Preparations [step]

Run these two commands to install necessary tools
```bash
sudo apt update
sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev p7zip-full
```

Download rootfs here [Ubuntu24.04 rootfs(64-bit)](https://community.t-firefly.com/en/doc/download/290), please use rootfs under kernel-6.12 folder.

After download, decompress and move the rootfs image to SDK/prebuilt_rootfs/, then create a symbolic link.

```bash
# Decompress
7z x Ubuntu24.04-xxxx.7z
mkdir ./SDK/prebuilt_rootfs/
mv Ubuntu24.04-xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf Ubuntu24.04-xxxx.img rk3588_ubuntu_rootfs.img
cd ..
```

### Configurations [step]

Run `./build.sh lunch` to list all available configs, enter a number to select. Please choose one with "ubuntu" keyword:
```bash
./build.sh lunch
\n############### Rockchip Linux SDK ###############\n
Manifest: common_linux6.12_next.xml
GIT commit: "c5b718df fix: oem add 1828 sodimm deb and set default"
Log colors: message notice warning error fatal
Log saved at /home4/lhj/arm64/rockchip/rk3588_6.12.69_sdk/output/sessions/2026-11-17_17-48-59
Pick a defconfig:\n
1. rockchip_defconfig
2. firefly_rk3588_aio-3588jd4_ubuntu_defconfig
3. firefly_rk3588_aio-3588l-qua_rk182x_debian_defconfig
4. firefly_rk3588_aio-3588l-qua_rk182x_ubuntu_defconfig
5. firefly_rk3588_aio-3588l_ubuntu_defconfig
6. firefly_rk3588_aio-3588q_defconfig
7. firefly_rk3588_aio-3588sg_ubuntu_defconfig
8. firefly_rk3588_aio-3588sjd4_defconfig
9. firefly_rk3588_itx-3588j_ubuntu_defconfig
10. firefly_rk3588_lga-3588l-mini-duo_rk182x_debian_defconfig
11. firefly_rk3588_lga-3588l-mini-duo_rk182x_ubuntu_defconfig
12. firefly_rk3588_roc-rk3588-rt_defconfig
13. firefly_rk3588_roc-rk3588s-pc_defconfig
14. rockchip_rk3588_evb10_v10_defconfig
15. rockchip_rk3588_evb1_lp4_v10_amp_defconfig
16. rockchip_rk3588_evb1_lp4_v10_defconfig
17. rockchip_rk3588_evb1_lp4_v10_mcu_defconfig
18. rockchip_rk3588_evb7_v11_defconfig
19. rockchip_rk3588_ipc_evb1_v10_defconfig
20. rockchip_rk3588_ipc_evb7_lp4_v11_defconfig
21. rockchip_rk3588_multi_ipc_evb1_v10_defconfig
22. rockchip_rk3588_thunderboot_evb7_v11_defconfig
23. rockchip_rk3588s_evb1_lp4x_v10_defconfig
Which would you like? [1]:
```

### Compilation [step]

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

* Only build kernel, will generate kernel/extboot.img.

```bash
./build.sh extboot
```

* Pack all parts to update.img

```bash
./build.sh updateimg
```
    </CodeBlockTab>

    <CodeBlockTab value="debian">
Build Debian Firmware

### Preparations [step]

Run these two commands to install necessary tools
```bash
sudo apt update
sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev p7zip-full
```

Download rootfs here [Debian13 Rootfs](https://community.t-firefly.com/en/doc/download/161), please use rootfs under kernel-6.12 folder.

After download, decompress and move the rootfs image to SDK/prebuilt_rootfs/, then create a symbolic link.
```bash
# Decompress
7z x debian13_xxxx_rootfs_xxxx.7z
# Move the decompressed rootfs image to sdk and then create a symbolic link.
mkdir ./SDK/prebuilt_rootfs/
mv debian13_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf debian13_xxxx_rtoofs_xxxx.img rk3588_debian_rootfs.img
cd ..
```

### Configurations [step]

Run `./build.sh lunch` to list all available configs, enter a number to select. Please choose one with "debian" keyword:
```bash
./build.sh lunch
\n############### Rockchip Linux SDK ###############\n
Manifest: common_linux6.12_next.xml
GIT commit: "c5b718df fix: oem add 1828 sodimm deb and set default"
Log colors: message notice warning error fatal
Log saved at /home4/lhj/arm64/rockchip/rk3588_6.12.69_sdk/output/sessions/2026-11-17_17-48-59
Pick a defconfig:\n
1. rockchip_defconfig
2. firefly_rk3588_aio-3588jd4_ubuntu_defconfig
3. firefly_rk3588_aio-3588l-qua_rk182x_debian_defconfig
4. firefly_rk3588_aio-3588l-qua_rk182x_ubuntu_defconfig
5. firefly_rk3588_aio-3588l_ubuntu_defconfig
6. firefly_rk3588_aio-3588q_defconfig
7. firefly_rk3588_aio-3588sg_ubuntu_defconfig
8. firefly_rk3588_aio-3588sjd4_defconfig
9. firefly_rk3588_itx-3588j_ubuntu_defconfig
10. firefly_rk3588_lga-3588l-mini-duo_rk182x_debian_defconfig
11. firefly_rk3588_lga-3588l-mini-duo_rk182x_ubuntu_defconfig
12. firefly_rk3588_roc-rk3588-rt_defconfig
13. firefly_rk3588_roc-rk3588s-pc_defconfig
14. rockchip_rk3588_evb10_v10_defconfig
15. rockchip_rk3588_evb1_lp4_v10_amp_defconfig
16. rockchip_rk3588_evb1_lp4_v10_defconfig
17. rockchip_rk3588_evb1_lp4_v10_mcu_defconfig
18. rockchip_rk3588_evb7_v11_defconfig
19. rockchip_rk3588_ipc_evb1_v10_defconfig
20. rockchip_rk3588_ipc_evb7_lp4_v11_defconfig
21. rockchip_rk3588_multi_ipc_evb1_v10_defconfig
22. rockchip_rk3588_thunderboot_evb7_v11_defconfig
23. rockchip_rk3588s_evb1_lp4x_v10_defconfig
Which would you like? [1]:
```

### Compilation [step]

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
    </CodeBlockTab>
  </CodeBlockTabs>
  </CodeBlockTab>
</CodeBlockTabs>