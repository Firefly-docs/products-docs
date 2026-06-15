# Compile Linux AMP firmware

## Overview

The AMP (Asymmetric Multi-Processing) system is a type of non-symmetric multi-core heterogeneous system, in which CPUs are grouped on the same chip and different systems are run on CPUs in different groups. Through efficient task and resource allocation, the AMP system can deliver enhanced performance.

The RK3568 chip has two types of CPUs: Cortex-A55 * 4 + RISC-V * 1. The four Cortex-A55 cores support Linux (kernel-4.19), Baremetal (HAL), and RTOS (RT-Thread), while the RISC-V core supports Baremetal (HAL).
With the help of AMP features, Linux can be run on Cortex-A55 * 4 as the main processor, while RISC-V * 1 can be used as an auxiliary core to run a bare-metal system; or Linux can be run on Cortex-A55 * 3 as the main processor, while Cortex-A55 * 1 can be used as an auxiliary core to run a bare-metal or real-time system, among other combinations using AMP.

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
# Create SDK directory
mkdir ~/proj/rk356x_amp_sdk
cd ~/proj/rk356x_amp_sdk

## Full SDK
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_amp_release.xml

## BSP (Only include some basic repositories and compile tools)
## BSP includes device/rockchip, docs, kernel, u-boot, hal, rt-thread, rkbin, tools and cross-compile toolchian
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_amp_bsp_release.xml
```

* Method Two

Download Firefly_Linux_SDK sub-volume compressed package: [rk356x_amp_release_20240607_v0.0.1a](https://en.t-firefly.com/doc/download/150.html#other_378)

Extract the SDK:

```bash
# add execution permissions
cd /path/to/rk356x_amp_release_20240607_v0.0.1a
chmod +x ./sdk_tools.sh

# create SDK directory
mkdir -p ~/proj/rk356x_amp_sdk

# unzip
./sdk_tools.sh --unpack -C ~/proj/rk356x_amp_sdk

# restore working directory
./sdk_tools.sh --sync -C ~/proj/rk356x_amp_sdk
```

### Sync Code

Execute the following command to synchronize the code:

```bash
# Enter the SDK root directory
cd ~/proj/rk356x_amp_sdk

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
├── buildroot   # Buildroot root filesystem build directory
├── build.sh    # Compile script
├── device      # Compile related configuration
├── docs        # SDK documentation
├── envsetup.sh
├── external
├── hal
│ ├── doc       # HAL documentation
│ ├── html      # HAL doxygen generates driver documentation
│ ├── lib       # HAL driver code
│ ├── project   # HAL project directory
│ ├── test      # HAL test code
│ └── tools     # HAL related tools
├── kernel      # Kernel
├── rt-thread   # RT-Thread
├── prebuilts   # SDK compilation toolchain (including compilation U-Boot, Kernel, HAL compilation toolchain)
├── rkbin
├── tools
│ ├── linux     # Linux platform burning tool
│ ├── mac       # Mac platform burning tool
│ └── windows   # Windows platform burning tools and drivers
└── u-boot      # U-boot
```
### Install dependencies

* Method 1:

Install directly on PC：
```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake \
unzip device-tree-compiler python-pip ncurses-dev python-pyelftools scons \
clang-format astyle libncurses5-dev build-essential python-configparser
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
./build.sh itx-3568q-amp-ubuntu.mk
```

#### Download Ubuntu filesystem

* Download: [Ubuntu rootfs(64-bit)](https://en.t-firefly.com/doc/download/150.html#other_379)，put in SDK path

* Unzip

```bash
7z x ubuntu-aarch64-rootfs.7z
```

* Move filesystem to `ubuntu_rootfs/`

```bash
mkdir ubuntu_rootfs
mv ubuntu-aarch64-rootfs.img ubuntu_rootfs/rk356x_ubuntu_rootfs.img
```

#### SDK unified compilation and packaging

The RK3568 supports Linux, HAL, and RT-Thread's AMP hybrid architecture design, allowing different cpus to run different systems to meet flexible product design needs. The SDK currently uses the mixed structure model of Linux + RT-Thread by default, in which the CPU running Linux is the primary core, and the CPU running rtt is the secondary core. Cores 0 to 2 run Linux, cores 3 run RT-Thread.

#### Compile configuration

The SDK unified compilation configuration script is located in the device/rockchip/rk356x/ directory. The compilation configuration script includes U-Boot, Kernel, HAL, and RT-Thread configurations, as well as AMP-related CPU allocation and memory allocation configurations. You can add or modify configuration script files to meet your compilation requirements. The main AMP configuration files are as follows:

```bash
rk3568_amp_linux.its                    # linux+hal's AMP packs ITS configuration files
rk3568_amp_linux_rtt.its                # linux+rtt's AMP packs ITS configuration files
firefly-rk3568-amp.mk                   # The AMP configuration file corresponding to linux+hal is also the AMP base configuration file
firefly-rk3568-linux-rtt.mk             # The AMP configuration file for linux+rtt
itx-3568q-amp-ubuntu.mk                 # ITX-3568Qcorresponds to the board-level configuration script file
```

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

Notice: The kernel device tree is located in the directory kernel/arch/arm64/boot/dts/rockchip/. The device tree rk3568-firefly-itx-3568q-amp.dts used by it is only compatible with 2GB of memory.
If you need to apply it to devices with other memory specifications, you will need to modify its memory description label memory yourself.

**The hardware resources used by AMP need to be disabled in the kernel device tree and declared under the rockchip_amp label in the rk3568-amp.dtsi device tree.**

```bash
./build.sh kernel
```

* recovery

```bash
./build.sh recovery
```

##### Compile AMP

Compile hal or rt-thread to make amp images

###### Use hal

* Compile hal
```bash
./build.sh hal 3
```
The generated image is hal/project/rk3568/GCC/hal3.bin

###### Use rt-thread

* Compile rt-thread

```bash
./build.sh rtthread 3
```
The generated image is rt-thread/bsp/rockchip/rk3568-32/Image/rtt3.bin

###### Pack the AMP firmware

* Pack the AMP images
```bash
./build.sh ampimg
```
Use the rockdev/amp/amp.its configuration file to package rockdev/amp/amp.img

###### Others

hal or rt-thread uses uart4 as the debug serial port by default with a baud rate of 115200.

* For more details, please refer to the amp documentation in SDK/docs/amp/manuals/rk3568.

#### Pack the firmware

* Update links

Update each part of the `.img` link to the directory `rockdev/`:

```bash
./mkfirmware.sh
```

* Pack the firmware

Pack the firmware, the firmware will be saved to the directory `rockdev/pack/`.

```bash
# Pack RK firmware
./build.sh updateimg
```

### Partition table

#### parameter

The parameter.txt file contains the partition information of the firmware. Take parameter-amp-ubuntu-fit.txt as an example:

path: `device/rockchip/rk356x/parameter-amp-ubuntu-fit.txt`

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
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00001000@0x00006000(misc),0x00001000@0x00007000(amp),0x00040000@0x00008000(boot:bootable),0x00020000@0x00048000(recovery),0x00010000@0x00068000(backup),0x49dc00@0x00078000(rootfs),-@0x515c00(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is where we are concerned. Take uboot as an example. In 0x00002000@0x00004000(uboot), 0x00004000 is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

#### package-file

The package-file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt.

path: `tools/linux/Linux_Pack_Firmware/rockdev/rk356x-amp-ubuntu-package-file`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
uboot           Image/uboot.img
misc            Image/misc.img
amp             Image/amp.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
userdata        RESERVED
backup          RESERVED
```
## Compile Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware. It is recommended to develop in the Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

