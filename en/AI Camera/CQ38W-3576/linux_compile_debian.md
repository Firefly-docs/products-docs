## Build Debian Firmware

### Preparations

Run these two commands to install necessary tools
```bash
sudo apt update

sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev p7zip-full
```

Download rootfs here [Debian rootfs](https://www.t-firefly.com/doc/download/327.html#other_791), in general, choose the filesystem with latest date.

After download, decompress and move the rootfs image to SDK/prebuilt_rootfs/, then create a symbolic link.
```bash
# Decompress
7z x Debian12-xxxx.7z

mkdir ./SDK/prebuilt_rootfs/
mv Debian12-xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf Debian12-xxxx.img rk3576_debian_rootfs.img
cd ..
```

### Configurations

Run `./build.sh lunch` to list all available configs, enter a number to select. Please choose one with "debian" keyword:
```bash
./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rk3576_linux_release_20250520_v1.0.1f.xml

Log colors: message notice warning error fatal

Log saved at /home2/liuth/project/linux/rk3576/output/sessions/2025-08-06_15-27-20
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3576_aio-3576c_buildroot_defconfig
3. firefly_rk3576_aio-3576c_debian_defconfig
4. firefly_rk3576_aio-3576c_ubuntu_defconfig
5. firefly_rk3576_aio-3576jd4_buildroot_defconfig
6. firefly_rk3576_aio-3576jd4_debian_defconfig
7. firefly_rk3576_aio-3576jd4_ubuntu_defconfig
......
......
24. firefly_rk3576_roc-rk3576-pc_buildroot_defconfig
25. firefly_rk3576_roc-rk3576-pc_debian_defconfig
26. firefly_rk3576_roc-rk3576-pc_ubuntu_defconfig
27. rockchip_rk3576_evb1_v10_defconfig
28. rockchip_rk3576_industry_evb_v10_defconfig
29. rockchip_rk3576_iotest_v10_defconfig
30. rockchip_rk3576_ipc_evb1_v10_defconfig
31. rockchip_rk3576_multi_ipc_evb1_v10_defconfig
32. rockchip_rk3576_test1_v10_defconfig
33. rockchip_rk3576_test2_v10_defconfig
34. rockchip_rk3576_vehicle_evb_v10_defconfig
35. rockchip_rk3576_vehicle_evb_v20_mos_defconfig
Which would you like? [1]:
```

### Compilation

#### Full Compilation

Run
```bash
./build.sh all
```

The output firmware will be `output/update/update.img`

#### Partial Compilation

After a full compilation, if you modified kernel or uboot, you can do partial compilation to save time.

* Only build u-boot, will generate u-boot/uboot.img

```bash
./build.sh uboot
```

* Only build kernel, will generate kernel/extboot.img

```bash
./build.sh extboot
```

* Pack all parts to update.img

```bash
./build.sh updateimg
```