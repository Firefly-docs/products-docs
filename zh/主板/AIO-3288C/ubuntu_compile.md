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

* 下载根文件系统：[Ubuntu 根文件系统(32位)](http://www.t-firefly.com/doc/download/51.html#other_193)，放到 SDK 路径下

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
```