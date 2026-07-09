# 编译 Linux 固件

## 获取 SDK

请联系销售(sales@t-firefly.com)获取 SDK 下载链接, 并且阅读下载链接的 readme 文档。

<font color=red>

**注意：**

**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**

**2. 编译环境请使用 Ubuntu22.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**

**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**

**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**
</font>## Linux SDK 配置介绍

### 目录介绍

```bash
$ tree -L 1
.
├── app
├── buildroot	# Buildroot 根文件系统编译目录
├── build.sh -> device/rockchip/common/scripts/build.sh # 编译脚本
├── Copyright_Statement.md -> docs/licenses/LICENSE
├── device		# 编译相关配置文件
├── docs		# 文档
├── external
├── firefly-release.sh -> device/rockchip/common/scripts/firefly-release.sh
├── firefly-update.sh -> device/rockchip/common/scripts/firefly-update.sh
├── hal
├── kernel -> kernel-6.1
├── kernel-6.1
├── Makefile -> device/rockchip/common/Makefile
├── prebuilts	# 交叉编译工具链
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
├── rtos
├── tools		# 工具目录
├── u-boot
└── yocto		# 编译 yocto 文件系统目录
```

### 配置文件介绍

在 `device/rockchip/rv1126b/` 目录下，有不同板型的配置文件(xxxx_defconfig)，用于管理 SDK 每个环节的编译配置。

配置文件命名规则：
```
<vendor>_<chip>_<model>-<extra>_<OS>_defconfig

vendor: 产品供应商
chip: 产品所用的芯片
model: 产品型号，表示这个配置文件是用于该型号的产品
extra: 额外属性，可以为空
OS: 产品将会运行的操作系统
```

配置文件内容介绍，比如 firefly_rv1126b_aio-1126bjd4v0_debian_defconfig :
```bash
#include "firefly.config"
RK_UBOOT_INI="RV1126BMINIALL.ini"
RK_UBOOT_SPL=y
RK_KERNEL_DTS_NAME="rv1126b-firefly-aio-1126bjd4v0"		# 指定编译内核所使用的设备树
RK_KERNEL_CFG="rockchip_linux_defconfig"
RK_MISC_RECOVERY=y
RK_RECOVERY_RAMDISK="common-recovery-arm64.cpio.gz"		# 指定预编译好的 recovery 文件系统
RK_BOOT_FIT_ITS_NAME="bootramdisk.its"
RK_RAMDISK_IMG="kernel/ramdisk-64.img"
RK_PRODUCT_MODEL="AIO-1126BJD4V0"				# 产品名称/型号
RK_PARAMETER="parameter-debian-fit.txt"				# 指定固件分区表
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rv1126b_debian_rootfs.img" # 指定要使用的预编译文件系统
```

### 分区说明

#### parameter 分区表

parameter 文件中包含了固件的分区信息，比如 `device/rockchip/rv1126b/parameter-xxxxxx-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RV1126B
MACHINE_ID: 007
MANUFACTURER: RV1126B
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
GROW_ALIGN: 0
CMDLINE: mtdparts=:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot),0x00040000@0x00048000(recovery),0x00010000@0x00088000(backup),0x00a00000@0x00098000(rootfs),-@0x00a98000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
uuid:boot=7A3F0000-0000-446A-8000-702F00006273
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为 uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。分区的起始位置 + 分区大小 = 下一个分区的起始位置。

parameter 中数字的单位是 “块”，每块 512 字节。所以 uboot 分区大小为 0x00002000，即 8192 块，共 8192 x 512 / 1024 / 1024 = 4 MiB

## 编译 Yocto 固件

### 编译

#### 选择映像
Yocto 项目提供了一些可用于不 layer 的映像。下表列出目前支持构建的映像和相关配方。

| 映像名字  | 描述 | 提供的layer |
| :--------: | :-------: | :-------: |
| core-image-minimal | A small image that only allows a device to boot | Poky |
| core-image-minimal-xfce | A XFCE minimal demo image | meta-openembedded/meta-xfce |
| core-image-sato | Image with Sato, a mobile environment and visual style for mobile devices. The image supports X11 with a Sato theme, Pimlico applications, and contains terminal, editor, and file manager | Poky |
| core-image-weston | A very basic Wayland image with a terminal | Poky |
| core-image-x11 | A very basic X11 image with a terminal | Poky |

### 编译映像文件

**使用 bitbake 命令构建的过程需要保证网络连接正常，如果是中国内陆客户需要保证能 ping 通外网**

进入目录 <path/to/yocto/poky> ，按顺序执行如下命令

```bash
# Install the required environment packages
# sudo apt install zstd
source oe-init-build-env

