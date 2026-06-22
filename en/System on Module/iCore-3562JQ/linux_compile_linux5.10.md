# Compile Linux firmware (kernel-5.10)

## Build the build environment

This chapter introduces the compilation environment of the Linux SDK

<font color=red>

**Note:**

**(1) It is recommended to develop in the Ubuntu 18.04 system environment. If other system versions are used, the compilation environment may need to be adjusted accordingly.**

**(2) Unzip and Compile with normal user, do not unzip or compile with root user authority. Do not unzip SDK to shared directory of VMs.** </font>


### Download Firefly_Linux_SDK

First prepare an empty folder to place SDK, better under home, here we use `~/proj` as example.

<font color=red>**Attention: To avoid unnecessary errors, please do not place/unzip the SDK in VM shared folders or non-english directories.**</font>

Get SDK needs:
```
sudo apt update
sudo apt install -y repo git python
```
* Method One

Download via repo, you can choose to get full SDK or BSP:

```bash
# Create SDK directory
mkdir ~/proj/rk3562_sdk
cd ~/proj/rk3562_sdk

# Please contact sales@t-firefly.com to get repo urls.
```

* Method Two

Download Firefly_Linux_SDK sub-volume compressed package: Please contact sales@t-firefly.com to get download urls.

Extract the SDK:

```bash
# add execution permissions
cd /path/to/rk3562_release
chmod +x ./sdk_tools.sh

# create SDK directory
mkdir -p ~/proj/rk3562_sdk

# unzip
./sdk_tools.sh --unpack -C ~/proj/rk3562_sdk

# restore working directory
./sdk_tools.sh --sync -C ~/proj/rk3562_sdk
```

### Sync Code

Execute the following command to synchronize the code:

```bash
# Enter the SDK root directory
cd ~/proj/rk3562_sdk

# Sync
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

You can use the following command to update the SDK later:

```bash
.repo/repo/repo sync -c --no-tags
```

### Directory

```bash
.
├── app
├── buildroot
├── build.sh -> device/rockchip/common/scripts/build.sh     # build script
├── device/rockchip                                         # repository for build and config management
├── docs                                                    # useful documents
├── external                                                # some components
├── kernel
├── Makefile -> device/rockchip/common/Makefile
├── output                                                  # build output
├── prebuilt_rootfs                                         # to place prebuilt rootfs
├── prebuilts/gcc/linux-x86/                                # cross-build tool chain
├── README.md -> device/rockchip/common/README.md
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
├── rockdev -> output/firmware                              # build output of different partition images
├── tools                                                   # some tools, include upgrade_tool
└── u-boot
```
### Install dependencies

* Method 1:

Install directly on PC：
```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake \
unzip device-tree-compiler python-pip ncurses-dev python-pyelftools
```
* Method 2: Use Docker

Use dockerfile to create a container, build SDK in the container, it will perfectly solve environment problems and isolate with host environments.

First install docker in the host PC, you can refer to [Docker instructions](https://wiki.t-firefly.com/en/Firefly-Linux-Guide/first_use.html#install-docker)

Create an empty folder as docker work dir, like `~/docker/`, then touch a dockerfile with contents:
```dockerfile
FROM ubuntu:18.04
MAINTAINER firefly "service@t-firefly.com"

ENV DEBIAN_FRONTEND=noninteractive

RUN apt update

RUN apt install -y build-essential crossbuild-essential-arm64 \
	bash-completion vim sudo locales time rsync bc python

RUN apt install -y repo git ssh libssl-dev liblz4-tool lib32stdc++6 \
	expect patchelf chrpath gawk texinfo diffstat binfmt-support \
	qemu-user-static live-build bison flex fakeroot cmake \
	unzip device-tree-compiler python-pip ncurses-dev python-pyelftools \
	subversion asciidoc w3m dblatex graphviz python-matplotlib cpio \
	libparse-yapp-perl default-jre patchutils swig expect-dev u-boot-tools

RUN apt update && apt install -y -f

# language support
RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8

# switch to a no-root user
RUN useradd -c 'firefly user' -m -d /home/firefly -s /bin/bash firefly
RUN sed -i -e '/\%sudo/ c \%sudo ALL=(ALL) NOPASSWD: ALL' /etc/sudoers
RUN usermod -a -G sudo firefly

USER firefly
WORKDIR /home/firefly
```
Create image
```bash
cd ~/docker
docker build -t sdkcompiler .
# sdkcompiler is image name, you can change it, notice that there's a '.' at the end of cmd
# This process takes a while, please wait
```
Then create a container
```bash
# Here we mount host SDK location into the container, so that you can access SDK inside container
# source= is host SDK location; target= is a folder inside container, must be an empty folder
# ubuntu18 is container name, firefly is container's hostname, you can change them
# sdkcompiler is the image created in last step
docker run --privileged --mount type=bind,source=/home/fierfly/proj,target=/home/firefly/proj --name="ubuntu18" -h firefly -it sdkcompiler
```
Now you can build SDK inside the container.

How to quit container and how to reopen:
```bash
# Execute "exit" inside container will quit and close it

