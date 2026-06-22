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