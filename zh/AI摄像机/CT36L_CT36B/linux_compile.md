# 编译 Linux 固件


## 获取 SDK

首先准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以`~/proj`为例

<font color=red>

**注意：**

**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**

**2. 编译环境请使用 Ubuntu18.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**

**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**

**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**
</font>

### 安装工具
获取 SDK 需要先安装：
```
sudo apt update
sudo apt install -y repo git python
```
### 初始化仓库

* 方法一（推荐国内用户使用）

**SDK 源码放置于压缩包内**

下载完成后先验证一下 MD5 码：

```bash
lvsx@tchip16:~ $ md5sum T36X.7z
a0aba53273aedef9ca085266bdad8806  T36X.7z
```

确认无误后，就可以解压：

```bash
7x x T36X.7z
```


## Linux IPC SDK 配置介绍

### SDK 目录结构说明

| Directory Path | Introduction |
|---|---|
|build.sh|SDK编译脚本<br/>软链接到project/build.sh|
|media|多媒体编解码、ISP 等算法相关|
|sysdrv|U-Boot、kernel、rootfs目录|
|project|参考应用、编译配置以及脚本目录|
|docs|SDK 文档目录|
|tools|烧录镜像打包工具以及烧录工具|
|output|SDK 编译后镜像文件存放目录|
|output/image|烧录镜像输出目录|
|output/out|编译生成的文件|
|output/out/app_out|参考应用编译后的文件|
|output/out/media_out|media 相关编译后的文件|
|output/out/sysdrv_out|sysdrv 编译后的文件|
|output/out/sysdrv_out/kernel_drv_ko|外设和多媒体的 ko 文件|
|output/out/rootfs_xxx|文件系统打包目录|
|output/out/S20linkmount|分区挂载脚本|
|output/out/userdata|userdata|

注：media 和 sysdrv 可以独立 SDK 编译。



### Sysdrv 目录说明
sysdrv 可以独立于 SDK 进行编译，包含 U-Boot、kernel、rootfs 以及一些镜像打包工具。
编译命令：
```bash
# 默认全部编译
make all

# 编译 U-Boot
make uboot_clean
make uboot

# 编译内核
make kernel_clean
make kernel

# 编译 rootfs
make rootfs_clean
make rootfs

# 清除编译
make clean

# 清除编译并删掉out目录
make distclean

# 查看编译配置，比如uboot、kernel详细的编译命令
make info
```

|sysdrv子目录|说明|
| ------ | ---- |
| cfg |内核和U-Boot编译相关的配置|
| out |sysdrv编译输出目录|
| out/bin/board_glibc_xxx |运行在板端的程序|
| out/bin/pc |运行在 PC 端的程序|
| out/bin/image_glibc_xxx |生成的烧录镜像输出目录|
| out/bin/rootfs_glibc_xxx |根文件系统目录|
| source/busybox |busybox编译目录，源码在sysdrv/tools/board/busybox|
| source/kernel |内核源码目录|
| source/uboot |U-Boot源码目录以及rkbin（ddr初始化预编译镜像）|
| tools/board |板端程序源码|
| tools/pc |PC 端打包镜像的工具|


### 配置文件介绍

在 `project/cfg/BoardConfig_IPC/` 目录下，有不同板型的配置文件(xxxx.mk)，用于管理 SDK 每个环节的编译配置，相关配置介绍：


