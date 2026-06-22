## Compile Ubuntu firmware

This chapter introduces the compilation process of Ubuntu firmware. It is recommended to develop under Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

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

#### Precompile Configuration

There are configuration files for different board in `device/rockchip/rk3588/`, select the configuration file:

```bash
./build.sh 

or

./build.sh  # MIPI DSI

```

#### Build

##### Automatic compilation

* Download: [Ubuntu rootfs(64-bit)]()，put in SDK path

```bash
7z x ubuntu-aarch64-rootfs.7z
mkdir ubuntu_rootfs
mv ubuntu-aarch64-rootfs.img ubuntu_rootfs/rootfs.img
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
./build.sh extboot
```

* recovery

```bash
./build.sh recovery
```

* Download: [Ubuntu rootfs(64-bit)]()，put in SDK path

```bash
7z x ubuntu-aarch64-rootfs.7z
mkdir ubuntu_rootfs
mv ubuntu-aarch64-rootfs.img ubuntu_rootfs/rootfs.img
```

* Update each part of the `.img` link to the directory `rockdev/`:

```bash
./mkfirmware.sh
```

* Pack the firmware, the firmware will be saved to the directory `rockdev/pack/`.

```bash
./build.sh updateimg
```