## Build Ubuntu Firmware

### Preparations

Run these two commands to install necessary tools
```bash
sudo apt update

sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev p7zip-full
```

Download rootfs here [Ubuntu rootfs(64-bit) Kernel6.1](https://en.t-firefly.com/doc/download/301.html), please use rootfs under kernel-6.1 folder.

After download, decompress and move the rootfs image to SDK/prebuilt_rootfs/, then create a symbolic link.
```bash
# Decompress
7z x Ubuntu22.04-xxxx.7z

mkdir ./SDK/prebuilt_rootfs/
mv Ubuntu22.04-xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf Ubuntu22.04-xxxx.img rk3588_ubuntu_rootfs.img
cd ..
```


### Configurations

Run `./build.sh lunch` to list all available configs, enter a number to select. Please choose one with "ubuntu" keyword:
```bash
./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rk3588_linux6.1_release_20250114_v1.1.1c

Log colors: message notice warning error fatal

Log saved at /home2/daijh/p/rk3588/sdk_6.1/output/sessions/2025-01-14_14-39-08
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3588_aio-3588jd4_buildroot_defconfig
3. firefly_rk3588_aio-3588jd4_debian_defconfig
4. firefly_rk3588_aio-3588jd4_ubuntu_defconfig
5. firefly_rk3588_aio-3588l_debian_defconfig
6. firefly_rk3588_aio-3588l_ubuntu_defconfig
7. firefly_rk3588_aio-3588q_debian_defconfig
8. firefly_rk3588_aio-3588q_ubuntu_defconfig
9. firefly_rk3588_aio-3588sjd4_debian_defconfig
10. firefly_rk3588_aio-3588sjd4_ubuntu_defconfig
11. firefly_rk3588_itx-3588j_buildroot_defconfig
12. firefly_rk3588_itx-3588j_debian_defconfig
13. firefly_rk3588_itx-3588j_ubuntu_defconfig
14. firefly_rk3588_roc-rk3588-rt-ext-10ge_debian_defconfig
15. firefly_rk3588_roc-rk3588-rt-ext-10ge_ubuntu_defconfig
16. firefly_rk3588_roc-rk3588-rt-ext-2g5e_debian_defconfig
17. firefly_rk3588_roc-rk3588-rt-ext-2g5e_ubuntu_defconfig
18. firefly_rk3588_roc-rk3588-rt_debian_defconfig
19. firefly_rk3588_roc-rk3588-rt_ubuntu_defconfig
20. firefly_rk3588_roc-rk3588s-pc-ext_debian_defconfig
21. firefly_rk3588_roc-rk3588s-pc-ext_ubuntu_defconfig
22. firefly_rk3588_roc-rk3588s-pc_debian_defconfig
23. firefly_rk3588_roc-rk3588s-pc_ubuntu_defconfig
24. rockchip_rk3588_evb1_lp4_v10_defconfig
25. rockchip_rk3588_evb7_v11_defconfig
26. rockchip_rk3588_ipc_evb1_v10_defconfig
27. rockchip_rk3588_ipc_evb7_lp4_v11_defconfig
28. rockchip_rk3588_multi_ipc_evb1_v10_defconfig
29. rockchip_rk3588s_evb1_lp4x_v10_defconfig
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