# See all containers (include exited ones)
docker ps -a

# Start an exited container and attach it
docker start ubuntu18 # container name
docker attach ubuntu18
```
## Compile Ubuntu firmware

This chapter introduces the compilation process of Ubuntu firmware. It is recommended to develop under Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

### A brief introduction to Ubuntu firmware
[What is Ubuntu Minimal ?](ubuntu_minimal_support.html)

[What is Ubuntu Desktop ?](ubuntu_desktop_support.html)

### Build Linux-SDK

#### Precompile Configuration

There are configuration files for different board in `device/rockchip/rk3562/`.

Return to SDK root directory to select the configuration file:

```bash
./build.sh firefly_rk3562_aio-3562jq_ubuntu_defconfig
```

The configuration file will be connected to `output/defconfig`, check the file to verify whether the configuration is successful.

Configruation content:

```bash
# prebuilt rootfs
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk3562_ubuntu_rootfs.img"
# UART for bluetooth
RK_WIFIBT_TTY="ttyS4"
# kernel defconfig
RK_KERNEL_CFG="firefly_linux_defconfig"
# device tree name
RK_KERNEL_DTS_NAME="rk3562-firefly-aio-3562jq"
# ramdisk image
RK_RAMDISK_IMG="ramdisk.img"
# image tree source for boot partition image
RK_BOOT_FIT_ITS="bootramdisk.its"
# prebuilt recovery image
RK_RECOVERY_RAMDISK="rk3562-recovery-arm64.cpio.gz"
# parameter
RK_PARAMETER="parameter-ubuntu-fit.txt"
# use FIT image
RK_USE_FIT_IMG=y
# use extlinux way to load kernel
USE_EXTBOOT=y
```

#### Download Ubuntu filesystem

* Download: [Ubuntu rootfs(64-bit)](https://en.t-firefly.com/doc/download/222.html#other_669)，put in SDK path

* Unzip

```bash
# For example, the archive you download is Ubuntu20.04-xxx_RK3562_xxx.7z
7z x Ubuntu20.04-xxx_RK3562_xxx.7z
```

* Move filesystem to `prebuilt_rootfs/`

```bash
mkdir prebuilt_rootfs
# For example, after unzip, the filesystem image is Ubuntu20.04-xxx_RK3562_xxx.img
mv Ubuntu20.04-xxx_RK3562_xxx.img prebuilt_rootfs/
ln -sf Ubuntu20.04-xxx_RK3562_xxx.img rk3562_ubuntu_rootfs.img
```

#### Automatic compilation

The automatic compilation will perform all compilation and packaging operations to generate RK firmware.

```bash
./build.sh all
```
Firmware will be saved to the directory `output/update/`.

#### Partial compilation

* u-boot

```bash
./build.sh uboot
```
Output image is u-boot/uboot.img

* kernel

Notice：Firefly kernel does not come with all kernel features, need extra kernel features please refer to [Kernel](kernel_introduction.md)

```bash
./build.sh extboot
```
Output image is kernel/extboot.img

#### Pack the firmware

```bash
./build.sh updateimg
```
Pack the firmware, the firmware will be saved to the directory `output/update/`.

### Partition table

#### parameter

The parameter.txt file contains the partition information of the firmware. Take parameter-ubuntu-fit.txt as an example:

path: `device/rockchip/rk3562/parameter-ubuntu-fit.txt`

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
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00040000@0x00048000(recovery),0x00010000@0x00088000(backup),0x00c00000@0x00098000(rootfs),-@0x00c98000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is where we are concerned. Take uboot as an example. In 0x00002000@0x00004000(uboot), 0x00004000 is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on. The unit is block, each block is 512 Byte.
## Compile Yocto firmware

### Get SDK

```bash
git clone https://gitlab.com/firefly-linux/git-repo.git
./git-repo/repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3562_yocto_kirkstone_release.xml
.repo/repo/repo sync -c --no-tags
```

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
bitbake-layers add-layer ../../meta-lts-mixins
bitbake-layers add-layer ../../meta-clang
bitbake-layers add-layer ../../meta-browser/meta-chromium
bitbake-layers add-layer ../../meta-rockchip
```



Choose one of the commands to compile the complete core-image recipes. The following is an x11 based core-image.

