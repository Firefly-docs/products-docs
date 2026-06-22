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

Download Firefly_Linux_SDK sub-volume compressed package: [Firefly_Linux_SDK Source]()

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
MACHINE= bitbake core-image-minimal
MACHINE= bitbake core-image-minimal-xfce
MACHINE= bitbake core-image-x11
MACHINE= bitbake core-image-sato
```



The following is a core-image based on wayland. You need to modify DISPLAY_PLATFORM to wayland in  ```/path/to/yocto/meta-rockchip/conf/machine/include/display.conf```. Modify as follows:

```
DISPLAY_PLATFORM ?= "wayland"
# DISPLAY_PLATFORM ?= "x11"
```

After completing the above modifications, execute the command to compile core-image-weston

```bash
MACHINE= bitbake core-image-weston
```

Note: If you need to change the compiled core-image recipes after you have already compiled core-image once, you need to clean up the currently compiled core-image and then compile a new core-image.

For example: the currently compiled one is core-image-minimal. You need to change it to core-image-sato.

```bash
MACHINE= bitbake core-image-minimal -c clean
MACHINE= bitbake core-image-sato
```



If you want to compile some recipes separately, you can refer to the following:

```bash
# kernel
MACHINE= bitbake linux-rockchip
        
# u-boot
MACHINE= bitbake u-boot-rockchip
        
# rkmpp
MACHINE= bitbake rockchip-mpp
        
# rockchip-librga
MACHINE= bitbake rockchip-librga
        
# See more compilation objects
MACHINE= bitbake -s
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
MACHINE= bitbake <target> <paramater>
# e.g
MACHINE= bitbake u-boot-rockchip -c clean
MACHINE= bitbake u-boot-rockchip
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

