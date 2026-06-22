# Compile Linux firmware





## Compile Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware. It is recommended to develop under Ubuntu 22.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

<font color=red>**The compilation portion of this tutorial works with SDK versions above  v1.0.0a**</font>

```
$ readlink -f .repo/manifest.xml
/home2/lvsx/project/rk3506/.repo/manifests/rk3506_linux_release_20250117_v1.0.0a.xml
```
### Preparatory work

#### Set up compilation environment

```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip \
device-tree-compiler ncurses-dev \
```

### Compile SDK

#### Precompile Configuration

There are configuration files for different board in `device/rockchip/rk3506/`, select the configuration file:

```bash
./build.sh firefly_rk3506_roc-rk3506b-cc-emmc_buildroot_defconfig # EMMC interface (the default is EMMC interface)

or

./build.sh firefly_rk3506_roc-rk3506b-cc-spi-flash_buildroot_defconfig # SPI FLASH interface (need to paste SPI FLASH)
```

#### Build

##### Automatic compilation

* start compiling
```bash
./build.sh
```
the firmware will be saved to the directory `output/update`.

##### Partial compilation

* u-boot

```bash
./build.sh uboot
```

* kernel

```bash
./build.sh kernel
```

* recovery

```bash
./build.sh recovery
```

* buildroot

```bash
./build.sh buildroot
```

* Pack the firmware, the firmware will be saved to the directory `output/update`.

```bash
./build.sh updateimg
```
# Compile Linux AMP firmware

## Overview

AMP (Asymmetric Multi-Processing) system is an asymmetric multi-core heterogeneous system, that is, in the same chip, by grouping CPUs and running different systems in different groups of CPUs, by reasonably allocating tasks and resources, AMP system can provide better performance.

The rk3506 chip has two types of CPUs: Cortex-A7 * 3 + Cortex-M0 * 1. The three Cortex-A7 cores support linux (kernel-6.1), Baremetal (HAL), RTOS (RT-Thread), and Cortex-M0 supports Baremetal (HAL).
Using the AMP feature, you can run linux on Cortex-A7 * 3, and Cortex-M0 * 1 as an auxiliary core to run a bare metal system; you can also run linux on Cortex-A7 * 2, and Cortex-A7 * 1 as an auxiliary core to run a bare metal or real-time system, and other combinations of AMP construction.

## Compilation Environment Setup

This chapter introduces the compilation environment setup for Linux SDK

<font color=red>

**Note:**

**(1) It is recommended to develop in the X86_64 Ubuntu 22.04 system environment. If you use other system versions, you may need to make corresponding adjustments to the compilation environment. **

**(2) Use a normal user to compile, and do not use root user privileges to compile. **</font>

## Get SDK

First, prepare an empty folder to store the SDK. It is recommended to be in the home directory. This article takes `~/proj` as an example

<font color=red>

**Note:**

**1. The SDK uses cross-compilation, so use the SDK on an X86_64 computer and do not download the SDK to the board**

**2. Please use Ubuntu22.04 (real machine or docker container) as the compilation environment. If you use other versions, compilation errors may occur**

**3. Do not store or decompress the SDK in a shared folder of the virtual machine or a non-English directory**

**4. Please use a normal user throughout the process of obtaining and compiling the SDK. Root permissions are not allowed or required (unless apt is required to install the software)**
</font>

### Installation tools
To obtain the SDK, you need to install:
```
sudo apt update
sudo apt install -y repo git python p7zip-full
```
### Initialize the warehouse

**We provide a basic SDK package. Please contact sales@t-firefly.com to obtain it. **

After downloading, verify the MD5 code first:

```bash
$ md5sum rk3506_linux_release_20250117_v1.0.0a/*
a5df458069569e9288b1d96097b43397  rk3506_linux_release_20250117_v1.0.0a.7z.001
7c1a1b050ae4f0daf64fe7488a7f4b02  rk3506_linux_release_20250117_v1.0.0a.7z.002
c7998713e7ef3512635ad2f79730d888  rk3506_linux_release_20250117_v1.0.0a.7z.003
```

After confirmation, you can decompress it:
```bash
mkdir -p ~/proj/rk3506_sdk
cd ~/proj/rk3506_sdk
7z x /path/to/rk3506_linux_release_20250117_v1.0.0a/rk3506_linux_release_20250117_v1.0.0a.7z.001
```
### Installation Dependencies

* Method 1:

Install the environment on your PC:
```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake \
unzip device-tree-compiler python-pip ncurses-dev python-pyelftools scons \
clang-format astyle libncurses5-dev build-essential python-configparser
```
* Method 2: Use Docker

Use dockerfile to create a container and compile in the container, which perfectly solves the compilation environment problem and isolates it from the host environment without affecting each other.

