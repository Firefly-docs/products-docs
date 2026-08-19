# 编译 Linux 固件 (内核版本 6.1)

## 获取 SDK

请联系销售(sales@t-firefly.com)获取 SDK 下载链接。

<font color=red>

**注意：**

**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**

**2. 编译环境请使用 Ubuntu20.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**

**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**

**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**
</font>

准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以`~/proj`为例

### 校验与解压

下载完成后先验证一下 MD5 码：

```bash
$ md5sum rk3562_6.1.tgz.split*
```

将得到的结果和 md5sum.txt 中的内容进行对比，确认无误后，就可以解压：
```bash
# 解压
cd ~/proj/
cat rk3562_6.1.tgz.split0* | tar -zx

# 解压后得到
~/proj/rk3562_6.1

# 安装 python2
sudo apt update && sudo apt install python2

# 从 repo 中提取文件
cd ~/proj/rk3562_6.1
python2 .repo/repo/repo sync -l --no-tags
```
## SDK 配置介绍

### 目录介绍

```bash
$ tree -L 1
.
├── app
├── buildroot                                               # buildroot
├── build.sh -> device/rockchip/common/scripts/build.sh     # 编译脚本
├── device                                                  # 编译系统，内含编译配置文件
├── docs                                                    # 开发文档
├── external                                                # 一些组件
├── kernel                                                  # 内核
├── Makefile -> device/rockchip/common/Makefile
├── prebuilt_rootfs                                         # 用于存放预编译好的文件系统
├── prebuilts                                               # 用于存放交叉编译工具链
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
├── tools                                                   # 工具，包含烧录工具等
└── u-boot                                                  # uboot
```

### 配置文件介绍

在 `device/rockchip/rk3562/` 目录下，有不同板型的配置文件(xxxx_defconfig)，用于管理 SDK 每个环节的编译配置。

配置文件命名规则：
```
<vendor>_<chip>_<model>-<extra>_<OS>_defconfig

vendor: 产品供应商
chip: 产品所用的芯片
model: 产品型号，表示这个配置文件是用于该型号的产品
extra: 额外属性，可以为空
OS: 产品将会运行的操作系统
```

配置文件内容介绍，比如 firefly_rk3562_aio-3562jq_debian_defconfig :

```bash
RK_WIFIBT_CHIP="AP6256"
RK_UBOOT_SPL=y
RK_KERNEL_DTS_NAME="rk3562-firefly-aio-3562jq"                   # 指定编译内核所使用的是设备树
RK_KERNEL_CFG_FRAGMENTS="firefly_linux.config"                   # 指定编译内核所使用的 config fragments
RK_UBOOT_CFG_FRAGMENTS="firefly-linux"                           # 指定编译 uboot 所使用的 config fragments
RK_PARAMETER="parameter-debian-fit.txt"                          # 指定固件分区表
RK_USE_FIT_IMG=y
RK_RAMDISK_IMG="kernel/ramdisk.img"
USE_EXTBOOT=y                                                    # 使用 exlinux 风格的 boot.img
RK_RECOVERY_RAMDISK="rk3562-recovery-arm64.cpio.gz"
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk3562_debian_rootfs.img"   # 指定要使用的预编译文件系统
RK_MISC_RECOVERY=y
RK_EXTRA_PARTITION_NUM=1
```

### 分区说明

parameter 文件中包含了固件的分区信息，比如 `device/rockchip/rk3562/parameter-xxxxxx-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3562
MACHINE_ID: 007
MANUFACTURER: RK3562
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
GROW_ALIGN: 0
CMDLINE: mtdparts=:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00040000@0x00048000(recovery),0x00010000@0x00088000(backup),0x00c00000@0x00098000(rootfs),-@0x00c98000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为 uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。分区的起始位置 + 分区大小 = 下一个分区的起始位置。

parameter 中数字的单位是 “块”，每块 512 字节。所以 uboot 分区大小为 0x00002000，即 8192 块，共 8192 x 512 / 1024 / 1024 = 4 MiB

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

下载根文件系统：[Debian 根文件系统](https://www.t-firefly.com/doc/download/242.html#other_844)，请使用网盘中 kernel-6.1 目录下的文件系统。

下载后将文件系统解压到 SDK/prebuilt_rootfs/ 下，并创建链接

```bash
# 解压
7z x debian12_xxxx_rootfs_xxxx.7z

# 将解压后的 rootfs 镜像移动到 sdk 并创建一个符号链接
mv debian12_xxxx_rootfs_xxxx.img ./rk3562_6.1/prebuilt_rootfs/
cd ./rk3562_6.1/prebuilt_rootfs/
ln -sf debian12_xxxx_rtoofs_xxxx.img rk3562_debian_rootfs.img
cd ..
```

### 编译前配置

执行 `./build.sh lunch` 列出所有可用的配置文件，输入编号选择对应的配置文件，注意要选择带有 debian 字样的配置文件：
```bash
./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rk3562_linux6.1_release_20241220_v1.0.0a.xml

Log colors: message notice warning error fatal

