# Compile Android Firmware

## Prerequisites [step]

Building the Android firmware requires a powerful host. Confirm the following conditions before compiling:

* 64-bit CPU, Ubuntu 20.04 or later
* 64 GB physical memory plus swap
* 400GB free disk space
* Run the build commands as a normal user, not root

For installing the build dependencies, refer to [Set Up the Build Environment](android_prepare_compile_android.md).

## Download the SDK [step]

The Android 16 SDK of Firefly-R3572A is provided by email. Send the order number to sales@t-firefly.com and state the required SDK name.

The SDK source archive is split into volumes, including `firefly_rk3572_android16.0_git_20261112.7z.001` and the following parts. Download all volumes into the same directory before extraction.

<Callout title="Note" type="info">

For the source archive name and the bundle name in this section, the actual file names, volume count and MD5 checksums are subject to the released SDK package.

</Callout>

Verify the MD5 checksums before extraction:

```bash
md5sum /path/to/firefly_rk3572_android16.0_git_20261112.7z.001
...
```

Check the remaining volumes as well, and extract only after all checksums match those provided with the release:

```bash
cd ~/proj/
7z x ./firefly_rk3572_android16.0_git_20261112.7z.001 -oRK3572_Android16.0
cd ./RK3572_Android16.0
git reset --hard
```

<Callout title="Note" type="warn">

Do not extract the SDK in a shared folder, a mounted directory or a path containing non-English characters, to avoid unnecessary errors.

</Callout>

## Update the SDK [step]

The SDK is updated through a bundle repository. Download the `rk357x-android16-bundle` archive from the resource download page, then run in the SDK root directory:

```bash
7z x rk357x-android16-bundle.7z* -r -o. && mv rk357x-android16-bundle .bundle
.bundle/update
git rebase FETCH_HEAD
```

`.bundle/update` fetches the updates into `FETCH_HEAD`, and `git rebase FETCH_HEAD` merges them into the current branch.

## Select the Product Configuration [step]

Select the product configuration for Firefly-R3572A through `lunch` before compiling. The name has three parts and must not omit `bp4a`.

Run the following commands in the SDK root directory:

```bash
source build/envsetup.sh
lunch rk3572_firefly_r3572a-bp4a-userdebug
```

## Full Build [step]

A full build compiles U-Boot, the kernel and Android and collects the images in one pass, then packs the unified firmware separately. It is suitable for the first build or a complete verification. Run the following in the SDK root directory:

```bash
./FFTools/make.sh -j8 -d rk3572-firefly-r3572a -l rk3572_firefly_r3572a-bp4a-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3572_firefly_r3572a-bp4a-userdebug
```

The command has two parts:

* `./FFTools/make.sh -j8 -d <dts> -l <lunch>`: the full build entry, which sequentially builds U-Boot, the kernel and Android and collects the images. `-d` specifies the kernel device tree (`rk3572-firefly-r3572a`), and `-l` specifies the three-part lunch (`rk3572_firefly_r3572a-bp4a-userdebug`). With `-l` but no stage argument, all three stages are built; this is not a single-module entry.
* `./FFTools/mkupdate/mkupdate.sh -l <lunch>`: packs the unified firmware, wrapping existing images only without compiling or cleaning the build directory.

## Partial Build [step]

When only one part needs to be rebuilt, enter the corresponding directory and build it separately instead of doing a full build each time. The following commands all start from the SDK root directory.

### Build the Kernel [step]

In the `kernel-6.12` directory, use `mkboot.sh` to build the kernel separately and generate `boot.img`:

```bash
cd kernel-6.12
./mkboot.sh rk3572-firefly-r3572a ../rockdev/Image-rk3572_firefly_r3572a/boot.img rk3572.config
```

The usage of `mkboot.sh` is `./mkboot.sh <dts name> <existing boot.img path> <chip config>`: the first argument is the kernel device tree `rk3572-firefly-r3572a`; the second is the path of an **existing boot.img** (used as input — the script replaces the kernel and resource into it, it is not an output path); the third is the chip config `rk3572.config`. The build produces `kernel-6.12/boot.img`.

### Build U-Boot [step]

In the `u-boot` directory, use `make.sh` to build U-Boot separately:

```bash
cd u-boot
./make.sh rk3572
```

### Build Android [step]

In the SDK root directory, build the Android upper layer:

```bash
source build/envsetup.sh
lunch rk3572_firefly_r3572a-bp4a-userdebug
make installclean
make -j8
./mkimage.sh
```

`./mkimage.sh` repacks `boot.img` and `super.img` and copies the related images to the product directory under `rockdev/`.

<Callout title="Note" type="warn">

