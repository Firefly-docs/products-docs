# 编译 Linux 固件

<CodeBlockTabs defaultValue="k612">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="k612">内核 6.12</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="k612">
## 获取 SDK [step]

首先准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以`~/proj`为例

<Callout title="编译说明" type="info">

**注意：**

**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**

**2. 编译环境请使用 Ubuntu22.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**

**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**

**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**

</Callout>
**SDK 配置介绍**

<Accordions>

<Accordion title="目录介绍" type="single" id="sdk612_dir">

```bash
$ tree -L 1
.
├── app                                                                       # sdk 自带的应用示例或工具
├── buildroot                                                                 # 构建 buildroot 根文件系统,当前sdk使用外部构建完成的
├── build.sh -> device/rockchip/common/scripts/build.sh        		          # 固件编译脚本
├── common -> device/rockchip/common                           		          # 编译配置文件
├── debian                                                     		          # 构建 debian 根文件系统,当前sdk使用外部构建完成的
├── device                                                     		          # 编译配置文件
├── docs                                                       		          # 开发文档
├── external                                                                  # 开发组件
├── firefly-build.sh -> device/rockchip/common/scripts/firefly-build.sh       # firefly的构建脚本
├── kernel -> kernel-6.12                                                     # kernel 6.12 (软链接)
├── kernel-6.12								                                 # kernel 6.12
├── Makefile -> device/rockchip/common/Makefile                              
├── output
├── prebuilt_rootfs                                                           # 存放外部构建完成的根文件系统镜像
├── prebuilts                                                                 # 交叉编译工具链
├── README.md -> device/rockchip/common/README.md                          
├── rkbin                                                                     # rockchip 的闭源二进制文件
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh                   # 烧录脚本
├── rockdev -> output/firmware                                                # firmware 输出路径
├── tools                                                                     # 工具
├── u-boot                                                                    # u-boot 2025.04 源码
└── yocto                                                                     # yocto 构建系统时才使用
```

</Accordion>

<Accordion title="配置文件介绍" type="single" id="sdk612_config">

在 `device/rockchip/rk3588/` 目录下，有不同板型的配置文件(xxxx_defconfig)，用于管理 SDK 每个环节的编译配置。

配置文件命名规则：
```
<vendor>_<chip>_<model>-<extra>_defconfig
vendor: 产品供应商
chip: 产品所用的芯片
model: 产品型号，表示这个配置文件是用于该型号的产品
extra: 额外属性，可以为空
```

配置文件内容介绍，比如 firefly_rk3588_lga-3588l-mini-duo_rk182x_defconfig :

```bash
#include "firefly.config"                                             # 引入公共配置
RK_KERNEL_DTS_NAME="rk3588-firefly-lga-3588l-mini-duo-rk182x"         # 指定编译内核所使用的设备树
RK_PRODUCT_MODEL="LGA-3588L-MINI-DUO-RK182X"                          # 产品名称/型号
RK_KERNEL_CFG_FRAGMENTS+="firefly-rk182x.config firefly-net.config"   # 指定编译内核所使用的配置片段
RK_EXTRA_PARTITION_NUM=1                                              # 额外分区数量
RK_EXTRA_PARTITION_1_DEV="oem"                                        # oem 分区
RK_EXTRA_PARTITION_1_NAME="oem"                                       # 分区名
RK_EXTRA_PARTITION_1_FSTYPE="ext4"                                    # oem 分区的文件系统类型
RK_EXTRA_PARTITION_1_SRC="rk1828-policy"                              # 目录位置
RK_EXTRA_PARTITION_1_SIZE="128M"                                      # 分区大小
```

</Accordion>

<Accordion title="分区说明" type="single" id="sdk612_partition">

