## 编译 Debian 固件

本章介绍 Debian 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

<font color=red>**本教程的编译部分适用于 v2.2.5f 以上 SDK 版本**</font>

```
$ realpath .repo/manifest.xml
/home/lvsx/project/rv1126_rv1109/.repo/manifests/rv1126_rv1109/rv1126_rv1109_linux_release_20240117_v2.2.5f.xml
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

在 `device/rockchip/rv1126_rv1109/` 目录下，有不同板型的配置文件，选择配置文件：

#### 编译 Debian 固件
* 编译 V2 mipi 屏幕前配置（旧版本屏幕，该屏幕已停产）
```
./build.sh cam-crv1109s2u-facial_gate-BE-45-debian.mk
```

* 编译 V3 mipi 屏幕前配置

```bash
./build.sh cam-crv1109s2u-facial_gate-mipi101-BSD1218-A101KL68-debian.mk
```

#### 编译

##### 全自动编译

* 下载根文件系统：[Debian 根文件系统(32位)](https://www.t-firefly.com/doc/download/101.html#other_693)，放到 SDK 路径下

```bash
7z x debian10_rootfs_2022_12_22.7z
mkdir ubuntu_rootfs
mv rootfs.img ubuntu_rootfs/rootfs.img
```

* 开始编译
```bash
./build.sh
```

生成的完整固件会保存到 `rockdev/pack/` 目录。

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

* 下载根文件系统：[Debian 根文件系统(32位)](https://www.t-firefly.com/doc/download/161.html)，放到 SDK 路径下

```bash
7z x debian10_rootfs_2022_12_22.7z
mkdir ubuntu_rootfs
mv rootfs.img ubuntu_rootfs/rootfs.img
```

* 更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

* 打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```