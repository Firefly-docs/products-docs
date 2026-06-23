# Compile BSP

## Introduction

BSP is a low-level software development kit between the bottom-level hardware and the upper-level software. Its main function is to shield the hardware and provide operating system drivers and hardware drivers. BSP is used to build system software, such as bootloader, kernel and file system.

## Environment preparation

Compilation host requirements:
- OS: Ubuntu 16.04/18.04 or above
- Storage: 40G of free disk space
- Docker is installed and the development user account has docker permissions

## Download BSP SDK

- Download `bsp-sdk-binary.tgz` from [Original SDK](https://gitee.com/sophon-ai/bsp-sdk)
- Go to [Download Center] to download `bm1684_build.docker.tar`, `ec-a1684jd4_bsp.initial.bundle` and `ec-a1684jd4_bsp.update.bundle`.

Run the following command to complete the initialization of the BSP SDK:

```shell
# This assumes that all downloaded files are placed in the current directory

# Install development tools
sudo apt install bison flex bc rsync kmod cpio sudo uuid-dev cmake libssl-dev fakeroot dpkg-dev device-tree-compiler u-boot-tools

# Clone the BSP SDK from ec-a1684jd4_bsp.initial.bundle
git init bm1684-sdk
cd bm1684-sdk
git fetch ../ec-a1684jd4_bsp.initial.bundle
git reset --hard FETCH_HEAD

# Update to the latest from ec-a1684jd4_bsp.update.bundle
git fetch ../ec-a1684jd4_bsp.update.bundle
git merge FETCH_HEAD

# Unzip the SDK package
cd bm1684-sdk
tar xf ../bsp-sdk-binary.tgz

# Import the Docker build image
docker load < ../bm1684_build.docker.tar
```

## Update BSP SDK
- Go to [Download Center] to download the latest `ec-a1684jd4_bsp.update.bundle`

Run the following command to complete the update of the BSP SDK:
```shell
# This assumes that all downloaded files are placed in the bsp sdk directory

# Pull the modified commit from the bundle
git fetch ec-a1684jd4_bsp.update.bundle

# View modification records
git log FETCH_HEAD

# Merge into the current development branch
git merge FETCH_HEAD
```

## Compile and upgrade firmware

Assuming the current BSP SDK directory, run the following command to generate the upgrade firmware:
```shell
# After the following command is run, you need to enter the user password to do sudo operations
./build.sh build_all

# The final generated firmware file is: install/soc_bm1684_asic/sdcard.zip
```

[Download Center]: https://en.t-firefly.com/doc/download/290.html