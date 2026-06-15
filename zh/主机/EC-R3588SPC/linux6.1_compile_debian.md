## 编译 Debian 固件

### 准备工作

执行下面两条命令来安装需要的工具
```bash
sudo apt update

sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev p7zip-full
```

下载根文件系统：[Debian 根文件系统(64位)](https://www.t-firefly.com/doc/download/161.html#other_717)，请使用网盘中 kernel-6.1 目录下的文件系统。

下载后将文件系统解压到 SDK/prebuilt_rootfs/ 下，并创建链接

```bash
# 解压
7z x debian12_xxxx_rootfs_xxxx.7z

# 将解压后的 rootfs 镜像移动到 sdk 并创建一个符号链接
mkdir ./SDK/prebuilt_rootfs/
mv debian12_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf debian12_xxxx_rtoofs_xxxx.img rk3588_debian_rootfs.img
cd ..
```

### 编译前配置

执行 `./build.sh lunch` 列出所有可用的配置文件，输入编号选择对应的配置文件，注意要选择带有 debian 字样的配置文件：
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