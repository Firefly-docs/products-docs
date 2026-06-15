# 编译 Linux 固件 (内核版本 6.1)

## 获取 SDK

请联系销售(sales@t-firefly.com)获取 SDK 下载链接, 并且阅读下载链接的 readme 文档。

<font color=red>

**注意：**

**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**

**2. 编译环境请使用 Ubuntu20.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**

**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**

**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**
</font>## SDK 配置介绍

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
├── kernel -> kernel-6.1
├── kernel-6.1                                              # 内核
├── Makefile -> device/rockchip/common/Makefile
├── prebuilt_rootfs                                         # 用于存放预编译好的文件系统
├── prebuilts                                               # 用于存放交叉编译工具链
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
├── tools                                                   # 工具，包含烧录工具等
└── u-boot                                                  # uboot
```

### 配置文件介绍

在 `device/rockchip/rk3588/` 目录下，有不同板型的配置文件(xxxx_defconfig)，用于管理 SDK 每个环节的编译配置。

配置文件命名规则：
```
<vendor>_<chip>_<model>-<extra>_<OS>_defconfig

vendor: 产品供应商
chip: 产品所用的芯片
model: 产品型号，表示这个配置文件是用于该型号的产品
extra: 额外属性，可以为空
OS: 产品将会运行的操作系统
```

配置文件内容介绍，比如 firefly_rk3588_itx-3588j_debian_defconfig :

```bash
RK_KERNEL_DTS_NAME="rk3588-firefly-itx-3588j"                    # 指定编译内核所使用的设备树
RK_KERNEL_CFG_FRAGMENTS="firefly-linux.config"                   # 指定编译内核所使用的 config fragments
RK_UBOOT_CFG_FRAGMENTS="firefly-linux"                           # 指定编译 uboot 所使用的 config fragments
RK_USE_FIT_IMG=y
RK_BOOT_FIT_ITS_NAME="bootramdisk.its"
RK_PARAMETER="parameter-debian-fit.txt"                          # 指定固件分区表
RK_PRODUCT_MODEL="ITX-3588J"                                     # 产品名称/型号
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk3588_debian_rootfs.img"   # 指定要使用的预编译文件系统
RK_MISC_RECOVERY=y
RK_RECOVERY_RAMDISK="rk3588-recovery-arm64.cpio.gz"
RK_RAMDISK_IMG="kernel/ramdisk.img"
USE_EXTBOOT=y                                                    # 使用 exlinux 风格的 boot.img
RK_EXTRA_PARTITION_NUM=1
```

### 分区说明

parameter 文件中包含了固件的分区信息，比如 `device/rockchip/rk3588/parameter-xxxxxx-fit.txt`

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
GROW_ALIGN: 0
CMDLINE: mtdparts=:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00040000@0x00048000(recovery),0x00010000@0x00088000(backup),0x01c00000@0x00098000(rootfs),-@0x01c98000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
uuid:boot=7A3F0000-0000-446A-8000-702F00006273
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为 uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。分区的起始位置 + 分区大小 = 下一个分区的起始位置。

parameter 中数字的单位是 “块”，每块 512 字节。所以 uboot 分区大小为 0x00002000，即 8192 块，共 8192 x 512 / 1024 / 1024 = 4 MiB

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

下载根文件系统：[Ubuntu 根文件系统(64位) Kernel6.1](https://www.t-firefly.com/doc/download/290.html)，请使用网盘中 kernel-6.1 目录下的文件系统。

下载后将文件系统解压到 SDK/prebuilt_rootfs/ 下，并创建链接

```bash
# 解压
7z x Ubuntu22.04-xxxx.7z


mkdir ./SDK/prebuilt_rootfs/
mv Ubuntu22.04-xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf Ubuntu22.04-xxxx.img rk3588_ubuntu_rootfs.img
cd ..
```

### 编译前配置

执行 `./build.sh lunch` 列出所有可用的配置文件，输入编号选择对应的配置文件，注意要选择带有 ubuntu 字样的配置文件：
```bash
./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rk3588_linux6.1_release_20250114_v1.1.1c

Log colors: message notice warning error fatal

