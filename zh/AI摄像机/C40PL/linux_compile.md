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

<font color=red>**非常重要：**</font>

开发前请根据 WIKI 把 SDK 更新到最新，尤其是使用 CS-R1 集群服务器用户（使用 CS-R1 用户如果代码版本太旧软件电平配置和硬件不匹配可能导致<font color=red>**硬件损坏**</font>）。

`CORE-1126-JD4/CORE-1109-JD4` 系列请下载 `rv1126_rv1109_linux_release` Linux SDK 软件包。

`CORE-1126-JD4/CORE-1109-JD4` 系列如果要进行 uvc 应用的开发，请下载 `rv1126_rv1109_linux_ai_camera_release` Linux SDK 软件包。

### SDK 开发包说明

`rv1126_rv1109_linux_release` SDK 说明：
1. `CORE-1126-JD4/CORE-1109-JD4` 系列产品可开发 IPC 固件（IPC 即网络摄像头）
2. `CAM-C1126S2U/CAM-C1109S2U` 系列产品可开发 Facial_gate 固件（Facial_gate 即人脸识别闸机）

`rv1126_rv1109_linux_ai_camera_release` SDK 说明：
1. `CORE-1126-JD4/CORE-1109-JD4` 系列产品可开发 AI_UVC 固件（UVC 即免驱摄像头）
2. `CAM-C1126S2U/CAM-C1109S2U` 系列产品可开发 AI_UVC 固件（UVC 即免驱摄像头）

完整 SDK 和 BSP SDK 说明：

`rv1126_rv1109_linux_release` 和 `rv1126_rv1109_linux_ai_camera_release` 都是完整 SDK 。

`rv1126_rv1109_linux_bsp_release` 和 `rv1126_rv1109_linux_ai_camera_bsp_release` 都是 BSP SDK 。

完整 SDK 和 BSP SDK 的区别：
1. 完整 SDK 可以编译出 buildroot 系统。文件体积比较大。里面包含 rockchip 提供的硬件加速文件目录。第一次使用建议拉取完整 SDK 。
2. BSP SDK 无法编译出 buildroot 系统。但是文件体积比较小。提供给只需要在公版固件的基础上，更新 u-boot 和 kernel 等分区固件的客户。

完整 SDK 和 BSP SDK 的相同点：

1. 都可以编译打包 Debian10 固件。

### 安装工具
获取 SDK 需要先安装：
```
sudo apt update
sudo apt install -y repo git python
```
### 初始化仓库

* 方法一（推荐国内用户使用）

