# 编译 Linux 固件

## 前提条件 [step]

编译 Linux 固件在 X86_64 架构的 Ubuntu 主机上进行，SDK 采用交叉编译，不能在开发板上编译。编译前确认以下条件：

* 操作系统：Ubuntu 20.04 或更高版本（真机或 Docker 容器）
* 磁盘空间：建议预留 100 GB 以上
* 内存：建议 16 GB 以上
* 使用普通用户执行编译命令，不要使用 root 用户

## 获取 SDK [step]

Firefly-R3572A 的 Linux SDK 请联系销售人员（sales@t-firefly.com）获取下载链接。

获取后将 SDK 压缩包解压到普通用户目录，解压后的根目录以下称为 SDK 根目录，本节命令均在 SDK 根目录执行。

<Callout title="注意" type="warn">

1. 不要在虚拟机共享文件夹中解压 SDK，符号链接可能创建失败。
2. 不要使用 root 用户解压或编译，否则可能引发文件权限错误。

</Callout>

## 安装编译依赖 [step]

在 SDK 根目录执行以下命令安装编译依赖：

```bash
sudo apt-get update
sudo apt-get install git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffutils \
binfmt-support qemu-user-static live-build bison flex fakeroot \
cmake gcc-multilib g++-multilib unzip device-tree-compiler \
python3-pip libncurses5-dev build-essential
```

## 选择板级配置 [step]

Firefly Linux SDK 通过 `./build.sh <defconfig>` 加载板级配置，格式为 `firefly_芯片_板型_系统_defconfig`。Firefly-R3572A 的板级配置如下：

| 系统 | 配置文件 |
| --- | --- |
| Debian | `firefly_rk3572_r3572a_debian_defconfig` |

在 SDK 根目录执行以下命令选择 Debian 编译配置：

```bash
./build.sh firefly_rk3572_r3572a_debian_defconfig
```

对应的内核设备树为 `rk3572-firefly-r3572a`，内核源码位于 `kernel-6.12/arch/arm64/boot/dts/rockchip/`。

<Callout title="注意" type="warn">

切换板型或系统后必须重新选择 defconfig，直接沿用旧配置会编译出错误的固件。

</Callout>

## 编译固件 [step]

编译分为**完整编译**（首次编译、或修改 rootfs / 整体配置后使用）与**部分编译**（只改动某个模块、快速验证时使用）两种方式，按需选择。两种方式都需先按上一节选好 defconfig。

### 完整编译 [step]

完整编译会依次构建 U-Boot、内核、Recovery、rootfs 并打包生成统一固件 `update.img`。在 SDK 根目录执行：

```bash
./build.sh firefly_rk3572_r3572a_debian_defconfig
./build.sh all
```

其中 `./build.sh all` 对应完整构建流程（内部调用 `mk-all.sh`）；若根文件系统为预编译镜像，需先按 rootfs 类型放置到 SDK 的 `prebuilt_rootfs/` 目录，具体以 SDK 文档为准。

完整编译耗时较长，首次编译建议预留充足时间。

### 部分编译 [step]

仅修改某个模块时，可单独编译对应部件，最后重新打包 `update.img`，无需每次全量编译：

| 编译目标 | 命令 |
| --- | --- |
| U-Boot | `./build.sh uboot` |
| 内核与设备树 | `./build.sh extboot` |
| Recovery | `./build.sh recovery` |
| 打包统一固件 | `./build.sh updateimg` |

例如仅重新编译内核并重新打包固件，在 SDK 根目录执行：

```bash
./build.sh firefly_rk3572_r3572a_debian_defconfig
./build.sh extboot
./build.sh updateimg
```

## 产物与验证 [step]

编译产物位于 SDK 根目录的 `output/firmware/` 与 `output/update/` 目录：内核与设备树编译生成 `extboot.img`（位于 `kernel-6.12/`），统一固件 `update.img` 及各分区镜像位于 `output/update/`。

执行以下命令确认产物生成：

```bash
ls output/update/
```

预期输出包含 `update.img`、`boot.img`、`uboot.img`、`parameter.txt` 等文件。烧录方法见 [使用 USB 线缆升级固件](../../getting_started/firmware_upgrade/upgrade_firmware.md)。