Log saved at /home4/lth/project/linux/rk3562_6.1/output/sessions/2024-12-25_18-03-39
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3562_aio-3562jq_buildroot_defconfig
3. firefly_rk3562_aio-3562jq_debian_defconfig
4. firefly_rk3562_aio-3562jq_ubuntu_defconfig
5. rockchip_rk3562_dictpen_test3_v20_defconfig
6. rockchp_rk3562_evb1_lp4x_v10_amp_defconfig
7. rockchip_rk3562_evb1_lp4x_v10_defconfig
8. rockchip_rk3562_evb2_ddr4_v10_amp_defconfig
9. rockchip_rk3562_evb2_ddr4_v10_defconfig
10. rockchip_rk3562_robot_evb1_lp4x_v10_defconfig
11. rockchip_rk3562_robot_evb2_ddr4_v10_defconfig
Which would you like? [1]:
```

### 编译

#### 完整编译

执行如下命令即可
```bash
./build.sh all
```

编译完成后会生成完整固件 `output/update/update.img`

#### 部分编译

一般来说自定义开发只涉及 kernel 和 uboot。因此，在进行过一次完整编译后，如果后续又修改了 kernel 或 uboot，可以部分编译来加快速度。

* 单独编译 u-boot，生成 u-boot/uboot.img

```bash
./build.sh uboot
```

* 单独编译 kernel，生成 kernel/extboot.img

```bash
./build.sh extboot
```

* 将各部件打包成 update.img

```bash
./build.sh updateimg
```
## 编译 Ubuntu 固件

### 准备工作

执行下面两条命令来安装需要的工具
```bash
sudo apt update

sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev p7zip-full
```

下载根文件系统：[Ubuntu 根文件系统](https://www.t-firefly.com/doc/download/242.html#other_774)，请使用网盘中 kernel-6.1 目录下的文件系统。

下载后将文件系统解压到 SDK/prebuilt_rootfs/ 下，并创建链接

```bash
# 解压
7z x Ubuntu22.04-Xfce_xxxx_xxxx.7z

# 将解压后的 rootfs 镜像移动到 sdk 并创建一个符号链接
mv Ubuntu22.04-Xfce_xxxx_xxxx.img ./rk3562_6.1/prebuilt_rootfs/
cd ./rk3562_6.1/prebuilt_rootfs/
ln -sf Ubuntu22.04-Xfce_xxxx_xxxx.img rk3562_ubuntu_rootfs.img
cd ..
```

### 编译前配置

执行 `./build.sh lunch` 列出所有可用的配置文件，输入编号选择对应的配置文件，注意要选择带有 ubuntu 字样的配置文件：
```bash
./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rk3562_linux6.1_release_20241220_v1.0.0a.xml

Log colors: message notice warning error fatal

Log saved at /home4/lth/project/linux/rk3562_6.1/output/sessions/2024-12-25_18-03-39
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3562_aio-3562jq_buildroot_defconfig
3. firefly_rk3562_aio-3562jq_debian_defconfig
4. firefly_rk3562_aio-3562jq_ubuntu_defconfig
5. rockchip_rk3562_dictpen_test3_v20_defconfig
6. rockchp_rk3562_evb1_lp4x_v10_amp_defconfig
7. rockchip_rk3562_evb1_lp4x_v10_defconfig
8. rockchip_rk3562_evb2_ddr4_v10_amp_defconfig
9. rockchip_rk3562_evb2_ddr4_v10_defconfig
10. rockchip_rk3562_robot_evb1_lp4x_v10_defconfig
11. rockchip_rk3562_robot_evb2_ddr4_v10_defconfig
Which would you like? [1]:
```

### 编译

#### 完整编译

执行如下命令即可
```bash
./build.sh all
```

编译完成后会生成完整固件 `output/update/update.img`

#### 部分编译

一般来说自定义开发只涉及 kernel 和 uboot。因此，在进行过一次完整编译后，如果后续又修改了 kernel 或 uboot，可以部分编译来加快速度。

* 单独编译 u-boot，生成 u-boot/uboot.img

```bash
./build.sh uboot
```

* 单独编译 kernel，生成 kernel/extboot.img

```bash
./build.sh extboot
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

### 编译前配置

执行 `./build.sh lunch` 列出所有可用的配置文件，输入编号选择对应的配置文件，注意要选择带有 buildroot 字样的配置文件：
```bash
./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rk3562_linux6.1_release_20241220_v1.0.0a.xml

Log colors: message notice warning error fatal

Log saved at /home4/lth/project/linux/rk3562_6.1/output/sessions/2024-12-25_18-03-39
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3562_aio-3562jq_buildroot_defconfig
3. firefly_rk3562_aio-3562jq_debian_defconfig
4. firefly_rk3562_aio-3562jq_ubuntu_defconfig
5. rockchip_rk3562_dictpen_test3_v20_defconfig
6. rockchp_rk3562_evb1_lp4x_v10_amp_defconfig
7. rockchip_rk3562_evb1_lp4x_v10_defconfig
8. rockchip_rk3562_evb2_ddr4_v10_amp_defconfig
9. rockchip_rk3562_evb2_ddr4_v10_defconfig
10. rockchip_rk3562_robot_evb1_lp4x_v10_defconfig
11. rockchip_rk3562_robot_evb2_ddr4_v10_defconfig
Which would you like? [1]:

```

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

* 单独编译 kernel，生成 kernel/extboot.img

```bash
./build.sh extboot
```

* 将各部件打包成 update.img

```bash
./build.sh updateimg
```

但是如果你修改了 buildroot，需要重新进行完整编译。