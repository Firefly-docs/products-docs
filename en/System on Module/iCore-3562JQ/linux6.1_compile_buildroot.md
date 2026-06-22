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

Manifest: rk3562_linux6.1_release_20241220_v1.0.0a.xml

Log colors: message notice warning error fatal

Log saved at /home4/lth/project/linux/rk3562_6.1/output/sessions/2024-12-25_18-03-39
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3562_aio-3562jq_buildroot_defconfig
3. firefly_rk3562_aio-3562jq_debian_defconfig
4. firefly_rk3562_aio-3562jq_ubuntu_defconfig
5. rockchip_rk3562_dictpen_test3_v20_defconfig
6. rockchp_rk3562_evb1_lp4x_v10_amp_defconfig
7. rockchip_rk3562_evb1_lp4x_v10_defconfig
8. rockchip_rk3562_evb2_ddr4_v10_amp_defconfig
9. rockchip_rk3562_evb2_ddr4_v10_defconfig
10. rockchip_rk3562_robot_evb1_lp4x_v10_defconfig
11. rockchip_rk3562_robot_evb2_ddr4_v10_defconfig
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