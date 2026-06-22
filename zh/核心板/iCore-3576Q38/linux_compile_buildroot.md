## 编译 Buildroot 固件

### 准备工作

执行下面两条命令来安装需要的工具
```bash
sudo apt update

sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev
```

### 编译前配置

执行 `./build.sh lunch` 列出所有可用的配置文件，输入编号选择对应的配置文件，注意要选择带有 buildroot 字样的配置文件：
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

### 编译

#### 完整编译

执行如下命令即可
```bash
./build.sh all
```

编译完成后会生成完整固件 `output/update/update.img`

#### 部分编译

一般来说自定义开发只涉及 kernel 和 uboot。因此，在进行过一次完整编译后，如果后续又修改了 kernel 或 uboot，可以部分编译来加快速度。

* 单独编译 u-boot，生成 u-boot/uboot.img

```bash
./build.sh uboot
```

* 单独编译 kernel，生成 kernel/extboot.img

```bash
./build.sh extboot
```

* 将各部件打包成 update.img

```bash
./build.sh updateimg
```

但是如果你修改了 buildroot，需要重新进行完整编译。