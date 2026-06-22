## Compile Debian firmware

### Preparatory work

Execute the following two commands to install the required tools.
```bash
sudo apt update

sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev p7zip-full
```

Download the root filesystem: [Debian Root Filesystem (64-bit)](https://en.t-firefly.com/doc/download/346.html#other_623). Please use the filesystem located in the kernel-6.1 directory of the cloud drive.

After downloading, extract the file system to SDK/prebuilt_rootfs/ and create a symbolic link:

```bash
# Extract
7z x debian12_xxxx_rootfs_xxxx.7z

# Move the extracted rootfs image to sdk and create a symbolic link
mkdir ./SDK/prebuilt_rootfs/
mv debian12_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf debian12_xxxx_rtoofs_xxxx.img rv1126b_debian_rootfs.img
cd ..
```

#### Pre-compilation Configuration

After downloading the SDK, you only need to execute `./build.sh` to select the chip platform for compilation during the first compilation. Skip this step if it's not the first compilation.

```bash
$ ./build.sh

############### Rockchip Linux SDK ###############

Manifest: rv1126b_linux6.1_release_20251212_v1.1.0a.xml

Log colors: message notice warning error fatal

Log saved at /home2/lvsx/project/rv1126b/output/sessions/2025-12-25_15-13-14
Pick a chip:

1. rk3562
2. rk356x
3. rk3576
4. rk3588
5. rv1126b
Which would you like? [1]:
```

**Enter the ID and select platform rv1126b.**

#### Select Board Configuration

Execute `./build.sh lunch` to list all available configuration files. Enter the file number to select the corresponding configuration file. Note that you should select the configuration file containing "debian":

```bash
$ ./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rv1126b_linux6.1_release_20251212_v1.1.0a.xml

Log colors: message notice warning error fatal

Log saved at /home2/lvsx/project/rv1126b/output/sessions/2025-12-25_15-00-32
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rv1126b_aio-1126bjd4_buildroot_defconfig
3. firefly_rv1126b_aio-1126bjd4_debian_defconfig
4. firefly_rv1126b_aio-1126bjd4_ubuntu_defconfig
5. firefly_rv1126b_aio-1126bjd4v0_buildroot_defconfig
6. firefly_rv1126b_aio-1126bjd4v0_debian_defconfig
7. firefly_rv1126b_aio-1126bjd4v0_ubuntu_defconfig
8. firefly_rv1126b_aio-1126bq38_buildroot_defconfig
9. firefly_rv1126b_aio-1126bq38_debian_defconfig
10. firefly_rv1126b_aio-1126bq38_ubuntu_defconfig
11. firefly_rv1126b_cam-1126bq38_buildroot_defconfig
12. firefly_rv1126b_cam-1126bq38_debian_defconfig
13. firefly_rv1126b_cam-1126bq38_ubuntu_defconfig
...
Which would you like? [1]:
```

**Enter the board type by entering the board number: firefly_rv1126b_aio-1126bjd4_debian_defconfig.**

### Compilation

#### Complete Compilation

Execute the following command:

```bash
./build.sh all
```

After compilation, a complete firmware `output/update/update.img` will be generated.

##### Partial Compilation

Generally, custom development only involves the kernel and uboot. Therefore, after a complete compilation, if the kernel or uboot is subsequently modified, partial compilation can be performed to speed up the process.

* Compile u-boot separately, generating u-boot/uboot.img

```bash
./build.sh uboot
```

* Compile kernel separately, generating kernel/boot.img

```bash
./build.sh kernel
```

* Compile recovery separately, generating output/recovery/ramboot.img

```bash
./build.sh recovery
```

* Package all components into update.img

```bash
./build.sh updateimg
```