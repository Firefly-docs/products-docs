## Compile Buildroot firmware

This chapter introduces the compilation process of Buildroot firmware. It is recommended to develop under Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

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

There are configuration files for different board in `device/rockchip/rv1126_rv1109/`, select the configuration file:

`rv1126_rv1109_linux_release` SDK and `rv1126_rv1109_linux_bsp_release` SDK Select configuration file:
```bash



# V2 MIPI screen (old version screen, this screen has been discontinued)
./build.sh aio-rv1109-jd4-BE-45.mk 

# V3 MIPI screen
./build.sh aio-rv1109-jd4-mipi101-BSD1218-A101KL68.mk


```

`rv1126_rv1109_linux_ai_camera_release` SDK and `rv1126_rv1109_linux_ai_camera_bsp_release` SDK select configuration file:


`CORE-1126-JD4/CORE-1109-JD4` series products are not very compatible with UVC function. Customers with the ability are requested to choose this configuration file for compilation.

```bash
./build.sh aio-rv1109-uvcc.mk 
```

#### Build

##### Automatic compilation

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

* buildroot

```bash
./build.sh rootfs
```

* Update each part of the `.img` link to the directory `rockdev/`:

```bash
./mkfirmware.sh
```

* Pack the firmware, the firmware will be saved to the directory `rockdev/pack/`.

```bash
./build.sh updateimg
```