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