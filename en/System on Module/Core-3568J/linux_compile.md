# Compile Linux4.19 firmware

## Build the build environment

This chapter introduces the compilation environment of the Linux SDK

<font color=red>

**Note:**

**(1) It is recommended to develop in the Ubuntu 18.04 system environment. If other system versions are used, the compilation environment may need to be adjusted accordingly.**

**(2) Compile with ordinary user, do not compile with root user authority.** </font>

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
mkdir ~/proj/rk356x_linux_release_v1.3.0b_20221213/
cd ~/proj/rk356x_linux_release_v1.3.0b_20221213/

## Full SDK
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_release.xml

## BSP (Only include some basic repositories and compile tools)
## BSP includes device/rockchip, docs, kernel, u-boot, rkbin, tools and cross-compile toolchian
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_bsp_release.xml
```

* Method Two

Download Firefly_Linux_SDK sub-volume compressed package: [Firefly_Linux_SDK Source](http://en.t-firefly.com/doc/download/89.html#other_378)

After downloading, verify the MD5 code:

```bash
$ md5sum rk356x_linux_release_v1.3.0b_20221213_split_dir/*firefly_split*
409b81a9ed3bb9a7d6af91223836cad5  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file00
75cef82f2bf91052a7d3c6f0b8405a89  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file01
6f20f62e9652f8f999692587a2ac4b79  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file02
113acbbcd18d3abe0552ef296e983a3f  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file03
624c88a4da2eaa4a48f380783b126d00  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file04
1cf861afb0b36c9ebcf26a7d6effb260  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file05
3009e46fc14481e77fe7ec143e217de4  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file06
5e1cc90b99e34f20b75fb506d3e9bcd7  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file07
1a512fa7c9e2fd1a0781f8d40e228402  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file08
```

After confirming that it is correct, you can unzip:
```bash
# unzip
mkdir ~/proj/
cd ~/proj/
cat path/to/rk356x_linux_release_v1.3.0b_20221213_split_dir/*firefly_split* | tar -xzv

# export data
.repo/repo/repo sync -l
```

### Sync Code

Execute the following command to synchronize the code:

```bash
# Enter the SDK root directory
cd ~/proj/rk356x_linux_release_v1.3.0b_20221213/

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
├── buildroot														# Buildroot root filesystem build directory
├── build.sh -> device/rockchip/common/build.sh					# Compile script
├── debian															# Debian root filesystem compilation directory
├── device															# Compile related configuration files
├── docs															# Documentation
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel															# Kernel
├── Makefile -> buildroot/build/Makefile
├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh	# Link script
├── prebuilts														# Cross compilation toolchain
├── rkbin
├── rkflash.sh -> device/rockchip/common/rkflash.sh				# Flash script
├── tools															# Tools directory
└── u-boot															# U-Boot
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
[What is Ubuntu Minimal firmware?](ubuntu_minimal_support.html)

[What is Ubuntu Desktop firmware?](ubuntu_desktop_support.html)

### Build Linux-SDK

#### Precompile Configuration

There are configuration files for different board in `device/rockchip/rk356x/`.

Return to SDK root directory to select the configuration file:

```bash
./build.sh aio-3568j-ubuntu.mk
```

The configuration file will be connected to `device/rockchip/.BoardConfig.mk`, check the file to verify whether the configuration is successful.

Configruation content:

```bash
# Target arch
export RK_ARCH=arm64 # 64-bit ARM
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly_rk3568 # u-boot configuration
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly_linux_defconfig # kernel configuration
# Kernel dts
export RK_KERNEL_DTS=firefly-rk3568-aioj # dts file
# parameter for GPT table
export RK_PARAMETER=parameter-ubuntu-fit.txt # partition table
# rootfs image path
export RK_ROOTFS_IMG=ubuntu_rootfs/rk356x_ubuntu_rootfs.img # filesystem path
```
#### Select Accessories to Compile

Under `device/rockchip/rk356x/` , in addition to aio-3568j-ubuntu.mk, there are other configuration files that come with different accessories

The name of the configuration file indicates the screen and camera used.
If the screen is not specified, the default HDMI display is used.
If the camera is not specified, the default single camera is used

```bash
# For example:
xxxx-ubuntu.mk                      # Use HDMI + single-camera
xxxx-2cam-ubuntu.mk                 # Use HDMI + dual-camera
xxxx-mipi-ubuntu.mk                 # Use mipi + single-camera
xxxx-mipi-2cam-ubuntu.mk            # Use mipi + dual-camera
xxxx-tf-hdmi-mipi-rk628-ubuntu.mk   # Use HDMI + HDMI TO MIPI_CSI(RK628D)
```

<font color="red">Note:The AIO-3568J  hardware version must be V1.3 and later to support the use of HDMI-TO-MIPI_CSI drive board.</font>


Chose the configuration file you like, execute `build.sh` to make it effective:

```bash
# For example:
./build.sh xxxx-mipi-2cam-ubuntu.mk
```

#### Download Ubuntu filesystem

* Download: [Ubuntu rootfs(64-bit)](http://en.t-firefly.com/doc/download/89.html#other_379)，put in SDK path

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
## Compile Yocto firmware

### Get SDK

```bash
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_yocto_kirkstone_release.xml
.repo/repo/repo sync -c
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
bitbake-layers add-layer ../../meta-clang
bitbake-layers add-layer ../../meta-browser/meta-chromium
bitbake-layers add-layer ../../meta-rockchip
```



Choose one of the commands to compile the complete core-image recipes. The following is an x11 based core-image.

```bash
MACHINE=aio-3568j bitbake core-image-minimal
MACHINE=aio-3568j bitbake core-image-minimal-xfce
MACHINE=aio-3568j bitbake core-image-x11
MACHINE=aio-3568j bitbake core-image-sato
```



The following is a core-image based on wayland. You need to modify DISPLAY_PLATFORM to wayland in  ```/path/to/yocto/meta-rockchip/conf/machine/include/display.conf```. Modify as follows:

```
DISPLAY_PLATFORM ?= "wayland"
# DISPLAY_PLATFORM ?= "x11"
```

After completing the above modifications, execute the command to compile core-image-weston

```bash
MACHINE=aio-3568j bitbake core-image-weston
```

Note: If you need to change the compiled core-image recipes after you have already compiled core-image once, you need to clean up the currently compiled core-image and then compile a new core-image.

For example: the currently compiled one is core-image-minimal. You need to change it to core-image-sato.

```bash
MACHINE=aio-3568j bitbake core-image-minimal -c clean
MACHINE=aio-3568j bitbake core-image-sato
```



If you want to compile some recipes separately, you can refer to the following:

```bash
# kernel
MACHINE=aio-3568j bitbake linux-rockchip
        
# u-boot
MACHINE=aio-3568j bitbake u-boot-rockchip
        
# rkmpp
MACHINE=aio-3568j bitbake rockchip-mpp
        
# rockchip-librga
MACHINE=aio-3568j bitbake rockchip-librga
        
# See more compilation objects
MACHINE=aio-3568j bitbake -s
```



### Adjust compilation speed

Modify the BB_NUMBER_THREADS and PARALLEL_MAKE variables in the file ```/path/to/yocto/meta-rockchip/conf/machine/firefly-rk356x.conf```. If the number of threads is set too large, the machine may run out of memory and cause compilation failure. Please set the compilation speed according to the configuration of the compilation machine.

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
MACHINE=aio-3568j bitbake <target> <paramater>
# e.g
MACHINE=aio-3568j bitbake u-boot-rockchip -c clean
MACHINE=aio-3568j bitbake u-boot-rockchip
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
## Compile Debian Firmware

This chapter introduces the compilation process of Debian firmware. It is recommended to develop under Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

### Compile

The only difference between the compilation method of Debian firmware and Ubuntu is the use of different filesystems, other steps including configuration files are exactly the same.

So please refer to [Compile Ubuntu firmware](linux_compile.html#compile-ubuntu-firmware)

Now only introduce the difference: replacing of filesystem

#### Download Debian filesystem

* Download ：[Debian rootfs(64-bit)](https://en.t-firefly.com/doc/download/89.html#other_524)，and put it in the SDK

* Decompress

```bash
7z x Debian10-xxxx_RK3568_xxxx.7z
```

* Put rootfs under `ubuntu_rootfs/`

```bash
mkdir ubuntu_rootfs
mv Debian10-xxxx_RK3568_xxxx.img ubuntu_rootfs/
```

* Create a symbolic link to `rk356x_ubuntu_rootfs.img`

```bash
cd ubuntu_rootfs
ln -sf Debian10-xxxx_RK3568_xxxx.img rk356x_ubuntu_rootfs.img
```
**Then follow the normal steps just like you are compiling ubuntu firmware**
## Compile Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware. It is recommended to develop in the Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

### Compile SDK

#### Configuration before compilation

In the `device/rockchip/rk356x/` directory, there are configuration files of different board types.

Return to SDK root directory to select the configuration file:

```bash
./build.sh aio-3568j-buildroot.mk
```

The configuration file will be linked to `device/rockchip/.BoardConfig.mk`, check the file to verify whether the configuration is successful.

Related configuration introduction:

```bash
# Target arch
export RK_ARCH=arm64 # 64-bit ARM architecture
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly_rk3568 # u-boot configuration file
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly_linux_defconfig # kernel configuration file
# Kernel dts
export RK_KERNEL_DTS=firefly-rk3568-aioj # dts file
# Buildroot config
export RK_CFG_BUILDROOT=rockchip_rk3568 # Buildroot configuration
# Recovery config
export RK_CFG_RECOVERY=rockchip_rk356x_recovery # recovery configuration
# parameter for GPT table
export RK_PARAMETER=parameter-buildroot-fit.txt # Partition table
# rootfs image path
export RK_ROOTFS_IMG=rockdev/rootfs.${RK_ROOTFS_TYPE} # root filesystem path
```

#### Select Accessories to Compile

Under `device/rockchip/rk356x/` , in addition to aio-3568j-buildroot.mk, there are other configuration files that come with different accessories

The name of the configuration file indicates the screen and camera used.
If the screen is not specified, the default HDMI display is used.
If the camera is not specified, the default single camera is used

```bash
# For example:
xxxx-buildroot.mk              # Use HDMI + single-camera
xxxx-2cam-buildroot.mk         # Use HDMI + dual-camera
xxxx-mipi-buildroot.mk         # Use mipi + single-camera
xxxx-mipi-2cam-buildroot.mk    # Use mipi + dual-camera
```

Chose the configuration file you like, execute `build.sh` to make it effective:

```bash
# For example:
./build.sh xxxx-mipi-2cam-buildroot.mk
```

#### Automatic compilation

Fully automatic compilation will perform all compilation and packaging operations to generate complete firmware.

```bash
./build.sh
```

#### Partial compilation

* Compile u-boot

```bash
./build.sh uboot
```

* Compile kernel

Notice：Firefly kernel does not come with all kernel features, need extra kernel features please refer to [Kernel](kernel_introduction.md)

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

#### Package the firmware

Update each part of the mirror link to the `rockdev/` directory:

```bash
./mkfirmware.sh
```

Pack the firmware, the generated complete firmware will be saved to the `rockdev/pack/` directory.

```bash
./build.sh updateimg
```

### Partition description

#### parameter partition table

The parameter.txt file contains the partition information of the firmware. Take `parameter-buildroot-fit.txt` as an example:

Path: `device/rockchip/rk356x/parameter-buildroot-fit.txt`

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
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00010000@0x00008000(boot),0x00010000@0x00018000(recovery),0x00010000@0x00028000(backup),0x00040000@0x00038000(oem),0x00c00000@0x00078000(rootfs),-@0x00c78000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is our focus. Taking uboot as an example, 0x00004000 in 0x00002000@0x00004000(uboot) is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

#### package-file

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