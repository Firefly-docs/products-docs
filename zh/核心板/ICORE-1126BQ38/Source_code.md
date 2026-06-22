# 源码编译

Firefly 提供完整的软件 SDK，方便用户进行软件开发。

## 源码获取

<font color=red>**非常重要：**</font>

开发前请根据 Wiki 把 SDK 更新到最新，尤其是使用 CS-R1 集群服务器用户（使用 CS-R1 用户如果代码版本太旧软件电平配置和硬件不匹配可能导致<font color=red>**硬件损坏**</font>）。

### 注意
`CORE-1126-JD4/CORE-1109-JD4` 系列请下载 `rv1126_rv1109_linux_release` REPO_SDK 软件包。

### 方式一（国内用户）

* 从[资源下载](http://www.t-firefly.com/doc/download/101.html)页面下载 `REPO_SDK` 软件包。</br>
* 比较 REPO_SDK 软件包的 MD5 码校验完整性，然后解压。
```
md5sum rv1126_rv1109_linux_release_20211022.tgz
596c6bc6bb3095aea97d54c9df4cf333  rv1126_rv1109_linux_release_20211022.tgz
tar xvf rv1126_rv1109_linux_release_20211022.tgz
```
* 解压后进入文件夹，完成同步工作。
```
#本压缩包内包含一个.repo目录，解压之后，在当前目录下执行以下操作
.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all

#后续可以使用以下命令更新SDK
.repo/repo/repo sync -c --no-tags

#因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行
```

### 方式二（国外用户）
```
repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_release.xml

repo sync -c
#需反复确认代码下载是否成功
```

## 源码编译

### 搭建环境

Ubuntu 16.04 x86 PC 主机：
```
sudo apt-get install repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-
tools device-tree-compiler gcc-aarch64-linux-gnu mtools parted libudev-dev
libusb-1.0-0-dev python-linaro-image-tools linaro-image-tools autoconf autotools-
dev libsigsegv2 m4 intltool libdrm-dev curl sed make binutils build-essential gcc
g++ bash patch gzip gawk bzip2 perl tar cpio python unzip rsync file bc wget
libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev libglade2-dev cvs git
mercurial rsync openssh-client subversion asciidoc w3m dblatex graphviz python-
matplotlib libc6:i386 libssl-dev expect fakeroot cmake flex bison liblz4-tool
libtool keychain
```

Ubuntu 17.04 除以上软件包外还需安装以下依赖：
```
sudo apt-get install lib32gcc-7-dev g++-7 libstdc++-7-dev
```

### 编译配置

SDK 包含 `uboot`、`kernel`、`recovery`、`buildroot`、`spl`等等全套源码，编译前需要选好配置文件，用于
配置每一个仓库编译时需要用到的配置项。

```
# 配置文件是和设备绑定 CORE-1126-JD4 官方使用 aio-rv1126-jd4.mk/aio-rv1109-jd4.mk
# 文件存放在 sdk/device/rockchip/rv1126_rv1109/

# 编译 CORE-1126-JD4 固件 适配的屏幕为 DM-M10R800 V2 屏幕模组
./build.sh device/rockchip/rv1126_rv1109/aio-rv1126-jd4-BE-45.mk

# 编译 CORE-1109-JD4 固件 适配的屏幕为 DM-M10R800 V2 屏幕模组
./build.sh device/rockchip/rv1126_rv1109/aio-rv1109-jd4-BE-45.mk

# 编译 CORE-1126-JD4 V1.1 快速启动固件 适配的屏幕为 DM-M10R800 V2 屏幕模组
./build.sh device/rockchip/rv1126_rv1109/aio-rv1126-jd4-BE-45-tb-v11.mk
```

### 开始编译

编译完成后固件输出路径` rockdev/update.img `，该目录下也包含了所有单独编译项目的编译输出。编译完成后
烧写升级操作请参考[《固件升级》](upgrade.html)。源码 SDK 介绍也可以直接参考 SDK 的 doc 文件夹。

```
# 全局编译命令
./build.sh

# 如果需要单独编译某一项执行以下操作
# uboot
./build.sh uboot

# kernel
./build.sh kernel

# rootfs 
./build.sh rootfs
```