**SDK 源码存放于 gitlab，国内用户可能下载完整的 SDK 仓库速度比较慢，所以我们提供了一个 SDK 基础包([Linux SDK](https://www.t-firefly.com/doc/download/101.html))，国内用户只需要在此基础包上同步 gitlab 上的代码就可以了**

注：该方法只能获取到完整 SDK 。BSP SDK 需要使用方法二获取。

* `rv1126_rv1109_linux_release` SDK 下载

下载完成后先验证一下 MD5 码：

```bash
md5sum rv1126_rv1109_linux_release_20211022.tgz
596c6bc6bb3095aea97d54c9df4cf333
```

确认无误后，就可以解压：
```bash
# 解压
mkdir -p ~/proj/rv1126_sdk
cd ~/proj/rv1126_sdk
mv /path/to/rv1126_rv1109_linux_release_20211022.tgz ./
tar xvf rv1126_rv1109_linux_release_20211022.tgz

#本压缩包内包含一个.repo目录，解压之后，在当前目录下执行以下操作
.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all

#后续可以使用以下命令更新SDK
.repo/repo/repo sync -c --no-tags

#因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行
```

* `rv1126_rv1109_linux_ai_camera_release` SDK 下载

​	下载完成后先验证一下 MD5 码：

```bash
md5sum rv1126_rv1109_linux_ai_camera_release_20210306.tgz
26fcb17ddd82a43d396eb3eac4b7479f
```

确认无误后，就可以解压：
```bash
# 解压
mkdir -p ~/proj/rv1126_sdk_ai
cd ~/proj/rv1126_sdk_ai
mv /path/to/rv1126_rv1109_linux_ai_camera_release_20210306.tgz ./
tar xvf rv1126_rv1109_linux_ai_camera_release_20210306.tgz

#本压缩包内包含一个.repo目录，解压之后，在当前目录下执行以下操作
.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all

#后续可以使用以下命令更新SDK
.repo/repo/repo sync -c --no-tags

#因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行
```


* 方法二

通过 repo 拉取代码，此方法对网络要求较高，有条件可以使用。可选择获取完整 SDK 或者 BSP SDK 。

* `rv1126_rv1109_linux_release` 完整 SDK 拉取方法
```bash
mkdir ~/proj/rv1126_sdk/
cd ~/proj/rv1126_sdk/

repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_release.xml

repo sync -c
#需反复确认代码下载是否成功
```

* `rv1126_rv1109_linux_bsp_release`  BSP SDK 拉取方法
```bash
mkdir ~/proj/rv1126_sdk/
cd ~/proj/rv1126_sdk/

repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_bsp_release.xml

repo sync -c
#需反复确认代码下载是否成功
```



* `rv1126_rv1109_linux_ai_camera_release` 完整 SDK 拉取方法

```bash
mkdir ~/proj/rv1126_sdk_ai/
cd ~/proj/rv1126_sdk_ai/

repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_ai_camera_release.xml

repo sync -c
#需反复确认代码下载是否成功
```



* `rv1126_rv1109_linux_ai_camera_bsp_release`  BSP SDK 拉取方法

```bash
mkdir ~/proj/rv1126_sdk_ai/
cd ~/proj/rv1126_sdk_ai/

repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_ai_camera_bsp_release.xml

repo sync -c
#需反复确认代码下载是否成功
```



注：默认 repo 工具是使用 python2 来拉取代码。如果 Linux PC 系统比较新，可能默认的是 python3 版本。如果代码无法拉取，请手动设置一下 Linux PC 系统的 python 版本为 python2。并且使用当前目录下的 `.repo/repo/repo` 工具进行代码拉取。查看 Linux PC 系统默认的 python 版本：

```shell
$ ls -l /usr/bin/python
lrwxrwxrwx 1 root root 9 4月  16  2018 /usr/bin/python -> python2.7*
```



### 同步代码

执行如下命令同步代码：

* 进入 rv1126_rv1109_linux_release SDK 根目录

```bash
cd ~/proj/rv1126_sdk
```

* 进入 rv1126_rv1109_linux_ai_camera_release SDK 根目录

```bash
cd ~/proj/rv1126_sdk_ai
```

* 同步命令

```bash
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

后续可以使用以下命令更新 SDK：

```bash
.repo/repo/repo sync -c --no-tags
```

因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行。
## Linux SDK 配置介绍

### 目录介绍

```bash
$ tree -L 1
.
├── app
├── buildroot                                               # Buildroot 根文件系统编译目录
├── build.sh -> device/rockchip/common/build.sh             # 编译脚本
├── device                                                  # 编译相关配置文件
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
```

### 配置文件介绍

在 `device/rockchip/rv1126_rv1109/` 目录下，有不同板型的配置文件(xxxx.mk)，用于管理 SDK 每个环节的编译配置，相关配置介绍：

```bash
# Target arch
export RK_ARCH=arm # 32 位 ARM 架构
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=xxxx_defconfig # u-boot 配置文件
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=xxxx_defconfig # kernel 配置文件
# Kernel defconfig fragment
export RK_KERNEL_DEFCONFIG_FRAGMENT=xxxx.config # kernel 配置文件（fragment）
# Kernel dts
export RK_KERNEL_DTS=xxxx.dts # dts 文件
# parameter for GPT table
export RK_PARAMETER=parameter-xxxx.txt # 分区表
# rootfs image path
export RK_ROOTFS_IMG=ubuntu_rootfs/rootfs.img # 根文件系统路径
```

### 分区说明

#### parameter 分区表

parameter.txt 文件中包含了固件的分区信息，以 parameter-ubuntu-fit.txt 为例：

路径：`device/rockchip/rv1126_rv1109/parameter-xxxxxx-fit.txt`

```bash
FIRMWARE_VER: 8.1
MACHINE_MODEL: RV1126
MACHINE_ID: 007
MANUFACTURER: RV1126
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 0xffffffff
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00010000@0x00008000(boot),0x00010000@0x00018000(recovery),0x00010000@0x00028000(backup),0x00300000@0x00038000(rootfs),0x00060000@0x00338000(oem),0x00400000@0x00398000(userdata),-@0x00798000(media:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

CMDLINE 属性是我们关注的地方，以 uboot 为例， 0x00002000@0x00004000(uboot) 中 0x00004000 为uboot 分区的起始位置，0x00002000 为分区的大小，以此类推。

#### package-file

package-file 文件用于打包固件时确定需要的分区镜像和镜像路径，同时它需要与 parameter.txt 文件保持一致。

路径：`tools/linux/Linux_Pack_Firmware/rockdev/rv1126_rv1109-package-file`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
#
# if uboot.img is fit, uboot.img had include uboot and trust,
# so ignore trust.img
# file Image/uboot.img
# Image/uboot.img: Device Tree Blob version 17
#
# trust         Image/trust.img
#
uboot           Image/uboot.img
misc            Image/misc.img
#resource       Image/resource.img
#kernel         Image/kernel.img
boot            Image/boot.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
oem             Image/oem.img
userdata        Image/userdata.img
# 要写入backup分区的文件就是自身（update.img）
# SELF 是关键字，表示升级文件（update.img）自身
# 在生成升级文件时，不加入SELF文件的内容，但在头部信息中有记录
# 在解包升级文件时，不解包SELF文件的内容。
backup          RESERVED
#update-script  update-script
#recover-script recover-script
```


## 编译 Debian 固件

本章介绍 Debian 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

<font color=red>**本教程的编译部分适用于 v2.2.5f 以上 SDK 版本**</font>

```
$ realpath .repo/manifest.xml
/home/lvsx/project/rv1126_rv1109/.repo/manifests/rv1126_rv1109/rv1126_rv1109_linux_release_20240117_v2.2.5f.xml
```
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

#### 编译前配置

在 `device/rockchip/rv1126_rv1109/` 目录下，有不同板型的配置文件，选择配置文件：

#### 编译 Debian 固件
* 编译 V2 mipi 屏幕前配置（旧版本屏幕，该屏幕已停产）
```
./build.sh aio-rv1126-jd4-BE-45-debian.mk
```

* 编译 V3 mipi 屏幕前配置

```bash
./build.sh aio-rv1126-jd4-mipi101-BSD1218-A101KL68-debian.mk
```

#### 编译

##### 全自动编译

* 下载根文件系统：[Debian 根文件系统(32位)](https://www.t-firefly.com/doc/download/101.html#other_693)，放到 SDK 路径下

```bash
7z x debian10_rootfs_2022_12_22.7z
mkdir ubuntu_rootfs
mv rootfs.img ubuntu_rootfs/rootfs.img
```

* 开始编译
```bash
./build.sh
```

生成的完整固件会保存到 `rockdev/pack/` 目录。

##### 部分编译

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

* 下载根文件系统：[Debian 根文件系统(32位)](https://www.t-firefly.com/doc/download/161.html)，放到 SDK 路径下

```bash
7z x debian10_rootfs_2022_12_22.7z
mkdir ubuntu_rootfs
mv rootfs.img ubuntu_rootfs/rootfs.img
```

* 更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

* 打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```
## 编译 Buildroot 固件

本章介绍 Buildroot 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

<font color=red>**本教程的编译部分适用于 v2.2.5f 以上 SDK 版本**</font>

```
$ realpath .repo/manifest.xml
/home/lvsx/project/rv1126_rv1109/.repo/manifests/rv1126_rv1109/rv1126_rv1109_linux_release_20240117_v2.2.5f.xml
```
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

#### 编译前配置

在 `device/rockchip/rv1126_rv1109/` 目录下，有不同板型的配置文件。

`rv1126_rv1109_linux_release` SDK 和 `rv1126_rv1109_linux_bsp_release` SDK 选择配置文件：
```bash



# V2 MIPI 屏幕（老版本屏幕，该屏幕已停产）
./build.sh aio-rv1126-jd4-BE-45.mk 

# V3 MIPI 屏幕
./build.sh aio-rv1126-jd4-mipi101-BSD1218-A101KL68.mk


```

`rv1126_rv1109_linux_ai_camera_release` SDK 和 `rv1126_rv1109_linux_ai_camera_bsp_release` SDK 选择配置文件：


`CORE-1126-JD4/CORE-1109-JD4` 系列产品对 UVC 功能适配并不太好，请有能力的客户再选择这个配置文件来编译。

```bash
./build.sh aio-rv1126-uvcc.mk 
```

#### 编译

##### 全自动编译

全自动编译会执行上述编译、打包操作，生成 RK 固件。

```bash
./build.sh
```

打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

##### 部分编译

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

* 编译 buildroot

```bash
./build.sh rootfs
```

* 更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

* 打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```
## 编译 Thunder-boot 固件
目前 CORE-1126-JD4 最新的 SDK 已适配快速启动。编译快速启动固件选项：
```
# 编译 CORE-1126-JD4 V1.1 快速启动固件 适配的屏幕为 DM-M10R800 V2 屏幕模组
./build.sh device/rockchip/rv1126_rv1109/aio-rv1126-jd4-BE-45-tb-v11.mk
```
快速启动固件启动后，MIPI DM-M10R800 V2 屏幕模组可预览摄像头画面。若接入网络，还可预览摄像头推流 rtsp。
* Linux PC 预览命令：
```
vlc rtsp://<设备 ip 地址>
```
* Windows PC 则需要下载 vlc 应用，并且输入串流地址来预览。

![](/img/C40PL/windows_vlc.jpg)
注意：该快速启动固件`仅适用于 CORE-1126-JD4 V1.1 版本的核心板`。核心板版本详见硬件上的丝印。