# 编译 Android 7.1

## 准备

### 硬件配置

编译 Android 7.1 开发环境硬件配置建议：

- 64 位 CPU
- 16GB  内存 + 交换内存
- 30GB  空闲空间用来编译， 源码树另占 8GB

另外可参考 Google 官方文档硬件和软件配置：

- [https://source.android.com/setup/build/requirements](https://source.android.com/setup/build/requirements)
- [https://source.android.com/setup/initializing](https://source.android.com/setup/initializing)

### 软件配置

**安装 JDK 8**

``` shell
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```

**安装环境包**

``` shell
sudo apt-get install git-core gnupg flex bison gperf libsdl1.2-dev \
  libesd0-dev libwxgtk2.8-dev squashfs-tools build-essential zip curl \
  libncurses5-dev zlib1g-dev pngcrush schedtool libxml2 libxml2-utils \
  xsltproc lzop libc6-dev schedtool g++-multilib lib32z1-dev lib32ncurses5-dev \
  lib32readline-gplv2-dev gcc-multilib libswitch-perl

sudo apt-get install gcc-arm-linux-gnueabihf \
  libssl1.0.0 libssl-dev \
  p7zip-full
```

## 下载 Android SDK

由于 SDK 较大，请选择以下链接下载SDK `firefly_rk3328_android7.1_git_20211216`：

* [下载链接](https://community.t-firefly.com/doc/download/34.html)

下载完成后，在解压前先校验下 MD5 码：

``` shell
$md5sum ~/firefly_rk3328_android7.1_git_20211216.7z.001
$md5sum ~/firefly_rk3328_android7.1_git_20211216.7z.002
$md5sum ~/firefly_rk3328_android7.1_git_20211216.7z.003

f4d62bfbb401f596328435e61cb1c68d  firefly_rk3328_android7.1_git_20211216.7z.001
0bb35911c8740119f789926d6d21c550  firefly_rk3328_android7.1_git_20211216.7z.002
19eebf52e1e033cea682cdd306d8f75a  firefly_rk3328_android7.1_git_20211216.7z.003
```

然后解压：

``` shell
mkdir -p  ~/proj/firefly_rk3328_android7.1
cd ~/proj/firefly_rk3328_android7.1
7z x ~/firefly_rk3328_android7.1_git_20211216.7z.001 -r -o.
git reset --hard
```

从 gitlab 处同步源码：

``` shell
git pull gitlab roc-rk3328-cc:roc-rk3328-cc

git checkout roc-rk3328-cc
```

## 使用 Firefly 脚本编译

**编译内核**

``` shell
./FFTools/make.sh -k -j8
```

**编译 U-Boot**

``` shell
./FFTools/make.sh -u -j8
```

**编译 Android**

``` shell
./FFTools/make.sh -a -j8
```

**编译全部**

如下指令会编译出内核, U-Boot 以及 Android：

``` shell
./FFTools/make.sh -j8
```

## 不使用脚本编译

编译之前请先执行如下命令配置好环境变量：

``` shell
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

**编译内核**

``` shell
make ARCH=arm64 firefly_defconfig
make -j8 ARCH=arm64 rk3328-roc-cc.img
```

**编译 U-Boot**

``` shell
make rk3328_box_defconfig
make ARCHV=aarch64 -j8
```

**编译 Android**

``` shell
source build/envsetup.sh
lunch roc_rk3328_cc_box-userdebug
make installclean
make -j8
./mkimage.sh
```

## 打包 RK 固件

**在 Linux 下打包固件**

编译完成后使用 Firefly 官方脚本即可打包所有的分区映像成 RK固件或 原始固件：

``` shell
rk固件:
./FFTools/mkupdate/mkupdate.sh update


原始固件:
./FFTools/mkupdate/sd_mkupdate.sh update
```

最终生成的文件是 `rockdev/Image-rk3328_firefly_box/update.img`.

RK固件需使用 `SD_Firmware_Tool` 工具，功能模式选择`SD启动`，来制作启动卡，
而原始固件可使用 `SDCard-installer` 制作启动卡

**在 Windows 下打包固件**

在 Windows 下打包 RK 固件 `update.img` 也是很简单的：

1. 拷贝所有在 `rockdev/Image-rk3328_firefly_box/` 目录下编译好的文件到 AndroidTool 的 `rockdev\Image` 目录下。
2. 运行在 AndroidTool 的 `rockdev` 目录下的 `mkupdate.bat` 文件。
3. 在 `rockdev\Image` 目录将会生成 `update.img`。

## 分区映像

`update.img` 是发布给最终用户的固件，方便升级开发板。而在实际开发中，更多的时候是修改并烧写单个分区映像文件，这样做大大节省开发时间。

下表总结了在各个编译阶段所生成的分区映像文件：

```text
|------------------|---------------------|-----------|
| Stage            | Product             | Partition |
|------------------|---------------------|-----------|
| Compiling Kernel | kernel/kernel.img   | kernel    |
|                  | kernel/resource.img | resource  |
|------------------|---------------------|-----------|
| Compiling U-Boot | u-boot/uboot.img    | uboot     |
|------------------|---------------------|-----------|
| ./mkimage.sh     | boot.img            | boot      |
|                  | system.img          | system    |
|------------------|---------------------|-----------|
```

注意，执行 `./mkimage.sh` 后， `boot.img` 和 `system.img` 将会被重新编译并打包到目录 `out/target/product/rk3328_firefly_box/` 下，所有生成的映像文件将会拷贝到目录 `rockdev/Image-rk3328_firefly_box/` 下。

如下是映像文件列表：

- `boot.img`： Android 的 initramfs 映像，包含Android根目录的基础文件系统，它负责初始化和加载系统分区。
- `system.img`： ext4 文件系统格式的 Android 文件系统分区映像。
- `kernel.img`： 内核映像。
- `resource.img`： Resource 映像， 包含启动图片和内核设备树。
- `misc.img`： misc 分区映像， 负责启动模式的切换和急救模式参数的传递。
- `recovery.img`： Recovery 模式映像。
- `rk3328_loader_v1.08.244.bin`： Loader 文件。
- `uboot.img`： U-Boot 映像文件。
- `trust.img`： Arm trusted file (ATF) 映像文件。
- `parameter.txt`： 分区布局和内核命令行。

[《上手指南》]: getting_started.md
[《常见问题解答》]: faqs.md
[《串口调试》]: debug.md
[《编译 Linux 根文件系统》]: linux_build_ubuntu_rootfs.md
[联系方式]: resource.md#社区
[原始固件]: getting_started.md#raw-firmware-format
[RK 固件]: getting_started.md#rk-firmware-format
[分区映像]: getting_started.md#partition-image
[SDCard Installer]: flash_sd.md#sdcard-installer
[Etcher]: flash_sd.md#etcher
[dd]: flash_sd.md#dd
[SD Firmware Tool]: flash_sd.md#sd-firmware-tool
[AndroidTool]: flash_emmc.md#androidtool
[upgrade_tool]: flash_emmc.md#upgrade-tool
[rkdeveloptool]: flash_emmc.md#rkdeveloptool
[Rockusb 模式]: flash_emmc.md#rockusb-mode
[Maskrom 模式]: flash_emmc.md#maskrom-mode
[Rockusb 驱动]: flash_emmc.md#rockusb-driver
[ROC-RK3328-CC]: http://www.t-firefly.com/product/rocrk3328cc.html "ROC-RK3328-CC 官网"
[下载页面]: http://www.t-firefly.com/doc/download/page/id/34.html
[论坛]: http://bbs.t-firefly.com
[脸书]: https://www.facebook.com/TeeFirefly
[Google+]: https://plus.google.com/u/0/communities/115232561394327947761
[油管]: https://www.youtube.com/channel/UCk7odZvUrTG0on8HXnBT7gA
[推特]: https://twitter.com/TeeFirefly
[在线商城]: http://store.t-firefly.com
[USB 转串口适配器]: https://store.t-firefly.com/goods.php?id=24
[5V2A 电源适配器]: https://store.t-firefly.com/goods.php?id=69
[eMMC 闪存]: https://store.t-firefly.com/goods.php?id=71
[《存储映射》]: http://opensource.rock-chips.com/wiki_Partitions#Default_storage_map