## Build Buildroot Firmware

### Preparations

Run these two commands to install necessary tools
```bash
sudo apt update

sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev
```

### Configurations

Run `./build.sh lunch` to list all available configs, enter a number to select. Please choose one with "buildroot" keyword:
```bash
./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rk3588_linux6.1_release_20241203_v1.1.1b.xml

Log colors: message notice warning error fatal

Log saved at /home4/lth/project/linux/rk3588_6.1_release/rk3588_6.1/output/sessions/2024-12-11_09-15-44
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3588_aio-3588l_debian_defconfig
3. firefly_rk3588_aio-3588q_debian_defconfig
4. firefly_rk3588_aio-3588sjd4_debian_defconfig
5. firefly_rk3588_itx-3588j_buildroot_defconfig
6. firefly_rk3588_itx-3588j_debian_defconfig
7. firefly_rk3588_roc-rk3588-rt-ext-10ge_debian_defconfig
8. firefly_rk3588_roc-rk3588-rt-ext-2g5e_debian_defconfig
9. firefly_rk3588_roc-rk3588-rt_debian_defconfig
10. firefly_rk3588_roc-rk3588s-pc-ext_debian_defconfig
11. firefly_rk3588_roc-rk3588s-pc_debian_defconfig
12. rockchip_rk3588_evb1_lp4_v10_defconfig
13. rockchip_rk3588_evb7_v11_defconfig
14. rockchip_rk3588_ipc_evb1_v10_defconfig
15. rockchip_rk3588_ipc_evb7_lp4_v11_defconfig
16. rockchip_rk3588_multi_ipc_evb1_v10_defconfig
17. rockchip_rk3588s_evb1_lp4x_v10_defconfig
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

But if you modified buildroot, you need to do full compilation.