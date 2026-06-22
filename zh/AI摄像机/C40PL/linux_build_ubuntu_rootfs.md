# 创建 Ubuntu 根文件系统

## 准备工作

### 安装 qemu

在 Linux PC 主机上安装模拟器：

```
sudo apt-get install qemu-user-static
```
### 下载和解压 ubuntu-core

C40PL Ubuntu 根文件系统是基于 Ubuntu base 18.04 来创建的。用户可以到 [ubuntu cdimg](http://cdimage.ubuntu.com/ubuntu-base/releases/18.04/release/) 下载，选择下载 `ubuntu-base-18.04.1-base-arm64.tar.gz`。

下载完之后，创建临时文件夹并解压根文件系统：

```
mkdir temp
sudo tar -xpf ubuntu-base-18.04.1-base-arm64.tar.gz -C temp
```

## 修改根文件系统

### 准备工作

准备网络：

```
sudo cp -b /etc/resolv.conf temp/etc/resolv.conf
```

准备 qemu：

```
sudo cp /usr/bin/qemu-aarch64-static temp/usr/bin/
```

进入根文件系统进行操作：

```
sudo chroot temp
```

### 更新及安装

更新：

```
apt update
apt upgrade
```

安装自己需要的功能

```
apt install vim git ....(根据自己需求添加)
```

安装 xubuntu：

```
apt-get install xubuntu-desktop
```

可能出现错误:

```
E: Unable to locate package xxxx
```

安装包的源没有添加到 `/etc/apt/source.list` 中,导致无法识别安装包,可以自行添加源，也可以使用下面给出的 `source.list` 覆盖原来的 `/etc/apt/source.list` 文件:

```
# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.

deb http://ports.ubuntu.com/ubuntu-ports/ xenial main restricted
deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://ports.ubuntu.com/ubuntu-ports/ xenial-updates main restricted
deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial-updates main restricted

## Uncomment the following two lines to add software from the 'universe'
## repository.
## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://ports.ubuntu.com/ubuntu-ports/ xenial universe
deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial universe
deb http://ports.ubuntu.com/ubuntu-ports/ xenial-updates universe
deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial-updates universe

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://ports.ubuntu.com/ubuntu-ports/ xenial-backports main restricted
deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial-backports main restricted

deb http://ports.ubuntu.com/ubuntu-ports/ xenial-security main restricted
deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial-security main restricted
deb http://ports.ubuntu.com/ubuntu-ports/ xenial-security universe
deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial-security universe
deb http://ports.ubuntu.com/ubuntu-ports/ xenial-security multiverse
deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial-security multiverse
```

### 添加用户及设置密码

添加用户：

```
useradd -s '/bin/bash' -m -G adm,sudo firefly
```

给用户设置密码：

```
passwd firefly
```

给root用户设置密码：

```
passwd root
```

修改完自己的根文件系统就可以退出了。

```
exit
```

## 制作根文件系统

制作自己的根文件系统，大小依据自己的根文件系统而定，注意依据 `temp` 文件夹的大小来修改 `count` 值：

```
mkdir  rootfs
dd if=/dev/zero of=linuxroot.img bs=1M count=4000
mkfs.ext4 linuxroot.img
sudo mount linuxroot.img rootfs/
sudo cp -rfp temp/*  rootfs/
sudo umount rootfs/
e2fsck -p -f linuxroot.img
resize2fs  -M linuxroot.img
```

这样 `linuxroot.img` 就是最终的根文件系统映像文件了。

## FAQs

### 根文件系统加载后，大小不正常，未占满整个分区：

在系统正确加载后执行扩展文件系统命令：

```
resize2fs /dev/mtd/by-name/linuxroot
```