# 高通 Linux 1.7 编译

高通 Linux（QLI）是一款专为高通芯片设计和优化的嵌入式 Linux 发行版。即高通官方 BSP/SDK。  
它提供了一套经过筛选和验证的软件栈，涵盖引导加载程序、内核以及用户空间，使开发者能为高通硬件构建程序。  
QLI 是基于 yocto 的，自定义 QLI 需要对 yocto 有足够了解。

## 环境准备

需要 Ubuntu22 X86_64 Host 环境，强烈建议 docker  
请自行上网搜索 docker 的安装方法和配置网络代理方法。

安装好 docker 之后，执行：
```bash
# 拉取 ubuntu22 基本镜像
docker pull ubuntu:22.04

# 创建容器
# -v src:des 表示将主机 src 目录挂载到容器的 des 目录，方便容器内访问项目文件，自行设置
# --name 表示容器名称，自行起一个
# --hostname 表示容器的 hostname，自行起一个
docker run --privileged -v /home/xxx/project:/home/xxx/project \
        --name container_name --hostname host_name -it ubuntu:22.04 /bin/bash
```

容器启动后，在容器内安装环境：
```bash
sudo apt update

sudo apt install build-essential crossbuild-essential-arm64 \
        bash-completion vim sudo locales time rsync bc python2 python3

sudo apt install git lz4 zstd diffstat qemu-user-static \
        xmlstarlet texinfo chrpath libarchive-dev ssh libselinux1-dev \
        fakechroot libiberty-dev liblz4-tool qemu-user-static qemu-user \
        gawk libwayland-dev fakeroot libpam0g-dev binutils-dev util-linux \
        uuid-dev m4 zip unzip cpio wget python3-pip lib32stdc++6 \
        binfmt-support binutils kconfig-frontends python3-rosdep2 libgtest-dev \
        python3-lxml libncurses5 pkg-config flex bison libxml2-utils \
        libxml-simple-perl whiptail language-pack-en file zlib1g-dev

sudo apt install gnupg curl libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev \
        libx11-dev lib32z1-dev libgl1-mesa-dev xsltproc fontconfig

sudo rm -rf /lib/ld-linux-aarch64.so.1
sudo ln -sf /usr/aarch64-linux-gnu/lib/ld-linux-aarch64.so.1 /lib/ld-linux-aarch64.so.1
sudo ln -sf /bin/bash /bin/sh
sudo ln -sf /usr/bin/python3 /usr/bin/python

pip3 install dtschema==2021.10
pip3 install jsonschema==4.0.0
pip3 install ruamel.yaml==0.17.17
pip3 install jinja2
pip3 install click
pip3 install lark
```

配置 git
```bash
git config --global user.email <Your email ID>
git config --global user.name <"Your Name">
git config --global http.postBuffer 1048576000
git config --global http.maxRequestBuffer 1048576000
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
git config --global http.followRedirects true
```

## 下载

