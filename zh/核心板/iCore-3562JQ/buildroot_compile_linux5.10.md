## 编译 Buildroot 固件

本章介绍 Buildroot 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

### 编译 SDK

#### 编译前配置

不同板型的配置文件存放在`device/rockchip/rk3562/`目录下

回到 SDK 根目录执行`build.sh`选择配置文件：

```bash
./build.sh firefly_rk3562_aio-3562jq_buildroot_defconfig
```

配置文件会链接到 `output/defconfig`，检查该文件可以验证是否配置成功。

相关配置介绍：

```bash
# 设备树名称
RK_KERNEL_DTS_NAME="rk3562-firefly-aio-3562jq"
# 内核配置文件
RK_KERNEL_CFG="firefly_linux_defconfig"
# 使用 FIT 格式镜像
RK_USE_FIT_IMG=y
# 蓝牙用到的 UART
RK_WIFIBT_TTY="ttyS8"
# 分区表
RK_PARAMETER="parameter-buildroot-fit.txt"
# boot 分区镜像的 Image tree source
RK_BOOT_FIT_ITS="bootramdisk.its"
# ramdisk 镜像
RK_RAMDISK_IMG="ramdisk.img"
# 使用 extlinux 的方式引导系统
USE_EXTBOOT=y
```

#### 全自动编译

全自动编译会执行所有编译、打包操作，生成完整固件。

```bash
./build.sh all
```
完整固件会保存到 `output/update/` 目录。

#### 部分编译

* 编译 u-boot

```bash
./build.sh uboot
```
生成的镜像为 u-boot/uboot.img

* 编译 kernel

注意：Firefly kernel 没有开启全部的内核功能，有需求请查看[Kernel 使用](kernel_introduction.md)

```bash
./build.sh extboot
```
生成的镜像为 kernel/extboot.img

* 编译 recovery

```bash
./build.sh recovery
```
生成结果在 output/recovery 下

* 编译 Buildroot 根文件系统

```bash
./build.sh buildroot

# 注：确保作为普通用户编译 Buildroot 根文件系统，避免不必要的错误。
```
编译目录在 buildroot/output/ 下

#### 打包固件

```bash
./build.sh updateimg
```
打包固件，生成的完整固件会保存到 `output/update/` 目录。

### 分区说明

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 `parameter-buildroot-fit.txt` 为例：

路径：`device/rockchip/rk3562/parameter-buildroot-fit.txt`

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
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00040000@0x00048000(recovery),0x00010000@0x00088000(backup),0x00040000@0x00098000(oem),0x00c00000@0x000d8000(rootfs),-@0x00cd8000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。单位是块，每块 512 字节。