## Compile Ubuntu firmware

This chapter introduces the compilation process of Ubuntu firmware. It is recommended to develop under Ubuntu 20.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

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

There are configuration files for different board in `device/rockchip/rk3576/`, select the configuration file:

```bash
./build.sh 
```

#### Build

##### Automatic compilation

* Download: [Rootfs](https://en.t-firefly.com/doc/download/140.html)，put in SDK path

```bash
7z x Ubuntu[xx.xx]-xxxx_RK3576_vx.x.xx_xxxxxxxx.7z
mkdir prebuilt_rootfs/
mv Ubuntu[xx.xx]-xxxx_RK3576_vx.x.xx_xxxxxxxx.img prebuilt_rootfs/rk3576_ubuntu_rootfs.img
```

* start compiling
```bash
./build.sh
```
the firmware will be saved to the directory `output/update/`.

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

* Download: [Rootfs](https://en.t-firefly.com/doc/download/140.html)，put in SDK path

```bash
7z x Ubuntu[xx.xx]-xxxx_RK3576_vx.x.xx_xxxxxxxx.7z
mkdir prebuilt_rootfs/
mv Ubuntu[xx.xx]-xxxx_RK3576_vx.x.xx_xxxxxxxx.img prebuilt_rootfs/rk3576_ubuntu_rootfs.img
```

* Pack the firmware, the firmware will be saved to the directory `output/update/`.

```bash
./build.sh updateimg
```