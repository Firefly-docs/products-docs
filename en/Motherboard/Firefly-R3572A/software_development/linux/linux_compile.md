# Compile Linux Firmware

## Prerequisites [step]

Compile the Linux firmware on an X86_64 Ubuntu host. The SDK uses cross-compilation and cannot be built on the board. Confirm the following conditions before compiling:

* OS: Ubuntu 20.04 or later (physical machine or Docker container)
* Disk space: 100 GB or more recommended
* Memory: 16 GB or more recommended
* Run the build commands as a normal user, not root

## Obtain the SDK [step]

Contact the sales team (sales@t-firefly.com) to obtain the Firefly-R3572A Linux SDK download link.

Extract the SDK archive into a normal user directory. The extracted root directory is referred to as the SDK root directory below, and all commands in this section are executed there.

<Callout title="Note" type="warn">

1. Do not extract the SDK in a virtual machine shared folder, as symbolic links may fail to create.
2. Do not extract or compile as root, otherwise file permission errors may occur.

</Callout>

## Install Build Dependencies [step]

Run the following commands in the SDK root directory to install the build dependencies:

```bash
sudo apt-get update
sudo apt-get install git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffutils \
binfmt-support qemu-user-static live-build bison flex fakeroot \
cmake gcc-multilib g++-multilib unzip device-tree-compiler \
python3-pip libncurses5-dev build-essential
```

## Select the Board Configuration [step]

The Firefly Linux SDK loads a board-level configuration through `./build.sh <defconfig>`, named `firefly_<chip>_<board>_<os>_defconfig`. The board configurations of Firefly-R3572A are listed below:

| OS | Configuration file |
| --- | --- |
| Debian | `firefly_rk3572_r3572a_debian_defconfig` |

Run the following command in the SDK root directory to select the Debian configuration:

```bash
./build.sh firefly_rk3572_r3572a_debian_defconfig
```

The corresponding kernel device tree is `rk3572-firefly-r3572a`, whose source is located under `kernel-6.12/arch/arm64/boot/dts/rockchip/`.

<Callout title="Note" type="warn">

After switching the board or OS, select the defconfig again. Reusing an old configuration produces a firmware for the wrong target.

</Callout>

## Compile the Firmware [step]

There are two ways to build: a **full build** (for the first build, or after changing the rootfs or overall configuration) and a **partial build** (when only one module changed and you want a quick verification). Select the defconfig as described above before using either method.

### Full Build [step]

A full build sequentially builds U-Boot, the kernel, Recovery and rootfs, then packs the unified firmware `update.img`. Run the following in the SDK root directory:

```bash
./build.sh firefly_rk3572_r3572a_debian_defconfig
./build.sh all
```

Here `./build.sh all` is the complete build flow (it calls `mk-all.sh` internally). If the rootfs is a prebuilt image, place it in the SDK's `prebuilt_rootfs/` directory according to the rootfs type; refer to the SDK documentation for details.

A full build takes a long time, so allow sufficient time for the first build.

### Partial Build [step]

When only one module is modified, you can build that part alone and then repack `update.img`, without a full build each time:

| Build target | Command |
| --- | --- |
| U-Boot | `./build.sh uboot` |
| Kernel and device tree | `./build.sh extboot` |
| Recovery | `./build.sh recovery` |
| Pack the unified firmware | `./build.sh updateimg` |

For example, to rebuild only the kernel and repack the firmware, run in the SDK root directory:

```bash
./build.sh firefly_rk3572_r3572a_debian_defconfig
./build.sh extboot
./build.sh updateimg
```

## Output and Verification [step]

Build output is located in `output/firmware/` and `output/update/` of the SDK root directory: the kernel and device tree build produces `extboot.img` (under `kernel-6.12/`), while the unified firmware `update.img` and the partition images are in `output/update/`.

Confirm the output with:

```bash
ls output/update/
```

The expected output includes `update.img`, `boot.img`, `uboot.img` and `parameter.txt`. For flashing, refer to [USB Firmware Upgrade](../../getting_started/firmware_upgrade/upgrade_firmware.md).