First install docker in the host, please refer to: [Installation Tutorial](https://wiki.t-firefly.com/en/Firefly-Linux-Guide/first_use.html#an-zhuang-docker)

Create a directory as the Docker working directory, for example, `~/docker/`, and create a file `Dockerfile` in it with the following content:
```dockerfile
FROM ubuntu:22.04
MAINTAINER firefly "service@t-firefly.com"

ENV DEBIAN_FRONTEND=noninteractive

RUN cp -a /etc/apt/sources.list /etc/apt/sources.list.bak
RUN sed -i 's@http://.*ubuntu.com@http://repo.huaweicloud.com@g' /etc/apt/sources.list

RUN apt update

RUN apt install -y build-essential crossbuild-essential-arm64 \
	bash-completion vim sudo locales time rsync bc python

RUN apt install -y repo git ssh libssl-dev liblz4-tool lib32stdc++6 \
	expect patchelf chrpath gawk texinfo diffstat binfmt-support \
	qemu-user-static live-build bison flex fakeroot cmake scons \
	unzip device-tree-compiler python-pip ncurses-dev python-pyelftools \
	subversion asciidoc w3m dblatex graphviz python-matplotlib cpio \
	libparse-yapp-perl default-jre patchutils swig expect-dev u-boot-tools \
    clang-format astyle libncurses5-dev build-essential python-configparser

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
Create an image
```bash
cd ~/docker
docker build -t sdkcompiler .
# sdkcompiler is the image name, which can be changed at will. Note that there is a ‘.’ at the end of the command.
# This process will take some time, please be patient
```
After the image is created, create a container and start it
```bash
# Here, mount the folder where the SDK is located in the host to the container, so that the container can access the SDK in the host
# source= fill in the directory where the SDK is located; target= fill in a directory in the container, which must be an empty directory
# ubuntu22 is the container name, firefly is the container hostname, both can be changed at will
# sdkcompiler is the image name in the previous step
docker run --privileged --mount type=bind,source=/home/fierfly/proj,target=/home/firefly/proj --name="ubuntu22" -h firefly -it sdkcompiler
```
Now you can compile the SDK in the container.

How to exit and restart the container:
```bash
# Type exit in the container to exit

# View all containers (including those that have been exited)
docker ps -a

# Restart an exited container and connect
docker start ubuntu22 # Container name
docker attach ubuntu22
```

## Compile Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware. It is recommended to develop in the Ubuntu 22.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

### Compile SDK

#### Pre-compilation configuration

Configuration files for different board types are stored in the `/path/to/sdk/device/rockchip/rk3506/` directory

Go back to the SDK root directory and execute `build.sh` to select the configuration file:

```bash
./build.sh firefly_rk3506_roc-rk3506b-cc-amp-emmc_buildroot_defconfig
```

#### Unified compilation and packaging of SDK

RK3506 supports the AMP hybrid architecture design of Linux, HAL, and RT-Thread, so that different CPUs can run different systems to meet flexible product design requirements. The SDK currently uses the Linux + RT-Thread hybrid structure model by default, where the CPU running Linux is the main core and the CPU running rtt is the slave core; cores 0~1 run Linux, and core 2 runs RT-Thread.

##### Compilation configuration

The unified compilation configuration script of the SDK is located in the device/rockchip/rk3506/ directory. The compilation configuration script includes the configuration of U-Boot, Kernel, HAL, RT-Thread, and AMP-related CPU allocation, memory allocation and other configurations. Users can add or modify the configuration script file as needed to meet their own compilation needs. The main AMP configuration files are as follows:

```bash
amp_linux.its								# Linux + rtt AMP package ITS configuration file
firefly_rk3506_roc-rk3506b-cc-amp-emmc_buildroot_defconfig		# Linux + rtt corresponding configuration file
```

The unified compilation script tool supports one-click compilation and packaging of U-Boot, Kernel, HAL, RT-Thread, ROOTFS, etc., and generates corresponding Image images.

```bash
./build.sh
```

#### Partial compilation

* Compile u-boot

```bash
./build.sh uboot
```

* Compile kernel

**The hardware resources used by AMP need to be disabled in the kernel device tree and declared under the rockchip_amp tag in the rk3506-amp.dtsi device tree**

```bash
./build.sh kernel
```

* Compile recovery

```bash
./build.sh recovery
```

* Compile the Buildroot root file system

Compile the Buildroot root file system, and the compilation output directory will be generated in `buildroot/output`:

```bash
./build.sh buildroot

# Note: Make sure to compile the Buildroot root file system as a normal user to avoid unnecessary errors.
```

##### Compiling AMP

Compile RT-Thread to create amp image

###### Compile and package amp firmware

* Compile and package amp firmware

```bash
./build.sh amp
```

Use /path/to/sdk/device/rockchip/rk3506/amp_linux.its configuration file to package output/firmware/amp.img

###### Other

By default, hal or RT-Thread uses uart5 as the debugging serial port with a baud rate of 1.5M.

#### Packaging firmware

Package the firmware, and the generated complete firmware will be saved in the `output/update/` directory.

```bash
./build.sh updateimg
```

### Partition Description

#### parameter partition table

The parameter.txt file contains the partition information of the firmware. Take `parameter-buildroot-amp-emmc-fit.txt` as an example:

Path: `device/rockchip/rk3506/parameter-buildroot-amp-emmc-fit.txt`

```bash
FIRMWARE_VER:8.1
MACHINE_MODEL:RK3506
MACHINE_ID:007
MANUFACTURER: RK3506
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3506
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
GROW_ALIGN: 0
CMDLINE:mtdparts=:0x00002000@0x00002000(uboot),0x00000800@0x00004000(misc),0x0000A000@0x00004800(recovery),0x00010000@0x0000E800(boot),0x00010000@0x0001E800(backup),0x00008000@0x0002E800(oem),0x00002000@0x00036800(amp),0x00c00000@0x00038800(rootfs),-@0x00c36800(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
uuid:boot=7A3F0000-0000-446A-8000-702F00006273
```

The CMDLINE attribute is what we are concerned about. Taking uboot as an example, 0x00002000@0x00002000(uboot) is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

#### Package-file

The package-file file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt file.

Path: `output/firmware/package-file`

```bash
# NAME  PATH
package-file    package-file
parameter       parameter.txt
bootloader      MiniLoaderAll.bin
uboot   uboot.img
misc    misc.img
recovery        recovery.img
boot    boot.img
backup  RESERVED
oem     oem.img
amp     amp.img
rootfs  rootfs.img
userdata        userdata.img
```