Log saved at /home2/daijh/p/rk3588/sdk_6.1/output/sessions/2025-01-14_14-39-08
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3588_aio-3588jd4_buildroot_defconfig
3. firefly_rk3588_aio-3588jd4_debian_defconfig
4. firefly_rk3588_aio-3588jd4_ubuntu_defconfig
5. firefly_rk3588_aio-3588l_debian_defconfig
6. firefly_rk3588_aio-3588l_ubuntu_defconfig
7. firefly_rk3588_aio-3588q_debian_defconfig
8. firefly_rk3588_aio-3588q_ubuntu_defconfig
9. firefly_rk3588_aio-3588sjd4_debian_defconfig
10. firefly_rk3588_aio-3588sjd4_ubuntu_defconfig
11. firefly_rk3588_itx-3588j_buildroot_defconfig
12. firefly_rk3588_itx-3588j_debian_defconfig
13. firefly_rk3588_itx-3588j_ubuntu_defconfig
14. firefly_rk3588_roc-rk3588-rt-ext-10ge_debian_defconfig
15. firefly_rk3588_roc-rk3588-rt-ext-10ge_ubuntu_defconfig
16. firefly_rk3588_roc-rk3588-rt-ext-2g5e_debian_defconfig
17. firefly_rk3588_roc-rk3588-rt-ext-2g5e_ubuntu_defconfig
18. firefly_rk3588_roc-rk3588-rt_debian_defconfig
19. firefly_rk3588_roc-rk3588-rt_ubuntu_defconfig
20. firefly_rk3588_roc-rk3588s-pc-ext_debian_defconfig
21. firefly_rk3588_roc-rk3588s-pc-ext_ubuntu_defconfig
22. firefly_rk3588_roc-rk3588s-pc_debian_defconfig
23. firefly_rk3588_roc-rk3588s-pc_ubuntu_defconfig
24. rockchip_rk3588_evb1_lp4_v10_defconfig
25. rockchip_rk3588_evb7_v11_defconfig
26. rockchip_rk3588_ipc_evb1_v10_defconfig
27. rockchip_rk3588_ipc_evb7_lp4_v11_defconfig
28. rockchip_rk3588_multi_ipc_evb1_v10_defconfig
29. rockchip_rk3588s_evb1_lp4x_v10_defconfig
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