| 配置项                        | 说明                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| RK_ARCH                       | arm或arm64  <br />定义编译32位或64位程序                     |
| RK_CHIP                       | 不可修改<br />不同的芯片对应不同的 SDK                       |
| RK_TOOLCHAIN_CROSS            | 不可修改<br />定义交叉工具链                                 |
| RK_BOOT_MEDIUM                | emmc 或 spi_nor 或 spi_nand <br />定义板子存储类型           |
| RK_UBOOT_DEFCONFIG            | U-Boot defconfig 文件名<br />文件目录 sysdrv/source/uboot/u-boot/configs |
| RK_UBOOT_DEFCONFIG_FRAGMENT   | U-Boot config 文件名（可选）<br/>文件目录sysdrv/source/uboot/u-boot/configs<br/>对RK_UBOOT_DEFCONFIG定义的defconfig进行覆盖 |
| RK_KERNEL_DEFCONFIG           | 内核 defconfig 文件名<br/>文件目录 sysdrv/source/kernel/arch/$RK_ARCH/configs |
| RK_KERNEL_DEFCONFIG_FRAGMENT  | 内核 defconfig 文件名（可选）<br/>文件目录 sysdrv/source/kernel/arch/$RK_ARCH/configs<br/>对 RK_KERNEL_DEFCONFIG 定义的 defconfig 进行覆盖 |
| RK_KERNEL_DTS                 | 内核 dts 文件名<br/>RK_ARCH=arm 目录：<br/>sysdrv/source/kernel/arch/arm/boot/dts<br/>RK_ARCH=arm64 目录：<br/>sysdrv/source/kernel/arch/arm64/boot/dts/rockchip |
| RK_MISC                       | 如果打开 recovery 功能，系统启动时读取标志选择进<br/>recovery 系统或应用系统（没有 recovery 时，可以去掉） |
| RK_CAMERA_SENSOR_IQFILES      | Camera Sensor 的 IQ 配置文件<br/>文件目录 media/isp/camera_engine_rkaiq/iqfiles 或<br/>media/isp/camera_engine_rkaiq/rkaiq/iqfiles<br/>多个 IQ 文件用空格隔开，例如<br/>RK_CAMERA_SENSOR_IQFILES="iqfile_1 iqfile_2" |
| RK_PARTITION_CMD_IN_ENV       | 配置分区表（重要）<br/>分区表格式：<br />```<partdef>[,<partdef>]```<br/><partdef>格式：```<size>[@<offset>](part-name)```<br />详细配置参考[分区表说明章节] |
| RK_PARTITION_FS_TYPE_CFG      | 配置分区文件系统类型以及挂载点（重要）<br/>格式说明：<br/>分区名称@分区挂载点@分区文件系统类型<br/>注：根文件系统的分区挂载点默认值是IGNORE（不可<br/>修改） |
| RK_SQUASHFS_COMP              | 配置squashfs镜像压缩算法（可选）<br/>支持：lz4/lzo/lzma/xz/gzip (default xz) |
| RK_UBIFS_COMP                 | 配置ubifs镜像压缩算法（可选）<br/>支持：lzo/zlib (default lzo) |
| RK_APP_TYPE                   | 配置编译参考的应用（可选）<br/>运行./build.sh info 可以查看支持的参考应用 |
| RK_APP_IPCWEB_BACKEND         | 配置是否编译web应用（可选）<br/>y:使能                       |
| RK_BUILD_APP_TO_OEM_PARTITION | 配置是否将应用安装到 oem 分区（可选）<br/>y:使能             |
| RK_ENABLE_RECOVERY            | 配置是否使能recovery功能（可选）<br/>y:使能                  |
| RK_ENABLE_FASTBOOT            | 配置是否快速启动功能（可选）<br/>y:使能<br/>需要配合U-Boot和内核修改，可以参考SDK提供的<br/>BoardConfig-*-TB.mk |
| RK_ENABLE_GDB                 | 配置是否编译gdb（可选）<br/>y:使能                           |
| RK_ENABLE_ADBD                | 配置是否支持adb功能（可选）<br/>y:使能<br/>注：需要内核打开对应USB配置 |
| RK_BOOTARGS_CMA_SIZE          | 配置内核CMA大小（可选）                                      |
| RK_POST_BUILD_SCRIPT          | 配置的脚本将会在打包rootfs.img前执行（脚本放在<br/>BoardConfig对应目录下）（可选） |
| RK_PRE_BUILD_OEM_SCRIPT       | 配置的脚本将会在打包oem.img前执行（脚本放在<br/>BoardConfig对应目录下）（可选） |


### 分区表说明

SDK 使用 env 分区设置分区表，分区表信息配置在 ```<SDK>/project/cfg/BoardConfig_IPC/BoardConfig-SPI_NOR-NONE-T36_V10-IPC.mk``` 里的 RK_PARTITION_CMD_IN_ENV 参数中。

```bash
export RK_PARTITION_CMD_IN_ENV="64K(env),128K@64K(idblock),128K(uboot),3M(boot),3M(rootfs),7M(oem),2M(userdata),-(media)"
```



分区表以字符串的形式保存在配置中，以下是各存储介质的分区表例子。

| 存储介质                | 分区表                                                       |
| ----------------------- | ------------------------------------------------------------ |
| eMMC                    | RK_PARTITION_CMD_IN_ENV="32K(env),512K@32K(idblock),4M(uboot),32M(boot),2G(rootfs),-<br/>(userdata)" |
| spi nand 或slc<br/>nand | RK_PARTITION_CMD_IN_ENV="256K(env),256K@256K(idblock),1M(uboot),8M(boot),32M(rootfs),-<br/>(userdata)" |
| spi nor                 | RK_PARTITION_CMD_IN_ENV="64K(env),128K@64K(idblock),128K(uboot),3M(boot),6M(rootfs),-<br/>(userdata)" |

每个分区的格式为：``` <size>[@<offset>](part-name) ```，其中，分区大小和分区名是必须的，偏移量则因情况而定（见以下注意事项第三点）。在配置分区表时有以下几点注意事项：

