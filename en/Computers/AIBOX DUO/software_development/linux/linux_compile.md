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

About config file contents, take firefly_rk3588_lga-3588l-mini-duo_rk182x_defconfig as an example:
```bash
RK_KERNEL_CFG_FRAGMENTS+="firefly.config firefly-rk182x.config firefly-net.config"   # set the config fragments to be used during kernel compilation
RK_KERNEL_DTS_NAME="rk3588-firefly-lga-3588l-mini-duo-rk182x"                        # set the dts to be used during kernel compilation
RK_PRODUCT_MODEL="LGA-3588L-MINI-DUO-RK182X"                                         # product name/model
RK_ROOTFS_PREBUILT_PATH="prebuilt_rootfs/rootfs.img"                                 # rootfs
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

Run the following command to initialize the build environment. You can either install the dependencies directly on your machine via apt, or use docker to set up the required build environment. Using docker is recommended. The environment setup requires Internet access.
```bash
cd xxx_sdk_dir
bash firefly-build.sh init -d

# If you prefer to set up the environment on the host via apt, use the following command:
cd xxx_sdk_dir
bash firefly-build.sh init -a
```
After running init, the environment is set up automatically; just wait for it to finish. `init -d` generates Dockerfile.firefly and docker-compose.yml in the SDK directory, and the build environment is installed through these two files.

Download the rootfs here: [Ubuntu24.04 rootfs (64-bit)](https://community.t-firefly.com/en/doc/download/), please use the rootfs under the Kernel-6.12 folder.

After downloading, decompress the rootfs into SDK/prebuilt_rootfs/ and create a symbolic link.

```bash
# Decompress
7z x ubuntu24.04_xxxx_rootfs_xxxx.7z
# Move the decompressed rootfs image to the SDK and create a symbolic link
mkdir ./SDK/prebuilt_rootfs/
mv ubuntu24.04_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf ubuntu24.04_xxxx_rootfs_xxxx.img rootfs.img
cd ..
```

### Configurations [step]

The configuration before compilation consists of the following steps:

```bash
# 1. Enter the docker bash (skip this step if you set up the environment on the host via apt):
docker compose -f docker-compose.yml exec firefly-sdk bash

# If Docker does not start and you are unable to enter bash, run the commands below to start Docker and get into the container.
docker compose -f docker-compose.yml run --rm firefly-sdk bash

# 2. Select the config. It lists all Firefly products; pick the number matching your carrier board:
bash firefly-build.sh config

[firefly] Pick a product:
  1) firefly_rk3572_aio-3572q38_defconfig                 (AIO-3572Q38)
  2) firefly_rk3572_ext-icore-3572q38_defconfig           (FlyCamera-RK3572)
  3) firefly_rk3572_r3572a_defconfig                      (Firefly-R3572A)
  4) firefly_rk3576_aio-3576q38_defconfig                 (AIO-3576Q38)
  5) firefly_rk3576_aio-3576q_defconfig                   (AIO-3576Q)
  6) firefly_rk3576_roc-rk3576-pc_defconfig               (ROC-RK3576-PC)
  7) firefly_rk3588_aio-3588jd4_defconfig                 (AIO-3588JD4)
  8) firefly_rk3588_aio-3588l-qua_rk182x_defconfig        (AIBOX-QUA)
  9) firefly_rk3588_aio-3588l_defconfig                   (AIO-3588L)
  10) firefly_rk3588_aio-3588q_defconfig                  (AIO-3588Q)
  11) firefly_rk3588_aio-3588sg_defconfig                 (AIO-3588SG)
  12) firefly_rk3588_aio-3588sjd4_defconfig               (AIO-3588SJD4)
  13) firefly_rk3588_itx-3588j_defconfig                  (ITX-3588J)
  14) firefly_rk3588_lga-3588l-mini-duo_rk182x_defconfig  (AIBOX-DUO)
  15) firefly_rk3588_roc-rk3588-rt_defconfig              (ROC-RK3588-RT)
  16) firefly_rk3588_roc-rk3588s-pc-ext_defconfig         (ROC-RK3588S-PC-EXT)
  17) firefly_rk3588_roc-rk3588s-pc_defconfig             (ROC-RK3588S-PC)
Which would you like? [1]:

# 3. Generate the .config file:
Enter the corresponding number after `Which would you like? [1]: `. For example, if the board is `AIBOX DUO`, select 14

# 4. Choose how the rootfs is built:
After selecting the xxx_defconfig and pressing Enter, you need to choose how the root filesystem is built. Since a prebuilt rootfs image is used here, select prebuilt_rootfs, i.e. number 2

[firefly] Pick a rootfs system:
  1) buildroot
  2) prebuilt_rootfs
Which would you like? [1]: 2
```

### Compilation [step]

#### Full Compilation

Run the following command:
```bash
bash firefly-build.sh all
```

After the build completes, the full firmware will be generated at `output/update/update.img`

#### Partial Compilation

Custom development usually only involves the kernel and u-boot. So after a full build has been done once, if you later modify the kernel or u-boot, you can do a partial build to speed things up.

* Only build u-boot, will generate u-boot/uboot.img

```bash
bash firefly-build.sh uboot
```

* Only build kernel, will generate kernel/extboot.img

```bash
bash firefly-build.sh kernel
```

* Only build recovery, will generate output/firmware/recovery.img

```bash
bash firefly-build.sh recovery
```

* Pack all parts to update.img

```bash
./build.sh updateimg
```

### Exit and Re-enter Docker [step]

```bash
# 1. Exit the shell; docker keeps running in the background
exit