# 添加 layer（只需要执行一次）
bitbake-layers add-layer ../../meta-openembedded/meta-oe
bitbake-layers add-layer ../../meta-openembedded/meta-python
bitbake-layers add-layer ../../meta-openembedded/meta-networking
bitbake-layers add-layer ../../meta-openembedded/meta-multimedia
bitbake-layers add-layer ../../meta-openembedded/meta-gnome
bitbake-layers add-layer ../../meta-openembedded/meta-xfce
bitbake-layers add-layer ../../meta-lts-mixins
bitbake-layers add-layer ../../meta-clang
bitbake-layers add-layer ../../meta-browser/meta-chromium
bitbake-layers add-layer ../../meta-rockchip
```

选择其中之一命令来编译完整 core-image recipes 。以下是基于 x11 的 core-image 。

```bash
MACHINE=aio-1126bjd4v0 bitbake core-image-minimal
MACHINE=aio-1126bjd4v0 bitbake core-image-minimal-xfce
MACHINE=aio-1126bjd4v0 bitbake core-image-x11
MACHINE=aio-1126bjd4v0 bitbake core-image-sato
```



以下是基于 wayland 的 core-image 。需要在 ```/path/to/yocto/meta-rockchip/conf/machine/include/display.conf``` 修改 DISPLAY_PLATFORM 为 wayland 。修改如下：

```
DISPLAY_PLATFORM ?= "wayland"
# DISPLAY_PLATFORM ?= "x11"
```

完成上述修改后，执行命令编译 core-image-weston

```bash
MACHINE=aio-1126bjd4v0 bitbake core-image-weston
```

注意：如果在已经进行了完整编译一次 core-image 的基础上，需要更换编译的 core-image recipes 。需要将当前编译过 core-image 的清理掉，再开始编译新的 core-image 。

例如：当前编译的是 core-image-minimal 。需要更换成 core-image-sato 。

```bash
MACHINE=aio-1126bjd4v0 bitbake core-image-minimal -c clean
MACHINE=aio-1126bjd4v0 bitbake core-image-sato
```



如果想单独编译部分 recipes 可以参考以下内容：

```bash
# kernel
MACHINE=aio-1126bjd4v0 bitbake linux-rockchip
        
# u-boot
MACHINE=aio-1126bjd4v0 bitbake u-boot-rockchip
        
# rkmpp
MACHINE=aio-1126bjd4v0 bitbake rockchip-mpp
        
# rockchip-librga
MACHINE=aio-1126bjd4v0 bitbake rockchip-librga
        
