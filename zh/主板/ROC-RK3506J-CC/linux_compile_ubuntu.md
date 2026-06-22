## 编译 Ubuntu 固件

本章介绍 Ubuntu 固件的编译流程，推荐在 Ubuntu 20.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

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

在 `device/rockchip/rk3576/` 目录下，有不同板型的配置文件，选择配置文件：

```bash
./build.sh  

```

#### 编译

##### 全自动编译

* 下载根文件系统：[Rootfs](https://www.t-firefly.com/doc/download/161.html)，放到 SDK 路径下

```bash
7z x Ubuntu[xx.xx]-xxxx_RK3576_vx.x.xx_xxxxxxxx.7z
mkdir prebuilt_rootfs/
mv Ubuntu[xx.xx]-xxxx_RK3576_vx.x.xx_xxxxxxxx.img prebuilt_rootfs/rk3576_ubuntu_rootfs.img
```

* 开始编译
```bash
./build.sh
```

生成的完整固件会保存到 `output/update/` 目录。

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

* 下载根文件系统：[Rootfs](https://www.t-firefly.com/doc/download/161.html)，放到 SDK 路径下

```bash
7z x Ubuntu[xx.xx]-xxxx_RK3576_vx.x.xx_xxxxxxxx.7z
mkdir prebuilt_rootfs/
mv Ubuntu[xx.xx]-xxxx_RK3576_vx.x.xx_xxxxxxxx.img prebuilt_rootfs/rk3576_ubuntu_rootfs.img
```

* 打包固件，生成的完整固件会保存到 `output/update/` 目录。

```bash
./build.sh updateimg
```