```bash
MACHINE=aio-3562jq bitbake core-image-minimal
MACHINE=aio-3562jq bitbake core-image-minimal-xfce
MACHINE=aio-3562jq bitbake core-image-x11
MACHINE=aio-3562jq bitbake core-image-sato
```



The following is a core-image based on wayland. You need to modify DISPLAY_PLATFORM to wayland in  ```/path/to/yocto/meta-rockchip/conf/machine/include/display.conf```. Modify as follows:

```
DISPLAY_PLATFORM ?= "wayland"
# DISPLAY_PLATFORM ?= "x11"
```

After completing the above modifications, execute the command to compile core-image-weston

```bash
MACHINE=aio-3562jq bitbake core-image-weston
```

Note: If you need to change the compiled core-image recipes after you have already compiled core-image once, you need to clean up the currently compiled core-image and then compile a new core-image.

For example: the currently compiled one is core-image-minimal. You need to change it to core-image-sato.

```bash
MACHINE=aio-3562jq bitbake core-image-minimal -c clean
MACHINE=aio-3562jq bitbake core-image-sato
```



If you want to compile some recipes separately, you can refer to the following:

```bash
# kernel
MACHINE=aio-3562jq bitbake linux-rockchip
        
# u-boot
MACHINE=aio-3562jq bitbake u-boot-rockchip
        
# rkmpp
MACHINE=aio-3562jq bitbake rockchip-mpp
        
# rockchip-librga
MACHINE=aio-3562jq bitbake rockchip-librga
        
# See more compilation objects
MACHINE=aio-3562jq bitbake -s
```



### Adjust compilation speed

Modify the BB_NUMBER_THREADS and PARALLEL_MAKE variables in the file ```/path/to/yocto/meta-rockchip/conf/machine/firefly-rk3562.conf```. If the number of threads is set too large, the machine may run out of memory and cause compilation failure. Please set the compilation speed according to the configuration of the compilation machine.

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
MACHINE=aio-3562jq bitbake <target> <paramater>
# e.g
MACHINE=aio-3562jq bitbake u-boot-rockchip -c clean
MACHINE=aio-3562jq bitbake u-boot-rockchip
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

## Compile Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware. It is recommended to develop in the Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

### Compile SDK

#### Configuration before compilation

In the `device/rockchip/rk3562/` directory, there are configuration files of different board types.

Return to SDK root directory to select the configuration file:

```bash
./build.sh firefly_rk3562_aio-3562jq_buildroot_defconfig
```

The configuration file will be linked to `output/defconfig`, check the file to verify whether the configuration is successful.

Related configuration introduction:

```bash
# device tree name
RK_KERNEL_DTS_NAME="rk3562-firefly-aio-3562jq"
# kernel defconfig
RK_KERNEL_CFG="firefly_linux_defconfig"
# use FIT image
RK_USE_FIT_IMG=y
# UART for bluetooth
RK_WIFIBT_TTY="ttyS8"
# parameter
RK_PARAMETER="parameter-buildroot-fit.txt"
# image tree source of boot partition image
RK_BOOT_FIT_ITS="bootramdisk.its"
# ramdisk image
RK_RAMDISK_IMG="ramdisk.img"
# use extlinux way to load kernel
USE_EXTBOOT=y
```

#### Automatic compilation

Fully automatic compilation will perform all compilation and packaging operations to generate complete firmware.

```bash
./build.sh all
```
Firmware will be saved to `output/update/` directory

#### Partial compilation

* Compile u-boot

```bash
./build.sh uboot
```
Output image is u-boot/uboot.img

* Compile kernel

Notice：Firefly kernel does not come with all kernel features, need extra kernel features please refer to [Kernel](kernel_introduction.md)

```bash
./build.sh extboot
```
Output image is kernel/extboot.img

* Compile recovery

```bash
./build.sh recovery
```
Outputs are under output/recovery

* Compile Buildroot root filesystem

```bash
./build.sh buildroot

# Note: Make sure to compile the Buildroot root filesystem as a normal user to avoid unnecessary errors.
```
Compiling the Buildroot root filesystem will generate a compilation output directory in `buildroot/output`

#### Package the firmware

```bash
./build.sh updateimg
```
Pack the firmware, the generated complete firmware will be saved to the `output/update/` directory.

### Partition description

#### parameter partition table

The parameter.txt file contains the partition information of the firmware. Take `parameter-buildroot-fit.txt` as an example:

Path: `device/rockchip/rk3562/parameter-buildroot-fit.txt`

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
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00040000@0x00048000(recovery),0x00010000@0x00088000(backup),0x00040000@0x00098000(oem),0x00c00000@0x000d8000(rootfs),-@0x00cd8000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is our focus. Taking uboot as an example, 0x00004000 in 0x00002000@0x00004000(uboot) is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on. The unit is block, each block is 512 Byte.