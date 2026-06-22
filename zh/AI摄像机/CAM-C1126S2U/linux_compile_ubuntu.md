## 编译 Ubuntu 固件

本章介绍 Ubuntu 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

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

在 `device/rockchip/rk3588/` 目录下，有不同板型的配置文件，选择配置文件：

```bash
./build.sh  

or

./build.sh  # MIPI 屏幕
```

#### 编译

##### 全自动编译

* 下载根文件系统：[Ubuntu 根文件系统(64位)](https://www.t-firefly.com/doc/download/161.html)，放到 SDK 路径下

```bash
7z x ubuntu-aarch64-rootfs.7z
mkdir ubuntu_rootfs
mv ubuntu-aarch64-rootfs.img ubuntu_rootfs/rootfs.img
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
./build.sh extboot
```

* 编译 recovery

```bash
./build.sh recovery
```

* 下载根文件系统：[Ubuntu 根文件系统(64位)](https://www.t-firefly.com/doc/download/161.html)，放到 SDK 路径下

```bash
7z x ubuntu-aarch64-rootfs.7z
mkdir ubuntu_rootfs
mv ubuntu-aarch64-rootfs.img ubuntu_rootfs/rootfs.img
```

* 更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

* 打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```