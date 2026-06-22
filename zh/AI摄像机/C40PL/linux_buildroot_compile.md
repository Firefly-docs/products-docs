# 编译 Buildroot 固件

本章介绍 Buildroot 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

## 准备工作

### 搭建编译环境

```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler python-pip ncurses-dev python-pyelftools
```

### 下载 Firefly_Linux_SDK 分卷压缩包

**由于 Firefly_Linux_SDK 源码包比较大，部分用户电脑不支持4G以上文件或单个文件网络传输较慢, 所以我们采用分卷压缩的方法来打包 SDK。用户可以通过如下方式获取 Firefly_Linux_SDK 源码包：**[Firefly_Linux_SDK源码包]()

下载完成后先验证一下 MD5 码：

```bash
$ md5sum rk356x_linux_release_v1.2.0_20211019_split_dir/*firefly_split*
b633414d69240faa0c3bd755c255ede8  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file0
5cb1e2b63bbb4e3595c731038f6723fd  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file1
e06c0f29fd5a870e0942139a1877a470  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file2
4e14fbf72ccbb9b87f81d83a256205b0  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file3
be8b003703b51e7220e52bb36439a357  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file4
```

确认无误后，就可以解压：

<font color=red>**注意：不要在共享文件夹、挂载文件夹以及非英文目录解压SDK，避免产生不必要的错误**</font>

```bash
mkdir ~/proj/
cd ~/proj/
cat path/to/rk356x_linux_release_v1.2.0_20211019_split_dir/*firefly_split* | tar -xzv
```

### 同步代码

执行如下命令同步代码：

```bash
# 进入 SDK 根目录
cd ~/proj/rk356x_linux_release_20211019/

# 同步
.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

后续可以使用以下命令更新 SDK：

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

## 编译 SDK

### 编译前配置

在 `device/rockchip/rk356x/` 目录下，有不同板型的配置文件，选择配置文件：

```bash
./build.sh 
```

配置文件会链接到 `device/rockchip/.BoardConfig.mk`，检查该文件可以验证是否配置成功。

相关配置介绍：

```bash
# Target arch
export RK_ARCH=arm64 # 64位 ARM 架构
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=rv1126_defconfig # u-boot 配置文件
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=rv1126_firefly_defconfig # kernel 配置文件
# Kernel dts
export RK_KERNEL_DTS=rv1126-firefly-jd4-mipi101-BSD1218-A101KL68.dts # dts 文件
# Buildroot config
export RK_CFG_BUILDROOT= # Buildroot 配置
# Recovery config
export RK_CFG_RECOVERY=rockchip_rk356x_recovery # recovery 配置
# parameter for GPT table
export RK_PARAMETER=parameter-buildroot-fit.txt # 分区表
# rootfs image path
export RK_ROOTFS_IMG=rockdev/rootfs.${RK_ROOTFS_TYPE} # 根文件系统路径
```

### 部分编译

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

### 打包固件

更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```

### 全自动编译

全自动编译会执行上述编译、打包操作，生成完整固件。

```bash
./build.sh
```

## 分区说明

### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 `parameter-buildroot-fit.txt` 为例：

路径：`device/rockchip/rk356x/parameter-buildroot-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3588
MACHINE_ID: 007
MANUFACTURER: RK3588
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00010000@0x00008000(boot),0x00010000@0x00018000(recovery),0x00010000@0x00028000(backup),0x00040000@0x00038000(oem),0x00c00000@0x00078000(rootfs),-@0x00c78000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

路径：`tools/linux/Linux_Pack_Firmware/rockdev/rk356x-package-file`

```bash
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
uboot           Image/uboot.img
misc            Image/misc.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
oem             Image/oem.img
userdata        Image/userdata.img
backup          RESERVED
```