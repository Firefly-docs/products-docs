## 编译 Buildroot 固件

本章介绍 Buildroot 固件的编译流程，推荐在 Ubuntu 22.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

<font color=red>**本教程的编译部分适用于 v1.0.0a 以上 SDK 版本**</font>

```
$ readlink -f .repo/manifest.xml
/home2/lvsx/project/rk3506/.repo/manifests/rk3506_linux_release_20250117_v1.0.0a.xml
```
### 准备工作

#### 搭建编译环境

```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip \
device-tree-compiler ncurses-dev \
```

### 编译 SDK

#### 编译前配置

在 `device/rockchip/rk3506/` 目录下，有不同板型的配置文件，选择配置文件：

```bash
./build.sh firefly_rk3506_roc-rk3506b-cc-emmc_buildroot_defconfig # EMMC 接口 （默认是 EMMC 接口）

or

./build.sh firefly_rk3506_roc-rk3506b-cc-spi-flash_buildroot_defconfig # SPI FLASH 接口（需要贴 SPI FLASH）
```

#### 编译

##### 全自动编译

全自动编译会执行上述编译、打包操作，生成 RK 固件。

```bash
./build.sh
```

打包固件，生成的完整固件会保存到 `output/update` 目录。

##### 部分编译

* 编译 u-boot

```bash
./build.sh uboot
```

* 编译 kernel

```bash
./build.sh kernel
```

* 编译 recovery

```bash
./build.sh recovery
```

* 编译 buildroot

```bash
./build.sh buildroot
```

* 打包固件，生成的完整固件会保存到 `output/update` 目录。

```bash
./build.sh updateimg
```