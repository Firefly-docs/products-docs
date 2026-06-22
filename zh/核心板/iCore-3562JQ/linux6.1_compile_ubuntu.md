## 编译 Ubuntu 固件

### 准备工作

执行下面两条命令来安装需要的工具
```bash
sudo apt update

sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev p7zip-full
```

下载根文件系统：[Ubuntu 根文件系统](https://www.t-firefly.com/doc/download/242.html#other_774)，请使用网盘中 kernel-6.1 目录下的文件系统。

下载后将文件系统解压到 SDK/prebuilt_rootfs/ 下，并创建链接

```bash
# 解压
7z x Ubuntu22.04-Xfce_xxxx_xxxx.7z

# 将解压后的 rootfs 镜像移动到 sdk 并创建一个符号链接
mv Ubuntu22.04-Xfce_xxxx_xxxx.img ./rk3562_6.1/prebuilt_rootfs/
cd ./rk3562_6.1/prebuilt_rootfs/
ln -sf Ubuntu22.04-Xfce_xxxx_xxxx.img rk3562_ubuntu_rootfs.img
cd ..
```

### 编译前配置

执行 `./build.sh lunch` 列出所有可用的配置文件，输入编号选择对应的配置文件，注意要选择带有 ubuntu 字样的配置文件：
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