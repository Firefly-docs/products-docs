# 编译 Linux 固件

## 编译环境搭建

本章介绍 Linux SDK 的编译环境搭建。

<font color=red>

**注意：**

**（1）推荐在 X86_64 Ubuntu 16.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。**

**（2）使用普通用户进行编译，不要使用 root 用户权限进行编译。**</font>

### 获取 SDK

首先准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以`~/proj`为例

<font color=red>**不要在虚拟机共享文件夹以及非英文目录存放、解压SDK，避免产生不必要的错误**</font>

获取 SDK 需要先安装：

```
sudo apt update
sudo apt install -y repo git python
```

* 方法一（推荐）

**Firefly_Linux_SDK 源码包比较大，可以通过如下方式获取 Firefly_Linux_SDK 源码包：**[下载链接](https://community.t-firefly.com/doc/download/51)

下载完成后先验证一下 MD5 码：

```bash
$ md5sum rk3288_linux_release_v2.5.0a_20230510_split_dir/*firefly_split*
f3b9309c574e04491a9e9f3e7a5c5540  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file0
fb0cd76f518405441bbc2ad2334ee6b9  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file1
cb26fddb36d49ff6b79f2fc934a731c1  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file2
43aa8447d4ef303ce41274c6ea7b00a2  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file3
c6c1ed77033af3d26be24974dda1b2aa  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file4
```

确认无误后，就可以解压：

```bash
# 解压
mkdir ~/proj/
cd ~/proj/
cat path/to/rk3288_linux_release_v2.5.0a_20230510_split_dir/*firefly_split* | tar -xzv

# 导出数据
.repo/repo/repo sync -l
```

* 方法二

通过 repo 拉取代码，此方法对网络要求较高，有条件可以使用

可选择获取完整 SDK 或者 BSP：

```bash
mkdir ~/proj/rk3288_linux_release_v2.5.0a_20230510/
cd ~/proj/rk3288_linux_release_v2.5.0a_20230510/

## 完整 SDK
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3288_linux_release.xml

## BSP （ 只包含基础仓库和编译工具 ）
## BSP 包括 device/rockchip 、docs 、 kernel 、 u-boot 、 rkbin 、 tools 和交叉编译链
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3288_linux_bsp_release.xml
```

### 同步代码

执行如下命令同步代码：

```bash
# 进入 SDK 根目录
cd ~/proj/rk3288_linux_release_v2.5.0a_20230510/

# 同步
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

后续可以使用如下命令更新 SDK：

```bash
.repo/repo/repo sync -c --no-tags
```

因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行。
### 目录结构

```bash
$ tree -L 1
.
├── app
├── buildroot                                               # Buildroot 根文件系统编译目录
├── build.sh -> device/rockchip/common/build.sh             # 编译脚本
├── debian                                                  # Debian 根文件系统编译目录
├── device                                                  # 编译相关配置文件
├── distro
├── docs                                                    # 文档
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel
├── Makefile -> buildroot/build/Makefile
├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh   # 链接脚本
├── prebuilts                                               # 交叉编译工具链
├── rkbin
├── rkflash.sh -> device/rockchip/common/rkflash.sh         # 烧写脚本
├── tools                                                   # 工具目录
├── u-boot
└── yocto
```

### 安装依赖

* 方法一：

在 PC 中自行安装环境：

```bash
sudo apt-get install repo git gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler mtools \
parted libudev-dev libusb-1.0-0-dev python-linaro-image-tools linaro-image-tools libssl-dev \
autotools-dev libsigsegv2 m4 libdrm-dev curl sed make binutils build-essential gcc g++ bash \
patch gzip bzip2 perl tar cpio python unzip rsync file bc wget libncurses5 libglib2.0-dev \
openssh-client lib32stdc++6 gcc-aarch64-linux-gnu libncurses5-dev lzop libssl1.0.0 libssl-dev \
libglade2-dev cvs mercurial subversion asciidoc w3m dblatex graphviz python-matplotlib \
libc6:i386 texinfo liblz4-tool genext2fs expect autoconf intltool libqt4-dev libgtk2.0-dev
```

* 方法二：使用 Docker

使用 dockerfile 创建容器，在容器中进行编译，完美解决编译环境问题，并且与主机环境隔离，互不影响。

首先在主机中安装 docker，请参考：[安装教程](https://wiki.t-firefly.com/zh_CN/Firefly-Linux-Guide/demo_docker.html#an-zhuang-docker)

创建一个目录作为 docker 工作目录，例如`~/docker/`，在其中创建文件`dockerfile`，内容如下：
```dockerfile
FROM ubuntu:18.04
MAINTAINER firefly "service@t-firefly.com"

ENV DEBIAN_FRONTEND=noninteractive

RUN cp -a /etc/apt/sources.list /etc/apt/sources.list.bak
RUN sed -i 's@http://.*ubuntu.com@http://repo.huaweicloud.com@g' /etc/apt/sources.list

RUN apt update

RUN apt install -y build-essential crossbuild-essential-arm64 \
	bash-completion vim sudo locales time rsync bc python

RUN apt install -y repo git ssh libssl-dev liblz4-tool lib32stdc++6 \
	expect patchelf chrpath gawk texinfo diffstat binfmt-support \
	qemu-user-static live-build bison flex fakeroot cmake \
	unzip device-tree-compiler python-pip ncurses-dev python-pyelftools \
	subversion asciidoc w3m dblatex graphviz python-matplotlib cpio \
	libparse-yapp-perl default-jre patchutils swig expect-dev u-boot-tools

RUN apt update && apt install -y -f

# language support
RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8

# switch to a no-root user
RUN useradd -c 'firefly user' -m -d /home/firefly -s /bin/bash firefly
RUN sed -i -e '/\%sudo/ c \%sudo ALL=(ALL) NOPASSWD: ALL' /etc/sudoers
RUN usermod -a -G sudo firefly

USER firefly
WORKDIR /home/firefly
```
创建镜像
```bash
cd ~/docker
docker build -t sdkcompiler .
# sdkcompiler 是镜像名称，可随意更改，注意命令最后有一个‘.’
# 此过程需要一段时间，请耐心等待
```
镜像创建完毕后，创建容器并启动
```bash
# 此处将主机内 SDK 所在文件夹挂载到容器内，这样容器内就能访问主机中的 SDK 了
# source= 填 SDK 所在目录；target= 填容器内的一个目录，必须是空目录
# ubuntu18 是容器名，firefly 是容器 hostname，均可随意更改
# sdkcompiler 是上一步的镜像名
docker run --privileged --mount type=bind,source=/home/fierfly/proj,target=/home/firefly/proj --name="ubuntu18" -h firefly -it sdkcompiler
```
现在就可以在容器中进行 SDK 的编译了。

退出容器、重启容器的方法：
```bash
# 在容器内输入 exit 即可退出

# 查看所有容器（包括已退出的）
docker ps -a

# 重启一个退出的容器并连接
docker start ubuntu18 # 容器名
docker attach ubuntu18
```
## 编译 Ubuntu 固件

本章介绍 Ubuntu 固件的编译流程。

### 编译 SDK

#### 编译前配置

在 `device/rockchip/rk3288/` 目录下，有不同板型的配置文件，选择配置文件：

##### HDMI

```bash
./build.sh aio-3288c-ubuntu.mk
```


##### LVDS DM-M10R800

```bash
./build.sh aio-3288c-lvds-ubuntu.mk
```


配置文件会链接到 `device/rockchip/.BoardConfig.mk`，检查该文件可以验证是否配置成功。

相关配置介绍：

```bash
# Target arch
export RK_ARCH=arm                                              # 32位 ARM 架构
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly-rk3288                        # u-boot 配置文件
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly_linux_defconfig              # kernel 配置文件
# Kernel dts
export RK_KERNEL_DTS=rk3288-firefly-aioc                              # dts 文件
# parameter for GPT table
export RK_PARAMETER=parameter-ubuntu.txt                        # 分区表
# rootfs image path
export RK_ROOTFS_IMG=ubuntu_rootfs/rk3288_ubuntu_rootfs.img     # 根文件系统路径
```

#### 部分编译

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

#### 下载 Ubuntu 根文件系统

* 下载根文件系统：[Ubuntu 根文件系统(32位)](https://community.t-firefly.com/doc/download/51)，放到 SDK 路径下

* 解压文件

```bash
7z x ubuntu-armhf-rootfs.7z
```

* 将根文件系统放到 `ubuntu_rootfs/` 目录下

```bash
mkdir ubuntu_rootfs
mv ubuntu-armhf-rootfs.img ubuntu_rootfs/rk3288_ubuntu_rootfs.img
```

#### 打包固件

更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```

#### 全自动编译

全自动编译会执行上述编译、打包操作，生成完整固件。

```bash
./build.sh
```

### 分区说明

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 parameter-ubuntu.txt 为例：

路径：`device/rockchip/rk3288/parameter-ubuntu.txt`

```bash
FIRMWARE_VER: 8.1
MACHINE_MODEL:RK3288
MACHINE_ID:007
MANUFACTURER:RK3288
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3288
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(trust),0x00002000@0x00008000(misc),0x00010000@0x0000a000(boot),0x00010000@0x0001a000(recovery),0x00010000@0x0002a000(backup),0x00600000@0x0005a000(rootfs),-@0x0065a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

#### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

路径：`tools/linux/Linux_Pack_Firmware/rockdev/rk3288-ubuntu-package-file`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
trust           Image/trust.img
uboot           Image/uboot.img
boot            Image/boot.img
misc            Image/misc.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
userdata:grow RESERVED
backup          RESERVED
```## 编译 Buildroot 固件

本章介绍 Buildroot 固件的编译流程。

### 编译 SDK

#### 编译前配置

在 `device/rockchip/rk3288/` 目录下，有不同板型的配置文件，选择配置文件：

##### HDMI

```bash
./build.sh aio-3288c-buildroot.mk
```


##### LVDS DM-M10R800

```bash
./build.sh aio-3288c-lvds-buildroot.mk
```


配置文件会链接到 `device/rockchip/.BoardConfig.mk`，检查该文件可以验证是否配置成功。

相关配置介绍：

```bash
# Target arch
export RK_ARCH=arm                                              # 32位 ARM 架构
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly-rk3288                        # u-boot 配置文件
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly_linux_defconfig              # kernel 配置文件
# Kernel dts
export RK_KERNEL_DTS=rk3288-firefly-aioc                              # dts 文件
# Buildroot config
export RK_CFG_BUILDROOT=rockchip_rk3288                         # Buildroot 配置
# Recovery config
export RK_CFG_RECOVERY=rockchip_rk3288_recovery                 # recovery 配置
# parameter for GPT table
export RK_PARAMETER=parameter-buildroot.txt                     # 分区表
# rootfs image path
export RK_ROOTFS_IMG=buildroot/output/$RK_CFG_BUILDROOT/images/rootfs.$RK_ROOTFS_TYPE   # 根文件系统路径
```

#### 部分编译

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

* 编译 Buildroot 根文件系统

编译 Buildroot 根文件系统，将会在 `buildroot/output` 生成编译输出目录：

```bash
./build.sh buildroot

# 注：确保作为普通用户编译 Buildroot 根文件系统，避免不必要的错误。
```

#### 打包固件

更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```

#### 全自动编译

全自动编译会执行上述编译、打包操作，生成完整固件。

```bash
./build.sh
```

### 分区说明

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 parameter-buildroot.txt 为例：

路径：`device/rockchip/rk3288/parameter-buildroot.txt`

```bash
FIRMWARE_VER: 8.1
MACHINE_MODEL:RK3288
MACHINE_ID:007
MANUFACTURER:RK3288
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3288
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(trust),0x00002000@0x00008000(misc),0x00010000@0x0000a000(boot),0x00010000@0x0001a000(recovery),0x00010000@0x0002a000(backup),0x00020000@0x0003a000(oem),0x00100000@0x0005a000(rootfs),-@0x0015a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

#### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

路径：`tools/linux/Linux_Pack_Firmware/rockdev/rk3288-package-file`

```bash
# name          relative path
#
#hwdef          hwdef
package-file    package-file
bootloader      image/miniloaderall.bin
parameter       image/parameter.txt
trust           image/trust.img
uboot           image/uboot.img
misc            image/misc.img
boot            image/boot.img
recovery        image/recovery.img
rootfs          image/rootfs.img
oem             image/oem.img
userdata:grow   image/userdata.img
backup          RESERVED
```