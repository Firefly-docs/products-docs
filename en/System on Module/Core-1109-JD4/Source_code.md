# Source Code Compilation

Firefly provides a complete software SDK to facilitate software development.

## Source Code Acquisition

<font color=red>**Very important:**</font>

Please update the SDK to the latest version according to the Wiki before development, especially for users using CS-R1 cluster servers (if the code version is too old, the software level configuration and hardware mismatch may cause <font color=red>**Hardware damage**</font> for CS-R1 users).

### Note
For `CORE-1126-JD4/CORE-1109-JD4` series, please download the `rv1126_rv1109_linux_release` REPO_SDK package.

### Method 1 (domestic users)

* Download the `REPO_SDK` package from the [Resource Download](http://www.t-firefly.com/doc/download/101.html) page. </br>
* Compare the MD5 code of the REPO_SDK package to verify integrity, then decompress it.
```
md5sum rv1126_rv1109_linux_release_20211022.tgz
596c6bc6bb3095aea97d54c9df4cf333  rv1126_rv1109_linux_release_20211022.tgz
tar xvf rv1126_rv1109_linux_release_20211022.tgz
```
* After decompression, enter the folder and complete the synchronization.
```
#This compressed package contains a .repo directory. After decompression, perform the following operations in the current directory
.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all

#You can use the following command to update the SDK later
.repo/repo/repo sync -c --no-tags

#Due to network environment and other reasons, the `.repo/repo/repo sync -c --no-tags` command may fail to update the code. You can execute it repeatedly
```

### Method 2 (foreign users)
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