下载根文件系统：[Debian 根文件系统(64位)](https://www.t-firefly.com/doc/download/161.html#other_717)，请使用网盘中 kernel-6.1 目录下的文件系统。

下载后将文件系统解压到 SDK/prebuilt_rootfs/ 下，并创建链接

```bash
# 解压
7z x debian12_xxxx_rootfs_xxxx.7z

# 将解压后的 rootfs 镜像移动到 sdk 并创建一个符号链接
mkdir ./SDK/prebuilt_rootfs/
mv debian12_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf debian12_xxxx_rtoofs_xxxx.img rk3588_debian_rootfs.img
cd ..
```

### 编译前配置

执行 `./build.sh lunch` 列出所有可用的配置文件，输入编号选择对应的配置文件，注意要选择带有 debian 字样的配置文件：
```bash
./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rk3588_linux6.1_release_20241203_v1.1.1b.xml

Log colors: message notice warning error fatal

Log saved at /home4/lth/project/linux/rk3588_6.1_release/rk3588_6.1/output/sessions/2024-12-11_09-15-44
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3588_aio-3588l_debian_defconfig
3. firefly_rk3588_aio-3588q_debian_defconfig
4. firefly_rk3588_aio-3588sjd4_debian_defconfig
5. firefly_rk3588_itx-3588j_buildroot_defconfig
6. firefly_rk3588_itx-3588j_debian_defconfig
7. firefly_rk3588_roc-rk3588-rt-ext-10ge_debian_defconfig
8. firefly_rk3588_roc-rk3588-rt-ext-2g5e_debian_defconfig
9. firefly_rk3588_roc-rk3588-rt_debian_defconfig
10. firefly_rk3588_roc-rk3588s-pc-ext_debian_defconfig
11. firefly_rk3588_roc-rk3588s-pc_debian_defconfig
12. rockchip_rk3588_evb1_lp4_v10_defconfig
13. rockchip_rk3588_evb7_v11_defconfig
14. rockchip_rk3588_ipc_evb1_v10_defconfig
15. rockchip_rk3588_ipc_evb7_lp4_v11_defconfig
16. rockchip_rk3588_multi_ipc_evb1_v10_defconfig
17. rockchip_rk3588s_evb1_lp4x_v10_defconfig
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

Manifest: rk3588_linux6.1_release_20241203_v1.1.1b.xml

Log colors: message notice warning error fatal

Log saved at /home4/lth/project/linux/rk3588_6.1_release/rk3588_6.1/output/sessions/2024-12-11_09-15-44
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3588_aio-3588l_debian_defconfig
3. firefly_rk3588_aio-3588q_debian_defconfig
4. firefly_rk3588_aio-3588sjd4_debian_defconfig
5. firefly_rk3588_itx-3588j_buildroot_defconfig
6. firefly_rk3588_itx-3588j_debian_defconfig
7. firefly_rk3588_roc-rk3588-rt-ext-10ge_debian_defconfig
8. firefly_rk3588_roc-rk3588-rt-ext-2g5e_debian_defconfig
9. firefly_rk3588_roc-rk3588-rt_debian_defconfig
10. firefly_rk3588_roc-rk3588s-pc-ext_debian_defconfig
11. firefly_rk3588_roc-rk3588s-pc_debian_defconfig
12. rockchip_rk3588_evb1_lp4_v10_defconfig
13. rockchip_rk3588_evb7_v11_defconfig
14. rockchip_rk3588_ipc_evb1_v10_defconfig
15. rockchip_rk3588_ipc_evb7_lp4_v11_defconfig
16. rockchip_rk3588_multi_ipc_evb1_v10_defconfig
17. rockchip_rk3588s_evb1_lp4x_v10_defconfig
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
## 编译 Openeuler 固件

### 准备工作

执行下面两条命令来安装需要的工具
```bash
sudo apt update

sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev p7zip-full
```

下载根文件系统：[Openeuler 根文件系统(64位) Kernel6.1](https://www.t-firefly.com/doc/download/290.html)，请使用网盘中 kernel-6.1 目录下的文件系统。

下载后将文件系统解压到 SDK/prebuilt_rootfs/ 下，并创建链接

```bash
# 解压
7z x Openeuler24.03-xxxx.7z


mkdir ./SDK/prebuilt_rootfs/
mv Openeuler24.03-xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf Openeuler24.03-xxxx.img rk3588_openeuler_rootfs.img
cd ..
```

### 编译前配置

执行 `./build.sh lunch` 列出所有可用的配置文件，输入编号选择对应的配置文件，注意要选择带有 Openeuler 字样的配置文件：
```bash
./build.sh lunch

############### Rockchip Linux SDK ###############

Manifest: rk3588_linux6.1_release_20250114_v1.1.1c

Log colors: message notice warning error fatal

Log saved at /home2/daijh/p/rk3588/sdk_6.1/output/sessions/2025-01-14_14-39-08
Pick a defconfig:

1. rockchip_defconfig
2. firefly_rk3588_aio-3588jd4_buildroot_defconfig
3. firefly_rk3588_aio-3588jd4_debian_defconfig
4. firefly_rk3588_aio-3588jd4_ubuntu_defconfig
5. firefly_rk3588_aio-3588l_debian_defconfig
6. firefly_rk3588_aio-3588l_ubuntu_defconfig
7. firefly_rk3588_aio-3588q_debian_defconfig
8. firefly_rk3588_aio-3588q_ubuntu_defconfig
9. firefly_rk3588_aio-3588sjd4_debian_defconfig
10. firefly_rk3588_aio-3588sjd4_ubuntu_defconfig
11. firefly_rk3588_itx-3588j_buildroot_defconfig
12. firefly_rk3588_itx-3588j_debian_defconfig
13. firefly_rk3588_itx-3588j_ubuntu_defconfig
14. firefly_rk3588_roc-rk3588-rt-ext-10ge_debian_defconfig
15. firefly_rk3588_roc-rk3588-rt-ext-10ge_ubuntu_defconfig
16. firefly_rk3588_roc-rk3588-rt-ext-2g5e_debian_defconfig
17. firefly_rk3588_roc-rk3588-rt-ext-2g5e_ubuntu_defconfig
18. firefly_rk3588_roc-rk3588-rt_debian_defconfig
19. firefly_rk3588_roc-rk3588-rt_ubuntu_defconfig
20. firefly_rk3588_roc-rk3588s-pc-ext_debian_defconfig
21. firefly_rk3588_roc-rk3588s-pc-ext_ubuntu_defconfig
22. firefly_rk3588_roc-rk3588s-pc_debian_defconfig
23. firefly_rk3588_roc-rk3588s-pc_ubuntu_defconfig
24. rockchip_rk3588_evb1_lp4_v10_defconfig
25. rockchip_rk3588_evb7_v11_defconfig
26. rockchip_rk3588_ipc_evb1_v10_defconfig
27. rockchip_rk3588_ipc_evb7_lp4_v11_defconfig
28. rockchip_rk3588_multi_ipc_evb1_v10_defconfig
29. rockchip_rk3588s_evb1_lp4x_v10_defconfig
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