1. 分区之间用英文逗号","隔开。
2. 分区大小的单位有：K/M/G/T/P/E，不区分大小写，无单位则默认为byte；"-"表示该分区大小为剩余容量。
3. 第一个分区若从 ```0x0``` 地址开始时，不加偏移量，反之，则必须添加偏移量。后续分区任意选择是否添加偏移量。
4. idblock 分区的偏移和大小是固定的，请勿修改。
5. 不建议修改 env 分区名。（如果要修改 env 地址和大小，需要修改对应 U-Boot 的 defconfig 配置CONFIG_ENV_OFFSET 和 CONFIG_ENV_SIZE，重新生成固件，擦除板端 0 地址的 env 数据，再烧录新的固件）
## 编译 RKIPC 固件

本章介绍 RKIPC 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。


### 准备工作

#### 搭建编译环境

```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip \
device-tree-compiler ncurses-dev \
```

### 编译 SDK

#### CT36L 编译前配置

在 `project/cfg/BoardConfig_IPC/` 目录下，有不同板型的配置文件，选择配置文件 14 ：

```bash
./build.sh lunch
...
# 此处省略其他版型的配置文件
...
----------------------------------------------------------------
14. BoardConfig_IPC/BoardConfig-SPI_NOR-NONE-RV1106_T36_V10-IPC.mk
                             boot medium(启动介质): SPI_NOR
                          power solution(电源方案): NONE
                        hardware version(硬件版本): RV1106_T36_V10
                             application(应用场景): IPC
----------------------------------------------------------------

Which would you like? [0]: 14
[build.sh:info] switching to board: /home/lvsx/project/rv1106/test/T36X/project/cfg/BoardConfig_IPC/BoardConfig-SPI_NOR-NONE-RV1106_T36_V10-IPC.mk
[build.sh:info] Running build_select_board succeeded.
```

#### CT36B 编译前配置

在 `project/cfg/BoardConfig_IPC/` 目录下，有不同板型的配置文件，选择配置文件 4 ：

```bash
./build.sh lunch
...
# 此处省略其他版型的配置文件
...
----------------------------------------------------------------
4. BoardConfig_IPC/BoardConfig-EMMC-NONE-RV1106_T36B_V10-IPC.mk
                             boot medium(启动介质): EMMC
                          power solution(电源方案): NONE
                        hardware version(硬件版本): RV1106_T36B_V10
                             application(应用场景): IPC
----------------------------------------------------------------

Which would you like? [0]: 4
[build.sh:info] switching to board: /home/lvsx/project/rv1106/test/T36X/project/cfg/BoardConfig_IPC/BoardConfig-EMMC-NONE-RV1106_T36B_V10-IPC.mk
[build.sh:info] Running build_select_board succeeded.
```

#### 编译

##### 全自动编译

全自动编译会执行上述编译、打包操作，生成 RKIPC 固件。

```bash
./build.sh
```

打包固件，生成的完整固件会保存到 `output/image/` 目录。

##### 部分编译

* 编译 u-boot

```bash
./build.sh clean uboot
./build.sh uboot

# ./build.sh info 可以查看uboot详细的编译命令格式
```
生成镜像文件：output/image/download.bin、output/image/idblock.img 和 output/image/uboot.img

* 编译 kernel

```bash
./build.sh clean kernel
./build.sh kernel

# ./build.sh info 可以查看kernel详细的编译命令格式
```
生成镜像文件：output/image/boot.img

* 编译 rootfs

```bash
./build.sh clean rootfs
./build.sh rootfs
```
编译后使用 ./build.sh firmware 命令打包成 rootfs.img
生成镜像文件：output/image/rootfs.img

* 编译 media

```bash
./build.sh clean media
./build.sh media
```
生成文件的存放目录：output/out/media_out

* 编译参考应用

```bash
./build.sh clean app
./build.sh app
```
生成文件的存放目录：output/out/app_out
注：app 依赖 media

* 编译内核驱动

```bash
./build.sh clean driver
./build.sh driver
```
生成文件的存放目录：output/out/sysdrv_out/kernel_drv_ko/


* 打包 env.img
```bash
./build.sh env
```
env.img是用uboot的mkenvimage工具进行打包。
env.img打包命令格式： 
```
mkenvimage -s $env_partition_size -p 0x0 -o env.img env.txt
```
注意：不同的存储介质，$env_partition_size 不一样，具体查看分区表说明
查看env.img内容： strings env.img
```
# 例如eMMC的env.txt内容
blkdevparts=mmcblk0:32K(env),512K@32K(idblock),256K(uboot),32M(boot),2G(rootfs),1
G(oem),2G(userdata),-(media)
sys_bootargs=root=/dev/mmcblk0p5 rk_dma_heap_cma=64M rootfstype=ext4
```
注意：不同的存储介质，env.img内容会不一样，可以用 strings env.img 查看。
blkdevparts和sys_bootargs会被uboot传递给内核，并覆盖内核对应bootargs的参数。


* 打包固件

```bash
./build.sh firmware
```
生成文件的存放目录： output/image





