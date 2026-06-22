## Build the build environment

This chapter introduces the compilation environment of the Linux SDK.

<font color=red>

**Note:**

**(1) It is recommended to develop in the Ubuntu 16.04 system environment. If other system versions are used, the compilation environment may need to be adjusted accordingly.**

**(2) Compile with ordinary user, do not compile with root user authority.** </font>

!INCLUDE "include/get_linux_sdk.mdpp"

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
