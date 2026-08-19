# 编译 Android 8.1

## 准备

### 硬件配置

编译 Android 8.1 开发环境硬件配置建议：

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

由于 SDK 较大，请在云盘下载 `RK3328_Android8.1_git_20190719.7z`：

* [下载链接](https://community.t-firefly.com/doc/download/62)

下载完成后，在解压前先校验下 MD5 码：

``` shell
$ md5sum /path/to/RK3328_Android8.1_git_20190719.7z.001
d2198123c17af2140a9336b2377e0e93  RK3328_Android8.1_git_20190719.7z.001

$ md5sum /path/to/RK3328_Android8.1_git_20190719.7z.002
9df675c96997b5e246549acf0789fa39  RK3328_Android8.1_git_20190719.7z.002

$ md5sum /path/to/RK3328_Android8.1_git_20190719.7z.003
56d54d78aa98a6737d171f631dc66960  RK3328_Android8.1_git_20190719.7z.003
```

然后解压：

``` shell
mkdir -p ~/proj/rk3328
cd ~/proj/rk3328

# 分卷解压出完整的 .git 目录
7z x RK3328_Android8.1.2_git_20190719.7z.001

# 重置工作目录，签出所有源码
git reset --hard
```

第一次及后续的 SDK 更新请按照以下说明进行正确操作：
``` shell
# 由于容量限制，完整 SDK 无法完全托管在 Gitlab 上，因此采用了折衷的方法，
# 以 git bundle 的方式存放增量的 SDK 更新。
# 首先更新并应用 bundle 仓库：
test -d .bundle || \
  git clone \
    https://gitlab.com/TeeFirefly/rk3328-android8.1-oreo-bundle.git \
    .bundle

.bundle/update

# 如果执行了上述命令后，提示 "[Info]Already up to date!"，那么就表示 SDK 没有更新；
# 反之，SDK 更新后的最新提交会保存到 FETCH_HEAD，这时可以查看本地和最新 SDK 的关系
# 再作处理：
git show-branch HEAD FETCH_HEAD

# 如果有本地提交，可以选择 rebase 或 merge 方式去合并 SDK 修改：
# rebase
git rebase FETCH_HEAD
# merge
git merge FETCH_HEAD
```

## 使用 Firefly 脚本编译

### 完全化编译

``` shell
./build.sh rk3328-core-jd4
```
以上命令执行完后，会编译 U-Boot、内核和 Android 上层，同时整理分区镜像并生成统一固件 `update.img`，放在 `rockdev/Image-rk3328_core_jd4/` 目录下。

### 模块化编译

**编译配置文件**

<font color=#ff0000 size=3>注意</font>：“编译配置文件”是前提，需完成该步骤才能往下执行。

``` shell
./FFTools/make.sh rk3328-core-jd4 
```

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

**编译全部分区**

``` shell
./FFTools/make.sh -j8
```
以上命令执行完后，会编译 UBoot、内核和 Android 上层，同时整理分区镜像到 `rockdev/Image-rk3328_core_jd4/` 目录下，但不会生成统一固件。


## 不使用脚本编译

编译之前请先执行如下命令配置好环境变量：

``` shell
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

**编译内核**

``` shell
make ARCH=arm64 rockchip_defconfig
make -j8 ARCH=arm64 rk3328-core-jd4.img
```

**编译 U-Boot**

``` shell
make rk3328_box_defconfig
make ARCHV=aarch64 -j8
```

**编译 Android**

``` shell
source build/envsetup.sh
lunch rk3328_core_jd4-userdebug
make installclean
make -j8
./mkimage.sh
```

## 打包 RK 固件

**在 Linux 下打包固件**

编译完成后使用 Firefly 官方脚本即可打包所有的分区映像成 RK 固件:

``` shell
./FFTools/mkupdate/mkupdate.sh update
```

最终生成的文件是 `rockdev/Image-rk3328_core_jd4/update.img`.

**在 Windows 下打包固件**

在 Windows 下打包 RK 固件 `update.img` 也是很简单的：

1. 拷贝所有在 `rockdev/Image-rk3328_firefly_box/` 目录下编译好的文件到 AndroidTool 的 `rockdev\Image` 目录下。
2. 运行在 AndroidTool 的 `rockdev` 目录下的 `mkupdate.bat` 文件。
3. 在 `rockdev\Image` 目录将会生成 `update.img`。

## 烧写固件

参考：[《升级固件》](03-upgrade_firmware.md) 

[《上手指南》]: started.md
[《常见问题解答》]: faqs.md
[《串口调试》]: debug.md
[《编译 Linux 根文件系统》]: linux_build_ubuntu_rootfs.md
[《编译 Linux 固件》]:linux_compile_gpt.md
[《编译 Android 固件》]:compile_android8.1_firmware.md
[《MaskRom》]:04-maskrom_mode.md
[《编译Linux固件(GPT)》]:linux_compile_gpt.md
[《烧写须知》]: 02-upgrade_table.md
[《ADB 介绍》]: adb_use.md
[Loader模式]:01-bootmode.md#loader_mode
[联系方式]: resource.md#社区
[原始固件]: started.md#raw-firmware-format
[RK 固件]: 02-upgrade_table.md#rk-firmware-format
[分区映像]: started.md#partition-image
[upgrade_tool]:03-upgrade_firmware.md#upgrade-tool
[AndroidTool]:03-upgrade_firmware.md#androidtool
[CORE-RK3328-JD4]:http://www.t-firefly.com/product/coreboard/core_3328_jd4.html?theme=pc
[下载页面]: https://community.t-firefly.com/doc/download/34
[论坛]: http://bbs.t-firefly.com
[脸书]: https://www.facebook.com/TeeFirefly
[Google+]: https://plus.google.com/u/0/communities/115232561394327947761
[油管]: https://www.youtube.com/channel/UCk7odZvUrTG0on8HXnBT7gA
[推特]: https://twitter.com/TeeFirefly
[在线商城]: http://store.t-firefly.com
[USB 转串口适配器]: https://store.t-firefly.com/goods.php?id=24
[5V2A 电源适配器]: https://store.t-firefly.com/goods.php?id=69
[《存储映射》]: http://opensource.rock-chips.com/wiki_Partitions#Default_storage_map