# 参看更多编译对象
MACHINE=aio-1126bjd4v0 bitbake -s
```



### 调整编译速度

修改文件```/path/to/yocto/meta-rockchip/conf/machine/firefly-kernel6.1-rv1126b.conf``` 中的 BB_NUMBER_THREADS 和 PARALLEL_MAKE 变量配置。若线程数量设置过大可能会导致机器内存不足，导致编译失败。请根据编译机器的配置来设置编译速度。

```
BB_NUMBER_THREADS = "4"
PARALLEL_MAKE = "-j 4"
```

- [BB_NUMBER_THREADS](https://docs.yoctoproject.org/ref-manual/variables.html#term-BB_NUMBER_THREADS): The maximum number of threads BitBake simultaneously executes.
- [BB_NUMBER_PARSE_THREADS](https://docs.yoctoproject.org/ref-manual/variables.html#term-BB_NUMBER_PARSE_THREADS): The number of threads BitBake uses during parsing.
- [PARALLEL_MAKE](https://docs.yoctoproject.org/ref-manual/variables.html#term-PARALLEL_MAKE): Extra options passed to the `make` command during the [do_compile](https://docs.yoctoproject.org/ref-manual/tasks.html#ref-tasks-compile) task in order to specify parallel compilation on the local build host.
- [PARALLEL_MAKEINST](https://docs.yoctoproject.org/ref-manual/variables.html#term-PARALLEL_MAKEINST): Extra options passed to the `make` command during the [do_install](https://docs.yoctoproject.org/ref-manual/tasks.html#ref-tasks-install) task in order to specify parallel installation on the local build host.

### 更多 bitbake 选项

从根本上说，BitBake 是一个通用任务执行引擎，它允许 shell 和 Python 任务高效并行运行，同时在复杂的任务间依赖约束下工作。 BitBake 的主要用户之一，OpenEmbedded，利用这个核心并使用面向任务的方法构建嵌入式 Linux 软件堆栈。更多详细使用方法请查看[《bitbake-user-manual》](https://docs.yoctoproject.org/bitbake/2.0/bitbake-user-manual/bitbake-user-manual-intro.html#)。

```bash
MACHINE=aio-1126bjd4v0 bitbake <target> <paramater>
# e.g
MACHINE=aio-1126bjd4v0 bitbake u-boot-rockchip -c clean
MACHINE=aio-1126bjd4v0 bitbake u-boot-rockchip
```

| Bitbake paramater  | 描述 |
| :--------: | :-------: |
| -c fetch | 拉取目标所需要的代码 |
| -c clean | 清除目标的输出文件 |
| -c cleanall | 删除目标所有输出文件、共享高速缓存（shared state cache）和源代码 |
| -c compile -f | 使用此选项可在部署映像后强制重新编译，但不建议使用，除非 Yocto Project 不知道目标代码已经发生改变 |
| -c listtasks | 列出目标定义的所有 target |



### 分区固件烧写

编译生成的固件位于目录```<path/to/yocto>/build/tmp/deploy/images/<board>/```

```bash
$ sudo upgrade_tool di -boot boot.img
$ sudo upgrade_tool di -uboot uboot.img
$ sudo upgrade_tool di -misc misc.img
$ sudo upgrade_tool di -recovery recovery.img
```

* 分区烧写适用于调试阶段，固件验证请使用下文的统一固件烧写
* rootfs 不支持单独烧写，需要打包完整固件再烧写

### 统一固件烧写

编译生成的固件位于目录```<path/to/yocto>/build/tmp/deploy/images/<board>/```，待下载的文件为.wic与update.img，进入loader模式后执行如下命令：

```bash
$ sudo upgrade_tool wl 0 <IMAGE NAME>.wic
$ sudo upgrade_tool uf update.img
```
* **固件默认登录账号为：root，密码为：firefly 。固件含有普通用户账号名称为：firefly ，密码为：firefly 。**

注意：如果客户在 Windows PC 上开发，使用 RKdevtool 直接烧录 update.img 即可，**不需要烧录 `<IMAGE NAME>.wic`**。但是要注意一点是 update.img 是一个链接文件，实际得选择链接文件所指向的实际文件。

### 相关概述
Yocto Project 是一个专注于嵌入式 Linux® 操作系统开发的开源协作项目，它提供灵活的工具集和开发环境，允许全球的嵌入式设备开发人员通过共享技术，软件堆栈，配置和用于创建这些定制的Linux映像的最佳实践进行协作。有关 Yocto 项目的更多信息，请参阅 Yocto Project 官网：[www.yoctoproject.org/](www.yoctoproject.org/)。 Yocto Project 官网上有 [Yocto Project Reference Manual](https://docs.yoctoproject.org/current/ref-manual/index.html) 和 [Yocto Project Overview](https://docs.yoctoproject.org/) 等相关文档详细描述了如何构建系统。


### Yocto Project Release layer 介绍

| layer  | 路径 | 优先级(数字越大优先级越高) | 描述 |
| :--------: | :-------: | :-------: | :-------: |
| meta-oe | meta-openembedded/meta-oe | 6 | contains a large amount of additional recipes |
| meta-python | meta-openembedded/meta-python |  7 |  Provide Python recipes|
| meta-qt5 | meta-qt5 | 7 | Provides QT5 recipes |
| meta-clang | meta-clang |  7 |  clang compiler |
| meta-rockchip | meta-rockchip |  9 |  Rockchip board level support available|
| meta | meta |  5 |  Contains the OpenEmbedded-Core metadata|
| meta-poky | meta-poky |  5 |  Holds the configuration for the Poky reference distribution|
| meta-yocto-bsp | meta-yocto-bsp |  5 |  Configuration for the Yocto Project reference hardware board support package.|
| meta-chromium | meta-chromium |  7 |  Provide chromium browser recipe|
## 编译 Debian 固件

### 准备工作

执行下面两条命令来安装需要的工具
```bash
sudo apt update

sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev p7zip-full
```

下载根文件系统：[Debian 根文件系统(64位)](https://www.t-firefly.com/doc/download/333.html#other_717)，请使用网盘中 kernel-6.1 目录下的文件系统。

下载后将文件系统解压到 SDK/prebuilt_rootfs/ 下，并创建链接

```bash
# 解压
7z x debian12_xxxx_rootfs_xxxx.7z

# 将解压后的 rootfs 镜像移动到 sdk 并创建一个符号链接
mkdir ./SDK/prebuilt_rootfs/
mv debian12_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf debian12_xxxx_rtoofs_xxxx.img rv1126b_debian_rootfs.img
cd ..
```

#### 编译前配置

下载 SDK 后，只有第一次编译需要执行 ./build.sh 选择编译的芯片平台。不是第一次编译则跳过该步骤。
```bash
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

执行 `./build.sh lunch` 列出所有可用的配置文件，输入编号选择对应的配置文件，注意要选择带有 debian 字样的配置文件：
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
**输入编号选择板型： firefly_rv1126b_aio-1126bjd4v0_debian_defconfig。**

### 编译

#### 完整编译

执行如下命令即可
```bash
./build.sh all
```

编译完成后会生成完整固件 `output/update/update.img`

##### 部分编译

一般来说自定义开发只涉及 kernel 和 uboot。因此，在进行过一次完整编译后，如果后续又修改了 kernel 或 uboot，可以部分编译来加快速度。

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

**输入编号选择板型： firefly_rv1126b_aio-1126bjd4v0_buildroot_defconfig。**

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
