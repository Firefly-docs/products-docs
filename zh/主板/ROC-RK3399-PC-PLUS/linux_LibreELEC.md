# 编译 LibreELEC 固件

使用LibreELEC官方提供的SDK，并打上firefly提供的补丁文件，本章详细的说明 SDK 的具体用法。

## 准备工作

### 搭建 SDK 编译环境

**以下文件请务必确认安装！**

这里使用Ubuntu18.04进行测试(推荐使用ubuntu18.04系统进行开发，或者使用 docker 部署 Ubuntu18.04 容器，否则无法安装好环境包)：
```
sudo apt-get update

sudo apt-get install repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler \
gcc-aarch64-linux-gnu mtools parted libudev-dev libusb-1.0-0-dev python-linaro-image-tools \
linaro-image-tools gcc-arm-linux-gnueabihf libssl-dev liblz4-tool genext2fs lib32stdc++6 \
gcc-aarch64-linux-gnu g+conf autotools-dev libsigsegv2 m4 intltool libdrm-dev curl sed make \
binutils build-essential gcc g++ bash patch gzip bzip2 perl tar cpio python unzip rsync file bc wget \
libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev libglade2-dev cvs git mercurial rsync openssh-client \
subversion asciidoc w3m dblatex graphviz python-matplotlib libc6:i386 libssl-dev texinfo fakeroot \
libparse-yapp-perl default-jre patchutils swig
```

### 下载LibreELEC源码
```
git clone https://github.com/LibreELEC/LibreELEC.tv.git
```

**建议：** 该源码包编译后占用空间大约32G左右，建议预留可用内存35G以上。

### LibreELEC源码主要目录介绍

目录：

```
├── LibreELEC.tv
│   ├── packages                                 需要下载和构建包的目录
│   ├── projects                                 支持平台soc设备的系统构建工程目录
│   ├── config
│   ├── scripts                                  系统构建脚本文件目录
│   ├── create_tar.sh
│   ├── sources                                  构建系统时从服务器拉取下来的资源包存放目录
│   ├── distributions                            配置构建系统需要的资源包目录
│   ├── target                                   存放编译结果的目录
│   ├── licenses
│   ├── Makefile
│   └── tools
```

## 编译 SDK

* 建议选择LibreELEC的9.95.4版本
```
cd LibreELEC.tv
# commit 04dc864eeba97cee2d5375981f898ea246440a21 (tag: 9.95.4)
git checkout 9.95.4
```

* 打上相应的patch
```
代码还在合并，近期内发布
```
* 如果需要使用调试串口终端，需要修改文件 LibreELEC.tv/projects/Rockchip/devices/RK3399/options ，修改EXTRA_CMDLINE选项，修改成如下样子：
```
EXTRA_CMDLINE="console=uart8250,mmio32,0xff1a0000 console=tty0 systemd.debug_shell=ttyS2"
```
也可以在写好镜像的SD卡上的LIBREELEC/extlinux/extlinux.conf文件里的boot里的console后面的信息修改成上述模样。

* 编译
```
# 网络环境不好的用户，可以使用以下命令下载好所需的源码包
#PROJECT=Rockchip ARCH=arm tools/download-tool

# 执行编译命令，构建系统。第一次系统构建需要比较长的时间（一般3到7个小时），请耐心等待。
# 编译命令在 LibreELEC.tv/projects/Rockchip/devices/RK3399/README.md 中有说明。
PROJECT=Rockchip DEVICE=RK3399 ARCH=arm UBOOT_SYSTEM=roc-pc-plus make image
```

**也可以下载已编译好[LibrEELEC系统镜像](https://www.t-firefly.com/doc/download/85.html#other_461)**

### 制作SD启动卡 (目前只支持USB/SD卡启动)
编译完成后的镜像文件存放在target目录中，这时需要下载SD卡系统制作工具,前往下面的地址下载系统制作工具。例如：如果你是linux64位操作系统，点击[LibreELEC.USB-SD.Creator.Linux-64bit.bin](https://wiki.libreelec.tv/installation/create-media)工具下载。

* 在第二项里点击选择文件选项，找到编译出来的.gz文件，然后选择你要烧写的USB或SD卡，最后写入镜像。


* 如果需要使用IR遥控，在终端执行这条命令，便可以使用ir遥控（断电便需重新指定）：
```
ir-keytable -c -w /usr/lib/udev/rc_keymaps/geekbox.toml
```
不同的遥控器对应的键值不一样，需要用户自行配置，通过以下这一条命令可观察遥控器的按下按键产生的值。（一般根据一两个键值便很容易找到对应的键值表）
```
ir-keytable -t
```
然后根据对应的键值在 /usr/lib/udev/rc_keymaps/目录下找到对应自己遥控器的键值表。(grep "键值 = 按键" *.toml)

如果你想遥控器永久有效，需要执行以下操作。（geekbox.toml是我遥控器配对的键值表）
```
cd /storage/.config/
cp rc_maps.cfg.sample rc_maps.cfg
在 rc_maps.cfg末尾增加一行：*               *               geekbox.toml
cp /usr/lib/udev/rc_keymaps/geekbox.toml ./rc_keymaps/
chmod 777 ./rc_keymaps/geekbox.toml
```

**注意**
如果第一次上电时发现耳机音频不能使用现象，需要重新烧写镜像。