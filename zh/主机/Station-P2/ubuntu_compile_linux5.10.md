## 编译 Ubuntu 固件

本章介绍 Ubuntu 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

### Ubuntu 固件简单介绍

[什么是 Ubuntu Minimal ？](ubuntu_minimal_support.md)

[什么是 Ubuntu Desktop ？](ubuntu_desktop_support.md)

### 编译 SDK

#### 编译前配置

不同板型的配置文件存放在`device/rockchip/rk3566_rk3568/`目录下

回到 SDK 根目录执行`build.sh`选择配置文件：

```bash
./build.sh 
```

配置文件会链接到 `output/defconfig`，检查该文件可以验证是否配置成功。

相关配置介绍：

```bash
# 预编译的文件系统位置
PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk356x_ubuntu_rootfs.img"
# 蓝牙用到的 UART
RK_WIFIBT_TTY="ttyS8"
# 内核配置文件
RK_KERNEL_CFG=""
# 设备树名称
RK_KERNEL_DTS_NAME=""
# ramdisk 镜像
RK_RAMDISK_IMG="ramdisk.img"
# boot 分区镜像的 Image tree source
RK_BOOT_FIT_ITS="bootramdisk.its"
# 预编译的 recovery 镜像
RK_RECOVERY_RAMDISK="rk356x-recovery-arm64.cpio.gz"
# 分区表
RK_PARAMETER="parameter-ubuntu-fit.txt"
# 使用 FIT 格式镜像
RK_USE_FIT_IMG=y
# 使用 extlinux 方式引导系统
USE_EXTBOOT=y
```

#### 选择编译配件

在 `device/rockchip/rk3566_rk3568/` 目录下，除了之外，还有其他不同配件搭配的配置文件

配置文件的名称中会标明所使用的屏幕和摄像头。如果没有标明屏幕，说明使用的是默认 HDMI 显示；如果没有标明摄像头，说明使用的是默认单目摄像头

```bash
# 例如：
xxxx-ubuntu_defconfig                      # 使用 HDMI + 单目摄像头
xxxx-2cam_ubuntu_defconfig                 # 使用 HDMI + 双目摄像头
xxxx-mipi_ubuntu_defconfig                 # 使用 mipi + 单目摄像头
xxxx-mipi_ubuntu_defconfig                 # 使用 mipi + 双目摄像头
```

选择好配置文件后，执行`build.sh`来使其生效：

```bash
# 例如：
./build.sh xxxx-mipi_ubuntu_defconfig
```

#### 下载 Ubuntu 根文件系统

* 下载根文件系统：[Ubuntu 根文件系统(64位)]()，放到 SDK 路径下

* 解压文件

```bash
# 假设下载的压缩包叫 Ubuntu20.04-xxx_RK3568_KERNEL-5.10_xxx.7z
7z x Ubuntu20.04-xxx_RK3568_KERNEL-5.10_xxx.7z
```

* 将解压出的根文件系统放到 `prebuilt_rootfs/` 目录下

```bash
mkdir prebuilt_rootfs
# 假设解压出的文件系统叫做 Ubuntu20.04-xxx_RK3568_KERNEL-5.10_xxx.img
mv Ubuntu20.04-xxx_RK3568_KERNEL-5.10_xxx.img prebuilt_rootfs/
cd prebuilt_rootfs
ln -sf Ubuntu20.04-xxx_RK3568_KERNEL-5.10_xxx.img rk356x_ubuntu_rootfs.img
```

#### 全自动编译

全自动编译会执行所有编译、打包操作，直接生成 RK 固件。

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

* 打包固件

```bash
./build.sh updateimg
```
打包固件，生成的完整固件会保存到 `output/update/` 目录。

### 分区说明

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 parameter-ubuntu-fit.txt 为例：

路径：`device/rockchip/rk3566_rk3568/parameter-ubuntu-fit.txt`

```bash
FIRMWARE_VER: 1.0
MACHINE_MODEL: RK3568
MACHINE_ID: 007
MANUFACTURER: RK3568
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
GROW_ALIGN: 0
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00020000@0x00048000(recovery),0x00010000@0x00068000(backup),0x00c00000@0x00078000(rootfs),-@0x00c78000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。单位是块，每块 512 字节。