parameter 文件中包含了固件的分区信息，比如 `device/rockchip/rk3588/parameter-firefly.txt`

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
CMDLINE: mtdparts=:0x00003000@0x00004000(uboot),\
		           0x00002000@0x00007000(misc),\
		           0x00080000@0x00009000(boot:bootable),\
		           0x00040000@0x00089000(recovery),\
		           0x00010000@0x000c9000(backup),\
		           0x01000000@0x000d9000(rootfs),\
		           0x00040000@0x010d9000(oem),\
		           -@0x01129000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
uuid:boot=7A3F0000-0000-446A-8000-702F00006273
```
<Callout title="parameter参数说明" type="info">

CMDLINE 属性是我们关注的地方，以 uboot 为例,0x00003000@0x00004000(uboot) 中 0x00004000 为 uboot 分区的起始位置，0x00003000 为分区的大小，以此类推。分区的起始位置 + 分区大小 = 下一个分区的起始位置。

parameter 中数字的单位是 “块”，每块 512 字节。所以 uboot 分区大小为 0x00003000，即 12288 块，共 12288 x 512 / 1024 / 1024 = 6 MiB

</Callout>

</Accordion>

</Accordions>
## 编译固件 [step]

  <CodeBlockTabs defaultValue="ubuntu">
    <CodeBlockTabsList>
      <CodeBlockTabsTrigger value="ubuntu">ubuntu</CodeBlockTabsTrigger>
      <CodeBlockTabsTrigger value="debian">debian</CodeBlockTabsTrigger>
    </CodeBlockTabsList>

    <CodeBlockTab value="ubuntu">
编译 Ubuntu 固件

### 准备工作 [step]

执行下面两条命令来安装需要的工具
```bash
sudo apt update
sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev p7zip-full
```

下载根文件系统：[Ubuntu24.04 根文件系统(64位)](https://community.t-firefly.com/doc/download/440)，请使用网盘中 Kernel-6.12 目录下的文件系统。

下载后将文件系统解压到 SDK/prebuilt_rootfs/ 下，软链接在「选择 rootfs」步骤中创建。

```bash
# 解压
7z x Ubuntu24.04-xxxx.7z
mkdir ./SDK/prebuilt_rootfs/
mv Ubuntu24.04-xxxx.img ./SDK/prebuilt_rootfs/
```

### 编译前配置 [step]

执行 `./build.sh lunch` 列出所有可用的配置文件，输入编号选择对应板型的配置文件：
```bash
./build.sh lunch
\n############### Rockchip Linux SDK ###############\n
Manifest: common_linux6.12_next.xml
GIT commit: "aa3794d3 fix: del lga-3588l-mini-duo"
Log colors: message notice warning error fatal
Log saved at /home2/zhangyw/linux_612/output/sessions/2026-11-24_17-40-42
Pick a defconfig:\n
1. rockchip_defconfig
2. firefly_rk3588_aio-3588jd4_defconfig
3. firefly_rk3588_aio-3588l-qua_rk182x_defconfig
4. firefly_rk3588_aio-3588l_defconfig
5. firefly_rk3588_aio-3588q_defconfig
6. firefly_rk3588_aio-3588sg_defconfig
7. firefly_rk3588_aio-3588sjd4_defconfig
8. firefly_rk3588_itx-3588j_defconfig
9. firefly_rk3588_lga-3588l-mini-duo_rk182x_defconfig
10. firefly_rk3588_roc-rk3588-rt_defconfig
11. firefly_rk3588_roc-rk3588s-pc_defconfig
12. rockchip_rk3588_evb10_v10_defconfig
13. rockchip_rk3588_evb1_lp4_v10_amp_defconfig
14. rockchip_rk3588_evb1_lp4_v10_defconfig
15. rockchip_rk3588_evb1_lp4_v10_mcu_defconfig
16. rockchip_rk3588_evb7_v11_defconfig
17. rockchip_rk3588_ipc_evb1_v10_defconfig
18. rockchip_rk3588_ipc_evb7_lp4_v11_defconfig
19. rockchip_rk3588_multi_ipc_evb1_v10_defconfig
20. rockchip_rk3588_thunderboot_evb7_v11_defconfig
21. rockchip_rk3588s_evb1_lp4x_v10_defconfig
Which would you like? [1]: 
```

### 选择 rootfs [step]

在 `prebuilt_rootfs/` 下创建 `rootfs.img` 软链接，指向下载的根文件系统镜像：

```bash
cd ./SDK/prebuilt_rootfs/
ln -sf Ubuntu24.04-xxxx.img rootfs.img
cd ..
```

### 编译 [step]

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

* 单独编译 kernel，生成 kernel/extboot.img.

```bash
./build.sh extboot
```

* 将各部件打包成 update.img

```bash
./build.sh updateimg
```
    </CodeBlockTab>

    <CodeBlockTab value="debian">
编译 Debian 固件

### 准备工作 [step]

执行下面两条命令来安装需要的工具
```bash
sudo apt update
sudo apt install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler ncurses-dev p7zip-full
```

下载根文件系统：[Debian13 根文件系统(64位)](https://community.t-firefly.com/doc/download/440)，请使用网盘中 Kernel-6.12 目录下的文件系统。

下载后将文件系统解压到 SDK/prebuilt_rootfs/ 下，软链接在「选择 rootfs」步骤中创建。

```bash
# 解压
7z x debian13_xxxx_rootfs_xxxx.7z
# 将解压后的 rootfs 镜像移动到 sdk
mkdir ./SDK/prebuilt_rootfs/
mv debian13_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
```

### 编译前配置 [step]

执行 `./build.sh lunch` 列出所有可用的配置文件，输入编号选择对应板型的配置文件：
```bash
./build.sh lunch
\n############### Rockchip Linux SDK ###############\n
Manifest: common_linux6.12_next.xml
GIT commit: "aa3794d3 fix: del lga-3588l-mini-duo"
Log colors: message notice warning error fatal
Log saved at /home2/zhangyw/linux_612/output/sessions/2026-11-24_17-40-42
Pick a defconfig:\n
1. rockchip_defconfig
2. firefly_rk3588_aio-3588jd4_defconfig
3. firefly_rk3588_aio-3588l-qua_rk182x_defconfig
4. firefly_rk3588_aio-3588l_defconfig
5. firefly_rk3588_aio-3588q_defconfig
6. firefly_rk3588_aio-3588sg_defconfig
7. firefly_rk3588_aio-3588sjd4_defconfig
8. firefly_rk3588_itx-3588j_defconfig
9. firefly_rk3588_lga-3588l-mini-duo_rk182x_defconfig
10. firefly_rk3588_roc-rk3588-rt_defconfig
11. firefly_rk3588_roc-rk3588s-pc_defconfig
12. rockchip_rk3588_evb10_v10_defconfig
13. rockchip_rk3588_evb1_lp4_v10_amp_defconfig
14. rockchip_rk3588_evb1_lp4_v10_defconfig
15. rockchip_rk3588_evb1_lp4_v10_mcu_defconfig
16. rockchip_rk3588_evb7_v11_defconfig
17. rockchip_rk3588_ipc_evb1_v10_defconfig
18. rockchip_rk3588_ipc_evb7_lp4_v11_defconfig
19. rockchip_rk3588_multi_ipc_evb1_v10_defconfig
20. rockchip_rk3588_thunderboot_evb7_v11_defconfig
21. rockchip_rk3588s_evb1_lp4x_v10_defconfig
Which would you like? [1]: 
```

### 选择 rootfs [step]

在 `prebuilt_rootfs/` 下创建 `rootfs.img` 软链接，指向下载的根文件系统镜像：

```bash
cd ./SDK/prebuilt_rootfs/
ln -sf debian13_xxxx_rootfs_xxxx.img rootfs.img
cd ..
```

### 编译 [step]

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
    </CodeBlockTab>
  </CodeBlockTabs>
  </CodeBlockTab>
</CodeBlockTabs>