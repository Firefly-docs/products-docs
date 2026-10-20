# Qualcomm Linux 1.7 Compile

Qualcomm Linux (QLI) is an embedded Linux distribution designed and optimized for Qualcomm chips, i.e. the official Qualcomm BSP/SDK.  
It provides a curated and verified software stack covering the bootloader, kernel and user space, enabling developers to build programs for Qualcomm hardware.  
QLI is based on Yocto, so customizing QLI requires a good understanding of Yocto.

## Environment Setup

An Ubuntu 22.04 x86_64 host environment is required, and Docker is strongly recommended.  
Please search online for how to install Docker and how to configure its network proxy.

After Docker is installed, run:
```bash
# Pull the base ubuntu22 image
docker pull ubuntu:22.04

# Create the container
# -v src:des mounts the host directory src into the container at des, so that project files can be accessed inside the container; set it as needed
# --name is the container name; pick one yourself
# --hostname is the container hostname; pick one yourself
docker run --privileged -v /home/xxx/project:/home/xxx/project \
        --name container_name --hostname host_name -it ubuntu:22.04 /bin/bash
```

Once the container is running, install the environment inside it:
```bash
sudo apt update

sudo apt install build-essential crossbuild-essential-arm64 \
        bash-completion vim sudo locales time rsync bc python2 python3

sudo apt install git lz4 zstd diffstat qemu-user-static \
        xmlstarlet texinfo chrpath libarchive-dev ssh libselinux1-dev \
        fakechroot libiberty-dev liblz4-tool qemu-user-static qemu-user \
        gawk libwayland-dev fakeroot libpam0g-dev binutils-dev util-linux \
        uuid-dev m4 zip unzip cpio wget python3-pip lib32stdc++6 \
        binfmt-support binutils kconfig-frontends python3-rosdep2 libgtest-dev \
        python3-lxml libncurses5 pkg-config flex bison libxml2-utils \
        libxml-simple-perl whiptail language-pack-en file zlib1g-dev

sudo apt install gnupg curl libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev \
        libx11-dev lib32z1-dev libgl1-mesa-dev xsltproc fontconfig

sudo rm -rf /lib/ld-linux-aarch64.so.1
sudo ln -sf /usr/aarch64-linux-gnu/lib/ld-linux-aarch64.so.1 /lib/ld-linux-aarch64.so.1
sudo ln -sf /bin/bash /bin/sh
sudo ln -sf /usr/bin/python3 /usr/bin/python

pip3 install dtschema==2021.10
pip3 install jsonschema==4.0.0
pip3 install ruamel.yaml==0.17.17
pip3 install jinja2
pip3 install click
pip3 install lark
```

Configure git
```bash
git config --global user.email <Your email ID>
git config --global user.name <"Your Name">
git config --global http.postBuffer 1048576000
git config --global http.maxRequestBuffer 1048576000
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
git config --global http.followRedirects true
```

## Download