### Compile SDK

#### Configuration before compilation

In the `device/rockchip/rk356x/` directory, there are configuration files of different board types.

Return to SDK root directory to select the configuration file:

```bash
./build.sh itx-3568q-amp-buildroot.mk
```

#### SDK unified compilation and packaging

The RK3568 supports Linux, HAL, and RT-Thread's AMP hybrid architecture design, allowing different cpus to run different systems to meet flexible product design needs. The SDK currently uses the mixed structure model of Linux + RT-Thread by default, in which the CPU running Linux is the primary core, and the CPU running rtt is the secondary core. Cores 0 to 2 run Linux, cores 3 run RT-Thread.

#### Compile configuration

The SDK unified compilation configuration script is located in the device/rockchip/rk356x/ directory. The compilation configuration script includes U-Boot, Kernel, HAL, and RT-Thread configurations, as well as AMP-related CPU allocation and memory allocation configurations. You can add or modify configuration script files to meet your compilation requirements. The main AMP configuration files are as follows:

```bash
rk3568_amp_linux.its                    # linux+hal's AMP packs ITS configuration files
rk3568_amp_linux_rtt.its                # linux+rtt's AMP packs ITS configuration files
firefly-rk3568-amp.mk                   # The AMP configuration file corresponding to linux+hal is also the AMP base configuration file
firefly-rk3568-linux-rtt.mk             # The AMP configuration file for linux+rtt
itx-3568q-amp-ubuntu.mk                 # ITX-3568Qcorresponds to the board-level configuration script file
```

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

Notice: The kernel device tree is located in the directory kernel/arch/arm64/boot/dts/rockchip/. The device tree rk3568-firefly-itx-3568q-amp.dts used by it is only compatible with 2GB of memory.
If you need to apply it to devices with other memory specifications, you will need to modify its memory description label memory yourself.

**The hardware resources used by AMP need to be disabled in the kernel device tree and declared under the rockchip_amp label in the rk3568-amp.dtsi device tree.**

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

##### Compile AMP

Compile hal or rt-thread to make amp images

###### Use hal

* Compile hal
```bash
./build.sh hal 3
```
The generated image is hal/project/rk3568/GCC/hal3.bin

###### Use rt-thread

* Compile rt-thread

```bash
./build.sh rtthread 3
```
The generated image is rt-thread/bsp/rockchip/rk3568-32/Image/rtt3.bin

###### Pack the AMP firmware

* Pack the AMP images
```bash
./build.sh ampimg
```
Use the rockdev/amp/amp.its configuration file to package rockdev/amp/amp.img

###### Others

hal or rt-thread uses uart4 as the debug serial port by default with a baud rate of 115200.

* For more details, please refer to the amp documentation in SDK/docs/amp/manuals/rk3568.

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

The parameter.txt file contains the partition information of the firmware. Take `parameter-amp-buildroot-fit.txt` as an example:

Path: `device/rockchip/rk356x/parameter-amp-buildroot-fit.txt`

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

Path: `tools/linux/Linux_Pack_Firmware/rockdev/rk356x-amp-package-file`

```bash
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
uboot           Image/uboot.img
misc            Image/misc.img
amp             Image/amp.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
oem             Image/oem.img
userdata        Image/userdata.img
backup          RESERVED
```