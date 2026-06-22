# Buildroot 开发

Buildroot 是 Linux 平台上一个构建嵌入式 Linux 系统的框架。整个 Buildroot 是由 Makefile(*.mk) 脚本和 Kconfig(Config.in) 配置文件构成的。你可以和编译 Linux 内核一样，通过 buildroot 配置，menuconfig 修改，编译出一个完整的可以直接烧写到机器上运行的 Linux 系统软件（包含 boot、kernel、rootfs 以及 rootfs 中的各种库和应用程序）。

[Buildroot 官网](https://buildroot.org/)

[Buildroot 开发手册](https://buildroot.org/downloads/manual/manual.html)

## 目录结构

```bash
buildroot/
├── arch                # CPU 架构的构建、配置文件
├── board               # 具体单板相关的文件
├── boot                # Bootloaders 的构建、配置文件
├── build
├── CHANGES             # Buildroot 修改日志
├── Config.in
├── Config.in.legacy
├── configs             # 具体单板的 Buildroot 配置文件
├── COPYING
├── DEVELOPERS
├── dl                  # 下载的程序、源码压缩包、补丁等
├── docs                # 文档
├── fs                  # 各种文件系统的构建、配置文件
├── linux               # Linux 的构建、配置文件
├── Makefile
├── Makefile.legacy
├── output              # 编译输出目录
├── package             # 所有软件包的构建、配置文件
├── README              # Buildroot 简单说明
├── support             # 为 Bulidroot 提供功能支持的脚本、配置文件
├── system              # 制作根文件系统的构建、配置文件
├── toolchain           # 交叉编译工具链的构建、配置文件
└── utils               # 实用工具
```

## 配置

选择默认配置文件：

```bash
# 进入 SDK 根目录
cd path/to/SDK/
# 选择配置文件
# `configs/rockchip_rk3288_defconfig`
source envsetup.sh rockchip_rk3288
```

执行完成后会生成编译输出目录，`output/rockchip_rk3288`，后续也可以在该目录下执行 `make` 相关操作。

### 软件包配置

打开配置界面：

```bash
make menuconfig
```

![](../../../rk3288_img/Buildroot_menuconfig.png)

我们可以在配置界面添加或裁剪一些工具，按需求定制系统功能。以添加 `qt53d` 为例：

输入 `/` 进入搜索界面，输入要查找的内容 `qt53d`，按回车进行搜索：

![](../../../rk3288_img/Buildroot_serach_qt53d.png)

![](../../../rk3288_img/Buildroot_qt53d.png)

选择 `1` 跳转到对应页面，按空格选中配置：

![](../../../rk3288_img/Buildroot_select_qt53d.png)

配置完成后，移动到 `Save` 按回车保存到 `.config`；移动到 `Exit` 按回车退出。

![](../../../rk3288_img/Buildroot_Save-Exit.png)

保存配置文件：

```bash
make savedefconfig
```

将修改保存到配置文件 `configs/rockchip_rk3288_defconfig`。

### Busybox 配置

打开配置界面，进行配置：

```bash
make busybox-menuconfig
```

![](../../../rk3288_img/Buildroot_busybox-menuconfig.png)

配置完成后，移动到 `Exit` 按回车退出，在弹窗页面选择 `Yes` 保存到 `.config`。

![](../../../rk3288_img/Buildroot_busybox-save.png)

保存配置文件：

```bash
make busybox-update-config
```

将修改保存到配置文件 `board/rockchip/common/base/busybox.config`。

## 编译

配置好 Buildroot 后，直接运行 `make` 进行编译。

### 编译说明

运行 `make` 进行编译时，会执行以下过程：

1. 下载源码；
2. 配置、编译、安装交叉编译工具链；
3. 配置、编译、安装选择的软件包；
4. 按选择的格式生成根文件系统；

关于 `make` 的更多用法，可通过 `make help` 获得。

### 编译软件包

我们可以执行 `make <package>` 单独编译某个软件包。软件包的编译主要包括下载，解压，打补丁，配置，编译，安装等过程，具体可以查看 `package/pkg-generic.mk`。

* 下载

  Buildroot 会根据配置 `package/<package>/<package>.mk`，自动从网络获取对应的软件包，包括一些第三方库，插件，实用工具等，放在 `dl/` 目录。

* 解压

  软件包会解压在 `output/rockchip_rk3288/build/<package>-<version>` 目录下。

* 打补丁

  补丁集中放在 `package/<packgae>/` 目录，Buildroot 会在解压软件包后为其打上相应的补丁。如果要修改源码，可以通过打补丁的方式进行修改。

* 配置
* 编译
* 安装

  编译完成后，会将需要的编译生成文件拷贝到 `output/rockchip_rk3288/target/` 目录。

对于某个软件包，我们可以通过 `make <package>-<target>` 调用软件包构建中的某一步骤，如下：

```bash
Package-specific:
  <pkg>                  - Build and install <pkg> and all its dependencies
  <pkg>-source           - Only download the source files for <pkg>
  <pkg>-extract          - Extract <pkg> sources
  <pkg>-patch            - Apply patches to <pkg>
  <pkg>-depends          - Build <pkg>'s dependencies
  <pkg>-configure        - Build <pkg> up to the configure step
  <pkg>-build            - Build <pkg> up to the build step
  <pkg>-graph-depends    - Generate a graph of <pkg>'s dependencies
  <pkg>-dirclean         - Remove <pkg> build directory
  <pkg>-reconfigure      - Restart the build from the configure step
  <pkg>-rebuild          - Restart the build from the build step
```

## 编译输出目录

编译完成后，在编译输出目录 `output/rockchip_rk3288` 会生成子目录，说明如下：

* `build/` 包含所有的源文件，包括 Buildroot 所需主机工具和选择的软件包，这个目录包含所有软件包源码。
* `host/` 主机端编译需要的工具，包括交叉编译工具。
* `images/` 包含压缩好的根文件系统镜像文件。
* `staging/` 这个目录类似根文件系统的目录结构，包含编译生成的所有头文件和库，以及其他开发文件，不过他们没有裁剪，比较庞大，不适用于目标文件系统。
* `target/` 包含完整的根文件系统，对比 `staging/`，它没有开发文件，不包含头文件，二进制文件也经过 `strip` 处理。

## 交叉编译工具

Buildroot 编译完成后，会在 `output/rockchip_rk3288/host/` 目录下，生成交叉编译工具，我们可以用来编译目标程序。

* 交叉编译工具目录

`output/rockchip_rk3288/host/bin/`

* 编译示例 hello.c

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
        printf("Hello World!\n");
        return 0;
}
```

* 编译

```bash
.../host/bin/arm-buildroot-linux-gnueabihf-gcc hello.c -o hello
```

* 运行

将可执行程序 `hello` 拷贝到设备，运行 `./hello`，则会看到打印信息 `Hello World!`。

## 重建

对于重建的具体说明，可以查看文档 `buildroot/docs/manual/rebuilding-packages.txt`。

### 重建软件包

在开发过程中，若修改了某个软件包的源码，Buildroot 是不会重新编译该软件包的。可以按如下方式操作：

* 方式一

```bash
make <package>-rebuild
```

* 方式二

```bash
# 删除软件包的编译输出目录
rm -rf output/rockchip_rk3288/build/<package>-<version>
# 编译
make <package>
```

### 完全重建

当通过 make menuconfig，make xconfig 或其他配置工具之一更改系统配置时，Buildroot 不会尝试检测应重建系统的哪些部分。在某些情况下，Buildroot 应该重建整个系统，在某些情况下，仅应重建软件包的特定子集。但是以完全可靠的方式检测到这一点非常困难，因此 Buildroot 开发人员已决定不尝试这样做。

#### 何时需要完全重建

* 更改目标体系结构配置时，需要完全重建；
* 更改工具链配置时，需要完全重建；
* 将其他软件包添加到配置中时，不一定需要完全重建；
* 从配置中删除软件包时，Buildroot 不会执行任何特殊操作。它不会从目标根文件系统或工具链中删除此软件包安装的文件。需要完全重建才能删除这些文件；
* 更改软件包的子选项时，不会自动重建软件包；
* 对根文件系统框架进行更改时，需要完全重建；

一般而言，当你遇到构建错误并且不确定所做的配置更改可能带来的后果时，请进行完全重建。具体说明可以查看文档 `rebuilding-packages.txt`。

#### 如何完全重建

* 方式一

直接删除编译输出目录，之后重新进行配置、编译。

```bash
rm -rf output/
```

* 方式二

执行如下命令，会删除编译输出并重新编译。

```bash
make clean all
```

## 新增本地源码包

开发过程中，Buildroot 自带的软件包有时可能无法满足我们的需求，为此我们需要添加自定义的软件包。Buildroot 支持多种格式的软件包，包括 generic-package、cmake-package、autotools-package 等，我们以 generic-package 举例说明。

* 创建工程目录

```bash
cd path/to/SDK/
mkdir buildroot/package/rockchip/firefly_demo/
```

* 新建 Config.in

在 `firefly_demo/` 下添加 Config.in：

```Kconfig
config BR2_PACKAGE_FIREFLY_DEMO
        bool "Simple Firefly Demo"
```

* 新建 firefly_demo.mk

在 `firefly_demo/` 下添加 firefly_demo.mk：

```makefile
##################################################
###########
#
### firefly_demo
#
##################################################
###########
ifeq ($(BR2_PACKAGE_FIREFLY_DEMO), y)

        FIREFLY_DEMO_VERSION:=1.0.0
        FIREFLY_DEMO_SITE=$(TOPDIR)/../external/firefly_demo/src
        FIREFLY_DEMO_SITE_METHOD=local

define FIREFLY_DEMO_BUILD_CMDS
        $(TARGET_MAKE_ENV) $(MAKE) CC=$(TARGET_CC) CXX=$(TARGET_CXX) -C $(@D)
endef

define FIREFLY_DEMO_CLEAN_CMDS
        $(TARGET_MAKE_ENV) $(MAKE) -C $(@D) clean
endef

define FIREFLY_DEMO_INSTALL_TARGET_CMDS
        $(TARGET_MAKE_ENV) $(MAKE) -C $(@D) install
endef

define FIREFLY_DEMO_UNINSTALL_TARGET_CMDS
        $(TARGET_MAKE_ENV) $(MAKE) -C $(@D) uninstall
endef

$(eval $(generic-package))
endif
```

* 创建源码目录

上文的 Makefile 文件里已经指定了源码目录 `external/firefly_demo/src`。

```bash
cd path/to/SDK/
mkdir external/firefly_demo/src
```

* 编写源码 firefly_demo.c

在 `firefly_demo/src/` 下添加 firefly_demo.c：

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
        printf("Hello World!\n");
        return 0;
}
```

* 编写 Makefile

在 `firefly_demo/src/` 下添加 Makefile：

```makefile
DEPS =
OBJ = firefly_demo.o
CFLAGS =
%.o: %.c $(DEPS)
        $(CC) -c -o $@ $< $(CFLAGS)

firefly_demo: $(OBJ)
        $(CXX) -o $@ $^ $(CFLAGS)

.PHONY: clean
clean:
        rm -f *.o *~ firefly_demo

.PHONY: install
install:
        cp -f firefly_demo $(TARGET_DIR)/usr/bin/

.PHONY: uninstall
uninstall:
        rm -f $(TARGET_DIR)/usr/bin/firefly_demo
```

* 修改上一级 Config.in

在 `buildroot/package/rockchip/Config.in` 末尾添加一行：

```Kconfig
source "package/rockchip/firefly_demo/Config.in"
```

* 配置软件包

打开配置菜单 `make menuconfig`，找到 firefly_demo 并选中配置。

* 编译

```bash
# 编译 firefly_demo
make firefly_demo
# 打包进根文件系统
make
# 若修改源码，重新编译软件包
make firefly_demo-rebuild
```

## rootfs-overlay

rootfs-overly 是一个相当不错的功能，它能够在目标文件系统编译完成后将指定文件覆盖到某个目录。通过这种方式，我们可以方便地添加或修改一些文件到根文件系统。

假设我们要在根文件系统的 `/etc/` 目录下添加文件 `overlay-test`，可以按如下步骤操作：

* 设置 rootfs-overlay 根目录

打开配置菜单 `make menuconfig`，通过设置 `BR2_ROOTFS_OVERLAY` 选项，添加用于覆盖的根目录。对于 rk3288，默认已添加了目录 `board/rockchip/rk3288/fs-overlay/`。

* 添加文件到覆盖目录

```bash
cd buildroot/board/rockchip/rk3288/fs-overlay/
mkdir etc/
touch etc/overlay-test
```

* 编译

```bash
make
```

* 下载根文件系统

将编译好的根文件系统 `output/rockchip_rk3288/images/rootfs.ext2` 下载到设备。启动设备，可以看到已添加文件 `/etc/overlay-test`。

也可以通过查看 `target/` 目录，验证是否添加成功：

```bash
ls buildroot/output/rockchip_rk3288/target/etc/overlay-test
```

## 交叉编译 Qt-5.12.2

### 交叉编译工具链

Firefly 提取了 Buildroot 的交叉编译工具链，用户可以直接使用该工具链开发 Buildroot 上的 Qt 应用程序，而无需下载编译 SDK 代码。

工具链支持 EGLFS、LinuxFB、Wayland 等插件。

使用环境：

```bash
主机：x86-64 / Ubuntu 16.04/18.04
设备： Firefly RK3399 RK3288 PX30 .. / Buildroot
```

#### 下载

对于 32位芯片，如 rk3288，下载 firefly-qt-5.12.2-arm.tar.gz；对于其它 64位芯片，如 rk3399，下载 firefly-qt-5.12.2-aarch64.tar.gz。

[下载链接](https://pan.baidu.com/s/1tMNJ6Ud8QpGXTh4yHjgeFA)(提取码：6dg7)

下文将以 firefly-qt-5.12.2-aarch64.tar.gz 为例进行说明。

#### 目录结构

解压下载好的压缩包：

```bash
tar -zxvf firefly-qt-5.12.2-aarch64.tar.gz
```

目录结构：

```bash
firefly-qt-5.12.2-aarch64/
├── aarch64.tar.gz # 用于 Qt 程序运行的库文件、插件等
├── demo
│   └── mainwindow # demo 程序
├── firefly-qt-5.12.2-aarch64 # 交叉编译工具链
└── ReadMe # 使用说明
```

### 配置交叉编译环境

将工具链拷贝到主机的指定目录：

```bash
cd firefly-qt-5.12.2-aarch64/
# 该目录 `/opt/` 不可修改
cp -rdf firefly-qt-5.12.2-aarch64 /opt/
```

若拷贝失败，先修改下目录权限，再执行上述操作：

```bash
sudo chmod 777 /opt/
```

#### 交叉编译

以 demo 程序为例，运行如下命令：

```bash
cd demo/mainwindow/
/opt/firefly-qt-5.12.2-aarch64/host/bin/qmake
make
```

编译完成后，会生成可执行程序 `demo/mainwindow/mainwindow`。

### 配置运行环境

使用 U盘将 aarch64.tar 拷贝到设备上：

```bash
cp /media/usb0/aarch64.tar /userdata/
```

解压：

```bash
cd /userdata/
tar -xvf aarch64.tar
```

将文件拷贝到对应目录：

```bash
cd aarch64/
cp -rdf usr/lib/* /usr/lib/
cp -rdf usr/qml/* /usr/qml/
cp usr/bin/gdbserver /usr/bin/
```

#### 运行程序

将编译好的可执行程序 mainwindow 拷贝到设备上。

默认板子运行了 Wayland 桌面环境，按如下方式运行：

```bash
# 设置环境变量
export XDG_RUNTIME_DIR=/tmp/.xdg
# 运行
./mainwindow -platform wayland
```

使用 eglfs 插件运行：

```bash
# 退出 Wayland 桌面环境
/etc/init.d/S50launcher stop
# 运行
./mainwindow -platform eglfs
```

### Qt Creator

下面介绍 Qt Creator 的使用说明，在操作前，请先按前面的步骤配置好交叉编译环境和运行环境。

#### 安装

进入 [Qt 官方下载页面](https://download.qt.io/archive/qt/5.12/5.12.2/)，下载 qt-opensource-linux-x64-5.12.2.run，下载完成之后，运行安装。

#### 配置

安装完成后，启动 Qt Creator，打开菜单 `Tools -> Options`，找到 Kits。

* 配置 Qt Versions

  qmake：`/opt/firefly-qt-5.12.2-aarch64/host/bin/qmake`

![](../../../rk3288_img/Qt-config-Versions.png)

* 配置 Compilers

  g++：`/opt/firefly-qt-5.12.2-aarch64/host/bin/aarch64-buildroot-linux-gnu-g++`

  gcc：`/opt/firefly-qt-5.12.2-aarch64/host/bin/aarch64-buildroot-linux-gnu-gcc`

![](../../../rk3288_img/Qt-config-Compilers_1.png)

![](../../../rk3288_img/Qt-config-Compilers_2.png)

为方便调试，配置 Debuggers 和 Devices 用于在线调试：

* 配置 Debuggers

  gdb：`/opt/firefly-qt-5.12.2-aarch64/host/bin/aarch64-buildroot-linux-gnu-gdb`

![](../../../rk3288_img/Qt-config-Debuggers.png)

* 配置 Devices

设置好设备的 IP、用户名(root)和密码(rockchip)。为了方便调试，可以在设备上设置静态 IP。

![](../../../rk3288_img/Qt-config-Devices.png)

* 配置 Kits

将前面设置的配置项添加到 Kits。

![](../../../rk3288_img/Qt-config-Kits.png)

#### 编译运行

打开 demo 程序，`Welcome -> Open Project`，选择要使用的 Kits：

![](../../../rk3288_img/Qt-Choose-Kit.png)

之后打开 `Projects -> Run`，配置命令行参数，这里设置为 `-platform wayland`：

![](../../../rk3288_img/Qt-command_line_arguments.png)

配置环境变量，即 `export XDG_RUNTIME_DIR=/tmp/.xdg`：

![](../../../rk3288_img/Qt-set-environment.png)

编译运行：

点击 `Build` 交叉编译 Qt 程序；点击 `Run` 或 `Debug` 在设备上运行或调试程序。要重新运行程序时，记得手动点击 `Stop` 关闭已经运行的程序。

![](../../../rk3288_img/Qt-Compile.png)

编译生成目录在 `demo/build-mainwindow-qt_5_12_2_aarch64-Debug`。

## 桌面应用

官方发布的 Buildroot 固件，默认已支持 Wayland 桌面环境以及一些 Qt 应用，如下图：

![](../../../rk3288_img/Buildroot_Desktop.png)

这些 Qt 应用提供了一些基础功能，如，摄像头预览，文件管理器，多媒体播放器，WiFi 连接等。

## 用户和密码

* 用户：root
* 密码：rockchip

## WiFi 连接

```bash
# wifi_start.sh wifi-ssid password
wifi_start.sh wifi-test 12345678
```

## 音/视频播放

```bash
# 播放 wav
aplay test.wav
gstwavplay.sh test.wav

# 播放 mp3
mp3play.sh test.mp3
gstmp3play.sh test.mp3

# 播放 mp4
gstmp4play.sh test.mp4
gstvideoplay.sh test.mp4
```

## SSH

Buildroot 默认已支持 SSH 服务。

* ssh 登录设备

让设备处于联网状态，在 PC 端执行登录命令：

```bash
# 用户：root；密码：rockchip
# IP：192.168.27.129
~$ ssh root@192.168.27.129
The authenticity of host '192.168.27.129 (192.168.27.129)' can't be established.
ECDSA key fingerprint is SHA256:+NwFawDiU0EwLHRFrIA/7snmlMc9ZfN6Nxa5vUSC7Pg.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.27.129' (ECDSA) to the list of known hosts.
root@192.168.27.129's password:
[root@rk3288:~]#
```

* scp

在 PC 端执行命令，拷贝文件到设备：

```bash
~$ scp ./file root@192.168.27.129:/
root@192.168.27.129's password:
file                           100%  154     0.2KB/s   00:00
```

## MIPI 摄像头(OV13850)

查看 ISP 驱动是否加载成功，成功可以看到如下设备信息：

```bash
root@firefly:~# grep '' /sys/class/video4linux/video*/name
/sys/class/video4linux/video0/name:rkisp1_mainpath
/sys/class/video4linux/video1/name:rkisp1_selfpath
/sys/class/video4linux/video2/name:rkisp1_dmapath
/sys/class/video4linux/video3/name:rkisp1-statistics
/sys/class/video4linux/video4/name:rkisp1-input-params

root@firefly:~# ls /dev/video*
video0  video1  video2  video3 video4
```

查看 OV13850 设备是否注册成功，成功可以看到如下打印：

```bash
root@firefly:~# dmesg |grep 13850
[    3.911130] ov13850 3-0010: driver version: 00.01.03
[    3.911168] ov13850 3-0010: GPIO lookup for consumer reset
[    3.911171] ov13850 3-0010: using device tree for GPIO lookup
[    3.911187] of_get_named_gpiod_flags: parsed 'reset-gpios' property of node '/i2c@ff150000/ov13850@10[0]' - status (0)
[    3.911370] ov13850 3-0010: could not get default pinstate
[    3.911373] ov13850 3-0010: could not get sleep pinstate
[    3.911376] ov13850 3-0010: GPIO lookup for consumer pwdn
[    3.911379] ov13850 3-0010: using device tree for GPIO lookup
[    3.911403] of_get_named_gpiod_flags: parsed 'pwdn-gpios' property of node '/i2c@ff150000/ov13850@10[0]' - status (0)
# 成功获取设备 ID
[    3.914401] ov13850 3-0010: Detected OV00d850 sensor, REVISION 0xb2
```

预览：

```bash
gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,format=NV12,width=640,height=480, framerate=30/1 ! videoconvert ! kmssink &

# 或者
export XDG_RUNTIME_DIR=/tmp/.xdg
gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,format=NV12,width=640,height=480, framerate=30/1 ! videoconvert ! waylandsink &
```

## 外部存储设备

Buildroot 支持自动挂载外部存储设备：

U 盘挂载路径：`/udisk`

TF 卡挂载路径：`/sdcard`

## 恢复出厂设置

Buildroot 支持恢复出厂设置。直接运行 `update` 或者加上 `factory/reset` 参数均可进入 recovery 恢复出厂设置。

**注意：此出厂设置表示恢复为设备最后一次升级固件之后的初始状态。**

```bash
update
# 或者
update factory / update reset
```

## 升级固件

Buildroot 支持从外部存储设备升级固件，以下是升级流程说明。

* 制作升级固件

按照正常的固件编译流程，制作用于升级的固件。[《编译 Buildroot 固件》](buildroot_compile.md)

升级固件不一定要全分区升级，可修改 `package-file` 文件，将不要升级的分区去掉，这样可以减少升级包的大小。

例如，修改文件 `tools/linux/Linux_Pack_Firmware/rockdev/rk3288-package-file`，将 `rootfs` 的相对路径改为 `RESERVED`，这样就不会打包根文件系统，即不升级根文件系统分区。

```bash
# name          relative path
#
#hwdef          hwdef
package-file    package-file
bootloader      image/miniloaderall.bin
parameter       image/parameter.txt
trust           image/trust.img
uboot           image/uboot.img
misc            image/misc.img
boot            image/boot.img
recovery        image/recovery.img
rootfs          RESERVED
oem             image/oem.img
userdata:grow   image/userdata.img
backup          RESERVED
```

将制作好的升级固件拷贝到 U 盘、TF 卡或者设备的 `/userdata/` 目录下，重命名为 `update.img`。

**注意：** 若将升级固件放至设备的 `/userdata/` 目录，则不要打包 `userdata.img`，将 `image/userdata.img` 改为 `RESERVED`。

* 升级

启动设备，在命令行执行升级命令，`update ota path/to/update.img`，设备会进入 recovery 进行升级。

```bash
# U 盘
update ota /udisk/update.img
# TF 卡
update ota /sdcard/update.img
# /userdata/
update ota /userdata/udpate.img
```

等待升级完成，升级成功后，设备会重新启动进入系统。

## FirmwareOta

firmwareota 是一个 Qt 桌面应用，支持从网络下载固件进行固件升级，下面是使用说明。

### 添加桌面应用

* 下载

[下载链接](https://pan.baidu.com/s/1cvuxjUlbgXHOnr-6gwakww)(提取码：1234)

* 解压

```bash
tar -zxvf firmwareota.tar.gz
```

* 按照目录结构，将文件拷贝到设备

```bash
└── usr
    ├── bin
    │   └── firmwareota
    └── share
        ├── applications
        │   └── firmwareota.desktop
        └── icon
            └── icon_firmwareota.png
```

* 重启桌面

```bash
/etc/init.d/S50launcher stop
/etc/init.d/S50launcher start
```

* 启动

设置完成后，则可以在桌面启动应用。

![](../../../rk3288_img/Buildroot_firmwareota.png)

### 设置下载地址

程序会通过默认的 Http 地址下载列表文件 `ota_list`，解析列表文件获取固件下载链接。

列表文件格式，例：

```bash
firmware_name_1
http://192.168.2.11:8000/firmware_name_1
firmware_name_2
http://192.168.2.11:8000/firmware_name_2
...
```

我们可以通过添加 `/userdata/list_url` 文件，设置指定列表文件下载地址，例：

```bash
http://192.168.2.11:8000/ota_list
```

## Weston 配置

我们可以通过配置 Weston 对显示进行一些自定义设置，下文对部分设置进行说明。

### 状态栏设置

Weston 支持在 weston.ini 配置文件的 shell 段设置状态栏的背景色、位置，以及在 launcher 段设置快捷启动程序，如：

```ini
# /etc/xdg/weston/weston.ini

[shell]
# 颜色格式为 ARGB8888
panel-color=0xff002244
# top|bottom|left|right|none
panel-position=bottom

[launcher]
icon=/usr/share/weston/terminal.png
path=/usr/bin/weston-terminal

[launcher]
# 图标路径
icon=/usr/share/weston/icon_flower.png
# 快捷启动命令
path=/usr/bin/qsetting
```

### 背景设置

Weston 支持在 weston.ini 配置文件的 shell 段设置背景图案、颜色，如：

```ini
# /etc/xdg/weston/weston.ini

[shell]
# 背景图案(壁纸)绝对路径
background-image=/usr/share/weston/background.png
# scale|scale-crop|tile
background-type=scale
# 颜色格式为 ARGB8888，未设置背景图案时生效
background-color=0xff002244
```

### 待机及锁屏配置

Weston 的超时待机时长可以在启动参数中配置，也可以在 weston.ini 的 core 段配置，如：

```bash
# /etc/init.d/S50launcher
    start)
        ...
        # 0 为禁止待机，单位为秒
        weston --tty=2 -B=drm-backend.so --idle-time=0&
```

或者：

```ini
# /etc/xdg/weston/weston.ini

[core]
# 设置 5 秒未操作后进入待机状态
idle-time=5
```

### 显示颜色格式配置

Buildroot SDK 内 Weston 目前默认显示格式为 ARGB8888，对于某些低性能平台，可以在 weston.ini 的 core 段配置为 RGB565，如：

```ini
# /etc/xdg/weston/weston.ini

[core]
# xrgb8888|rgb565|xrgb2101010
gbm-format=rgb565
```

也可以在 weston.ini 的 output 段单独配置每个屏幕的显示格式，如：

```ini
# /etc/xdg/weston/weston.ini

[output]
# output 的 name 可以查看 /sys/class/drm/card0-name
name=LVDS-1
# xrgb8888|rgb565|xrgb2101010
gbm-format=rgb565
```

### 屏幕方向设置

Weston 的屏幕显示方向可以在 weston.ini 的 output 段配置，如：

```ini
# /etc/xdg/weston/weston.ini

[output]
name=LVDS-1
# normal|90|180|270|flipped|flipped-90|flipped-180|flipped-270
transform=180
```

如果需要动态配置屏幕方向，可以通过动态配置文件，如：

```bash
echo "output:all:rotate90" > /tmp/.weston_drm.conf # 所有屏幕旋转 90 度
echo "output:eDP-1:rotate180" > /tmp/.weston_drm.conf # eDP-1 旋转 180 度
```

### 分辨率及缩放配置

Weston 的屏幕分辨率及缩放可以在 weston.ini 的 output 段配置，如：

```ini
# /etc/xdg/weston/weston.ini

[output]
name=HDMI-A-1
# 需为屏幕支持的有效分辨率
mode=1920x1080
# 需为整数倍数
scale=2
```

如果需要动态配置分辨率及缩放，可以通过动态配置文件，如：

```bash
echo "output:HDMI-A-1:mode=800x600" > /tmp/.weston_drm.conf # 修改 HDMI-A-1 分辨率为800x600
```

这种方式缩放时需要依赖 RGA 加速。

### 冻结屏幕

在启动 Weston 时，开机 logo 到 UI 显示之间存在短暂切换黑屏。如需要防止黑屏，可以通过以下种动态配置文件方式短暂冻结 Weston 屏幕内容：

```bash
# /etc/init.d/S50launcher
    start)
        ...
        export WESTON_FREEZE_DISPLAY=/tmp/.weston_freeze # 设置特殊配置文件路径
        touch /tmp/.weston_freeze # 冻结显示
        weston --tty=2 -B=drm-backend.so --idle-time=0&
        ...
        sleep 1 && rm /tmp/.weston_freeze& # 1 秒后解冻
```

### 多屏配置

Buildroot SDK 的 Weston 支持多屏同异显及热拔插等功能，不同显示器屏幕的区分根据 drm 的 name (通过 /sys/class/drm/card0-name 获取)，相关配置通过环境变量设置，如：

```bash
# /etc/init.d/S50launcher

    start)
        ...
        export WESTON_DRM_PRIMARY=HDMI-A-1 # 指定主显为 HDMI-A-1
        export WESTON_DRM_MIRROR=1 # 使用镜像模式(多屏同显)，不设置此环境变量即为异显
        export WESTON_DRM_KEEP_RATIO=1 # 镜像模式下缩放保持纵横比，不设置此变量即为强制全屏
        export WESTON_DRM_PREFER_EXTERNAL=1 # 外置显示器连接时自动关闭内置显示器
        export WESTON_DRM_PREFER_EXTERNAL_DUAL=1 # 外置显示器连接时默认以第一个外显为主显
        weston --tty=2 -B=drm-backend.so --idle-time=0&
```

镜像模式缩放显示内容时需要依赖 RGA 加速。

同时也支持在 weston.ini 的 output 段单独禁用指定屏幕：

```ini
# /etc/xdg/weston/weston.ini

[output]
name=LVDS-1
mode=off
# off|current|preferred|<WIDTHxHEIGHT@RATE>
```

### 输入设备相关配置

Weston 服务默认需要至少一个输入设备，如无输入设备，则需要在 weston.ini 中的 core 段特殊设置：

```ini
# /etc/xdg/weston/weston.ini

[core]
require-input=false
```