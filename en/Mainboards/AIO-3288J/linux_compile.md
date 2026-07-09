# Compile Linux firmware

## Build the build environment

This chapter introduces the compilation environment of the Linux SDK.

<font color=red>

**Note:**

**(1) It is recommended to develop in the Ubuntu 16.04 system environment. If other system versions are used, the compilation environment may need to be adjusted accordingly.**

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
mkdir ~/proj/rk3288_linux_release_v2.5.0a_20230510/
cd ~/proj/rk3288_linux_release_v2.5.0a_20230510/

## Full SDK
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3288_linux_release.xml

## BSP (Only include some basic repositories and compile tools)
## BSP includes device/rockchip, docs, kernel, u-boot, rkbin, tools and cross-compile toolchian
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3288_linux_bsp_release.xml
```

* Method Two

Download the SDK source code package.

Download: [Firefly_Linux_SDK](http://en.t-firefly.com/doc/download/16.html#other_306)

After downloading, verify the MD5 code:

```bash
$ md5sum rk3288_linux_release_v2.5.0a_20230510_split_dir/*firefly_split*
f3b9309c574e04491a9e9f3e7a5c5540  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file0
fb0cd76f518405441bbc2ad2334ee6b9  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file1
cb26fddb36d49ff6b79f2fc934a731c1  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file2
43aa8447d4ef303ce41274c6ea7b00a2  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file3
c6c1ed77033af3d26be24974dda1b2aa  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file4
```

After confirming that it is correct, you can unzip:

```bash
# unzip
mkdir ~/proj/
cd ~/proj/
cat path/to/rk3288_linux_release_v2.5.0a_20230510_split_dir/*firefly_split* | tar -xzv

# export data
.repo/repo/repo sync -l
```

### Sync Code

Execute the following command to synchronize the code:

```bash
# Enter the SDK root directory
cd ~/proj/rk3288_linux_release_v2.5.0a_20230510/

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
$ tree -L 1
.
├── app
├── buildroot                                               # Buildroot filesystem
├── build.sh -> device/rockchip/common/build.sh             # compile script
├── debian                                                  # Debian filesystem
├── device                                                  # config files
├── distro
├── docs                                                    # document
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel
├── makefile -> buildroot/build/makefile
├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh   # link script
├── prebuilts                                               # cross-compile toolchain
├── rkbin
├── rkflash.sh -> device/rockchip/common/rkflash.sh         # flashing script
├── tools
├── u-boot
└── yocto
```

### Install dependencies

* Method 1:

Install directly on PC：

```bash
sudo apt-get install repo git gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler mtools \
parted libudev-dev libusb-1.0-0-dev python-linaro-image-tools linaro-image-tools libssl-dev \
autotools-dev libsigsegv2 m4 libdrm-dev curl sed make binutils build-essential gcc g++ bash \
patch gzip bzip2 perl tar cpio python unzip rsync file bc wget libncurses5 libglib2.0-dev \
openssh-client lib32stdc++6 gcc-aarch64-linux-gnu libncurses5-dev lzop libssl1.0.0 libssl-dev \
libglade2-dev cvs mercurial subversion asciidoc w3m dblatex graphviz python-matplotlib \
libc6:i386 texinfo liblz4-tool genext2fs expect autoconf intltool libqt4-dev libgtk2.0-dev
```

* Method 2: Use Docker

Use dockerfile to create a container, build SDK in the container, it will perfectly solve environment problems and isolate with host environments.

First install docker in the host PC, you can refer to [Docker instructions](https://wiki.t-firefly.com/en/Firefly-Linux-Guide/demo_docker.html#install-docker)

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
## Build Ubuntu firmware

This chapter introduces the compilation process of Ubuntu firmware.

### Build Linux-SDK

#### Precompile Configuration

There are configuration files for different board in `device/rockchip/rk3288/`, select the configuration file:

##### HDMI

```bash
./build.sh aio-3288j-ubuntu.mk
```


##### LVDS DM-M10R800

```bash
./build.sh aio-3288j-lvds-ubuntu.mk
```


The configuration file will be connected to `device/rockchip/.BoardConfig.mk`, check the file to verify whether the configuration is successful.

Configruation content:

```bash
# Target arch
export RK_ARCH=arm                                              # 32-bit ARM
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly-rk3288                        # u-boot configuration
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly_linux_defconfig              # kernel configuration
# Kernel dts
export RK_KERNEL_DTS=rk3288-firefly-aio                              # dts file
# parameter for GPT table
export RK_PARAMETER=parameter-ubuntu.txt                        # partition table
# rootfs image path
export RK_ROOTFS_IMG=ubuntu_rootfs/rk3288_ubuntu_rootfs.img     # filesystem path
```

#### Partial compilation

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

#### Download Ubuntu filesystem

* Download: [Ubuntu rootfs(32-bit)](http://en.t-firefly.com/doc/download/16.html#other_122)，put in SDK path

* Unzip

```bash
7z x ubuntu-armhf-rootfs.7z
```

* Move filesystem to `ubuntu_rootfs/`

```bash
mkdir ubuntu_rootfs
mv ubuntu-armhf-rootfs.img ubuntu_rootfs/rk3288_ubuntu_rootfs.img
```

#### Pack the firmware

Update each part of the `.img` link to the directory `rockdev/`:

```bash
./mkfirmware.sh
```

Pack the firmware, the firmware will be saved to the directory `rockdev/pack/`.

```bash
./build.sh updateimg
```

#### Automatic compilation

The automatic compilation will perform the above compilation and packaging operations to generate complete firmware.

```bash
./build.sh
```

### Partition table

#### parameter

The parameter.txt file contains the partition information of the firmware. Take parameter-ubuntu.txt as an example:

path: `device/rockchip/rk3288/parameter-ubuntu.txt`

```bash
FIRMWARE_VER: 8.1
MACHINE_MODEL:RK3288
MACHINE_ID:007
MANUFACTURER:RK3288
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3288
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(trust),0x00002000@0x00008000(misc),0x00010000@0x0000a000(boot),0x00010000@0x0001a000(recovery),0x00010000@0x0002a000(backup),0x00600000@0x0005a000(rootfs),-@0x0065a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is where we are concerned. Take uboot as an example. In 0x00002000@0x00004000(uboot), 0x00004000 is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

#### package-file

The package-file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt.

path: `tools/linux/Linux_Pack_Firmware/rockdev/rk3288-ubuntu-package-file`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
trust           Image/trust.img
uboot           Image/uboot.img
boot            Image/boot.img
misc            Image/misc.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
userdata:grow RESERVED
backup          RESERVED
```## Build Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware.

### Build Linux-SDK

#### Precompile Configuration

There are configuration files for different board in `device/rockchip/rk3288/`, select the configuration file:

##### HDMI

```bash
./build.sh aio-3288j-buildroot.mk
```


##### LVDS DM-M10R800

```bash
./build.sh aio-3288j-lvds-buildroot.mk
```


The configuration file will be connected to `device/rockchip/.BoardConfig.mk`, check the file to verify whether the configuration is successful.

Configruation content:

```bash
# Target arch
export RK_ARCH=arm                                              # 32-bit ARM
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly-rk3288                        # u-boot configuration
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly_linux_defconfig              # kernel configuration
# Kernel dts
export RK_KERNEL_DTS=rk3288-firefly-aio                              # dts file
# Buildroot config
export RK_CFG_BUILDROOT=rockchip_rk3288                         # Buildroot configuration
# Recovery config
export RK_CFG_RECOVERY=rockchip_rk3288_recovery                 # recovery configuration
# parameter for GPT table
export RK_PARAMETER=parameter-buildroot.txt                     # partition table
# rootfs image path
export RK_ROOTFS_IMG=buildroot/output/$RK_CFG_BUILDROOT/images/rootfs.$RK_ROOTFS_TYPE   # filesystem path
```

#### Partial compilation

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

* Buildroot filesystem

Compile the Buildroot filesystem and generate the compiled output directory in `buildroot/output`:

```bash
./build.sh buildroot

# Note: Be sure to compile the Buildroot filesystem as a normal user to avoid unnecessary errors.
```

#### Pack the firmware

Update each part of the `.img` link to the directory `rockdev/`:

```bash
./mkfirmware.sh
```

Pack the firmware, the firmware will be saved to the directory `rockdev/pack/`.

```bash
./build.sh updateimg
```

#### Automatic compilation

The automatic compilation will perform the above compilation and packaging operations to generate complete firmware.

```bash
./build.sh
```

### Partition table

#### parameter

The parameter.txt file contains the partition information of the firmware. Take parameter-buildroot.txt as an example:

path: `device/rockchip/rk3288/parameter-buildroot.txt`

```bash
FIRMWARE_VER: 8.1
MACHINE_MODEL:RK3288
MACHINE_ID:007
MANUFACTURER:RK3288
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3288
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(trust),0x00002000@0x00008000(misc),0x00010000@0x0000a000(boot),0x00010000@0x0001a000(recovery),0x00010000@0x0002a000(backup),0x00020000@0x0003a000(oem),0x00100000@0x0005a000(rootfs),-@0x0015a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is where we are concerned. Take uboot as an example. In 0x00002000@0x00004000(uboot), 0x00004000 is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

#### package-file

The package-file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt.

path: `tools/linux/Linux_Pack_Firmware/rockdev/rk3288-package-file`

```bash
# name          relative path
#
#hwdef          hwdef
package-file    package-file
bootloader      image/miniloaderall.bin
parameter       image/parameter.txt
trust           image/trust.img
uboot           image/uboot.img
misc            image/misc.img
boot            image/boot.img
recovery        image/recovery.img
rootfs          image/rootfs.img
oem             image/oem.img
userdata:grow   image/userdata.img
backup          RESERVED
```