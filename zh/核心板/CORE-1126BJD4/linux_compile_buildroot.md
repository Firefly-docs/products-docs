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

#### 编译前配置

下载 SDK 后，只有第一次编译需要执行 ./build.sh 选择编译的芯片平台。不是第一次编译则跳过该步骤。
```
$ ./build.sh

############### Rockchip Linux SDK ###############

Manifest: rv1126b_linux6.1_release_20251212_v1.1.0a.xml

Log colors: message notice warning error fatal

Log saved at /home2/lvsx/project/rv1126b/output/sessions/2025-12-25_15-13-14
Pick a chip:

1. rk3562
2. rk356x
3. rk3576
4. rk3588
5. rv1126b
Which would you like? [1]:
```
**输入编号选择平台 rv1126b 。**


#### 选择板型配置

执行 `./build.sh lunch` 列出所有可用的配置文件，输入编号选择对应的配置文件，注意要选择带有 buildroot 字样的配置文件：
```bash
$ ./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rv1126b_linux6.1_release_20251212_v1.1.0a.xml

Log colors: message notice warning error fatal

Log saved at /home2/lvsx/project/rv1126b/output/sessions/2025-12-25_15-00-32
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rv1126b_aio-1126bjd4_buildroot_defconfig
3. firefly_rv1126b_aio-1126bjd4_debian_defconfig
4. firefly_rv1126b_aio-1126bjd4_ubuntu_defconfig
5. firefly_rv1126b_aio-1126bjd4v0_buildroot_defconfig
6. firefly_rv1126b_aio-1126bjd4v0_debian_defconfig
7. firefly_rv1126b_aio-1126bjd4v0_ubuntu_defconfig
8. firefly_rv1126b_aio-1126bq38_buildroot_defconfig
9. firefly_rv1126b_aio-1126bq38_debian_defconfig
10. firefly_rv1126b_aio-1126bq38_ubuntu_defconfig
11. firefly_rv1126b_cam-1126bq38_buildroot_defconfig
12. firefly_rv1126b_cam-1126bq38_debian_defconfig
13. firefly_rv1126b_cam-1126bq38_ubuntu_defconfig
...
Which would you like? [1]:
```

**输入编号选择板型： firefly_rv1126b_aio-1126bjd4_buildroot_defconfig。**

### 编译

#### 完整编译

执行如下命令即可
```bash
./build.sh all
```

编译完成后会生成完整固件 `output/update/update.img`

#### 部分编译

进行过一次完整编译后，如果后续又修改了 kernel 或 uboot，可以部分编译来加快速度。

* 单独编译 u-boot，生成 u-boot/uboot.img

```bash
./build.sh uboot
```

* 单独编译 kernel，生成 kernel/boot.img

```bash
./build.sh kernel
```

* 单独编译 recovery，生成 output/recovery/ramboot.img

```bash
./build.sh recovery
```

* 将各部件打包成 update.img

```bash
./build.sh updateimg
```

但是如果你修改了 buildroot，需要重新进行完整编译。