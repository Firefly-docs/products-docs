# Compiling BSP

## Introduction

BSP (Board Support Package) is a low-level software development kit that serves as an intermediary between the underlying hardware and upper-level software. Its primary function is to abstract the hardware and provide operating system drivers and hardware drivers. BSP is used for building system software, such as bootloaders, kernels, and file systems.

## Environment Preparation

Requirements for the compilation host:
- Operating System: Ubuntu 16.04/18.04 or later
- Storage Space: 40G of free disk space
- Docker must be installed, and the development user account should have Docker permissions

## Download BSP SDK

- Download `bsp-sdk-binary.tgz` from the [official SDK](https://gitee.com/sophon-ai/bsp-sdk)
- Download `bm1688_build.docker.tar`, `ec-a1688jd4_bsp.initial.bundle`, and `ec-a1688jd4_bsp.update.bundle` from the [download center]

Run the following commands to complete the initialization of the BSP SDK:

```shell
# Assuming all downloaded files are in the current directory

# Install development tools
sudo apt install bison flex bc rsync kmod cpio sudo uuid-dev cmake libssl-dev fakeroot dpkg-dev device-tree-compiler u-boot-tools

# Check out BSP SDK from ec-a1688jd4_bsp.initial.bundle
git init bm1688-sdk
cd bm1688-sdk
git fetch ../ec-a1688jd4_bsp.initial.bundle
git reset --hard FETCH_HEAD

# Update to the latest from ec-a1688jd4_bsp.update.bundle
git fetch ../ec-a1688jd4_bsp.update.bundle
git merge FETCH_HEAD

# Extract the SDK package
tar xf ../bsp-sdk-binary.tgz

# Import Docker build image
docker load < ../bm1688_build.docker.tar
```

## Updating BSP SDK
- Download the latest `ec-a1688jd4_bsp.update.bundle` from the [download center]

Run the following commands to complete the BSP SDK update:
```shell
# Assuming all downloaded files are in the BSP SDK directory

# Pull modification commits from the bundle
git fetch ec-a1688jd4_bsp.update.bundle

# View modification records
git log FETCH_HEAD

# Merge into the current development branch
git merge FETCH_HEAD

# Remove the bundle file
rm ec-a1688jd4_bsp.update.bundle
```

## Compiling Upgrade Firmware

Assuming you are currently in the BSP SDK directory, run the following command to generate the upgrade firmware:
```shell
# The following command will require the user password for sudo operations
./build.sh build_all

# The final generated firmware file will be: install/soc_bm1688_asic/sdcard.zip
```

[Download Center]: https://www.t-firefly.com/doc/download/266.html