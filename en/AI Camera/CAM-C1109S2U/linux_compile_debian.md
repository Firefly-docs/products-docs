## Compile Debian firmware

This chapter introduces the compilation process of Debian firmware. It is recommended to develop under Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

<font color=red>**The compilation portion of this tutorial works with SDK versions above v2.2.5f**</font>

```
$ realpath .repo/manifest.xml
/home/lvsx/project/rv1126_rv1109/.repo/manifests/rv1126_rv1109/rv1126_rv1109_linux_release_20240117_v2.2.5f.xml
```
### Preparatory work

#### Set up compilation environment

```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip \
device-tree-compiler ncurses-dev \
```

### Compile SDK

#### Pre-compilation configuration

In the `device/rockchip/rv1126_rv1109/` directory, there are configuration files for different board types. Select the configuration file:

#### Compile Debian firmware
* Pre-compilation configuration for V2 mipi screen (old version screen, which has been discontinued)
```
./build.sh cam-crv1109s2u-facial_gate-BE-45-debian.mk
```
* Compile V3 mipi screen configuration

```bash
./build.sh cam-crv1109s2u-facial_gate-mipi101-BSD1218-A101KL68-debian.mk
```

#### Build

##### Automatic compilation

* Download: [Debian rootfs(32-bit)](https://en.t-firefly.com/doc/download/page/id/85.html)，put in SDK path

```bash
7z x debian10_rootfs_2022_12_22.7z
mkdir ubuntu_rootfs
mv rootfs.img ubuntu_rootfs/rootfs.img
```

* start compiling
```bash
./build.sh
```
the firmware will be saved to the directory `rockdev/pack/`.

##### Partial compilation

* u-boot

```bash
./build.sh uboot
```

* kernel

```bash
./build.sh kernel
```

* recovery

```bash
./build.sh recovery
```

* Download: [Debian rootfs(32-bit)]()，put in SDK path

```bash
7z x debian10_rootfs_2022_12_22.7z
mkdir ubuntu_rootfs
mv rootfs.img ubuntu_rootfs/rootfs.img
```

* Update each part of the `.img` link to the directory `rockdev/`:

```bash
./mkfirmware.sh
```

* Pack the firmware, the firmware will be saved to the directory `rockdev/pack/`.

```bash
./build.sh updateimg
```