从容器中退出，回到宿主机。前往 [下载页面](https://community.t-firefly.com/doc/download/432) 下载 SDK。

将下载好的压缩包存放到容器能访问到的地方（前文创建容器时 -v 的位置），检查分卷压缩的 md5 值:
```bash
md5sum ./qcom-linux-*split*
```

将得到值与 SDK 附带的 md5sum.txt 进行比较，确认无误后解压并检出代码:
```bash
cat ./qcom-linux-1.7.tgz.split* | tar zx
cd qcom-linux-1.7
git checkout -f HEAD
```

## 编译

回到容器，移动到 SDK 根目录。里面存在现成的编译脚本。

> ffbuild.sh : 主编译脚本。自动编译。  
> bbenv.sh : yocto 环境初始化脚本，用于启动环境进行手动编译。

### 完整编译

```bash
# 先执行 lunch 选择编译目标
./ffbuild.sh lunch

# 然后进行编译
./ffbuild.sh image
```

> 注意：如果因为网络问题导致 tensorflow-lite 一直编译失败，可以使用 remove_tensorflow-lite.patch 补丁去除该功能再尝试编译。

输出目录: `build-qcom-wayland/tmp-glibc/deploy/images/<TARGET>/qcom-multimedia-image` 其中 TARGET 表示你选择的编译目标型号，后文同理。使用该目录即可进行烧录。

> 注意：默认生成的是 ufs 固件。

对于 QCS6490 产品，需要做如下操作将它改为 emmc 版本：

```bash
# 进入到刷机包里面
cd build-qcom-wayland/tmp-glibc/deploy/images/<TARGET>/qcom-multimedia-image

# 删除原本 ufs 的 xml
rm rawprogram*.xml
rm patch*.xml

# 复制 emmc 的 xml 和 bin
cp partition_emmc/rawprogram[0-1].xml ./
cp partition_emmc/patch[0-1].xml ./
cp partition_emmc/*.bin ./
```

### 自定义内核

使用 yocto 的 devtool 工具进行内核的自定义:
```bash
# 启动 yocto 环境
source bbenv.sh

# 检查工作区状态，第一次执行应该会显示当前无工作区
devtool status

# 创建内核工作区，linux-qcom-custom 为内核配方的名称
devtool modify linux-qcom-custom

# 等待上一步完成后，内核代码会位于工作区中，可以前往修改
cd workspace/sources/linux-qcom-custom

# 修改完毕后回到编译目录，重新检查工作区状态
cd /path/to/sdk/build-qcom-wayland
devtool status

# 确认 devtool status 能正确识别到 linux-qcom-custom 工作区后，可进行编译
bitbake linux-qcom-custom qcom-multimedia-image
```

### 只编译设备树

后续开发只修改设备树可以:
```bash
# 启动 yocto 环境
source bbenv.sh

# 用 bitbake 编译 dtb
bitbake dtb-qcom-image
```

生成的文件是：`build-qcom-wayland/tmp-glibc/deploy/images/<TARGET>/dtb-qcom-image-*.vfat` 重命名为 `dtb.bin` 即可。

## 制作 Ubuntu 固件

Ubuntu 系统不支持直接编译，请前往[下载中心](https://community.t-firefly.com/doc/download/432)下载发布的现成固件。如有自定义需求，需要将 QLI 编译生成的 kernel、modules、firmware 等部分移植到 Ubuntu 系统中。

### 准备工作

先将 QLI 的一些编译产物复制到方便的地方，这里以 ~/work 为例:
```bash
# dtb
cp build-qcom-wayland/tmp-glibc/deploy/images/<TARGET>/dtb-qcom-image-<TARGET>.rootfs-*.vfat ~/work/

# kernel
cp -rfd build-qcom-wayland/tmp-glibc/work/<TARGET>-qcom-linux/linux-qcom-custom/6.6/image/boot ~/work/

# modules
cp -rfd build-qcom-wayland/tmp-glibc/work/<TARGET>-qcom-linux/qcom-multimedia-image/1.0/rootfs/usr/lib/modules ~/work/
cp -rfd build-qcom-wayland/tmp-glibc/work/<TARGET>-qcom-linux/qcom-multimedia-image/1.0/rootfs/usr/lib/modules-load.d ~/work/

# firmware
cp -rfd build-qcom-wayland/tmp-glibc/work/<TARGET>-qcom-linux/qcom-multimedia-image/1.0/rootfs/usr/lib/firmware ~/work/
```

将下载好的 Ubuntu 固件解压到方便的地方，这里以 ~/ubun_firmware 为例。

准备脚本 setup.sh 放入 ~/ubun_firmware/ 下，脚本内容为:
```bash
#!/bin/bash

usage() {
        echo "Usage:"
        echo "$0 xxxx.img ----- mount and prepare chroot for this image"
        echo "$0 umount   ----- umount image"
}

if [[ $# != 1 ]]; then
        usage
        exit 1
fi

img=$1
mkdir mnt &> /dev/null

do_losetup() {
        dev=$(sudo losetup --sector-size=4096 --find --show --partscan $img)
        if [[ -n $dev ]]; then
                echo "Modify image: $img"
                echo "======================="
                return 0
        else
                return 1
        fi
}

do_mount() {
        if [[ -b "${dev}p3" ]]; then
                sudo mount "${dev}p3" mnt || return 1
                sudo mount "${dev}p1" mnt/boot/efi || return 1
        else
                sudo mount "${dev}" mnt || return 1
        fi
        return 0
}

prepare_chroot() {
        sudo mount --bind /dev mnt/dev || return 1
        sudo mount --bind /dev/pts mnt/dev/pts || return 1
        sudo mount -t proc proc mnt/proc || return 1
        sudo mount -t sysfs sysfs mnt/sys || return 1
        echo "chroot ready !"
        echo "Run \"sudo chroot mnt /bin/bash\" to start."
        return 0
}

do_umount() {
        sudo umount -R mnt && sudo losetup -D
}

case $1 in
        *.img) do_losetup && do_mount && prepare_chroot ;;
        umount) do_umount ;;
        *) usage ;;
esac
```

最终效果如下:
```
~/work/
├── boot
├── dtb.bin
├── firmware
├── modules
└── modules-load.d

~/ubun_firmware/
├── aop.mbn
├── cpucp.elf
......
├── prog_firehose_ddr.elf
├── prog_firehose_lite.elf
├── rawprogram0.xml
......
├── setup.sh
......
```

> **然后将 ~/work 和 ~/ubun_firemware 全部复制到设备上，下面的操作需要在 arm64 设备上进行以避免架构不同带来的错误。**

### 开始移植

#### 1. 复制设备树
```bash
cp ~/work/dtb.bin ~/ubun_firmware/
```

#### 2. 挂载文件系统

使用 setup.sh 脚本进行挂载:

```bash
cd ~/ubun_firmware
./setup.sh system.img
```

顺利的话，此时文件系统已经挂载在当前的 mnt 目录下。

#### 3. 复制文件

**复制文件必须使用 root 权限！**

```bash
# 复制 kernel 相关
sudo cp ~/work/boot/System.map* ~/ubun_firmware/mnt/boot/
sudo cp ~/work/boot/config-* ~/ubun_firmware/mnt/boot/
sudo cp ~/work/boot/Image-* ~/ubun_firmware/mnt/boot/vmlinuz-*

# 删除旧的 modules 和 firmware
sudo rm -rf ~/ubun_firmware/mnt/usr/lib/modules
sudo rm -rf ~/ubun_firmware/mnt/usr/lib/modules-load.d
sudo rm -rf ~/ubun_firmware/usr/lib/firmware

# 复制新的 modules 和 firmware
sudo cp -rfd ~/work/modules ~/ubun_firmware/mnt/usr/lib/
sudo cp -rfd ~/work/modules-load.d ~/ubun_firmware/mnt/usr/lib/
sudo cp -rfd ~/work/firmware ~/ubun_firmware/mnt/usr/lib/

# 确保文件改动生效
sync
```

#### 4. chroot 自定义(可选)

如果还有额外定制需求，可以 chroot 到文件系统中进行操作。
```bash
cd ~/ubun_firmware/

# 备份并复制 resolv.conf
sudo mv mnt/etc/resolv.conf mnt/etc/resolv.conf.bk
sudo cp /etc/resolv.conf mnt/etc/

# chroot 进入文件系统
sudo chroot mnt /bin/bash
```

此时可以进行 apt 安装软件或其他操作。定制结束后:
```bash
# 还原 resolv.conf
rm /etc/resolv.conf
mv /etc/resolv.conf.bk /etc/resolv.conf

# 退出 chroot
exit

# 确保文件改动生效
sync
```

#### 5. 取消挂载

使用 setup.sh 脚本取消挂载:
```bash
./setup.sh umount
```

到此移植结束，~/ubun_firmware 即成为新的 Ubuntu 固件，可用于烧录。