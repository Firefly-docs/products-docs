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

Download Firefly_Linux_SDK sub-volume compressed package: [rk356x_amp_release_20240607_v0.0.1a]()

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