1. To build the kernel or modules separately, use `mkboot.sh` / `make.sh` above, not the `-K` / `-A` arguments of `FFTools/make.sh` — those are stage arguments of the full build entry and have different semantics.
2. After building a part separately, run `./FFTools/mkupdate/mkupdate.sh -l rk3572_firefly_r3572a-bp4a-userdebug` again to repack the unified firmware if needed.
3. The build toolchain is provided by the Android 16 SDK (Clang, Rust, kernel-build-tools, libelf / libdw); use the SDK's own environment.

</Callout>

## Pack the Unified Firmware update.img [step]

After the build, pack the unified firmware with the official Firefly script:

```bash
./FFTools/mkupdate/mkupdate.sh -l rk3572_firefly_r3572a-bp4a-userdebug
```

The unified firmware is generated in the `rockdev/Image-rk3572_firefly_r3572a/` directory, with a file name in the format `product_version_date.img`, for example `rk3572_firefly_r3572a_Android16_date.img` (subject to the actual output).

Packing the unified firmware `update.img` on Windows is also simple: copy the built files into the `rockdev\Image` directory of AndroidTool, then run the `mkupdate.bat` batch file in the `rockdev` directory. This creates `update.img` and stores it in the `rockdev\Image` directory.

## Build Output [step]

After building and packing, the partition images are output to the `rockdev/` directory. Common partition images are:

| Firmware | Description |
| --- | --- |
| boot.img | contains ramdisk, kernel and dtb |
| dtbo.img | Device Tree Overlays |
| MiniLoaderAll.bin | contains the first-stage loader |
| misc.img | contains the recovery-wipe boot flag |
| parameter.txt | contains the partition layout |
| recovery.img | contains recovery-ramdisk, kernel and dtb |
| super.img | contains odm, product, vendor, system and system_ext |
| uboot.img | contains the U-Boot firmware |
| vbmeta.img | contains AVB verification information |
| update.img | contains the above images for flashing the whole firmware |

The image output directory of Firefly-R3572A is `rockdev/Image-rk3572_firefly_r3572a/`, expected to contain `boot.img`, `super.img`, `update.img` and others.

Confirm the output with `ls rockdev/`. For flashing, refer to [USB Firmware Upgrade](../../getting_started/firmware_upgrade/upgrade_firmware.md).

## OTA Build [step]

### Full OTA Package [step]

Append the `-o` argument to the full build command to perform a global OTA build and generate a full OTA package:

```bash
./FFTools/make.sh -j8 -d rk3572-firefly-r3572a -l rk3572_firefly_r3572a-bp4a-userdebug -o
```

The generated full OTA package is located at:

```bash
out/target/product/rk3572-firefly-r3572a/rk3572-firefly-r3572a-ota-eng.xxx.zip
```

The base target-files package (used to generate a differential package) is located at:

```bash
out/target/product/rk3572-firefly-r3572a/obj/PACKAGING/target_files_intermediates/rk3572-firefly-r3572a-target_files-eng.xxx.zip
```

Copy out the base package and rename it for later differential package creation:

```bash
mkdir -p ~/ota
cp out/target/product/rk3572-firefly-r3572a/obj/PACKAGING/target_files_intermediates/rk3572-firefly-r3572a-target_files-eng.xxx.zip ~/ota/v1.zip
```

### OTA Differential Package [step]

A differential package requires two versions of the base package. First obtain the old `v1.zip` as above, then modify the kernel DTS or Android code and run the `-o` build again to generate a new base package, copied out as `v2.zip`:

```bash
./FFTools/make.sh -j8 -d rk3572-firefly-r3572a -l rk3572_firefly_r3572a-bp4a-userdebug -o
cp out/target/product/rk3572-firefly-r3572a/obj/PACKAGING/target_files_intermediates/rk3572-firefly-r3572a-target_files-eng.xxx.zip ~/ota/v2.zip
```

Then generate the differential package:

```bash
out/host/linux-x86/bin/ota_from_target_files --block -v -i ~/ota/v1.zip -p out/host/linux-x86/ \
    -k build/target/product/security/testkey ~/ota/v2.zip update.zip
```

Put the generated `update.zip` in the device's `/sdcard/` directory and reboot; the system will detect it and prompt to complete the OTA upgrade (full and differential packages are applied in the same way, by placing `update.zip` in `/sdcard/` and rebooting).

<Callout title="Note" type="info">

The OTA commands and paths above depend on the actual output of the Firefly-R3572A Android 16 SDK. For setting up an OTA server, refer to the OTA documentation provided with the SDK.

</Callout>

## FAQ [step]

### Q1: git clone of the remote bundle repository fails while updating the SDK

A1: Increase the buffer size and retry:

```bash
git config --global https.postBuffer 1048576000
```