Exit the container and go back to the host. Go to the [download page](https://community.t-firefly.com/en/doc/download/426) to download the SDK.

Put the downloaded archives somewhere the container can access (the location mounted with `-v` when creating the container earlier), and check the md5 of the split archives:
```bash
md5sum ./qcom-linux-*split*
```

Compare the result with the md5sum.txt shipped with the SDK. After confirming it matches, extract and check out the code:
```bash
cat ./qcom-linux-1.7.tgz.split* | tar zx
cd qcom-linux-1.7
git checkout -f HEAD
```

## Build

Go back to the container and change to the SDK root directory. Ready-made build scripts are provided there.

> ffbuild.sh : The main build script. Builds automatically.  
> bbenv.sh : The Yocto environment initialization script, used to set up the environment for manual builds.

### Full Build

```bash
# First run lunch to select the build target
./ffbuild.sh lunch

# Then start the build
./ffbuild.sh image
```

> Note: If tensorflow-lite keeps failing to build due to network issues, you can use the remove_tensorflow-lite.patch patch to remove this feature and try building again.

Output directory: `build-qcom-wayland/tmp-glibc/deploy/images/<TARGET>/qcom-multimedia-image`, where TARGET is the build target model you selected; the same applies below. This directory can be used directly for flashing.

> Note: The default output is UFS firmware.

For QCS6490 products, do the following to switch it to the eMMC version:

```bash
# Enter the flash package
cd build-qcom-wayland/tmp-glibc/deploy/images/<TARGET>/qcom-multimedia-image

# Remove the original UFS xml files
rm rawprogram*.xml
rm patch*.xml

# Copy the eMMC xml and bin files
cp partition_emmc/rawprogram[0-1].xml ./
cp partition_emmc/patch[0-1].xml ./
cp partition_emmc/*.bin ./
```

### Custom Kernel

Use the Yocto devtool to customize the kernel:
```bash
# Set up the Yocto environment
source bbenv.sh

# Check the workspace status; the first run should show that there is no workspace yet
devtool status

# Create the kernel workspace; linux-qcom-custom is the name of the kernel recipe
devtool modify linux-qcom-custom

# After the previous step finishes, the kernel source will be in the workspace, where you can modify it
cd workspace/sources/linux-qcom-custom

# When done, go back to the build directory and check the workspace status again
cd /path/to/sdk/build-qcom-wayland
devtool status

# After confirming that devtool status correctly detects the linux-qcom-custom workspace, you can start the build
bitbake linux-qcom-custom qcom-multimedia-image
```

### Build Device Tree Only

If subsequent development only modifies the device tree, you can:
```bash
# Set up the Yocto environment
source bbenv.sh

# Build the dtb with bitbake
bitbake dtb-qcom-image
```

The generated file is `build-qcom-wayland/tmp-glibc/deploy/images/<TARGET>/dtb-qcom-image-*.vfat`; simply rename it to `dtb.bin`.

## Creating Ubuntu Firmware

The Ubuntu system does not support direct compilation. Please go to the [download center](https://community.t-firefly.com/en/doc/download/426) to download the released ready-made firmware. For customization, you need to port the kernel, modules, firmware and other parts generated by the QLI build into the Ubuntu system.

### Preparation

First copy some of the QLI build outputs to a convenient location; ~/work is used as an example here:
```bash
# dtb
cp build-qcom-wayland/tmp-glibc/deploy/images/<TARGET>/dtb-qcom-image-<TARGET>.rootfs-*.vfat ~/work/

# kernel
cp -rfd build-qcom-wayland/tmp-glibc/work/<TARGET>-qcom-linux/linux-qcom-custom/6.6/image/boot ~/work/

# modules
cp -rfd build-qcom-wayland/tmp-glibc/work/<TARGET>-qcom-linux/qcom-multimedia-image/1.0/rootfs/usr/lib/modules ~/work/
cp -rfd build-qcom-wayland/tmp-glibc/work/<TARGET>-qcom-linux/qcom-multimedia-image/1.0/rootfs/usr/lib/modules-load.d ~/work/

# firmware
cp -rfd build-qcom-wayland/tmp-glibc/work/<TARGET>-qcom-linux/qcom-multimedia-image/1.0/rootfs/usr/lib/firmware ~/work/
```

Extract the downloaded Ubuntu firmware to a convenient location; ~/ubun_firmware is used as an example here.

Prepare the setup.sh script and put it into ~/ubun_firmware/, with the following content:
```bash
#!/bin/bash

usage() {
        echo "Usage:"
        echo "$0 xxxx.img ----- mount and prepare chroot for this image"
        echo "$0 umount   ----- umount image"
}

if [[ $# != 1 ]]; then
        usage
        exit 1
fi

img=$1
mkdir mnt &> /dev/null

do_losetup() {
        dev=$(sudo losetup --sector-size=4096 --find --show --partscan $img)
        if [[ -n $dev ]]; then
                echo "Modify image: $img"
                echo "======================="
                return 0
        else
                return 1
        fi
}

do_mount() {
        if [[ -b "${dev}p3" ]]; then
                sudo mount "${dev}p3" mnt || return 1
                sudo mount "${dev}p1" mnt/boot/efi || return 1
        else
                sudo mount "${dev}" mnt || return 1
        fi
        return 0
}

prepare_chroot() {
        sudo mount --bind /dev mnt/dev || return 1
        sudo mount --bind /dev/pts mnt/dev/pts || return 1
        sudo mount -t proc proc mnt/proc || return 1
        sudo mount -t sysfs sysfs mnt/sys || return 1
        echo "chroot ready !"
        echo "Run \"sudo chroot mnt /bin/bash\" to start."
        return 0
}

do_umount() {
        sudo umount -R mnt && sudo losetup -D
}

case $1 in
        *.img) do_losetup && do_mount && prepare_chroot ;;
        umount) do_umount ;;
        *) usage ;;
esac
```

The final layout looks like this:
```
~/work/
├── boot
├── dtb.bin
├── firmware
├── modules
└── modules-load.d

~/ubun_firmware/
├── aop.mbn
├── cpucp.elf
......
├── prog_firehose_ddr.elf
├── prog_firehose_lite.elf
├── rawprogram0.xml
......
├── setup.sh
......
```

> **Then copy both ~/work and ~/ubun_firmware to the device. The following steps must be performed on an arm64 device to avoid errors caused by architecture mismatch.**

### Start Porting

#### 1. Copy Device Tree
```bash
cp ~/work/dtb.bin ~/ubun_firmware/
```

#### 2. Mount the Filesystem

Use the setup.sh script to mount:

```bash
cd ~/ubun_firmware
./setup.sh system.img
```

If all goes well, the filesystem is now mounted under the current mnt directory.

#### 3. Copy Files

**Copying files must be done with root privileges!**

```bash
# Copy kernel related files
sudo cp ~/work/boot/System.map* ~/ubun_firmware/mnt/boot/
sudo cp ~/work/boot/config-* ~/ubun_firmware/mnt/boot/
sudo cp ~/work/boot/Image-* ~/ubun_firmware/mnt/boot/vmlinuz-*

# Remove the old modules and firmware
sudo rm -rf ~/ubun_firmware/mnt/usr/lib/modules
sudo rm -rf ~/ubun_firmware/mnt/usr/lib/modules-load.d
sudo rm -rf ~/ubun_firmware/usr/lib/firmware

# Copy the new modules and firmware
sudo cp -rfd ~/work/modules ~/ubun_firmware/mnt/usr/lib/
sudo cp -rfd ~/work/modules-load.d ~/ubun_firmware/mnt/usr/lib/
sudo cp -rfd ~/work/firmware ~/ubun_firmware/mnt/usr/lib/

# Make sure the file changes take effect
sync
```

#### 4. Chroot Customization (Optional)

If further customization is needed, you can chroot into the filesystem to make changes.
```bash
cd ~/ubun_firmware/

# Back up and copy resolv.conf
sudo mv mnt/etc/resolv.conf mnt/etc/resolv.conf.bk
sudo cp /etc/resolv.conf mnt/etc/

# Chroot into the filesystem
sudo chroot mnt /bin/bash
```

Now you can install software with apt or perform other operations. When the customization is finished:
```bash
# Restore resolv.conf
rm /etc/resolv.conf
mv /etc/resolv.conf.bk /etc/resolv.conf

# Exit chroot
exit

# Make sure the file changes take effect
sync
```

#### 5. Unmount

Use the setup.sh script to unmount:
```bash
./setup.sh umount
```

The porting is now complete. ~/ubun_firmware becomes the new Ubuntu firmware, ready for flashing.