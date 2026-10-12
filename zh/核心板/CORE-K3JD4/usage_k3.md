## 软件升级

**AIBOX-K3 暂时不支持 `apt upgrade` 进行系统软件全量升级**。普通的软件包可以通过 apt update 和 apt install 进行安装更新。执行 apt update 和 apt install 命令是没有问题的。若想要升级 esos / uboot / kernel 等重要软件包，则需要下载 Firefly 官方提供的 SDK 软件包并通过编译烧录的方式进行更新。

## 文档

下面链接文档包含《产品介绍》、《数据手册》和《用户手册》。

* [芯片产品文档](https://spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs)

## SDK 获取

Buildroot代码托管在 Github 上，包含若干个仓库，使用 repo 管理，下载前需：

1. 安装 repo
   * 如果可访问 Google，请参考 [这篇文档](https://gerrit.googlesource.com/git-repo/+/refs/heads/main/README.md#install) 安装
   * 也可参考 [Git Repo 镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/git-repo/) 安装。

要求 repo 版本 >= 2.41，否则下载过程会异常。

```bash
$ repo --version
repo version v2.48
       (from https://mirrors.tuna.tsinghua.edu.cn/git/git-repo)
       (tracking refs/heads/stable)
       (Mon, 7 Oct 2024 18:44:19 +0000)
repo launcher version 2.50
       (from /usr/bin/repo)
       (currently at 2.48)
repo User-Agent git-repo/2.48 (Linux) git/2.25.1 Python/3.8.10
git 2.25.1
git User-Agent git/2.25.1 (Linux) git-repo/2.48
Python 3.8.10 (default, Nov  7 2024, 13:10:47)
[GCC 9.4.0]
OS Linux 5.4.0-196-generic (#216-Ubuntu SMP Thu Aug 29 13:26:53 UTC 2024)
CPU x86_64 (x86_64)
Bug reports: https://issues.gerritcodereview.com/issues/new?component=1370071
```

### 从 Github 下载

```bash
mkdir ~/k3-buildroot-sdk
cd ~/k3-buildroot-sdk

repo init -u https://github.com/Firefly-spacemit-linux/manifests.git -b spacemit -m k3_linux6.18_release.xml

# Synchronize SDK code
repo sync -c --no-tags
```

## 编译 Linux 固件

注意：

1. 默认在容器中构建。如需在宿主机上构建，请配置环境变量 export DIRECT_BUILD=1，切换容器中构建和宿主机构建时需要清理output目录。

2. 新增一组命令。与旧命令不兼容，如需使用新命令，需删除项目根目录下旧命令生成的env.mk文件，之后可用make help查看新命令列表，例如可以make k3-build 直接编译指定方案：

   ```
   $ cd ~/k3-buildroot-sdk-1.0
   $ rm -rf env.mk
   $ make help
   Buildroot Build System - Help
   
   Available solutions:
     firefly-k3 k3 k3_ci k3_fpga k3_plt
   
   Development Commands:
     make vars                          # Show project information
     make <solution>-supported          # Check if solution is supported
     make <solution>-config             # Apply solution defconfig
     make <solution>-menuconfig         # Configure buildroot for solution
     make <solution>-linux-menuconfig   # Configure Linux kernel for solution
     make <solution>-uboot-menuconfig   # Configure U-Boot for solution
     make <solution>-busybox-menuconfig # Configure BusyBox for solution
     make <solution>-build              # Build specified solution
     make <solution>-pkg PKG=<package>  # Build specified package for solution
     make <solution>-shell              # Enter build container for solution
     make <solution>-source             # Download all source packages for solution
     make <solution>-clean              # Clean solution build artifacts
     make <solution>-cleanbuild         # Clean and rebuild solution
     make build-docker-image            # Build Docker image
     make update-docker-image           # Update Docker image
   
   Quick Start: Run 'make <solution>-build' to get started, e.g.:
     make k3-build
   ```

### 编译 Buildroot 固件

#### 编译前配置

```
$ cd ~/k3-buildroot-sdk-1.0
$ make envconfig
Available configs in /home2/lvsx/ext/project/k3/buildroot-ext/configs:
  1. spacemit_firefly-k3_defconfig
  2. spacemit_k3_ci_defconfig
  3. spacemit_k3_defconfig
  4. spacemit_k3_fpga_defconfig
  5. spacemit_k3_plt_defconfig

Your choice (1-5):
```

这里输入 1 选择 `spacemit_firefly-k3_defconfig`。



编译过程可能需要下载一些第三方软件包，具体耗时和网络环境相关。如果提前下载 buildroot 依赖的第三方软件包，推荐硬件配置编译耗时约为1小时。

编译产物位于`/output/k3/images/`目录下，编译打包成功log如下：

```
Generating sdcard image...................................
INFO: cmd: "mkdir -p "/firefly-k3/build/genimage.tmp"" (stderr):
INFO: cmd: "rm -rf "/firefly-k3/build/genimage.tmp"/*" (stderr):
INFO: cmd: "mkdir -p "/firefly-k3/images"" (stderr):
INFO: hdimage(Buildroot-firefly-k3-20260508163419-sdcard.img): adding primary partition 'env' from 'env.bin' ...
INFO: hdimage(Buildroot-firefly-k3-20260508163419-sdcard.img): adding primary partition 'bootinfo' from 'factory/bootinfo_block.bin' ...
INFO: hdimage(Buildroot-firefly-k3-20260508163419-sdcard.img): adding primary partition 'fsbl' from 'factory/FSBL.bin' ...
INFO: hdimage(Buildroot-firefly-k3-20260508163419-sdcard.img): adding primary partition 'esos' from 'esos.itb' ...
INFO: hdimage(Buildroot-firefly-k3-20260508163419-sdcard.img): adding primary partition 'opensbi' from 'fw_dynamic.itb' ...
INFO: hdimage(Buildroot-firefly-k3-20260508163419-sdcard.img): adding primary partition 'uboot' from 'u-boot.itb' ...
INFO: hdimage(Buildroot-firefly-k3-20260508163419-sdcard.img): adding primary partition 'bootfs' (in MBR) from 'bootfs.img' ...
INFO: hdimage(Buildroot-firefly-k3-20260508163419-sdcard.img): adding primary partition 'rootfs' (in MBR) from 'rootfs.ext4' ...
INFO: hdimage(Buildroot-firefly-k3-20260508163419-sdcard.img): adding primary partition '[MBR]' ...
INFO: hdimage(Buildroot-firefly-k3-20260508163419-sdcard.img): adding primary partition '[GPT header]' ...
INFO: hdimage(Buildroot-firefly-k3-20260508163419-sdcard.img): adding primary partition '[GPT array]' ...
INFO: hdimage(Buildroot-firefly-k3-20260508163419-sdcard.img): adding primary partition '[GPT backup]' ...
INFO: hdimage(Buildroot-firefly-k3-20260508163419-sdcard.img): writing GPT
INFO: hdimage(Buildroot-firefly-k3-20260508163419-sdcard.img): writing protective MBR
INFO: hdimage(Buildroot-firefly-k3-20260508163419-sdcard.img): writing MBR
INFO: cmd: "rm -rf "/firefly-k3/build/genimage.tmp/"" (stderr):
Successfully generated at /firefly-k3/images/Buildroot-firefly-k3-20260508163419-sdcard.img
```



其中 `Buildroot-K3-xxx.zip`适用于[Titan Flasher](https://www.spacemit.com/community/document/info?lang=zh&nodepath=tools/user_guide/flasher_user_guide.md)，或者解压后用fastboot刷机；`Buildroot-K3-xxx-sdcard.img`为sdcard固件，解压后可以用dd命令或者[balenaEtcher](https://etcher.balena.io/)写入sdcard。

固件默认用户名：`root`，密码：`firefly`。



### 编译 Bianbu 固件

#### 准备工作

下载固件压缩包并解压获得 rootfs.ext4 根文件系统。

#### 查看 SDK 版本

```bash
$ realpath .repo/manifests/k3_linux6.18_release.xml
```

例如 SDK v1.0.0 版本输出信息

```bash
$ realpath .repo/manifests/k3_linux6.18_release.xml
K3_SDK/.repo/manifests/k3/k3_linux6.18_release_20260611_v1.0.0a.xml
```

例如 SDK v1.0.9 版本输出信息

```bash
$ realpath .repo/manifests/k3_linux6.18_release.xml
K3_SDK/.repo/manifests/k3/k3_linux6.18_release_20261010_v1.0.9a.xml
```

**注意：SDK V1.0.0 版本和 SDK 1.0.9 及以上版本进行打包固件的方式不一样。建议更新 SDK 到最新再根据教程进行操作。**



* 仅 SDK V1.0.0 版本固件打包教程

下载固件压缩包后将固件解压，并拷贝固件内的 rootfs.ext4 文件到 SDK/buildroot-ext/board/spacemit/k3/ 下

```bash
# 例如固件名称是 Firefly-K3_Bianbu-LXQt_v1.0.9a_261010.zip 。根据固件名称自行调整解压命令
unzip Firefly-K3_Bianbu-LXQt_v1.0.9a_261010.zip -d Firefly-K3_Bianbu-LXQt_v1.0.9a_261010

# 将固件解压后的 rootfs.ext4 拷贝到 sdk 指定目录。名称需指定为 rootfs.ext4
cp Firefly-K3_Bianbu-LXQt_v1.0.9a_261010/rootfs.ext4 /SDK/buildroot-ext/board/spacemit/k3/rootfs.ext4
```

* SDK V1.0.9 及以上版本固件打包教程

```bash
# 例如固件名称是 Firefly-K3_Bianbu-LXQt_v1.0.9a_261010.zip 并拷贝到 SDK 根目录。根据固件名称自行调整解压命令
unzip Firefly-K3_Bianbu-LXQt_v1.0.9a_261010.zip -d Firefly-K3_Bianbu-LXQt_v1.0.9a_261010

# 将固件解压后的 rootfs.ext4 拷贝到 sdk 指定目录
cd /path/to/sdk
mkdir -p prebuilt_rootfs
cp Firefly-K3_Bianbu-LXQt_v1.0.9a_261010/rootfs.ext4 /SDK/prebuilt_rootfs/Bianbu-Lxqt_rootfs.ext4
cd prebuilt_rootfs/
ln -sf Bianbu-Lxqt_rootfs.ext4 rootfs.ext4
```

说明：rootfs.ext4 需要链接到实际的文件系统文件。编译脚本会根据 rootfs.ext4 链接到不同的名称，从而生成对应的固件名称。


#### 编译前配置

```
$ cd ~/k3-buildroot-sdk-1.0
$ make envconfig
Available configs in /home2/lvsx/ext/project/k3/buildroot-ext/configs:
  1. spacemit_firefly-k3_defconfig
  2. spacemit_k3_ci_defconfig
  3. spacemit_k3_defconfig
  4. spacemit_k3_fpga_defconfig
  5. spacemit_k3_plt_defconfig

Your choice (1-5):
```

这里输入 1 选择 `spacemit_firefly-k3_defconfig`。



SDK V1.0.0 版本固件打包说明：编译过程会使用 /SDK/buildroot-ext/board/spacemit/k3/rootfs.ext4 文件系统来替换 buildroot 的文件系统。



## 配置

配置需要进入到 docker 编译环境中

```bash
make firefly-k3-shell
```



### buildroot

配置：

```bash
make menuconfig
```



保存配置，默认保存到`buildroot-ext/configs/spacemit_firefly-k3_defconfig`：

```bash
make savedefconfig
```



### Linux

配置：

```bash
make linux-menuconfig
```



保存配置，默认保存到`bsp-src/linux-6.18/arch/riscv/configs/k3_bianbu_defconfig`：

```bash
make linux-update-defconfig
```



### u-boot

配置：

```bash
make uboot-menuconfig
```



保存配置，默认保存到`bsp-src/uboot-2022.10/configs/k3_defconfig`：

```bash
make uboot-update-defconfig
```



### 再次完整编译



如果已经使用`make envconfig`完整编译，可以直接使用`make`再次完整编译。



### 编译指定包



buildroot支持编译指定package，可以`make help`查看指南。

常用命令：

- 删除`<pkg>`的编译目录：`make <pkg>-dirclean`
- 编译`<pkg>`：`make <pkg>`
- 重新编译`<pkg>`: `make <pkg>-rebuild`

#### u-boot

```bash
#编译
make uboot

#清理
make uboot-dirclean

#重新编译
make uboot-rebuild
```



#### Linux

内核默认配置为`k3_bianbu_defconfig`

```bash
#编译
make linux

#清理
make linux-dirclean

#重新编译
make linux-rebuild
```



#### opensbi

```bash
#编译
make opensbi

#清理
make opensbi-dirclean

#重新编译
make opensbi-rebuild
```



编译指定包之后，可以单独下载到设备验证，或者编进固件：

```bash
make
```