# Compile LibreELEC firmware

Use the SDK officially provided by LibreELEC and attach the patch file provided by firefly. This chapter describes the specific usage of the SDK in detail.

## Ready to work

### Set up SDK compilation environment

**Be sure to confirm the installation of the following files!**

Here we use Ubuntu 18.04 for testing (It is recommended to use the ubuntu18.04 system for development, or use docker to deploy the Ubuntu18.04 container, otherwise the environment package cannot be installed):
```
sudo apt-get update

sudo apt-get install repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler \
gcc-aarch64-linux-gnu mtools parted libudev-dev libusb-1.0-0-dev python-linaro-image-tools \
linaro-image-tools gcc-arm-linux-gnueabihf libssl-dev liblz4-tool genext2fs lib32stdc++6 \
gcc-aarch64-linux-gnu g+conf autotools-dev libsigsegv2 m4 intltool libdrm-dev curl sed make \
binutils build-essential gcc g++ bash patch gzip bzip2 perl tar cpio python unzip rsync file bc wget \
libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev libglade2-dev cvs git mercurial rsync openssh-client \
subversion asciidoc w3m dblatex graphviz python-matplotlib libc6:i386 libssl-dev texinfo fakeroot \
libparse-yapp-perl default-jre patchutils swig
```

### Download LibreELEC source code
```
git clone https://github.com/LibreELEC/LibreELEC.tv.git
```

**Recommendation:** After the source code package is compiled, it occupies about 32G, and it is recommended to reserve more than 35G of available memory.

### Introduction to the main directory of LibreELEC source code

Table of Contents:

```
├── LibreELEC.tv
│   ├── packages                                 Need to download and build the package directory
│   ├── projects                                 System construction project directory supporting platform soc equipment
│   ├── config
│   ├── scripts                                  System build script file directory
│   ├── create_tar.sh
│   ├── sources                                  The storage directory of the resource package pulled from the server when building the system
│   ├── distributions                            Configure the resource package directory required by the build system
│   ├── target                                   The directory where the compiled results are stored
│   ├── licenses
│   ├── Makefile
│   └── tools
```

## Compile

* It is recommended to choose version 9.95.4 of LibreELEC
```
cd LibreELEC.tv
# commit 04dc864eeba97cee2d5375981f898ea246440a21 (tag: 9.95.4)
git checkout 9.95.4
```

* Apply the corresponding patch
```
The code is still being merged and will be released soon
```

* Compile
```
# For users with poor network environment, you can use the following command to download the required source code package
#PROJECT=Rockchip ARCH=arm tools/download-tool

# Execute the compile command to build the system. The first system construction takes a long time (usually 3 to 7 hours), please be patient.
# The compilation commands are described in LibreELEC.tv/projects/Rockchip/devices/RK3399/README.md.
PROJECT=Rockchip DEVICE=RK3399 ARCH=arm UBOOT_SYSTEM=roc-pc-plus make image
```

**You can also download the compiled [LibrEELEC system mirror](http://en.t-firefly.com/doc/download/78.html#other_391)**

### Make SD boot card (currently only support USB/SD card boot)
The compiled image file is stored in the target directory. At this time, you need to download the SD card system creation tool. Go to the following address to download the system creation tool. For example: If you are a linux64-bit operating system, click on the [LibreELEC.USB-SD.Creator.Linux-64bit.bin](https://wiki.libreelec.tv/installation/create-media) tool to download.

* Click the select file option in the second item, find the compiled .gz file, then select the USB or SD card you want to burn, and finally write the image.

**Note**
If you find that the headset audio cannot be used at the first power-on, you need to reprogram the mirror image.