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