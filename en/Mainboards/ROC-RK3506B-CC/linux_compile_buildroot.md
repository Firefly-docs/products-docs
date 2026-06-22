## Compile Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware. It is recommended to develop under Ubuntu 22.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

<font color=red>**The compilation portion of this tutorial works with SDK versions above  v1.0.0a**</font>

```
$ readlink -f .repo/manifest.xml
/home2/lvsx/project/rk3506/.repo/manifests/rk3506_linux_release_20250117_v1.0.0a.xml
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

There are configuration files for different board in `device/rockchip/rk3506/`, select the configuration file:

```bash
./build.sh firefly_rk3506_roc-rk3506b-cc-emmc_buildroot_defconfig # EMMC interface (the default is EMMC interface)

or

./build.sh firefly_rk3506_roc-rk3506b-cc-spi-flash_buildroot_defconfig # SPI FLASH interface (need to paste SPI FLASH)
```

#### Build

##### Automatic compilation

* start compiling
```bash
./build.sh
```
the firmware will be saved to the directory `output/update`.

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

* buildroot

```bash
./build.sh buildroot
```

* Pack the firmware, the firmware will be saved to the directory `output/update`.

```bash
./build.sh updateimg
```