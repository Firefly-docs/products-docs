# Set Up the Build Environment

## Prerequisites [step]

Building the Android firmware requires a powerful host. Confirm the following configuration before compiling:

* 64-bit CPU
* 64 GB physical memory plus swap
* 400GB free disk space
* Ubuntu 20.04 or later

<Callout title="Note" type="warn">

The SDK uses cross-compilation and must be used on an X86_64 computer. Do not download the SDK to the board.

</Callout>

## Install Dependencies [step]

Run the following commands on the Ubuntu host to install the build dependencies:

```bash
sudo apt-get update
sudo apt-get install git-core gnupg flex bison build-essential \
zip curl zlib1g-dev libc6-dev-i386 x11proto-core-dev libx11-dev \
lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig \
device-tree-compiler liblz4-tool python3-pyelftools
```

## Build FAQ [step]

PC systems and environments vary, so the build may still fail due to missing packages after installing the dependencies. For example:

```text
OBJCOPY spl/u-boot-spl-nodtb.bin
CAT     spl/u-boot-spl-dtb.bin
COPY    spl/u-boot-spl.bin
CFGCHK  u-boot.cfg
ERROR: No 'dtc', please: apt-get install device-tree-compiler
```

Install the package indicated by the error message. For the example above, install `device-tree-compiler`:

```bash
sudo apt-get install device-tree-compiler
```

After the environment is ready, continue with [Compile Android Firmware](android_compile_firmware.md).