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

Download rootfs here [Ubuntu rootfs](https://www.t-firefly.com/doc/download/257.html#other_791), in general, choose the filesystem with latest date.

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
## Compile Yocto firmware

### Compile

#### Select image
The Yocto project provides some images that can be used without layers. The following table lists currently supported build images and associated recipes.

| Image name  | Target | provided by layer |
| :--------: | :-------: | :-------: |
| core-image-minimal | A small image that only allows a device to boot | Poky |
| core-image-minimal-xfce | A XFCE minimal demo image | meta-openembedded/meta-xfce |
| core-image-sato | Image with Sato, a mobile environment and visual style for mobile devices. The image supports X11 with a Sato theme, Pimlico applications, and contains terminal, editor, and file manager | Poky |
| core-image-weston | A very basic Wayland image with a terminal | Poky |
| core-image-x11 | A very basic X11 image with a terminal | Poky |


### Build image

**The process of building with the bitbake command needs to ensure that the network connection is normal. If it is a customer in inland China, you need to ensure that it can ping the external network**

* Enter the directory <path/to/yocto/poky> and execute the following commands in sequence
```bash
# Install the required environment packages
# sudo apt install zstd
source oe-init-build-env

# add layer
bitbake-layers add-layer ../../meta-openembedded/meta-oe
bitbake-layers add-layer ../../meta-openembedded/meta-python
bitbake-layers add-layer ../../meta-openembedded/meta-networking
bitbake-layers add-layer ../../meta-openembedded/meta-multimedia
bitbake-layers add-layer ../../meta-openembedded/meta-gnome
bitbake-layers add-layer ../../meta-openembedded/meta-xfce
bitbake-layers add-layer ../../meta-clang
bitbake-layers add-layer ../../meta-browser/meta-chromium
bitbake-layers add-layer ../../meta-rockchip
```



Choose one of the commands to compile the complete core-image recipes. The following is an x11 based core-image.

```bash
MACHINE=aio-3576jd4 bitbake core-image-minimal
MACHINE=aio-3576jd4 bitbake core-image-minimal-xfce
MACHINE=aio-3576jd4 bitbake core-image-x11
MACHINE=aio-3576jd4 bitbake core-image-sato
```



The following is a core-image based on wayland. You need to modify DISPLAY_PLATFORM to wayland in  ```/path/to/yocto/meta-rockchip/conf/machine/include/display.conf```. Modify as follows:

```
DISPLAY_PLATFORM ?= "wayland"
# DISPLAY_PLATFORM ?= "x11"
```

After completing the above modifications, execute the command to compile core-image-weston

```bash
MACHINE=aio-3576jd4 bitbake core-image-weston
```

Note: If you need to change the compiled core-image recipes after you have already compiled core-image once, you need to clean up the currently compiled core-image and then compile a new core-image.

For example: the currently compiled one is core-image-minimal. You need to change it to core-image-sato.

```bash
MACHINE=aio-3576jd4 bitbake core-image-minimal -c clean
MACHINE=aio-3576jd4 bitbake core-image-sato
```



If you want to compile some recipes separately, you can refer to the following:

```bash
# kernel
MACHINE=aio-3576jd4 bitbake linux-rockchip
        
# u-boot
MACHINE=aio-3576jd4 bitbake u-boot-rockchip
        
# rkmpp
MACHINE=aio-3576jd4 bitbake rockchip-mpp
        
# rockchip-librga
MACHINE=aio-3576jd4 bitbake rockchip-librga
        
# See more compilation objects
MACHINE=aio-3576jd4 bitbake -s
```



### Adjust compilation speed

Modify the BB_NUMBER_THREADS and PARALLEL_MAKE variables in the file ```/path/to/yocto/meta-rockchip/conf/machine/firefly-rk3576.conf```. If the number of threads is set too large, the machine may run out of memory and cause compilation failure. Please set the compilation speed according to the configuration of the compilation machine.

```
BB_NUMBER_THREADS = "4"
PARALLEL_MAKE = "-j 4"
```

- [BB_NUMBER_THREADS](https://docs.yoctoproject.org/ref-manual/variables.html#term-BB_NUMBER_THREADS): The maximum number of threads BitBake simultaneously executes.
- [BB_NUMBER_PARSE_THREADS](https://docs.yoctoproject.org/ref-manual/variables.html#term-BB_NUMBER_PARSE_THREADS): The number of threads BitBake uses during parsing.
- [PARALLEL_MAKE](https://docs.yoctoproject.org/ref-manual/variables.html#term-PARALLEL_MAKE): Extra options passed to the `make` command during the [do_compile](https://docs.yoctoproject.org/ref-manual/tasks.html#ref-tasks-compile) task in order to specify parallel compilation on the local build host.
- [PARALLEL_MAKEINST](https://docs.yoctoproject.org/ref-manual/variables.html#term-PARALLEL_MAKEINST): Extra options passed to the `make` command during the [do_install](https://docs.yoctoproject.org/ref-manual/tasks.html#ref-tasks-install) task in order to specify parallel installation on the local build host.


### More bitbake options

Fundamentally, BitBake is a generic task execution engine that allows shell and Python tasks to be run efficiently and in parallel while working within complex inter-task dependency constraints. One of BitBake’s main users, OpenEmbedded, takes this core and builds embedded Linux software stacks using a task-oriented approach.For more detailed usage, please check[《bitbake-user-manual》](https://docs.yoctoproject.org/bitbake/2.0/bitbake-user-manual/bitbake-user-manual-intro.html#)。

```bash
MACHINE=aio-3576jd4 bitbake <target> <paramater>
# e.g
MACHINE=aio-3576jd4 bitbake u-boot-rockchip -c clean
MACHINE=aio-3576jd4 bitbake u-boot-rockchip
```

| Bitbake paramater  | Description |
| :--------: | :-------: |
| -c fetch | Fetches if the downloads state is not marked as done |
| -c clean | Removes all output files for a target |
| -c cleanall | Removes all output files, shared state cache, and downloaded source files for a target |
| -c compile -f | It is not recommended that the source code under the temporary directory is changed directly, but if it is, the Yocto Project might not rebuild it unless this option is used. Use this option to force a recompile after the image is deployed. |
| -c listtasks | Lists all defined tasks for a target |

### Partition firmware upgrade

The compiled firmware is located in the directory ```<path/to/yocto>/build/tmp/deploy/images/<board>/```

```bash
$ sudo upgrade_tool di -boot boot.img
$ sudo upgrade_tool di -uboot uboot.img
$ sudo upgrade_tool di -misc misc.img
$ sudo upgrade_tool di -recovery recovery.img
```

* Partition burning is suitable for debugging stage. For firmware verification, please use the unified firmware burning below.
* Rootfs does not support separate burning. You need to pack the complete firmware before burning.

### Unified firmware upgrade

The compiled firmware is located in the directory ```<path/to/yocto>/build/tmp/deploy/images/<board>/```, the files to be downloaded are .wic and update.img, and after entering the loader mode, execute the following commands :

```bash
$ sudo upgrade_tool wl 0 <IMAGE NAME>.wic
$ sudo upgrade_tool uf update.img
```

* **The default login account of the firmware is: root, password: firefly. The firmware contains a common user account named firefly, and the password is firefly.** 

Note: If you are developing on a Windows PC, you can use RKdevtool to directly burn update.img, **no need to burn `<IMAGE NAME>.wic`**. However, please note that update.img is a link file, so you must select the actual file that the link file points to.

### Related overview
The Yocto Project is an open source collaborative project focused on embedded Linux® operating system development that provides a flexible toolset and development environment that allows embedded device developers worldwide to share technologies, software stacks, configurations and tools for creating these customizations Best Practices for Linux Imaging Collaboration. For more information about the Yocto Project, please refer to the official Yocto Project website:[www.yoctoproject.org/](www.yoctoproject.org/). The Yocto Project home page has the [Yocto Project Reference Manual](https://docs.yoctoproject.org/current/ref-manual/index.html) and the [Yocto Project Overview](https://docs.yoctoproject.org/) and other related documents describe in detail how to build the system.

### Introduction to Yocto Project Release layer

| layer  | path | priority(The higher the number, the higher the priority) | describe |
| :--------: | :-------: | :-------: | :-------: |
| meta-oe | meta-openembedded/meta-oe | 6 | contains a large amount of additional recipes |
| meta-python | meta-openembedded/meta-python |  7 |  Provide Python recipes|
| meta-qt5 | meta-qt5 | 7 | Provides QT5 recipes |
| meta-clang | meta-clang |  7 |  clang compiler |
| meta-rockchip | meta-rockchip |  9 |  Rockchip board level support available|
| meta | meta |  5 |  Contains the OpenEmbedded-Core metadata|
| meta-poky | meta-poky |  5 |  Holds the configuration for the Poky reference distribution|
| meta-yocto-bsp | meta-yocto-bsp |  5 |  Configuration for the Yocto Project reference hardware board support package.|
| meta-chromium | meta-chromium |  7 |  Provide chromium browser recipe|
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

Download rootfs here [Debian rootfs](https://www.t-firefly.com/doc/download/257.html#other_791), in general, choose the filesystem with latest date.

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

But if you modified buildroot, then you have to do full compilation.