# 2. Re-enter docker
docker compose -f docker-compose.yml exec firefly-sdk bash

# 3. Exit and stop docker
docker compose -f docker-compose.yml down

# 4. Start docker again
docker compose -f docker-compose.yml run --rm firefly-sdk bash
```
    </CodeBlockTab>

    <CodeBlockTab value="debian">
Build Debian Firmware

### Preparations [step]

Run the following command to initialize the build environment. You can either install the dependencies directly on your machine via apt, or use docker to set up the required build environment. Using docker is recommended. The environment setup requires Internet access.
```bash
cd xxx_sdk_dir
bash firefly-build.sh init -d

# If you prefer to set up the environment on the host via apt, use the following command:
cd xxx_sdk_dir
bash firefly-build.sh init -a
```
After running init, the environment is set up automatically; just wait for it to finish. `init -d` generates Dockerfile.firefly and docker-compose.yml in the SDK directory, and the build environment is installed through these two files.

Download the rootfs here: [Debian13 rootfs (64-bit)](https://community.t-firefly.com/en/doc/download/), please use the rootfs under the Kernel-6.12 folder.

After downloading, decompress the rootfs into SDK/prebuilt_rootfs/ and create a symbolic link.

```bash
# Decompress
7z x debian13_xxxx_rootfs_xxxx.7z
# Move the decompressed rootfs image to the SDK and create a symbolic link
mkdir ./SDK/prebuilt_rootfs/
mv debian13_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf debian13_xxxx_rootfs_xxxx.img rootfs.img
cd ..
```

### Configurations [step]

The configuration before compilation consists of the following steps:

```bash
# 1. Enter the docker bash (skip this step if you set up the environment on the host via apt):
docker compose -f docker-compose.yml exec firefly-sdk bash

# If Docker does not start and you are unable to enter bash, run the commands below to start Docker and get into the container.
docker compose -f docker-compose.yml run --rm firefly-sdk bash

# 2. Select the config. It lists all Firefly products; pick the number matching your carrier board:
bash firefly-build.sh config

[firefly] Pick a product:
  1) firefly_rk3572_aio-3572q38_defconfig                 (AIO-3572Q38)
  2) firefly_rk3572_ext-icore-3572q38_defconfig           (FlyCamera-RK3572)
  3) firefly_rk3572_r3572a_defconfig                      (Firefly-R3572A)
  4) firefly_rk3576_aio-3576q38_defconfig                 (AIO-3576Q38)
  5) firefly_rk3576_aio-3576q_defconfig                   (AIO-3576Q)
  6) firefly_rk3576_roc-rk3576-pc_defconfig               (ROC-RK3576-PC)
  7) firefly_rk3588_aio-3588jd4_defconfig                 (AIO-3588JD4)
  8) firefly_rk3588_aio-3588l-qua_rk182x_defconfig        (AIBOX-QUA)
  9) firefly_rk3588_aio-3588l_defconfig                   (AIO-3588L)
  10) firefly_rk3588_aio-3588q_defconfig                  (AIO-3588Q)
  11) firefly_rk3588_aio-3588sg_defconfig                 (AIO-3588SG)
  12) firefly_rk3588_aio-3588sjd4_defconfig               (AIO-3588SJD4)
  13) firefly_rk3588_itx-3588j_defconfig                  (ITX-3588J)
  14) firefly_rk3588_lga-3588l-mini-duo_rk182x_defconfig  (AIBOX-DUO)
  15) firefly_rk3588_roc-rk3588-rt_defconfig              (ROC-RK3588-RT)
  16) firefly_rk3588_roc-rk3588s-pc-ext_defconfig         (ROC-RK3588S-PC-EXT)
  17) firefly_rk3588_roc-rk3588s-pc_defconfig             (ROC-RK3588S-PC)
Which would you like? [1]:

# 3. Generate the .config file:
Enter the corresponding number after `Which would you like? [1]: `. For example, if the board is `AIBOX DUO`, select 14

# 4. Choose how the rootfs is built:
After selecting the xxx_defconfig and pressing Enter, you need to choose how the root filesystem is built. Since a prebuilt rootfs image is used here, select prebuilt_rootfs, i.e. number 2

[firefly] Pick a rootfs system:
  1) buildroot
  2) prebuilt_rootfs
Which would you like? [1]: 2
```

### Compilation [step]

#### Full Compilation

Run the following command:
```bash
bash firefly-build.sh all
```

After the build completes, the full firmware will be generated at `output/update/update.img`

#### Partial Compilation

Custom development usually only involves the kernel and u-boot. So after a full build has been done once, if you later modify the kernel or u-boot, you can do a partial build to speed things up.

* Only build u-boot, will generate u-boot/uboot.img

```bash
bash firefly-build.sh uboot
```

* Only build kernel, will generate kernel/extboot.img

```bash
bash firefly-build.sh kernel
```

* Only build recovery, will generate output/firmware/recovery.img

```bash
bash firefly-build.sh recovery
```

* Pack all parts to update.img

```bash
./build.sh updateimg
```

### Exit and Re-enter Docker [step]

```bash
# 1. Exit the shell; docker keeps running in the background
exit

# 2. Re-enter docker
docker compose -f docker-compose.yml exec firefly-sdk bash

# 3. Exit and stop docker
docker compose -f docker-compose.yml down

# 4. Start docker again
docker compose -f docker-compose.yml run --rm firefly-sdk bash
```
    </CodeBlockTab>
  </CodeBlockTabs>
  </CodeBlockTab>
</CodeBlockTabs>